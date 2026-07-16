# CLAUDE.md — ROMULUS public site (romulus.vote)

**Read this first.** This repo is the **public, multi-tenant ROMULUS dashboard** served at
**romulus.vote**. It is ONE single-page app (`index.html`) that presents a marketing landing
page, a developer passcode gate, and — after auth — a client dashboard that can switch between
multiple candidate "silos". Everything below maps the file so a fresh chat can work safely.

> ⚠️ This is a LIVE public site. Test locally before every push (see bottom). A push to `main`
> auto-redeploys romulus.vote via GitHub Pages (~1 min). Only ONE active chat should own this repo.

---
## DEPLOYMENT
- Repo: `github.com/amsmith31/tedjo-2026-dashboard` · Local: `C:\Users\ASmith3\Projects\tedjo-2026-dashboard`
- Host: **GitHub Pages**, custom domain via `CNAME` = `romulus.vote`. Push `main` → Pages rebuild.
- The whole app is **one file, `index.html`** (HTML + CSS + JS inline). Assets: `opener.mp4`
  (client-intro film), `opener-promo.mp4` (60-second hero film), `romulus-mark.png` / `romulus-mark`.
- Markdown files (like this one) are inert — Pages ignores them; they never affect what visitors see.

## PAGE FLOW (three stages)
1. **`#landing` — the hero / marketing page** (sections `ld-*`): hero → `ld-what` (the emphasized
   **challenger / incumbent** boxes) → `ld-cap` (**6** capability boxes, gold borders) → `ld-how`
   → `ld-why` → `ld-video` (the 60-sec film) → `ld-pricing` → `ld-demo` → `ld-contact` → footer
   (footer carries the **ROMULUS & CO** registration + Ontario BIN, linked to the ONBIS registry).
   The **"Sign In →"** button (`#ld-access`) hides `#landing` and reveals the gate.
2. **`#splash` / `#pwgate` — REAL sign-in (Supabase Auth).** ⚠️ **There is NO passcode in this file
   any more.** Until 2026-07-15 the gate was `PW="Satorus2026"` in plaintext — and worse, every
   silo was world-readable regardless of it, so anyone could Ctrl+U and read all four clients'
   data. Both are fixed. Users now type a **surname only**; `romLoginToEmail()` appends
   `@romulus.vote`, and `romSignIn()` exchanges it for a JWT at that silo's own `/auth/v1/token`.
   - `romulus@romulus.vote` (master) → sessions in ALL silos → keeps HQ.
   - `<surname>@romulus.vote` (client) → their silo ONLY → HQ nav hidden, picker emptied,
     Back/popstate routed to Overview. Straight to their dashboard; no cinematic.
   - **Never add a `fetch()` to Supabase that bypasses `authedFetch()`** — it is the single choke
     point that attaches the JWT and refreshes it on a 401.
   See the `memory/` notes for the full architecture and the traps.
3. **Dashboard shell** — sidebar nav (`button[data-tab=...]`) + `<section class="view" id="view-*">`
   (hq, home, romulus, message, promises, attacks, council, evidence, candidates, news, web, ads,
   social, mentions). Master starts on **`hq`**; a client never sees it.

## THE VIDEO PLAYERS — there are **THREE**, not two
(This file used to say "two". It cost a chat real time on 2026-07-15. There are three.)
- **Intro sequence** — `#intro-vid` inside `#intro`, the FIRST thing every visitor sees. Clicking
  **Enter ▸** plays BOTH films back-to-back (`opener.mp4` → `opener-promo.mp4`); the `<video>` has no
  `<source>` in the markup because the JS sets `.src` from a `SEQ` array. Control: **Skip**
  (`#intro-skip`) — advances to the next film, then to the hero page.
- **Hero 60-second film** — `#lx-video` (`opener-promo.mp4`) inside `#ld-explainer` (ld-video
  section). Ambient scripted scenes render in `#lx-stage`; clicking the poster/Replay plays the mp4.
  Controls: **Replay** (`#lx-replay`) + **Pause/Play** (`#lx-pause`).
- **Client-intro cinematic** — `#ci-vid` (`opener.mp4`) via `playClientIntro()`. **No longer played**
  as of 2026-07-15 — entering a silo goes straight to the dashboard. The function and markup remain
  but nothing calls them.

**All controls are uniform: `.vid-ctl` (bottom:18px right:18px, flex) + `.vid-btn` (14px).** Keep any
new player on that pattern. All three videos are `object-fit:cover` — `contain` letterboxes, which is
what the black bars were. Note `.ld-film::before/::after` are a *deliberate* 8% cinematic letterbox on
the hero film, hidden during playback via `.filmon` — leave them alone. `#splash-vid` CSS is dead code
(no such element).

## NAVIGATION (fixed — no hard refresh needed)
- `#backBtn` (top bar) is **always visible** and walks up one level per press:
  sub-view → Overview → Municipality picker → **back to the hero page** (`showLanding()` re-shows `#landing`).
- Brand logo (top-left) → Overview. Browser Back/Forward step through tabs via `history` + `popstate`.

## MULTI-TENANT CLIENT REGISTRY (`const CLIENTS`, ~line 859)
Each client is its OWN isolated Supabase project (data never crosses silos). Entry shape:
`{ office, status:"live", fn:"<edge-fn slug or ''>", url:"https://<ref>.supabase.co",
   key:"sb_publishable_…", demo?:true, msgTable?, council?, note? }`.
Only **publishable** keys live here — safe in the browser (RLS is read-only). `connectClient()`
(~line 1848) swaps the active Supabase creds; the dashboard adapts to whatever tables the silo has.
Tier values are normalized across silos ("Very Strong" | `VERY_STRONG` | integer 1–5).

Current clients:
| Client | Municipality | Supabase ref | Edge fn | Notes |
|---|---|---|---|---|
| **Alvin Tedjo** | Mississauga | `lblxjbomfusfhsnwehez` | `romulus` | **DEMONSTRATION** |
| **Rob Burton** | Oakville | `poddxshidtrulrsavxmy` | `legatus` | operational; `msgTable=burton_messaging` |
| **Ted Opitz** | Etobicoke-Lakeshore | `ofzfhrfjtoywniawkkua` | _(none yet)_ | council loaded (Morley+Grimes); news/video/AI being provisioned |

## ADD A NEW CLIENT
1. Create the client's Supabase project + load its data + (optional) deploy its `romulus` Edge Function.
2. Add one entry to `CLIENTS` with its `url` + **publishable** key + `fn`. HQ renders it automatically.
   (See the commented template right below the `CLIENTS` object, and `NEW-CLIENT.md` if present.)

## CONVENTIONS
- Link-sourced + clickable everything. Tiers: Very Strong > Strong > Medium > Adequate.
- Only publishable/anon keys in this HTML — NEVER a service/secret key (this file is public).
- Each client = its own Supabase silo. Never mix client data.

## RECENT CHANGES
**2026-07-15 — the security rebuild (see SECURITY above; this was the big one).**
Replaced the plaintext passcode with real Supabase Auth (surname login, per-silo isolation, master
`romulus@romulus.vote`); locked 78 open `SELECT`-to-`public` policies across all four silos; added
`callerRole()` guards to all four chat functions. Also: "Developer Access" → "Sign In"; removed the
client-intro cinematic from silo entry entirely; unified all video controls to one `.vid-ctl` /
`.vid-btn` pattern (bottom-right, 14px) across **three** players — `#intro-vid`, `#lx-video`,
`#ci-vid` — and set all three to `object-fit:cover` (the intro Skip button used to sit top-right and
two players letterboxed). Added `#ld-contact` + the ROMULUS & CO / Ontario BIN footer registration.
NOTE: `playClientIntro()` / `#client-intro` are now unused; `#splash-vid` CSS is dead (no element).

**2026-07 (earlier).** Removed the 4 hero stat boxes; condensed capabilities 8→6 (matching gold
borders); emphasized the challenger/incumbent boxes; added Pause/Play to BOTH videos; made the back
arrow always-visible and walk all the way to the hero page.

## DEFERRED (do not touch until asked)
- ~~The hero-page contact email = the operator's Gmail~~ — **done**; it is `andrew.smith@romulus.vote`
  everywhere (footer, contact bar, `#ld-contact`, and the FormSubmit target on the demo form).

## SECURITY — DO NOT REGRESS (2026-07-15)
- **Never put a password, passcode or secret key in `index.html`.** It is a public file; View Source
  is two keystrokes. Only publishable keys belong here — and they are useless on their own now.
- **Every silo's tables are `SELECT` to `authenticated` only.** Do not "fix" a blank tab by opening a
  policy back up to `public`/`anon` — that reopens the leak on every client at once.
- **Each silo's chat Edge Function carries a `callerRole()` guard** rejecting any caller whose JWT
  role isn't `authenticated`. `verify_jwt=true` is **NOT** sufficient by itself — it accepts the
  publishable key printed in this very file. Proven: with `verify_jwt` on, a stranger using the
  scraped key was handed a full OPPO briefing on a client's own opponent. Never remove that guard.
- **Two things that look like dead code but are load-bearing:** Tedjo's `leads` table must keep
  `INSERT` to `anon` (the public demo form writes to it, and that visitor is never logged in — it has
  no SELECT policy, so it is already write-only and correct); and Burton's Edge Function is still
  *named* `legatus` (an old slug for what the UI calls "ROMULUS Chat") — deleting it kills his chat.

## ROMULUS UNIFORMITY STANDARD (Burton is the reference — applies to EVERY silo)
This is the ONE shared file that renders EVERY client silo, so all UI must be uniform — a fix here
serves all silos and every future client. Burton is the standard to match.
- **Candidates tab — grouped and ordered BY RACE, never a flat scroll.** Mayoral first, then each Ward
  in order (Ward 1, 2, 3…), then school-board/other; incumbent/principal marked. A viewer must find any
  candidate at a glance. Derive the race key from `office_group`, or **parse it from the `office` string
  when `office_group` is missing** (e.g. Mississauga rows are `office="Mayor of Mississauga"` with no
  office_group — they must still land under "Mayor", not "Other"). This mirrors the Social tab's existing
  race-grouping (`socRaceKey`/`groupLabel`) — reuse that pattern for Candidates.
- **OPPO tab — pick a race, then a candidate. No "All".** Same cascade as Candidates/Evidence/
  Promises (`_opRace` → `_opCand`); nothing renders until both are chosen. Race key comes from
  `socRaceKey()` — **never** `office_group` alone. (Until 2026-07-15 OPPO used `office_group||"other"`,
  which filed Mississauga's entire mayoral race — Tedjo AND Crombie, both `office_group=null`,
  `office="Mayor of Mississauga"` — under "Other Races".)
- **The principal appearing in their OWN silo's OPPO list is INTENTIONAL** — self-opposition research
  (know your own vulnerabilities) is deliberate campaign practice. Rob Burton has 13 findings on
  himself in his own silo. **Do not "fix" this.** Confirmed by Andrew 2026-07-15.
- **Race LABELS are not uniform across silos**, because each silo's `office_group` values differ
  (Burton: clean `mayor`/`ward1`; Opitz renders "Ward 3 Etobicoke Lakeshore"; Monica "Councillor,
  City - Wards 7, 8"). That is a DATA normalisation job for each silo's own chat, not a fix for
  this file — `groupLabel()` already prettifies whatever it is given.
- **OPPO video pins are a STANDARD for EVERY candidate in EVERY silo, present and future.** Any OPPO
  finding whose `source_citations` contains a YouTube URL renders a thumbnail pin (bottom-right, above
  the primary-source row), deep-linked to the timestamp. The DISPLAY is already universal — `renderAttacks()`
  builds pins from `allUrls(source_citations)` filtered by `ytId()`, with no per-silo/per-candidate gating
  (Opitz/Morley is the reference). So pins appear automatically the moment the DATA exists. What makes them
  exist is each silo chat's job: OPPO findings backed by an interview/video MUST cite a real
  `https://www.youtube.com/watch?v=<id>&t=<sec>s` URL (several moments space-separated = several pins),
  NEVER a `brief:Section N` stub. The transcript→timestamp→citation pipeline (see the Burton silo:
  `mine_transcripts.py` / `mine_transcripts_whisper.py` / `generate_oppo.py`) is the reference for
  producing them. Opitz + Monica already do; Burton + Tedjo are being backfilled. Nothing to change in
  THIS file — it's a data-population standard the silo chats own.
- **Every silo's ROMULUS chat Edge Function must identify its OWN candidate.** New silo functions
  are copy-pasted from an existing one; if the `SYSTEM` prompt + dossier `ctx` headers aren't fully
  relabelled, the chat introduces itself as the wrong client (a cross-client bleed). This actually
  happened: Monica's function said "analyst for TED OPITZ's 2026 campaign for Toronto City Council,
  Ward 3" (2026-07-16). Before deploying ANY silo's `romulus`/`legatus` function, grep its source for
  every OTHER candidate/city name and confirm zero foreign references. The silo chats own this.
- **Same tab set, order, naming, and card styling for every silo.** No silo-specific layout drift.
- **Improve once, apply everywhere.** Verify on all four silos in preview before pushing.

## HOW TO WORK IN HERE (safety)
- **Test locally before every push:** `python -m http.server 8790` in this folder → open
  `http://127.0.0.1:8790/index.html`; check the console for errors and click through the flow.
- Commit → `git push origin main` redeploys romulus.vote. Pull first if unsure.
- ONE active chat owns this repo at a time (two chats pushing = git conflicts). Chats are disposable —
  a fresh one reads this file and continues.
- Commit messages end with: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.

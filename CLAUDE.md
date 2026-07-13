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
   → `ld-why` → `ld-video` (the 60-sec film) → `ld-demo`. The **"Developer Access →"** button
   (`#ld-access`, ~line 488) hides `#landing` and reveals the gate.
2. **`#splash` / `#pwgate` — passcode gate.** Admin code `PW="Satorus2026"` (~line 738; client-side
   only — NOT real security). Correct code → `enterTerminal()` hides the splash → dashboard shows.
   **"← Back to site"** (`#pw-back`) re-shows `#landing`.
3. **Dashboard shell** — sidebar nav (`button[data-tab=...]`) + `<section class="view" id="view-*">`
   (hq, home, romulus, message, promises, attacks, council, evidence, candidates, news, web, ads,
   social, mentions). Starts on **`hq`** (Municipality → candidate picker).

## THE TWO VIDEOS (both now have pause)
- **Hero 60-second film** — `#lx-video` (`opener-promo.mp4`) inside `#ld-explainer` (ld-video
  section). Ambient scripted scenes render in `#lx-stage`; clicking the poster/Replay plays the mp4.
  Controls: **Replay** (`#lx-replay`) + **Pause/Play** (`#lx-pause`) — player logic ~line 660.
- **Client-intro cinematic** — `#ci-vid` (`opener.mp4`), plays via `playClientIntro()` (~line 1815)
  when you enter a client silo. Controls: **Skip** (`#ci-skip`) + **Pause/Play** (`#ci-pause`).

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

## RECENT CHANGES (2026-07)
Removed the 4 hero stat boxes; condensed capabilities 8→6 (matching gold borders); emphasized the
challenger/incumbent boxes; added Pause/Play to BOTH videos; made the back arrow always-visible and
walk all the way to the hero page.

## DEFERRED (do not touch until asked)
- The hero-page contact email on file = the operator's Gmail; a change is planned LATER.

## ROMULUS UNIFORMITY STANDARD (Burton is the reference — applies to EVERY silo)
This is the ONE shared file that renders EVERY client silo, so all UI must be uniform — a fix here
serves all silos and every future client. Burton is the standard to match.
- **Candidates tab — grouped and ordered BY RACE, never a flat scroll.** Mayoral first, then each Ward
  in order (Ward 1, 2, 3…), then school-board/other; incumbent/principal marked. A viewer must find any
  candidate at a glance. Derive the race key from `office_group`, or **parse it from the `office` string
  when `office_group` is missing** (e.g. Mississauga rows are `office="Mayor of Mississauga"` with no
  office_group — they must still land under "Mayor", not "Other"). This mirrors the Social tab's existing
  race-grouping (`socRaceKey`/`groupLabel`) — reuse that pattern for Candidates.
- **Same tab set, order, naming, and card styling for every silo.** No silo-specific layout drift.
- **Improve once, apply everywhere.** Verify on all three silos in preview before pushing.

## HOW TO WORK IN HERE (safety)
- **Test locally before every push:** `python -m http.server 8790` in this folder → open
  `http://127.0.0.1:8790/index.html`; check the console for errors and click through the flow.
- Commit → `git push origin main` redeploys romulus.vote. Pull first if unsure.
- ONE active chat owns this repo at a time (two chats pushing = git conflicts). Chats are disposable —
  a fresh one reads this file and continues.
- Commit messages end with: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.

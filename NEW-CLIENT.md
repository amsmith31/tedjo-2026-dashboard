# NEW-CLIENT.md — Onboard a new ROMULUS client

Use this when adding a new candidate/client to ROMULUS. It is both a **checklist** and a
**kickoff you can paste into a fresh Claude Code chat** for that client's backend. Every client is
its own isolated Supabase silo; the public site (romulus.vote) just registers it. Keep the ROMULUS
quality bar: link-sourced, tiers Very Strong > Strong > Medium > Adequate, nothing fabricated,
never reuse another client's secrets.

## 0. Gather from the operator (ask — never invent)
- Candidate name, office, municipality, election date, filing deadline.
- Council/agenda portal base URL for that municipality (each city differs; may be WAF-blocked).
- Their Supabase project: **URL + publishable key + service key** (create the project first).
- Optional keys: `GEMINI_API_KEY` (for the chat Edge Function), `ANTHROPIC_API_KEY` (AI analysis),
  `META_ACCESS_TOKEN` + numeric page IDs (political ads), `YOUTUBE_API_KEY`.

## 1. Stand up the client's backend (its own repo)
Create a repo `<client>-2026-intel`. Paste the full ROMULUS build prompt into a fresh Claude Code
chat there — the master template is `Downloads\ROMULUS_Template_Alvin_Tedjo.md` (generalize the
candidate/municipality values). That prompt builds: schema (16 tables + RLS public-read), scrapers
(council/web/news/youtube — city-site scrapers run LOCALLY on a residential IP if the muni WAF blocks
datacenter IPs), AI analysis jobs, and the Gemini `romulus` Edge Function.
- Put secrets in that repo's gitignored `.env`; never commit them.
- Add a `CLAUDE.md` to that repo (copy Burton's as the pattern) so future chats pick up instantly.

## 2. Load the client's data
Candidates (with socials + dossiers in `manual_research_brief`), council record (motions/votes with
deep source URLs → `council_evidence`), promises, messaging, news. Everything sourced and clickable.

## 3. (Optional) Deploy the chat Edge Function
`supabase functions deploy romulus --no-verify-jwt --project-ref <REF>` and
`supabase secrets set GEMINI_API_KEY=…`. Grounded: answers only from the DB, cites every claim.

## 4. Register the client on the PUBLIC site (this repo, romulus.vote)
Edit `index.html` → the `CLIENTS` object (~line 859). Add ONE entry — **publishable key only**:
```js
"<Municipality>": { candidates: {
  "<Candidate Name>": { office:"<office> · 2026", status:"live",
    fn:"romulus" /* Edge Function slug, or "" if none yet */,
    url:"https://<ref>.supabase.co", key:"sb_publishable_…" /* NEVER a secret key */ } } }
```
The HQ picker renders it automatically and the dashboard adapts to whatever tables the silo has.

## 5. Verify (before you tell anyone)
- Test locally: `python -m http.server 8790` → open the site → Developer Access → passcode →
  pick the new client → confirm data loads and the console has no errors.
- Then commit + push this repo (redeploys romulus.vote in ~1 min). Pull first if unsure.

## Golden rules
- ONE active chat per repo at a time (two chats pushing one repo = git conflicts).
- Public `index.html` = publishable keys ONLY. Service/secret keys live in each backend's `.env`.
- City-site scrapers run locally when the muni WAF blocks the cloud; third-party APIs run in CI.

# ROMULUS silo-chat kickoff template

The reusable brief for standing up (or realigning) a per-client DATA chat, so every silo
— and every new client — runs to the same standard. Fill the `<…>` fields and paste it as
the first message to a Claude Code chat opened on that client's `-intel` repo.

## The template

```
You are the <CLIENT NAME> 2026 silo DATA chat — <office>, <municipality>. You own the
Supabase data + intel pipeline for project <supabase-ref> that feeds this client's
dashboard on romulus.vote. Repo: <repo-name>.

LANE — DATA ONLY:
- You own: Supabase tables, the scrapers/transcribers/oppo-generators in this repo, and
  this client's edge function.
- You do NOT edit ../tedjo-2026-dashboard (the shared romulus.vote dashboard). It's a
  separate repo/chat and already renders your data. If something doesn't show on the
  dashboard, the fix is almost always your DATA, not the dashboard.

CRITICAL (2026-07-15 security rebuild):
- Every silo's tables are SELECT-to-authenticated. A publishable/anon key reads ZERO
  rows. ALL local scripts must read with SUPABASE_SERVICE_KEY (in .env, gitignored).
  Audit your scrapers now — this silently breaks anything still using a publishable key.

OPPO VIDEO PINS — the cross-silo standard for EVERY candidate:
- Any OPPO finding whose source_citations holds a real
  https://www.youtube.com/watch?v=<id>&t=<sec>s URL renders a video pin automatically
  (bottom-right, above the primary source). NEVER "brief:Section N" stubs.
- Pipeline: transcript (captions or Whisper) -> pick quotes the CANDIDATE said, never the
  host -> match each to its timestamp -> write the &t= citation. Reference implementation:
  the Burton tools (generate_oppo.py / mine_transcripts_whisper.py) in
  ../burton-silo-tools.
- tier values allowed by attack_messaging: VERY_STRONG | STRONG | MEDIUM | WEAK.

Confirm: (1) you are the SOLE chat on this repo, (2) your read scripts use the service
key. Then continue.
```

## Registry
| Client | Municipality | repo | Supabase ref | Notes |
|---|---|---|---|---|
| Ted Opitz | Etobicoke-Lakeshore (Toronto W3) | opitz-2026-intel | ofzfhrfjtoywniawkkua | challenger; rivals Morley + Grimes; timestamped OPPO is the reference |
| Monica Singh Soares | Brampton W7&8 | sinesoares-2026-intel | jjqzzozpygaiptkjezbg | challenger vs incumbent Rod Power |
| Alvin Tedjo | Mississauga (Mayor) | tedjo-2026-intel | lblxjbomfusfhsnwehez | DEMO client; rival Crombie; `leads` keeps anon INSERT — don't lock it |
| Rob Burton | Oakville (Mayor) | burton-2026-intel | poddxshidtrulrsavxmy | edge fn slug is `legatus`; O'Meara + JHT pins already live |
| Sammy Ijaz | Milton (Regional Cllr) | (new) | (create) | NEW — needs project + data + registry entry + login before it appears |

## New-client sequence (the dashboard chat, not the silo chat, does the last two)
1. Silo chat: create the Supabase project, load candidates + council record + run the
   video/OPPO pipeline (copy tools from ../burton-silo-tools).
2. Silo chat: hand the project ref + publishable key to the dashboard chat.
3. Dashboard chat: add the client to `CLIENTS` in index.html.
4. Dashboard chat: create the client's Supabase Auth login (surname@romulus.vote).
Only after 3+4 does the client appear on romulus.vote.

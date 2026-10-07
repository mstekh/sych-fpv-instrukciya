# CLAUDE.md

**Read `AGENTS.md` first and follow it.** It holds the mandatory rules for this repository
(remote is the source of truth, fetch before any edit, single-writer lock, checkpoints,
commit + push + verify at the end of every session). Its rules take precedence over
anything below.

Then read `docs/PROJECT_STATE.md` and the newest entries in `docs/WORKLOG.md`.

Start a session with `PROJECT_GUARD_AGENT=claude python3 tools/project_guard.py start`
and end it with `python3 tools/project_guard.py finish -m "<summary>"`.

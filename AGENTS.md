# AGENTS.md — mandatory rules for every agent and human

Applies to **everyone** who changes this repository: Claude Code, ChatGPT/Codex,
Cursor, GitHub Copilot, any other AI agent, and humans, on any computer.
If a tool-specific file (`CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, …)
disagrees with this file, **this file wins**.

> Project context: read `docs/PROJECT_STATE.md` (what this is, current state, next steps)
> and the latest entries of `docs/WORKLOG.md` before doing anything.

## 1. The six rules

1. **NO EDIT BEFORE FETCH.** The REMOTE (`origin`) is the single source of truth.
   Before any change run `git fetch --all --prune --tags` and compare local HEAD with the remote.
2. **NO WORK ON OUTDATED COPY.** If local is behind, fast-forward only (`git merge --ff-only`).
   If local and remote have **diverged** or a merge **conflicts** → **STOP**. Do not merge,
   rebase or reset automatically. Report the situation to the human and wait.
3. **NO PARALLEL WRITERS.** Only one machine/agent writes at a time. Take the remote
   single-writer lock (section 3) before editing; if someone else holds it → do not edit.
4. **NO SERIOUS CHANGE WITHOUT CHECKPOINT.** Before a risky or large change, commit + push
   the current state and push a `checkpoint/<timestamp>` tag.
5. **NO SESSION END WITHOUT COMMIT + PUSH + VERIFY.** At the end of every working session:
   commit, push, and verify that the remote branch SHA equals local HEAD. After a milestone,
   also push a tag.
6. **NO LOST CONTEXT.** After meaningful work update `docs/PROJECT_STATE.md` and append an
   entry to `docs/WORKLOG.md` (newest on top) in the same commit.

## 2. Forbidden without explicit, per-case human approval

- `git reset --hard`, `git clean -f*`, `git checkout -- .`/`git restore .` over others' work,
  `git push --force` / `--force-with-lease` to shared branches, history rewrites
  (rebase/amend/filter) of pushed commits, deleting branches or tags you did not create.
- Deleting or overwriting **uncommitted changes of the user**. If the working tree is dirty
  when you arrive, leave those changes alone: ask, or work around them.
- Committing secrets (`.env`, keys, tokens, credentials). Use `.gitignore`.
- Large refactors of application code that were not requested.

## 3. Single-writer lock

The lock is a remote branch **`agent-lock`** holding one file `LOCK.json`
(owner, branch, acquired_at, expires_at, released). Every acquire/release is a new commit
on top of the tip you observed, pushed **without `--force`**: if anyone moved the tip in
between, git rejects the push as non-fast-forward. That compare-and-swap is the mutex.
The branch is never deleted or force-pushed; `"released": true` in the tip means free.

| Step | Command |
|---|---|
| Check state + lock | `python3 tools/project_guard.py status` |
| Start session (fetch, ff-only sync, take lock) | `python3 tools/project_guard.py start` |
| Checkpoint before a serious change | `python3 tools/project_guard.py checkpoint -m "before X"` |
| End session (commit, push, verify, release lock) | `python3 tools/project_guard.py finish -m "summary"` |
| Milestone | `python3 tools/project_guard.py finish -m "summary" --tag vX.Y` |

Set `PROJECT_GUARD_AGENT=claude|codex|cursor|copilot|human` so the lock shows who holds it.
Lock TTL is 4 h by default. An **expired** lock may be taken over with `start --take-stale`
only after checking that nobody is actually working. A live lock held by someone else
means: **do not edit**.

Without Python, do the same by hand:
`git fetch --all --prune --tags` → `git fetch origin agent-lock && git show FETCH_HEAD:LOCK.json`
(absent, or `"released": true`, or expired) → work → commit → push → `git ls-remote origin refs/heads/<branch>` equals
`git rev-parse HEAD`.

Cloud agents that cannot keep a lock across a session (e.g. a single PR-producing run)
must at least: fetch first, check that `agent-lock` is free or theirs, work on their own
branch, and never push to a branch someone else holds the lock for.

## 4. Session checklist

```
[ ] git fetch --all --prune --tags          (or project_guard.py start)
[ ] local == remote, or fast-forwarded; diverged → STOP
[ ] lock free → taken
[ ] read docs/PROJECT_STATE.md + top of docs/WORKLOG.md
[ ] checkpoint before serious changes
[ ] update docs/PROJECT_STATE.md + docs/WORKLOG.md
[ ] commit + push + verify remote SHA       (or project_guard.py finish)
[ ] milestone → tag
[ ] lock released
```

## 5. Unknowns

If something cannot be checked (build, tests, hardware behaviour), write **UNKNOWN** or
**UNVERIFIED** in the docs. Never invent results.

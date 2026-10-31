# TEAM RULES (read once, follow always)

## Who owns what
| Person | Owns (only edit these) | Branch name |
|---|---|---|
| A | `frontend/` | `person-a/frontend` |
| B | `backend/` | `person-b/backend` |
| C | `fusion/`, `eval/` | `person-c/fusion-eval` |
| Everyone (ask the group first) | `shared/`, `docs/API_CONTRACT.md`, `docs/PROJECT_SPEC.md`, `requirements.txt`, `.env.example` | - |

Personal progress files: `docs/PROGRESS_A.md`, `docs/PROGRESS_B.md`, `docs/PROGRESS_C.md` (each person edits only their own).
Rule of thumb: **if it is not in your folder, don't touch it.** That is how we avoid git conflicts.

## Git in 6 lines (each person, on their own branch)
```bash
git status                      # what changed?
git add .                       # stage everything
git commit -m "feat(fusion): add fuse prompt v1"   # save a snapshot
git push                        # upload your branch
git pull origin main            # get teammates' merged work (do this every ~2 hours)
```
Commit message style: `feat(...)` new thing, `fix(...)` bug fix, `docs(...)` docs, `data(...)` dataset, `eval(...)` evaluation. Commit small and often.

## Merge schedule (to main)
| When | What |
|---|---|
| Hour ~4 | B merges the first working backend skeleton (health + stt) so everyone has a live URL |
| **Hour ~10** | **First full integration.** Everyone merges to main, pulls main, tests end to end |
| Hour ~16 | Second merge |
| **Hour 18** | **FEATURE FREEZE.** Only bug fixes after this |
| Hour ~21 | Final merge, deploy, demo rehearsal |

How to merge your branch into main (one person at a time, tell the group first):
```bash
git checkout main
git pull origin main
git merge <your-branch>
git push origin main
git checkout <your-branch>
git merge main                   # bring main's latest back into your branch
```
(or open a Pull Request on GitHub and have a teammate click Merge)

## Secrets
- Keys live only in `.env` (ignored by git). **Never** commit keys, never paste them in the group chat, never hard-code them.
- Everyone has their own `.env`. Check `.gitignore` already lists `.env` (it does).
- ElevenLabs credits are limited: **cache results, don't run loops that spend credits**.

## Contract rules
- `docs/API_CONTRACT.md` is the law. Need a change? Tell everyone, edit it, bump the version, everyone pulls.
- If you build something that disagrees with the contract, **you** fix yours, not theirs.

## Voices and data
- Only use voices of teammates who agreed. Never clone anyone else.
- Simulated data is always named `SIMULATED_*` and described as simulated.
- Do not commit consent recordings or raw voice samples (they are git-ignored).

## Definition of done (per feature)
1. It works on your machine. 2. It follows the contract. 3. It has a quick test. 4. It is committed and pushed. 5. You updated your progress file. 6. You told the group in chat.

## Daily rhythm
Every 2 hours: 5-minute check-in (what's done, what's blocked, what do you need from someone). Say it early if you're stuck for more than 20 minutes.

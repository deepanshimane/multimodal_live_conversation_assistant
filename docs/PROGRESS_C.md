# PROGRESS - Person C (LLM fusion + data + evaluation + demo)

> Claude updates this file after every step. Percent = finished steps / 17.

**Overall: 0% (0 of 17 steps done)**
**Last updated:** (not started)

| # | Step | Status | Target hour |
|---|---|---|---|
| 0 | Check my computer, keys, and tools | ⬜ not started | 0-1 |
| 1 | Claude reads everything + creates fusion/eval skeleton | ⬜ not started | 1 |
| 2 | LLM client (Anthropic/Gemini switch) | ⬜ not started | 1-2 |
| 3 | Fusion prompt v1 + `fuse.py` | ⬜ not started | 2-4 |
| 4 | Reply-suggestion prompt + `replies.py` | ⬜ not started | 4-5 |
| 5 | Hand-test 10 cases (English, Hindi, Hinglish) | ⬜ not started | 5-6 |
| 6 | FastAPI router `fusion/router.py` + tests | ⬜ not started | 6-7 |
| 7 | Test sentences dataset (60 rows + labels) | ⬜ not started | 6-8 |
| 8 | Record clean audio + validate recordings | ⬜ not started | 8-9 |
| 9 | `simulate.py` (make atypical speech, label SIMULATED) | ⬜ not started | 9-11 |
| 10 | Simulated signals generator (gestures/face/lips) | ⬜ not started | 11-12 |
| 11 | `evaluate.py` (3 modes + metrics + cache), small test | ⬜ not started | 12-14 |
| 12 | Full evaluation run #1 + failure analysis | ⬜ not started | 14-16 |
| 13 | Improve prompts (up to 3 rounds), rerun, before/after | ⬜ not started | 16-18 |
| 14 | Results report, charts, `summary.json` | ⬜ not started | 18-20 |
| 15 | Integration help with A and B (first full run was at hour ~10) | ⬜ not started | 10 and 18-20 |
| 16 | Final docs: README parts, demo script, slides text, backup plan | ⬜ not started | 20-23 |

Legend: ⬜ not started, 🔄 in progress, ✅ done, ⛔ blocked

## Log (newest first)
- (nothing yet)

## Numbers so far (fill in as they appear)
| Metric | Value |
|---|---|
| WER raw Scribe | - |
| WER with fusion | - |
| Intent top-1 (voice only / +context / full fusion) | - / - / - |
| Intent top-3 (voice only / +context / full fusion) | - / - / - |
| Latency fuse mean / p95 | - |
| Latency total mean / p95 | - |

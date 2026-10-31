# CONTEXT FILE FOR CLAUDE - PERSON C
*(Give this whole file to Claude in Antigravity at the start of every new chat. If Antigravity has a "rules" or "agent instructions" setting, paste it there too so it is never forgotten.)*

---

## 0. HOW TO USE THIS FILE (for the human, Person C)
1. Open the repo folder in Antigravity. Start a chat with Claude.
2. Paste: **"Read docs/CLAUDE_CONTEXT_PERSON_C.md fully, then read docs/PROJECT_SPEC.md, docs/API_CONTRACT.md, docs/TEAM_RULES.md, and everything in shared/. Then tell me what you understood in 10 simple lines and show the progress report."**
3. After that, just type one of these short commands:

| You type | Claude does |
|---|---|
| `status` | Prints the Progress Report (section 12) |
| `do step N` | Does step N from section 13 completely |
| `next` | Does the next unfinished step |
| `explain` | Explains what was just done in very simple words |
| `what should I do now` | Tells me the exact thing **I** (the human) must do right now |
| `check my work` | Runs the tests/checks for the current step and tells me pass/fail |
| `stuck` | Claude diagnoses the problem step by step, simply |
| `save` | Claude commits my work with a good message (and asks before pushing) |

---

## 1. WHO YOU ARE AND HOW TO TALK (rules for Claude)
You are the coding partner of **Person C**, a **beginner**, in a **24-hour hackathon** with 2 teammates (A = frontend, B = backend). Follow these rules ALWAYS:

1. **Use very simple, friendly language.** No jargon without a one-line explanation in brackets. Short sentences. Example: "WER (word error rate = what % of words are wrong)".
2. **Say what you are about to do, do it, then say what happened** in plain words. Never silently do big things.
3. **After finishing ANY step (or any big chunk), update `docs/PROGRESS_C.md` and print the Progress Report** (section 12). Never forget this. The human must always know where the project stands.
4. **Do not touch files outside `fusion/`, `eval/`, and `docs/PROGRESS_C.md`.** `frontend/` is Person A's, `backend/` is Person B's, `shared/` and the contract need the whole team's OK. If something outside seems wrong, tell me the exact message to send to the teammate instead.
5. **Never break the API contract** (`docs/API_CONTRACT.md`). If you think it must change, stop and tell me why in simple words, and write the exact message I should send to my team.
6. **Never write real API keys in any file.** Keys live only in `.env` (git-ignored). If a key is missing, tell me how to get it and where to paste it.
7. **Ask at most ONE question at a time**, and only if you really cannot continue. Otherwise make a sensible choice, tell me what you chose, and move on.
8. **Check docs, don't guess.** Before writing code that calls ElevenLabs or an LLM API, check the official documentation (use web search/browse if you can) because model names and parameters change. Model IDs we use are in `.env.example`.
9. **Save money and time:** ElevenLabs credits are limited. Cache every paid API result on disk (`eval/.cache/`) so re-running never pays twice. Never put paid calls inside an unbounded loop. Before any big run, tell me the approximate number of API calls and ask "go?".
10. **Test everything you write** with a tiny run, and show me the result. If a test fails, fix it before moving on. Do not tell me something works unless you ran it.
11. **Be honest about results.** If numbers are bad, report them as they are and suggest improvements. Never fake or tune numbers to look good. All test data is SIMULATED and must be labelled that way everywhere.
12. **Keep code beginner-readable:** small functions, comments in plain English at the top of each file ("what this file does"), clear names, and no clever tricks.
13. **Commit often** on my branch `person-c/fusion-eval` (section 14). Never commit to `main`. Never push without telling me.
14. If a step is going to take more than ~45 minutes, tell me and offer a smaller version that still works ("minimum viable").
15. **Always prefer the simplest thing that works** for a demo. The hackathon is judged on: working end-to-end demo + real evaluation numbers + honesty + consent.

---

## 2. THE PROJECT IN SIMPLE WORDS
We are building **Voice Bridge**: a web app that helps people with speech impairments (ALS, cerebral palsy, Parkinson's, stroke, laryngectomy) take part in live conversations.

The patient tries to say something. Normal speech-to-text fails because their speech is unclear. Our app uses **three kinds of clues**:
1. **Voice**: ElevenLabs Scribe turns audio into text (it will often be wrong/garbled).
2. **Camera**: MediaPipe (running in the browser) tracks **hands/gestures**, **face expressions**, and **lip movement**.
3. **Conversation**: what the other person just said, plus a small patient profile.

An **LLM** combines all clues and returns **3-4 ranked guesses** of what the patient meant. The patient confirms one with one easy action. The app then **speaks** it using an ElevenLabs voice (cloned from a **consenting** person, or a designed voice). When the other person replies, the app transcribes it and **suggests 4 replies**.

Languages: **English, Hindi, Hinglish (code-mixed)**.

### Judging checklist (everything the problem statement demands)
- [ ] Real-time: camera landmarks + voice -> fused intent
- [ ] LLM ranks candidate intents using conversation context
- [ ] Listens to the other person and suggests replies
- [ ] Speaks the confirmed message in a personalised voice
- [ ] ElevenLabs used for STT and voice generation (mandatory)
- [ ] English, Hindi, Hinglish
- [ ] Consent enforced for voice cloning
- [ ] Measured where Scribe fails on atypical speech
- [ ] WER with vs without gesture fusion
- [ ] Intent accuracy (top-1, top-3)
- [ ] End-to-end latency (target < 2 s)
- [ ] User effort to confirm a reply
- [ ] Simulated data clearly labelled

---

## 3. THE TEAM AND WHO OWNS WHAT
| Person | Job | Folder |
|---|---|---|
| A | Frontend: webcam, MediaPipe, UI, confirm methods, PWA | `frontend/` |
| B | Backend: FastAPI, ElevenLabs STT/TTS, voice clone/design, **consent enforcement**, deploy | `backend/` |
| **C (me)** | **LLM fusion, reply suggestions, simulated dataset, evaluation, results, demo script and slides text** | **`fusion/`, `eval/`** |

My deliverables to the team:
- `fusion/router.py` exposing a FastAPI `router` with `POST /api/fuse` and `POST /api/replies` (B mounts it).
- `eval/results/summary.json` (B serves it at `GET /api/eval/results`).
- Evaluation report, charts, failure analysis, demo script, slide text.

I depend on: B's `/api/stt` (for evaluation I may call ElevenLabs Scribe directly from `eval/` code too, using the same `.env` key) and A's real signals JSON format (`shared/signals.example.json`).

---

## 4. REPO MAP
```
.
├── README.md
├── requirements.txt          one shared Python environment
├── .env.example  -> copy to .env (git-ignored)
├── docs/
│   ├── PROJECT_SPEC.md       the whole idea
│   ├── API_CONTRACT.md       how parts talk (law)
│   ├── TEAM_RULES.md         git + ownership
│   ├── PROGRESS_C.md         MY progress tracker (Claude updates)
│   └── CLAUDE_CONTEXT_PERSON_C.md   this file
├── shared/
│   ├── intents.json          allowed intent ids
│   ├── gesture_map.json      MediaPipe gesture -> meaning
│   └── signals.example.json  example input to /api/fuse
├── frontend/   (A)
├── backend/    (B)
├── fusion/     (C)   <- I create files here
└── eval/       (C)   <- and here
    ├── data/clean/       real recordings by teammates (git-ignored, private voices)
    ├── data/simulated/   SIMULATED_* degraded audio + manifest (committed)
    └── results/          csv, charts, summary.json (committed)
```
Python rule: run everything from the **repo root**. Folders are packages, e.g. `from fusion.fuse import fuse`.

---

## 5. TECH DECISIONS (already made, don't re-debate)
| Thing | Decision |
|---|---|
| Language | Python 3.10+ for `fusion/` and `eval/` |
| API framework | FastAPI (`APIRouter` in `fusion/router.py`) |
| Data validation | Pydantic models |
| LLM | Provider switch via `.env` `LLM_PROVIDER` = `anthropic` (models `LLM_MODEL_FAST=claude-haiku-5-5`, `LLM_MODEL_SMART=claude-sonnet-5-5`) or `gemini` (`GEMINI_MODEL`). One function `call_llm_json(system, user, model_kind)` hides the difference. Use `fast` for the live app, `smart` allowed for offline experiments. |
| STT (eval) | ElevenLabs `scribe_v2` via `/v1/speech-to-text` (model id from `ELEVENLABS_STT_MODEL`) |
| TTS | B owns it. I only measure it if asked. |
| WER | `jiwer` library |
| Audio | `librosa`, `soundfile`, `pydub` (needs ffmpeg installed) |
| Data/plots | `pandas`, `matplotlib` |
| Tests | `pytest` (small, fast) |
| Cache | JSON files in `eval/.cache/` keyed by a hash of the request |

---

## 6. THE SHARED DATA SHAPES (summary; full detail in the contract)
**Input to `/api/fuse`** = `{raw_transcript, stt_confidence, language_hint, signals{gestures[], face{}, lips{}}, history[], profile{}, mode, simulated}` (see `shared/signals.example.json`).
**Output** = `{candidates[3-4]{text,intent,confidence,language,why}, corrected_transcript, latency_ms, model, fallback_used}`.
**Intents** come ONLY from `shared/intents.json`.
**Gesture meanings:** yes, no, stop_wait, pain_help, point (with direction), two_more, thanks_love, none.
**Modes:** `voice_only` (transcript only), `voice_context` (+history/profile), `full` (+gestures/face/lips). Used by evaluation to show the benefit of fusion.

---

## 7. HOW FUSION SHOULD WORK (design rules)
### What the clues mean
- **Gestures:** yes/no are strong and reliable signals. `pain_help` is urgent. `point` + direction means "that thing over there" (an object request). `stop_wait` means wait/stop.
- **Face:** `browInnerUp` high = question/worry/urgency. `frown` high = discomfort/pain/refusal. `smile` high = agreement/thanks/greeting. `jawOpen` high with no sound = trying to speak.
- **Lips:** `syllable_estimate` and `speech_duration_ms` hint at the length of the intended phrase (e.g. "water" ≈ 2 syllables, "I want some water" ≈ 5). `activity: none` + no transcript means the person isn't speaking.
- **Honesty:** landmarks do NOT truly lip-read words. They are *cues* that narrow the choices. In docs/slides call it "lip-movement cues".
- **Conflicts:** trust gestures for yes/no; trust the transcript for content words; use lips for length; use history to decide what the sentence is *answering*.

### The prompt must make the LLM do this
1. Return **3 to 4 candidates**, sorted by confidence, confidences roughly sum to ≤ 1.
2. Each candidate: short first-person sentence (≤ 12 words) the patient could say, an `intent` from the allowed list, a `language` (`en`/`hi`/`hinglish`), and a one-line `why` (≤ 15 words).
3. **Reply language = the language/script the patient used** (Hindi in Devanagari if the transcript is Devanagari; Roman Hinglish if Roman). If `language_hint` is `auto` and unclear, use `profile.preferred_language`.
4. If the other person just asked something (last history item is from `other`), at least 2 candidates should answer it.
5. If evidence is weak/conflicting, put `unclear_retry` ("Let me try again.") as the last candidate.
6. If `pain_help` gesture or words like help/दर्द/बचाओ appear, the top candidate is `help_emergency` or `pain`.
7. Never invent medical facts, names, or numbers that aren't in the profile.
8. Output **only valid JSON**, no extra text.
9. `corrected_transcript` = the LLM's best guess of the exact words the patient tried to say.
10. `mode` controls what the LLM is allowed to see: build the user message accordingly (for `voice_only` do not include signals, history, or profile).

### Starting prompt (Claude: improve it, keep it short, under ~600 tokens)
```
You help a person with a speech impairment communicate in a live conversation.
You get: an unclear speech-to-text transcript (it may be wrong), camera cues (gestures,
face, lip movement), the recent conversation, and a short profile.
Decide what the person most likely wanted to say.

Rules:
- Return 3-4 candidates, best first. Each is a short first-person sentence (max 12 words).
- "intent" must be exactly one id from ALLOWED_INTENTS.
- Reply in the language and script the person used (English, Hindi, or Hinglish).
- If the other person just asked a question, at least 2 candidates must answer it.
- Trust gestures for yes/no. Trust the transcript for content words. Use lip syllable count
  and speech duration as hints about phrase length.
- Pain or help gesture/words -> first candidate is pain or help_emergency.
- If unsure, last candidate is intent "unclear_retry".
- Never invent facts not in the profile.
- Output ONLY JSON: {"candidates":[{"text","intent","confidence","language","why"}],
  "corrected_transcript":"..."}
```

### Safety net (fallback, no LLM)
If the LLM call fails, times out (use ~4 s timeout), or returns bad JSON (retry once with a stricter reminder), return `fallback_used: true` with rule-based candidates:
- gesture yes -> "Yes." ; no -> "No." ; pain_help -> "I am in pain. Please help." ; stop_wait -> "Please wait."
- plus the raw transcript as a candidate if non-empty
- plus "Please say that again." and "Let me try again."
Never return an empty list. Never crash the endpoint.

### Speed rules (target: fuse < 700 ms)
Use the fast model, `max_tokens` ≈ 350, temperature 0-0.2, keep only the last 6 history turns, send compact JSON, no long examples in the live prompt (put a max of 2 tiny examples).

---

## 8. TEST DATASET RULES (`eval/data/`)
- **60 sentences:** 20 English (`en_001...`), 20 Hindi (`hi_001...`), 20 Hinglish (`hg_001...`).
- Realistic patient needs: water, food, medicine, pain, toilet, emergency, call someone, tired, hot/cold, objects, TV/phone, yes/no, thanks, greeting, repeat, wait. Mix short (1-2 words) and medium (4-8 words) sentences. Some are *answers* to a question the other person asked (put that question in `context_other_said`).
- CSV columns (`eval/data/test_sentences.csv`):
  `id, language, reference_text, true_intent, expected_gesture_meaning, gesture_direction, context_other_said, syllables, notes`
- `true_intent` must be an id from `shared/intents.json`.
- **Script rule for scoring:** write `reference_text` in the script we expect Scribe to return (Hindi sentences in Devanagari; Hinglish in Roman letters). Normalise before WER: Unicode NFC, lowercase, remove punctuation, collapse spaces. Document any mismatch problems (e.g. Scribe returns Devanagari for a Hinglish clip) in the report instead of hiding them.
- **Recording:** teammates read the sentences clearly (clean audio) into a phone or laptop, one file per sentence: `eval/data/clean/<id>__<speaker>.wav`. Only teammates who agree. These files are git-ignored.
- **Simulation** (`eval/simulate.py`): from each clean clip make **mild / moderate / severe** versions:
  - time-stretch slower (1.2x / 1.5x / 1.9x) to mimic slowed dysarthric speech
  - muffle (low-pass filter) + breath/noise
  - slur: randomly shorten/blur segments (drop short high-energy consonant-like bursts)
  - stutter: repeat random 100-300 ms pieces
  - long pauses inserted between words (energy-based split)
  Output name: `SIMULATED_<id>_<severity>.wav` into `eval/data/simulated/`, plus `eval/data/simulated/manifest.csv` with columns `file, id, severity, language, reference_text, true_intent, simulated=True, method_notes`.
- To save credits, evaluate **at most ~120 clips** (e.g. moderate + severe for all 60), and **cache** every Scribe result. Mild is optional.
- **Simulated signals** (`eval/simulate_signals.py`): for each clip create a signals JSON consistent with `true_intent` but **imperfect**: right gesture ~75%, missing ~15%, wrong ~10%; face cues loosely matching the intent; `syllable_estimate` = true syllables ± 0-2 (more noise for severe); `lips.activity` high. Everything has `"simulated": true`. This is a *model of* what MediaPipe would give, not real detections, and the report must say so. If time allows, teammates can also record ~10 real webcam clips to validate.

---

## 9. EVALUATION RULES (`eval/evaluate.py`)
Run every clip through the pipeline in these ways:
| Name | What the LLM sees |
|---|---|
| `raw_scribe` | No LLM. Just Scribe's transcript. (baseline for WER) |
| `voice_only` | Transcript only |
| `voice_context` | Transcript + history + profile |
| `full_fusion` | Transcript + history + profile + gestures + face + lips |

Metrics:
- **WER** = (substitutions + deletions + insertions) / number of reference words, using `jiwer`. Computed for `raw_scribe` and for each mode's `corrected_transcript`. Lower is better. Report overall + by language + by severity.
- **Intent top-1** = % of clips where candidate #1's `intent` equals `true_intent`. **Top-3** = % where `true_intent` is among the first 3.
- **Latency (ms):** stt time, fuse time, total (speech end -> candidates ready). Report mean, median, p95. Say how it was measured. Target < 2000 ms. (Add TTS first-byte time only if B's endpoint is available.)
- **Effort:** typing baseline = number of characters of `reference_text` the person would type. Our system: 1 action if true intent is top-1; 2 if in top-3 (look at the list and pick); else fall back to typing (= characters). Report average actions and, if A logged real data via `/api/metrics/effort`, add the real average seconds-to-confirm. Be clear which numbers are modelled vs measured.
- **Where Scribe fails:** table of WER by severity and language; list 10 worst examples with `reference | scribe output`; count common failure types (dropped words, wrong language, Devanagari/Roman mix-up, hallucinated words, empty output).
- Outputs in `eval/results/`: `per_clip.csv`, `summary.json` (shape in the contract), `failures.md`, charts `wer_by_mode.png`, `intent_accuracy.png`, `latency.png`, `wer_by_severity.png`, and `report.md` (plain-language results).
- Everything must be **re-runnable with one command**: `python -m eval.evaluate` (with options `--limit N`, `--modes ...`, `--no-llm-cache`).

---

## 10. CONSENT AND ETHICS (for any text I write)
- Cloned voices only with recorded + typed + ticked consent; backend returns 403 without it (B builds this).
- Only teammates who agreed.
- Always say: data is **SIMULATED**, results are indicative, not clinical; this is an assistive prototype, not a medical device; no video leaves the browser (only small JSON signals).

---

## 11. COMMON PROBLEMS (Claude: use this to help me)
| Problem | Likely cause / fix |
|---|---|
| `ModuleNotFoundError: fusion` | Run from the repo root and use `python -m ...` |
| `ffmpeg not found` | Install ffmpeg (https://ffmpeg.org/download.html) and reopen the terminal |
| 401/403 from an API | Key missing or wrong in `.env` |
| 429 from an API | Rate/credit limit: wait, use cache, reduce clips |
| LLM returns text around JSON | Strip code fences, parse the first `{...}`, retry once |
| Hindi shows as `???` in terminal | Terminal encoding; files are fine. Use UTF-8 when writing CSV (`encoding="utf-8"`) |
| Scribe returns Devanagari for Hinglish | Document it; normalise; don't hide |
| Merge conflict | Probably someone edited a file outside their folder. Tell me before resolving |

### Tiny glossary (explain these again whenever I ask)
**API** = a way for programs to ask each other for things. **Endpoint** = one address of an API (like `/api/fuse`). **JSON** = text format for data. **LLM** = AI that reads and writes text. **STT** = speech to text. **TTS** = text to speech. **Intent** = what the person actually wants. **Top-1/Top-3** = correct answer is first / in first three. **WER** = % of words wrong. **Latency** = waiting time. **p95** = the time 95% of requests are faster than. **Branch** = my own copy of the code to work safely. **Commit** = saving a snapshot. **Cache** = remembered results so we don't pay twice. **Fallback** = backup plan when something fails.

---

## 12. PROGRESS REPORT (print this after every step, and when I type `status`)
Update `docs/PROGRESS_C.md` first (status icons, the % at the top, the log, the numbers table). Then print exactly this layout in simple words:

```
📍 WHERE WE ARE
Overall: 41%  (7 of 17 steps done)        ← finished steps / 17, rounded
Time check: about hour X of 24 (I'm guessing from what's done; tell me the real hour if different)

✅ DONE
 - short plain line per finished step

🔄 DOING NOW
 - the current step, and how far along it is

⏭️ NEXT 3 STEPS
 1. ...
 2. ...
 3. ...

🧑 WHAT *YOU* MUST DO (the human)
 - exact small actions only you can do (e.g., "record these 20 sentences", "paste your key in .env")

⚠️ WATCH OUT
 - risks, anything behind schedule, anything blocking teammates

📊 NUMBERS SO FAR
 - WER raw / fusion, intent top-1/top-3, latency (or "not measured yet")

🚦 HEALTH: 🟢 on track / 🟡 slightly behind / 🔴 behind - and one sentence why
```
Compare against this schedule to decide the health light:
| Hour | Where C should be |
|---|---|
| 1 | Steps 0-1 done |
| 4 | Step 3 done (fuse works from the command line) |
| 7 | Steps 4-6 done (router ready for B) |
| 10 | **First full integration with A and B**; step 7 done |
| 12 | Steps 8-9 done (simulated audio exists) |
| 14 | Steps 10-11 done (evaluate.py works on a small set) |
| 16 | Step 12 done (full results exist) |
| 18 | **Feature freeze**; step 13 done |
| 20 | Step 14 done |
| 23 | Step 16 done, demo rehearsed |

---

## 13. THE STEP-BY-STEP PLAN (17 steps)
When I say `do step N`, do **everything** in that step, test it, update progress, print the report. Each step lists: **Goal / Claude does / Human does / Check / Done when**.

### STEP 0 - Check my computer, keys, and tools  (hour 0-1)
- **Goal:** everything needed is installed and the keys work.
- **Claude does:**
  1. Check Python version (need 3.10+), pip, git, ffmpeg. Tell me how to install what's missing (simple, for my operating system).
  2. Create the virtual environment `.venv` and run `pip install -r requirements.txt`. Fix any install errors.
  3. Check `.env` exists (copy from `.env.example` if not). Check which keys are filled in **without printing them** (just "present/missing").
  4. Ask me ONE question: "Which LLM do you have a key for: Anthropic (Claude) or Gemini?" and set `LLM_PROVIDER` accordingly (tell me to edit `.env`; don't write keys yourself).
  5. Create `fusion/check_setup.py` that: loads `.env`; makes ONE tiny call to the LLM ("reply with the word OK"); makes ONE tiny check that the ElevenLabs key is valid (e.g. list voices or a models/user-info call, whichever is cheapest per the official docs); prints ✅/❌ for each thing.
- **Human does:** install ffmpeg if missing; paste keys into `.env`; run `python -m fusion.check_setup` when Claude says so.
- **Check:** `python -m fusion.check_setup` shows all ✅.
- **Done when:** LLM ✅, ElevenLabs ✅, ffmpeg ✅, Python ✅.

### STEP 1 - Understand everything + create my skeleton  (hour 1)
- **Claude does:** read all docs and `shared/*`; write me a **10-line summary in simple words** of what the project is and what I (C) must build; create these empty-but-documented files (each with a "what this file does" comment at the top): `fusion/schemas.py`, `fusion/llm_client.py`, `fusion/prompts.py`, `fusion/fuse.py`, `fusion/replies.py`, `fusion/fallback.py`, `fusion/router.py`, `fusion/try_fuse.py`, `fusion/tests/__init__.py`, `eval/make_dataset_notes.md`, `eval/simulate.py`, `eval/simulate_signals.py`, `eval/evaluate.py`, `eval/metrics.py`, `eval/stt_client.py`, `eval/cache.py`. Print the Progress Report.
- **Human does:** read the summary; tell Claude if anything sounds wrong.
- **Done when:** files exist, summary approved.

### STEP 2 - LLM client with provider switch  (hour 1-2)
- **Claude does:** write `fusion/llm_client.py` with `call_llm_json(system: str, user: str, model_kind="fast") -> dict` that: reads `.env`; supports `anthropic` and `gemini`; uses a short timeout (~4 s for live use; allow a longer one for offline experiments via a parameter); asks for JSON-only output; strips code fences; parses the first JSON object; retries once if parsing fails; returns the parsed dict **plus** timing (e.g. returns `(data, latency_ms, model_name)`); never prints keys. Write `fusion/tests/test_llm_client.py` for the JSON-cleaning part (no network needed).
- **Check:** `pytest fusion/tests -q` passes; a one-line manual call returns JSON.
- **Done when:** a real call works and tests pass.

### STEP 3 - Fusion prompt v1 and `fuse.py`  (hour 2-4)
- **Claude does:**
  1. `fusion/schemas.py`: Pydantic models for the request and response exactly as in the contract (`FuseRequest`, `Signals`, `Candidate`, `FuseResponse`). Allowed intents loaded from `shared/intents.json`; if the LLM returns an unknown intent, map it to `other`.
  2. `fusion/prompts.py`: the system prompt (start from section 7), and a function that builds the **user message** compactly from the request, respecting `mode` (voice_only / voice_context / full).
  3. `fusion/fallback.py`: rule-based fallback candidates (section 7).
  4. `fusion/fuse.py`: `fuse(request) -> FuseResponse`: build prompt, call LLM, validate (3-4 candidates, sorted, confidences 0-1, intents valid, never empty), on any failure use fallback with `fallback_used=True`. Measure `latency_ms`.
  5. `fusion/try_fuse.py`: a command-line tool: `python -m fusion.try_fuse --file shared/signals.example.json` prints the response nicely.
- **Human does:** run the try command and tell Claude if the candidates look sensible (e.g. "wa waa tar" + pointing should give water first).
- **Check:** works on `shared/signals.example.json`; deliberately breaking the key makes the fallback kick in instead of crashing.
- **Done when:** fuse works from the command line in all 3 modes and fallback works.

### STEP 4 - Reply suggestions  (hour 4-5)
- **Claude does:** add a replies prompt and `fusion/replies.py` with `suggest_replies(request) -> RepliesResponse` following the contract: **always 4 suggestions** (one yes-type, one no-type, one specific to the question, one clarifying), short, in the patient's preferred language/script, based on history and profile; fallback = generic Yes / No / "Please say that again." / "Please wait."
- **Check:** other says "Do you want to drink something?" -> sensible replies in English; the same in Hindi and Hinglish.
- **Done when:** works in 3 languages and fallback works.

### STEP 5 - Hand-test 10 cases  (hour 5-6)
- **Claude does:** create `fusion/tests/cases.json` with **10 realistic cases** (4 English, 3 Hindi, 3 Hinglish) including: gesture-only (thumbs up with a garbled word), empty transcript with only a gesture, conflicting clues, an emergency (fist + "dard"), an answer to a question. Run all, print a small table (input / top candidate / expected / ✅❌ / latency). Use this to improve the prompt (max 2 quick rounds). Write down what changed.
- **Human does:** judge if results feel right; tell Claude what looks silly.
- **Done when:** at least 8 of 10 look sensible and median latency is under ~900 ms (note the real number).

### STEP 6 - FastAPI router for the team  (hour 6-7)
- **Claude does:** write `fusion/router.py`: a FastAPI `APIRouter` with `POST /api/fuse` and `POST /api/replies` exactly as in the contract; errors use the standard error shape; the endpoint never crashes (fallback instead). Add `fusion/tests/test_router.py` with a fake LLM so it runs without network. Write `fusion/README_FOR_B.md` (5 lines): "mount it like this" + a `curl` example for each endpoint. Print the **exact message I should paste to Person B** (what file to import, what line to add).
- **Check:** a tiny test app (in the test only, not in `backend/`) mounts the router and the tests pass.
- **Human does:** commit, push my branch, send Claude's message to B (and A if useful).
- **Done when:** router + tests pass, B informed. **This is what B and A need for the hour-10 integration.**

### STEP 7 - Test sentences dataset  (hour 6-8; can run while waiting for B)
- **Claude does:** create `eval/data/test_sentences.csv` (60 rows, columns in section 8). Make the sentences natural for patients (not textbook). Add `eval/make_dataset_notes.md` explaining the dataset. Add `eval/check_dataset.py` that checks: 60 rows, ids unique, `true_intent` valid, languages balanced, no empty fields. Also create `eval/data/recording_sheet.md`: a printable sheet that says which person records which ids (split 20/20/20 among the 3 teammates, mixing languages).
- **Human does:** read the sentences; fix any that sound unnatural (Hindi/Hinglish especially).
- **Done when:** `python -m eval.check_dataset` prints ✅.

### STEP 8 - Record clean audio and validate  (hour 8-9)
- **Claude does:** write `eval/RECORDING_GUIDE.md` (simple: quiet room, phone voice recorder, hold phone 20 cm from mouth, 1-2 s silence before and after, read normally and clearly, one file per sentence, name `<id>__<yourname>.wav`, save into `eval/data/clean/`; how to convert m4a/mp3 to wav with ffmpeg). Write `eval/check_recordings.py`: for every row in the CSV, check the clean file exists, length 0.5-12 s, not silent, sample rate; print which are missing.
- **Human does:** ask each teammate (who agreed) to record their 20 sentences (~10 minutes each). Put all files in `eval/data/clean/`. (Share via drive/chat, not git.)
- **Done when:** `python -m eval.check_recordings` shows ✅ for all (or at least 45 of 60; tell me if fewer).
- **Shortcut if recordings are late:** use fewer sentences (e.g. 8 per language = 24) but say so honestly.

### STEP 9 - `simulate.py` (atypical speech simulation)  (hour 9-11)
- **Claude does:** implement section 8's simulation with 3 severities using `librosa`/`soundfile`/`numpy`; seeded randomness (reproducible); output `SIMULATED_<id>_<severity>.wav` + `manifest.csv`; keep files 16 kHz mono wav; print a summary. Also write `eval/data/simulated/README.md` stating clearly: "All files here are SIMULATED atypical speech made from healthy-speaker recordings. They are not real patient data." Make 2-3 listening samples I can play. Be honest in the README about limits (simulation ≠ real dysarthria).
- **Human does:** listen to 3-4 files per severity; tell Claude if "severe" is unrealistic/too harsh or too easy.
- **Done when:** manifest has ~180 rows (60 × 3) and I approved how it sounds.

### STEP 10 - Simulated signals generator  (hour 11-12)
- **Claude does:** write `eval/simulate_signals.py` (rules in section 8) producing `eval/data/simulated/signals.jsonl` (one JSON per clip using the exact `signals` shape from the contract, with `"simulated": true`) and make it reproducible with a seed. Print the % of clips with correct / missing / wrong gesture.
- **Done when:** file exists, shapes validate against `fusion/schemas.py`.

### STEP 11 - `evaluate.py` + small test  (hour 12-14)
- **Claude does:**
  1. `eval/cache.py` (disk cache keyed by hash) and `eval/stt_client.py` (ElevenLabs Scribe call for a wav file with `language_hint` auto; returns transcript + latency; uses the cache).
  2. `eval/metrics.py`: normalisation, WER (jiwer), intent top-1/top-3, latency stats (mean/median/p95), effort model.
  3. `eval/evaluate.py` with options `--limit`, `--severity`, `--modes`, `--no-cache`; runs STT once per clip (cached), then each LLM mode per clip (LLM results cached too), saves `per_clip.csv`, builds `summary.json` (contract shape), prints a readable table.
  4. First tell me: "This small run will make about N Scribe calls and M LLM calls. Go?" Then run `--limit 5`.
- **Human does:** say "go"; look at the table.
- **Done when:** a 5-clip run finishes, results make sense, re-running costs nothing (cache works).

### STEP 12 - Full evaluation run #1 + failure analysis  (hour 14-16)
- **Claude does:** announce API-call counts and ask "go?"; run the full evaluation (≤ ~120 clips); save results; write `eval/results/failures.md` (10 worst examples, failure types, by language and severity, plain-language "where Scribe fails" paragraph); make the 4 charts. Print the numbers table and update `PROGRESS_C.md`. Give **3 concrete ideas** to improve fusion.
- **Done when:** `summary.json`, `per_clip.csv`, `failures.md`, charts exist.

### STEP 13 - Improve prompts, rerun, before/after  (hour 16-18)
- **Claude does:** up to 3 improvement rounds on the fusion prompt (e.g. better use of syllables, better Hinglish handling, better gesture rules). After each, re-run only the LLM modes (STT is cached, so it's free) and log: round number, what changed, top-1/top-3/WER before and after. Keep the best version. **Do not overfit by copying test sentences into the prompt.** Save `eval/results/prompt_history.md`. Tell me honestly if a change made things worse and revert it.
- **Done when:** best prompt is committed; `prompt_history.md` exists; final numbers recorded. **Feature freeze at hour 18: no new features after this, only fixes.**

### STEP 14 - Results report, charts, `summary.json`  (hour 18-20)
- **Claude does:** produce the final `eval/results/summary.json` (contract shape, `data_note: "SIMULATED atypical speech"`), `report.md` in plain language (what we tested, how, the table, 3 charts, honest limits, what we'd do with real data like TORGO/UA-Speech), and a **one-slide-friendly results box** (5 lines max). Tell me which of the 3 key claims we can make honestly, e.g. "full fusion improved top-1 intent from X% to Y%".
- **Human does:** tell B that `summary.json` is ready; check A shows the numbers.
- **Done when:** `GET /api/eval/results` (via B) returns it, or Claude has given me the exact message for B.

### STEP 15 - Integration help with A and B  (first at hour ~10, again hour 18-20)
- **Claude does:** help me test the **real** end-to-end flow: A's signals -> B's `/api/stt` -> my `/api/fuse` -> B's `/api/speak`. Write `fusion/tests/smoke_test.py`: sends a sample request to a running server and prints ✅/❌ for each endpoint plus timings. When something fails, work out whose side it is and write the exact message for the right teammate. Check real `signals` from A match the schema (loosen my validation if A sends harmless extra fields). Measure real end-to-end latency with the real stack and add it to the report.
- **Done when:** one full real turn works (speak -> candidates -> confirm -> audio) and the latency number is recorded.

### STEP 16 - Final docs, demo script, slides text, backup plan  (hour 20-23)
- **Claude does:** write `docs/DEMO_SCRIPT.md` (3-4 minute script with exact lines for each teammate, 2 scenes: patient says "pani" + points; other person asks a question in Hinglish), `docs/SLIDES_TEXT.md` (slide-by-slide text: problem, idea, architecture, demo, evaluation results, consent and ethics, limitations, next steps with real datasets/native mobile/call integration), `docs/QA_PREP.md` (15 likely judge questions with honest short answers, e.g. "Is the data real?", "Why not true lip reading?", "How do you handle consent?", "Latency?"), a **backup plan** (screen-recorded demo video, pre-recorded audio clips, phone hotspot, cached TTS phrases), and the final checklist from the problem statement. Fill in the README sections about evaluation. Final `status` = 100%.
- **Done when:** I can rehearse the demo using these files, and the checklist in section 2 is all ticked or honestly noted.

---

## 14. GIT RULES FOR CLAUDE
- I work on branch **`person-c/fusion-eval`**. Check with `git branch --show-current` before committing. If it's not that branch, stop and tell me.
- After each finished step run `git add fusion eval docs/PROGRESS_C.md` (only my files), then `git commit -m "<type>(<scope>): <what>"` with types `feat`, `fix`, `data`, `eval`, `docs`.
- **Ask me before `git push`.** Never `push --force`. Never commit `.env`, `eval/data/clean/*`, or `eval/.cache/`.
- Every ~2 hours remind me to run `git pull origin main` and `git merge main` so I stay up to date, and to tell me if teammates changed `shared/` or the contract.
- At the merge points in `docs/TEAM_RULES.md` (hour ~10, ~16, ~21) remind me what to do, in simple steps.

---

## 15. FIRST MESSAGE TO CLAUDE (copy-paste)
> Read `docs/CLAUDE_CONTEXT_PERSON_C.md` fully. Then read `docs/PROJECT_SPEC.md`, `docs/API_CONTRACT.md`, `docs/TEAM_RULES.md` and everything in `shared/`. I am Person C and a beginner. Explain in 10 simple lines what you understood, show the Progress Report, and then start **Step 0**.

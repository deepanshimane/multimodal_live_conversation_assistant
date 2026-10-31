# API CONTRACT v1

**Status: FROZEN at hour 1.** To change anything: tell the whole team, edit this file, bump the version at the top, and everyone pulls.
Base URL (local): `http://localhost:8000`. All JSON uses `snake_case` except MediaPipe blendshape names (kept as MediaPipe gives them, e.g. `browInnerUp`).
Times are in **milliseconds (ms)**. Confidence numbers are **0.0 to 1.0**.

## Who builds what
| Endpoint | Owner | Notes |
|---|---|---|
| `GET /api/health` | B | |
| `POST /api/stt` | B | ElevenLabs Scribe |
| `POST /api/speak` | B | ElevenLabs TTS |
| `POST /api/consent`, `POST /api/voice/clone`, `POST /api/voice/design`, `DELETE /api/voice/{voice_id}` | B | consent enforced here |
| `POST /api/fuse`, `POST /api/replies` | **C** | C writes `fusion/router.py` (a FastAPI `APIRouter`). **B mounts it** in `backend/main.py` with `app.include_router(fusion_router)` |
| `POST /api/metrics/effort` | B | stores what A sends |
| `GET /api/eval/results` | B | just returns the file `eval/results/summary.json` made by C |

Python import rule: run the server from the **repo root** with `uvicorn backend.main:app`. Folders `backend`, `fusion`, `eval` are Python packages (they contain `__init__.py`). B imports C's router like this:
```python
from fusion.router import router as fusion_router
app.include_router(fusion_router)
```

## Standard error shape (every endpoint)
```json
{"error": {"code": "consent_required", "message": "Human readable text"}}
```
Common codes: `bad_request` (400), `consent_required` (403), `not_found` (404), `upstream_error` (502, ElevenLabs/LLM failed), `timeout` (504).

---

## GET /api/health
Response: `{"status":"ok","version":"1"}`

## POST /api/stt  (B)
Request: `multipart/form-data`
- `audio` (file: webm/ogg/wav/mp3, max ~30 s)
- `language_hint` (text: `auto` | `en` | `hi`, default `auto`)
- `speaker` (text: `patient` | `other`, default `patient`)

Response:
```json
{
  "transcript": "I want water",
  "language": "en",
  "confidence": 0.82,
  "words": [{"text":"I","start":0.1,"end":0.3}],
  "latency_ms": 640
}
```
`words` may be empty. `confidence` may be `null` if ElevenLabs does not give one.

## POST /api/fuse  (C)  - the brain
Request (JSON): see `shared/signals.example.json`.
```json
{
  "raw_transcript": "wa waa tar",
  "stt_confidence": 0.41,
  "language_hint": "auto",
  "signals": {
    "window_ms": 3000,
    "gestures": [{"name":"Pointing_Up","meaning":"point","direction":"left","duration_ms":1200,"confidence":0.9}],
    "face": {"browInnerUp":0.6,"smile":0.1,"jawOpen":0.4,"frown":0.0,"blink_count":1},
    "lips": {"activity":"high","syllable_estimate":2,"speech_duration_ms":1500,"mouth_open_max":0.5}
  },
  "history": [{"speaker":"other","text":"Do you need anything?"}],
  "profile": {"name":"Ravi","needs":["water"],"preferred_language":"hinglish"},
  "mode": "full",
  "simulated": false
}
```
- `mode` (optional, default `"full"`): `"voice_only"` = LLM sees only the transcript; `"voice_context"` = transcript + history/profile; `"full"` = everything. **Only the evaluation uses the other two modes.** The frontend always sends `full` (or omits it).
- `signals` may be empty/missing pieces: the system must still work.
- `history`: last 6 turns max. `speaker` is `patient` or `other`.

Response:
```json
{
  "candidates": [
    {"text":"I want some water.","intent":"request_water","confidence":0.72,"language":"en","why":"Pointing + 2 syllables + 'wa..tar' sounds like water"},
    {"text":"Can you give me the cup?","intent":"request_object","confidence":0.15,"language":"en","why":"Pointing at an object"},
    {"text":"Let me try again.","intent":"unclear_retry","confidence":0.05,"language":"en","why":"Fallback"}
  ],
  "corrected_transcript": "water",
  "latency_ms": 710,
  "model": "claude-haiku-5-5",
  "fallback_used": false
}
```
- 3 to 4 candidates, sorted by `confidence` (highest first).
- `intent` must be one of the ids in `shared/intents.json`.
- `language` is `en` | `hi` | `hinglish`. Hindi text may be in Devanagari or Roman letters; keep the script the patient used.
- Never returns an empty list. If the LLM fails, `fallback_used` is `true` and simple rule-based candidates are returned (HTTP 200, not an error).

## POST /api/replies  (C)
Request:
```json
{
  "other_text": "Do you want to drink something?",
  "history": [{"speaker":"patient","text":"I want water"}],
  "profile": {"name":"Ravi","needs":["water"],"preferred_language":"hinglish"},
  "language_hint": "auto"
}
```
Response:
```json
{
  "suggestions": [
    {"text":"Yes, water please.","intent":"yes","language":"en"},
    {"text":"No, thank you.","intent":"no","language":"en"},
    {"text":"Tea, please.","intent":"request_food","language":"en"},
    {"text":"Please say that again.","intent":"repeat_please","language":"en"}
  ],
  "latency_ms": 520,
  "model": "claude-haiku-5-5",
  "fallback_used": false
}
```
Always 4 suggestions: include one yes-type, one no-type, one specific, one clarifying.

## POST /api/speak  (B)
Request (JSON):
```json
{"text":"I want some water.","voice_id":null,"model":"flash","language":"en"}
```
- `model`: `flash` (= `eleven_flash_v2_5`, default, fast) | `v3` (= `eleven_v3`, more expressive, slower)
- `voice_id`: `null` means use the default voice.
Response: audio stream (`audio/mpeg`). Header `X-First-Byte-Ms` = time until the first audio byte.

## POST /api/consent  (B)
Request: `multipart/form-data`: `name` (text), `agreed` (`true`/`false`), `consent_audio` (file of the person reading the consent sentence aloud).
Response: `{"consent_id":"c_123","name":"Ravi","created_at":"2026-11-01T10:00:00Z"}`
Fails with `bad_request` if `agreed` is not `true`, the name is empty, or there is no audio.

## POST /api/voice/clone  (B)
Request: `multipart/form-data`: `consent_id`, `voice_name`, `samples` (1+ audio files).
Response: `{"voice_id":"abc...","voice_name":"Ravi voice"}`
**Returns 403 `consent_required` if `consent_id` is missing or unknown.**

## POST /api/voice/design  (B)
Request: `{"description":"warm, calm middle-aged Indian man","voice_name":"Designed voice 1"}`
Response: `{"voice_id":"abc...","voice_name":"Designed voice 1","cloned":false}`
(No consent needed because no real person's voice is copied. The UI must say "designed voice, not a clone".)

## DELETE /api/voice/{voice_id}  (B)
Deletes the voice at ElevenLabs and removes the stored consent link. Response: `{"deleted":true}`

## POST /api/metrics/effort  (B)
Request (A sends after every confirmed message):
```json
{"turn_id":"t_17","method":"tap","actions":1,"seconds_to_confirm":2.3,"candidate_rank_chosen":1,"total_latency_ms":1650}
```
`method`: `tap` | `gesture` | `dwell` | `blink` | `typed`. Response: `{"ok":true}`

## GET /api/eval/results  (B serves, C produces)
Returns the content of `eval/results/summary.json`. Shape (C may add fields, never remove):
```json
{
  "generated_at": "2026-11-01T14:00:00Z",
  "data_note": "SIMULATED atypical speech",
  "wer": {"raw_scribe": 0.0, "voice_only": 0.0, "voice_context": 0.0, "full_fusion": 0.0},
  "intent_accuracy": {
    "voice_only": {"top1": 0.0, "top3": 0.0},
    "voice_context": {"top1": 0.0, "top3": 0.0},
    "full_fusion": {"top1": 0.0, "top3": 0.0}
  },
  "latency_ms": {"stt_mean": 0, "fuse_mean": 0, "fuse_p95": 0, "total_mean": 0, "total_p95": 0},
  "effort": {"typing_chars_avg": 0, "our_actions_avg": 0},
  "by_language": {},
  "by_severity": {}
}
```

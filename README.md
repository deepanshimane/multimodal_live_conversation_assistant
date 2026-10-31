# Voice Bridge (working name) - Multimodal Live Conversation Assistant

Hackathon project for **Problem Statement 16: Multimodal Live Conversation Assistant for People with Speech Impairments** (AI for Biomedical Engineering and Assistive Technology).

## What it does (simple words)
A person whose speech is hard to understand talks to the app. The app:
1. **Hears** them (ElevenLabs Scribe speech-to-text).
2. **Sees** them (MediaPipe: hands, face, lips, running in the browser).
3. **Thinks** (an LLM combines what it heard + what it saw + the conversation so far and guesses what they meant).
4. **Offers 3-4 options**; the person confirms one with a tap, gesture, or dwell.
5. **Speaks** it in a personalised, **consented** voice (ElevenLabs text-to-speech / voice cloning / voice design).
6. **Listens** to the other person and suggests replies.
Supports English, Hindi, and Hinglish (mixed).

## Folder map
| Folder | Owner | What is inside |
|---|---|---|
| `frontend/` | Person A | Web app: webcam, MediaPipe, UI |
| `backend/` | Person B | FastAPI server, ElevenLabs STT/TTS, voice cloning, consent |
| `fusion/` | Person C | LLM fusion + reply suggestion code and prompts |
| `eval/` | Person C | Simulated dataset, evaluation scripts, results |
| `shared/` | Everyone (changes need group OK) | Intent list, gesture map, example signals JSON |
| `docs/` | Everyone | Spec, API contract, team rules, progress files |

## Team
| Person | Name | Role |
|---|---|---|
| A | _(fill in)_ | Frontend + MediaPipe |
| B | _(fill in)_ | Backend + ElevenLabs + consent |
| C | _(fill in)_ | LLM fusion + data + evaluation + demo |

## Quick start
```bash
# 1. get the code
git clone <REPO_URL>
cd <REPO_FOLDER>

# 2. secrets
cp .env.example .env        # Windows: copy .env.example .env
# open .env and paste your keys

# 3. Python
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 4. run backend (from the repo root)
uvicorn backend.main:app --reload --port 8000

# 5. run frontend (new terminal)
cd frontend
npm install
npm run dev
```

## Read these first
- `docs/PROJECT_SPEC.md` - the whole idea and requirements
- `docs/API_CONTRACT.md` - how the 3 parts talk to each other (do not change without everyone agreeing)
- `docs/TEAM_RULES.md` - git rules and who owns which files

## Ethics, consent, and honesty
- Voice cloning only works after a recorded, typed, and ticked **consent** (enforced in the backend).
- We only use voices of team members who agreed.
- Test data is **SIMULATED** atypical speech (clearly labelled `SIMULATED_*`). Results are indicative, not clinical.
- This is an assistive prototype, not a medical device.

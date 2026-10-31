# PROJECT SPEC - Voice Bridge

## 1. Problem statement (Problem Statement 16)
**Multimodal Live Conversation Assistant for People with Speech Impairments**
Domain: AI for Biomedical Engineering and Assistive Technology.

People with speech impairments caused by ALS, cerebral palsy, Parkinson's disease, stroke, or laryngectomy struggle to take part in live conversations: their speech is hard for standard systems to recognise, and voice-only tools ignore the gestures, facial expressions and lip movements they naturally use to communicate.

Participants must build a **real-time system** that:
1. analyses the patient's **voice together with gestures, facial expressions and lip movements** from a camera,
2. **fuses** them to infer the intended message,
3. **listens to the other person** in the conversation,
4. helps the patient respond by **suggesting replies** and **speaking the confirmed message in a natural, personalised voice**.

Rules:
- **ElevenLabs for speech recognition and voice generation is MANDATORY.**
- Support **English, Hindi, and code-mixed** speech.
- **Enforce consent** for any voice cloning. Use only consented voices.

Challenge: design, train, and validate a robust prototype that demonstrates end-to-end functionality, scalability, and real-world applicability within the hackathon timeframe.

Hints given:
- ElevenLabs Speech-to-Text (Scribe) for both speakers; **measure where it fails on atypical speech**.
- MediaPipe for hand, face, lip landmarks; fuse with the transcript using an LLM that **ranks candidate intents using conversation context**.
- ElevenLabs voice cloning (or Voice Design when no recordings exist) with Flash or v3 models.
- Start with TORGO or UA-Speech, or **simulate** atypical speech and gestures and **label the data clearly as simulated**.
- Evaluate: **WER with and without gesture fusion, intent accuracy, end-to-end latency (target < 2 s), and effort needed from the user to confirm a reply.**

## 2. Our solution in simple words
Hear + See + Think + Confirm + Speak + Listen (see README). Web app (PWA) that works on laptop and phone. Camera analysis (MediaPipe) runs **in the browser**; only small JSON "signals" go to the server (no video is uploaded).

## 3. Data flow
```
Patient speaks -> mic -> POST /api/stt -> raw transcript
Camera -> MediaPipe (browser) -> signals JSON
raw transcript + signals + history + profile -> POST /api/fuse -> 3-4 ranked candidates
Patient confirms one (tap / thumbs-up / dwell / number)
Confirmed text -> POST /api/speak -> audio plays in the patient's (consented) voice
Other person speaks -> POST /api/stt (speaker=other) -> text -> POST /api/replies -> 4 reply suggestions
```

## 4. Tech stack
| Part | Choice |
|---|---|
| Frontend | React + Vite (mobile-first, PWA), MediaPipe Tasks Vision (JS): GestureRecognizer, HandLandmarker, FaceLandmarker (blendshapes) |
| Backend | Python FastAPI |
| STT | ElevenLabs Scribe v2 (`scribe_v2`). Upgrade: `scribe_v2_realtime` |
| TTS | `eleven_flash_v2_5` (fast default), `eleven_v3` (expressive) |
| Voice | Instant Voice Cloning (consented) or Voice Design |
| LLM | `claude-haiku-5-5` (fast) / `claude-sonnet-5-5`; Gemini as backup provider |
| Eval | Python: jiwer (WER), librosa/pydub (audio simulation), pandas, matplotlib |

## 5. Features
**Must-have:** webcam + landmarks; speech to Scribe transcript; gesture chips; fusion to 3-4 ranked candidates; one-action confirm; TTS speaks it; other-person listening + reply suggestions; history; EN/HI/Auto toggle; consent gate; evaluation page.
**Should-have:** latency badge; quick phrases (Yes/No/Pain/Water/Help/Wait); patient profile; emergency button; dwell-select.
**Nice-to-have:** blink/nod confirm; realtime Scribe; learning from corrections; Meet/Teams demo path (tab audio + virtual mic); PWA install.

## 6. Consent rules (non-negotiable)
1. Before any clone: consent screen, person ticks box, types full name, reads the consent sentence aloud (recorded).
2. Backend stores consent record and **returns 403 on clone without it**.
3. "Delete my voice" button works.
4. Only voices of team members who agreed. No celebrities, no one else.
5. No consent needed for Voice Design (no real voice copied) but the UI must label it "designed voice".

## 7. Evaluation (what we must report)
| Metric | Meaning |
|---|---|
| WER (raw Scribe) | Word error rate of Scribe's transcript vs the true sentence |
| WER with fusion | WER of the LLM's `corrected_transcript` |
| Intent accuracy | top-1 / top-3: is the true intent among the first 1 / 3 candidates |
| Latency | speech end -> candidates shown (mean, p95), target < 2000 ms; plus confirm -> first audio byte |
| User effort | actions and seconds to confirm vs typing the sentence |
| Where Scribe fails | WER broken down by language and severity + typical error types |

All test data is **SIMULATED** and must be labelled `SIMULATED` in file names, README, UI and slides.

## 8. Risks
Wi-Fi/API failure during demo (backup video + cached phrases); Hindi/Hinglish TTS sounds odd (test early); latency > 2 s (small prompts, fast model, streaming); integration surprises (first full integration at hour 10); credits running out (cache API results, check plan limits); scope creep (freeze features at hour 18).

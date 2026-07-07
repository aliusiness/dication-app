# Implementation Plan — Local Wispr Flow Clone (Whisper + Ollama)

Goal: recreate Wispr Flow's base functionality 100% locally. Hold a hotkey
anywhere, speak, release — clean text appears at the cursor. Speech-to-text
runs on a local Whisper engine; the AI cleanup/formatting stage runs on a
local model served by **Ollama**. No audio or text ever leaves the machine.

> Ollama cannot transcribe audio (see docs/RESEARCH.md §4), so the pipeline is:
> **mic → local Whisper (STT) → Ollama (cleanup LLM) → text injection**.

## Architecture

```
┌─────────────────────────────── tray / menu-bar app ───────────────────────────────┐
│                                                                                    │
│  Global hotkey listener ──▶ Recorder (mic, 16kHz mono) ──▶ VAD gate (silero)       │
│   (push-to-talk / toggle)                                        │                 │
│                                                                  ▼                 │
│                                    STT engine (faster-whisper / whisper.cpp)       │
│                                                                  │ raw transcript  │
│  Frontmost-app detector ──▶ tone profile ──┐                     ▼                 │
│  Personal dictionary ──────────────────────┼──▶ Ollama cleanup (temp 0, strict     │
│                                            │    "rewrite-only" prompt, keep_alive) │
│                                            │                     │ cleaned text    │
│                                            │                     ▼                 │
│                     History store (SQLite) ◀── Text injector (clipboard-paste,     │
│                                                 save/restore clipboard)            │
└────────────────────────────────────────────────────────────────────────────────────┘
```

**Recommended stack (default assumption — adjust once goals are confirmed):**
Python 3.11+, `faster-whisper`, `ollama` (python client), `sounddevice`,
`pynput`, `pyperclip`, `silero-vad`, `pystray` (tray) — with whisper.cpp and
an Electron shell as fallback options if we later want a richer UI.
Rationale: fastest path to a working MVP; every stage is swappable.

**Default models** (tune to hardware):
| Hardware | STT model | Ollama cleanup model |
|---|---|---|
| Apple Silicon (16GB+) | `large-v3-turbo` (whisper.cpp Metal or mlx-whisper) | `llama3.1:8b` or `qwen2.5:7b` |
| NVIDIA GPU | faster-whisper `large-v3-turbo` (CUDA) | `qwen2.5:7b` |
| CPU-only | faster-whisper `distil-small.en` or Parakeet | `llama3.2:3b` (or skip cleanup in "fast mode") |

## Phases

### Phase 0 — Environment (half a day)
- Install Ollama, pull the cleanup model, verify `ollama run` works
- Python project scaffold (`uv` or `poetry`), deps, mic sanity check
- Grant OS permissions (macOS: Microphone + Accessibility)
- **Exit criteria:** a script records 5s of mic audio to WAV; `ollama list` shows the model

### Phase 1 — Core dictation loop, no LLM (1–2 days)
- Global push-to-talk hotkey (hold = record, release = process)
- Record mic → in-memory buffer → faster-whisper transcription
- Inject raw transcript at cursor via clipboard-paste (save/restore user clipboard)
- Minimal recording indicator (even a sound cue is fine at this stage)
- **Exit criteria:** hold key, speak a sentence in any app, release → text appears in <2s

### Phase 2 — Ollama cleanup layer (1–2 days)
- Post-process transcript through Ollama: remove fillers, fix punctuation/
  capitalization/grammar, format spoken lists ("first, second…" → bullets)
- Strict rewrite-only system prompt, temperature 0, `keep_alive` to stay warm
- Guardrails: skip LLM for utterances under ~5 words; 4s timeout → fall back
  to raw transcript; never let cleanup block insertion
- Config toggle: raw / light cleanup / full cleanup
- **Exit criteria:** "um so basically I think we should uh meet on on tuesday"
  → "I think we should meet on Tuesday."

### Phase 3 — Context awareness (2–3 days)
- Frontmost-app detection → per-app tone profiles (casual for Slack/Discord,
  professional for mail clients, plain/terse for terminals & editors)
- Personal dictionary: user-editable word list injected into both Whisper
  (`initial_prompt`) and the cleanup prompt; "add word" affordance
- Hands-free toggle mode with silero-VAD auto-stop on silence
- Dictation history in SQLite + "copy last transcript" hotkey
- **Exit criteria:** same sentence dictated into Slack vs. email visibly differs in tone; custom jargon transcribes correctly

### Phase 4 — UX polish (3–5 days)
- Tray/menu-bar app: settings (hotkeys, models, tone profiles, dictionary), history browser
- Floating recording pill (small always-on-top window) with level meter
- Command Mode: hotkey #2 reads the current selection (via clipboard), speak an
  instruction ("make this formal", "bullet this"), Ollama rewrites, replace selection
- View-diff of AI edits; per-app cleanup disable list
- **Exit criteria:** daily-drivable without touching a terminal

### Phase 5 — Performance & stretch (ongoing)
- Streaming/chunked transcription while still recording (target ~1s end-to-end)
- Parakeet backend option for near-instant STT on English
- Multilingual support; snippets; auto-learning dictionary from user corrections
- Packaging: py2app / PyInstaller (or migrate shell to Electron/Tauri if a richer UI is wanted)

## Risks & mitigations
| Risk | Mitigation |
|---|---|
| LLM "cleanup" hallucinates or answers instead of rewriting | temp 0, rewrite-only prompt with examples, skip-LLM threshold, raw-transcript fallback, diff view |
| Whisper hallucinates on silence | VAD gate before transcription |
| Cleanup latency on weak hardware | 3B model, fast mode (no LLM), keep_alive warm |
| Synthetic paste blocked by some apps | char-typing fallback; per-app injector override |
| Clipboard clobbering | save/restore around paste |
| Wayland (Linux) blocks global hotkeys/synthetic input | document X11 first; ydotool/compositor shortcuts later |

## Open questions (answers will tune this plan)
1. Target OS and hardware (picks STT backend, models, injector)
2. Python MVP vs. Electron/native shell from the start
3. English-only or multilingual (Parakeet vs. Whisper choice)
4. How aggressive should cleanup be by default (literal vs. Wispr-style rewriting)
5. Which features from Phase 3–4 matter most vs. can be cut

# How Wispr Flow Works — Research Notes

*Research date: July 2026. Sources listed at the bottom.*

Wispr Flow (often misheard as "Whisperflow") is an AI dictation app: you hold a
hotkey anywhere on your computer, speak naturally, release, and clean polished
text appears at your cursor — in any app. This document breaks down how it
works and what it takes to recreate the core experience fully locally.

## 1. The user-facing loop

1. **Activate** — a global hotkey with two modes: *push-to-talk* (hold to
   record, release to finish) and *hands-free* (toggle on/off). A small
   floating "pill" indicator shows recording state; the app itself lives in
   the menu bar / system tray.
2. **Speak** — natural speech, including whispering. Filler words, false
   starts, and self-corrections are allowed; the pipeline cleans them up.
3. **Insert** — within ~1 second, the final text is inserted at the cursor of
   whatever app has focus (Gmail, Slack, VS Code, a random web form).

Reviewers consistently note the "magic" is not the transcription itself but
the cleanup: output arrives already punctuated, de-filled ("um", "uh",
"like" removed), capitalized, and formatted (saying "first… second… third…"
produces a list).

## 2. The processing pipeline (cloud-side)

Wispr Flow is **cloud-based and does not work offline**. Audio is captured
locally, sent to AWS (us-east-1), and processed through subprocessors
including Baseten, OpenAI, Anthropic, and Cerebras. The pipeline has two
distinct AI stages:

| Stage | Job | Cloud equivalent | Local equivalent |
|---|---|---|---|
| 1. Speech-to-text | Raw transcript from audio | Whisper-class ASR models | whisper.cpp / faster-whisper / Parakeet |
| 2. LLM post-processing ("AI edits") | Remove fillers, fix grammar, punctuate, format lists, match tone to the target app, apply personal dictionary | GPT/Claude-class LLMs | **Ollama** running a 3–8B instruct model |

This two-stage split is the single most important architectural fact for a
clone: **transcription and cleanup are separate models with separate jobs.**

## 3. Feature inventory (base functionality vs. extras)

**Core (what "base functionality" means):**
- Global push-to-talk + hands-free toggle hotkeys (configurable, incl. mouse buttons)
- System-wide text insertion at the cursor, works in every app
- Auto-edits: filler removal, punctuation, capitalization, sentence repair
- List/structure formatting from spoken cues
- Floating recording indicator + tray/menu-bar app
- Dictation history

**Context features:**
- **Tone matching per app** — detects the frontmost app; Slack output is casual,
  email is professional, code comments are terse
- **Personal dictionary** — names/jargon learned from your corrections, applied
  in future transcripts
- **Command Mode** — select existing text, hold a second hotkey, speak an
  instruction ("make this more formal", "turn into bullets"); the selection is
  replaced with the rewrite

**Extras (later phases for a clone):**
- 100+ languages, whisper-voice recognition, snippets, scratchpad, "view diff"
  of AI edits, mobile keyboard (iOS/Android)

**Claimed performance:** ~220 effective WPM (vs ~45 typing), ~97% accuracy,
roughly 1s from release-of-key to inserted text. Pricing $15/mo. Requires
constant internet — this is the gap a local clone fills.

## 4. Key finding: Ollama cannot do speech-to-text

Ollama officially supports **LLMs only**. It cannot accept audio input:

- [ollama/ollama#5451](https://github.com/ollama/ollama/issues/5451) — open
  feature request for speech-to-text support
- [ollama/ollama#11798](https://github.com/ollama/ollama/issues/11798) — audio-capable
  models (e.g. Qwen2-Audio) load but cannot receive audio input
- The "whisper" models listed on ollama.com are community GGUF uploads that
  cannot actually transcribe through Ollama's API

**Consequence:** the local clone uses Ollama for stage 2 (LLM cleanup — its
sweet spot) and a dedicated local ASR engine for stage 1.

## 5. Local building blocks

### Stage 1 — local speech-to-text engines

| Engine | Strengths | Notes |
|---|---|---|
| **faster-whisper** (Python, CTranslate2) | ~4x faster than reference Whisper; CUDA + CPU; built-in VAD; easiest to integrate from Python | Best default for a Python app |
| **whisper.cpp** | C/C++, runs on Metal/CUDA/Vulkan/CPU; `--stream` mode; ideal sidecar for Electron/native apps | ~10x realtime with large-v3 on Apple Silicon Metal |
| **mlx-whisper** | Apple-Silicon-native (MLX) | Mac-only |
| **NVIDIA Parakeet** (via sherpa-onnx or parakeet.cpp) | Dramatically faster than Whisper (50–100x realtime on CPU; TDT v3 up to ~3000x on GPU); near-instant push-to-talk feel | v3 covers 25 languages; Whisper still wins for broad multilingual |

Model size guidance for dictation (short utterances, latency-sensitive):
- GPU / Apple Silicon: `large-v3-turbo` or `distil-large-v3`
- CPU-only: `small` / `base` / `distil-small.en`, or Parakeet on CPU
- Anti-hallucination: gate recording with VAD (e.g. `silero-vad`) so silence
  is never sent to Whisper (Whisper invents text on silent audio)

### Stage 2 — Ollama cleanup layer

- Any instruct model works; good fits: `llama3.1:8b`, `qwen2.5:7b` (16GB+ RAM
  or GPU), `llama3.2:3b` / `qwen2.5:3b` (CPU-only machines), `gemma3:4b`
- Temperature 0, strict system prompt: *rewrite only — never answer, never add
  content*; return only the cleaned text
- Personal dictionary = custom vocab injected into the prompt (plus Whisper's
  `initial_prompt`/hotwords for stage 1 biasing)
- Tone matching = detect frontmost app name, select a per-app style instruction
- Keep the model warm with Ollama's `keep_alive` to avoid cold-start latency
- Fallback: if Ollama times out, insert the raw transcript (never block the user)

### App shell (per OS)

| Concern | macOS | Windows | Linux |
|---|---|---|---|
| Global hotkey | pynput / Quartz event tap (Accessibility permission needed) | pynput / RegisterHotKey | pynput (X11); Wayland needs compositor-level shortcuts |
| Audio capture | `sounddevice` (mic permission) | `sounddevice` | `sounddevice` (PipeWire/Pulse) |
| Text insertion | Clipboard set → synthetic ⌘V → restore clipboard; or char-typing fallback | Clipboard → Ctrl+V via SendInput | X11: `xdotool type`; Wayland: `wtype`/`ydotool` (caveats) |
| Frontmost app detection (tone matching) | NSWorkspace | Win32 `GetForegroundWindow` | `xprop`/compositor APIs |
| Permissions | Microphone + Accessibility (System Settings) | none special | ydotool needs uinput perms on Wayland |

Clipboard-paste is what shipping apps (incl. OpenWhispr) use: it is instant
and works with non-ASCII text; save-and-restore the user's clipboard around
the paste, with slow character-typing as a fallback for apps that block
synthetic paste.

## 6. Prior art worth studying (or forking)

| Project | Stack | Takeaway |
|---|---|---|
| [OpenWhispr](https://github.com/OpenWhispr/openwhispr) | Electron 41 + React + TS; whisper.cpp + sherpa-onnx (Parakeet) for STT; llama.cpp for local cleanup; SQLite history | Closest full-featured open-source Wispr Flow clone; cross-platform |
| [FreeFlow](https://github.com/zachlatta/freeflow) | Swift, macOS-only; cloud STT (Groq) + OpenAI-compatible cleanup — **explicitly supports Ollama as the cleanup endpoint** | Validates the STT + Ollama-cleanup split; hold-to-talk & toggle modes |
| VoiceInk | Swift, macOS, GPL v3, local Whisper | System-wide local dictation reference |
| OmniDictate | Python, Windows, real-time local dictation | Python feasibility reference |

A from-scratch build is entirely reasonable — the MVP is ~4 components
(hotkey, recorder, ASR, injector) — and keeps you in control of the
Ollama-centric design.

## 7. Latency budget (the make-or-break metric)

Wispr Flow feels instant (~1s). A local clone on decent hardware can hit 1–3s:

| Step | Target | How |
|---|---|---|
| Stop-of-speech → transcript | 0.3–1.0s | small/turbo model, GPU/Metal; optionally transcribe chunks *while* recording |
| Transcript → cleaned text | 0.5–2.0s | 3–8B model kept warm; skip LLM entirely for very short utterances; stream nothing to the user until complete |
| Injection | <0.1s | clipboard paste |

CPU-only machines should default to Parakeet or `distil-small.en` plus a 3B
cleanup model — or offer a "fast mode" that skips the LLM stage.

## Sources

- [Wispr Flow](https://wisprflow.ai/) · [tl;dv review](https://tldv.io/blog/wisprflow/) · [Zapier review](https://zapier.com/blog/wispr-flow/) · [Voibe review](https://www.getvoibe.com/resources/wispr-flow-review/) · [Willow review](https://willowvoice.com/blog/wispr-flow-review-voice-dictation) · [Spokenly review](https://spokenly.app/blog/wispr-flow-review)
- [Weesper: Does Wispr Flow work offline?](https://weesperneonflow.ai/en/blog/2026-02-09-wispr-flow-review-cloud-dictation-2026/) (cloud architecture, subprocessors)
- [Wispr docs: Command Mode](https://docs.wisprflow.ai/articles/4816967992-how-to-use-command-mode) · [Wispr docs: hotkeys](https://docs.wisprflow.ai/articles/2612050838-supported-unsupported-keyboard-hotkey-shortcuts)
- [ollama#5451 STT request](https://github.com/ollama/ollama/issues/5451) · [ollama#11798 audio input](https://github.com/ollama/ollama/issues/11798)
- [OpenWhispr](https://github.com/OpenWhispr/openwhispr) · [FreeFlow](https://github.com/zachlatta/freeflow) · [OmniDictate](https://aicybr.com/blog/omnidictate-free-local-ai-dictation-windows)
- STT engine comparisons: [Parakeet vs Whisper](https://spokenly.app/blog/parakeet-vs-whisper) · [whisper.cpp vs faster-whisper benchmarks](https://www.promptquorum.com/power-local-llm/local-whisper-stt-comparison-2026) · [Northflank open-source STT benchmarks](https://northflank.com/blog/best-open-source-speech-to-text-stt-model-in-2026-benchmarks)
- [pynput](https://pypi.org/project/pynput/) (global hotkeys / keyboard control)

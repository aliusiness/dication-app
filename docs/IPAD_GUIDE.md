# Getting a Wispr-Flow-Like Experience on an iPad (No Coding Needed)

**Situation:** the owner's only computer is an iPad Air M1. The desktop clone
described in PLAN.md cannot run on iPadOS — Ollama does not exist for iPad,
and Apple only allows system-wide text input through custom keyboards, which
can't be built or installed without a Mac and a developer account. Wispr Flow
itself has no dedicated iPad app (only a limited iPhone version that can run
on iPad).

**The good news:** an iPad Air M1 already includes both halves of the Wispr
Flow pipeline, built in and running on-device:

| Wispr Flow stage | Built-in iPad equivalent |
|---|---|
| Speech-to-text | Keyboard dictation (mic button) — runs on the iPad, works offline, auto-punctuates |
| AI cleanup / tone rewriting | Apple Intelligence **Writing Tools** (Proofread, Rewrite, Friendly/Professional/Concise) — M1 iPads support this |

## Setup (one time)

1. **Turn on dictation:** Settings → General → Keyboard → turn on **Enable
   Dictation**.
2. **Turn on Apple Intelligence:** Settings → **Apple Intelligence & Siri** →
   turn it on (requires iPadOS 18.1 or later — update in Settings → General →
   Software Update if the option isn't there).

## Daily use (in Mail, Messages, Docs — anywhere)

1. Tap into any text field and tap the **microphone key** on the keyboard.
2. Speak naturally; tap the mic again when done.
3. To clean it up: **select the text** → tap **Writing Tools** in the popup
   menu → choose **Proofread** (fix mistakes) or **Rewrite** (change tone:
   Friendly / Professional / Concise).

That two-step flow — dictate, then Writing Tools — is the same two-stage
pipeline Wispr Flow runs in the cloud, except it all happens on the iPad.

## If more is wanted later

- **Wispr Flow iPhone app on iPad:** installable from the
  [App Store](https://apps.apple.com/us/app/wispr-flow-ai-voice-keyboard/id6497229487),
  but it is iPhone-sized on iPad and officially unsupported; a real iPad app
  is in development per
  [Wispr's docs](https://docs.wisprflow.ai/articles/2428135343-wispr-flow-on-ipad).
- **If a Mac or Windows computer is ever available:** PLAN.md in this repo is
  a ready-to-build blueprint for the fully local Whisper + Ollama clone.

# CLAUDE.md

This file guides Claude Code (claude.ai/code) and other AI assistants working in this repository. Follow it in every session.

## Project Overview

`dication-app` is a dictation (speech-to-text) mobile app for iOS and Android.

**Planned stack:** React Native with Expo, written in TypeScript. This was chosen because:

- One codebase covers both iOS and Android.
- Expo handles native builds, microphone permissions, and app store publishing.
- The owner can test changes on a real phone quickly using Expo Go.

**Current status: no application code exists yet.** The repository holds only documentation. Nothing has been scaffolded — there is no `package.json`, no Expo project, and no source files. Do not describe build, test, or lint commands as if they exist; there are none until the app is created.

## Repository Structure

The entire tracked contents of the repo today:

```
.
├── CLAUDE.md    # This guidance file
├── README.md    # One-line project title, no other content yet
└── prompts/     # Portable AI prompts (not app code)
    ├── coach-hormozi-williamson.md   # Ready-to-paste goal-coach system prompt
    └── coach-agent-spec.md           # The same coach as a structured 7-part agent spec
```

There is no `src/`, no `app/`, no configuration, and no dependencies. When you read a request that assumes running code, check the file tree first and correct the assumption if needed. The `prompts/` folder holds standalone AI prompts the owner uses elsewhere; it is not part of the dictation app's runtime.

## How to Work With the Project Owner

The owner describes features in plain English and reviews plans before any code is written. In every session:

- Ask at least three clarifying questions before starting any complex task.
- Present a plan and wait for approval before building a feature.
- Never guess when important information is missing — ask instead.
- When several approaches exist, explain the tradeoffs in simple terms.
- Review your own work before delivering it.

## Communication Style

- Write in clear, conversational English with simple language.
- Avoid buzzwords, corporate jargon, and vague statements.
- Explain concepts as if speaking to an intelligent beginner.
- Use short paragraphs, strong structure, and practical examples.
- Keep outputs concise — no filler, and stay within the requested format.

## Git Workflow

- The default branch is `main`.
- Do feature work on separate branches pushed to `origin`; the owner reviews before merging.
- Use clear, descriptive commit messages.

## Scaffolding Checklist (do this when the app is first created)

The moment real code lands (for example via `npx create-expo-app`), update this file so future sessions are productive immediately. Fill in the real details for each item below and remove this note:

- **Setup:** how to install dependencies (e.g. `npm install`) and any environment or Expo account requirements.
- **Run:** how to start the app locally (e.g. `npx expo start`) and how to open it in Expo Go or a simulator.
- **Build:** how production builds are produced (e.g. EAS Build) and where config lives.
- **Test:** the test runner and command, once tests exist.
- **Lint / format:** the exact lint and formatting commands and their config files.
- **Architecture:** the folder layout, navigation approach, state management, and where the speech-to-text integration lives.
- **Conventions:** naming, component patterns, and any project-specific rules worth following.

Until those exist, keep the "Current status" and "Repository Structure" sections above accurate.

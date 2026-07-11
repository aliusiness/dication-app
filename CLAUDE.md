# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`dication-app` is a dictation (speech-to-text) mobile app for iOS and Android.

**Planned stack:** React Native with Expo, written in TypeScript. Chosen because one codebase covers both platforms, Expo handles builds, microphone permissions, and app store publishing, and the owner can test changes on a real phone quickly with Expo Go.

**Current status:** No application code exists yet. When the project is scaffolded (e.g. `npx create-expo-app`), update this file with the real build, test, and lint commands.

## How to Work With the Project Owner

The owner describes features in plain English and reviews plans before code is written. Follow these rules in every session:

- Ask at least three clarifying questions before starting any complex task.
- Present a plan and wait for approval before building a feature.
- Never make assumptions when important information is missing — ask instead.
- When multiple approaches exist, explain the tradeoffs simply.
- Review your work before delivering it.

## Communication Style

- Write in clear, conversational English with simple language.
- Avoid buzzwords, corporate jargon, and vague statements.
- Explain concepts as if speaking to an intelligent beginner.
- Use short paragraphs, strong structure, and practical examples.
- Keep outputs concise — no filler content, stay within requested formats.

## Git Workflow

- The default branch is `main`.
- Do feature work on separate branches pushed to `origin`; the owner reviews before merging.
- Keep this file current: once the app is scaffolded, document the commands and architecture here so future sessions can be productive immediately.

# LightNotes

A local-first notes and productivity app. Open it at the start of a work session to see what needs doing and how much time is left. Tasks have live countdowns, recurring tasks track streaks, and every task keeps a markdown log.

> **Status:** Phase 0 (foundations). No app code yet.

## Planned features
- A Timers page with Short, Long and Continuous (recurring) tasks, each showing a live countdown ring coloured by urgency
- Session timers on tasks; a grace period, late completion and backdated completion
- Markdown notes with Obsidian-style live preview, autosave, folders, tags, attachments and full-text search
- Reminders that fire even when the app is closed
- Later: statistics, accounts, and sync with automatic merging across devices

## Platforms
Linux → Browser → Android → Windows → iOS

## Tech stack (planned)
Tauri 2 · TypeScript · Svelte 5 · CodeMirror 6 · SQLite · Vitest · Playwright

## Documentation
| Document | Purpose |
|---|---|
| [docs/SRS.md](docs/SRS.md) | Software Requirements Specification: **what** it does |
| [docs/SDD.md](docs/SDD.md) | Software Design Document: **how** it is built |
| [docs/development-guide.md](docs/development-guide.md) | Workflow, TDD, CI/CD and the roadmap checklist |
| [docs/adr/](docs/adr/) | Architecture Decision Records |
| [docs/devlog/](docs/devlog/) | Weekly development log |

## Getting started
*Coming in Phase 0, once the app is scaffolded.*

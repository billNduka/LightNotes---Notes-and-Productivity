# LightNotes — Software Requirements Specification
*Version 0.2 (draft) · 2026-11-03*

Requirements use **Must** (needed for that phase), **Should** (wanted, can slip) and **Could** (nice to have). Items marked **TBD** link to [[#12. Open questions]]. How the app is built is described in [[SDD]].

---

# 1. Purpose
LightNotes is a local-first notes and task app. You open it at the start of a work session and it shows, at a glance, **what needs doing and how much time is left** for each task. Each task keeps a running log of progress, written as an ordinary markdown note.

It is also a learning project: a vehicle for practising good engineering (testing, architecture, CI, documentation) with minimal AI assistance.

### 1.1 Why not just Obsidian?
Obsidian stores notes well but does not track running tasks implicitly. Nothing in it counts down, reminds you, or tells you how consistently you are doing a recurring habit. LightNotes puts the tasks first and attaches notes to them.

# 2. Scope
**In scope:** tasks with countdowns, recurring tasks, reminders, markdown notes with live preview, task logs, statistics, offline use, and optional accounts with cross-device sync.

**Out of scope (for now):** collaboration/sharing with other people, a public plugin API, wikilinks/backlinks/graph view, calendar integration, AI features.

# 3. Glossary
| Term | Meaning |
|---|---|
| **Task** | Something to do. Has a title, type, status, due date or schedule, and one log note. |
| **Short task** | A task due soon (e.g. an assignment due in 2 days). |
| **Long task** | A task due more than a week away (e.g. a reminder to invest). |
| **Continuous task** | A recurring task (e.g. play keyboard daily, post weekly). Made of occurrences. |
| **Occurrence** | One scheduled instance of a continuous task, with its own due time and completion. |
| **Countdown** | The live timer on a task card showing time remaining until it is due. |
| **Session** | A fixed-length work timer started from a task (e.g. 30 min of practice). |
| **Grace period** | Extra time after a due time during which completing still counts as on time. |
| **Missed** | Past due + grace and not completed. Can still be completed late. |
| **Note** | A markdown document. |
| **Log note** | The note attached to a task. Entries are appended under date/time headings. |
| **Guest mode** | Using the app without an account; all data stays on the device. |

# 4. Users
- **Primary:** the author, one person on several devices (Linux desktop, browser, Android phone).
- **Possible later:** other individuals with the same need. Each user's data is private.

# 5. Functional requirements

## 5.1 Tasks (FR-T)
- **FR-T1 (Must)** Create, edit, archive and delete tasks. Fields: title, type, status, due date/time *or* recurrence rule, icon, tags.
- **FR-T2 (Must)** Statuses: **To-do**, **In progress**, **Completed**. The time of every status change is recorded (needed for stats). For continuous tasks, the status belongs to the **current occurrence**, not the task as a whole.
- **FR-T3 (Must)** Short and long tasks have a single due date/time. The user chooses the type when creating the task, and it never changes automatically.
- **FR-T4 (Must)** Continuous tasks have a recurrence rule: daily, weekly, specific weekdays, or every *N* days. Each occurrence is either *scheduled* (do at 19:00) or *deadline-based* (post by Sunday 23:59).
- **FR-T5 (Must)** Completing an occurrence records it and schedules the next one; the continuous task itself stays active.
- **FR-T6 (Must)** A missed occurrence is **flagged as missed and carried over**. It stays visible until completed (late) or dismissed, and the next occurrence is still scheduled as normal.
- **FR-T7 (Must)** Overdue tasks and missed occurrences are visibly flagged. **Back-to-back misses are shown too:** a continuous task's card shows how many open missed occurrences it has (e.g. "missed ×3") and a strip of recent occurrences (done / late / missed).
- **FR-T10 (Must)** **Grace period:** each task has a grace period after its due time (default 1 hour for short and continuous tasks, 1 day for long tasks; editable per task). Completing within the grace period counts as **on time**. After it, the task or occurrence becomes *missed*, but can still be completed, which counts as **late**.
- **FR-T11 (Must)** **Backdated completion:** when marking something done, the user can say when it was actually finished ("done at 18:40"), since they may not have had their device with them. The completion time can't be in the future or before the task was created.
- **FR-T12 (Must)** A late completion **does not break a streak** (FR-S4). Only missed occurrences that are dismissed, or still open, break it. Late completions still count against the on-time rate (FR-S1).
- **FR-T8 (Must)** **Urgency colour:** each card's ring colour shows how much time is left, as a share of the task's time window (e.g. green > 50 %, amber 20–50 %, red < 20 %, a distinct overdue colour). Exact thresholds are configurable.
- **FR-T9 (Must)** Reminders: zero or more per task, as offsets from the due time (e.g. at due, 1 hour before, 1 day before). **Reminders fire even when the app is closed:**
  - Desktop: a small background process in the system tray, started at login.
  - Android: notifications scheduled with the operating system.
  - Browser: Web Push, which needs a server, so browser reminders arrive with Phase 2.

## 5.2 Timers page (FR-P)
The home screen. Visual reference: the MultiTimer Android app (grid of coloured rings).
- **FR-P1 (Must)** Shows tasks as a grid of cards, each with a ring and a live countdown, grouped or filterable by type (Short / Long / Continuous) and status.
- **FR-P2 (Must)** All countdowns run simultaneously, with no limit on how many are visible.
- **FR-P3 (Must)** Countdowns are calculated from stored timestamps (`dueAt − now`), never by incrementing a counter. They stay correct after the app is closed, the device sleeps, or it restarts.
- **FR-P4 (Must)** The countdown format adapts to the time left: `12d 4h` when over a day, `hh:mm:ss` when under a day, `overdue 2h` when past.
- **FR-P5 (Should)** The ring shows the share of the task's time window used up (from creation or last occurrence until due).
- **FR-P6 (Must)** Quick actions on a card: start (→ In progress), complete, add log entry.
- **FR-P7 (Must)** Selecting a card opens the task's log note.
- **FR-P8 (Must)** A sidebar lists the 10 most recently edited notes.

## 5.2b Session timers (FR-D)
Every timer belongs to a task; there are no standalone timers. A task can have a **session length** (e.g. "Play keyboard: 30 min"). Starting a session runs a MultiTimer-style countdown for that length. Pomodoro (5.7) builds on these.
- **FR-D1 (Must)** A task optionally has a session length. Its card can start, pause, resume and reset a session.
- **FR-D2 (Must)** Any number of sessions run at the same time, alongside the deadline countdowns.
- **FR-D3 (Must)** A finished session raises an alarm/notification, even if the app is closed (same mechanisms as FR-T9).
- **FR-D4 (Must)** Sessions survive closing the app: a running one is stored as its start time plus duration; a paused one as its remaining time.
- **FR-D5 (Must)** Every session is recorded against its task (and occurrence), which gives time-spent stats (FR-S8). Finishing a session offers to mark the task or occurrence complete.
- **FR-D6 (Should)** A finished session appends a line to the task's log, e.g. `Session: 30 min`.

## 5.3 Notes (FR-N)
- **FR-N1 (Must)** Markdown editor with **live preview**: formatting renders in place; the raw markdown appears only on the line being edited (as in Obsidian).
- **FR-N2 (Must)** Autosave: changes are persisted within 1 second of the last keystroke, with no save button. Closing or crashing the app loses at most 1 second of typing.
- **FR-N3 (Must)** Tags written as `#tag` in a note; notes can be filtered by tag.
- **FR-N4 (Must)** Images and file attachments can be pasted or dragged into a note and are stored with it.
- **FR-N5 (Must)** Note list with search across titles and content.
- **FR-N6 (Must)** Export one note or everything as plain `.md` files plus attachments.
- **FR-N7 (Must)** Folders, which can be nested. A note lives in exactly one folder; tags work across folders.

## 5.4 Task logs (FR-L)
- **FR-L1 (Must)** Every task has exactly one log note, created with the task.
- **FR-L2 (Must)** "Add log entry" appends a heading with the current local date and time (e.g. `## Tue 3 Nov 2026 · 14:32`) and places the cursor beneath it.
- **FR-L3 (Must)** A log note is a normal note: same editor, tags, search and attachments.
- **FR-L4 (Could)** Status changes (started, completed, missed) are optionally appended to the log automatically.

## 5.5 Statistics (FR-S) — Phase 2
Ideas, to be trimmed later:
- **FR-S1** **On-time rate:** % of tasks/occurrences completed before their due time.
- **FR-S2** **Promptness:** average time completed before (or after) the deadline, overall and per type.
- **FR-S3** **Start delay:** how much of a task's time window passes before it moves to *In progress*. This measures procrastination.
- **FR-S4** **Streaks** for continuous tasks: current and longest run of completed occurrences.
- **FR-S5** **Consistency:** % of occurrences completed in the last 7/30/90 days, per continuous task.
- **FR-S6** **Activity heatmap** (GitHub-style) of completions and log entries per day.
- **FR-S7** **Best times:** which weekdays/hours you most often complete tasks.
- **FR-S8** **Time spent** per task, once focus/Pomodoro sessions exist (Phase 3).
- **FR-S9** **Estimate vs actual**, if tasks gain an optional effort estimate.

## 5.6 Accounts and sync (FR-A) — Phase 2
- **FR-A1 (Must)** Every feature except sync works without an account (guest mode).
- **FR-A2 (Must)** Accounts use email + password. Implementation approach (own auth vs a provider, hosting) deferred to Phase 2.
- **FR-A3 (Must)** Logging in for the first time merges the device's guest data into the account; nothing is lost.
- **FR-A4 (Must)** Edits made on several devices, including while offline, **merge automatically**. Concurrent edits to the same note merge at character level; there are no "conflict copy" files.
- **FR-A5 (Must)** Concurrent edits to task fields merge per field (last writer wins for that field only).
- **FR-A6 (Should)** A visible sync status (synced / pending / offline).

## 5.7 Pomodoro (FR-F) — Phase 3
- **FR-F1** Start a focus session (Pomodoro) on a task, with configurable work/break lengths.
- **FR-F2** Completed sessions are logged against the task and feed FR-S8.

# 6. Non-functional requirements
- **NFR-1 Performance:** no perceptible lag.
  - Typing in a 10,000-line note: every keystroke rendered within one frame (16 ms).
  - Timers page stays at 60 fps with 100 running countdowns. All countdowns share one update tick per second.
  - Cold start under 1.5 s on the author's Linux machine.
- **NFR-2 Offline:** every feature except sync and account management works with no network.
- **NFR-3 Durability:** local storage is transactional; a crash never corrupts data.
- **NFR-4 Portability:** all data can be exported at any time as markdown + JSON, readable without LightNotes.
- **NFR-5 Platforms** in priority order: **Linux → Browser → Android → Windows → iOS.**
- **NFR-6 Security (Phase 2):** passwords hashed with a modern algorithm (argon2/bcrypt) or delegated to a trusted provider; all traffic over TLS; each user can access only their own data.
- **NFR-7 Accessibility:** fully usable by keyboard. Colour is never the only signal (rings also show text). Light and dark themes.
- **NFR-8 Maintainability:** code is split into modules (tasks, scheduling, notes, storage, sync, UI) with clear interfaces, so features like Pomodoro and stats can be added without touching unrelated modules. *This is what "extensible" means here.*

# 7. Data model (draft)
All IDs are UUIDs generated on the device, which offline use and sync require. This is a summary; the authoritative schema is in [[SDD#5.2 Database schema]].
```text
Task          { id, title, type, status, dueAt?, recurrence?, graceMs, sessionMs?,
                icon, tags[], logNoteId, createdAt, updatedAt, archivedAt? }
StatusEvent   { id, taskId, occurrenceId?, from, to, at }
Occurrence    { id, taskId, dueAt, status, completedAt?, dismissedAt? }
Reminder      { id, taskId, offset }
Session       { id, taskId, occurrenceId?, durationMs, startedAt?, remainingWhenPausedMs?,
                endedAt? }
Folder        { id, name, parentId? }
Note          { id, folderId, title, content, tags[], createdAt, updatedAt }
Attachment    { id, noteId, fileName, mimeType, contentHash, size }
```
# 8. Technical direction (to be confirmed with ADRs)
The core idea: build the app **once as a web app** (HTML, CSS, TypeScript), then wrap it so the same code runs as a desktop app, a phone app and a website.

| Concern | Proposed choice | What it is, and why |
|---|---|---|
| App shell | **Tauri 2** | Wraps a web app into a native app for Linux, Windows, macOS, Android and iOS, using the system's built-in web view. Much smaller and lighter than Electron (which ships a whole copy of Chrome). Also gives access to native features we need: system tray, notifications, files, SQLite. |
| Language | **TypeScript** (+ a little **Rust**) | TypeScript is JavaScript with types: the compiler catches mistakes before you run the code. Browsers only run JavaScript, so a browser target makes it the natural choice. Rust is Tauri's own language, needed only for small native pieces. |
| UI framework | **Svelte 5** (alternative: React) | Keeps the screen in sync with the data, so you don't update the page by hand. Svelte does most of its work at build time, so there's less code running in the app. React is more popular, with more jobs and tutorials, but a little heavier. |
| Editor | **CodeMirror 6** | A text-editor library for the web, and the engine under Obsidian's editor. It handles cursors, selection, undo and huge files; we write the live-preview rendering on top of it. |
| Local storage | **SQLite** (desktop/mobile), **IndexedDB** (browser), behind one storage interface | A database is a single file that the app can query, e.g. "all tasks due this week, sorted" or "notes tagged #uni". Writes are all-or-nothing, so a crash never leaves half-saved data (NFR-3). Plain `.md` files would work for notes, but tasks, occurrences, reminders and stats don't fit in files, and browsers can't write files freely. Export (FR-N6) keeps your notes portable. |
| Merging (Phase 2) | **Yjs** (a CRDT) | See §8.1. |
| Testing | **Vitest** (unit tests), **Playwright** (drives the real app to test it end-to-end) | |
| Sync server / auth | Decide in Phase 2 | |

## 8.1 Why merging needs Yjs
Say you write in a note on your laptop while offline, and also on your phone. When both reconnect, the server holds two different versions of the same note.
- **Last write wins:** keep whichever saved last. The other device's edits are silently lost.
- **Conflict copies:** keep both as separate files and make you merge by hand (what Dropbox does).
- **Merge:** combine both sets of edits automatically. This is what you asked for (FR-A4).

Merging plain text correctly is hard. Comparing two versions can't always tell what each device intended, especially if text moved or both edited the same line. A **CRDT** (Conflict-free Replicated Data Type) solves this. It records each edit as an operation ("insert 'x' after character #123") instead of saving a snapshot of the whole note. Every device can then apply every other device's operations, in any order, and all end up with exactly the same text. **Yjs** is a mature CRDT library, and it already has a CodeMirror integration.

Yjs is deferred to Phase 2. Turning an existing note into a Yjs document is a one-time conversion, so Phase 1 only needs data that is *ready* to sync: device-generated IDs and `updatedAt` on every record.

# 9. Roadmap
1. **Phase 1 — MVP (Linux desktop):** tasks, recurrence, timers page, duration timers, notes with live preview, autosave, folders, logs, tags, attachments, reminders, export.
2. **Phase 1b — Browser:** same app as a web build or PWA, using browser storage. No reminders while the tab is closed yet.
3. **Phase 2:** statistics, accounts, sync with Yjs merging, browser push reminders.
4. **Phase 3:** Android, Pomodoro, then Windows and iOS.

# 10. Engineering practices (learning goals)
- Git with feature branches, PRs reviewed by me, and conventional commit messages.
- Domain logic (countdowns, recurrence, stats) is pure code with no UI, written test-first.
- Lint, format, type check and tests run in CI (GitHub Actions) on every push.
- An **ADR** (Architecture Decision Record) for every significant choice, kept in `docs/adr/`.
- A dev log of problems hit and how they were solved.
- AI used for planning and reviewing, not for writing the code.

# 11. Decisions log
| # | Question | Decision | Where |
|---|---|---|---|
| 1 | Short vs long: chosen or derived? | Chosen by the user, never changes; time left shown by colour | FR-T3, FR-T8 |
| 2 | Plain duration timers before Pomodoro? | Yes | 5.2b |
| 3 | Missed occurrences? | Flagged as missed **and** carried over | FR-T6 |
| 4 | Reminders when the app is closed? | Yes | FR-T9 |
| 5 | Folders as well as tags? | Yes | FR-N7 |
| 6 | "In progress" on a continuous task? | Applies to the current occurrence | FR-T2 |
| 7 | Store notes as `.md` files or in a database? | Database, plus export | §8 |
| 8 | Back-to-back misses? | Shown on the card as a count plus a history strip | FR-T7 |
| 9 | Standalone timers? | No; every timer belongs to a task (session length) | 5.2b |
| 10 | Late completion? | Allowed; grace period counts as on time; backdating allowed; doesn't break streaks | FR-T10–12 |

# 12. Open questions
None right now. New ones go here as they come up during development.

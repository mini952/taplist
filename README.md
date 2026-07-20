# TapList

A personal, single-user PWA for **retroactively verifying** that a recurring task was actually done across a fixed past date range — and spotting the gaps. It's one self-contained `index.html` (vanilla JS, no framework, no build step) that runs entirely in your browser, with no server and no login. Its data lives on-device in IndexedDB.

The core idea in three points:

1. **Retroactive audit, not a habit tracker** — you give it a recurrence rule plus a fixed past date range, and it generates every occurrence in that span.
2. **Verify the past** — you tap to confirm each occurrence that was actually done; red cells are the gaps you're hunting.
3. **At-a-glance health** — a home overview shows coverage and gap counts across every list.

This repository is **deployed and in daily use**. The build phase is done; current work is real-world use plus small, well-scoped polish. A cross-device sync backend is scoped but deliberately parked (see `PROJECT_PLAN.md`).

---

## What's in here

```
index.html        The whole app — one self-contained file (HTML + CSS + JS inline).
icon.png          512x512 app icon (home-screen / PWA install target).
README.md         You are here.
PROJECT_PLAN.md   The living source of truth: purpose, data model, decisions, roadmap.
```

---

## How to run it

Open the live URL — GitHub Pages serves it straight from this repo:

**https://mini952.github.io/taplist/**

On a phone, use **Add to Home Screen** (iOS Safari) to install it fullscreen with the app icon. On desktop Chrome/Brave, use **⋮ → Cast, save, and share → Install page as app**.

To update the live app: edit `index.html` in this repo, commit to `main`, and Pages redeploys in a minute or two. (Hard-refresh, or reopen the installed app, to clear the cached version.)

---

## Where the data lives (worth knowing)

TapList's data — your lists and which occurrences are checked — is stored **on each device in IndexedDB**, not in this repo. Two consequences:

- Each browser/device keeps its **own** checkmarks; opening the same URL elsewhere starts fresh.
- **Export / Import JSON** (in the app header) is the manual backup and the device-to-device bridge. Keep those exports somewhere private — they hold your real data and must **never** be committed to this public repo.

This repo is the source of truth for the **app and its project docs**, not for your personal data.

---

## Canonical doc & raw-URL base

`PROJECT_PLAN.md` is the living brief — data model, design decisions, roadmap, and debrief notes. It is the authoritative context for any future work session.

Files are fetched (by tooling, or by a Claude Project) via the branch-tracked raw base:

```
https://raw.githubusercontent.com/mini952/taplist/refs/heads/main/<path>
```

# TapList — Project Brief & Living Doc

*A personal, single-file web app for **retroactively verifying** that a recurring task was actually done across a fixed past date range — and spotting the gaps.*

This document is the living brief for the TapList project (formerly "Audit Tracker"). It lives in the repo as `PROJECT_PLAN.md` and is fetched fresh from its raw URL at the start of each Claude session, so every new chat works from the current version. It pairs with the general **Personal App Development Framework & SOP** (the parent process doc) — that file governs *how* we build; this file governs *this specific app*.

---

## ⏭️ CURRENT STATE (start here in the next chat)

**Deployed and installed. The build phase is done.** TapList is live on GitHub Pages at:

> **https://mini952.github.io/taplist/**

It's installed to the iPhone home screen via Safari → Add to Home Screen, launches fullscreen with the app icon, and IndexedDB storage is reliable (the `file://` flakiness is gone).

**The core is done; the stance is still "use it for real, then debrief."** Keep Export JSON as the occasional backup and phone↔laptop bridge, and let the §9 debrief questions answer themselves through actual use rather than pre-emptive changes. The only larger build work on the horizon (the Supabase backend, §6) stays parked until manual Export/Import genuinely annoys.

**Latest iteration (two quality-of-life features added — see §4a):** (1) **rename a list in place** — tap the list's name in the detail view to edit it inline; (2) **completion celebration** — a confetti burst + "All done" popup when a list crosses into 100%. Both were deliberate, small, well-scoped additions on top of the finished core, not a reopening of the build phase. Deploying them is a plain `index.html` file-replace on GitHub Pages (§6).

Everything below is settled context.

---

## 0. Naming & deployment facts (settled)

- **Display / brand name:** **TapList** (camelCase). Set in the page `<title>`, the `apple-mobile-web-app-title` meta (the label under the home-screen icon), and the PWA manifest `name` / `short_name`. The in-app home-screen header also reads "TapList."
- **Why the rename:** "Audit Tracker" felt too narrow. TapList reflects the broadened scope — not just auditing, but quick checklists for verifying any repeating task was done. (Shortlist stress-tested: QuickList, Kept, Re:Check, TapList, Checked. All are crowded app-name spaces with taken domains, but this is a personal GitHub Pages tool, not a public product, so collisions don't matter — TapList was simply the favorite.)
- **Repo slug:** lowercase **`taplist`** (no spaces, no separator — fused one-word convention). The lowercase slug is independent of the styled "TapList" brand in the manifest.
- **Hosted file:** **`index.html`** (renamed from `audit-tracker.html`). Naming it `index.html` is what serves the app at the clean root URL `.../taplist/` and gives Add-to-Home-Screen a clean install target.
- **Icon:** a real **`icon.png`** (512×512, white checkmark on brand green `#1D9E75` — visually "a done cell") sits alongside `index.html`. Referenced by `<link rel="apple-touch-icon" href="./icon.png">` and in the manifest `icons` array.
- **Paths are relative** (`./icon.png`, `start_url:"."`) so nothing breaks under the `/taplist/` subpath.

---

## 1. The point of the project

Build a small tool to answer questions like: *"Did I actually update my step tracker every Tuesday for the past 6 months?"*

This is a **retroactive audit**, not a forward-looking habit tracker. The distinction drives the whole design:

- It takes a **recurrence rule** + a **fixed past date range** and generates every occurrence in that span.
- You go through and **check off** each occurrence you can confirm was done.
- The value is **verifying the past and surfacing gaps** at a glance — not scheduling or reminding.

### Primary use case
Confirming a weekly-on-a-weekday task over ~6 months (the step-tracker / Tuesday example).

### Other intended use cases
Any "prove I did X on schedule over a past window" check — e.g. monthly backups (1st of the month over a year), an every-N-days routine, or a daily log.

---

## 2. Core concept & data model

An **audit** = a named recurring task + a fixed date range + a record of which occurrences are done. *(Note: "audit" remains the internal term for a single entry, even though the app is now called TapList.)*

```
audit = {
  id,                       // stable unique id (generated, never an array index)
  name,                     // e.g. "Update step tracker"
  rule: {                   // one of four types:
    type: "daily"           //   every day
        | "weekly"          //   { weekdays: [0..6] }  (0 = Sun … 6 = Sat)
        | "everyN"          //   { n: <int> }          every N days from start
        | "monthly",        //   { dayOfMonth: 1..31 } (months lacking the day are skipped)
    ...
  },
  startDate, endDate,       // fixed range, "YYYY-MM-DD", set at creation
  createdAt,
  done: { "YYYY-MM-DD": true }  // checkmarks keyed by the occurrence's own date
}
```

**Key design point:** occurrences are *generated on the fly* from `rule + range`; only the `done` map is stored, keyed by date string. Keying checkmarks by date (not by list position) means editing/reordering/regeneration can never silently wipe checked state — the single most important persistence lesson from the framework.

---

## 3. Decisions log (settled)

1. **Multiple audits**, built multi-ready — each audit stored under its own key, stable IDs, data separate from rendering. A future "duplicate audit" is then trivial.
2. **Fixed date range**, set at creation (not a rolling window). The occurrence list is therefore stable.
3. **All four recurrence types** in v1: daily, specific weekday(s), every N days, monthly on a day-of-month.
4. **Single `.html`** (vanilla JS, no dependencies) — installable, offline-capable, keepable. Not `.jsx`. Persistence = **IndexedDB** (with in-memory fallback). *One deliberate exception:* a static `icon.png` ships alongside it (see §0 and §5) — the single-file principle is about no dependencies / portability, which one static image doesn't compromise.
5. **Two-screen structure:**
   - **Home = overview grid.** One row per audit sharing a common left axis: name + `done / total` + gap count + a compact strip of colored squares (one per occurrence). **No per-square date labels** — cadences/ranges differ between audits, so it's a pure at-a-glance health read. Tap a row to open it.
   - **Detail = per-audit grid.** The audit name is the title (so it's unambiguous this is one audit). Rows = months, each square = one occurrence, color = state. Tap a square to toggle.
6. **Color states** (derived from `done` + today's date, so the only user action is one tap-to-toggle):
   - **Green** = done.
   - **Red** = gap: not done, and the date is today or in the past (the thing you're hunting).
   - **Neutral gray** = upcoming: not done, date still in the future (only appears if a range runs past today).
   - A fresh 6-month audit starts mostly red and turns green as you verify — intentional and honest.
7. **Cross-device sync is deliberately NOT in v1.** Manual **Export / Import JSON** is the bridge (see §6).
8. **Deployed via GitHub Pages** (see §0 and §6) — settled and done.

---

## 4. v1 feature set (built + deployed)

- Create an audit: name + recurrence rule + fixed start/end dates.
- Quick-fill buttons for start date: **last 3 / 6 / 12 months** (dates stay fixed once set).
- All four recurrence types, with conditional form fields per type.
- Home overview grid (colored strips, coverage + gap counts).
- Per-audit detail grid (months as rows, occurrences as tappable squares, legend).
- Tap to toggle done / not-done; persists to IndexedDB (debounced save).
- Delete an audit (with confirm).
- **Export / Import JSON** (manual cross-device bridge + backup; import upserts by id).
- PWA install meta + inline manifest + real app icon → Add to Home Screen launches fullscreen with the TapList checkmark icon.
- Full dark mode via `prefers-color-scheme`.
- Clean-minimal design tokens: accent green `#1D9E75`, thin 0.5px borders, ~12px card radius, two font weights.
- **Live and installed** on GitHub Pages at `https://mini952.github.io/taplist/`.

Recurrence logic was verified against edge cases (month boundaries, monthly-31st skipping short months, inverted ranges, weekday correctness).

---

## 4a. Later additions (post-v1)

Two quality-of-life features added after the core shipped. Both are **detail-view only** and touch none of the internal identifiers (§5) — existing stored data and backups stay compatible.

**Rename a list in place (name only).** In the detail view, tap the list's name to edit it inline; **Enter** or blur saves, **Escape** cancels, an empty value reverts to the previous name. The rename persists immediately and shows on the home screen on return. This is name-only editing — it never touches the rule or date range, so occurrence generation and checkmarks are untouched. (Rule / date-range editing remains deferred, §8.)

**Completion celebration.** When a list *crosses into* 100% — every square green, i.e. `done === total` — a confetti burst (canvas, no dependency) plus a centered "All done / N of N verified" popup fires. **Tap anywhere to dismiss** — no auto-timeout (a deliberate choice). Key behaviors:
- Fires **only** on the tap that crosses into complete; **opening an already-complete list does nothing** (the trigger lives solely in the cell-tap handler, not in `renderDetail()`).
- **Refires** if you uncheck a square and recheck back to full.
- Honors `prefers-reduced-motion`: the popup still shows, the confetti is skipped.
- **"Complete" means every square green, including any future/gray cells** if a range runs past today. For a normal all-past audit that's just clearing the reds; for a range with future occurrences you'd have to green those too. (Flagged as a debrief question, §9.)

---

## 5. Technical architecture (summary)

- **One self-contained HTML file** (`index.html`) — HTML + CSS + JS inline, no external dependencies — plus one static `icon.png` (the only accompanying asset; a real file rather than an inline data URI because iOS Safari picks up `apple-touch-icon` reliably only from a real image at a URL).
- **Data layer:** structured JS objects as the single source of truth; `save()` debounced to IndexedDB; `load()` on boot. Falls back to in-memory (session-only) with a warning toast if IndexedDB is unavailable.
- **Render layer:** a `render()` that switches between home / create / detail views from the data; targeted `renderDetail()` on toggle so a tap doesn't re-render everything.
- **Interaction:** event delegation on the main container (survives re-renders).
- **Occurrence generation:** date-safe helpers (`ymd` / `parseYMD` build local dates from Y/M/D integers to avoid timezone off-by-one bugs).
- **Safety:** storage calls wrapped in try/catch; user-entered text escaped before insertion into HTML.
- **Feedback:** lightweight toast for confirmations.

**Notes on the §4a additions:**
- The detail-view **list name is rendered in the header** (`#hTitle`), not in the body `detail-head` (which shows the rule + coverage). So rename targets the header title, gated to `view==="detail"` via an `editable` class toggled in `render()` (the same `#hTitle` reads "TapList" on home and "New taplist" on create, which must stay non-editable).
- The **celebration overlay** (`#celebrate` popup) and a full-screen `<canvas id="confetti">` live **outside `#app`** so they survive `renderDetail()`'s `innerHTML` rewrites. Confetti is a small `requestAnimationFrame` particle system; the canvas is sized to `devicePixelRatio` and re-sized on window resize.
- **Completion is detected by comparing `stats(a)` before vs after the toggle** and firing only on a false→true crossing — which is what makes "never on open" and "refire after uncheck" both fall out for free.
- **Tunable knobs:** confetti density = the `i<90` loop in `spawnConfetti`; palette = `CELEBRATE_COLORS` (set to just `#1D9E75` for strictly-on-brand green); popup wording = the `All done` / `Every occurrence verified` copy.

**Multi-ready checklist (all done in v1):** stable unique IDs · data separate from render · seed/create writes a clean cloneable structure · per-record storage.

**⚠️ Don't casually rename internal identifiers.** The IndexedDB database name (`auditTracker`) and the Export payload tag (`app:"audit-tracker"`) were intentionally left unchanged during the TapList rename. Changing the DB name would orphan all existing stored data; changing the export tag could break continuity with existing backup files. These are invisible to the user and should stay as-is. (Some in-app body copy — the empty state, the "N audit(s)" counts, the export filename `audits-YYYY-MM-DD.json` — also still says "audit"; harmless and deliberately left, see §8.)

---

## 6. Cross-device, hosting & the "real app" branch

### Hosting (GitHub Pages) — DONE
- Repo `taplist` (public — required for free-tier Pages), Source = "Deploy from a branch," `main` / root.
- `index.html` + `icon.png` at the repo root; relative paths throughout.
- **What it gave us:** a real HTTPS URL → reliable IndexedDB (vs. flaky `file://`) and proper Add-to-Home-Screen install.
- **What it did NOT give us:** cross-device sync. Open the same URL on laptop and phone and each browser keeps its *own* separate checkmarks.
- **Updating the live app:** edit `index.html` in the repo (or re-upload), commit → Pages redeploys in a minute or two. On the phone, close/reopen the installed app or hard-refresh Safari to shake loose the cached version.

### The manual bridge — Export / Import JSON
The current cross-device story. Export packages all audits into one JSON file; move it to the other device (AirDrop / email / iCloud Files); Import loads it. Manual snapshot, not live sync. Also serves as the backup.

### The "real app" graduation (parked, deliberate — not yet)
True auto-sync requires a backend + accounts. Scoped estimate (Supabase, for a person with these skills):

- **Effort:** ~a focused weekend for a minimal version — one `audits` table, Supabase Auth (email magic-link or Google), swap the IndexedDB data layer for Supabase queries, online-only, last-write-wins conflicts. Offline support is a further step up.
- **Biggest jump:** authentication — the moment data lives on a server it needs login, even for one person on two devices. This changes the app's character.
- **Cost:** $0 on the Supabase free tier (500 MB DB, 50k MAUs — vastly more than needed) as of mid-2026. Two gotchas: free projects **pause after 7 days of inactivity** (fix: a free GitHub Actions scheduled ping, ~20 min) and **no automatic backups** (mitigated by periodic Export JSON). Pro is $25/mo if we ever want backups + no pause; likely unnecessary for a personal tool.
- **The Export/Import JSON format is the migration path in** — the seed for loading existing data into a backend.

**Recommendation (per the framework — graduate deliberately, not preemptively):** now hosted and installed, live with Export/Import for a while, and only build the backend once manual sync genuinely annoys — which is also the best way to learn what the sync should actually do before building it.

---

## 7. Development plan / roadmap

1. **✅ Done:** host on **GitHub Pages** (reliability + install) and rename to TapList. *Live at `mini952.github.io/taplist/`, installed on iPhone.*
2. **▶ Now:** use it for real; keep Export JSON as an occasional backup and the phone↔laptop bridge. *Real-use observations, not new features, drive what happens next.* (The two §4a additions were a brief, deliberate exception — small polish on the finished core, then back to using it.)
3. **Debrief after real use** (see §9) and fold lessons back into this brief and the app.
4. **Later, if the manual bridge starts to hurt:** scope the Supabase backend as its own project (§6).

---

## 8. Deferred features (named, not lost)

- Editing an audit's **rule or date range** after creation (still delete-and-remake). *Name editing is now done (§4a); rule/range editing stays deferred because changing the range regenerates the occurrence set — a real jump beyond a label change.*
- "Nth weekday of month" recurrence (e.g. *first* Tuesday) — a real complexity jump.
- Per-occurrence notes / timestamps.
- Reordering audits on the home screen.
- An explicit "confirmed missed" state, distinct from "not yet checked."
- Cross-device auto-sync / accounts (the §6 "real app" branch).
- Full TapList rename of in-app body copy (empty state, "N audit(s)" counts, export filename). User-visible *branding* is done; this remaining "audit" wording is cosmetic and low-value — do only if it starts to grate.

---

## 9. Debrief questions to revisit after real use

- Does the **auto-red-in-the-past** behavior feel right, or is a fresh all-red audit off-putting? (Alternative: neutral until you actively mark a miss.)
- Will **in-place editing** be wanted sooner than "deferred" implies?
- Is the **home strip** readable at real scale (e.g. a daily audit = ~180 squares)?
- Does **manual Export/Import** hold up, or is the backend worth building?
- Does the **completion celebration** stay rewarding without getting annoying on lists you complete often? And is **"every square green (incl. future cells)"** the right definition of complete, or should it be "no reds left" (past occurrences only)?

---

*Keep this doc updated after each meaningful iteration or debrief.*

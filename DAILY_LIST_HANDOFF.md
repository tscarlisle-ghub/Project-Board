# Handoff — adding the Daily List to the Project Board site

Scott decided to move this work over to the TSC-BOARD project on Cowork rather than continue building the standalone Daily List. These notes are the briefing for whoever (Claude or Scott) picks it up over there.

## What Scott wants

Add the **Daily List** as the **first page** of the TSC-BOARD site. Use the project board's existing look-and-feel (typography, masthead, navbar, color palette, spacing language) — but keep all of the Daily List's existing functionality. The current standalone Daily List can be retired or kept as an archive once this is wired in.

## Where the source material lives

- **Standalone Daily List (DONE — design language is set):** `/Users/tscarlisle/Dropbox (Personal)/-Claude Cowork/DAILY LIST/`
  - `index.html` — the working single-file app
  - `data.json` — its starter data file
  - `SESSION_NOTES.md` — running log of every design decision, in plain language. Worth scanning before you start; it explains *why* each typographic and behavioral choice is the way it is.
  - `README.md` — the GitHub-sync + Shortcut setup guide

- **Project Board site (the destination):** `/Users/tscarlisle/Dropbox (Personal)/-Claude Cowork/TSC-BOARD/`
  - `index.html` — current project board (1,783 lines, ~80KB). Sticky masthead → sticky navbar → sticky staff filter → grid of project entries below.
  - `data.json` — projects data (different shape from Daily List's data; do not collide).
  - `session-notes.md` — its own running log of the project board's evolution.
  - `Logo_Only.png`, `archive/`, `index copy.html`.

## What the Daily List does (functionality to preserve)

- Single-file vanilla-JS app, no framework.
- Lets you add, check off, defer, un-defer (Today / Tomorrow / Next Week / Next Month), and delete items.
- Storage: a JSON file in a GitHub repo, read/written via the GitHub API using a Personal Access Token the user pastes into a Settings panel (stored in `localStorage` on each device). LocalStorage is also a cache so the page loads instantly.
- Sync indicator under the settings button: "Syncing…", "Synced", "Synced — N from iPhone", "Offline — using local cache", "Save failed — will retry".
- iPhone bridge: a Shortcut named "Daily List" pulls Reminders Today and PUTs a JSON file into `inbox/` in the repo. On load (and every 5 minutes, and on window focus) the app drains that folder, merges items deduped by lowercased text, and deletes the inbox files.
- Completed items linger on the main view for 30 seconds (so undo is possible), then collapse into a `Completed (n)` panel that mirrors the existing `Deferred (n)` collapsible.
- Defer row appears on hover (desktop) or tap (mobile), and auto-hides after 4 seconds if nothing is chosen. Only one row is expanded at a time.

## What the project board site already looks like (relevant excerpts)

It uses the same Typekit kit Scott has been using (`ikf0hkb`), but a different family palette:

- Body: `warnock-pro` (serif)
- UI: `ff-meta-web-pro` and `ff-meta-web-pro-condensed` (sans-serif)
- Headings / project titles: `interstate-condensed`

Color tokens (already defined as CSS variables in TSC-BOARD's `index.html`):

```
--ink: #2e2e2e
--ink-mid: #6a5e50
--ink-light: #9a8e80
--ink-faint: #eeebe4
--page: #ffffff
--rule: #d8e4ec
--rule-strong: #a0b4c8
--brand: #d04030    (the red used for active nav state and accents)
--green: #3a7060
--amber: #a06010
--red: #d04030
--teal: #2a5a6a
```

Layout:

- Three sticky bars at the top of the page: masthead (`.masthead`, ~74px tall, sticky at top:0), navbar (`.navbar`, sticky at top:74px), staff filter (`.staff-bar`, sticky at top:111px).
- Masthead contains: title "Project Board" set in `interstate-condensed` 40px bold uppercase, current date, version label, logo.
- Navbar has `<button class="nav-link">` entries — Board · Gantt · Archive · Due · | All (staff filter starts here).
- Each navbar button calls `showPage('board' | 'gantt' | 'archive' | 'summary')`. There's already a single-page-app pattern in place — switching pages is a function, not a navigation.
- Body width is capped at 1600px, padded 40px.

This means: **the Daily List should be added as another `showPage(...)` view inside `index.html`** — *not* as a separate file. That way it shares the masthead, navbar, sticky behavior, and color/type tokens for free.

## Recommended integration approach

1. Add a new nav-link to the navbar — likely **"Daily"** as the first link, before "Board". So the navbar order becomes: Daily · Board · Gantt · Archive · Due.
2. Add a new view container (e.g., `<div id="page-daily">…</div>`) that the existing `showPage()` function can show/hide alongside the existing pages.
3. Inside that container, rebuild the Daily List UI using the project board's existing CSS variables and font families. Translation table:
   - **Daily List title** → use `interstate-condensed` like `.masthead-title` already does (no need for a separate big title — the masthead is the title now). The view itself can open with a small phase-style label, or no label.
   - **Item text** → `interstate-condensed`, 26px, uppercase, similar treatment to `.project-name`.
   - **Defer/Delete hover row** → `ff-meta-web-pro-condensed`, 11–12px, uppercase, 0.08–0.14em letter-spacing — same as `.proj-act-btn` already in the site. `.proj-act-btn.danger` already exists for the delete style.
   - **Section labels (Completed, Deferred)** → use `.phase-label` styling (`ff-meta-web-pro-condensed` 13px 0.18em uppercase with a bottom rule).
   - **Background** → `--page` (white).
   - **Hairlines between sections** → `--rule` (light blue-gray) rather than the brown tone used in the standalone app.
4. Use the GitHub-sync logic from the standalone Daily List's `<script>` block essentially as-is, but:
   - Pick a different data file name in the repo so it doesn't collide with the project board's `data.json`. Suggestion: `daily.json` (and the inbox folder becomes `daily-inbox/`).
   - The PAT / owner / repo settings can be reused if the project board uses the same auth mechanism; if not, add a separate Daily List settings panel (or fold it into the project board's existing settings if there is one).
   - The sync-status indicator should live in the existing nav-right area — the project board already has a `.sync-status` class with `.ok`, `.err`, `.working` modifiers. Use those.
5. Make the Daily List the *default* view (first thing shown when the page loads). Right now `nav-board` has `class="active"` — switch the active class and the initial render call to the new daily view.
6. Carry over (or rebuild) the inbox-drain logic and the iPhone Shortcut. The Shortcut PUTs into `daily-inbox/` (or whatever folder name is chosen). The Shortcut instructions in `DAILY LIST/README.md` can be updated to reflect the new repo + folder names.

## Open decisions when work resumes

- **Same repo or different repo?** TSC-BOARD likely already lives in its own GitHub repo. Daily List can live in the same repo (simpler — one PAT, one Settings panel) or stay separate. Simpler is same-repo with `daily.json` alongside the existing `data.json`.
- **Should Daily and Board share completion / staff filtering?** Probably no — Daily List is personal to Scott; the project board's staff filter is firm-wide. Keep them independent.
- **Mobile target.** The project board is currently desktop-optimized (1600px page, 40px padding). The Daily List has been tuned for iPhone portrait too. The Daily view in the integrated site should keep mobile-friendly padding and the auto-hide defer-row behavior already built.
- **What happens to the standalone `DAILY LIST/` folder?** Once the integration works, Scott can either archive it (`archive/daily-list/`) or just leave it as a reference. Don't delete `SESSION_NOTES.md` from it without copying the content into the TSC-BOARD session notes — those notes contain the design rationale.

## File-level summary for the picker-upper

- Read `DAILY LIST/SESSION_NOTES.md` in full. It's the design rationale.
- Read `DAILY LIST/index.html` in full. The CSS-variable definitions, the JS module structure (config / sync / list-ops / render), and the buildList function with its `kind` switch are all reusable.
- Read `TSC-BOARD/index.html` masthead and navbar (~lines 819–840) and the CSS-variable block (~lines 12–32) to ground the type/color translation.
- Make changes by adding a new view + updating `showPage()` and the navbar — don't fork into a separate file.

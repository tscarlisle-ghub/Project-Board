# Project Board — Session Notes

## 2026-04-28

**Imported prior chat history** — Scott pasted in the full transcript of the original Claude.ai conversation (share link `2364ccad-...` was unreachable due to network allowlist). Conversation covered the entire build of the project board app from initial concept through V1 (full-featured) and into V2 (Tufte-inspired redesign).

**State of the app at end of imported chat:**
- V1 live at `https://tscarlisle-ghub.github.io/Project-Board/`
- V2 live at `https://tscarlisle-ghub.github.io/Project-Board/project-board-v2.html`
- Both share `data.json` in the same repo
- V2 most recent change: two-column project grid + archive/delete in bottom right of card
- Version constants in V2: `APP_VERSION = '1.00'`, `APP_VERSION_DATE = 'Apr 27, 2026 · 3:45 PM'`

**Memory established:**
- Scott's role + working style
- Project Board app architecture and current state
- Code delivery preferences (run `node --check`, full files, V1 vs V2 explicit)
- Session notes preference (this file, written after every change)

No code changes made this entry — purely context import and memory setup.

---

**Promoted V2 to be the new `index.html`** — Scott downloaded the live repo files into `TSC-BOARD/` and asked to make `project-board-v2.html` the main entry point.

Actions taken:
- Verified `index copy.html` (93,974 bytes) is preserved as the backup of the old V1.
- Overwrote `index.html` with a byte-for-byte copy of `project-board-v2.html` (73,315 bytes). `cmp` confirmed identical.
- Left `project-board-v2.html` in place so existing links to that URL keep working — can be deleted from the repo later if desired.
- Confirmed no self-references inside V2 (no internal links to `project-board-v2.html` or `index.html`) needed rewriting.
- Confirmed `data.json` is loaded via the GitHub API (`GH_USER`/`GH_REPO`/`GH_FILE` constants), not a relative path, so it works identically regardless of the host filename.
- Ran `node --check` on the extracted inline `<script>` block (44,951 bytes) — OK.

**Files in `TSC-BOARD/` after this change:**
- `index.html` — now the V2 app (73,315 bytes)
- `index copy.html` — backup of old V1 (93,974 bytes)
- `project-board-v2.html` — duplicate of new `index.html` (73,315 bytes)
- `data.json`, `Logo_Only.png` — unchanged

Next step for Scott: upload the new `index.html` (and optionally remove `project-board-v2.html`) to the GitHub repo via the web UI. The live URL `https://tscarlisle-ghub.github.io/Project-Board/` will then serve the V2 app.

---

**Major redesign: phase header + design phase slider + Tasks-by-Due-Date page** — Bumped APP_VERSION to **1.10** (`Apr 28, 2026 · 1:13 PM`). Both `index.html` and `project-board-v2.html` updated and mirrored byte-identical (79,328 bytes). `node --check` on the inline script block: OK.

Changes:

- **Phase label moved to top of each block.** Was a 220px left-gutter label beside a 1fr project grid; now a newspaper-section header (small caps, hairline rule under) sitting above the projects. Removed `.phase-block-inner` grid. `.phase-projects` simplified, padding adjusted to keep breathing room around cards.
- **2-column grid bug fixed.** Root cause: `renderProjects` was inserting an `<hr>` with `grid-column:1/-1` between every project, which forced each project onto its own row in the left column. Removed the HR insertion; bumped `.phase-projects` gap from 3px to 8px so cards still read as separate.
- **Design phase slider added to every project card.** Range input (`step="5"`, 0–100) with proportionally-banded background — SD 0–20% (palest), DD 20–40%, CD 40–90% (largest band, deeper steel), CA 90–100% (brand red). Thin dark-ink thumb. Phase abbreviations (SD/DD/CD/CA) labeled under the bands at the band start positions. New `proj-phase-label` next to project name shows e.g. `CD · 65%`. Two handlers: `oninput` updates state + label live (no re-render, so dragging stays smooth); `onchange` calls `scheduleSave()` (1.5s debounce → GitHub). New `designPercent` field defaults to 0; back-filled in `loadFromGitHub`/`loadLocal`/`createProject`.
- **Summary page replaced with "Tasks by Due Date".** Nav button: "Summary" → "Due". Page title: "Task Summary" → "Tasks by Due Date". `renderSummary()` rewritten — now buckets all open, dated tasks across active projects into Overdue (red, pinned at top) / Today / This Week (grouped by day) / Next Week (grouped by day) / Later (grouped by day). Each row: checkbox · task name (click → edit due date) · project (click → board) · staff · relative date chip ("3d ago · Apr 25", "Tomorrow · Apr 29", etc). Honors the active staff filter. Completed tasks excluded.
- **Version label darkened.** `color:var(--ink-faint)` (#eeebe4, barely visible) → `var(--ink-light)` (#9a8e80).

**Next-step suggestions parked for later** (per the design-review thread): client/site metadata strip on each card, design-phase pip indicator vs. slider trade-off, permit-status field, "last touched" relative-time signal, optional hero thumbnail per project. These touch the data model, so worth doing as a separate focused pass.

---

**Phase label restyled + slider bands rebalanced** — Bumped APP_VERSION to **1.11** (`Apr 28, 2026 · 1:26 PM`). Both `index.html` and `project-board-v2.html` mirrored byte-identical (79,344 bytes). `node --check`: OK.

Changes (CSS + DP_BANDS only — same data model):

- `.proj-phase-label` (the "CD · 65%" chip next to the project name) restyled: font-family bumped from `ff-meta-web-pro-condensed 11px` to `interstate-condensed 18px` (matching project-name family at ~60% larger than before), letter-spacing tightened to 0.03em to match project-name, color shifted from `--ink-mid` to `--ink-light` to keep it clearly subordinate to the name. `margin-left` 10→14px, added `line-height:1.1` so it sits on the project-name baseline cleanly.
- Slider bands rebalanced so CD and CA are visually equal width. New layout: SD 0–20% (20pp), DD 20–40% (20pp), CD 40–70% (30pp), CA 70–100% (30pp). Updated in three coordinated places: gradient stops in `.dp-bands`, `DP_BANDS` array `end`/`start`, the `if(pct<70)` cutoff in `designBand()`, and the CA `dp-band-label` left position 90%→70%. Step stays at 5%, range 0–100, so the slider still increments in 5% jumps within every phase.

Note that the underlying value is still 0–100 across the whole slider, so a project that's been pegged at "85%" pre-change will now read as `CA · 85%` instead of `CD · 85%`. If any projects are currently sitting above 70%, they'll jump from CD to CA. Worth a quick eye-check after the deploy.

---

**V2 retired** — moved `project-board-v2.html` to `TSC-BOARD/archive/project-board-v2.html`. Going forward, edits land in `index.html` only — no more byte-for-byte mirroring. `index copy.html` (the original V1 backup) is left in place at the top level for now; can be archived later if Scott wants.

After Scott pushes this state to GitHub, the URL `https://tscarlisle-ghub.github.io/Project-Board/project-board-v2.html` will return 404 — that's expected. The live app remains at `https://tscarlisle-ghub.github.io/Project-Board/`.

Memory updated to reflect single-file workflow. Also saved a deferred-feature note: Scott wants a Siri/Shortcuts "Add Project Task" voice command later — direct-voice flavor (no Reminders middleman), to revisit when he's ready.

---

**v1.12 — phase label rework + Gantt menu unified** (`Apr 28, 2026 · 1:38 PM`). `index.html` only (V2 retired). `node --check`: OK. File size 78,774 bytes.

Changes:

- **Phase chip next to project name now shows in-phase progress, not overall slider value.** `designPhaseLabel(pct)` was returning `KEY · ${pct}%` (e.g., `CD · 55%` for slider value 55). Rewrote it to compute `(pct - band.start) / (band.end - band.start) * 100` and round — so for slider 55 (which sits in CD's 40–70 band), the chip now reads `CD · 50%` (50% through CD). Boundary behavior: at slider 20/40/70, chip reads `0%` of the next phase; at 100, `CA · 100%`.
- **Phase chip restyled to title sibling.** Was 18px bold condensed; now 26px (matching `.project-name`), `font-weight:400` (not bold), kept `--ink-light` color so it stays clearly subordinate to the bold name. `margin-left` 14→24px for breathing room.
- **Gantt page now uses the main masthead/nav.** Previously the Gantt page hid `main-masthead` and rendered its own `.gantt-masthead` strip with title/date/back-button. Removed the gantt-masthead block, removed the `display='none'` toggle (now always `'block'`), and dropped the now-orphan `gantt-date-label` update from `showPage`. Net effect: the navbar (Board · Gantt · Archive · Due, plus staff filters and Sync/Import/Settings) stays visible on the Gantt page; clicking the **Board** nav link is the back-to-board action.

Note for Scott: the standalone "← Board" pill is gone. The **Board** link in the nav now does double-duty as back-to-board. If you'd rather have an explicit chevron-back button in the nav row alongside the regular links, easy to add — say the word.

Open thread: Scott reports he wants to be able to set/edit due dates on tasks and "thinks that used to be there." Investigating next — memory says the click-time-to-open-calendar popup should still exist; verifying whether it broke or is just not where he expects.

---

**v1.13 — task due-date editing made discoverable** (`Apr 28, 2026 · 1:48 PM`). `node --check`: OK. `index.html` now 79,025 bytes.

The mechanic for editing due dates was actually still wired up — it just had no visible affordance for tasks that had no date set. `buildTaskRow` was rendering the `task-due` div *only* when `fmtDue(t)` returned a non-empty string (i.e., the task already had a date). For tasks with no date, the only path to set one was the right-click context menu's "Edit due date" item, which is invisible and unusable on iPad.

Fix: always render the date row beneath every task. When the task has a due date, it shows the existing label as before ("Due in 3d · May 1", "Overdue · Apr 25", etc., color-coded); when it doesn't, it shows a faint italic "+ Set due date" placeholder that brightens on row hover. Either way, clicking opens the existing inline edit form with the calendar picker, and saveTaskTime persists `t.due` to `data.json` as before.

Also added a subtle hover underline on the date label so it reads as a clickable target on desktop. Right-click context menu still works as a secondary path.

**Tasks tracking review:** prior session tasks (#1–#8) all completed. None marked stale.

---

**v1.14 — slider snaps to 5% in-phase increments** (`Apr 28, 2026 · 1:56 PM`). `node --check`: OK. `index.html` 79,521 bytes.

Previously the slider had `step="5"` on the raw 0–100 range. Because the bands have different widths (SD 20pp, DD 20pp, CD 30pp, CA 30pp), the displayed in-phase percentages weren't clean — CD walked through 0%, 17%, 33%, 50%, 67%, 83% instead of 5% steps.

Fix:
- Slider input: `step="5"` → `step="any"` so the underlying value can land on any decimal.
- Added `snapDesignPercent(raw)` helper that, on every input event, looks up which band `raw` falls in, computes in-phase %, rounds to nearest 5%, and converts back to a raw value (e.g., 5% in CD = raw 41.5; 50% in CD = raw 55). The slider's `.value` is then forced to that snapped raw, so the thumb visibly clicks into 5%-spaced positions while dragging.
- Special case: hitting 100% of a non-final band rolls to 0% of the next band (so you never display "SD · 100%" — you cross straight into "DD · 0%"). At the end of CA, you can land on "CA · 100%".
- Also rounded `designPhaseLabel` output to nearest 5% so any pre-existing project values (saved before this change) display cleanly too. Stored values aren't rewritten on load — they just round in the label.

Sanity-checked the math against all four bands; each one reaches every 5% increment from 0% through 95%, plus CA can hit 100%. Boundary transitions verified at 20/40/70.

---

**Per-phase sliders + minty in-construction tint** — Replaced the single 0–100% design-phase slider with 4 separate sliders (SD/DD/CD/CA), each with 5% increments. Each slider's label shows its phase letter with the current % in smaller text right next to it (e.g. `SD` then `75%`). Filled track color matches the phase: SD light blue, DD mid blue, CD teal, CA red. Existing `designPercent` values auto-migrate to `phasePercents` on load so nothing is lost. Also changed the In-Construction phase background tint from light blue to light minty green (`#ecf6ee`).

- `index.html` — slider CSS, helpers, `buildProjectEntry`, load + create migrations
- Version bumped to 1.15 · Apr 28, 2026 · 4:30 PM

---

**v1.16 — phase rows restyled as sparklines, project total % beside name** (`Apr 28, 2026 · 5:45 PM`). `node --check`: OK. `index.html` 81,043 bytes.

Three coordinated changes per Scott's request ("more like sparklines with small dots, but I like the colors; show in larger type next to the project name the percent complete, and on the slider itself just above the dot, not next to the phase label"):

1. **Sparkline rows.** The 4px filled bar with a 6×14 vertical-bar thumb is replaced by a 1px hairline (`#d4cfc5`) with a colored 9×9 round dot thumb. Phase color (SD light blue, DD mid blue, CD teal, CA red) now lives in the dot itself plus the fill segment from 0→value; the rest of the row is just a hairline. Track height grew 14→22px to make room for the floating label above.
2. **Per-phase % moved above the dot.** Removed the `<span class="dp-pct">` from the `dp-row-label` (label is now just `SD`/`DD`/`CD`/`CA`, narrower flex 64→28px). Added a new absolutely-positioned `.dp-row-pct` element inside `.dp-row-track` whose `left:${v}%` and `transform:translateX(calc(-1% * var(--pct)))` keep it horizontally centered over the dot at any value (and snaps cleanly to the left edge at 0% / right edge at 100% so it never hangs into the project margins). Live updated on `oninput`.
3. **Project total % next to project name.** Added `projTotalPct(p)` helper computing weighted overall % (SD 0.20 + DD 0.20 + CD 0.30 + CA 0.30 — matches band widths). Rendered as `<span class="project-total-pct">` in the project head, set to 22px Interstate Condensed Bold, `--ink-light` so it reads as subordinate to the 26px project name. Also live-updated whenever any phase slider moves.

Verification: extracted the inline JS, ran `node --check` — clean. Bumped APP_VERSION 1.15 → 1.16 with timestamp. Force-reload reminder: append `?v=16` (or any new value) to the URL on Safari.


---

**v1.17 — phase-by-phase % readout, 2-column compact sliders** (`Apr 28, 2026 · 6:05 PM`). `node --check`: OK. `index.html` 81,138 bytes.

Two follow-up tweaks per Scott's request: "I want the percentage next to the project name to be sd%, dd%, etc." and "make the sliders two columns and more compact."

1. **Per-phase readout next to name.** Replaced `projTotalPct(p)` with `projPhaseSummary(p)` which returns `"SD 75% · DD 50% · CD 0% · CA 0%"` (joined by middle-dots). The `.project-total-pct` element now renders this string, with CSS swapped from 22px Interstate Condensed to 15px ff-meta-web-pro-condensed bold uppercase, `--ink-light` so it stays subordinate to the 26px project name. flex-wrap on the head row lets it drop below the name on narrow widths instead of overflowing. Live-update handler also switched to `projPhaseSummary`.
2. **Two-column compact slider grid.** `.dp-wrap` is now `display:grid; grid-template-columns:1fr 1fr; gap:2px 18px;` so SD+DD share one row and CD+CA share the next. Tightened the row internals: label flex 28→22px, label font 12→11px, gap 12→8px, track height 22→20px, hairline at top 14→13, input top 7→6, thumb 9×9 → 8×8 with margin-top 2.5→3, floating % font 9→8px. Net effect: the slider block now takes about half the vertical space it used to and reads as a tighter "phase dashboard."

Verification: extracted JS, ran `node --check` — clean. Bumped APP_VERSION 1.16 → 1.17. Force-reload `?v=17` in Safari.


## 2026-04-28 — v1.18: today as default add-task date + lighter phase % readout
- New tasks: the **+ Add task** form now defaults the due date to **today** (was: 7 days out). Used local date (year/month/day) so the boundary near midnight stays correct in Birmingham time.
- Project name % readout: changed to non-bold (font-weight 400, was 700) and reformatted from "SD 75%" to "SD:75%" with a colon between phase and percent.
- Bumped APP_VERSION → 1.18.
- Verified: `node --check` passed.
- Force-reload: append `?v=18` to the URL after pushing.

## v1.19 · Apr 28, 2026 · 8:30 PM
- **Phase % label above dot slider**: font-size 8px → 13px (60% larger); adjusted track height to 28px and repositioned band/input to match
- **Phase summary next to project name**: `%` signs now bold (`<b>%</b>`), rendered via innerHTML
- **More space between phases**: dp-wrap vertical gap 2px → 8px
- **Completed tasks darker**: row opacity 0.4 → 0.56 (40% darker)

## v1.20 · Apr 28, 2026 · 9:05 PM
- **"+ Set due date" placeholder readable on tinted backgrounds**: `.task-due.empty` color `--ink-faint` (#eeebe4) → `--ink-light` (#9a8e80). Hover state `--ink-light` → `--ink-mid` (#6a5e50) to keep the affordance distinct. Was nearly invisible against the cream/mint/blush phase tints (#fdfbf3, #ecf6ee, #fdf5f3 etc.) since `--ink-faint` is itself a near-cream color.
- **Completed task name no longer washed out**: `.task-name.done` color `rgba(200,195,185,0.9)` → `var(--ink)` (#2e2e2e). Row already fades to opacity 0.56 (set in v1.19), so the strikethrough text now renders as a clearly-readable muted dark gray instead of disappearing into the background. Strikethrough still applied.
- `node --check` on extracted inline JS: OK. `index.html` 81,205 bytes.
- Force-reload `?v=20` after pushing to GitHub.

## 2026-05-18 — v1.19: task delete usable on iPad
- The board page already had an ✕ delete button per task, but it was `opacity:0` until hover — invisible on iPad/iPhone Safari (no hover state).
  - Now: faintly visible by default (opacity 0.4), full opacity on hover, and bumped to 0.6 on touch devices via `@media (hover:none)`.
- Added a matching ✕ delete button on the **Tasks by Due Date** page rows (none existed before).
- `deleteTask()` now shows a `confirm("Delete task ...?")` prompt to prevent accidental taps. It also re-renders the summary page if you're on it.
- Bumped APP_VERSION → 1.19.
- Verified: `node --check` passed.
- Force-reload: append `?v=19`.

## 2026-05-22 — v1.21: Daily List integrated as first page
Followed the plan in `DAILY_LIST_HANDOFF.md`. The standalone Daily List (previously at `/Users/tscarlisle/Dropbox (Personal)/-Claude Cowork/DAILY LIST/`, since copied into `TSC-BOARD/DAILY LIST/` as reference) is now the **first page** of the project board site, sharing the masthead, navbar, type families, and color palette.

**Nav order is now:** Daily · Board · Gantt · Archive · Due. Daily is the default landing page. The `+ New project` sub-navbar auto-hides on the Daily page since it doesn't apply there; `showPage()` toggles it back on for the other pages.

**Type & color translation** (project-board language, not the brown/cream of the standalone Daily List):
- Item text: `interstate-condensed` 26px bold uppercase, `var(--ink)` — matches `.project-name`.
- Defer/Delete pills: `ff-meta-web-pro-condensed` 11px 0.14em uppercase — matches `.proj-act-btn`.
- Section/collapsible labels (`Completed (n)` / `Deferred (n)`): same as `.phase-label` — `ff-meta-web-pro-condensed` 13px 0.18em uppercase with a 1px hairline under.
- Check button: circular `var(--ink-mid)` 1.5px border, fills `var(--ink-light)` when done.
- New-item input: `interstate-condensed` 22px with a dotted underline in `--rule-strong`.
- Hairline color used inside Daily is `--rule` / `--ink-light` rather than the brown the standalone used.

**Behavior preserved from the standalone app** (all of it):
- Add via Enter on the input field. Spellcheck + autocapitalize-sentences + autocorrect on the input.
- Click circle to mark done. Completed items linger inline for 30s, then collapse into a `Completed (n)` panel (deletion available from there).
- Defer row appears on hover (desktop) or tap (mobile). Auto-hides after 4s if no action. Only one row expanded at a time.
- Defer destinations: Tomorrow · Next Week · Next Month · Delete. Deferred items get an extra `Today` action.
- `Deferred (n)` collapsible at the bottom for items waiting on a future date.

**Sync setup** — reuses the project board's existing `ghToken` and `GH_USER`/`GH_REPO`/`GH_BRANCH` constants. No separate Settings panel.
- Data file in the repo: **`daily.json`** (not `data.json` — that's the project board's).
- Inbox folder for the iPhone Shortcut: **`daily-inbox/`** (not `inbox/`).
- Pulls `daily.json` on init, after 5-minute interval, on window focus. Drains `daily-inbox/` (dedup by lowercased text), pushes the merged list back, deletes the consumed inbox files.
- localStorage cache key: `daily_v1_cache` (doesn't collide with the board's `pb_v3`).
- A missing `daily.json` is treated as an empty list (404 → no error). First save will create it.

**iPhone Shortcut change needed:** the existing "Daily List" Shortcut needs to PUT into `daily-inbox/` in the `tscarlisle-ghub/Project-Board` repo (not the old `inbox/` in the standalone repo). Same JSON shape, same PAT.

**Files touched:** `index.html` only. Added a CSS block (`#daily-page` and `.daily-*`) before `</style>`, added `<div id="daily-page">…</div>` before `#board-page` (with `style="display:none;"` added to `#board-page` so Daily is the default visible view), prepended the Daily nav button + separator, updated `showPage()` to toggle the new view + the sub-navbar visibility, changed `currentPage` default from `'board'` to `'daily'`, appended ~190 lines of Daily List JS module (sync, list ops, render, init) before the KEYBOARD section, and added a `dailyInit()` call after the existing GH bootstrap.

**Verification:** extracted inline `<script>` block (60,618 bytes) and ran `node --check` — clean. Final `index.html` is 97,872 bytes / 2,157 lines.

**Standalone `DAILY LIST/` folder** at `TSC-BOARD/DAILY LIST/` is preserved as reference. Once Scott confirms the integrated view works, that folder can be moved to `archive/daily-list/` (or deleted) — but its `SESSION_NOTES.md` should be kept somewhere since the design rationale for the typographic decisions lives there.

**Bump:** APP_VERSION → 1.21 (skipped 1.20 since the May 18 iPad-delete entry above is labeled v1.19 but the live file's `APP_VERSION` constant was still `'1.19'` — picking 1.21 keeps the version line monotonic from the file's actual state).

**Force-reload:** append `?v=21` to the URL in Safari after pushing to GitHub.

## 2026-05-22 — v1.22: Daily page becomes the due list
Scott: *"i want to populate the list with all of the tasks on the board; subtitle each item with the project name; this will become the due list, but keep it simple looking."*

Repurposed the Daily page from a standalone item list (with its own `daily.json` + inbox) into a **flat view of all open project tasks**. The Daily List concept as an independent data store is gone; everything now reads directly from `projects` (the project board's data).

**Each row:**
- Task name: Interstate Condensed 26px bold uppercase (unchanged from v1.21)
- Project name subtitle: ff-meta-web-pro-condensed 11px 0.14em uppercase, `var(--ink-light)`
- Due chip on the right: short relative + date ("Today · May 22", "3d late · May 19", "5d · May 27"). Color-coded — red for overdue (`var(--brand)`), amber for today, mid-ink for "soon" (≤7 days), light gray for anything further out.
- Tasks with no due date show no chip and sink to the bottom of the list.

**Sort:** by due date, earliest first. Undated tasks at the end, sorted alphabetically by project.

**Filtering:** honors the existing staff filter bar (All / Maddie / Kat / Kathleen). Changing the filter re-renders the Daily list immediately. `setStaffFilter()` now calls `renderDaily()` when on the daily page.

**Check circle:** marking complete sets the underlying `task.done=true` on the project board's data and calls `scheduleSave()` — so the change syncs to GitHub like any other board edit. The row disappears from the Daily view immediately (no 30-second linger; Scott asked for simple).

**What was removed:**
- The "Add new item..." input at the top of the Daily page (this list isn't a source of truth, it's a view).
- All the standalone Daily List sync machinery: `daily.json` / `daily-inbox/` fetch/push/drain, the `daily_v1_cache` localStorage cache, the `dailyItems` array, and the defer/Tomorrow/Next Week/Next Month/Today/Delete hover row. The iPhone Shortcut inbox path was wired into a separate data store that no longer exists — the Shortcut is currently a no-op until/unless we wire it into the project board's task model (worth a future thread; for now Scott can drop tasks into a project directly).

**What was kept:**
- `dailyEsc()` for HTML escaping
- The `.daily-item` / `.daily-check-btn` / `.daily-item-text` CSS (Interstate Condensed treatment matches the project-board's `.project-name` family — looks like a "real" page of the site, not a bolt-on)
- Project board's existing `dueDate()` and `dueDays()` helpers — used as-is to compute chip text

**`loadFromGitHub`** and **`loadLocal`** now both call `renderDaily()` after populating `projects`, so the list paints as soon as the data is in.

**Verification:** extracted inline script (53,406 bytes, down from 60,618) and ran `node --check` — clean. `index.html` is now 91,081 bytes / 2,069 lines.

**Force-reload:** append `?v=22` to the URL in Safari after pushing to GitHub.

**Open thread:** the existing `/Due` page (bucketed Overdue / Today / This Week / Next Week / Later) is now largely redundant with the new Daily view. Not removed yet — easy to drop once Scott confirms he doesn't need the buckets.

## 2026-05-22 — v1.23: scrap Daily, efficiency pass
Scott: *"scrap the daily. let's go back to how it was. revie teh codebase and let's see if we can make this a more efficent code."*

### Daily page fully removed
- CSS block (`#daily-page`, `.daily-*`, mobile @media): gone (~185 lines)
- Daily HTML page div + nav button + separator: gone
- JS: `renderDaily()`, `dailyDueChip()`, `dailyEsc()`, `dailyMarkTaskDone()`, `dailyInit()`, the `setInterval(renderDaily,60000)`, and the sub-navbar visibility hack: gone (~80 lines)
- `currentPage` default reset from `'daily'` → `'board'`; Board is the landing page again.
- Board page div had `style="display:none;"` (used while Daily was the default) — removed so Board paints on load with no flash.
- Existing **Due** page (Overdue/Today/This Week/Next Week/Later buckets) is unchanged and remains the way to see tasks across all projects by due date.

### Efficiency / dedup wins
- **Dead var removed**: `activeStaffFilter` was declared but never read (the real one is `activeStaff`).
- **`normalizeProjects()` helper**: the long inline `projects.forEach(p=>{p.tasks=...;p.type=...;p.staff=...;p.colorIdx=...;p.designPercent=...;ensurePhasePercents(p);})` was duplicated word-for-word in `loadFromGitHub` and `loadLocal`. Now one helper, called from both.
- **`renderCurrentPage()` helper**: the pattern `if(currentPage==='summary')renderSummary();else renderProjects();` was repeated in `openInlineAdd`, `cancelInlineAdd`, `openEdit`, `cancelEdit`, `deleteTask`. Replaced with `renderCurrentPage()`, which also correctly routes to renderArchive/renderGantt when on those pages. **Latent fix**: `markDoneCtx` previously called `renderProjects()` unconditionally, so checking off a task from the Due page didn't refresh the Due view — it now does, via the new helper.
- **`toggleTaskDone(pId,tId,closeEditor)` helper**: `markDone` and `markDoneCtx` were near-identical (only difference: whether `editingTask` is cleared). Both now thin wrappers around one function.
- **`setLastSyncLabel(d)` helper**: the "Synced [weekday] [month] [day] [time]" formatting was duplicated in `saveToGitHub` and the bootstrap. One helper now.
- **`showPage()` rewritten** to loop over a `PAGES` array (`['board','gantt','archive','summary']`) instead of five hardcoded `getElementById` calls plus a separate hardcoded loop for nav-link active state.
- **`setStaffFilter` loop simplified**: removed the dead `n==='all'?'all':n` ternary (collapses to just `n`).

### Latent CSS bug fixed
- `var(--font-condensed)` was referenced in 8 places (masthead-firm, gantt headers, modal labels, archive count, archive header, import buttons, ctx menu) but **the variable was never defined in `:root`**. Those font-families silently fell through to whatever parent style was in scope. Added `--font-condensed:"ff-meta-web-pro-condensed","ff-meta-web-pro",sans-serif;` and `--font-display:"interstate-condensed",sans-serif;` (utility var, ready to use to replace literal strings later) to `:root`.

### CSS consolidation
- Three repeated inline-style stanzas on `#today-date`, `#version-label`, `#last-sync-label` (each ~140-char `style="font-family:'interstate-condensed'…color:var(--ink-light)…"`) replaced with a single `.masthead-label` class + a tiny per-id rule for the variant spacing.

### Size delta
- Lines: **2,069 → 1,808 (-261, -12.6%)**
- Bytes: **91,081 → 81,539 (-9,542, -10.5%)**
- Inline JS: **53,406 → 48,993 (-4,413, -8.3%)**

### Verification
- Extracted inline `<script>` (48,993 bytes), `node --check` clean.
- `grep -i daily` against the final file: 0 hits.

### Force-reload
- Append `?v=23` to the URL in Safari after pushing to GitHub.

### Open thread (still deferred)
- iPhone Shortcut for voice-add tasks is still pending (was wired briefly into the Daily inbox in v1.21, then removed in v1.22). Not addressed in this pass.

## 2026-05-22 — v1.24: V5 redesign + billing integration
Scott: *"yes, do the lightest version and add back teh fee column; wire in the billing and rewrite it"*

Applied the **Project Board V5** handoff (`handoff 3/Project-Board-V5/`) with the user's customizations on top of the README. Pre-V5 file backed up to `archive/index-v1.23-pre-v5.html`.

### What's new

**Engraved masthead** — `CARLISLE MOORE` in Columbia Titling 18px caps (var `--tr-engraved`) + thin-spaced "Residential Architecture" eyebrow + mono Canterbury Lane address on the right. Replaces the bigger Interstate "Project Board" header (which now moves to the page-title row).

**V5 tab bar** — Board · Gantt · Archive · Due as Meta Condensed 11px caps with a 1.5px forest underline on the active tab. The right side carries last-sync, sync-status, and the three nav actions (Sync / Import / Settings). Replaces the old Interstate nav-link buttons.

**Left sidebar (Board view only)** — 220px, paper-1 background. Two groups:
- **Office** — All projects / Active / Prospects / On hold / Past due / Complete with live derived counts. Past due renders in `--v5-alert` (#8a2a1f) when non-zero.
- **Tools** — Billing → `tscarlisle-ghub.github.io/TSC-BILLING/`, Cost estimator → `.../TSC-SQF/`, Questionnaire → `.../TSC-quest/`. All open in new tabs.

**Page-title row** — `PROJECT BOARD` engraved + italic-serif subline ("9 projects · 6 active · 2 prospects · 1 past due", live) + hairline search input + ink-fill **+ New Project** button.

**Phase strip + Assigned strip** — Meta Condensed 9px labels. PHASE: All / Prospects / SD / DD / CD / CA / Done. ASSIGNED: All / Maddie / Kathleen / Kat. Italic-serif counts in forest-1. Middle-dot separators.

**Project list (7-col table)** — `stamp | identity | tasks | open | fee·billed | status | chevron`. Sticky header in Meta Condensed 10px ink-3 with a 1px ink-0 bottom rule. No pills. Each row:
- **Stamp**: mono 11px 0.18em caps in forest-1 (PROS / SD / DD / CD / CA / Done).
- **Identity**: client name in Columbia Titling 15px caps engraved + Warnock italic scope.
- **Tasks**: up to 3 inline (13×13 square checkbox, mono date in ink-3 or alert, Meta Web Pro 12px title, mono assignee in forest-1). Tasks dim to 35% when staff filter is set and their `who` doesn't match. "+ N more tasks" indicator in italic-serif if overflow.
- **Open**: mono open-task count, right-aligned.
- **Fee · billed**: mono fee + tiny "$X billed" stack — or italic "— pending fee" if no billing match.
- **Status**: italic Warnock right-aligned, color-coded by status word (active/past due/on hold/complete/prospect).
- **Chevron**: ▾ / ▴.

**Expanded detail** (one row open at a time) — forest-4 background with 2px forest-1 left rule. Phase + Type two-col grid. If billing fee>0: Fee/Billed/Remaining stat blocks in mono 14px + a 4px progress bar (paper-2 track, forest-1 fill) + "N% billed" label. Then the existing **SD/DD/CD/CA design-phase sliders** (kept as the source of truth — editing them updates the row's phase stamp). Then an Activity timeline (synthesized from each task's createdAt and done state, last 5). Dashed **+ Add task** button. A row of Edit / Archive / Delete actions at the bottom.

**Totals footer** — Below the list, separated by a 3px double rule (ink-0). Project count + total open + total fee + total billed.

**New Project modal** — Columbia Titling spaced title `N E W   P R O J E C T`, engraved rule underneath, three hairline-bordered fields (Client/project name, Type dropdown, Lead designer dropdown), ghost-outline Cancel + ink-fill Create.

### Data model — one new field

- **Per-task `who` (assignee)**. On load, every existing task that doesn't have a `who` inherits its project's `staff` value. The next save writes the migrated tasks back to `data.json` (so this happens once). New tasks default to: the chosen value in the assignee dropdown → otherwise the currently-active staff chip → otherwise the project's lead. No other new fields stored.

### Derived (not stored)

- **Phase stamp**: `PROS` if `type==='prospect'`; `Done` if every task is done; otherwise the highest phase with non-zero progress in `phasePercents` (CA > CD > DD > SD).
- **Status word**: `prospect` / `done` / `alert` (any open task overdue by >7 days) / `active`. `hold` left as a manual-only slot in the sidebar (count always 0 until a future field is added).

### Billing integration (lightweight, one-way read)

- At app load, `loadBilling()` fetches `https://tscarlisle-ghub.github.io/TSC-BILLING/data.json` (same-origin, no token). Falls back to the GitHub raw URL if Pages doesn't serve `data.json`. Silently degrades to `billingMap={}` if both fail — the Fee column then shows "— pending fee" everywhere with a console warning.
- **Join key**: normalized name. Both `project.name` and billing's `client.shortName`/`fullName`/`name`/`projectName` are lowercased and stripped of non-alphanumerics, then matched directly. So `"LaRussa"` ↔ `"larussa"` ↔ `"La Russa"` all collapse to `larussa`.
- **Aggregates per project**: `fee = client.currentFee`, `billed = sum(invoice.amount where sent or paid)`, `collected = sum(where paid)`, `outstanding = billed - collected`.
- The billing app **never gets written to** — Project Board reads it as a snapshot at load time.

### Preserved (untouched by the rewrite)

GitHub sync (PAT, `data.json` round-trip), Settings modal, Import modal, Calendar popup, Right-click context menu, SD/DD/CD/CA design-phase sliders (now inside the expanded detail), Gantt page, Archive page, Due/Summary page, the `dueDate()`/`dueDays()`/`fmtDue()`/`dueClass()` helpers.

### Removed

- All the old phase-block / project-card markup (`.phase-block`, `.phase-projects`, `.phase-new-construction`, `.project-entry`, `.project-footer`, `.project-total-pct`, `.add-task-btn`).
- The inline `+ New Project` panel (`.new-project-panel`, `.np-*`) — replaced by the new modal.
- The old top nav-link buttons, staff filter buttons, and sub-navbar (`.masthead-*`, `.nav-link`, `.nav-action`, `.staff-bar`, `.staff-btn`, `.sub-navbar`).
- The duplicate `setStaffFilter` function and `activeStaff` variable; replaced with `Object.defineProperty(window,'activeStaff',{get(){...}})` that reads from `v5State.staff` so the Due page picks up the V5 staff chip selection without duplicate state.
- All status pills (the README required this — status is italic Warnock right-aligned, no background fill).
- `buildNewProjectPanel`, `buildProjectEntry`, `buildTaskRow`, `buildEditForm`, `buildAddForm`, `npSelectType`, `npSelectColor`, `createProject`, the legacy `toggleNewProject`. Their behavior is rebuilt by the new `v5*` functions.

### Latent things fixed along the way

- `updatePhasePercentLive` no longer tries to look up `.project-entry` (which no longer exists); the per-row phase-summary readout is gone, the slider just updates its band/pct/fill in place.
- `openProjEdit` positions itself relative to the closest `.v5-row` (board) or `.arch-entry` (archive) rather than the old `.project-entry`.
- The proj-edit popup's click-outside handler now also tolerates `.v5-client` and `.v5-tab-action` so they don't close the popup unintentionally.
- `markDoneCtx` already flowed through `renderCurrentPage()` (from v1.23) so checking off a task from the Due page now refreshes the Due view correctly.

### Size delta (v1.23 → v1.24)

- Lines: 1,808 → 2,357 (+549, +30%)
- Bytes: 81,539 → 104,368 (+22,829, +28%)
- Inline JS: 48,993 → 60,627 (+11,634, +24%)

The growth comes from: sidebar HTML+CSS, billing fetch+map, derivation helpers, V5 render functions (header/row/expanded), new-project modal, the new design tokens. The old phase-block / project-card / inline-panel CSS and JS that's removed offsets some of it.

### Verification

- Extracted inline `<script>` (60,627 bytes), `node --check` clean.
- `grep -i daily`: 0 hits (Daily stays scrapped).
- `grep buildProjectEntry / buildTaskRow / setStaffFilter / showNewProject`: 0 hits in real code (only in the "removed in V5" comment marker).

### Force-reload

Append `?v=24` to the URL in Safari after pushing to GitHub.

### Open thread

- **Billing join verification**: works only if Project Board project names match billing client short-names after normalization. First time Scott loads the new Board, check the Fee column. Mismatches will say "— pending fee" — fix by renaming either side. If many mismatches, we can add a per-project `billingKey` override field as a follow-up.
- **`On hold` status**: sidebar slot exists but always shows 0 since there's no signal in the current data. Easy to wire: add an optional `onHold:true` flag to a project and update `v5ComputeStatus`.
- **Type dropdown in the new-project modal** — keeps the existing four (new-construction / renovation / in-construction / prospect). The V5 prototype's PROS phase is now derived from `type==='prospect'` automatically.
- **iPhone Shortcut for voice-add tasks** — still deferred.

## 2026-05-22 — v1.25: V5 polish to match the Claude Design screenshot
Scott shared a screenshot of the actual V5 prototype from Claude Design and asked to tweak everything to look more like it. Pulled the implementation closer to the design:

### What changed

**`lead` field is back** — reverses the v1.24 "drop Lead" call. Stored on each project as `'TSC'` or `'BM'` (defaults to TSC on first load). Shows as "Lead · TSC" / "Lead · BM" beneath the address line in each row's identity column. The expanded detail now resolves these to full names ("T. Scott Carlisle" / "Bill Moore") via a `LEAD_NAMES` lookup. New helper: `leadFull(code)`.

**Expanded detail is now 3-column** — `PHASE | LEAD | ADDRESS` (the Lead value gets a 2px forest-1 underline beneath it to match the screenshot's focus-line treatment, via `.v5-meta-value-emph`). Type still surfaces in the row's scope subtitle. The whole grid is center-aligned with a 1px ink-5 bottom rule for separation.

**Design-phase sliders moved out of the expanded detail** — they're not in the screenshot. Moved into the project-edit popup (the `#proj-edit-popup` that opens on a row's Edit button or by clicking the client name). The popup is now wider (480px) and has six sections: Project name · Type · Lead · Default assignee · Design progress (the SD/DD/CD/CA sliders) · Color. New helpers: `pepSetLead()`, `_pepLead`.

**Fee section restructured** — the progress bar is now full-width on its own line, with the right-aligned "N% billed" label below it. Previously it was squeezed into a flex item alongside the Fee/Billed/Remaining stat blocks.

**Phase strip cleaned up** — dropped the **Done** chip (kept All/Prospects/SD/DD/CD/CA — 6 chips matching the screenshot). Removed the middle-dot separators between phase chips (screenshot uses generous whitespace only). Bumped chip typography to Meta Condensed 11px 0.12em caps to look more like a label run. Italic-serif counts in forest-1 stayed.

**Assigned strip kept** the middle-dot separators (matches screenshot).

**Page title bumped** 22px → 28px Columbia caps with `line-height:1` so it baseline-aligns with the italic-serif subline.

**Search input got a magnifying-glass icon** — small SVG (circle + diagonal line) inside `.v5-search-wrap` which carries the hairline bottom border now, so the icon and input share a single underline.

**"+ NEW PROJECT" → all caps**.

**Chevron is now SVG** — drawn at 11×7 with a 1.2px stroke instead of unicode triangles, so it reads as a thin caret like the screenshot. New helper: `v5ChevronSvg(up)`.

**Expanded row reads as one continuous block** — `.v5-row.expanded` clears its bottom border so the row and the expand panel share the forest-tinted background without a visible seam.

**Sliders still drive the stamp** — editing the SD/DD/CD/CA percentages in the edit popup updates `phasePercents`, which `v5ComputePhase` reads, which determines the stamp ("SD"/"DD"/"CD"/"CA"). End-to-end live: change the slider in the popup → save → stamp updates on the next render.

**Type dropdown in the new-project modal** now also gets a "Principal lead" select (TSC/BM) defaulting to TSC.

### Latent bug fixed along the way

`.v5-chev` was declared twice (once near the row CSS and once near the status CSS), with different rules. Consolidated to one declaration plus a `.v5-row.expanded .v5-chev` override that darkens the chevron on the expanded row.

### What did NOT change

Billing fetch + join (still by normalized name, same fallback URLs), GitHub sync, Gantt, Archive, Due, Calendar, Import flow, Settings modal, all status pills (already removed in v1.24), task assignee field (already added in v1.24).

### Size delta (v1.24 → v1.25)

- Lines: 2,357 → 2,427 (+70)
- Bytes: 104,368 → 107,640 (+3,272)
- Inline JS: 60,627 → 62,253 (+1,626)

The growth comes from: lead field plumbing (~30 lines), 3-col expand grid + Lead label resolution, chevron SVG helper, search-icon SVG, project-edit popup additions (Lead toggle + slider section), and the new-project modal's Lead select.

### Verification

- `node --check` on extracted inline JS (62,253 bytes): clean.
- File copied to `TSC-BOARD/index.html` (107,640 bytes / 2,427 lines).

### Force-reload

Append `?v=25` to the URL in Safari after pushing to GitHub.

### Still open

- **Address field** in the expanded detail currently renders an em-dash placeholder ("—"). The project data doesn't carry a project address yet. Could pull from billing data (`client.projectAddress`) on next pass — the billing app's data already has it. Easy to wire when needed.
- **Activity text** is still auto-synthesized from task createdAt/done. The screenshot shows human-written notes ("Norwood · Design Development set sent for review."). Would need a real `activities[]` array on each project. Deferred.
- **iPhone Shortcut for voice-add tasks** — still deferred.

## 2026-05-22 — v1.26: legibility pass + top bar restructure + mobile rebuild
Scott: tighten the project name spacing; everything else (scope, address, lead, tasks, dates, sidebar, chips, fees, status) is too small — bump it. Don't let the layout stretch full-width on a wide monitor. Strip the page-title area (PROJECT BOARD + subline) and the masthead address. Move version info to the bottom of the sidebar. Make the search bar wider and the + New Project button bigger. Rethink the iPhone view from scratch.

### Top bar restructured

**Masthead.** Address removed (`2402-½ Canterbury Lane · Mountain Brook, AL`). Today-date kept on the right. `Carlisle Moore` firm name bumped 18 → 20px; eyebrow 10 → 11px. Padding tightened (12/28/10 → 10/24/8).

**Tab bar.** Tabs bumped 11 → 13px, tab-action 10 → 12px, last-sync 10 → 11px. Padding 6/28 → 5/24. Sticky offset 44 → 42 (the slightly shorter masthead means the tabbar can sit a touch higher).

**Page-title row gutted.** The engraved `PROJECT BOARD` title, the 1px vertical divider, and the italic-serif subline (`9 projects · 6 active · 2 prospects · 1 past due.`) are all removed. The row now contains only the search input and the `+ NEW PROJECT` button. `renderV5TitleBar` still exists but no-ops since the subline element is gone.

**Search bar enlarged.** Was 200px wide / 12px font / 2px vertical padding / 13px icon. Now flexes up to 520px wide / 16px font / 8px vertical padding / 18px icon. The hairline underline thickened 1 → 1.5px so it reads as a real input field.

**+ New Project button enlarged.** 10px → 13px Meta Condensed, padding 7×16 → 12×22. Reads as a primary action now.

**Version label moved.** Was a mono span at the top right of the masthead. Now rendered into a `.v5-sb-footer` at the bottom of the sidebar — the sidebar became a flex column with `margin-top:auto` pushing the footer to the bottom. The bootstrap line that wrote `v…` into the old element is now a no-op (null check handles it); `renderV5Sidebar` re-applies the text after every sidebar render since the element gets replaced.

### Content width capped at 1400px

Wrapped the masthead and tabbar in `.v5-mast-inner` and `.v5-tabbar-inner` (max-width 1400px, centered), and capped `.v5-shell` (sidebar + main) the same way. On a 4K monitor the project list no longer stretches to the edges — the sticky chrome bars span full-width but their contents center.

### Type & spacing bumps (everything was a touch too small)

| Element | Before | After |
|---|---|---|
| **Client name** (`.v5-client`) | 15px / 0.14em | **17px / 0.07em** (tighter) |
| Scope (`.v5-scope`) | 13px italic | **15px** |
| Address (`.v5-addr`) | 10px mono | **12px** |
| Lead (`.v5-lead`) | 10px mono | **12px** |
| Stamp (`.v5-stamp`) | 11px / 0.18em | **13px / 0.16em** |
| Open count (`.v5-cell-num`) | 13px | **15px** |
| Fee amount (`.v5-fee-amt`) | 13px | **15px** |
| Fee billed (`.v5-fee-billed`) | 10px | **12px** |
| Fee pending (`.v5-fee-pending`) | 12px | **13px** |
| Status (`.v5-status`) | 13px italic | **15px italic** |
| Task title (`.v5-task-title`) | 12px | **14px** |
| Task date (`.v5-task-date`) | 10px mono | **12px** |
| Task assignee (`.v5-task-who`) | 10px mono | **12px** |
| Task checkbox | 13×13 | **14×14** |
| More-tasks indicator | 11px italic | **13px italic** |
| Sidebar item (`.v5-sb-item`) | 13px | **14px** |
| Sidebar count (`.v5-sb-count`) | 11px | **12px** |
| Sidebar group label | 9px | **10px** |
| Sidebar tool | 13px | **14px** |
| Strip label (`.v5-strip-label`) | 9px | **11px** |
| Phase / Assigned chip (`.v5-chip`) | 11px | **13px** |
| Chip count (`.v5-chip-count`) | 12px italic | **14px italic** |
| Header row (`.v5-header-row`) | 10px | **12px** |
| Totals row | 10px, num 13px | **12px, num 15px** |
| Tab (`.v5-tab`) | 11px | **13px** |
| Tab meta (last-sync, etc) | 10px | **11px** |
| Tab action | 10px | **12px** |

**Project name spacing tightened** (`.v5-client`): letter-spacing 0.14em → 0.07em. The name reads as a unit now instead of as letters with air between them.

**Row spacing tightened.** Padding 14/24 → 12/24, gap 16 → 14, grid columns rebalanced (`44px 1.6fr 2.1fr 0.55fr 0.95fr 0.75fr 24px` → `42px 1.7fr 2fr 0.5fr 0.95fr 0.75fr 24px`) to give identity slightly more breathing room and pull tasks in a hair.

### Mobile rebuild (`<700px`)

The old `@media(max-width:900px)` was a column reshuffle that hid one column — fine for tablet, useless on phone. Now split into two breakpoints.

**Tablet (`max-width:1000px`)** — sidebar collapses to a horizontal pill bar above the list (chips, no group labels, version footer right-aligned). Fee column hidden. Row grid tightened.

**Phone (`max-width:700px`)** — rebuilt from scratch as a card layout:

- Masthead shrinks: firm name 20 → 16px, eyebrow hidden entirely.
- Tabbar becomes horizontally scrollable; tab-meta hidden.
- Title bar: search and + New Project both stretch to 100% (they stack on narrow widths).
- Header row hidden.
- Each project row becomes a CSS-grid card with three rows:
  1. **Top:** stamp · client name · chevron
  2. **Middle:** task list (assignee hidden on phone, task title allowed to wrap)
  3. **Bottom:** open count · fee · status, all inline as a footer strip
- `.v5-totals` collapses to a single column.
- Expanded detail: 3-col metadata grid becomes 1-col left-aligned; fee-blocks wrap.

The phone view is now a stack of readable cards instead of a squashed table. Type was bumped so nothing falls below ~12px.

### Files touched

`index.html` only.

### Size delta (v1.25 → v1.26)

- Lines: 2,427 → 2,540 (+113)
- Bytes: 107,640 → 111,982 (+4,342)
- Inline JS: 62,253 → 62,678 (+425; mostly the sidebar version-label re-apply + a comment)

CSS grew the most — the new mobile breakpoint alone is ~85 lines.

### Verification

- `node --check` on extracted inline JS (62,678 bytes): clean.
- HTML wrappers in place: `v5-mast-inner`, `v5-tabbar-inner`, `v5-sb-footer` all rendered.
- `v5-page-title`, `v5-subline`, `v5-address` removed from HTML (the `v5-subline` lookup in `renderV5TitleBar` safely returns since the element doesn't exist).

### Force-reload

Append `?v=26` to the URL in Safari after pushing to GitHub.

### Still open (carried from v1.25)

- Address field in the expanded detail still renders `—` — needs project address data, easiest path is `client.projectAddress` from billing.
- Activity text still auto-synthesized.
- iPhone Shortcut for voice-add tasks — still deferred.

## 2026-05-22 — v1.27: +20% fonts, type grouping, masthead gone, favicon, CSS extracted
Scott: bump all fonts 20%; enlarge the version/file-update text at the bottom; group jobs by type (New Build first, then Renovation, In Construction, Prospect) with subtle background differentiation; drop the Carlisle Moore masthead; make a PB favicon in the color scheme; clean up the codebase and extract CSS to its own file.

### Top chrome: masthead gone

The `.v5-masthead` block (Carlisle Moore + Residential Architecture eyebrow + today-date) is removed entirely. The tabbar is now the only top bar, sticky at `top:0`. Today-date moved into the sidebar footer alongside the version label. The HTML wrapper `#main-masthead` is preserved so the existing `showPage()` toggle still works; it just contains only the tabbar now.

Net effect: the page reads about 50px shorter at the top, more vertical space for the project list, and the firm-identity engraving (which was decorative) is gone. If we ever want to bring it back as a print/PDF header it's easy to drop in.

### Fonts +20% across the board

Every V5 font-size was multiplied by ~1.2 and rounded to clean values. The full table is in the diff, but the headline numbers:

| Element | v1.26 | v1.27 |
|---|---|---|
| Client name | 17px | **20px** |
| Scope | 15px italic | **18px** |
| Stamp | 13px | **16px** |
| Task title | 14px | **17px** |
| Open / Fee / Status | 15px | **18px** |
| Sidebar item | 14px | **17px** |
| Sidebar count | 12px | **14px** |
| Sidebar tool | 14px | **17px** |
| Section/group label | 10px | **12px** |
| Tab | 13px | **16px** |
| Tab action | 12px | **14px** |
| Chip | 13px | **16px** |
| Chip count | 14px italic | **17px italic** |
| Search input | 16px | **19px** |
| + New Project button | 13px | **16px** |
| Header row | 12px | **14px** |
| Activity row | 11/12px | **13/14px** |
| Modal title / field / buttons | 14/9/10px | **17/11/12px** |

**Sidebar footer** got more than 20%: today-date 11→13, version label 10→14. Both legible at a glance from across a desk.

Grid column widths bumped slightly (`42px 1.7fr 2fr 0.5fr 0.95fr 0.75fr 24px` → `46px 1.7fr 2fr 0.5fr 0.95fr 0.75fr 28px`) to absorb the bigger stamp and chevron.

Mobile breakpoints also bumped proportionally — phone task titles 14 → 16, client name 16 → 18, etc.

### Grouping by type with tinted backgrounds

`renderV5List` now buckets projects by `p.type` and renders each group with an engraved section header and a tinted wrapper. Order is **New Construction · Renovation · In Construction · Prospect · Unassigned** (matching `V5_TYPE_ORDER`).

Section header: Columbia Titling 17px 0.22em caps + italic-serif count on the right, on a paper-0 background with a 1px engraved-rule under it. Looks like a newspaper section head.

Per-type tints (all very pale, ~3% saturation, intentionally subtle):
- `.v5-section.type-new-construction` — **#faf3df** (warm wheat)
- `.v5-section.type-renovation` — **#eef0f5** (cool slate)
- `.v5-section.type-in-construction` — **#eaf1ea** (pale moss)
- `.v5-section.type-prospect` — **#f1eadf** (taupe)
- `.v5-section.type-unassigned` — transparent

The expanded row's forest-4 tint still overrides per-section tints via `.v5-row.expanded` specificity, so clicking a row to expand it still pops correctly.

New constants: `V5_TYPE_ORDER` and `V5_TYPE_NAMES` near the renderer (intentionally local — they're the rendering shape, not the data shape, which is still `BOARD_PHASES`/`PHASE_LABELS` in the constants block).

### Favicon — engraved PB

Inline SVG `data:` URI: `f7f3ea` paper background, ink-dark Georgia serif "PB" centered, with a 1.6px forest-green (#2f4b2a) underline beneath. Matches the active-tab underline treatment, so the tab icon visually rhymes with the in-app interaction. No extra file, no PNG/ICO to maintain.

### In-file cleanup pass (the "no-regret" reorg)

- **Big TOC comment at the top of the file** — section map with line markers + edit conventions (full-file replacements, run node --check, bump APP_VERSION + CSS ?v= together, append session-notes). Future-me opens index.html and sees the layout immediately.
- **Section banners through the JS** — `§12 CONSTANTS`, `§13 STATE`, `§14 GITHUB SYNC`, `§15 PAGE NAVIGATION`, `§16 V5 RENDER`, `§17 MODALS · EDIT POPUP · CALENDAR`, `§18 LEGACY PAGE RENDERERS`, `§19 CALENDAR POPUP`, `§20 INIT`. Each is a multi-line comment band so it's visible while scrolling. The numbering matches the TOC.
- **Section banners through the CSS** — `V5 TAB BAR`, `V5 BOARD SHELL`, `V5 TITLE BAR`, `V5 STRIPS`, `V5 PROJECT LIST`, `V5 NEW PROJECT MODAL`, `V5 RESPONSIVE`.
- **Constants stay grouped** at the top of the script block (~line 1466–1610 in v1.27): GH targets, TOOL_LINKS, V5_PHASES, V5_STAFF, PALETTE, TYPE_LABELS, STAFF_MEMBERS, DEFAULT_STAFF, PHASE_COLORS, BOARD_PHASES, PHASE_LABELS, SAMPLE_PROJECTS, LEAD_NAMES. SAMPLE_PROJECTS is the only data-heavy one and lives in the same block; not worth pulling to a separate file.

### CSS extracted to cma-board.css

The entire inline `<style>` block (~41 KB) is now its own file. `index.html` loads it via `<link rel="stylesheet" href="cma-board.css?v=27">` just under the Typekit link. **Both files need to be pushed to GitHub** for the new version to work.

**Cache-buster discipline**: the `?v=NN` on the CSS link must match `APP_VERSION` in the JS. A comment marker right above `const APP_VERSION` calls this out. Bump them together; if you forget, Safari will serve the old CSS against the new JS and the page will look broken.

Why CSS only (not JS too)? Per the earlier discussion: CSS extraction is the highest-value split — independent of the JS, makes navigation easier, halves the file you're scrolling through. Extracting JS would add fragility (load order, two cache-busters, two pushes for every JS change) without much readability win.

### Size delta (v1.26 → v1.27)

- `index.html`: 111,982 → **82,794 bytes** (-29 KB; CSS lifted out, ~+5 KB of TOC and banners added)
- `cma-board.css`: **43,747 bytes** (new file)
- Combined: 111,982 → **126,541 bytes** (+14.5 KB; the bumps + section grouping + cleanup banners net out at +13%)
- Inline JS: 62,678 → **66,650 bytes** (+4 KB; V5_TYPE_ORDER + renderV5List rewrite + section banners)
- Lines: index 2,540 → **1,387**; cma-board.css **1,289**

### Verification

- `node --check` on extracted inline script (66,650 bytes): clean.
- Hashes match between `outputs/` and `TSC-BOARD/` for both files (md5sum confirmed).
- Spot-check: tabbar sticky at top:0 (no masthead above), sidebar footer renders date + version, section headers render before each non-empty type group, type-tints applied via wrapping `.v5-section` class.

### Push instructions for Scott

Push **two files** to GitHub: `index.html` AND `cma-board.css`. Then force-reload Safari with `?v=27`. (Both files need to be pushed in the same commit, or the page will load the new HTML against the old CSS for one cycle.)

### Still open

- Address field in the expanded detail still renders `—` — needs project address data; easiest path is `client.projectAddress` from billing.
- Activity text still auto-synthesized.
- iPhone Shortcut for voice-add tasks — still deferred.

## 2026-05-22 — v1.28: billing schema fix (Fee column actually populates now)
Scott asked what was needed to tie the board to the billing PROJECT. Confirmed the integration plumbing was already in place (v1.24 wired the fetch + join), but two field-name mismatches between `buildBillingMap` and the live TSC-BILLING data were silently making every join resolve to `fee = 0` even when the names matched. Patched.

### What was wrong
- `buildBillingMap` read `c.currentFee` (camelCase, top-level). TSC-BILLING's actual shape nests it: `c.fee_basis.current_fee` (snake_case). Result: `fee` was always `0` across all projects, regardless of whether the name matched.
- It also looked up `c.fullName` for one of the name-lookup variants. TSC-BILLING uses `c.full_name`. Lookups by short name (`c.name`) still worked, so name-matched rows did join — they just showed `Fee: $0` and a 0%-billed progress bar.

### The fix (one function, `buildBillingMap`)
- Read `fee` from `c.fee_basis.current_fee` first, falling back to `c.fee_basis.starting_fee`, then to the older flat-field variants. Also accept `feeBasis` (camelCase nesting) for completeness.
- Expanded the name-lookup list to include the snake_case variants: `c.short_name`, `c.full_name`, `c.client_name`, `c.project_name` (alongside the existing camelCase keys).
- No change to the invoice math — TSC-BILLING already uses flat `i.sent`, `i.paid`, `i.amount` which the existing reducer reads correctly.

### What you should see after reloading
Confirmed by cross-referencing the live `tscarlisle-ghub.github.io/TSC-BILLING/data.json` against the current `data.json` (26 active board projects vs 35 active billing clients). These board projects name-match billing today and will populate Fee/Billed/Remaining + the progress bar immediately:

- Casadaban - Magnolia ($22,000)
- Casadaban - Payne Street ($0 — billing record exists, no fee set)
- Clikas ($7,500)
- Davis - Liberty Park ($400,000)
- Davis Winfield → Davis - Winfield ($420,000)
- Fox ($25,000)
- LaRussa ($8,000)
- Lepere → LePere ($169,482)
- Ouellette ($50,000)
- Visintainer ($50,000)
- Young ($125,000)

The remaining ~15 board projects (Ford - Amber & Adam, Weil Poolhouse, Russell Lands, Sanders Jim & Betsy, Taylor Chris & Paget, Robinson Brandon & Brandi, Ray Webster & clay, Callahan Barn, Callahan Residence & Poolhouse, Atkins Residence (Tuscaloosa), DCS Montevallo, Casadaban - Lot 18, Davis Collins and Erin, Matt Demo, Mid Century Reno) won't match until either the board name is shortened to match the billing `name` field, or we add a per-project `billingKey` override field. That's the deferred step 2.

### Files touched
`index.html` only. `cma-board.css` unchanged.

### Verification
- Extracted inline `<script>` (67,092 bytes), `node --check` clean.
- Bumped `APP_VERSION` 1.27 → 1.28 and CSS link `?v=27` → `?v=28` (CSS file itself unchanged, but the cachebuster keeps Safari from serving stale).

### Force-reload
Append `?v=28` to the URL in Safari after pushing the new `index.html` to GitHub.

### Still open
- **`billingKey` override field** on projects so the unmatched names can be wired without renaming board entries. Two-line code change + a small input in the edit popup. Deferred until you see the v1.28 board and decide which path you want.
- Address field still renders `—`; carryover.
- Activity text still auto-synthesized; carryover.
- iPhone Shortcut for voice-add tasks; carryover.

## 2026-06-02 — v1.29: design-system mockup applied to live board

Took the iterative `design-mockup.html` we'd been refining in-chat and rolled the decisions into the live board (`index.html` + `cma-board.css`). Pushed the cache-buster from `?v=28` → `?v=29` and bumped `APP_VERSION` 1.28 → 1.29.

### Type changes (cma-board.css)

- **Client name (`.v5-client`)** — switched from Columbia Titling 20px / 0.07em tracking to **Interstate Bold Compressed 35px / 0.01em**. Stack: `"Interstate-BoldCompressed","interstate-compressed","Interstate Compressed",Arial`. Line-height collapsed to 1 with 4px margin-bottom so the row doesn't gain too much vertical air.
- **Section heads (`.v5-section-header`)** — now **BioRhyme Expanded ExtraBold** (weight 800), up from 400. Stack: `"BioRhyme Expanded", var(--ff-display)`.
- **Sidebar items (`.v5-sb-item`)** — **Whitney Condensed Light** (weight 300, .04em tracking). Active item only goes to weight 400 (was 600). Stack: `"Whitney Condensed","Whitney", var(--ff-sans-cond)`.
- **Task titles (`.v5-task-title`)** — **Whitney Light** (weight 300). Stack: `"Whitney", var(--ff-sans)`.
- **Activity row text (`.v5-activity-row .a-text`)** — also Whitney Light 300.
- **New: `.v5-strap`** — under each client name, the assigned staffer renders in **Whitney Book caps**, 13px, 0.12em tracking, `--cma-accent-soft` (#c47a48). Replaces the old "Lead · TSC" line.

### Layout changes (index.html)

- **`v5TaskInline`**: dropped the `.v5-task-who` assignee column from each task row. Tasks are now checkbox · date · title only. CSS grid `.v5-task-line` was `20px auto 1fr auto`; now `20px auto 1fr`.
- **`v5ProjectRow`**: removed the `.v5-scope` line (which rendered `TYPE_LABELS[p.type]` — redundant with the section heading) and the `.v5-lead` "Lead · TSC" line. Replaced with a single `.v5-strap` showing `p.staff || p.lead || DEFAULT_STAFF`.
- **`v5ExpandedDetail`**: removed the top three-up `.v5-expand-grid` block (Phase / Lead / Type). The expanded panel now opens straight to the fee section / activity timeline / action buttons.

### Font loading (index.html)

The live app was loading only Typekit (`ikf0hkb.css`) which doesn't carry Whitney, BioRhyme Expanded, Penumbra Flare, or Interstate Mono. Added a `<link>` to the firm's hosted font sheet:

```html
<link rel="stylesheet" href="https://tscarlisle-ghub.github.io/fonts/tsc-fonts.css">
<link rel="stylesheet" href="https://use.typekit.net/ikf0hkb.css">
```

For belt-and-suspenders, bundled the local woff2 files we have to `TSC-BOARD/fonts/` and declared `@font-face` blocks for them at the top of `cma-board.css`:

- `fonts/BioRhymeExpanded-ExtraBold.woff2`
- `fonts/Whitney-Light.woff2`
- `fonts/Whitney-Book.woff2`
- `fonts/Whitney-BookItalic.woff2`
- `fonts/Whitney-Bold.woff2`

So section heads and Whitney type render correctly even if the hosted sheet fails to load. Interstate-BoldCompressed comes from Typekit (`interstate-compressed` at weight 700).

### New token

Added `--cma-accent-soft: #c47a48` to the V5 token block (alongside `--cma-accent` and `--cma-accent-tint`).

### Files touched

- `index.html` — font sheet link, APP_VERSION bump, CSS cache-buster bump, `v5TaskInline`, `v5ProjectRow`, `v5ExpandedDetail`.
- `cma-board.css` — @font-face block, `--cma-accent-soft` var, `.v5-sb-item`, `.v5-section-header`, `.v5-client`, **new `.v5-strap`**, `.v5-task-line`, `.v5-task-title`, `.v5-activity-row .a-text`.
- `fonts/` (new directory) — 5 woff2 files for local fallback.

### Verification

- `node --check` on extracted inline `<script>` (66,552 bytes): clean.
- md5sums match between `outputs/` and `TSC-BOARD/` for both files.
- Sizes: `index.html` 82,780 bytes; `cma-board.css` 45,306 bytes.

### Push instructions

Push **three things** to GitHub in the same commit:
1. `index.html`
2. `cma-board.css`
3. `fonts/` folder (all 5 woff2 files)

Then force-reload Safari with `?v=29`.

### Still open (deferred carryovers)

- `billingKey` override field for unmatched billing names (v1.28 carryover).
- Address field still rendered `—` in the old expanded panel; that whole panel is gone now, so this is moot until we decide to surface address elsewhere.
- Activity text still auto-synthesized.
- iPhone Shortcut for voice-add tasks.

## 2026-06-03 — v1.30: strip phase + fee columns, assigned into sidebar, cyan client names
Scott: take off the phase strip, take off the fee·billed column, drop the "Office" header text on the sidebar (keep its items), put the Assigned filter in the sidebar, and widen the client-name column with cyan caps that fit on one line.

### What changed
- **Phase strip removed.** `#v5-phase-strip` div gone from the HTML; `renderV5Strips()` retired (replaced with a comment marker).
- **Assigned strip removed from the top of the list** — moved into the left sidebar as its own group under the existing Office filters.
- **Sidebar restructured.** "Office" group label removed (the items All projects / Active / Prospects / On hold / Past due / Complete still render, just with no header). A new **Assigned** group sits below them with All / Maddie / Kathleen / Kat — wired to the same `v5SetStaffChip()` action the old chips used, so the staff filter still drives `v5State.staff` and downstream filtering identically.
- **Fee · Billed column removed** from the row body, header, and totals. The grid template went from 7 cols (`46px 1.7fr 2fr 0.5fr 0.95fr 0.75fr 28px`) to 6 cols (`46px 2.8fr 1.6fr 0.5fr 0.75fr 28px`) — identity claims the freed-up space. `v5HeaderRow()`, `v5ProjectRow()`, `renderV5Totals()` all updated to match.
- **Client name column widened + cyan caps.** `.v5-client` font dropped 35→30px, color switched from `var(--v5-engraved)` to `#0891b2` (deep cyan), added `white-space:nowrap;overflow:hidden;text-overflow:ellipsis;` so each name sits on one line. With identity at 2.8fr (~48% of the list width) the longest current names ("Callahan Residence & Poolhouse", "Atkins Residence (Tuscaloosa)") fit without truncation. Phone breakpoint relaxes to `white-space:normal` so the card layout can still wrap.
- **Mobile breakpoints rebalanced.** Tablet grid trimmed to 6 cols. Phone card layout's `nth-child` selectors renumbered (5 = footer, 6 = chev) since the fee column went away. Tablet's "hide Fee · billed" rule removed (no longer needed).
- **Identity wrapper** gets a `.v5-identity` class with `min-width:0` so ellipsis works inside a CSS grid track.

### Files touched
- `index.html` — strip removal, `renderV5Sidebar` rewrite (Assigned group added, Office label dropped), `renderV5Strips` deleted, `v5HeaderRow` / `v5ProjectRow` / `renderV5Totals` shortened, APP_VERSION + CSS cache-buster bumped.
- `cma-board.css` — `.v5-row-grid` + `.v5-totals` grid template (6 cols), `.v5-client` cyan/nowrap, mobile breakpoints updated, `.v5-identity{min-width:0;}` helper added.

### Verification
- Extracted inline `<script>` (~65 KB), `node --check`: clean.
- `grep renderV5Strips`: 0 hits in real code (only the comment marker).
- Sidebar: 6 Office items render with no header, then an "ASSIGNED" group with 4 items (All + 3 staff).

### Push
Push **both** `index.html` and `cma-board.css` to GitHub in the same commit. Force-reload Safari with `?v=30`.

### Still open (carryovers)
- `billingKey` override field for unmatched billing names (v1.28 carryover).
- Activity text still auto-synthesized.
- iPhone Shortcut for voice-add tasks.
- Per-row billing fee is no longer surfaced on the list; the expanded detail still shows Fee/Billed/Remaining + the progress bar when billing data is matched.

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

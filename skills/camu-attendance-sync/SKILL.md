---
name: camu-attendance-sync
description: Read today's (or a chosen past day's) class attendance counts from camu.in's Schedule page using an already-logged-in browser session, and record them into a local weekly Excel grid (Section x Day = present/total). Use when the user asks to "sync attendance", "update camu attendance", or similar, and mentions camu.in.
---

# camu.in Attendance Sync

Turns the manual "open camu.in, click through each class, note the present/total,
copy into a spreadsheet" chore into something you just ask Claude to do, using
the Claude-in-Chrome (or equivalent) browser tool plus the bundled
`scripts/build_attendance_grid.py`.

**Requires:** Claude Code (or an equivalent agent runtime) with a browser-automation
tool connected to the user's real, already-authenticated browser session (e.g. the
Claude-in-Chrome extension). This skill assumes such a tool is available under
whatever name your runtime exposes it as — adjust tool names in the steps below
to match.

## Hard rule: never touch credentials

This skill **only reads an already-logged-in session**. It must never type a
username or password into camu.in, never accept "helpfully" logging the user in,
and must never store or transmit credentials anywhere. If you land on a camu.in
login page during this workflow, **stop and tell the user to log in themselves**,
then resume once they confirm they're logged in.

## Step 0 — Confirm the browser session

1. Check the browser tool is connected (list tabs / get context). If not connected,
   tell the user how to enable it and wait.
2. Confirm there's a tab already on the institution's camu.in domain, logged in
   (the top-right should show the user's name, not a login form). If not, ask the
   user to log in themselves in their own browser first — do not proceed otherwise.

## Step 1 — Read the day's schedule

1. Navigate the tab to `#/staffhome` (camu.in's landing/dashboard route).
2. The "Schedule" widget shows the currently-selected date and that day's classes.
   To view a different (past) day, click the left-arrow next to the date — note
   that **date state resets to "today" every time you navigate back to `#/staffhome`**,
   so each day you want to inspect requires re-clicking the left-arrow the right
   number of times from today.
3. Use the browser tool's accessibility-tree / page-read function (not just a
   screenshot) targeting the schedule widget — this returns every class listed for
   that day in one call (course name, time slot, department/degree, semester,
   section), even ones below the visible viewport. **Do not rely on a screenshot
   alone for this step** — it's easy to miss classes that require scrolling.
4. Record the full list for that day before doing anything else: you'll need it to
   know how many classes to visit.

## Step 2 — Read each class's attendance count

For each class found in Step 1:

1. Click that class's title/row to select it (this is what makes its "Attendance"
   button appear/become active — there is typically only one such button in the
   DOM at a time, tied to whichever class was last selected).
2. Click the "Attendance" button. This opens that specific class session's roster.
3. Read the summary bar at the top — it shows something like `45/60` and a
   percentage (e.g. "75% attendance"), plus the date and section/course it belongs
   to. A screenshot is the most reliable way to read this bar; a page-text read
   may not reliably surface it depending on how the page renders.
4. Note: `present`, `total`, `section`, `course/semester label`, `date`.
5. Navigate back to `#/staffhome` (a fresh navigation, not just browser "back" —
   the date-state resets either way, so treat it as a checkpoint) and re-click the
   left-arrow to return to the target date before visiting the next class.

Repeat until every class from Step 1 has a recorded count (or is confirmed as "no
data available" if camu.in shows the session as not yet finalized).

## Step 3 — Build/update the Excel grid

1. Group the day's classes however makes sense for the user's institution — most
   commonly by semester/year, with one sheet per group. Ask the user if unsure how
   they want classes grouped (by semester? by program? by your own course load?).
2. Within a group, rows are sections (e.g. A1, A2, A3...) and columns are weekdays.
   Use these cell values:
   - `[present, total]` — a confirmed count for that day
   - `"NA"` — confirmed no class was scheduled that section that day (only write
     this if you actually saw the full day's schedule and it wasn't there)
   - `"?"` — a class was scheduled/held that day but you haven't read its count yet
   - `""` (empty) — that day hasn't been checked at all
3. Write a JSON config matching `examples/sample_config.json`'s shape, with the
   real data you collected, and an `output_path` pointing at wherever the user
   wants their workbook kept (ask them once, then remember it for next time).
4. Run:
   ```
   python scripts/build_attendance_grid.py --config path/to/config.json
   ```
   This creates the workbook if it doesn't exist, or updates just the sheets named
   in your config if it does — sheets for other weeks are left untouched.
5. Tell the user what was written (which sheets, which cells changed) so they can
   sanity-check it, and mention any `?`/blank cells still outstanding.

## Adapting this skill to a different institution

camu.in is used (white-labeled) by many different colleges — the exact wording of
labels, the semester/section naming convention, and even minor DOM differences can
vary by institution. Before relying on this skill at a new institution:

- Walk through Steps 0-2 once manually (with the user watching) to confirm the
  Schedule widget, class-selection click, and "Attendance" button/summary-bar all
  behave as described above. Adjust the instructions above if the institution's
  camu.in instance differs.
- Confirm how that institution's teachers actually want classes grouped into
  sheets (semester? department? something else) — don't assume the semester-based
  grouping from `examples/sample_config.json`.
- Confirm the section/label vocabulary (e.g. some institutions may not use "A1,
  A2..." naming) and adjust `sections` accordingly.

## Privacy

Nothing in this skill stores, logs, or transmits the user's camu.in credentials.
It only reads pages the user is already authenticated to view in their own
browser, and writes to a local file on their own machine.

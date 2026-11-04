# SCULPT v3.4.0: Profile (InBody) and small touches

## Intent
The user wants to store InBody scan results in the app and track them over time, and wants the app to feel more complete without becoming complex. Single-file PWA (index.html), used on a Galaxy S24 Ultra at the gym. All data in localStorage under key `sculpt_v3` (object `S`), exported/imported as JSON.

## Goals
- Profile sheet from Menu with InBody history and trends.
- Three small touches that make logged data more useful: workout duration, new-best toast, backup reminder.

## Non-goals / anti-goals
- No accounts, sync, or network calls.
- No new screens outside Menu except the Profile sheet.
- No long text. UI strings 1 to 8 words. Same visual language (lime on dark, Space Grotesk headings).
- No new libraries.

## Conventions already in the app
- `S` state object, `save()`, `openModal(id)`, `switchModal(id)`, `closeAll()`, `toast(msg, red)`, `buzz(pattern)`, `$()`.
- Sheets are `<div class="modal" id="mX"><div class="box">...</div></div>`, with `.grab` and `.xbtn[data-close]` at the top.
- Menu rows: `<button class="mrow" data-menu="...">`. Click handler at the end of the script (`$('#mMenu').addEventListener`).
- Tiles: `.hero > .tile`, sparkline: `spark(points, invert)`. Rows: `.row` / `.row.tap`. Labels: `.lbl`, `.kicker`, `.note`.
- Dates are `YYYY-MM-DD` strings via `ds()` / `todayStr()`.
- `fresh()` lists all state fields; new fields must be added there so import of old backups fills defaults.

## Feature 1: Profile (builder A)
Data:
- `S.profile = { h: 0 }` height in cm (0 = unset).
- `S.body = []` entries `{ d: 'YYYY-MM-DD', w: 62.3, m: 24.1, f: 28.5, s: 74 }`. `w` weight kg (required), `m` skeletal muscle mass kg, `f` body fat %, `s` InBody score. Optional fields stored as `null`. Keep sorted by `d` ascending; one entry per date (same date replaces).

UI:
- Menu row "Profile" with small text "InBody and body trend" above the Backup section (placeholder already in index.html as `data-menu="profile"`). Opens sheet `mProfile` (empty container already in index.html) via `switchModal`.
- Sheet content, rendered by `renderProfile()`:
  - `.grab`, `.xbtn`, `<h2>Profile</h2>`.
  - Height row: label "Height" with a number input (cm). Saved on change. When set and an entry exists, BMI is shown in the latest tile line.
  - Three tiles (`.hero`): Weight kg, Muscle kg, Fat %. Value = latest entry; delta line = change vs the previous entry (e.g. "▲ 0.8 kg vs last", lime when muscle up or fat down, red when fat up or muscle down; weight change is neutral colour). If no entry: "—".
  - One sparkline-style chart per tile is too much; instead show one `.chart` block with a 3-line SVG trend (weight, muscle, fat %) over the last 12 entries is over-scope. Keep: each tile gets the existing `spark()` under its value (muscle and weight normal, fat `invert=true` so down is up).
  - Button `.btn.btn-p` "Add InBody" opens the entry form (inside the same sheet, replacing the list, with "Save" and "Cancel"). Form fields: Date (type=date, default today), Weight kg, Muscle kg, Fat %, Score. Inputs `type=number inputmode=decimal`, min-height 48px, dark style matching `.opt`.
  - List of entries newest first as `.row.tap`: date, "62.3 kg · 24.1 m · 28.5 %", tap to edit (same form, prefilled) with "Delete" button (`.btn.btn-r`, confirm()).
  - Empty state `.empty`: "Add your first InBody scan."
- Validation: weight required and 20 to 300; others optional; fat 0 to 70; muscle 0 to 100; score 0 to 100. Bad input: toast "Check the numbers" red.
- All inputs must allow text selection and typing (the global `user-select: none` is fine for inputs, but make sure `input` has `user-select: text` and font-size 16px to avoid zoom).

Also:
- Add one line to the How it works sheet step 4 (Review): keep it as is; instead add a `.note` under the Profile tiles: "Scan every 4 weeks, same time of day." (one line only).

## Feature 2: Workout duration (builder B)
- `S.start = {}` map dayId -> timestamp ms. Set when the first set of that day is marked done (or first class item checked) if not set.
- `logSession()` stores `min: Math.round((Date.now() - S.start[day.id]) / 60000)` when start exists (cap at 240; ignore if < 1), then deletes `S.start[day.id]`. `resetDay()` also deletes it.
- Finish sheet `.kv` grid gains a third cell "Time" with `${min} min` when known (make `.kv` 3 columns when 3 cells; otherwise unchanged).
- Month list row small text: `${date} · ${sets} sets · ${min} min` when `min` exists.

## Feature 3: New best (builder B)
- When a set is marked done (the `.ok` handler, non-noW, non-assist exercise), compare `s.w` with the max weight in `S.history[c.hk]` (all entries, all sets). If strictly greater and history has at least one entry: `toast('New best · ' + fmtW(s.w) + ' kg')` instead of nothing (do not change the existing timer flow; show toast after `startTimer`/`showRate`). Mark `s.pb = true`.
- `logSession()` collects names of exercises with any `pb` set and stores `pbs: ['Leg Press', ...]` on the session (omit key if none).
- Finish sheet: when today's done sets include `pb`, show a `.note` line under the kv grid: "New best: Leg Press 45 kg, Hip Thrust 30 kg" (lime text). Keep to one line, truncate list to 3 names + "+N".

## Feature 4: Backup reminder (builder B)
- `S.lastExport = ''` date string. `exportData()` sets it to `todayStr()` and saves.
- Menu backup note (element `#backupNote`, already in index.html) text: "Last export: never" / "Last export: today" / "Last export: N days ago". Add class `red` (colour `var(--red)`) when never or > 30 days. Update the text each time the Menu opens (in the `#btnMenu` click handler) and after export.

## Builder rules
- Edit only index.html. Do not bump APP_VER or sw.js VERSION (the integrator does).
- Keep the existing code style: one-line helpers, template strings, no frameworks.
- Test in headless Chrome with a small Python script (python at `D:/Miniconda3/envs/gym-tracking/python.exe`, Chrome at `C:\Program Files\Google\Chrome\Application\chrome.exe`) or at least load the page with `--dump-dom` and check the console has no errors. Add 2 to 3 fake entries via `localStorage` to screenshot the Profile sheet at 412x915.

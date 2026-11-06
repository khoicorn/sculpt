# v3.8.0 review fixes (PT check + wording review, 2026-11-06)

These override the main spec (`2026-11-06-intensity-and-mat-band-design.md`) where they differ.

## Safety and logic (PT check)
1. Pump rules. Rule 2 stays `Lightest plates. Lunges: no bar.` Rule 3 becomes `Half squats, exhale up. Skip crunches.` Light rule 2 stays `No plates on squats or lunges.` Rules sheet "Lifting" line `No barbell squat. No heavy deadlift.` becomes `Gym: no barbell squat, no heavy deadlift.` (Pump's light bar is the only exception, and its rules say so.)
2. `hardBlock()`: after a 10+ day break (`suggestMode().why === 'After a break'`) return `Back after a break. Not today.`
3. `#feel` handler: `Easy` raises kg only in Normal. On Light and Hard, Easy keeps kg. Hard rating still lowers 1 step and adds 30 s in every mode.
4. `cardClass()`: when a main class is swapped (by hand or by Light), the card still shows the class's red rule 1 (`Dizzy, chest tight, heart racing = stop.`) first, then the alt's cue lines. Same for Walk → Easy Run (its own rules already start with it).
5. Easy Run cue: `Walk 5 min. Run 1, walk 2, x5. Walk 5.` (= 25 min). Rule 4: `After 2 easy runs: run 1 min longer.`
6. `runBlock()` also blocks when today is Light (`Light today. Walk only.`) or a Hard session was logged today (`Hard today. Walk only.`).
7. Mat day: the Band RDL swap of `mat_kick` never gets the Hard +1 set (no added sets on hinges). `mat_press` cue 1 becomes `Stand up slowly. Band across upper back.`
8. Mat day Hard keeps 90 s rest (`keepRest` applies to Hard too).
9. 21-day rule also for `noW` moves with `repTop`: after 21+ days since the last non-Light entry, target = `max(c.reps, last target − one step)`.

## Wording and clutter
10. Walk tab: no add rows; main button `Done` (not `Class done`); done line `✓ Done` for non-class items (`icon !== 'cls'`).
11. Light hint under the picker: `Easier today. Progress kept.`
12. "Light" names one thing only (the mode):
    - `Hip Thrust Machine · Light` → `Hip Thrust Machine · Finisher`; `Abductor Machine · Light` → `Abductor Machine · Finisher`. Other `· Light` exercise names → `· Easy`.
    - Add sheet `Light only today.` → `Class day. Easy moves only.`
    - `No light moves today` (hint and empty state) → `No easy moves today`.
    - Yoga rule `Light extras only.` → `Easy extras only.`
13. Mat day swap rows: `Swap to <b>X</b>` with no question (`swapAsk` empty on home days).
14. Guide on home days (Mat + Band): hide the "My gym has no …" button.
15. Set rows of band moves: no `weight` label under `Band`. `Body` rows unchanged.
16. Cool-down rules: `Leave when you can chat.` / `Stop when you can chat.`
17. 1-arm pulldown: name `1-Arm Cable Pulldown`; what `Half kneel. Pull one handle to your side.`; cue 2 `Same-side knee down, arm reaching up.`
18. Set rating button `Hard` → `Too hard`. Rules line → `After each set tap Easy, Good or Too hard.` Any text that names the rating follows.
19. Combat swapped (Light or by hand): hide `Class late? Exercises first`. Alt cues drop the minutes the card already shows: Easy Elliptical `Can chat. Or join Yoga.`; Easy Bike `Easy pace.`
20. When `maxExtra(day) === 0`, hide the add rows (no dead `Light day. No extras.` row). The addExtra toast stays as a guard.
21. Stats workout list: a logged Easy Run is titled `Easy Run`; hide `· 0 sets` when sets are 0.
22. Rules sheet: move `Extras: Light 0, Normal 2, Hard 1.` into the Intensity section; heading `Intensity` → `How hard`; `Hard max 2x a week.` → `Hard max 2x per week.`; Safety line 6 → `Dizzy, chest tight, heart racing = stop.`
23. Cues: `bandbridge` `Heels close, feet hip width.` → `Heels near hips, feet hip width.`; `bandpress` `Small split stance. Elbows at 45°.` → `One foot forward. Elbows at 45°.`; name `Prone Y Raise` → `Face-Down Y Raise` (hk stays `proney`).
24. Finish tiles CSS: `.kv div` → `.kv > div` so label and value are not boxed separately.
25. Stats "Classes" counts (line and week view) leave out Walk.

Not taken: removing the Guide `≈ N kcal / set` line (the user asked for per-set kcal).

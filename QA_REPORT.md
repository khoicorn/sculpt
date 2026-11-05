# QA Report · SCULPT v3.7.0 · 2026-11-05

## Verdict: PASS

No Blocker or Major issue. 6 small notes below (2 Minor, 3 Cosmetic, 1 Info). v3.7.0 can ship as is. N1 and N2 are worth a quick 3.7.1.

## Re-test after fixes (same day)

All 6 notes were fixed in v3.7.0 before deploy, with the exact fixes in the table below. N6: the "Added today" row now has a transparent border.
Re-run after the fixes, all 0 fails and 0 console errors: m37 52, m37 rm 53, qa37 q1 37, q2 8, q3 45, q7 24, t11 57, t12 11, qa361 p1_func 84. q6 now shows no outline on focused Profile fields.

## Summary

- All 12 suite runs pass with the expected counts. 0 fails, 0 console errors. Width scan is CLEAN at 360, 384 and 412 px.
- Known old failures (t9, qa36h p5, p6, p7, p11, p13) give the same FAIL lines on the v3.6.1 backup. Not regressions.
- New probes in `qa37\` (timer, beep, reload, sheet guard, swipe, press, fast taps, no replay, hand flag, "Added today", reduced motion, widths): 205 checks, all pass.
- Safety is unchanged. A text diff of v3.6.1 vs v3.7.0 finds 1 new UI string ("Added today") and no other change. HR gates, 90 s rest minimum, class HR cap and "Max safe load" caps have the same logic and text.
- Haptics: 59 `buzz(` calls, all valid names, no number patterns left.
- Version: `APP_VER` 3.7.0 = sw.js `sculpt-v3.7.0`.

## Automated suites

Run one at a time, each with `timeout 170`. Output: `qa37\suites\` and `qa37\run_all.log`.

| Suite | Checks | Fail | Console errors |
|---|---|---|---|
| t6 | 33 | 0 | 0 |
| t7 | 7 | 0 | 0 |
| t8 | 37 | 0 | 0 |
| qa_width | CLEAN × 3 (360 / 384 / 412) | 0 | 0 |
| t10 | 34 | 0 | 0 |
| t11 | 57 | 0 | 0 |
| t12 | 11 | 0 | 0 |
| t13 | 11 | 0 | 0 |
| qa361 p1_func | 84 | 0 | 0 |
| qa361 p2_width | 17 screens × 3 widths, 0 errors | 0 | 0 |
| ux37 m37 | 52 | 0 | 0 |
| ux37 m37 rm (forced reduced motion) | 53 | 0 | 0 |
| qa37 q1_timer (new) | 37, plus 33 in reduced motion | 0 | 0 |
| qa37 q2_resume (new) | 8 | 0 | 0 |
| qa37 q3_input (new) | 45, plus 46 in reduced motion | 0 | 0 |
| qa37 q7_width (new, iframe 360 / 384 / 412) | 12 × 3 | 0 | 0 |
| Known old fails, v3.7.0 vs v3.6.1 backup | t9 2, p5 2, p6 3, p7 1, p11 1, p13 1 | same on both | 0 |

Folder: `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\97fa529f-458e-437d-a894-6259a8e77f66\scratchpad\qa37\`.

## Findings

| # | Severity | Where | Steps | Expected | Actual | Fix |
|---|---|---|---|---|---|---|
| N1 | Minor | `index.html` line 2452, `inWorkout` | Tap HR, log the session, leave the app, come back on a day tab | Screen can sleep again, as in v3.6.1 | Wake lock is taken again, because "HR tapped today" counts as a workout. The screen stays on while the app is open, all day. q1_timer: 1 lock request after log | `const inWorkout = () => S.view !== 'stats' && (!!S.timer \|\| !!S.start[S.view] \|\| (!!hrNow() && !S.sessions.some(x => x.date === todayStr())));` |
| N2 | Minor | `index.html` line 2453, `visibilitychange` | Start a rest, turn the screen off, come back before the rest ends | Beep at 0:00 | On return the ring is re-synced but the beep is not re-scheduled. If Chrome paused the audio clock while hidden, the beep comes late, or the 4 s auto-close cuts it. Cannot be proven headless | In the handler: `if (timerActive()) { ringRun(); scheduleBeep(); }` |
| N3 | Cosmetic | `index.html` line 523 | Menu > Profile > Add scan, tap Weight | One lime border. v3.6.1 sets `.inp { outline: none }` on line 411 | The new `:focus-visible` rule comes later and wins. The field shows the lime border plus a lime outline 2 px outside it (double ring). Shot `qa37\f_input_focus.png` | Change line 523 to `:focus-visible:not(.inp) { outline: 2px solid var(--lime); outline-offset: 2px; }` |
| N4 | Cosmetic | `index.html` line 1840 | Do a set that is a new best, tap Hard within 420 ms | The Hard toast "Next set X kg · +30s rest" stays 2 s | The delayed "New best" toast replaces it after about 120 ms. Same for "Rest first" in that window. Shot `qa37\e_red_on.png` | `const t0 = toastT; setTimeout(() => { if (toastT === t0) toast('New best · ' + fmtW(s.w) + ' kg', false, 3000); }, 420);` |
| N5 | Cosmetic | `index.html` line 1966, `logSession` | Log with nothing done | Every refusal uses `block` | "Nothing to log" red toast has no haptic. Rare: Finish is disabled at 0 sets | Add `buzz('block');` before `return` |
| N6 | Info, owner call | `index.html` line 519, `.swap.fixed` | Added card, do its first set | Row reads as a label | Same height (52 px), set rows move 0 px, tap does nothing. It looks neat. The border and "+" icon still look a little like a disabled button | Optional: `border-color: transparent` (height stays) |

Not from v3.7.0: the HUD blur shows a done lime circle under it as a soft glow (`e_red_on.png`, right of Skip). Same CSS as v3.6.1.

## Checked and OK

**A. Safety**
- Text diff `index.v361.html` vs `index.html`: only "Added today" is new. No safety line changed or dropped.
- HR "> 100" keeps sets locked and gets `pick` only (no alarm buzz), as decided.
- Skip before 90 s: refused, red toast, `block`. Skip after 90 s needs 2 taps. Same as v3.6.1.
- Stepper and pills still cap at "Max safe load" (t12: 60 kg). Cap gives `block`.

**B. Rest timer (q1_timer, q2_resume)**
- Ring is 1 WAAPI animation. Duration = time left, ends at full offset (empty) when GO shows.
- Hard adds 30 s once. Old beep stopped (2 oscillators), new beep scheduled, ring re-run.
- Undo the set that started rest, Skip, and Log during rest: timer ends, beep stopped, ring animation gone.
- Rest end on time: 1 `restEnd` buzz, scheduled beep kept, refill and GO fade once. Back from hidden: no second buzz, refill or breath.
- +30 s after GO: rest restarts, beep and ring re-run, the 4 s auto-close does not end it.
- Late end (10 s): no buzz, beep cancelled, GO shown, HUD closes 4 s later.
- Reload during rest: timer, HUD and ring come back. No audio until a tap, no error. Breath runs once on the next circle.
- `buzz` does nothing while the page is hidden.

**C. One-shot effects**
- Every `fx()` call is followed by `render()`, `refreshCard()` or `renderStats()`. `fxQ` is empty after every action tested.
- No replay of set, complete, unlock, breath or flip after: stepper on another card, HR tap again, Easy, swap, tab away and back, rest ticks, Stats and back, Reorder on/off.

**D. Taps and sheets (q3_input)**
- Sheet guard lets real taps through after 350 ms: Import opens the file picker (click not prevented), "Open my week", plan pick. A second tap at 80 ms on Add and at 100 ms after a category swap is dropped.
- Swipe: 70 px slow drag snaps back. 160 px drag closes and the box only moves down (100 → 432 px). 60 px flick closes. touchcancel resets box and scrim. Up then down does not close.
- Press: on at pointerdown, instant (0 s), off on pointerup, pointercancel and scroll. None on disabled buttons. None stuck after a rebuild.
- 10 fast taps all count (reps 12 → 22, kg 20 → 60 cap). Max 1 tick per number.
- Hand flag: pill or stepper edit keeps its kg after Easy (toast "Saved"). Copy forward and "Use X kg" clear it. Easy still raises an unedited next set.
- "Added today": same height, rows do not move, inert, no press state. Undo brings "Remove from today" back.

**E. Reduced motion**
- m37 rm 53 pass. q3_rmcheck: 25 WAAPI calls, 0 with transform keyframes. Guide figures have 0 `<animate>`. Ring steps by the 250 ms tick. GO and ticks fade only.

**F. Widths**
- 360, 384, 412 px (iframe): no page overflow and no element outside the screen with a pressed pill, kg tick, set done + rest, long toasts, Easy, reorder FLIP, sheet open, Stats bars, class "Exercises first" FLIP.

**G. Screens at 412 px** (`qa37\a_lift_rest.png`, `b_added.png`, `c_red_toast.png`, `d_stats_week.png`, `e_red_on.png`)
- Glutes A in rest: neat. Done row tint and check, next circles dim, red ring, rate row clear.
- Red toast and red "on" states (HR "> 100", Hard): dark text #09090B on red reads well (5.1:1).
- Stats week: bars full after the grow, no clipping.

**H. Wording**
- "Added today" is short, clear and matches "Remove from today". No other visible text change.

## Check on the phone

Headless cannot run CSS transitions in `--dump-dom` (probe q4_timeline: values stay at the start). Please check on the S24 Ultra:

1. Press shows within 1 frame on a set circle, a stepper and an HR button. Start a scroll on a button: the press clears.
2. Settings > Accessibility > Visibility enhancements > Remove animations: sheets fade with no slide, ring moves in steps, guide figures are still.
3. Start a 90 s rest, lock the screen, wait: beep at 0:00. Repeat with YouTube open on top.
4. Haptics: tick, pick, done and block feel different. Rest end buzzes only when the screen is on.
5. Swipe a sheet down 150 px and let go: it keeps going down and fades, never jumps up. A short drag snaps back.
6. Rest ring is smooth at 120 Hz, also while the page scrolls after a set.
7. Open a video, come back mid-rest: the screen stays on.

---

# QA Report · SCULPT v3.6.1 · 2026-11-04

## Verdict: PASS

No blocking or major issue. 4 small notes below. All are optional cosmetic fixes and can ship as is.

## Summary

- All 8 suites pass with the expected counts. 0 fails, 0 console errors. Width scan is CLEAN at 360, 384 and 412 px.
- New probes: `qa361\p1_func.py` (84 checks) and `qa361\p2_width.py` (17 screens × 3 widths). All pass, 0 console errors.
- Safety text is intact. Each of the 67 review rows is applied as written. The lateral walk shows "15 steps / side", moves by 5, and remembers the best from the last session.

## Automated suites

Run one at a time, each with `timeout 170`. Output: `qa361\suites.out`.

| Suite | Checks | Fail | Console errors |
|---|---|---|---|
| t6 | 33 | 0 | 0 |
| t7 | 7 | 0 | 0 |
| t8 | 37 | 0 | 0 |
| qa_width | CLEAN × 3 (360 / 384 / 412) | 0 | 0 |
| t10 | 34 | 0 | 0 |
| t11 | 57 | 0 | 0 |
| t12 | 11 | 0 | 0 |
| t13 | 11 | 0 | 0 |
| qa361 p1_func (new) | 84 | 0 | 0 |
| qa361 p2_width (new) | 17 screens × 3 widths, no overflow, no clipped text, no target < 44 px | 0 | 0 |

Folder for `qa361`: `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\97fa529f-458e-437d-a894-6259a8e77f66\scratchpad\qa361\`.

## Findings

| # | Severity | Where | Steps | Expected | Actual | Fix |
|---|---|---|---|---|---|---|
| N1 | Cosmetic | `index.html` line 2061, `renderAdd` | Read the code | Comment matches the code | Comment says "up to 2 short lines"; the code now shows 1 (`lines.slice(0, 1)`) | Change the comment to "one line: the safety line first, else the answer" |
| N2 | Cosmetic | `index.html` lines 1784 and 1786, `openFinish` | Read the code | No unused variables | `addN` is still computed but no longer shown (row F4 cut it) | Remove `addN` |
| N3 | Cosmetic, owner call | Picker, "Band and mat" group, Banded Lateral Walk row | Log 30 steps, open Add > Glutes | Same wording as the card | Row says "Band · Last 30 steps". The card says "30 steps / side". The number is the same, so nothing is wrong | Optional: in `lastTxt`, add `(d.side ? ' / side' : '')` to the `noW` line. This also changes Dead Bug and Reverse Lunge rows |
| N4 | Info | Set rows (D9) | Open Glutes A at 360 px | Rows look even | Columns and row heights are equal (69 px each). The number on set 1 sits about 6 px higher than on sets 2 to 4, because "kg" / "reps" sits under it on set 1 only | No fix needed |

## Checked and OK

**A. Safety text (diff of `wording\index.v360.html` vs `index.html`)**
- HR box: "Resting heart rate", "< 90 · 90–100 · > 100", "Tap one to unlock your sets.", "Tap one before class.", both "Too high…" lines. Unchanged.
- D2: "exhale every rep" is kept, same red `.ex` span, same place. Only "N exercises ·" is cut.
- D8: sets and rest left the card but are in the "i" start box ("· 4 sets · 120s rest"). Rest timer, 90 s minimum and rest lock code are unchanged.
- C2: main classes and Yoga (`icon === 'cls'`) keep "No class or HR high? Swap to …". Only Cool-down Walk, Walk-down and Stretch say "Busy? Swap to …". The HR > 100 auto-swap is unchanged.
- C3: Combat keeps the same HR 120 gate. Text is "HR under 120? Start extras", "✓ HR under 120 · extras unlocked", "HR over 100 today. No extras."
- P8: one banner. Order is HR high (red, all rows disabled), then "Tap resting HR first.", then "Light only today.". Tested all 3.
- P10: "Light only today." Same rule (`i.lo`).
- M4: "Class days: max 3 light extras." Same limit.
- All `rules:` arrays, all `warm:` lines, Heart and Lifting lists: unchanged, except C5 ("5 min." cut; card meta still says "Treadmill · 5 min").
- All `avoid:` arrays: unchanged. One line added: Glutes B lateral walk now also has "Holding breath." (same as the Pump copy).
- Safety toasts and other section 3 strings: all present, none dropped.

**B. Each row applied as written**
- All 67 rows checked in the diff and in the browser.
- No double spaces, stray " · " or empty wrappers on: Glutes A, "i" sheet, Pump (before and after class), Finish, picker (both views, class day and lift day), Stats Week, 4 wk, 12 wk, Lifts.
- D9: units show on set 1 only. After +/− on rows 1 and 2, `refreshRow` keeps units off rows 2+. Columns line up (same x and width per column).
- C4: toast reads "Finisher unlocked" (Pump) and "Extras unlocked" (Combat).
- G2: "Start 20 kg × 12 reps" with no history. "Today 30 kg × 12 reps" after history, same as the card ("30 kg × 12", ▲ +5 kg). Every lift card on all 4 lift days, main and swap: "i" sheet kg and reps = card. 0 mismatches.
- A1: Pump and Combat show 1 add row: "Add before class" until the class starts, then 1 row at the end.
- R1: 1 Done button in Reorder. R2: class row sub reads "45 min".
- F1 kicker "Nov 4". F2/F3 tiles "Lifted, Sets, Time". T2/T3/T4: no Reset, Removed or Added toast.
- S10 date reads "Mon Nov 2 · 17 sets". S2 header has no "N short". S5 hides "Classes" at 0.
- APP_VER `3.6.1` = sw.js `sculpt-v3.6.1`.

**C. Banded Lateral Walk**
- Card: "Band × 15 steps / side". "i" sheet: "Start Band × 15 steps / side · 4 sets · 90s rest" and "Step right, then the same back left."
- Steps +/− by 5 (15 → 20 → 10).
- History with 20, 30, 25 → card and "i" sheet suggest 30. History with 10 → still 15 (never below program).
- Weighted moves unaffected: Cable Hip Abduction all hit → +1 rep at same kg; miss + hard → lighter. Same as v3.6.0.
- Volume = 0 for the band move (`setVol` returns 0 for `noW`).
- Weekly count: 3 logged sets = 3 Glutes sets. `side` does not double it (`histCat` counts `e.sets.length`).
- Same on Pump (wed_ab swap).

**E. Layout at 360, 384, 412 px**
- Glutes A, Glutes B with lateral walk, Pump before and after class, Combat before and after class, picker muscles and moves (class day and lift day), "i" sheet (weighted and lateral walk), Finish sheet, Stats Week, 4 wk, 12 wk, Lifts.
- No page wider than the screen, no clipped text, all tap targets ≥ 44 px. Screenshot `qa361\shot_latwalk360.png` looks right.

## Not tested

- A real phone. Only headless Chrome at set widths.
- Service worker update flow ("Updated to v3.6.1" toast) on a live install.
- A real double tap faster than 600 ms (covered in t12 from v3.6.0; code is unchanged).
- Stats Lifts rows with 3+ sessions were scanned for layout only. The "· 5×" cut (S14) was checked in the code diff, not on screen.

---

# QA Report · SCULPT v3.6.0 · 2026-11-04

**Verdict after fixes: PASS** (first pass: PASS WITH FIXES, see below)

## Re-test after fixes (same day)

| Finding | Fix applied | Re-test |
|---|---|---|
| F1 Start text stale after pre-class sets | On a day with a main class, a set tap (done or undo) redraws the whole list, so the class card, "Exercises first" sub line and "Back to class first" row update at once | qa36 p3b_start 4/4, p3_class 65/65; t12 "F1 Start text after 1 pre-class set" PASS |
| F2 Double tap hits the control that moved under the finger | `lastLayout` set by Class late / Back to class first, add and remove; main-list taps ignored for 600 ms after, except move arrows | t12 (real double tap at the same point, guard active): Class late moved once and no stepper change; Remove removes 1 of 2; picker row adds and keeps; taps work after 600 ms; arrows never blocked |
| F3 "All on track." while a short muscle was trained yesterday | Line reads "<Muscle> done yesterday. Rest it." (or "done today") | code review + t11 picker checks |
| F4 Lifts headline 9 words | "Up on X of Y lifts." | t12 |
| F5 "Finisher sets" on Yoga | "Finisher sets" only on days with a main class and no volume | t12 "F5 Yoga finish says Sets" PASS |
| Owner note: steppers not capped by `max` | Weight +/− and +kg pills stop at `max` (leg press 60, angled 40, hip thrust machine 60, light machines 30/40) with toast "Max safe load N kg" | t12 "Stepper capped at 60 kg", "Pill capped at 60 kg" PASS |

Suites after fixes: t6 33/0, t7 7/0, t8 37/0, t10 34/0, t11 57/0, t12 11/0, qa_width CLEAN ×3, p10_width CLEAN 32 states ×3, 0 console errors. QA probes in `qa36` re-run with human-paced taps (600 ms apart, so the new double-tap guard does not fire between scripted steps): p1 24, p3 65, p3b 4, p4 46, p5 15, p6 17, p7 18, p11 9, p13 8, all 0 fail. t11 uses the same human-paced tap helper. Open owner note: DB Hip Thrust (`thrust`) has no `max`.

---

**First-pass verdict: PASS WITH FIXES**

No data loss, no crash, no blank screen, no calorie UI. Export and Import work both ways. Every set lock holds in every order: resting HR, 90 to 120 s rest lock, class HR, post-class "HR under 120", pelvic floor caps. Fix F1 (Major) before deploy. F2 is Minor. F3 to F5 are Cosmetic.

## Summary

- Old suites: t6, t7, t8, t10, t11, qa_width, qa1b, qa2, qa3 all pass (191 checks + 6 width scans), 0 console errors. qa1 has 2 fails that are not regressions. Details below.
- 13 new probe files in `qa36`: 211 checks, 207 pass, 4 fail. All 4 fails are F1. Width scan: 42 states x 3 widths, clean. 0 console errors in every probe.
- Speed with 1Q data (13 weeks): every Stats tab renders in 12 ms or less (target 100 ms).
- Static: script and sw.js parse; APP_VER `3.6.0` = sw.js `sculpt-v3.6.0`; manifest valid; no new emoji glyphs; no debug code; no "kcal" or "calor" in index.html.
- Line numbers refer to `D:\Projects\Personal\Gym Tracking\index.html` (2287 lines).
- The spec was tested with the 12 review fixes (R1 to R12) as the intended behaviour where they differ from the spec.

## Automated suites

| Suite | Passed | Failed | Notes |
|---|---|---|---|
| t6.py (core lift flow) | 33 | 0 | console errors 0 |
| t7.py (gym machine swaps) | 7 | 0 | console errors 0 |
| t8.py (Stats, week plan, menu, guide) | 37 | 0 | console errors 0 |
| t10.py (Pump/Combat flow) | 34 | 0 | console errors 0 |
| t11.py (v3.6 spec hooks + review fixes) | 57 | 0 | console errors 0 |
| qa_width.py (360/384/412) | CLEAN x3 | 0 | |
| qa/static.py | 2 parse + 5 file checks | 0 | APP_VER = VERSION, icons, SHELL |
| qa/qa1.py | 42 | 2 + abort | Not regressions. 4d: known stale (same as v3.5.0). 4e: Lifts hides moves with under 3 sessions (R7, intended). Abort at X2: no calm button when HR > 100 (v3.5.0 fix F2). Same 4d fail and abort on v3.5.0. |
| qa36/qa1fix (corrected copy of qa1) | 56 | 1 | 4e taps "more"; X2 skips the calm tap. Only 4d (known stale) fails. |
| qa/qa1b.py | 13 | 0 | |
| qa/qa2.py (Pump/Combat layout) | 0 issues x3 | 0 | |
| qa/qa3.py | 10 | 0 | |
| qa36/p1_reorder (lift day reorder, timer, reload) | 24 | 0 | |
| qa36/p3_class (Pump + Combat order and gates) | 63 | 2 | F1 |
| qa36/p3b_start (real taps before class) | 2 | 2 | F1 |
| qa36/p4_add (add flow, limits, light lists, caps) | 46 | 0 | |
| qa36/p5_log (logging, counting, rounding) | 15 | 0 | |
| qa36/p6_rec (recommendation edge cases) | 17 | 0 | |
| qa36/p7_migr (v3.5.0 boot, backup round trips) | 18 | 0 | |
| qa36/p8_cal_speed + p8b_speed (no calories, speed) | 5 | 0 | 3 datasets, every screen |
| qa36/p10_width (360/384/412, 42 states each) | CLEAN x3 | 0 | |
| qa36/p11_static | 9 | 0 | |
| qa36/p12_geom (fast double taps) | info | - | F2 |
| qa36/p13_yoga (Yoga reorder, finish) | 8 | 0 | F5 seen here |

Probe files: `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\97fa529f-458e-437d-a894-6259a8e77f66\scratchpad\qa36\`

## Findings

### F1 · Major · "HR under 120? Start class" does not show after real set taps
- Where: `.ok` click handler, line 1683 (`refreshCard(ex)`) and `afterChange`, line 1728. The Start text is built only in `cardClass`, line 1461.
- Steps: Combat (or Pump) day, tap HR "< 90", tap "Class late? Exercises first", tap Face Pull set 1, let the rest end, do every pre-class set.
- Expected (spec D4, section 5): Start button reads "HR under 120? Start class" once a set above the class is done.
- Actual: button still reads "Start". The day sub still reads "Exercises first" and the row still reads "Back to class first" when no exercise is left above the class. The text is right only after a full render (tab switch, HR tap, reload). `#tN` correctly reads "Next: New Combat".
- Why Major: this is the only HR reminder between pre-class sets and the class. All set locks still work, so it is not a BLOCK.
- Fix: in the `.ok` handler (both the done and the undo branch), when `mainOf(DAYS[S.view])` exists, call `render(true)` instead of `refreshCard(ex)`. Or in `afterChange`, re-render the main class card and the day sub when the day has a main class.

### F2 · Minor · Fast double tap hits the control that moves under the finger
- Where: mainEl click handler, `[data-clslater]` line 1629 and `[data-xrm]` line 1631; `addExtra` line 2105; `removeExtra` line 2111.
- Steps and actual (360 and 412 px, second tap at the same point right after the first):
  - "Remove from today" twice: the second tap lands on the next added card's "Remove from today". Both cards are removed.
  - Picker exercise row twice on a lift day: the sheet closes, the second tap lands on the new card's "Remove from today". The card is removed again.
  - "Class late? Exercises first" twice: the second tap lands on a "−" stepper of the moved card. Weight or reps drop 1 step without notice.
- Expected: second tap ignored, like Start and set circles (1.2 s guard).
- Fix: keep `let lastLayout = 0`. Set it to `Date.now()` in `classLater`, `addExtra` and `removeExtra`. At the top of the mainEl click handler, return when `Date.now() - lastLayout < 600`, except for `[data-mv]` arrows (a repeat move is intended there).

### F3 · Cosmetic · Picker says "All on track." while a category is short but rested
- Where: `recommend`, line 1335.
- Steps: plan has Push today, Shoulders done 2 days ago, Chest 3 sets done yesterday. Open the picker.
- Expected: a line that matches the rows.
- Actual: banner "All on track.", Chest row 3 / 7 with hint "Done yesterday". The 48 h rule is correct (spec D9), only the line reads wrong.
- Fix: when no badge and a category has gap 2+ but was skipped by the 48 h rule, show "<Cat> done yesterday. Rest it." instead of "All on track.".

### F4 · Cosmetic · Lifts headline is 9 words
- Where: `statsLifts`, line 2009.
- Actual: "Up on 17 of 28 lifts since your first session." Review fix R7 asked for "Up on X of Y lifts". The rule is 8 words max.
- Fix: drop " since your first session".

### F5 · Cosmetic · Yoga Finish sheet says "Finisher sets"
- Where: `openFinish`, line 1790 (`vol ? 'Sets' : 'Finisher sets'`).
- Steps: Yoga day, add Dead Bug (no weight), do 3 sets, Yoga Done, Finish.
- Actual: tiles "Finisher sets 3 · Time 60 min". Yoga has no finisher.
- Fix: use "Finisher sets" only when `mainOf(day)` exists, else "Sets".

## Checked and OK

- Reorder (lift day): up/down arrows 48 x 48 px; first up and last down disabled; done card pinned "Done" and others can pass it; order saved as `S.order[day] = { d: today, ids }`; same order after `load()`; old-date order with 0 done sets is dropped; tab switch exits Reorder mode.
- Rest timer: moving cards during rest keeps `S.timer.end`; every pending circle stays locked; tapping one gives "Rest first"; `#tN` follows the new order ("Next: Chest Press Machine" after a move); timer and order survive reload. Finish + log and Reset both restore program order.
- Class days (Pump and Combat): "Class late" gives `[Hip Thrust · Light, Abductor · Light, New Pump, Cool-down Walk]` and `[Face Pull, Side-Lying, New Combat, Walk-down, Stretch]`. Cards above the class open with HR < 90 or 90 to 100 and stay locked with no HR tap ("Tap resting HR first"). Cards below stay locked ("Class first, then HR under 120"). HR > 100 locks all, swaps the class, hides "Class late". During class: "Class in progress". After class: "Tap HR under 120 first" until the calm tap. HR > 100 after calm still locks. Tail cool-down pinned "Stays last"; Walk-down moves with Combat as one block; a forced bad order is repaired by `fixOrder`. Started class pinned "Class started", nothing crosses it, no "Class late" row. Fast second tap on Start ignored. Calm row hidden when nothing is pending.
- Start during a rest timer ends the timer and starts the class (owner item 8, as specified).
- Add exercise: 3 taps (row, category, move). Card shows history-based suggestion (4 kg x 16 after 4 x 15), figure, "i" guide, steppers, pills, no swap line, "Added" badge. Rest 90 s, set 2 locked, Skip under 90 s refused, Hard rating saves `f`, lowers next set, adds 30 s. Card with a done set survives reload and a date change; card with 0 done from an old date is dropped. Remove in 1 tap (only with 0 done). hk with underscores (`cablefly_lh_l`) works. Max 3 per day, 4th refused. Duplicate refused, "Today" chips on Pump (`thrust_mf`, `abductor_f`) and Glutes A (`thrust_m` and its swap). HR > 100: rows disabled, add refused.
- Light-only lists on Yoga, Pump and Combat match spec section 3 (Back row disabled). Glutes row disabled the day before Glutes A or B on any day type, hint "Glutes B tomorrow. Rest them." (R2). Added cards on class days: "Add before class" goes above the class (lift gate), bottom row goes before Stretch (post-class gate).
- Caps: added cards keep `max` (`thrust_mf` 40, `abductor_f` 30, `legpress` 60, `thrust_m` 60, `anglepress` 40). Every hk has the same w, step, max and rest on every day. Suggestions never pass the cap.
- Logging: added sets go to `S.history[hk]`; swap sets to the swap hk with `swap: true`; session has `extra`, `cmin`; Finish "Sets" tile = logged `sets` on lift and class days; `vol` includes added sets; order and extras cleared after log.
- Counting: weighted sets by hand (primary 1, secondary 0.5, one round half up): Glutes 12, Chest 3, Back 2.5 (shown 3), Shoulders 6.5 (shown 7), Legs 3.5 (shown 4), Core 0. Screen matches. Swap Back Extension to DB Glute Bridge gives no Legs credit. Legacy `bridge_bw` counts as Glutes. Classes line "Classes · 2 · 105 min" (45 from `cmin`, 60 from `CLSMIN` for an old Yoga session); classes add 0 muscle sets. Picker counts live done sets.
- Recommendation: empty data "No data yet. Pick any."; all on track (no badge, 2 lines on Yoga, R1); HR > 100 red line, rows off; tie Chest = Shoulders at 100% goes to Chest; 3+ weighted sets yesterday excludes (2 sets does not); Glutes A/B tomorrow excludes Glutes (and a control run without it badges Glutes); tomorrow already logged does not block; class day Back "Do Pull day next" (R8); target 0 shows "0 / 0 · On track"; custom `S.plans` drives targets. Week answer "Behind: Chest · 7 short · do Push next".
- Migration: v3.5.0 state with 16 hks (incl. removed `foo_removed`), 24 sessions (one with `cal`), plans, basePlan, body, missing, swaps, active timer, cin, calm, start, lastExport boots with every old key deep-equal; new keys `{}`, `{}`, `'1m'`; timer keeps running and locking; Stats renders all tabs.
- Backup: old export imports; v3.6 export (with live order and an added card) then import is deep-equal except `lastExport`; after reload the added card and order are back; `order/extra/statRange/cin/calm: null` import renders day and Stats; junk shapes are cleaned (extras cut to 3); a non-backup file gives "Bad file".
- No calorie text on any screen with 1W, 1M and 1Q data: all 7 day tabs, picker, lists, Finish, all Stats tabs and ranges, last week, Lifts all, Menu, Profile, Rules, How, Guide.
- Layout 360/384/412 with 1Q data: 7 day tabs, Reorder on Push/Pump/Combat, class-later, started class in Reorder, picker on Yoga/Pump/Push, every category list, added cards, rest timer, Yoga done, Finish, HR high, Stats Week / 4 wk / 12 wk / Lifts / Lifts all / last week / 4 wk back. No page wider than the viewport, nothing past the edge or clipped, every visible button 44 x 44 px or more.
- Stats with dummy data: 12 wk shows "Since Oct 26" for 1W (R5); strength "+N% · K lifts" from program moves with 3+ sessions (R3); 4 wk rolling "Lowest: Core 75%" (R4); last week "Missed: Glutes A" (R6); Lifts "16 more · tap to show" (R7).
- Speed (1Q, max of 5 runs, no virtual time): Week 2.5 ms, 4 wk 2.7 ms, 12 wk 4.1 ms, Lifts 11.7 ms, picker 1.7 ms.
- Static: no PROGRAM id starts with `x_`; no hk has `~`; every hk has TAGS with valid `to`/`lv`; Legs never primary; each move in one picker category; Yoga rule "Light extras only.", Yoga `min: 60`, `vn: 'Class'`; Rules and How text updated per spec.

## Not tested

- Real touch timing on a phone. F2 was shown by geometry (element under the same point after the first tap).
- Vibration, wake lock, audio cue: headless only.
- Service-worker install, update toast and offline reload (file:// has no SW). Only the static SHELL and version check.
- Import of a v3.6.0 backup into the old v3.5.0 app (spec says not required).
- Google Fonts load and font fallback.
- Owner notes, not regressions: Start class does not require a resting-HR tap (same as v3.5.0). Weight steppers and +kg pills are not capped by `max`, only suggestions are (same as v3.5.0). DB Hip Thrust (`thrust`) has no `max` and is now also offered in the picker on lift days.

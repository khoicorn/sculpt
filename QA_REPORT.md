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

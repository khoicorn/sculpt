# QA Report · SCULPT v3.5.0 · 2026-11-04

**Verdict after fixes: PASS** (first pass: PASS WITH FIXES, see below)

## Re-test after fixes (same day)

| Finding | Fix applied | Re-test |
|---|---|---|
| F1 Start double-tap | second tap on Start ignored for 1.2 s; undo shown in every done state | t10 "Fast second tap on Start ignored" PASS; qa1b 13/13 |
| F2 HR > 100 contradiction | calm button replaced by "HR over 100 today. No finisher." + undo | qa2 hr-high state clean, no calm button |
| F3 Sets mismatch | logged `sets` on class days = finisher sets, same as the sheet | t10 Combat logged sets 4 PASS |
| F4 norm() missing keys | `cin`, `calm` added to norm | static + import probe PASS |
| F5 HR corrected leaves swap | low/mid tap clears the auto-swap when class not started | t10 "Swap back" PASS |
| F6 stale Start / class done | cin older than 4 h dropped; class done on another day dropped, with its start time | qa3: 30 h old class shows "Class · 45 min", no calm button |
| F7 undo 43.5 px | `.calmok .btn-q { min-width: 48px }` | qa3 "undo button exact size: 48.00x44.00" |
| F8 red cue on swapped class | `hrr` only when not swapped | qa2 clean |
| F9 note | intended: the "HR under 120?" tap is the post-class HR check | owner confirmed by design |

Suites after fixes: t6 33/0, t7 7/0, t8 37/0, t10 34/0, qa_width CLEAN ×3, qa1b 13/0, qa3 10/0, qa2 0 issues, 0 console errors. qa1 keeps 1 stale expectation (4d) that qa1b supersedes.

---

**First-pass verdict: PASS WITH FIXES**

No data loss, no crash, no broken core flow. Lift days, Yoga, Stats, Menu, Guide, Rules unchanged. The new Pump/Combat flow works end to end. Fix F1 (Start double-tap) and F2 (HR > 100 contradiction) before or right after deploy. The rest are minor or cosmetic.

## Summary

- 5 existing suites: 110 checks, 0 failures, 0 console errors. Width scan clean at 360/384/412.
- 3 new probe files (qa1, qa1b, qa3) + 1 layout probe (qa2): 90 checks. 7 real findings, 1 major.
- Static: script and sw.js parse, APP_VER 3.5.0 = VERSION sculpt-v3.5.0, manifest valid, icons match declared sizes, no leftover text from removed features.
- Lines below refer to `D:\Projects\Personal\Gym Tracking\index.html`.

## Automated suites

| Suite | Passed | Failed | Notes |
|---|---|---|---|
| t6.py (core lift flow) | 33 | 0 | console errors 0 |
| t7.py (gym machine swaps) | 7 | 0 | console errors 0 |
| t8.py (Stats, week plan, menu, guide) | 37 | 0 | console errors 0 |
| qa_width.py (360/384/412) | CLEAN ×3 | 0 | no overflow, no small target |
| t10.py (new Pump/Combat flow) | 33 | 0 | console errors 0 |
| qa/static.py (new) | 2 parse + 5 file checks | 0 | see Checked and OK |
| qa/qa1.py (new, regression + 4a-4g + extras) | 60 | 4 | 2 were wrong expectations (reset, re-checked in qa1b); 2 real: F3, F4 |
| qa/qa1b.py (new, reset + double-tap geometry) | 13 | 1 | real: F1 |
| qa/qa3.py (new, tap size, stale state, HUD) | 10 | 1 | real: F7 (undo 43.5 px wide) |
| qa/qa2.py (new, layout wed/sat × 6 states × 3 widths) | 0 overflow, 0 clipped | 1 | same as F7 |

Probe files: `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\97fa529f-458e-437d-a894-6259a8e77f66\scratchpad\qa\`

## Findings

### F1 · Major · "Start" double-tap marks the class done after 0 minutes
- Where: click handler `[data-clschk]`, lines 1302-1308; `cardClass` line 1079-1081.
- Steps: Pump or Combat day, tap resting HR, tap **Start** twice fast (under 300 ms).
- Expected: second tap ignored (the app already ignores a fast second tap on 1-tap cards and on undo).
- Actual: after the first tap the button re-renders at the same spot as **Class done**; the second tap marks the class done. `S.classes` set, card collapses, calm button appears. There is no undo in this state (undo only shows after the calm tap), so the user must tap "HR under 120? Start finisher" then "undo", or Reset the whole day.
- Fix: in the `[data-clschk]` handler, when `S.cin[id]` exists and `now - S.cin[id] < 1200`, return. Also show the `undo` button next to the calm button (done, not yet calm state), not only inside `.calmok`.

### F2 · Minor · HR "> 100" on a class day: calm button says "unlocked" but sets stay locked
- Where: `hrBlocked` line 1141-1145; `cardClass` line 1081-1083.
- Steps: Combat day, tap **> 100** (auto-swap to Easy Elliptical), Start, Class done, tap **HR under 120? Start cool-down**, tap a Cable Face Pull set circle.
- Expected: one clear rule. Either the finisher unlocks after the calm tap, or the calm button is not offered and the card says "HR too high. No finisher today."
- Actual: card shows "✓ HR under 120 · cool-down unlocked", toast says "HR too high today", circles stay locked. Safe, but contradictory.
- Fix: in `cardClass`, when `hrNow() === 'high'` show a static line "HR over 100 today. No finisher." instead of the calm button. Or drop `hrNow() === 'high'` from the class branch of `hrBlocked` if the calm tap is the intended gate.

### F3 · Minor · Finish sheet "Sets" differs from the logged session sets on class days
- Where: `openFinish` line 1386 vs `logSession` line 1415.
- Steps: Pump day, class done, calm, 2 Hip Thrust sets, Cool-down Walk done, Finish.
- Expected: same number in the sheet and in the saved session.
- Actual: sheet shows Sets 2 (exercise sets only); saved session has `sets: 4` (`p.d`, which counts the 2 class cards). Stats > Month then lists "4 sets" and Edit session shows 4.
- Fix: in `logSession`, for class days use the same count as `openFinish` (exercise sets only), or store both.

### F4 · Minor · Import of a backup with `cin: null` or `calm: null` breaks every render
- Where: `norm` line 896 lists 10 object keys but not `cin` and `calm`.
- Steps: import a JSON where `"cin": null` (hand-edited or a corrupt file). Normal exports always contain `{}`, so this needs a damaged file.
- Expected: `norm` fills `{}` like it does for `sets`, `classes`, etc.
- Actual: `S.cin[it.id]` throws "Cannot read properties of null". The state is saved before `render()` throws, so the app is blank on every reload until localStorage is cleared.
- Fix: add `'cin','calm'` to the array in `norm`.

### F5 · Minor · HR corrected from "> 100" to "< 90" leaves the class swapped
- Where: HR handler line 1238-1240.
- Steps: class day, tap **> 100** by mistake, then tap **< 90**.
- Expected: main class comes back, or at least the user is told it is still swapped.
- Actual: card still shows Light Circuit / Easy Elliptical. The "Back to New Pump" button is there, so it is recoverable. Header minutes also stay on the alt (20 min class).
- Fix: on `low`/`mid` tap, clear `S.swaps[main]` if the class is not done (mirror of the auto-swap).

### F6 · Minor · Stale "Start" is not date-scoped
- Where: `cardClass` line 1076-1077; `S.cin` persists like `S.start`.
- Steps: tap Start, leave the app, open the Pump tab the next day.
- Expected: card back to Start, or "Started yesterday".
- Actual: "In progress · 1560 min". If Class done is tapped then, Time tile shows 240 min (sessMin cap). Same class of issue as `S.classes` not being date-scoped (pre-existing: a class done 30 h ago still shows "✓ Done · 45 min" and the calm button).
- Fix: in `cardClass` or at load, drop `S.cin[id]` (and `S.start[day]`) when older than 4 h.

### F7 · Cosmetic · "undo" button is 43.5 × 44 px
- Where: `.calmok .btn-q` line 164 sets `min-height: 44px` only.
- Actual: width 43.53 px at 360/384/412. Target rule is 44 × 44.
- Fix: add `min-width: 44px` (or `padding: 0 12px`).

### F8 · Cosmetic · First cue line of a swapped class is painted red as an HR rule
- Where: `cardClass` line 1075 and the `hrr` class on index 0 (line 1086).
- Steps: Combat day, tap **> 100** (or the swap button). Easy Elliptical card shows "20 min." in red.
- Expected: red only for the HR rule of the real class.
- Fix: apply `hrr` only when `!sw`.

### F9 · Note for the owner · Finisher no longer needs a resting-HR tap on class days
- In 3.4.3 the Pump finisher sets were locked until HR < 90 or 90–100 was tapped. In 3.5.0, Class done + calm tap unlocks the sets with no HR tap at all (verified: `hrNow() === ''` and set done = true). The calm tap is itself an HR check, so this may be intended. Confirm. If not, add `!['low','mid'].includes(hrNow())` to the class branch of `hrBlocked`.

## Checked and OK

- Static: single `<script>` parses with `new Function`; sw.js parses; manifest.json valid; `start_url ./index.html`, `scope ./`; 3 icons on disk, PNG, 192×192 / 512×512 / 512×512 as declared; APP_VER 3.5.0 = `sculpt-v3.5.0`; `?go=next` / `?go=stats` handler present (line 1722-1724).
- SHELL list has `./`, index.html, manifest.json and all 3 icons. The page needs nothing else at first load (fonts are runtime-cached, sw.js is held by the browser).
- Grep: no "Incline Walk", "Mark done", `class="alt"`, "kcal", "calorie", "Easy Walk", "Cool-down Stretch". Only hit is "DB Glute Bridge" (lift-day swap, expected).
- Lift days mon/tue/thu/fri: set tap blocked before HR ("Tap resting HR first"), HR low unlocks, first set starts timer (120 s / 90 s), HUD shows timer, next circle locked and "Rest first" toast, unlock after timer. Headers still "N exercises · ~M min · exhale every rep", no chips, HR label "Resting heart rate", warm-up box present. HR > 100 locks sets, no auto-swap on lift days.
- Yoga: sub text unchanged, no warm-up box, no set circles, HR > 100 does not swap, 1-tap Done works.
- Stats Week/Month/Lifts render; Menu, Guide, Rules, Profile sheets open; 0 console errors in every run.
- 4a: reload mid-class keeps `S.cin`, button says "Class done", meta "In progress · 0 min".
- 4b: "Class done" double tap: second tap lands 154 px below the new calm button on the next card body (Pump: Hip Thrust suggestion area, Combat: Walk-down rules). Nothing changes. Class stays done, calm not set, no swap toggled. Calm button double tap lands on `.calmok` text, nothing changes.
- 4c: undo clears `S.classes` and `S.calm`, button back to Start, sets re-lock with toast "Class first, then HR under 120".
- 4d: Reset clears cin, calm, classes, done sets, start; HUD 0 / 6; sets locked again; gym-missing swap kept.
- 4e: logged Pump session → Week shows PM slot ✓ done, "Classes 1 / 2"; Lifts tab lists "Hip Thrust Machine · Light 15 kg · 1× new"; history under `thrust_mf` with 2 sets at 15 kg.
- 4g: export → import gives identical state (ignoring timer); card still "Class done"; old backup without cin/calm imports and renders.
- Class-only session: Finish enabled, sheet shows only "Time 45 min" (no Volume, no Sets tiles), logged min 45, vol 0, title "New Pump". `dayMin` uses elapsed time (80) when larger than class minutes (45).
- Swapped class logs as "New Pump · swap"; swapped finisher (Frog Pump) logs under `frogpump` with `swap: true`. Exercise swap and class swap are independent.
- Combat: chip "Max 1× per week" before, red "Combat done this week" + kicker "done 1× this week" after a log; tab shows "Done". Header "45 min class + 20 min after · ~70 min"; swapped header "20 min class + 20 min after · ~45 min". HUD 0 / 7.
- Non-main class cards (Cool-down Walk, Walk-down, Stretch) are 1-tap, not gated, no calm button. Guide sheet opens for Side-Lying Leg Raise.
- Layout 360/384/412, wed and sat, states fresh / in-class / class done / after calm / 1 set with rest HUD / finish sheet / HR high swapped: no page overflow, no horizontal overflow or clipping in `.stp .v`, `.meta`, `.day-head p`, `.swap .tx`, `.ccard li`, `.calmok`, `.big`, `.chips`, `.kv`; all `.big` buttons ≥ 56 px high; only small target is F7.
- Screenshots at 384 px (qa\shot_*.png) match the spec text: Pump header, HR box label, warm-up line, red HR rule first, swap button, Start / Class done / calm button, finish tiles.

## Not tested

- Vibration, wake lock, audio cue: headless only.
- Real service-worker install, update toast and offline reload: file:// has no SW. Only the static SHELL/version check was done.
- Real install from manifest, home-screen shortcuts on a device.
- Touch gestures (swipe back, long press), real double-tap timing on a phone. F1 was shown by geometry: the second tap point sits on the new button; the 1.2 s undo guard does not cover it.
- Google Fonts load and font fallback.
- Timer accuracy over a real 90 s with the screen off.

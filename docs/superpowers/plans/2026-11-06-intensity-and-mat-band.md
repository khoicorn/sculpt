# SCULPT v3.8.0 Intensity + Mat/Band + Walk Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace every heart-rate check with a Light / Normal / Hard intensity picker, add 16 mat and band moves, a "Mat + Band" workout, a "Walk" tab with an Easy Run swap, a distinct half-kneeling 1-arm pulldown, and a kcal estimate that replaces kg volume in reports.

**Architecture:** Everything lives in the single file `index.html` (vanilla JS, state object `S` in localStorage key `sculpt_v3`). Intensity is a layer: `suggest()` stays mode-free; a new `planOf(c)` applies today's mode on top. New moves sit in a `LIBRARY` array merged into `HKDEF`. Tasks run one after another because they all edit the same file.

**Tech Stack:** HTML/CSS/vanilla JS PWA. Tests: Python 3.12 probe runner that injects JS into a copy of `index.html` and runs it in headless Chrome, reading PASS/FAIL lines from `document.title`.

**Spec:** `docs/superpowers/specs/2026-11-06-intensity-and-mat-band-design.md` (read it first; this plan points to its sections for exact values and strings).

## Global Constraints
- Single file `index.html`; no new libraries; no network calls; no calorie UI.
- UI strings: 1 line, 3-8 words, exactly as written in the spec.
- Medical limits hold in every mode: kg never above today's suggestion or the item's `max`; rest never under 90 s (skip lock stays); every set stops with 2+ reps left; no HIIT; exhale on every push.
- Done sets are never deleted or changed by any mode change, swap or reload.
- Old data (v3.7 localStorage and old JSON backups with `hr` / `calm`) must load without errors through `norm()`.
- Version 3.8.0 in both `sw.js` `VERSION` and `index.html` `APP_VER`.
- Phone widths 360 / 384 / 412 px: no horizontal scroll, tap targets 44 px or more.
- Python: `D:/Miniconda3/envs/gym-tracking/python.exe -X utf8` with `PYTHONIOENCODING=utf-8`. Chrome: `C:\Program Files\Google\Chrome\Application\chrome.exe`.

## Review Focus
1. Mode switched after sets were done out of order (e.g. set 3 done, sets 1-2 pending), then switched back: the done set stays on its row with its kg.
2. Midnight rollover with the app open: yesterday's `S.mode` must not apply today; the suggestion is recomputed.
3. Swapped exercise on a Hard+ item: the swap gets the +1 set; swapping back keeps done sets.
4. Tiny weights on Light: suggestion equal to one `step` (e.g. 2.5 kg) must not become 0 kg; never below one `step`.
5. User already has 3 extras today (allowed in v3.7) and Normal now caps at 2: nothing is removed, Add is refused with the cap toast.

Each line has a test in the owning task (Tasks 2, 3).

---

## Test harness (shared by all tasks)

Folder: `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\71c8ab23-87a6-4dce-824c-bdfcbf9fe832\scratchpad\qa38\` (call it `QA`).
- `QA\qrun.py`: copy of `C:\Users\ngoho\AppData\Local\Temp\claude\52c4674560278348\97fa529f-458e-437d-a894-6259a8e77f66\scratchpad\qa37\qrun.py`. Keep `run(name, body, budget, pre_state, rm, src, size)` and `run_frame(name, body, width)`. Change only `fresh0()` in `HELP`: drop nothing else; it must still work after `S.hr` is gone.
- A probe file is `QA\pN_name.py`: `from qrun import run` then `run('pN_name', body)` where `body` is JS using `ok(name, cond)`, `info()`, `q()`, `qa()`, `sleep()`, `okBtn(id,k)`, `fresh0()`.
- Run one probe: `cd "$QA" && PYTHONIOENCODING=utf-8 /d/Miniconda3/envs/gym-tracking/python.exe -X utf8 pN_name.py`
- Pass means: every line starts `PASS` or `INFO`, and the last line is `console errors: 0`.
- After each task also run every earlier `QA\p*.py`; all must still pass.

Branch: `feat/v3.8-intensity` from `main`. Commit after each task. Leave untracked `t7.html` alone.

---

### Task 1: Remove heart-rate checks, add the mode picker shell

**Files:**
- Modify: `index.html` (HR sites listed in spec "Feature 2"; state `fresh()` ≈1071, `norm()` ≈1074, `hrNow` ≈1125, `gate` ≈1278, `renderDay` HR box ≈1474, class card ≈1555-1577, click handlers ≈1777-1780 and ≈1849-1850, `renderAdd` ≈2217-2240, `addExtra` ≈2254, `inWorkout` ≈2452, CSS ≈75-82 / 161-168 / 529-532, Rules / How it works / Menu HTML ≈600-660, `PROGRAM` warm-ups and class rules ≈867-991)
- Modify: `manifest.json` (description)
- Test: `QA\p1_hr.py`

**Interfaces:**
- Produces: `S.mode` (`{v,d}` or `null`); `modeNow() -> 'light'|'normal'|'hard'` (in this task: `S.mode.v` if dated today else `'normal'`; Task 3 adds the suggestion); picker markup `.mode` with buttons `[data-mode="light|normal|hard"]` and `.on` on the current one; `setMode(v)` saves `S.mode = {v, d: todayStr()}` and re-renders.

- [ ] **Step 1: Write the failing probe `p1_hr.py`** with these checks:
  - `typeof hrNow === 'undefined'` and `!('hr' in fresh())` and `!('calm' in fresh())` and `fresh().mode === null`.
  - Migration: `norm({hr:{v:'high',d:todayStr()}, calm:{wed:todayStr()}, mode:'bad', sessions:[], history:{}})` returns an object with no `hr`, no `calm`, `mode === null`; `norm({...,mode:{v:'hard',d:'2026-11-06'}}).mode.v === 'hard'`.
  - On `mon`: `q('[data-hr]') === null`; `qa('[data-mode]').length === 3`; Normal has `.on`; label text `How hard today?`; tapping `[data-mode="light"]` gives `S.mode.v === 'light'` and the hint `Less kg. Progress kept.` is visible; tapping Normal removes the hint.
  - Sets open without any tap: `gate(DAYS.mon, EX.mon_thrustm) === ''`.
  - Pump (`wed`): before class `gate(DAYS.wed, EX.wed_ht) === 'Class first'`; main button text `Start`; after class done, `gate(...) === ''`, no `[data-calm]` exists, done line text contains `Walk 3 min, then finisher.`; Combat done line is `✓ Class done` (no walk text).
  - Swap button on Pump class card contains `No class? Swap to`.
  - `inWorkout()` is false after only tapping a mode button (no set done).
  - Text scan: for each of `#app`, `#mRules`, `#mHow`, `#mMenu` and every day rendered via `S.view=id; render()`, `innerText` has no match for `/\bHR\b|heart rate|bpm|calm\b/i` except the allowed phrase `breathing is calm` and `heart racing`.
  - Rules sheet contains each of the 6 Safety lines, the 3 Intensity lines and `Extras: Light 0, Normal 2, Hard 1.`, `Gym full? Do Mat + Band.` (spec "Feature 2 > Rules sheet").
  - Warm-up of `mon` equals `Bike 8 min, easy. Can chat.`; Pump class rule 1 is `Dizzy, chest tight, heart racing = stop.` and has class `hrr`.
- [ ] **Step 2: Run it.** Expected: FAIL lines (hrNow defined, `[data-hr]` present).
- [ ] **Step 3: Implement.** Follow spec "Feature 2" and "Feature 1 > State / UI" for the shell only (no mode effects on sets yet). Move the unlock fade from the calm handler into the "Class done" branch. Delete the HR auto class-swap. Rename `.hr` CSS to `.mode`; drop `.opt.bad`, `.hr.bad`, `.big.calm`, `.calmok.red`. Pump done line comes from `day.after` existing. `manifest.json` description: remove "Heart-safe".
- [ ] **Step 4: Run `p1_hr.py`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git add index.html manifest.json && git commit -m "v3.8 T1: remove HR checks, add intensity picker shell"`

---

### Task 2: Intensity engine on lift days

**Files:**
- Modify: `index.html` (`suggest` ≈1140-1156, `sets()` ≈1157-1164, `wNext` ≈1133, `lastSession` ≈1131, card render `cardEx` ≈1517-1533, `refreshRow` ≈1866-1875, "Use X kg" ≈1786-1790, `#feel` ≈1749-1755, set done ≈1830-1840, rest skip floor ≈1645 / 1735, time estimate ≈1462, Guide ≈1903, `logSession` ≈1939-1973, Finish sheet, Stats list, `statsLifts` ≈2162, 4-week ≈2085, 12-week ≈2134, `lastTxt` ≈2204, `recommend` ≈1351-1370, extras cap ≈1075 / 1498 / 2256, `PROGRAM` Hard+ flags)
- Test: `QA\p2_engine.py`

**Interfaces:**
- Consumes: `modeNow()`, `setMode(v)` from Task 1.
- Produces:
  - `planOf(c) -> { sets: number, w: number, r: number, rest: number }` for any card def `c` (program item, swap, or added extra) under `modeNow()`.
  - `lastBase(hk) -> historyEntry | null`: newest entry of `hk` whose `mode !== 'light'`.
  - `maxExtra(day) -> 0|1|2` from `modeNow()` (Light 0, Normal 2, Hard 1). Yoga and Walk use Normal.
  - Item flag `hp: 1` on Hard+ program items: `mon_thrustm`, `mon_kickm`, `tue_cpress`, `tue_latm`, `thu_hipabd`, `thu_backext`, `fri_lpd`, `fri_row` (Task 5 adds `mat_bridge`, `mat_kick`).
  - Done set field `m: 'l'|'n'|'h'`; history entry `mode: 'light'|'hard'` (absent = Normal); session `mode` (absent = Normal).

- [ ] **Step 1: Write the failing probe `p2_engine.py`.** Seed history so `suggest(EX.mon_thrustm)` gives 30 kg, step 5, reps 12. Checks:
  - Normal: `planOf(EX.mon_thrustm)` = `{sets:4, w:30, r:12, rest:120}`.
  - Light: `{sets:3, w:20, rest:120}` (80% of 30 = 24 → round down to step 5 = 20; at least 1 step under). Dead Bug (`noW`, reps 10) Light reps = 8; Hip Thrust reps unchanged.
  - Light tiny weight: item with suggestion 2.5, step 2.5 → Light w = 2.5 (never below one step).
  - Light assist machine (`fri_apu`, `assist: true`): Light w = suggestion + 1 step.
  - Hard: `planOf(EX.mon_thrustm).sets === 5`, `w === 30`; `planOf(EX.mon_legpress).sets === 3`; rest of Hard+ item 120.
  - Hard+ swap: swap `mon_thrustm` to its DB version; its Hard sets = program sets + 1.
  - Cards follow the plan: on Light the Hip Thrust card shows 3 set rows and 20 kg; the "Use X kg" button never offers more than the plan kg.
  - Mode change safety: Normal, mark set index 2 done at 30 kg; switch Light → card still has the done set at 30 kg on its row, total rows = max(done count, 3); switch Normal → 4 rows, the done set unchanged.
  - Stamp: the done set has `m === 'n'`; a set done on Light has `m === 'l'`.
  - Log: finish a session with one Light set and three Normal sets → history entry `mode === 'light'`; `lastBase('thrust_m')` returns the previous entry, and next `suggest` is unchanged from before the session.
  - Hard log: 5 sets all reps hit → next suggestion moves at most +1 step (judged on first 4).
  - Easy on Light: rating Easy does not raise the next set's kg; Hard rating lowers 1 step and adds 30 s.
  - 21-day rule: last non-Light entry dated 22 days ago at 30 kg → `suggest` gives 25 kg.
  - `noW` with `repTop`: def `{noW:'Band', reps:15, repTop:20, sets:3}`, last non-Light entry all 3 sets at 15 → next target 16; at 20 stays 20; `unit:'sec'` 20 → 25.
  - Extras: `maxExtra` per mode; with 3 extras already in `S.extra.mon`, `norm(S)` keeps all 3 and `addExtra` refuses with toast `Max 2 added today`.
  - Light: `recommend(DAYS.mon).cat === null`.
  - Finish sheet date line ends ` · Light` on Light; Stats workout row for that session ends ` · Light`; nothing appended on Normal.
  - Strength views: a Light entry is skipped by `statsLifts`, the 4-week compare, the 12-week chart and `lastTxt`.
  - Midnight: `S.mode = {v:'hard', d: <yesterday>}` → `modeNow() === 'normal'`.
- [ ] **Step 2: Run it.** Expected: FAIL (`planOf` undefined).
- [ ] **Step 3: Implement** per spec "Feature 1 > Rules per mode: lift days", "Progression", "Mode change mid-workout". Point every display and seed call at `planOf`. In `sets()` remove only pending rows, never below done count. On mode tap, reseed cards with no done set. Skip floor: `Math.min(90, total)`. Replace the literal 3 extras cap with `maxExtra(day)`; toast `Max ${n} added today`; when `n === 0` toast `Light day. No extras.`
- [ ] **Step 4: Run `p2_engine.py` and `p1_hr.py`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T2: intensity engine, safe set resize, progression guards"`

---

### Task 3: Suggested mode, Hard lock, class-day behaviour

**Files:**
- Modify: `index.html` (`modeNow`, picker render, `PROGRAM` Pump / Combat items, class card render, Combat swap logic)
- Test: `QA\p3_suggest.py`

**Interfaces:**
- Consumes: `modeNow`, `setMode`, `planOf`, `maxExtra`.
- Produces:
  - `suggestMode() -> { v: 'light'|'normal', why: 'After Pump'|'After Combat'|'After a break'|'' }`.
  - `hardBlock() -> ''|string` (toast text when Hard is not allowed today).
  - `modeNow()` now falls back to `suggestMode().v` when `S.mode` is not dated today.

- [ ] **Step 1: Write the failing probe `p3_suggest.py`.** Checks:
  - Session `{day:'wed', date:<yesterday>}` logged → `suggestMode()` = `{v:'light', why:'After Pump'}`; Light button shows small `After Pump`; Light is `.on` with no tap. Same for `sat` → `After Combat`.
  - Yoga or Mat logged yesterday → Normal. Last session 10 days ago → `{v:'light', why:'After a break'}`; 9 days → Normal. No sessions at all → Normal.
  - After the first set is done, `S.mode` is written with today's date (suggestion frozen).
  - `hardBlock()`: Pump yesterday → `Pump yesterday. Not today.`; Combat → `Combat yesterday. Not today.`; a session with `mode:'hard'` yesterday → `Hard yesterday. Not today.`; 2 Hard sessions in the last 7 days → `2 Hard days this week.`; else `''`. Blocked Hard button has class `off`, tap shows the toast and leaves the mode unchanged.
  - Pump Light: rule 2 text `No plates on squats or lunges.`; finisher cards `wed_ht`, `wed_ab` not rendered and not counted in day progress; cool-down walk still there. Pump Hard: finisher cards have 3 rows.
  - Combat Light (class not started): main class card shows the alt (Easy Elliptical); switching to Normal restores the class; if the user swapped by hand before picking Light, Normal keeps the hand swap. Class started → Light does not swap. Combat Hard: post-class cards `sat_fp`, `sat_sl` have 3 rows.
  - Yoga (`sun`): no `.mode` picker; `maxExtra(DAYS.sun) === 2`.
- [ ] **Step 2: Run it.** Expected: FAIL.
- [ ] **Step 3: Implement** per spec "Feature 1 > When the app suggests Light", "Hard availability", "Rules per mode: class days". Track the Light-made Combat swap in `S.swaps` with a marker value (e.g. `'mode'`) so restore only undoes its own swap.
- [ ] **Step 4: Run `p1`-`p3`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T3: suggested Light, Hard lock, class-day modes"`

---

### Task 4: Mat and band library data, picker, new pulldown swap

**Files:**
- Modify: `index.html` (new `LIBRARY` after `PROGRAM`; merge into `HKDEF` ≈1061; `TAGS` ≈1004-1056; `ADDGROUPS` ≈2215; row label ≈2240; Pull day swap ≈936)
- Test: `QA\p4_library.py`

**Interfaces:**
- Produces: `LIBRARY` (array of 16 defs, fields per spec "Feature 3": `hk, name, what, vn, muscle, target, sets, reps, repTop, rest: 90, noW, side?, unit?, guide:{setup[3], avoid[2-3]}, move`); `TAGS[hk]` for each with `pl` in `'Mini band'|'Long band'|'Mat'`, `l: 1` where the spec marks it, and `bar: 1` on `bandpu`; swap `cablepd1` on `fri_lpd` with `move: 'kneelpd1'`; `TAGS.cablepd.old = 1`.
- Poses are Task 5; in this task each new `move` key may point to a placeholder that renders (any existing pose), so nothing crashes.

- [ ] **Step 1: Write the failing probe `p4_library.py`.** Checks:
  - `LIBRARY.length === 16`; each of the 16 hk names from the spec table is in `HKDEF` with the spec's name, sets, reps, repTop, side, unit, `noW`, and its `TAGS` p / s / pl / l.
  - Each def has `guide.setup.length === 3`, `guide.avoid.length` 2 or 3, every cue 3-8 words, `what` ≤ 10 words, `rest === 90`.
  - `TAGS.abd.pl === 'Mini band'`, `TAGS.latwalk.pl === 'Mini band'`, `TAGS.bandpu.pl === 'Long band'`, `TAGS.bandpu.bar === 1`.
  - Add sheet on `mon`, category Glutes: a group labelled `Band` lists `Long-Band Glute Bridge` with small text starting `Long band ·`; a group `Mat` lists `Single-Leg Glute Bridge`.
  - Pump (class day), Glutes: only `l: 1` moves (e.g. Banded Clamshell, Fire Hydrant) and existing light moves appear.
  - Adding `bandbridge` on `mon` creates a card that renders, opens its Guide (setup, avoid, video search link), logs, and survives `location.reload()` and Export → Import.
  - `sideplank` card shows seconds (`20 s`), per side.
  - Pull day: swap of `fri_lpd` is `Half-Kneeling 1-Arm Pulldown`, hk `cablepd1`, `side: true`, `w: 7.5`; old `cablepd` history still counts in Back volume.
- [ ] **Step 2: Run it.** Expected: FAIL (`LIBRARY` undefined).
- [ ] **Step 3: Implement** with values and cues copied from spec "Feature 3" and "Feature 6".
- [ ] **Step 4: Run `p1`-`p4`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T4: mat and band library, 1-arm pulldown swap"`

---

### Task 5: Stick figures for 17 new moves

**Files:**
- Modify: `index.html` (`MOVES` ≈702-842)
- Test: `QA\p5_poses.py` plus PNG screenshots in `QA\poses\`

**Interfaces:**
- Consumes: `move` keys from Task 4.
- Produces: `MOVES` keys `bandbridge, slbridge, bandkick, hydrant, clamshell, bandrdl, birddog, sideplank, bandpd, bandrow, proney, pullapart, bandlat, bander, kneepush, bandpress, kneelpd1`. One shared all-fours base for `bandkick`, `hydrant`, `birddog`.

- [ ] **Step 1: Write the failing probe `p5_poses.py`.** Checks: every key above exists in `MOVES` with `A` and `B` arrays of even length; `moveSVG(key,false)` and the Guide's animated SVG render with no error; pose A ≠ pose B; for each pair of near neighbours from the spec pose notes (bandbridge vs bridge vs frogpump; clamshell vs sidelying; bandrdl vs rdl; kneelpd1 vs latpull; bandkick vs hydrant vs birddog; bandpd vs latpull) the B arrays differ by more than 15 units summed. Also save a 54 px and a 210 px screenshot per move into `QA\poses\` (use `run` with a page that lays them out in a grid).
- [ ] **Step 2: Run it.** Expected: FAIL (keys missing).
- [ ] **Step 3: Draw the poses** from spec "Feature 3 > Stick figures" and "Feature 6 > Pose". Coordinates are the builder's call. Ground y = 90.5.
- [ ] **Step 4: Run `p5_poses.py`, then look at every screenshot** (Read the PNGs) and fix any figure that reads wrong or matches its neighbour. Then run `p1`-`p4`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T5: stick figures for mat, band and 1-arm pulldown moves"`

---

### Task 6: Mat + Band workout and Walk tab with Easy Run

**Files:**
- Modify: `index.html` (`PROGRAM` new entries `mat` and `walk`; `ORDER` ≈994; `ABBR` ≈1057; plan sheet ≈2191; week grid CSS ≈305; `statsWeek` ≈2043; `glutesTomorrow` ≈1332; `rowsFor` / `recommend` / `renderAdd` filter; class card swap text and Run block; `logSession` `run` flag; `cardEx` swap guard ≈1523)
- Test: `QA\p6_days.py`

**Interfaces:**
- Consumes: `LIBRARY`, `planOf`, `suggestMode`, `hardBlock`, `modeNow`.
- Produces: `DAYS.mat` (type `lift`, items `mat_bridge, mat_clam, mat_kick, mat_bird, mat_pd, mat_press, mat_pull` with swaps and `hp: 1` on `mat_bridge`, `mat_kick`); `DAYS.walk` (type `class`, main item `walk_walk`, alt Easy Run with `alt.rules`); `runBlock() -> ''|string`; session field `run: 1`.

- [ ] **Step 1: Write the failing probe `p6_days.py`.** Checks:
  - Tabs in order end `... PULL, PUMP, COMBAT, YOGA` with `MAT` after `PULL` and `WALK` last (match `ORDER`); plan sheet lists `Mat + Band` (`Lift`) and `Walk` (`Cardio`).
  - Mat day: 7 cards in spec order with spec sets; every card has a swap; warm-up `March 4 min, easy. 5 belly breaths.`; time estimate within ±3 min of 53 (Normal), 43 (Light, rest stays 90 s on this day), 58 (Hard).
  - Mat day Hard: `mat_bridge` and `mat_kick` have +1 set.
  - Mat day Add sheet: only Band and Mat groups; `bandpu` absent.
  - `glutesTomorrow()` returns `'mat'` when tomorrow's plan is Mat.
  - Stats: week grid shows 9 columns with `MB` and `WK`; at 360 px (use `run_frame`) no horizontal scroll; a Mat session counts as a workout, not in the "4 lifts" count.
  - Walk tab: no `.mode` picker; card `Walk`, 30 min, 4 rules in spec order (rule 1 red); swap button text `Feel good? Swap to Easy Run`; after swap the card shows `Easy Run`, 25 min, cue `Walk 5 min. Run 1, walk 2, x5.`, and the 4 run rules.
  - `runBlock()` returns each spec toast: Pump or Combat yesterday `Class yesterday. Walk today.`; today `Class today. Walk only.`; Hard yesterday `Hard yesterday. Walk today.`; run yesterday `Ran yesterday. Walk today.`; 2 runs in last 7 days `2 runs this week. Walk today.`; 10-day break `Back after a break. Walk today.`; else `''`. When blocked, the swap button has class `off` and tapping it shows the toast and does not swap.
  - Logging a Run gives a session with `day:'walk'`, `run:1`, minutes stored. Next day `suggestMode().v === 'normal'` and `hardBlock() === ''` (Run is not Hard, no Light after).
  - Rules sheet line 1 is `No HIIT. Easy runs on Walk tab only.`
- [ ] **Step 2: Run it.** Expected: FAIL (`DAYS.mat` undefined).
- [ ] **Step 3: Implement** per spec "Feature 4" and "Feature 5". `statsWeek` decides lift vs class with `DAYS[x].type`, not `LIFTS`. Guard `cardEx` for items without a swap.
- [ ] **Step 4: Run `p1`-`p6`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T6: Mat + Band workout, Walk tab with Easy Run"`

---

### Task 7: Calorie estimate replaces kg volume

**Files:**
- Modify: `index.html` (`PROGRAM` class items and alts get `met`; new kcal helpers near the muscle-count code; Finish sheet tiles ≈1993; Stats workout list ≈2169; Stats weekly grid ≈2112; Guide sheet; any other user-visible use of `dayVolume` / `sess.vol`)
- Test: `QA\p7_kcal.py`

**Interfaces:**
- Consumes: `HKDEF`, `TAGS`, `planOf`, `DAYS.mat`, `DAYS.walk`, session `run`.
- Produces:
  - `KC = { MAT: 3.0, BIG: 5.0, LIFT: 3.5, H: 1.05, BW: 62 }`
  - `kcGroup(hk) -> 'MAT'|'BIG'|'LIFT'`
  - `bwOn(date) -> number` (kg)
  - `kcSet(hk, reps, bw) -> number` (unrounded)
  - `kcMin(met, min, bw) -> number` (unrounded)
  - `kcSession(sess) -> number` (unrounded; works for new and old sessions)
  - `kcToday(day) -> number` (unrounded; done sets and done class items of the open day)
  - `kc5(x) -> number` (nearest 5, minimum 5 when x > 0)

- [ ] **Step 1: Write the failing probe `p7_kcal.py`.** Seed `S.body = [{d: <30 days ago>, w: 62}]`. Checks (values from spec "Feature 7 > Reference numbers", tolerance ±5 unless stated):
  - `kcGroup`: `bandbridge`, `revlunge`, `backext` → `MAT`; `legpress`, `rdl`, `thrust_m`, `thrust_mf` → `BIG`; `latpull`, `chestpress`, `abductor`, `unknown_x` → `LIFT`.
  - `kcSet('bandbridge', 15, 62)` within ±0.3 of 4.3; per-side move doubles only the rep time; `unit:'sec'` uses seconds.
  - `bwOn`: weight on/before date; before the first entry → earliest; empty `S.body` → 62.
  - Full Normal Glutes A logged at program sets/reps → `kc5(kcSession(sess))` = 115 ±5; Light 80 ±5; Pull 95; Mat + Band 85; Pump day Normal 165; Yoga 85; Walk 80; Run (`run:1`) 95.
  - Old v3.7-style session with `cmin` and no item detail computes without error.
  - Finish sheet shows a tile with `kcal` and no `Lifted` / `kg` volume tile; Stats workout rows show `kcal`, not `kg`; weekly grid right header reads `kcal`; Guide shows `≈ N kcal / set` with N = round(kcSet(...)).
  - Per-lift kg charts and "Stronger than 4 weeks ago?" still render.
  - Text check: no visible `Lifted`.
- [ ] **Step 2: Run it.** Expected: FAIL (`kcSet` undefined).
- [ ] **Step 3: Implement** per spec "Feature 7". Class kcal uses planned minutes, not session minutes.
- [ ] **Step 4: Run `p1`-`p7`.** Expected: all PASS, `console errors: 0`.
- [ ] **Step 5: Commit** `git commit -am "v3.8 T7: kcal estimate replaces kg volume"`

---

### Task 8: Release gate

**Files:**
- Modify: `sw.js` (`VERSION` line 2 → 3.8.0), `index.html` (`APP_VER` → 3.8.0)
- Create: `QA_REPORT.md` (written by the QA agent)

- [ ] **Step 1: Bump versions**, run `p1`-`p7`, commit `v3.8.0: version bump`.
- [ ] **Step 2: Wording/declutter agent** (read-only): checks every new or changed UI string against the spec and the 3-8 word rule; returns a fix list. Apply fixes, rerun probes, commit.
- [ ] **Step 3: PT check agent** (read-only): opens the built app's `PROGRAM` / `LIBRARY` / mode rules and confirms numbers and cues match the spec and the medical limits. Apply fixes, rerun probes, commit.
- [ ] **Step 4: QA agent** (must not edit app files): runs `p1`-`p7`, writes its own probes for spec "QA must cover" 1-12 and this plan's Review Focus 1-5, checks 360 / 384 / 412 px, writes `QA_REPORT.md` with verdict PASS / PASS WITH FIXES / BLOCK. Fix and re-run until PASS.
- [ ] **Step 5: Merge and deploy on PASS only.** `git checkout main && git merge --no-ff feat/v3.8-intensity`, commit `QA_REPORT.md`, `git push origin main`. Check `gh auth status` shows khoicorn active first.

# SCULPT v3.8.0: Intensity modes, Mat + Band, no heart-rate checks

## Intent
The user went to the gym late, found every machine taken, walked and went home. It felt right: the day after Pump they were tired. Two things follow:
1. The resting heart-rate (HR) tap is noise for them. Their resting HR is always normal; when they feel bad they do not train. What they need is a way to say how hard today should be: Light, Normal or Hard.
2. When machines are busy, or at home, they want exercises that need only a mat, a mini loop band and a long loop band (~2 m power band). They own both bands.

Plus one bug: "Kneeling Cable Pulldown" looks exactly like "Lat Pulldown" (same stick figure, same movement).

Design came from two PTs working independently (strength coach, and clinical PT for hyperthyroidism and pelvic floor), plus an engineer code map. Where they disagreed, the choice is marked **[my call]**.

## User decisions (2026-11-06)
- Bands owned: mini loop band and long loop band. Nothing else at home except a mat.
- Remove ALL HR checks: the resting HR box, the post-class "HR under 120?" button, HR numbers in rules, warm-ups and cues. Safety stays as plain text rules.
- Mat and band moves go into the Add exercise sheet AND a new full workout "Mat + Band".

## Goals
- Intensity picker (Light / Normal / Hard) replaces the HR box. It changes sets, kg, rest and extras for today.
- Light days never hurt progression. Hard days never break medical limits.
- 16 new mat/band moves, each with its own stick figure.
- New template "Mat + Band", pickable any day like the others.
- Kneeling pulldown replaced by a move that is truly different.

## Non-goals and anti-goals
- No new libraries, no network, no accounts. Single file `index.html`.
- No calorie anything.
- No added text on workout cards beyond what is listed here. UI strings 1 line, 3-8 words. Prefer removing.
- Hard never means: kg above today's suggestion, sets to failure, rest under 90 s, faster pace, jumps, HIIT, breath holds.
- Must not lose done sets when the mode changes mid-workout.

## Medical limits (unchanged, apply in every mode)
Hyperthyroidism. Pelvic floor: exhale on every push, no breath hold, no barbell squat, no heavy deadlift, leg press light with feet high and stop at 90°, existing `max` kg caps stay. No HIIT. Every set stops with 2+ reps left. Rest timer lock stays (skip only after 90 s).

---

## Feature 1: Intensity picker

### State
- `S.mode = { v: 'light'|'normal'|'hard', d: 'YYYY-MM-DD' }` or `null`. Add to `fresh()`.
- `norm()`: delete `s.hr` and `s.calm`. If `s.mode` is not that exact shape, set `null`.
- `modeNow()`: `S.mode.v` if `S.mode.d === todayStr()`, else the suggested mode (below), else `'normal'`.
- When the first set (or class item) of the day is marked done, write the current `modeNow()` into `S.mode` so a computed suggestion cannot drift during the day. The user can still change it by tapping.

### UI
- Same spot and panel as the old `.hr` box (rename class to `.mode`), same 3-column `.opt` grid.
- Label: `How hard today?`. Buttons: `Light` / `Normal` / `Hard`.
- No lock: the current mode is lit by default and sets are open. A tap saves it. It can change any time.
- Suggested Light shows a small second line inside the Light button (reuse `.feel button small` style): `After Pump`, `After Combat` or `After a break`.
- One hint line under the grid, only when Light is on: `Less kg. Progress kept.` Nothing for Normal or Hard.
- Hard greyed out (still tappable): tap shows a toast with the reason and `buzz('block')`:
  - `Pump yesterday. Not today.` / `Combat yesterday. Not today.`
  - `Hard yesterday. Not today.`
  - `2 Hard days this week.`
- Picker hidden on Yoga (and on Walk if added, see Feature 5).
- Finish sheet date line: append ` · Light` or ` · Hard` when not Normal. Stats workout list rows: same suffix. Nothing else in Stats.

### When the app suggests Light
Pre-select Light if either:
1. A Pump (`wed`) or Combat (`sat`) session was logged yesterday.
2. 10 or more days since the last logged session (any type).

Otherwise Normal. Never suggest Hard. Yoga and Mat days never trigger Light the next day. **[my call]** PT A also wanted "Hard yesterday" and "trained 2 days in a row"; PT B showed the second would mark every Saturday Pump; Hard-yesterday is already covered by greying Hard. Left out to keep the rule simple.

### Hard availability
Greyed if any: Pump or Combat logged yesterday; Hard used yesterday; 2 Hard days already in the last 7 days.

### Rules per mode: lift days (Glutes A, Push, Glutes B, Pull, Mat + Band)

| | Light | Normal | Hard |
|---|---|---|---|
| Sets | program − 1 (min 2) | program | +1 set on the 2 "Hard+" moves only |
| Reps (weighted) | same target | same | same |
| Reps (band / bodyweight, `noW`) | × 0.8, round to nearest whole; `sec` units round to nearest 5 | same | same |
| kg | 80% of suggestion, round DOWN to `step`, at least 1 step under suggestion. Assist machines: 1 step MORE assist | suggestion | suggestion, never more |
| Rest | 120 s on all | program (90/120) | program; 120 s on the Hard+ moves |
| Extras allowed | 0 | 2 | 1 |

Hard+ moves (+1 set):
- Glutes A: Hip Thrust Machine, Glute Kickback Machine **[my call: PT A said Abductor; Glutes B already adds side glutes]**
- Push: Chest Press Machine, Lateral Raise Machine
- Glutes B: Cable Hip Abduction, Back Extension
- Pull: Lat Pulldown, Seated Cable Row
- Mat + Band: Long-Band Glute Bridge, Kneeling Band Kickback
- Never add sets to RDL, Leg Press, Angled Leg Press, Bulgarian Split Squat.
- If a Hard+ move is swapped, the swap gets the +1 set.

Extras: the cap replaces the literal 3 (`maxExtra()`). Extras the user already added are never removed when the mode changes. Added cards keep today's set rule (`Math.min(3, d.sets)`), then Light/Hard rules apply like any card. **[my call]** Both PTs set Normal to 2 (was 3) to keep session volume near 22 sets.

### Rules per mode: class days
- **Pump**
  - Light: class rule 2 text becomes `No plates on squats or lunges.` Finisher lift cards are hidden for today (not pending, not counted). Cool-down walk stays.
  - Normal: as today.
  - Hard: finisher cards get 3 sets each (from 2). Same plates.
- **Combat**
  - Light: switch the main class to its alt (Easy Elliptical 20 min) unless the class is started or done. Leaving Light restores the class only if Light made the swap. Post-class lift cards follow the Light lift rules.
  - Normal: as today.
  - Hard: post-class lift cards get 3 sets each (from 2). Class unchanged.
- **Yoga**: no picker. Behaves as Normal (light extras only, max 2).
- Class days keep "light moves only" for extras (`l: 1`).

### Progression (protect auto kg)
- History entries and sessions store `mode` only when not Normal. Missing = Normal (old data).
- `suggest()` stays mode-free and uses `lastBase(hk)`: the last history entry where `mode !== 'light'`. If none, start values. `tr` keeps storing the Normal target.
- New `planOf(c)` applies the mode on top of `suggest(c)` and returns `{ sets, w, r, rest }`. Every place that shows or seeds kg, reps, sets or rest uses `planOf` (cards, `refreshRow`, "Use X kg" button, Guide, time estimate, rest timer, set seeding).
- A Light session never raises or lowers the next suggestion.
- **[my call] Band and bodyweight moves (`noW`) that have `repTop`** (all 16 library moves): when every program set of the last non-Light entry hit the target, next target = +1 rep (`sec` units: +5 s), capped at `repTop`. At `repTop` it stays there. Existing `noW` moves without `repTop` keep today's behaviour.
- A Hard session is judged only on the first `c.sets` (program) sets. Max change +1 step or +1 rep, same as Normal.
- In-session rating: on a Light day, `Easy` does not raise kg; `Hard` still lowers 1 step and adds 30 s.
- 21+ days since the last non-Light entry of a move: suggest 1 step lighter than that entry.
- Strength views skip Light entries: `statsLifts`, "Stronger than 4 weeks ago?", 12-week chart, `lastTxt`. Volume, sets per muscle and minutes count everything.
- Weekly targets (`plannedSets`) stay on program sets. On Light, `recommend()` returns `cat = null` (no push to add extras).

### Mode change mid-workout
- **[my call]** Each done set stores the mode it was done in (`m: 'l'|'n'|'h'`). At log time a history entry gets `mode: 'light'` if ANY of its sets is Light, else `mode: 'hard'` if any set is Hard, else no key. So a mixed session never becomes the base for the next kg. The session's `mode` is the picker value at log time.
- Hard judging uses the first `c.sets` done sets of that entry (it is never Light by the rule above).
- `S.mode` is written on the first done set so a reload keeps today's mode; the picker can still change it after that.
- Done sets are never removed or changed, whatever row they sit on.
- Cards with no done set: reseed from `planOf`.
- Cards with done sets: only add or remove pending rows. Row count never drops below done count.
- Fix first: today `sets()` (≈1159-1161) cuts rows when the target shrinks. That would delete done sets.

---

## Feature 2: Remove every HR check
The engineer map lists every site (`gate`, `recommend`, `renderAdd`, `addExtra`, class swap at ≈1779, `inWorkout`, calm button ≈1567-1571 and handler ≈1849, CSS `.hr`, `.calm`, `.opt.bad`, `li.hrr`). Target behaviour:
- `gate()`: no main class → open. Class done → open (finisher unlocks at once with the existing unlock fade, moved into the "Class done" branch). Class started → `Class in progress`. Card below the class → `Class first`.
- Before-class button: `Start` (was "HR under 120? Start class").
- After Pump is done, the done line reads `✓ Class done. Walk 3 min, then finisher.` plus undo. Other classes: `✓ Class done` plus undo. **[my call]** Both PTs wanted a 3-min locked walk timer and a "Breathing calm?" button. The user chose plain text only, so this is one text line, no lock.
- Class swap button: `No class? Swap to <b>…</b>`.
- `inWorkout()`: `!!S.timer || !!S.start[S.view]` (also fixes the screen staying awake all day).
- Delete the class auto-swap on HR tap (it also reset manual swaps, a bug).
- Search the visible UI for `HR`, `heart rate`, `calm`, `bpm`: 0 hits after the change (the safety rule "heart racing" is allowed).

### New text
Warm-ups:
- Glutes A: `Bike 8 min, easy. Can chat.` Push: `Elliptical 6 min, easy. Can chat.` Glutes B: `Rowing 5 min, damper 3. Can chat.` Pull: `Bike 6 min, easy. Can chat.`
- Pump: `Walk 5 min, flat, easy.`

Pump class rules (rule 1 keeps the red `hrr` style):
1. `Dizzy, chest tight, heart racing = stop.`
2. `Lightest plates. Lunges: no bar.` (Light: `No plates on squats or lunges.`)
3. `Squats half depth. Skip crunches.` (core track crunches load the pelvic floor)
4. `No full sentence? Sit out one track.`

Combat class rules:
1. `Dizzy, chest tight, heart racing = stop.`
2. `Low option every track. No jumps.`
3. `Kicks knee height. Never lock knee.`
4. `No full sentence? March until you can.`

Walks and alts:
- Pump cool-down rule `Leave with HR under 110.` → `Leave when breathing is calm.` Alt cue → `5 min. Easy pace.`
- Combat walk-down rule `Stop when HR is under 110.` → `Stop when breathing is calm.` Alt cue → `5 min. Easy pace.`
- Combat alt (Easy Elliptical) cue → `20 min. Can chat. Or join Yoga.`
- Yoga rules: unchanged.

Rules sheet:
- Menu row small text: `Safety and lifting`.
- "Your week": keep lines; replace `Class days: max 3 light extras.` with `Extras: Light 0, Normal 2, Hard 1.`; add `Gym full? Do Mat + Band.`; move `Combat max 1x per week.` here.
- "Heart" becomes "Safety":
  1. `No HIIT. Easy runs on Walk tab only.`
  2. `Stop every set with 2-3 reps left.`
  3. `Rest timer locks the next set. Skip only after 90 s.`
  4. `Tired? Pick Light, or skip today.`
  5. `Can't speak a full sentence? Slow down.`
  6. `Dizzy, chest tight, heart racing: stop.`
- New short section "Intensity": `Light: 1 set less, lighter kg.` / `Hard: 1 more set on 2 moves.` / `Hard max 2x a week.`
- "Lifting" and "Weights": unchanged.

How it works step 2: `Open today's workout. Pick how hard. Reorder if a class is late.`
`manifest.json` description: drop "Heart-safe"; keep it short.

---

## Feature 3: Mat + band library (16 moves)
New `LIBRARY` array merged into `HKDEF` after it is built from `PROGRAM`, before `load()`. Each move gets a `TAGS` entry.

`pl` values: `'Mini band'`, `'Long band'`, `'Mat'`. Update existing: `abd` and `latwalk` → `'Mini band'`; `bandpu` → `'Long band'` (needs a pull-up bar; exclude from Mat day). `ADDGROUPS`: Machine, Cable, Dumbbell, `Band` (Mini band + Long band, row small text shows which band), `Mat`.

All: `noW` (`'Band'` or `'Body'`), rest 90 s, reps progress up to `repTop`. `l: 1` = allowed on class days.

| # | hk | Name | What | Gear (`vn`) | Cat (p / s) | Muscle (target) | Sets × reps (top) | Side | l |
|---|---|---|---|---|---|---|---|---|---|
| 1 | bandbridge | Long-Band Glute Bridge | On your back. Band over hips. Lift hips. | Mat + long band | Glutes / - | Glute max (glute_max) | 3×15 (20) | no | |
| 2 | slbridge | Single-Leg Glute Bridge | One foot down, one knee hugged. Lift hips. | Mat | Glutes / Legs | Glute max (glute_max) | 3×10 (15) | yes | |
| 3 | bandkick | Kneeling Band Kickback | On all fours. Push one foot back. | Mat + long band | Glutes / - | Glute max (glute_max) | 3×12 (15) | yes | |
| 4 | hydrant | Banded Fire Hydrant | On all fours. Lift one knee out sideways. | Mat + mini band | Glutes / - | Glute med (side) (glute_med) | 2×12 (15) | yes | 1 |
| 5 | clamshell | Banded Clamshell | On your side. Open the top knee. | Mat + mini band | Glutes / - | Glute med (side) (glute_med) | 2×15 (20) | yes | 1 |
| 6 | bandrdl | Band Romanian Deadlift | Stand on band. Hips back, stand tall. | Long band | Glutes / Legs | Hamstring + glute (hams) | 3×15 (20) | no | |
| 7 | birddog | Bird Dog | On all fours. Reach opposite arm and leg. | Mat | Core / Glutes | Deep abs + low back (abs) | 2×8 (10) | yes | 1 |
| 8 | sideplank | Kneeling Side Plank | On your side, knees down. Lift hips. | Mat | Core / Glutes | Side abs (abs) | 3×20 s (30 s), `unit: 'sec'` | yes | |
| 9 | bandpd | Band Overhead Pulldown | Kneel tall. Pull band from overhead to chest. | Mat + long band | Back / - | Lats (back width) (lats) | 3×12 (15) | no | |
| 10 | bandrow | Seated Band Row | Sit, legs long. Band around feet. Row. | Mat + long band | Back / - | Mid back (posture) (upper_back) | 3×15 (20) | no | |
| 11 | proney | Prone Y Raise | Lie face down. Lift arms in a Y. | Mat | Back / Shoulders | Lower traps (posture) (upper_back) | 2×10 (15) | no | 1 |
| 12 | pullapart | Band Pull-Apart | Arms straight ahead. Pull the band apart. | Long band | Shoulders / Back | Rear shoulder (rear_delt) | 2×15 (20) | no | 1 |
| 13 | bandlat | Band Lateral Raise | Stand on band. Lift arms to the sides. | Long band | Shoulders / - | Side shoulder (side_delt) | 3×15 (20) | no | |
| 14 | bander | Band External Rotation | Elbows at sides. Rotate hands out. | Mini band | Shoulders / Back | Rear shoulder (rear_delt) | 2×15 (20) | no | 1 |
| 15 | kneepush | Knee Push-Up | Push-up with knees on the mat. | Mat | Chest / Shoulders | Chest (inner_chest) | 3×8 (12) | no | |
| 16 | bandpress | Band Incline Press | Band behind back. Press up and forward. | Long band | Chest / Shoulders | Upper chest (upper_chest) | 3×12 (15) | no | |

**[my call]** Row: seated (PT A), not bent-over (PT B): floor move, no hinge on a tired day. Kickback: long band (PT B), not mini band donkey kick (PT A): more resistance.

Cues (setup 3 / avoid 2-3):
1. bandbridge. Setup: `Band over hip bones, ends under hands.` / `Heels close, feet hip width.` / `Exhale, lift hips. Hold 2 s.` Avoid: `Arching the lower back on top.` / `Holding breath at the top.` / `Pushing through the toes.`
2. slbridge. Setup: `Hug one knee to your chest.` / `Other heel flat, close to hip.` / `Exhale. Push heel down, lift hips.` Avoid: `Hips dropping to one side.` / `Pushing through the toes.`
3. bandkick. Setup: `Band around one foot, ends under hands.` / `Hands under shoulders, knees under hips.` / `Exhale. Push heel back to hip height.` Avoid: `Arching the lower back.` / `Kicking above hip height.`
4. hydrant. Setup: `Mini band just above knees.` / `Hands under shoulders, knees under hips.` / `Lift knee out to hip height. Pause.` Avoid: `Leaning away from the moving leg.` / `Twisting the hips open.`
5. clamshell. Setup: `Band just above knees.` / `Hips and knees bent. Feet together.` / `Open top knee. Pause 1 s.` Avoid: `Top hip rolling backward.` / `Feet coming apart.`
6. bandrdl. Setup: `Stand on band, feet hip width.` / `Hold top of loop, arms long.` / `Hips back. Exhale, stand, squeeze.` Avoid: `Rounding the back.` / `Squatting instead of hinging.` / `Leaning back at the top.`
7. birddog. Setup: `Hands under shoulders, knees under hips.` / `Exhale. Reach arm and opposite leg.` / `Hold 2 s, keep breathing.` Avoid: `Hips tilting to one side.` / `Leg above hip height.`
8. sideplank. Setup: `Elbow under shoulder. Knees bent.` / `Lift hips to a straight line.` / `Breathe slow the whole hold.` Avoid: `Holding your breath.` / `Hips sagging down.` / `Holding past 30 s.`
9. bandpd. Setup: `Kneel tall. Band overhead, hands wide.` / `Exhale. Pull elbows down to ribs.` / `Stretch band apart to upper chest.` Avoid: `Arching the lower back.` / `Shrugging shoulders up.`
10. bandrow. Setup: `Sit tall. Band around both soles.` / `Exhale. Pull hands to lower ribs.` / `Squeeze shoulder blades 1 s.` Avoid: `Rounding the back.` / `Leaning back to pull.`
11. proney. Setup: `Face down. Forehead on a towel.` / `Arms overhead in a Y, thumbs up.` / `Exhale. Lift arms. Hold 2 s.` Avoid: `Lifting the chest high.` / `Shrugging to your ears.`
12. pullapart. Setup: `Arms straight at shoulder height.` / `Hands shoulder width. Too hard? Wider.` / `Exhale. Band to chest. Squeeze 1 s.` Avoid: `Shrugging shoulders up.` / `Ribs flaring forward.`
13. bandlat. Setup: `Stand on band. One side per hand.` / `Lead with elbows to shoulder height.` / `Lower in 3 s.` Avoid: `Shrugging shoulders up.` / `Swinging the body.`
14. bander. Setup: `Mini band around both wrists.` / `Elbows bent 90°, tucked at sides.` / `Rotate hands out. Pause 1 s.` Avoid: `Elbows leaving your sides.` / `Leaning back to cheat.`
15. kneepush. Setup: `Hands a bit wider than shoulders.` / `Straight line knees to head.` / `Lower slow. Exhale, push up.` Avoid: `Hips sagging or piking up.` / `Elbows flaring out wide.` / `Holding breath at the bottom.`
16. bandpress. Setup: `Band across upper back, under arms.` / `Small split stance. Elbows at 45°.` / `Exhale. Press up to eye level.` Avoid: `Shrugging shoulders up.` / `Arching the lower back.` / `Locking the elbows hard.`

Stick figures (each must look different from its nearest neighbour at thumbnail 54 px and Guide 210 px):
1. bandbridge: side view on back, hips rise to a straight knee-hip-shoulder line, band arc over hips to hands on floor; feet flat and apart (Frog Pump has soles together; DB bridge has a dumbbell).
2. slbridge: only one foot on floor; other knee pulled to chest while hips rise.
3. bandkick: side view on all fours; one leg straight back at hip height; band line from hands to that foot.
4. hydrant: all fours seen from behind (`front: true`); one bent knee lifts out sideways to hip height.
5. clamshell: side-lying, knees bent and stacked, feet touching, top knee opens up (Side-Lying Leg Raise lifts a straight leg).
6. bandrdl: side view, torso hinged ~45°, shins near vertical, arms straight down, band line feet to hands, no dumbbells.
7. birddog: side view all fours, one arm straight forward and opposite leg straight back, both level.
8. sideplank: on one forearm and bent knees, shoulder-hip-knee in one line, top hand on hip; a still hold (small hip lift between A and B).
9. bandpd: front view kneeling tall; arms go from wide Y overhead to W with elbows at ribs; band across chest; no seat, no bar.
10. bandrow: sitting on the floor, legs straight, band from soles to hands, elbows drive back past the body.
11. proney: face down, legs flat, straight arms overhead at 45° lift a little off the mat.
12. pullapart: front view, hands together in front at shoulder height open to a wide T, band across chest.
13. bandlat: front view, both arms rise to shoulder height, two band lines from feet to hands in a V.
14. bander: front view, elbows pinned at ribs bent 90°, forearms open out to a wide V, small loop at wrists.
15. kneepush: side view on knees, straight slanted body line, chest lowers near the floor.
16. bandpress: side view, split stance, band from upper back to hands, arms finish ~30° up at eye level.

Reuse one all-fours base pose for bandkick, hydrant, birddog. No new prop types needed (`cable` with `c: 'bdn'` draws a band, `band` joins two joints).

Left out on purpose (pelvic floor or HIIT): crunches, sit-ups, V-ups, Russian twists, straight-leg raises, leg lowers, hollow holds, full planks over 20 s, bear holds, mountain climbers, jump squats, skaters, burpees, banded jacks, loaded band squats, pike push-ups, band behind the neck.

---

## Feature 4: "Mat + Band" workout
- `PROGRAM` entry: `id: 'mat'`, `tab: 'MAT'`, `tt: 'Lift'`, `type: 'lift'`, `title: 'Mat + Band'`, `sub: 'Gym full or home. Mat + 2 bands.'`, `warm: 'March 4 min, easy. 5 belly breaths.'`.
- Item ids `mat_*`, built from `LIBRARY` defs with these sets. Every item has a swap (avoids the `cardEx` crash on `ex.swap.name`).

| # | Move | Sets × reps | Swap | Hard+ |
|---|---|---|---|---|
| 1 | Long-Band Glute Bridge | 3×15 | Single-Leg Glute Bridge | yes |
| 2 | Banded Clamshell | 2×15 / side | Banded Fire Hydrant | |
| 3 | Kneeling Band Kickback | 3×12 / side | Band Romanian Deadlift | yes |
| 4 | Bird Dog | 2×8 / side | Kneeling Side Plank | |
| 5 | Band Overhead Pulldown | 3×12 | Seated Band Row | |
| 6 | Band Incline Press | 3×12 | Knee Push-Up | |
| 7 | Band Pull-Apart | 2×15 | Prone Y Raise | |

- Order: floor, then kneeling, then standing, so the trainee stands up once (light-headedness on tired days). Move 6 setup cue 1 on this day: `Stand up slowly.` replaces its first cue only here. **[my call]** If this complicates the def, drop it.
- **[my call]** On this day Light keeps rest at 90 s (not 120 s): mat moves are low effort and 120 s made the tired-day session longer.
- Time, by the app's own estimate formula (one-side moves count both sides): Normal 18 sets ≈ 53 min, Light 14 sets ≈ 43 min, Hard 20 sets ≈ 58 min. Same range as the gym lift days (Glutes A ≈ 54 min). Do not change the estimate formula.
- Add sheet on this day shows only Band and Mat rows, and never `bandpu`.
- Wiring (engineer map): `ORDER` adds `'mat'` after `'fri'`; `ABBR.mat = 'MB'`; plan sheet lists it (small text `Lift`); weekly grid CSS `repeat(7,1fr)` → 9 columns with Walk (Feature 5), check 360 px; `statsWeek` uses `DAYS[x].type` not `LIFTS` to tell lift from class.
- Not in `LIFTS`, not in `STANDARD`. It does not count toward "4 lifts". **[my call]** It counts as a workout in Stats.
- `glutesTomorrow()` treats `mat` as a glute day (Glutes A/B spacing).

---

## Feature 5: Walk tab with Easy Run (user approved 2026-11-06)
User: "Walk can include Run, so on OK days I run in the park." Rules by PT B.

- 9th template: `id: 'walk'`, `tab: 'WALK'`, `tt: 'Cardio'`, `type: 'class'`, `title: 'Walk'`, `sub: 'Easy cardio. Park or treadmill.'`, `warm: 'None.'`. `ORDER` adds `'walk'` last. `ABBR.walk = 'WK'`. Weekly grid becomes 9 columns (check 360 px). Plan sheet small text `Cardio`.
- One main class item `walk_walk`: `cls: true, main: true` (match how other main class items are marked), `name: 'Walk'`, `vn: 'Park or treadmill'`, `min: 30`, `icon: 'cardio'`, `move: 'walk'`.
  - Rules: `Dizzy, chest tight, heart racing = stop.` (red, rule 1) / `Easy pace. You can chat.` / `Flat route. Good shoes.` / `Tired? 20 min is enough.`
- Its alt is the run: `name: 'Easy Run'`, `vn: 'Park, flat path'`, `min: 25`, `cue: 'Walk 5 min. Run 1, walk 2, x5.'`. When swapped, the card rules become: `Dizzy, chest tight, heart racing = stop.` / `Run slow enough to talk.` / `Heaviness, leaking or pressure? Stop, walk.` / `Flat path. 2 good runs: +1 min run.` (Builder: if class alts cannot carry their own rules, add an `alt.rules` field.)
- Swap button text on this tab: `Feel good? Swap to <b>Easy Run</b>`.
- Run-walk only, never faster. Progression is the trainee's, by rule 4 (the app shows step 1 only).
- Run not allowed today when any is true. The swap button stays visible but greyed; tap shows the toast and `buzz('block')`:
  - Pump or Combat logged yesterday or today: `Class yesterday. Walk today.` (today: `Class today. Walk only.`)
  - Hard mode used yesterday: `Hard yesterday. Walk today.`
  - A Run logged yesterday: `Ran yesterday. Walk today.`
  - 2 Runs logged in the last 7 days: `2 runs this week. Walk today.`
  - Light suggested today because of a 10+ day break: `Back after a break. Walk today.`
- Logging: the session has `day: 'walk'` and `run: 1` when the Easy Run swap was used. Minutes count to workout minutes.
- No intensity picker on this tab. Walk and Run never count as Hard days and never trigger Light the next day. They do count as "a session" for the 10-day break rule.
- Rules sheet: `No HIIT. Cardio = warm-up only.` becomes `No HIIT. Easy runs on Walk tab only.`
- Stats: counts as a workout (class-type row), not a lift.

---

## Feature 6: Kneeling Cable Pulldown → Half-Kneeling 1-Arm Cable Pulldown
Two-handed kneeling pulldown is the same movement as Lat Pulldown. **[my call, PT A]** Replace it with a one-arm version: longer lat stretch, each side alone, clearly different.
- New swap on Pull day (≈line 936): `hk: 'cablepd1'`, `name: 'Half-Kneeling 1-Arm Pulldown'`, `what: 'Half kneel. Pull one handle down to your side.'`, `vn: 'Cable + single handle, high'`, `muscle: 'Lats (back width)'`, `target: 'lats'`, reps 12, `w: 7.5`, `step: 2.5`, `rest: 90`, `side: true`, `move: 'kneelpd1'`. Sets come from the main item (4) **[my call: per side, so 3 sets would be fair too; builder may set `sets: 3` on the swap if swaps support it]**.
- Guide: Setup `Handle on high cable. Half kneel.` / `Inside knee down, arm reaching up.` / `Exhale. Elbow down to your ribs.` Avoid `Twisting toward the cable.` / `Shrugging the shoulder up.`
- `TAGS.cablepd1 = { p: 'Back', s: [], pl: 'Cable' }`. `TAGS.cablepd` becomes `old: 1` (old history still counts).
- Pose `kneelpd1`: side view, one knee down and the other foot forward (both knees 90°), one arm pulls from high overhead down to the ribs, free hand on the front knee, cable line from a high pulley. Lat Pulldown stays seated with a knee pad.

---

## Feature 7: Calorie estimate replaces kg volume (user asked 2026-11-06)
User: male, 173 cm, 62 kg, hyperthyroid ("only a small boost"). Kcal must be good enough to see "was this day / week a lot or a little", not precise. Two PTs designed it independently, then compared and agreed; where they still differed, the Compendium code was used. This reverses the 2026-11-04 "no calorie UI" decision for estimates only (no food or intake logging).

### Formula
- Net kcal, like a watch's "active kcal": `kcal = (MET - 1) × bw × hours × 1.05`.
- One set: `t = reps × 4 s × (side ? 2 : 1) + 60 s` (`unit: 'steps'`: 2 s per step; `unit: 'sec'`: the seconds as given). Rest beyond 60 s is not counted. No kg factor. No mode factor (Light / Hard change sets and reps, kcal follows).
- Lift groups (gross MET), checked in this order:
  1. `MAT 3.0`: every `noW` move (band, mat, bodyweight, incl. all 16 library moves, `revlunge`, `backext`).
  2. `BIG 5.0`: `TAGS[hk].s` includes `'Legs'`, plus `thrust_m`, `thrust`, `thrust_mf`.
  3. `LIFT 3.5`: everything else, incl. unknown and legacy keys.
- Time items on planned minutes (`clsLen` / item `min`), gross MET: Pump 3.5, Light Circuit 2.5, Combat 4.8, Easy Elliptical 4.0, Yoga 2.3, Home Mobility 2.3, Stretch 2.3, cool-down walks / bikes / Walk-down / Slow Walk 3.5, Walk 3.5, Easy Run 4.5. Put a `met` field on each class item and alt.
- Pump finisher and Combat post-class lifts use the set formula. Warm-ups are not counted.
- Body weight: latest `S.body` `w` on or before the session date; else the earliest entry; else 62.
- Old sessions are computed the same way: lift sets from `S.history` entries of that date; class part from `cmin` × the main class MET (title ending ` · swap` → alt MET; `run: 1` → 4.5). Kcal is computed, not stored.
- Rounding: sessions and weeks to the nearest 5 (minimum 5 when above 0). Guide per set to the nearest 1. Always sum unrounded values.

### Reference numbers at 62 kg (QA checks these within ±5)
- Lift days Normal / Light: Glutes A 115 / 80, Push 85 / 60, Glutes B 120 / 85, Pull 95 / 70, Mat + Band 85 / 60.
- Pump class 120, Pump day Normal 165, Light 135. Combat class 185. Yoga 60: 85. Walk 30: 80. Easy Run 25: 95.
- One band set (MAT, not per side) 15 / 20 / 25 / 30 reps: ≈ 4 / 5 / 6 / 7 kcal. Hip Thrust Machine 4×12 ≈ 30. Lat Pulldown 4×12 ≈ 20.

### Where it shows (no new text on workout cards)
- Finish sheet: the `Lifted … kg` tile becomes kcal (`≈ 115 kcal`).
- Stats workout list: the kg column becomes kcal.
- Stats weekly grid: the right-hand `sets` column becomes `kcal` (week total).
- Any other user-visible kg volume number becomes kcal. The per-lift kg charts and "Stronger than 4 weeks ago?" stay (strength, not volume).
- Exercise Guide: one line `≈ 6 kcal / set`, from `planOf` reps and today's weight.

---

## Release
- Version 3.8.0: `VERSION` in `sw.js` and `APP_VER` in `index.html`.
- Order: build → wording/declutter review → PT check of the built app (cues, numbers) → QA agent writes `QA_REPORT.md` → deploy only on PASS.
- Old QA probes in the previous session scratchpad tap `[data-hr]` and `[data-calm]`; QA rewrites what it reuses.
- `t7.html` (untracked, stale copy of v3.7) is not part of the release.

## QA must cover
1. Mode switch after done sets: no done set lost on any row; pending rows added or removed right.
2. Light log does not change next Normal kg or reps; Easy on Light does not raise kg; Hard moves progress once; `tr` stays the Normal target; Light entries absent from strength charts.
3. Migration: v3.7 data and an old backup with `hr`/`calm` load; keys gone; Export has no `hr`/`calm`; bad `mode` → `null`; old sessions count as Normal.
4. Class flow: `Start`, `Class in progress`, `Class first`, class done opens finisher at once with fade, undo, "Class late? Exercises first", Combat Light auto-swap and restore.
5. Rest seconds per mode; skip floor 90 s; Hard rating still adds 30 s.
6. Suggestion: day after Pump/Combat Light is lit with `After Pump`/`After Combat`; not after Yoga or Mat; 10-day break; across week change; resets at midnight; Hard grey reasons.
7. Add sheet: no HR banners; extras cap by mode; class days light-only; Mat day only band and mat, no `bandpu`.
8. Every library move: card, Guide (cues, video search link), `sec`/side display, log, Lifts tab, survives reload and import.
9. Mat day: tab, plan pick, week slot `MB`, 8-column grid at 360/384/412 px, no crash, time estimate.
10. Stick figures: all 17 new poses render, differ from neighbours, animate.
11. UI has 0 visible `HR`/`heart rate`/`calm`; picker fits at 360 px; tap targets ≥ 44 px.
12. Wake lock not held all day.

## Left to the builder
Code structure of `planOf`, `lastBase`, `maxExtra`, `LIBRARY`; exact pose coordinates; CSS details; how hidden Pump finisher cards are filtered; rounding helpers; which old QA probes to reuse.

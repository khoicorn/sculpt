# SCULPT v3.10.0: dumbbells first at the gym, fewer band moves

## Intent
User (2026-11-09): prefers dumbbells over bands for chest / push and other moves ("easier"), glutes with dumbbells are fine too; add the best moves, many is OK; remove fiddly band moves to keep the app clean. Saw people standing and circling dumbbells with both arms; not sure which move. He has NO dumbbells at home, so the home Mat + Band day keeps bands. Two PTs worked independently; differences settled toward the safer option (marked **[call]**).

## Remove 5 band moves (keep old history readable: move defs to `RETIRED`, `TAGS[hk].old = 1`)
`bandpress` (band behind the back), `bandkick` (band under hands on all fours), `bandbridge` (band pinned under hands), `bandpu` (needs a bar; its def is inline in `fri_apu` alts), `bandlat` (no card needs it).
Keep: mini band `clamshell, hydrant, latwalk, abd, bander`; long band `bandrdl, bandpd, bandrow, pullapart`; mat `slbridge, frogpump, sidelying, birddog, sideplank, deadbug, proney, kneepush, revlunge`.

## New dumbbell moves (11)
All `pl: 'Dumbbell'` (never on the Mat day), `move` = hk, rest 90 unless stated. `pair: true` = 2 dumbbells, kg each. Caps (`max`) are kg per hand; at the cap add reps up to `repTop`.

| hk | name | what | vn | TAGS p / s / l | target, muscle | sets × reps (top) | w / step / max | other |
|---|---|---|---|---|---|---|---|---|
| pullover | DB Pullover | Lie on a bench. Arc one dumbbell overhead. | Flat bench + 1 dumbbell | Back / Chest | lats, Lats (back width) | 3 × 12 (15) | 6 / 2 / 12 | one DB in both hands |
| csrow | Chest-Supported DB Row | Chest on an incline bench. Row both dumbbells. | Bench 45° + 2 dumbbells | Back / – | upper_back, Mid back (posture) | 3 × 12 (15) | 6 / 1 / – | pair |
| incy | Incline DB Y Raise | Chest on a bench. Raise dumbbells in a Y. | Bench 30° + 2 light dumbbells | Back / Shoulders / l | upper_back, Lower traps (posture) | 2 × 12 (15) | 1 / 1 / 4 | pair |
| dber | Side-Lying DB External Rotation | Lie on your side. Rotate a light dumbbell up. | Mat + 1 light dumbbell | Shoulders / Back / l | rear_delt, Rotator cuff | 2 × 15 (20) | 1 / 1 / 4 | side |
| arnold | Seated Arnold Press | Sit upright. Turn palms out as you press. | Bench upright + 2 dumbbells | Shoulders / – | side_delt, Shoulders | 3 × 10 (12) | 3 / 1 / 8 | pair, rest 120 |
| lyinglat | Side-Lying DB Lateral Raise | Lie side-on on a tilted bench. Raise one arm. | Bench 30° + 1 light dumbbell | Shoulders / – | side_delt, Side shoulder | 3 × 12 (15) | 2 / 1 / 6 | side |
| squeeze | Incline DB Squeeze Press | Press two dumbbells together, up from a tilted bench. | Bench 30° + 2 dumbbells | Chest / Shoulders | inner_chest, Inner + upper chest | 3 × 12 (15) | 4 / 1 / 10 | pair |
| aroundw | Lying DB Around the World | Lie on a bench. Circle dumbbells hips to overhead. | Flat bench + 2 light dumbbells | Chest / Back / l | upper_chest, Chest + shoulder range | 2 × 10 (12) | 2 / 1 / 4 | pair |
| suitcase | Suitcase Carry | Walk tall holding one dumbbell at your side. | 1 dumbbell + clear floor | Core / – | abs, Side abs (posture) | 2 × 30 s (40 s) | 8 / 2 / 14 | `unit: 'sec'`, side |
| kstrdl | DB Kickstand RDL | Back toe down for balance. Hinge on the front leg. | 2 dumbbells | Glutes / Legs | glute_max, Glute + hamstring | 3 × 10 (12) | 6 / 2 / 10 | pair, side, `hinge: 1`, rest 120 |
| slthrust | Single-Leg DB Hip Thrust | Upper back on a bench. Lift hips on one leg. | Bench + 1 dumbbell | Glutes / – | glute_max, Glute max | 3 × 10 (12) | 4 / 2 / 12 | side |

**[call]** Not added: DB Halo (PT B: neck poke and rib flare behind the head, little value) and standing DB full lateral raise / around the world (PT A: long lever overhead while standing, rib flare). The circle he saw is offered as the lying version `aroundw` (back on the bench, 2-4 kg) and the seated `arnold`.

Cues (setup 3 / avoid 2-3, copy exactly):
- pullover: `Lie along the bench. Feet flat.` / `Cup one dumbbell. Elbows slightly bent.` / `Lower to your ears. Exhale, pull back.` · `Lowering past your ears.` / `Lower back arching off the bench.` / `Holding breath at the stretch.`
- csrow: `Bench at 45°. Chest on the pad.` / `Arms hang straight. Palms face in.` / `Exhale. Elbows back. Squeeze 1 s.` · `Lifting chest off the pad.` / `Shrugging shoulders to ears.` / `Jerking the weight up.`
- incy: `Bench at 30°. Chest on the pad.` / `Thumbs up. Arms in a Y.` / `Exhale. Lift to ear height. Hold 1 s.` · `Shrugging shoulders to ears.` / `Lifting chest off the pad.` / `Going heavy: 1-4 kg is enough.`
- dber: `Lie on side. Towel under top elbow.` / `Elbow at 90°, pinned to ribs.` / `Exhale. Rotate hand up. Lower in 3 s.` · `Elbow leaving your side.` / `Rolling the body back.` / `Going heavy: 1-4 kg is enough.`
- arnold: `Bench upright. Back on the pad.` / `Dumbbells at chin, palms facing you.` / `Exhale. Turn palms out, press up.` · `Arching off the pad.` / `Pressing through a shoulder pinch.` / `Holding breath at the top.`
- lyinglat: `Bench at 30°. Lie on your side.` / `Top arm hangs in front, slight bend.` / `Exhale. Lift to just below vertical.` · `Shrugging toward the ear.` / `Rolling the body back.` / `Swinging the weight up.`
- squeeze: `Bench at 30°. Palms face each other.` / `Press dumbbells together over chest.` / `Exhale. Press up, keep squeezing.` · `Dumbbells drifting apart.` / `Elbows flaring out wide.` / `Holding breath while squeezing.`
- aroundw: `Lie on a flat bench. Feet flat.` / `Dumbbells by hips, palms up.` / `Sweep wide to overhead. Exhale back.` · `Lower back arching off the bench.` / `Locking the elbows straight.` / `Going heavy: 2-4 kg only.`
- suitcase: `Dumbbell in one hand. Stand tall.` / `Shoulders level. Ribs down. Short steps.` / `Breathe normally the whole time.` · `Leaning toward the dumbbell.` / `Holding breath or bracing hard.` / `Going heavier than 14 kg.`
- kstrdl: `Front foot flat. Back toes by front heel.` / `Most weight on the front leg.` / `Hips back, back flat. Exhale, stand.` · `Rounding the back.` / `Pushing up with the back leg.` / `Holding breath. Stay light.`
- slthrust: `Upper back on bench edge. Foot flat.` / `Dumbbell on working hip. Other knee up.` / `Exhale. Drive heel, hips up. Hold 1 s.` · `Hips tilting to one side.` / `Arching the lower back on top.` / `Holding breath at the top.`

Stick figures (each must differ from its neighbour at 54 px and 210 px):
- pullover: side view lying along a flat bench; straight arms from above the chest (A) back to beside the ears (B); one DB in both hands.
- csrow: side view, chest on a 45° bench; arms hang (A), elbows drive back to the ribs (B). dbrow is a flat torso with one hand on a bench.
- incy: same bench as csrow; straight arms rise from hanging (A) to in line with the body past the head (B).
- dber: side-lying on the floor like clamshell; only the top forearm turns from across the belly (A) to pointing up (B); elbow stays on the ribs.
- arnold: front view seated; A elbows together in front of the chin, forearms up; B pressed overhead, arms wide. dbspress starts with elbows wide at ear height.
- lyinglat: body diagonal on a bench, lying on the side; the top arm sweeps from hanging across the body (A) to almost vertical (B).
- squeeze: like incpress, but the DBs touch at the midline the whole rep and move straight up.
- aroundw: seen from above, lying; arms start at the hips (A) and sweep out wide to above the head (B) in a half circle.
- suitcase: front view, upright mid-stride; one arm hangs with a DB, the other slightly out; legs alternate.
- kstrdl: side view like rdl, but the back foot is half a step behind with only the toe down (base: the unused `slrdl` pose if present).
- slthrust: hip thrust bench like `hipthrust`, but the far knee is pulled up and only the near foot pushes.

kcal: new weighted moves fall in LIFT; `kstrdl` is BIG (has Legs); add `slthrust` to the BIG hip-thrust list.

## Card changes
**Main card:** `tue_cpress` main becomes `incpress` (both PTs: 30° DB press builds the upper chest line better than the flat machine; he prefers DBs). Set `max: 12`. `chestpress` becomes an alternative (full def in its entry).

**Alternatives (best first). Glute cards' and others' lists not named here stay as in v3.9 minus removed band moves.**
| Card | alts |
|---|---|
| mon_thrustm | thrust, bridge, slbridge |
| mon_kickm | kick, slthrust |
| mon_abdm | cableabd, abd, latwalk |
| mon_abs | suitcase, pallof, birddog, sideplank |
| tue_cpress (main incpress) | chestpress, squeeze, kneepush |
| tue_pec | incfly, squeeze, cablefly_lh_l |
| tue_spress | dbspress, arnold |
| tue_latm | latraise, lyinglat, cablelat |
| tue_face | reardelt, incy, dber, proney |
| thu_rdl | kstrdl, pullthrough |
| thu_bss | stepup, revlunge |
| thu_alp | legpress, gobletsq |
| thu_hipabd | abductor, latwalk, abd |
| thu_backext | bridge, thrust, slbridge |
| fri_lpd | pullover, cablepd1 |
| fri_apu | cablepd1, pullover |
| fri_row | csrow, dbrow |
| fri_face | reardelt, incy, dber, proney |
| fri_fly | aroundw, cablefly_lh_l |
| fri_lat | lyinglat, cablelat, latmach |
| wed_ht | frogpump |
| wed_ab | latwalk, abd, sidelying |
| sat_fp | reardelt, incy, dber, proney |
| sat_sl | abductor_f, abd, latwalk |

Move inline defs as needed (e.g. `stepup` from `thu_alp` to `thu_bss`; `abd`, `pallof` keep their object defs where listed first).

**Mat + Band day (home, 7 cards, ~45 min, PT B order: floor → all fours → kneeling → standing):**
1. `mat_bridge`: slbridge, grp Hip thrust, 3 sets, hp 1; alts frogpump
2. `mat_clam`: clamshell, grp Side glutes, 2 sets; alts hydrant, sidelying, latwalk, abd
3. `mat_bird`: birddog, grp Core, 2 sets; alts deadbug, sideplank
4. `mat_press`: kneepush, grp Chest press, 3 sets; no alts
5. `mat_pd`: bandpd, grp Vertical pull, 3 sets; alts bandrow
6. `mat_lunge` (new id): revlunge, grp Lunge, 3 sets, hp 1; alts bandrdl. First standing move: its first cue on this day only is `Stand up slowly. Light hand on wall.` (`revlunge` must be readable by `matItem`.)
7. `mat_pull`: pullapart, grp Rear delts, 2 sets; alts bander, proney
Card `mat_kick` is removed. Old sessions still show by hk.

## Rules sheet
"Lifting" gets one line: `Dumbbells overhead: seated, back on pad.`

## Migration (`norm`, safe twice; light effort: personal app, phone is on v3.9)
- Cards whose main hk changed (`tue_cpress`: chestpress → incpress; `mat_bridge`: bandbridge → slbridge; `mat_press`: bandpress → kneepush): rows under `S.sets[id]` move to `S.sets[id + '~' + oldMain]`; a swap or rows on the new main hk (`id~newMain`) move to the main key, swap → 0. Keep a small map `MAINCHG` for this.
- Swaps pointing at a removed or no-longer-listed hk → 0 (main); their rows stay under `id~hk` and still log under that hk.
- Removed card `mat_kick` and other unknown ids: drop (no crash).
- v3.8 backups (`SWAP38`): best effort, no crash; v3.8 `fri_apu: 'bandpu'` and `mat_kick` rows may be dropped.

## Version
`APP_VER` 3.10.0, `sw.js` `VERSION` `sculpt-v3.10.0`.

## Light QA
Run p1-p9 (update checks that named removed moves or old lists), plus `p10_db.py`: every new move renders (card, Guide, Add sheet Dumbbell group, sheet row), lists match this spec, removed band moves are gone from Add sheet and lists but old history still shows by name, Mat day 7 cards with the right sets and ~45 min estimate, `tue_cpress` main is Incline DB Press with old chestpress rows migrated, kcal of a new move is > 0, 360 px no horizontal scroll.

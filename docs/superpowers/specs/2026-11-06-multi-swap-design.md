# SCULPT v3.9.0: pick among several swaps of the same purpose

## Intent
User (2026-11-06): "For the swap part, can you re-categorize, so I can choose among several alternatives when I swap? Only ones that are alike, with the same purpose." Today each exercise card has exactly one swap. Two PTs grouped every move by purpose (same main muscle + same movement pattern) and chose 1-5 alternatives per card; they agreed on almost everything, and the remaining 4 differences were settled (marked **[call]**). An engineer mapped the code (data model below).

## Non-goals
- Class main items (Pump, Combat, Yoga, Walk → Easy Run) keep their single alt and current behaviour (`'use'` / `'mode'` / 0).
- No new exercises. No change to the Add sheet groups (muscle → equipment).
- No new text on cards beyond the strings below.

## Function groups (used for the sheet's note line)
Hip thrust: thrust_m, thrust_mf, thrust, bridge, bandbridge, slbridge, frogpump, backext · Hinge: rdl, pullthrough, bandrdl · Kickback: glutemach, kick, bandkick · Leg press: legpress, anglepress, gobletsq · Lunge: bss, revlunge, stepup · Side glutes: abductor, abductor_f, abd, cableabd, latwalk, sidelying, hydrant, clamshell · Core: deadbug, pallof, birddog, sideplank · Chest press: chestpress, incpress, bandpress, kneepush · Chest fly: pecdeck, incfly, incfly_l, cablefly_lh_l · Shoulder press: shoulderpress, dbspress · Side delts: latmach, latraise, cablelat, bandlat · Rear delts: facepull, facepull_l, reardelt, pullapart, proney, bander · Vertical pull: latpull, cablepd1, assistpu, bandpu, bandpd · Row: cablerow, dbrow, bandrow.

Each card gets `grp` = its group name. Where a card's list borrows from another group (marked ~), the note still shows the card's own group.

## Alternatives per card (best first; first = today's swap unless noted)
| Card | grp | alts |
|---|---|---|
| mon_thrustm | Hip thrust | thrust, bridge, bandbridge, slbridge |
| mon_legpress | Leg press | gobletsq, anglepress |
| mon_kickm | Kickback | kick, bandkick |
| mon_abdm | Side glutes | abd, cableabd, latwalk, hydrant, clamshell |
| mon_abs | Core | pallof, birddog, sideplank |
| tue_cpress | Chest press | incpress, bandpress, kneepush |
| tue_pec | Chest fly | incfly, cablefly_lh_l |
| tue_spress | Shoulder press | dbspress |
| tue_latm | Side delts | latraise, cablelat, bandlat |
| tue_face | Rear delts | reardelt, pullapart, proney, bander |
| thu_rdl | Hinge | pullthrough, bandrdl |
| thu_bss | Lunge | revlunge, stepup |
| thu_alp | Leg press | legpress, gobletsq, stepup~ (today's swap stepup moves last) |
| thu_hipabd | Side glutes | latwalk, abductor, abd, sidelying, hydrant |
| thu_backext | Hip thrust | bridge, thrust, bandbridge, slbridge **[call: PT B; a second hinge next to RDL is not wanted]** |
| fri_lpd | Vertical pull | cablepd1, bandpd, bandpu |
| fri_row | Row | dbrow, bandrow |
| fri_apu | Vertical pull | cablepd1, bandpd, bandpu **[call: PT B; band is ~3× harder than 50 kg assist, risks breath hold, so last]** |
| fri_face | Rear delts | reardelt, pullapart, proney, bander |
| fri_fly | Chest fly | cablefly_lh_l **[call: PT A; pecdeck would bring Push-day kg into the easy slot]** |
| fri_lat | Side delts | cablelat, latmach, bandlat |
| wed_ht | Hip thrust | frogpump, bandbridge |
| wed_ab | Side glutes | latwalk, abd, sidelying, clamshell, hydrant |
| sat_fp | Rear delts | reardelt, pullapart, proney, bander |
| sat_sl | Side glutes | abductor_f, abd, latwalk, clamshell, hydrant (drop `swapAsk: 'No mat?'`) |
| mat_bridge | Hip thrust | slbridge, frogpump |
| mat_clam | Side glutes | hydrant, sidelying, latwalk, abd |
| mat_kick | Kickback | bandrdl~, slbridge~ |
| mat_bird | Core | deadbug, sideplank **[call: PT B; same opposite arm-leg pattern, no hold]** |
| mat_pd | Vertical pull | bandrow~ |
| mat_press | Chest press | kneepush |
| mat_pull | Rear delts | proney, bander |

Existing inline swap defs (e.g. today's `thrust`, `reardelt`, `cablepd1` objects) become the alt entries; hk strings resolve through `HKDEF`. Entries may be an hk string or `{ hk, ...overrides }`.

TAGS changes: `reardelt` and `bandbridge` get `l: 1` (light enough for class days); `rdl` and `pullthrough` get `hinge: 1`.

## Rules for a chosen alternative in a slot
1. Sets = the slot's sets (not the alternative's own). A per-side alternative in a two-sided slot gets `min(slot sets, 3)`; a per-side slot keeps its sets. (Changes: `mat_bird` → sideplank 3 → 2 sets; `fri_row` → dbrow 4 → 3 per side.)
2. Reps, repTop, unit, kg, step, `max` cap, rest (alt's, else slot's), progression and history come from the alternative's own hk. Never copy kg or caps across.
3. Hard: +1 set when the slot has `hp: 1` and the alternative has no `hinge: 1` (after the per-side cap). Class-day cards: Hard = `max(3, slot sets)`. Light: the usual rules (sets −1 min 2, kg 80%, band reps ×0.8; Mat keeps 90 s rest; Pump Light finisher hidden unless a set is done on any version).
4. One hk once per day: an alternative already on screen on another card, or added as an extra, is disabled in the sheet (`Today`). The Add sheet blocks only the on-screen version and versions with done sets of each card.
5. Switching between any versions never loses done sets. Each version's done sets are logged under its own hk (`swap: true`), and all count in the HUD, progress, muscle sets and kcal.
6. "My gym has no X" (Guide, not on Mat day) sets the default to the on-screen alt (or the first alt when on main). While a default is set, picking another alt in the sheet makes that the new default.

## Data model (engineer map)
- `S.swaps[id]` for exercise cards = chosen hk, or 0 = main. Class cards unchanged.
- Rows per version: `S.sets[id]` (main) and `S.sets[id + '~' + hk]` (each alt). A switch never deletes a key; Finish and Reset delete `id` and every `id~*` key.
- `asAlt(ex, hk)` builds the card def: slot gives `id`, sets (rule 1), `hp`, `from`, `isSwap`; alt gives hk, name, what, vn, move, muscle, target, reps, repTop, w, step, max, pair, assist, noW, side, unit, hinge, guide, rest; clear `swap/alts/swapAsk/added`.
- Replace `isSwapped / asSwap / otherCur / otherDone` with `altHk(ex)`, `asAlt(ex, hk)`, `others(ex)` (all version keys except the on-screen one); keep `otherDone(ex)` as a flat list for counters.
- Places to update: `offToday`, `dayVolume`, `liveCat`, `kcToday`, `dayPbs`, `logSession` (one history entry per version), `resetDay`, the `#feel` handler (def from the ref key; only relabel on-screen rows), undo of a done set (compare `key` too), `relabelTimer` / `nextLabel` after a switch, `todaySet`, `DAYHK` (add every alt hk), `openGuide`, `matItem`, `xItem` (`alts: null`). Delete unused `HKNAME`, `HKASSIST`.
- `norm()` migration (safe to run twice; load and Import): `'use'` → `SWAP38[id]` (fixed map of the v3.8 single swaps) if still in the list, else `alts[0]`; `'mode'` or unknown hk on exercise cards → 0; `S.sets[id + '~s']` → `id + '~' + SWAP38[id]` (append if the target exists); `S.missing[id] === true` → that hk; `timer.ref.key` / `rate.key` ending `~s` renamed the same way. Helpers used by `norm` must be defined above `let S = load()`.
- Build check at startup (console warn + drop entry): hk exists, not the main hk, no duplicate canon hk in one card, TAGS exists, `MOVES[move]` exists, class-day alt is `l: 1` or a light twin, Mat-day alt is band/mat and not `bar`.

## UI (short strings)
- Card row, main on screen: 1 alt → `Busy? Swap to <b>X</b>` (unchanged, 1 tap). 2+ alts → `Busy? Swap: <b>N options</b>` (opens the sheet). Mat day: no question (`Swap: <b>N options</b>` / `Swap to <b>X</b>`).
- Card row, alt on screen: `Back to <b>Main</b>` (1 tap) plus, with 2+ alts, a narrow `Change` button on the right that opens the sheet.
- Done line: 1 other version `✓ 2 done on X` (unchanged); 2+ → `✓ 3 done on 2 other moves`.
- Sheet `mSwap` (`#swapBox`): `<h2>Swap to</h2>`, note line = the card's `grp`. Rows like Add-sheet `.xrow`: thumbnail, name, small `vn · Last 12 kg`, badge, chevron. Order: main first, then alts in PT order. Badges: `Now` (gray, disabled) on screen; `Today` (gray, disabled) in use elsewhere; `✓ 2` (lime) done on this version today. Tap: switch, save, close, refresh card and timer label; toast `Swapped: X` or `Back to X`.
- Guide: on an alt, `Main: <name>` (unchanged); on main, 1 alt → `Swap: X`, 2+ → `Swap: N options` (switches to the sheet).

## Test cases that matter (probe `p9_swaps.py`)
1. Glutes A, Hip Thrust Machine, Normal: 1 set on main, 1 on alt A, 1 on alt B, back to main → main shows 1 done + 1 pending, HUD 3/4, `✓ 2 done on 2 other moves`; reload after each step keeps it.
2. Log → 3 history entries (thrust_m, A, B) with 1 set each, alts `swap: true`; session sets 3; no `mon_thrustm*` keys left.
3. Progression per hk: next session alt B moves, main unchanged.
4. Hard: every alt of a Hard+ card shows +1 row; Mat kickback on bandrdl stays at slot sets; Combat post-class on Hard shows 3.
5. Light: reseed per version; Pump Light finisher with 1 done set on an alt stays visible.
6. kcal: Finish = sum over versions = Stats after log.
7. Migration from a v3.8 state (`~s` key with 2 done, swaps `'use'`, missing `true`, timer ref `~s`) → `~thrust`, `'thrust'`, ref fixed, 2 done shown; twice = same; Import = same.
8. Timer and rating across a switch: rating applies to the version that was done; undo on another version does not end the rest; label names the on-screen version.
9. Sheet: `Now`, `Today`, `✓ N` badges; disabled rows do nothing; class main cards unchanged (Pump Light Circuit, Combat Light auto-swap, Walk → Easy Run block).
10. 360 px: swap row and sheet rows fit, no horizontal scroll.

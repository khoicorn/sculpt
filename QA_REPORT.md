# QA Report: SCULPT v3.10.0

**Date:** 2026-11-09
**Verdict:** PASS (light QA: personal app)

## Change
Dumbbells first at the gym: 11 new dumbbell moves (incl. 2 glute moves and the lying "around the world" circle), Push main is now Incline DB Press, gym swap lists use dumbbells instead of bands, 5 fiddly band moves removed (old history kept), new 7-card Mat + Band day at home, one new Rules line.

## Probe results (headless Chrome, `scratchpad\qa38\`)
| Probe | Result |
|---|---|
| p1-p9 (updated for v3.10 lists) | all PASS, 0 console errors |
| p10_db (new moves, lists, removed moves, Mat day, migration, kcal, 360 px) | 67 PASS, 0 console errors |
| p11_dbposes (11 new stick figures, worktree run) | 0 FAIL, 0 console errors |

## Known
- Mat + Band time estimate shows Normal ~55 min, Light ~45 min (3 cards are per side).
- Incline DB Press and Squeeze Press look close at thumbnail size; clear in the Guide.

## Not tested
No real-device test.

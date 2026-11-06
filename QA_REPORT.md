# QA Report: SCULPT v3.8.0

**Date:** 2026-11-06
**Verdict:** PASS (light QA, per user: personal app)

## Probe results (headless Chrome, `scratchpad\qa38\`)
| Probe | Area | Result |
|---|---|---|
| p1_hr | HR removed, picker, migration, text | 128 PASS, 0 console errors |
| p2_engine | Light / Normal / Hard sets, kg, rest, progression | 144 PASS, 0 console errors |
| p3_suggest | Suggested Light, Hard lock, class-day modes | 139 PASS, 0 console errors |
| p4_library | 16 mat/band moves, 1-arm pulldown | 112 PASS, 0 console errors |
| p5_poses | 17 stick figures | 287 PASS, 0 console errors |
| p6_days | Mat + Band day, Walk / Easy Run | 152 PASS, 0 console errors |
| p7_kcal | kcal estimate | 128 PASS, 0 console errors |
| p8_fixes | 25 review fixes, 360/384/412 px | 160 PASS, 0 console errors |

## Fixed from the partial deep QA run
- Major: a workout running past midnight switched mode (Light/Hard) mid-session; a tap on a stale screen could log the wrong kg or throw. Fix: an open workout keeps its mode; a tap on a screen drawn before midnight redraws first; returning to the app redraws for the new day.

## Known, not fixed (cosmetic)
- Some long strings wrap to 2 lines at 360 px (e.g. "Band Overhead Pulldown" in the Add sheet, the Guide start line, a few swap buttons). No horizontal scroll.
- A mode change mid-card keeps pending rows at their old kg; the "Use X kg" button offers the new plan kg.

## Not tested
Deep QA was stopped at the user's request. No real-device test.

# QA Report: SCULPT v3.11.1

**Date:** 2026-11-10
**Verdict:** PASS (independent QA agent + own probes; 10 defects found, 9 fixed and rechecked, 1 kept by design)

## Change
Weekly planner and gamification:
- **Week:** core week is 2 Pump (or 3) + 1 Combat, with Tue and Sun rest. Mon is Push/Pull and alternates weekly. Fri is an optional Flex Day with a one-tap "Take Friday off".
- **Readiness:** daily check-in (feel, soreness, where) that drives Light/Hard advice.
- **XP:** levels, a weekly streak, 4 weekly quests, 10 badges, and capped XP for extra sets.
- **Storage:** XP is computed from sessions and never stored. Sessions get stable ids.
- **Migration:** past weeks are pinned to the old plan.

## v3.11.1 hotfix
On the phone, the Week view drew done items as 56 px circles. The new class name `ok` clashed with the round set button `.ok`. It is renamed to `yes`. Past weeks no longer show plan warnings or the "Room for more glutes" tip. p13 now checks for round or squeezed rows, and screenshots at 360 and 412 px were checked by eye.

## Probe results (headless Chrome)
| Probe | Result |
|---|---|
| p12_planner main (30 spec criteria) | 73 PASS, 0 console errors |
| p12_planner fri (clock on a Friday) | 12 PASS, 0 console errors |
| p13_width (360 / 384 / 412 px) | all PASS after the plan-list grid fix |
| p14_fixes (QA defects D1-D10) | 14 PASS, 0 console errors |
| Independent QA probes i1-i7, rerun after fixes | 124 PASS; 4 FAIL, all explained below |
| Old suites p1-p10 (p1-p3, p6-p9 rerun on a mocked Monday) | p2, p5, p7, p9, p10 all PASS. Remaining fails are intended changes (below). |

## Intended changes that old probes flag
- Hard is blocked the day before a planned Pump or Combat, and on Okay, Poor or sore days (PT rule).
- Combat no longer turns Light after Pump. "After Combat" Light was removed, so Combat is not swapped for the elliptical every week.
- Plan sheet rows show time and muscle. The "Lifts x/y · Classes x/y" line and "N/4" copy were replaced by the core goal.
- Rules and How-it-works copy were updated. slbridge and kick are now light moves (Pump-day glute add-ons).

## Known / by design
- Deleting a session keeps its lift history, as in v3.10, so weekly quest sets stay counted (QA C21 / D5).
- A v3.10 custom standard week is applied to past weeks only. Future weeks use the new template.
- Rest-day XP for today can drop if you then train Normal or Hard that day.

## Not tested
No real-device test.

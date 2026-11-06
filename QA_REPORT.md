# QA Report: SCULPT v3.9.0

**Date:** 2026-11-06
**Verdict:** PASS (light QA: personal app)

## Change
Each exercise card can swap to one of several alternatives of the same purpose (2 PTs grouped moves by main muscle + movement pattern). Swap sheet "Swap to" with the group name; done sets kept across any switch; each version logs under its own move.

## Probe results (headless Chrome, `scratchpad\qa38\`)
| Probe | Area | Result |
|---|---|---|
| p1-p8 | v3.8 features (updated for multi-swap) | all PASS, 0 console errors |
| p9_swaps | 3 switches keep done sets, log per move, progression per move, Hard/Light per version, kcal, v3.8 migration (twice + Import), timer and rating across switches, sheet badges, class main cards unchanged, 360 px | 123 PASS, 0 console errors |

## Not tested
No real-device test.

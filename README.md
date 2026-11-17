# gta6-twitch-ccu-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/mizuseii/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/mizuseii/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecast GTA VI's launch-day Twitch viewers hour by hour, from the first hour's print — a Python 3.13 stdlib-only program that reads a make.fun case and writes five quantiles (5%, 25%, 50%, 75%, 95%) for each of hours 2–24 of the release day.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

## How it works

- Anchor on the case's hour-1 print `p1`.
- From every comparable past launch, take its release-day hourly curve and form log-ratios `log(v_h / v_1)`.
- Per hour, take a weighted median of those log-ratios across launches. Weight favours launches whose own hour-1 print is close in scale to `p1` (log-distance Gaussian kernel, bandwidth 1.2) and console-only launches (GTA VI is console-only at launch). Scale matters: launches whose UTC hour 1 fell before their real unlock have tiny hour-1 prints and gigantic later ratios; without scale weighting they would swamp the median.
- Spread per hour is the weighted standard deviation of the log-ratios (× 0.9, clipped to [0.25, 1.1]); quantiles are `q_tau = p1 * exp(median_h + z_tau * s_h)`. The median curve is lightly smoothed with a 3-hour kernel, which keeps the unlock-wave shape (US East ~hour 6, US West ~hour 9, evening peak) while cutting hour-to-hour noise.
- The GTA VI category's own pre-release history is never used as a curve (its "release day" is the 2026-08-27 premiere day, which fails the completeness check).
- Deterministic, no network, no randomness; ~0.05 s per case.

## Validation (leave-one-out over the 37 usable comparables)

Each comparable launch was forecast as if it were the case (its hour-1 print in, the other 37 launches as comparables) and scored with the contest's log-scale pinball loss against its actual hours 2–24:

| | pinball loss | typical miss | 90% coverage | wins |
|---|---|---|---|---|
| This model | **0.395** | 127.0% | 0.877 | 32 / 37 |
| House persistence baseline | 0.930 | 804.9% | 0.624 | — |

Neighbouring settings score 0.395–0.403, so the tuned constants are not knife-edge. On the 4 practice cases all outputs are valid (23 rows, ordered quantiles, ~1.5 KB each) and byte-identical across repeated runs.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>

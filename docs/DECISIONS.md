# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.

- 2026-11-21 — Kept `cnt_median_3` in the feature set. The previous three calendar days’ ride counts are available before the forecast day. On validation, HGB improved from 0.2359 to 0.2557 R² and from 1281.3 to 1258.4 MAE. Ridge declined from 0.6475 to 0.6457 R² and from 832.9 to 837.1 MAE. Both baselines stayed unchanged. I kept the feature for its benefit to HGB and made this decision using validation scores only.

- 2026-11-21 — For Exercise 4, I trained HGB to predict the change from yesterday and added yesterday’s count back to get a forecast. This is defensible for a one-day-ahead forecast because yesterday’s total is already known. On validation, HGB improved from 0.2557 to 0.6163 R² and from 1258.4 to 814.7 MAE. Ridge had higher R² (0.6457), but transformed HGB had lower MAE than ridge (837.1). I used validation data for this comparison and retained the original target for the main pipeline.

- 2026-11-21 — Added the three-day median availability rule and the /seed-fault workflow to CLAUDE.md so future agent sessions preserve the forecast timing and record failures before repairing them.

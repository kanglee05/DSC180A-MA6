## Results

I trained on 2011, used January through June 2012 for validation, and reserved July through December 2012 for the final comparison. The forecast target was total rides per day. MAE reports the average absolute error in rides per day.

The original pipeline used 14 features. I added `cnt_median_3`, the median ridership over the previous three calendar days. All three counts are available by the morning of the forecast date. Both feature sets retained 714 rows, so the comparison uses the same dates. I generated the before and after results with `make all` from commits `0771eff` and `ea01d18`, respectively.

| Feature set | Method | Validation R² | Validation MAE | Test R² | Test MAE |
|---|---|---:|---:|---:|---:|
| Before | baseline_mean | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Before | baseline_persistence | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Before | ridge | 0.6475 | 832.9 | 0.5833 | 947.1 |
| Before | hgb | 0.2359 | 1281.3 | -0.4780 | 1954.7 |
| After | baseline_mean | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| After | baseline_persistence | 0.4507 | 900.6 | 0.5099 | 840.1 |
| After | ridge | 0.6457 | 837.1 | 0.5777 | 943.5 |
| After | hgb | 0.2557 | 1258.4 | -0.4364 | 1925.6 |

## Headline finding

The dataset contains an exact shortcut: casual rides plus registered rides equals total rides on every day. A model using those two inputs can achieve R² = 1 by reconstructing the answer. A forecaster would only learn those counts after the rides occurred.

With those columns excluded and the data split by date, the choice of baseline changed how I interpreted the results. On the final test period, predicting the 2011 average scored −2.1441 R². Predicting yesterday’s count reached 0.5099, a much stronger benchmark. With the median feature included, ridge reached 0.5777. Gradient boosting scored −0.4364: it beat the training-mean baseline but fell well short of persistence. The rise in ridership makes a model trained to predict historical demand levels a poor fit for many later days.

I kept the median feature based on validation. It raised gradient boosting’s validation R² from 0.2359 to 0.2557 and reduced its MAE from 1,281.3 to 1,258.4 rides. Ridge’s validation MAE rose slightly, from 832.9 to 837.1. The benefit was specific to the tree model.

## Exercise 4: target transformation

I changed the tree’s training target to the difference between today’s count and yesterday’s count. To recover a forecast in rides, I added the predicted difference back to yesterday’s observed count. For example, if yesterday had 5,000 rides and the tree predicts an increase of 400, the forecast is 5,400. This lets the tree predict a demand level above the range it saw during training without having to learn the entire growth trend.

On validation, predicting the change raised HGB’s R² from 0.2557 to 0.6163 and reduced its MAE from 1,258.4 to 814.7 rides. Ridge still had the higher R² at 0.6457, while the transformed tree had the lower MAE by 22.4 rides. I consider the transformation defensible for a one-day-ahead forecast because yesterday’s total is already observed. For a forecast several days ahead, those intervening counts would be unknown and this evaluation would no longer describe the deployment setting. I evaluated the transformation on validation only and kept the original target for the final results table.

## Exercise 5: ridge vs persistence

I would deploy persistence if the planning goal is to minimize the average absolute error in daily ridership. Before the median feature, ridge beat persistence on test R² by 0.0734, but its MAE was 947.1 rides versus 840.1. The decision stayed the same after adding the feature: ridge scored 0.5777 R² against 0.5099 for persistence, yet persistence missed by 103.4 fewer rides per day on average (840.1 versus 943.5). Ridge’s higher R² means it did better under squared error, which gives larger misses more weight. For the stated planning goal, persistence wins on the chosen metric and requires only one observed count.

## Attribution

I used Qwen 3.5 9B through local Ollama, switched to the NRP-hosted `glm-5` after repeated API retries, and launched a fresh Claude Code session with NRP’s `qwen3` for the branch review. I also used ChatGPT Codex for test code, Terminal scripts, and report drafting.

Agent contributions:

- Codex prepared the initial Git checkpoint and the failing named-date test for `cnt_median_3`. I invoked `/new-feature` in Claude Code and verified the resulting feature on my Mac. The Terminal fallback found the implementation already present.
- Codex supplied scripts for comparing models, checking for unclassified raw columns, testing the weather encoder, and capturing fault evidence. Claude Code performed the forbidden-column edit and repair and the Bash copy demonstration. I recorded manual hook replays separately from observed Claude tool activity.
- I ran the validation and final comparisons on my Mac using the starter pipeline. The final table uses those saved outputs. Codex helped turn those results into this write-up.

My decisions:

- I kept the three-day median after comparing both models on validation and documenting the tradeoff in `docs/DECISIONS.md`.
- I evaluated daily change as an alternative HGB target because its reference value, yesterday’s demand, is available at forecast time. I kept the original target for the main comparison.
- For a daily planning decision where absolute error is the criterion, I favor persistence based on its held-out MAE.

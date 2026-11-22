# Forecasting AAPL’s next trading-day closing price

AMS 586 Time Series · Stony Brook University · `Stock_AAPL` branch

We forecast one price per target session, AAPL’s regular-session closing price. We issue the first forecast after the previous trading day closes and update it once the target day’s official opening price becomes available. Both forecasts are scored against the same closing price. The first uses the previous close as its benchmark. The update uses the target day’s open.

## Notebook walkthrough

The steps below follow one target trading day at a time. For example, Friday’s close is the benchmark for Monday’s close. Once Monday’s official open becomes available, the opening model updates the forecast for that same Monday close.

| Step | What the code does | Output to inspect |
|---|---|---|
| 1 · Daily baseline | Sort AAPL prices by trading date and pair each close with the next observed session. Use the current close as the forecast and score by target date so year-end forecasts fall in the correct year. | `AAPL_baseline_daily_2025.csv` |
| 2 · Diagnose large errors | Split each close-to-close move into the overnight gap and the open-to-close move. Rank the ten largest absolute baseline errors and measure their share of total squared error. | `AAPL_worst_10_days_2025.csv` |
| 3 · Test daily predictors | Align AAPL and QQQ by date. Build trailing returns, AAPL’s open-to-close return and its closing position within the daily range. Add volatility, volume or QQQ returns to the price model and compare forecasts on the same dates. | `AAPL_step3_model_scores.csv` |
| 4A–4C · Build closing-window data | Check one Alpaca session, then one month including an early close. Use the trading calendar to select the last 30 minutes of each session, cache the monthly downloads and check coverage for both symbols. | `AAPL_QQQ_closing_features_history.csv` |
| 4D–4E · Test closing-window predictors | Add AAPL’s final-30-minute return relative to QQQ, then separately test AAPL’s final-30-minute volume relative to its recent history. Compare each with the same daily price model. | `AAPL_step4d_scores.csv`, `AAPL_step4e_scores.csv` |
| 5A–5B · Compare model types | Hold the four AAPL price features, target dates and 756-session training window fixed. Fit ordinary least squares, ridge and shallow XGBoost. Standardize the ridge inputs within each training window. Examine XGBoost errors on the same ten dates identified in Step 2. | `AAPL_step5b_model_scores.csv`, `AAPL_step5b_tail_scores_2025.csv` |
| 6A–6B · Update at the open | Fit open-to-close return against the overnight gap using the previous 756 completed sessions. Multiply the observed open by one plus the predicted return. Compare with using the open directly, first in 2023–2025 and then through November 6, 2026. | `AAPL_step6a_open_scores.csv`, `AAPL_step6b_open_scores_2026.csv` |
| 7 · Record a forward forecast | Keep the dated prior-close predictions. Once the target session opens, record the official open, the updated forecast and the actual issue time. Score both forecast times after the close. | `AAPL_forecast_2026-11-23.csv` is the initial prior-close record. |

## Results

All dollar amounts are per share. MAE measures the average absolute error. RMSE gives more weight to large misses. The last column counts forecasts within $5 of the actual close.

| Evaluation period | Forecast | RMSE | MAE | Within $5 |
|---|---|---:|---:|---:|
| 2023–2025 | Previous close | $3.22905 | $2.19935 | 686/752 |
| 2023–2025 | Price regression | $3.23204 | $2.19395 | 688/752 |
| 2023–2025 | Ridge | $3.22903 | $2.18869 | 687/752 |
| 2023–2025 | Shallow XGBoost | $3.23750 | $2.19519 | 683/752 |
| 2023–2025 | Opening price | $2.72391 | $1.84521 | 707/752 |
| 2023–2025 | Opening-gap regression | $2.70025 | $1.82958 | 710/752 |
| Jan 2–Nov 6, 2026 | Previous close | $4.92249 | $3.56264 | 162/214 |
| Jan 2–Nov 6, 2026 | Opening price | $4.11899 | $3.11035 | 172/214 |
| Jan 2–Nov 6, 2026 | Opening-gap regression | $4.09626 | $3.05859 | 175/214 |

In the 2025 baseline, ten sessions accounted for 45.69% of total squared error. On seven of those sessions the absolute intraday move exceeded the overnight move.

For a forecast issued the night before, we retained the previous close as the primary benchmark. Ridge reduced three-year MAE by about one cent. XGBoost had the lowest 2025 RMSE among the four main prior-close models, but a higher RMSE than the benchmark in 2023 and 2024. The QQQ and closing-window features produced no consistent annual RMSE gain.

For the opening update, the gap regression reduced RMSE relative to the opening price in each of 2023, 2024 and 2025 and in the 2026 review period. In 2026, observing the open reduced MAE by about $0.45. The regression reduced it by a further $0.052. Its error exceeded $5 on 39 of 214 days. The 2026 period had already been inspected during development and is reported as a historical check.

## Reproducing the analysis in RStudio

- Open the project from the branch root. Open the R Markdown analysis notebook and run its historical chunks in order. The file `Starting Code` contains the team’s original exploration.
- The AAPL input is `MacroTrends_Data_Download_AAPL.csv`. The header follows 14 metadata lines. Use the split-adjusted OHLC columns consistently. Daily price and volume data are from [Macrotrends](https://www.macrotrends.net/stocks/charts/AAPL/apple/stock-price-history).
- The QQQ daily input is `QQQ_daily.csv`, with columns `date` and `qqq_close`. The QQQ download uses Yahoo Finance through `quantmod`.
- The closing-window experiment uses Alpaca SIP minute bars for AAPL and QQQ and the trading calendar for each session, including early closes. It covers 1,549 sessions from November 2019 through December 2025. To rebuild the daily input, run the download chunks with `APCA_API_KEY_ID` and `APCA_API_SECRET_KEY` set locally. A saved `AAPL_QQQ_closing_features_history.csv` can be used to skip the minute-bar download. [Alpaca historical stock bars](https://docs.alpaca.markets/reference/stockbars)
- The daily analysis uses `dplyr`, `zoo` and `xgboost`. The data retrieval chunks also use `quantmod` and `httr2`. The XGBoost scores in the table are the RStudio results.
- The final score tables are `AAPL_step5b_model_scores.csv` (prior-close models), `AAPL_step6a_open_scores.csv` (opening models, 2023–2025) and `AAPL_step6b_open_scores_2026.csv` (2026 historical check). The historical 2026 scores end on November 6. The November 23 forward forecast uses a later data cutoff, November 20.

## Forward evaluation and next experiments

1. **Score forecasts as they are issued.** The first prior-close forecast was recorded for November 23, 2026, with a primary forecast of $342.56 and a price-regression forecast of $344.46. Record the opening-price benchmark and the opening-gap forecast when the official open becomes available. After the session, add the realized close and score all four forecasts. Preserve the original issue times and values.
2. **Quantify uncertainty in the gains.** Use paired daily errors and a time-block bootstrap to estimate an interval for the MAE difference between the opening-gap model and the opening-price benchmark. Track both dollar and percentage errors to compare periods with different share-price levels.
3. **Test earnings information at the original forecast time.** Start with an indicator for a scheduled earnings release between forecast issuance and the target close. Later, test an earnings surprise only for forecasts issued after the announcement. Keep the model specification fixed on subsequent forward dates and report performance on earnings and ordinary days separately.

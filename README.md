Kaggle Competition Link: https://www.kaggle.com/competitions/store-sales-time-series-forecasting/data

`storesales.rmd` implies time series analysis processes, including importing datasets, EDA, fitting models, and prediction generation.

`submission.csv` will store the predictions. For evaluation purpose, submit the file to the competition.

`holdout.Rmd`: compares three model structures on a train/test split.
Model: regression on log(1 + sales), then auto.arima on each series' residuals.
Structures: regression coefficients estimated per series(store x product family), per product family, or globally.

selection.Rmd is an improved version of holdout.rmd which adds another testing/training window, tests starting from 2013 vs 2015, and does variable selection on the covariates.

Because of the time taken to perform all the permutations of model selection on both storesales_holdout.rmd and selection.rmd, the model results are stored in the _cache folders. If you change the model code, delete the affected cache files, or the old results will be reused. Also, you need to unzip selection_cache to before it is usable.

Current Scores: 
- 0.46833. (from storesales.rmd)
- 0.44214 (from storesales.rmd on data starting from 2015)
- 0.41059 (global model from selection.rmd)
- 0.42693 ("per-family" model from selection.rmd, takes the best scoring structure (global vs family vs series) on the internal testing data for each family as the submission)
- 0.42357 (family model from selection.rmd)
- 0.44094 (series model from selection.rmd)
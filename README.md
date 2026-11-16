Kaggle Competition Link: https://www.kaggle.com/competitions/store-sales-time-series-forecasting/data

`storesales.rmd` implies time series analysis processes, including importing datasets, EDA, fitting models, and prediction generation.

`submission.csv` will store the predictions. For evaluation purpose, submit the file to the competition.

`holdout.Rmd`: compares three model structures on a train/test split.
Model: regression on log(1 + sales), then auto.arima on each series' residuals.
Structures: regression coefficients estimated per series(store x product family), per product family, or globally.


Current Score: 0.46833. (from storesales.rmd)

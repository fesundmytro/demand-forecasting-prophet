# Demand Forecasting with Prophet

Weekly sales forecasting for the Global Superstore dataset based on historical orders from 2011 to 2014.

The project includes two models:

* weekly sales value forecasting using `sales`;
* evaluation of sold-unit quantity forecasting using `quantity`.

## Project Objective

Build a Prophet model that forecasts weekly sales four weeks ahead, evaluate its performance on historical data, and compare the results with a simple seasonal baseline forecast.

An additional objective is to evaluate how accurately the model can forecast the number of units sold.

## Data

The project uses the Global Superstore dataset:

* 51,290 orders;
* period: 2011–2014;
* global sales data;
* first date: January 1, 2011;
* last date: December 31, 2014;
* missing values in `sales`, `quantity`, and `order_date` after cleaning: 0;
* rows removed during cleaning: 0.

Main columns used:

| Column       | Description          |
| ------------ | -------------------- |
| `order_date` | order date           |
| `sales`      | sales amount         |
| `quantity`   | number of units sold |

## Tools

* Python
* pandas
* NumPy
* Prophet
* scikit-learn
* matplotlib
* openpyxl
* Google Colab

## Data Source

Download the public Global Superstore dataset from:

(https://www.kaggle.com/datasets/laibaanwer/superstore-sales-dataset)

## Data Preparation

The following data preparation steps were performed:

1. Removed BOM characters and extra symbols from column names.
2. Converted `sales` to numeric format.
3. Converted `quantity` to numeric format.
4. Converted `order_date` to datetime format.
5. Checked for missing values.
6. Sorted the dataset by date.
7. Excluded incomplete weeks at the beginning and end of the dataset.
8. Aggregated individual orders into weekly totals.

After aggregation, the resulting time series contains:

* number of weeks: **208**;
* start date: **January 9, 2011**;
* end date: **December 28, 2014**;
* frequency: weekly;
* missing weeks: none.

The data was converted to the standard Prophet format:

| Column | Meaning                        |
| ------ | ------------------------------ |
| `ds`   | week-ending date               |
| `y`    | weekly sales or quantity total |

## Train/Test Split

The time series was split chronologically:

* train: first **204 weeks**;
* test: last **4 weeks**;
* test period: December 7–28, 2014.

Random splitting was not used because time-series data must preserve chronological order.

## Model

The Prophet model was configured with the following parameters:

```python
Prophet(
    yearly_seasonality=True,
    weekly_seasonality=False,
    daily_seasonality=False,
    seasonality_mode='additive',
    interval_width=0.80
)
```

Weekly and daily seasonality were disabled because the source data had already been aggregated to weekly frequency.

## Sales Forecast Results

The model was evaluated on the last four weeks of historical data.

### Prophet Metrics

| Metric                            |        Result |
| --------------------------------- | ------------: |
| MAPE                              |     **8.13%** |
| RMSE                              | **11,896.96** |
| MAE                               |  **8,521.27** |
| Simplified accuracy `100% − MAPE` |    **91.87%** |

A MAPE of 8.13% means that the weekly forecast deviated from actual sales by approximately 8.13% on average during the test period.

The `100% − MAPE` value is used only as a simplified business interpretation and is not a separate standard machine-learning metric.

## Baseline Comparison

A seasonal naive forecast was used as the baseline:

> Sales for the current week are assumed to be equal to sales from the corresponding week of the previous year.

A 52-week lag was used for the weekly time series.

| Metric |       Prophet | Seasonal Baseline |
| ------ | ------------: | ----------------: |
| MAPE   |     **8.13%** |        **22.15%** |
| RMSE   | **11,896.96** |     **26,213.43** |

Prophet reduced RMSE by approximately **54.6%** compared with the seasonal baseline.

Therefore, on the selected test period, Prophet performed significantly better than the simple seasonal forecasting rule.

## Cross-Validation

Time-series cross-validation was used for additional model evaluation:

* initial training period: 730 days;
* interval between forecast cutoffs: 90 days;
* forecast horizon: 28 days;
* number of forecast windows: 8.

Cross-validation results:

| Metric  |        Result |
| ------- | ------------: |
| CV MAPE |    **15.16%** |
| CV RMSE | **15,887.16** |
| CV MAE  | **12,582.32** |

The cross-validation metrics are worse than the results from the final four-week holdout period.

This indicates that forecast performance varies across historical periods, so the holdout result should not be interpreted as guaranteed accuracy for every future period.

## Final Sales Forecast

After evaluation, the model was retrained on all 208 weeks of historical data.

A forecast was generated for the first four weeks of 2015:

| Week       | Sales Forecast | Lower Bound | Upper Bound |
| ---------- | -------------: | ----------: | ----------: |
| 2015-01-04 |     102,140.97 |   89,164.36 |  114,189.00 |
| 2015-01-11 |      85,990.72 |   72,166.92 |   98,552.43 |
| 2015-01-18 |      76,061.72 |   63,037.13 |   88,976.66 |
| 2015-01-25 |      75,603.51 |   61,976.55 |   88,379.01 |

Total sales forecast for the four-week period:

* expected sales: **339,796.92**;
* lower forecast bound: **286,344.96**;
* upper forecast bound: **390,097.09**.

This is a historical forecasting demonstration because the source dataset ends in December 2014.

## Quantity Forecast

A separate model was also built using the `quantity` column.

The model was evaluated on the same final four-week period.

### Quantity Forecast Metrics

| Metric              |           Result |
| ------------------- | ---------------: |
| MAPE                |        **8.00%** |
| RMSE                | **137.43 units** |
| MAE                 | **127.87 units** |
| Simplified accuracy |       **92.00%** |

Results for the test weeks:

| Week       | Actual | Forecast | Error, % |
| ---------- | -----: | -------: | -------: |
| 2014-12-07 |  1,758 | 1,627.31 |    7.43% |
| 2014-12-14 |  1,715 | 1,645.68 |    4.04% |
| 2014-12-21 |  1,450 | 1,656.55 |   14.24% |
| 2014-12-28 |  1,675 | 1,570.09 |    6.26% |

In the current version of the project, the `quantity` model is evaluated only on the historical test period.

A separate future quantity forecast for January 2015 was not generated.

## Business Conclusion

The Prophet model forecasts total weekly sales four weeks ahead with a MAPE of 8.13% on the final holdout period.

The forecast can be used as an additional input for:

* procurement planning;
* logistics resource planning;
* budget allocation;
* workload estimation;
* identifying periods of increased demand.

The `quantity` model is more relevant to inventory planning because it forecasts the number of units sold rather than only the monetary value of sales.

However, to directly reduce stockout risk, forecasts should be built separately by product, category, or subcategory and combined with information about:

* current inventory levels;
* supplier lead times;
* safety stock;
* minimum order quantities;
* supplier availability.

## Project Limitations

* the test set contains only four weeks;
* the dataset ends in 2014;
* the main forecast is based on total company-wide sales;
* cross-validation shows a higher error than the holdout test;
* the model does not include promotions, holidays, or marketing campaigns;
* the model does not include inventory levels or supplier lead times;
* the `quantity` forecast is not separated by individual products;
* the Prophet interval represents model uncertainty but does not guarantee that actual values will fall within the predicted range.

## Exported Results

The notebook exports results in two formats:

* `.csv` — for GitHub and further processing in Python;
* `.xlsx` — for convenient viewing in Microsoft Excel.

The Excel files include:

* separate columns;
* bold headers;
* filters;
* frozen header rows;
* adjusted column widths;
* formatted dates and numeric values.

## Repository Structure

```text
demand-forecasting-prophet/
├── demand_forecasting_prophet.ipynb
├── README.md
├── requirements.txt
├── data/
│   └── weekly_sales.csv
├── results/
│   ├── forecast_vs_actual.csv
│   ├── next_4_weeks_forecast.csv
│   └── quantity_forecast_vs_actual.csv
└── excel_exports/
    ├── weekly_sales.xlsx
    ├── forecast_vs_actual.xlsx
    ├── next_4_weeks_forecast.xlsx
    └── quantity_forecast_vs_actual.xlsx
```

## Project Files

* `demand_forecasting_prophet.ipynb` — complete analysis and models;
* `weekly_sales.csv` — weekly sales time series;
* `forecast_vs_actual.csv` — comparison of forecasted and actual sales;
* `next_4_weeks_forecast.csv` — four-week sales forecast;
* `quantity_forecast_vs_actual.csv` — comparison of forecasted and actual unit quantities;
* `.xlsx` files — formatted versions of the result tables for Microsoft Excel.

## Running the Project

Install the required libraries:

```bash
pip install pandas numpy prophet scikit-learn matplotlib openpyxl
```

Open the notebook:

```text
demand_forecasting_prophet.ipynb
```

Upload the source file:

```text
SuperStoreOrders.csv
```

Then run all notebook cells from top to bottom.

## Possible Improvements

Future project improvements may include:

1. Forecasting sales separately by category.
2. Forecasting quantities by subcategory and product.
3. Adding holidays and seasonal events to Prophet.
4. Tuning model parameters.
5. Comparing Prophet with ARIMA, SARIMA, XGBoost, and LightGBM.
6. Using rolling backtesting with multiple forecast horizons.
7. Adding inventory-level data.
8. Calculating reorder points and safety stock.


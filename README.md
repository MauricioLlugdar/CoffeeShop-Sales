# CoffeeShop Sales Forecasting

This project analyzes and models a dirty transactional sales dataset from a coffee shop. The main goal is to clean the data, build weekly aggregated datasets, and evaluate whether historical sales patterns can be used to predict the quantity of units sold in the following week.

The project is divided into two main stages:

1. **Exploratory analysis, cleaning, and feature engineering**
2. **Regression modeling and evaluation**

---

## Project objective

The main prediction task is:

> Given the information available up to the current week, predict the number of units sold during the following week.

Two modeling perspectives are considered:

* **Weekly total demand:** predict the total number of units sold next week.
* **Weekly item-level demand:** prepare the data to predict how many units of each item may be sold next week.

This second perspective is useful from a stock planning point of view, since estimating future demand by item can help anticipate inventory needs.

---

## Dataset description

The original dataset contains transactional records from a coffee shop. Each row represents a transaction and includes the following columns:

| Column             | Description                                             |
| ------------------ | ------------------------------------------------------- |
| `Transaction ID`   | Unique identifier for each transaction.                 |
| `Item`             | Product sold, such as Coffee, Tea, Sandwich, Cake, etc. |
| `Quantity`         | Number of units purchased.                              |
| `Price Per Unit`   | Unit price of the item.                                 |
| `Total Spent`      | Total transaction amount.                               |
| `Payment Method`   | Payment method used by the customer.                    |
| `Location`         | Whether the purchase was in-store or takeaway.          |
| `Transaction Date` | Date of the transaction.                                |

The dataset is intentionally dirty and includes missing values, invalid labels such as `ERROR` and `UNKNOWN`, and numeric columns stored as strings.

---

## Repository structure

```text
CoffeeShop-Sales/
│
├── 01_CoffeeShopSales_Analysis_Exploration_Cleaning.ipynb
├── 02_CoffeeShopSales_Modeling.ipynb
│
├── processed_data/
│   ├── weekly_total_clean.csv
│   └── weekly_items_clean.csv
│
└── README.md
```

---

## Notebook 1: Analysis, exploration and cleaning

The first notebook performs the initial data analysis and cleaning process.

Main steps:

* Load the raw dirty dataset.
* Analyze missing values, invalid values and column types.
* Replace invalid labels such as `ERROR` and `UNKNOWN` with missing values.
* Convert numeric columns to proper numeric types.
* Convert transaction dates to datetime format.
* Use the relationship between `Quantity`, `Price Per Unit` and `Total Spent` to recover missing values when possible.
* Handle missing categorical values.
* Create weekly aggregated datasets.
* Build features for weekly prediction.
* Save the processed datasets into `processed_data/`.

Generated files:

* `weekly_total_clean.csv`: weekly total sales dataset.
* `weekly_items_clean.csv`: weekly item-level sales dataset.

---

## Notebook 2: Modeling

The second notebook uses the processed datasets to train and evaluate regression models.

The main modeling task is to predict:

```text
target_next_week_units
```

This target represents the number of units sold in the following week.

Since the target is numeric, the problem is treated as a **regression problem**.

---

## Feature engineering

The weekly total dataset includes historical and seasonal features such as:

* `units_sold`: units sold in the current week.
* `avg_last_4_weeks`: moving average of the last 4 available weeks.
* `avg_last_8_weeks`: moving average of the last 8 available weeks.
* `historical_avg_units`: expanding historical average up to the current week.
* `next_week_year_sin`: cyclical representation of the next week within the year.
* `next_week_year_cos`: cyclical representation of the next week within the year.

The sine/cosine seasonal encoding is used because calendar variables such as month or week number are cyclical. For example, the end of December and the beginning of January are close in time, but their raw numeric values may appear far apart if encoded directly as `52` and `1`.

---

## Modeling approach

The modeling process follows a baseline-first approach.

The models evaluated include:

* `DummyRegressor`
* `LinearRegression`
* `Ridge`

The `DummyRegressor` is used as a basic baseline. It does not learn from the input features and allows us to check whether the other models are actually adding predictive value.

`LinearRegression` is used as a simple interpretable regression model.

`Ridge` is used as a regularized linear model. This is useful because the weekly dataset has a limited number of observations and some features are correlated, such as current sales and moving averages. Ridge helps reduce overly large coefficients and can make the model more stable.

For Ridge and other linear models with regularization, `StandardScaler` is used inside a pipeline to ensure that all features are on comparable scales before applying the model.

---

## Evaluation metrics

The models are evaluated using several regression metrics:

| Metric | Interpretation                                                                     |
| ------ | ---------------------------------------------------------------------------------- |
| MAE    | Average absolute error in units sold. Easy to interpret.                           |
| RMSE   | Similar to MAE, but penalizes larger errors more strongly.                         |
| R²     | Measures how much variance is explained compared to a constant baseline.           |
| WAPE   | Total absolute error relative to total actual sales. Useful for sales forecasting. |

The main metric considered is **MAE**, because it directly answers:

> On average, how many units is the model off by?

However, RMSE, R² and WAPE are also analyzed to better understand model behavior.

---

## Preliminary interpretation

The first results show that linear models improve over the basic `DummyRegressor`, but the improvement is moderate rather than dramatic.

This suggests that the historical and seasonal features contain some predictive signal, but they are not enough to fully explain the week-to-week variability in sales.

A plot of real vs predicted weekly sales shows that Ridge tends to predict a smoother version of the sales series. It captures the general sales level, but it has difficulty anticipating sharp peaks and drops.

This is expected because the available dataset only contains basic transactional information. It does not include external variables that could explain sudden changes in demand, such as:

* promotions
* holidays
* weather
* local events
* stock availability
* days closed
* marketing campaigns

Therefore, the model should be interpreted as a first forecasting baseline rather than a final production-level forecasting system.

---

## How to run the project

To reproduce the full workflow:

1. Run `01_CoffeeShopSales_Analysis_Exploration_Cleaning.ipynb`
2. Run `02_CoffeeShopSales_Modeling.ipynb`

The first notebook generates the processed datasets inside the `processed_data/` folder. The second notebook reads those processed files and trains the regression models.

Since the processed CSV files are already included in the repository, the modeling notebook can also be run directly.

---

## Main limitations

This project has some important limitations:

* The dataset contains only basic transactional variables.
* The weekly dataset has a small number of observations.
* The model does not include external demand drivers such as holidays, promotions, weather or events.
* The current models capture the average sales level better than sudden weekly fluctuations.
* The results should be interpreted as an initial forecasting experiment, not as a definitive demand forecasting solution.

---

## Possible future improvements

Future work could include:

* Adding external calendar features such as holidays.
* Including weather data.
* Adding promotion or discount information if available.
* Testing item-level demand models.
* Comparing against stronger temporal baselines.
* Trying tree-based models such as Random Forest or Gradient Boosting if more data is available.
* Performing a more detailed error analysis for weeks with large prediction errors.

---

## Technologies used

* Python
* pandas
* numpy
* scikit-learn
* matplotlib
* seaborn
* Jupyter Notebook / Google Colab

---

## Author

Mauricio Llugdar

# Predicting Regional Retail Trade in Kazakhstan

A linear regression project that predicts monthly retail trade volume across Kazakhstan's 20 regions, built on a dataset I assembled by hand from official government statistics rather than a pre-packaged Kaggle CSV.

## Why this dataset

The first version of this project used the well-known Walmart sales dataset (US retail data, 2010–2012). It's a fine dataset for learning the mechanics of `scikit-learn`, but it has nothing to do with where I live or what I'm actually trying to get good at — and using the exact same dataset as thousands of other beginner ML portfolios doesn't say much about how I think.

So for this version I went directly to the source: the Bureau of National Statistics of the Republic of Kazakhstan (stat.gov.kz), which publishes a monthly bulletin called *"Socio-Economic Development of the Republic of Kazakhstan."* For each of the country's 20 regions, it reports retail trade volume, inflation, household income, unemployment, capital investment, and more. I pulled four of these bulletins — February 2024, June 2024, June 2025, and December 2025 — and manually compiled them into a single panel dataset: 20 regions × 4 time points = 80 observations.

It's not a huge dataset, and I'm upfront about what that costs the model in the limitations section below. But every number in it is real, recent, traceable to a specific government PDF, and tied to a country and economy I actually understand.

## Problem statement

**Can monthly retail trade volume in a Kazakhstani region be predicted from population, income, inflation, investment, and unemployment?**

And just as importantly: *which* of those factors matters most, and why? A model that predicts well but can't explain itself isn't very useful for understanding regional economics — which is the whole point of using linear regression here instead of a black-box model.

## Dataset

| | |
|---|---|
| **Source** | [Bureau of National Statistics of Kazakhstan](https://stat.gov.kz/) — monthly "Socio-Economic Development" bulletins |
| **Rows** | 80 (20 regions × 4 periods) |
| **Time span** | February 2024 – December 2025 |
| **Target** | `Retail_Trade_mln_KZT` — monthly retail trade volume, million KZT |

Full column descriptions are in [`data/DATA_DICTIONARY.md`](data/DATA_DICTIONARY.md), including direct links to each of the four source PDFs.

## Exploratory analysis

### Feature distributions
![Distributions](images/1_distributions.png)

Retail trade, population, and investment are all heavily right-skewed — Almaty city is roughly 160x larger than the smallest region, Ulytau. That's normal for city-size data (it roughly follows a power law), but it's a problem for linear regression on raw values, since the model would mostly end up reacting to two or three huge cities.

### Log transform
![Log transform](images/1b_log_transform.png)

Log-transforming `Retail_Trade`, `Population`, `Investment`, and `Income` turns the comparison from "how many more million KZT" into "what percent more" — a question that's meaningful whether you're comparing Ulytau to itself a year later, or comparing Ulytau to Almaty. This is also what makes the resulting coefficients interpretable as elasticities later on.

### Correlation
![Correlation](images/2_correlation.png)

Investment (r≈0.67) and population (r≈0.65) correlate most strongly with retail trade — but through different mechanisms. Population is a near-mechanical driver: more people, more purchases. Investment is more of an indirect signal — it marks economically active, growing regions, and growth tends to pull consumption along with it.

### Population and income vs. retail trade
![Scatter](images/3_scatter_population_income.png)

Worth noting: Atyrau region, with its oil-driven income, sits well above the income trend line but not proportionally above it on retail trade — a lot of that income likely leaves the region as savings or investment rather than local spending.

### Regional comparison
![Regions Dec 2025](images/4_regions_dec2025.png)

Almaty outsells Astana by roughly 2.3x, despite the two cities' populations being much closer (2.3M vs. 1.6M). That gap reflects Almaty's long-standing role as the country's commercial center, not a population difference.

### Inflation over time
![CPI trend](images/5_cpi_trend.png)

Inflation stayed persistently high (109–113% year-over-year) across the sample period, and nominal retail trade nearly doubled. That's not a doubling of real consumption — retail trade is reported in current prices, so part of that growth is just things costing more, not people buying more. This distinction matters for how the CPI coefficient should be read later on.

## Model

```
Model:    Linear Regression (scikit-learn)
Features: log(Population), log(Income per capita), CPI YoY %,
          log(Investment), Unemployment rate %
Target:   log(Retail Trade)
Split:    75% train / 25% test, random_state=42
```

### Results

| Metric | Train | Test |
|---|---|---|
| MAE (log units) | 0.357 | 0.330 |
| RMSE (log units) | 0.471 | 0.439 |
| **R²** | 0.741 | **0.720** |

5-fold cross-validation: R² = 0.530 ± 0.322 (per-fold scores: 0.76, 0.25, 0.72, 0.88, 0.05)

In real KZT terms, the test-set mean absolute error is about 21.1 billion KZT, with a MAPE of roughly 41%.

> **Why R² as the primary metric:** MAE and RMSE aren't directly comparable to each other (different units of interpretation). R² gives a single, bounded measure of explained variance that's comparable across train and test.

![Actual vs predicted](images/6_actual_vs_predicted.png)

A test R² of 0.72 is a reasonable result for 80 fairly noisy regional observations. The wide swing in the 5-fold CV scores is expected given the small effective sample size — see *Limitations* below.

## Coefficients — what they actually mean

![Coefficients](images/7_coefficients.png)

Because the model is fit in logs for four of five features, most coefficients are **elasticities**: the percentage change in retail trade associated with a 1% change in the feature.

| Feature | Coefficient | Interpretation |
|---|---|---|
| `log(Income per capita)` | **+1.53** | A 1% increase in income is associated with retail trade growing by *more than* 1% — an elasticity above one, meaning spending on non-essentials (dining out, electronics, clothing) grows disproportionately faster than spending on necessities as income rises. |
| `log(Population)` | **+1.22** | Also above 1, which means larger cities sell disproportionately more per capita, not just proportionally more. Likely because Almaty and Astana function as retail hubs for surrounding, less populated regions, not just for their own residents. |
| `Unemployment rate` | **−0.44** | Exponentiating gives exp(−0.437) ≈ 0.65 — a one-percentage-point rise in unemployment is associated with roughly 35% lower retail trade, holding everything else fixed. The only negative effect in the model, and the most intuitive one: no job, no paycheck, less spending. |
| `log(Investment)` | **+0.11** | The weakest meaningful effect. Capital investment (factories, roads, infrastructure) tends to feed into consumption with a lag of months or years through new jobs, not instantly within the same month. |
| `CPI year-over-year` | **+0.02** | Small and positive — each extra point of inflation adds about 2.3% to nominal retail trade. Most of the inflation effect on nominal sales is already absorbed into the time trend between the four periods, so the standalone CPI effect, once other variables are controlled for, comes out modest. |

## Limitations

I'm not presenting this as a production-ready forecasting model. Specifically:

**Small effective sample.** 80 rows sounds like a reasonable size, but it's really 20 regions sampled 4 times, not 80 independent observations — successive snapshots of the same region are strongly correlated. The real statistical power is closer to n=20, and the wide swing in cross-validation scores (0.05 to 0.88 across folds) reflects exactly that.

**Nominal, not real, prices.** Retail trade is measured in current prices, so the model can't separate inflation-driven growth from genuine increases in consumption volume. Deflating the target by CPI before modeling would be the natural fix.

**No interaction effects.** A linear model assumes the effect of, say, income is the same in Almaty as it is in a largely agricultural region like Turkestan — which probably isn't true given how different the spending structures are.

**No explicit seasonality control.** The four snapshots span different months (Feb, Jun, Dec), and December in particular is a higher-spending month, which could be quietly distorting some coefficients (especially CPI and investment).

**Unemployment has very little variation to work with.** Regional unemployment in Kazakhstan sits in a narrow 4.0–5.1% band. The true effect could be larger than what the model shows — there's just not enough spread in this sample to detect it confidently.

## What I'd do next

Add more, ideally monthly, time points instead of four snapshots; switch the target to CPI-deflated (real) retail trade; add regional dummy variables to capture structural differences between cities and rural oblasts; and compare this linear model against a Random Forest to check how much of the relationship here is genuinely linear.

## Stack

```
Python 3.12
pandas, numpy        — data wrangling
matplotlib, seaborn   — visualization
scikit-learn          — LinearRegression, train_test_split, cross_val_score, metrics
```

## Repository structure

```
.
├── data/
│   ├── kazakhstan_regional_retail_trade.csv
│   └── DATA_DICTIONARY.md
├── notebooks/
│   └── kazakhstan_retail_regression.ipynb
├── images/
│   ├── 1_distributions.png
│   ├── 1b_log_transform.png
│   ├── 2_correlation.png
│   ├── 3_scatter_population_income.png
│   ├── 4_regions_dec2025.png
│   ├── 5_cpi_trend.png
│   ├── 6_actual_vs_predicted.png
│   └── 7_coefficients.png
├── requirements.txt
└── README.md
```

## Running it

```bash
git clone https://github.com/<your-username>/kazakhstan-retail-trade-regression.git
cd kazakhstan-retail-trade-regression
pip install -r requirements.txt
jupyter notebook notebooks/kazakhstan_retail_regression.ipynb
```

## Sources

- Data: [Bureau of National Statistics of the Republic of Kazakhstan](https://stat.gov.kz/) — see [`data/DATA_DICTIONARY.md`](data/DATA_DICTIONARY.md) for direct links to all four source bulletins.

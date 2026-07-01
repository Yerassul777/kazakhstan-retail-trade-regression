# Predicting Regional Retail Trade in Kazakhstan

multiple linear regression analysis of monthly retail trade volume across Kazakhstan's 20 administrative regions. dataset compiled from four official monthly bulletins published by the Bureau of National Statistics of the Republic of Kazakhstan (stat.gov.kz), covering February 2024 to December 2025.

---

## Problem Statement

retail trade volume is one of the more direct proxies for regional consumer activity. unlike GDP, it is measured monthly and reflects actual household spending rather than an estimated aggregate. that makes it a useful leading signal for regional economic health.

the question this project addresses is concrete: can the monthly retail trade volume of a Kazakhstani region be estimated from a small set of macroeconomic and demographic indicators - population, income per capita, inflation, capital investment, and unemployment? and if yes, which of those indicators carries the most weight, and why does the data produce that ranking rather than some other?

the choice of linear regression is deliberate. a gradient-boosted tree might predict slightly more accurately, but it would not answer the second question. linear regression gives a direct, auditable answer: each coefficient states how much retail trade changes per unit change in one variable, with everything else held fixed.

---

## Dataset

| | |
|---|---|
| **Source** | Bureau of National Statistics of Kazakhstan ([stat.gov.kz](https://stat.gov.kz/)) |
| **Publication** | monthly bulletin "Socio-Economic Development of the Republic of Kazakhstan" |
| **Observations** | 80 (20 regions x 4 time periods) |
| **Time span** | February 2024 - December 2025 |
| **Target** | `Retail_Trade_mln_KZT` - monthly retail trade volume, million KZT |

four bulletins were used: February 2024, June 2024, June 2025, and December 2025. each bulletin covers all 20 regions, including the cities of Astana, Almaty, and Shymkent. income and unemployment are published quarterly; each monthly observation was matched to the nearest available quarter.

full column definitions and direct PDF links are in [`data/DATA_DICTIONARY.md`](data/DATA_DICTIONARY.md).

---

## Exploratory Data Analysis

### Feature Distributions

![Distributions](images/1_distributions.png)

three variables - retail trade, population, and investment - show pronounced right-skew. retail trade is the clearest case: the bulk of observations fall below 100 bln KZT, while Almaty city exceeds 1.2 trillion KZT in December 2025, roughly 160 times the volume in Ulytau.

the skew is not a data quality issue - it reflects the actual structure of the Kazakhstani economy, where two or three urban centers concentrate a disproportionate share of economic activity. a model fit on raw values gets pulled toward explaining the variance of those two or three outlier cities, at the cost of the relationship that holds across the other 18 regions. the log transformation fixes this.

CPI and unemployment are the exceptions: narrow, roughly symmetric, no transformation needed. inflation ranged from 107.8% to 113.8% across the sample; unemployment from 4.0% to 5.1%.

### Log Transformation

![Log transform](images/1b_log_transform.png)

the left panel shows retail trade in raw units; the right shows log(retail trade). after the transform, the distance between a 50 bln KZT region and a 100 bln KZT region is the same as the distance between 500 bln and 1 trillion - both are a factor of two. this is the right scale for comparing regions that differ by orders of magnitude, and what makes the coefficients readable as elasticities.

### Correlation Structure

![Correlation](images/2_correlation.png)

investment (r = 0.67) and population (r = 0.65) lead the table, but the near-equal values are somewhat coincidental - they drive retail trade through different channels.

population's correlation is high because retail trade is, at its most basic level, the sum of transactions by individual residents. more people means more buyers. the relationship holds across regions with very different income profiles, which is why it stays strong even in a 20-region sample with considerable economic diversity.

investment's correlation is slightly higher than population's despite being a more indirect measure. capital investment is a proxy for general economic dynamism: regions attracting large capital flows are expanding across multiple dimensions simultaneously - employment, infrastructure, commercial development. a high investment figure co-occurs with higher income, better retail infrastructure, and growing employment, all of which drive retail trade. the correlation coefficient captures all of those indirect paths, not just investment's direct effect.

income per capita (r = 0.39) comes in lower than might be expected. the Atyrau effect explains most of it: Atyrau has among the highest income per capita in Kazakhstan due to its oil sector, but retail trade is moderate relative to that income. a substantial share of oil-sector earnings does not cycle through local stores, which compresses the income-retail correlation across the full sample.

unemployment (r = -0.16) shows the weakest effect. the regional range is only 4.0-5.1%, which gives the correlation little variation to work with. the true relationship between unemployment and spending is likely stronger - the data just doesn't carry enough spread in that variable to detect it clearly.

### Population and Income vs. Retail Trade

![Scatter](images/3_scatter_population_income.png)

the left panel shows the population relationship is close to linear in log-log space. the slope above 1 (the model estimates 1.22) means doubling a region's population is associated with more than doubling its retail trade. the excess over 1 comes from agglomeration: large cities draw retail traffic from surrounding areas, sustain more diverse retail formats, and maintain commercial density that smaller regions cannot.

the right panel shows income per capita. atyrau is the clearest anomaly - it sits far right (high income) but near the trend vertically (retail trade in line with population, not income). the gap reflects the structure of an oil economy: a relatively small workforce earns high wages, but those wages do not circulate broadly into local retail. broad-based employment, not aggregate income, tends to drive consumer activity.

### Regional Comparison

![Regions Dec 2025](images/4_regions_dec2025.png)

in December 2025, Almaty's retail trade exceeded Astana's by a factor of approximately 2.3, against a population ratio closer to 1.4. the gap is structural rather than demographic. almaty's wholesale and retail infrastructure - distribution centers, established chains, international brands - was built up before Astana became the capital in 1997 and has not relocated. residents of surrounding oblasts travel to Almaty for purchases unavailable locally, inflating its retail figures beyond what its resident population would produce.

### Inflation and Average Sales Over Time

![CPI trend](images/5_cpi_trend.png)

between February 2024 and December 2025, average nominal retail trade nearly doubled. inflation ran at 110-113% year-over-year across the same period, meaning a share of that nominal growth is higher prices rather than more goods sold. the distinction matters for the CPI coefficient: some of what the model sees as retail growth is the same basket of goods at higher prices.

the dual-axis chart shows one clear pattern: nominal retail trade trends up consistently across all four periods, while CPI moves in a narrower band. retail is growing faster than inflation alone would produce, which suggests some real consumption growth - but the model works in nominal terms and cannot separate the two.

---

## Model

```
model:    LinearRegression (scikit-learn)
features: log(population), log(income per capita), CPI YoY %,
          log(investment), unemployment rate %
target:   log(retail trade)
split:    75% train / 25% test, random_state=42
```

### Results

| metric | train | test |
|---|---|---|
| MAE (log units) | 0.357 | 0.330 |
| RMSE (log units) | 0.471 | 0.439 |
| **R²** | 0.741 | **0.720** |

5-fold cross-validation R²: mean = 0.530, std = 0.322 (per-fold: 0.76, 0.25, 0.72, 0.88, 0.05)

in real KZT: test-set MAE approximately 21 bln KZT, MAPE approximately 41%

R² is used as the primary metric for comparing train and test performance. MAE and RMSE are both reported but cannot be compared directly to each other because they aggregate errors differently. R² has a fixed [0, 1] scale that is consistent across both sets.

![Actual vs predicted](images/6_actual_vs_predicted.png)

test R² of 0.72 means the model captures roughly 72% of the variance in log(retail trade) using five inputs. the gap between train and test is small (0.741 vs 0.720), indicating the model generalizes rather than overfitting the training sample.

the residual distribution is approximately symmetric around zero - no systematic directional bias, the model is not consistently off in one direction. the CV scores span 0.05 to 0.88, which reflects the small effective sample size rather than model instability. with only 20 independent regions, a single fold can end up with an unrepresentative composition that pulls R² far from the average.

---

## Coefficient Interpretation

![Coefficients](images/7_coefficients.png)

for the four log-transformed features, each coefficient is an elasticity: the percentage change in retail trade per 1% change in the feature, with all others held constant. CPI and unemployment remain in percentage-point units, so their coefficients are the absolute change in log(retail trade) per one percentage-point shift.

| feature | coefficient | what it means |
|---|---|---|
| log(income per capita) | +1.53 | a 1% rise in regional income is associated with a 1.53% rise in retail trade. the elasticity exceeds 1 because discretionary spending - restaurants, electronics, clothing - grows faster than income, while spending on basic necessities is relatively flat. as income rises, a larger share of it goes to categories with high unit value. |
| log(population) | +1.22 | an elasticity above 1 means larger regions are not just proportionally larger retail markets, they are disproportionately larger ones. this is consistent with agglomeration effects: major cities draw retail traffic from outside their administrative boundaries, run more diverse retail formats, and sustain commercial infrastructure that smaller regions cannot support. |
| unemployment rate | -0.44 | exponentiating this gives exp(-0.437) = 0.65, meaning a one percentage-point increase in unemployment is associated with retail trade being approximately 35% lower. this is the only negative coefficient in the model. the mechanism is direct: unemployment reduces household income, which reduces the budget available for consumption. |
| log(investment) | +0.11 | the weakest of the significant effects. fixed capital investment affects consumption through job creation and infrastructure, not immediately. a new logistics center or factory takes months to years before it generates employment income that flows into local retail spending. within a single monthly snapshot, the contemporaneous effect is small. |
| CPI YoY % | +0.02 | each additional percentage point of inflation corresponds to approximately 2.3% higher nominal retail trade, once population, income, investment, and unemployment are held constant. the effect is small because most of the inflation signal is already absorbed into the time dimension of the data - prices rose across all regions simultaneously, so it shows up in period-to-period trends rather than cross-regional variation. |

---

## Limitations

**effective sample size.** the dataset has 80 rows, but 20 of those regions each appear 4 times. observations from the same region across periods are correlated - Almaty in February 2024 and Almaty in December 2025 share the same fundamental economic structure. the effective number of independent units is closer to 20 than 80, which is what the cross-validation variance is reflecting.

**nominal prices throughout.** retail trade is reported in current prices. the model does not separate real volume growth from inflation-driven nominal growth. deflating retail trade by CPI before modeling would isolate the real consumption signal, but that transformation was not applied here.

**no interaction terms.** the model assumes that the effect of income on retail trade is the same in every region. the actual relationship likely differs between a city like Almaty, where a high-income household might spend primarily on services and premium goods, and a rural region where that same income increase shifts spending toward durable goods and food. a linear model averages these into one coefficient.

**seasonality not controlled.** the four observation periods fall in February, June, and December. december retail trade is typically higher than June or February due to holiday spending. this seasonal signal is not isolated in the current setup, which could be affecting the CPI and investment coefficients in particular.

**unemployment range is too narrow to draw firm conclusions.** regional unemployment across the four periods sits between 4.0% and 5.1%. in a dataset where a key variable only moves within a 1.1 percentage-point range, even a strong underlying relationship will produce a weak estimated coefficient. the -0.44 figure should be read as directionally correct but not as a precise magnitude estimate.

---

## Sources

bureau of national statistics of the republic of Kazakhstan - [stat.gov.kz](https://stat.gov.kz/). direct links to all four source bulletins are in [`data/DATA_DICTIONARY.md`](data/DATA_DICTIONARY.md).

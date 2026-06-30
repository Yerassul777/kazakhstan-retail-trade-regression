# Data Dictionary

File: `data/kazakhstan_regional_retail_trade.csv`
Rows: 80 (20 regions × 4 time periods)

| Column | Type | Unit | Description |
|---|---|---|---|
| `Region` | string | — | One of Kazakhstan's 20 administrative regions, including the three cities of republican significance (Astana, Almaty city, Shymkent) |
| `Period` | string | YYYY-MM | Snapshot month: 2024-02, 2024-06, 2025-06, or 2025-12 |
| `Retail_Trade_mln_KZT` | float | million KZT | **Target variable.** Retail trade turnover for that single month (not cumulative year-to-date) |
| `CPI_YoY_percent` | float | % | Consumer Price Index, year-over-year (e.g. 112.3 means prices were 12.3% higher than the same month one year prior) |
| `Population_thousand` | float | thousand people | Regional population at the start of the period |
| `Investment_mln_KZT` | float | million KZT | Fixed capital investment for that single month |
| `Income_per_capita_KZT` | float | KZT/month | Average nominal monthly income per capita, for the calendar quarter nearest to the period (income is reported quarterly, not monthly, by the source) |
| `Unemployment_rate_percent` | float | % | Unemployment rate (ILO methodology), for the calendar quarter nearest to the period |

## Provenance

All figures were manually transcribed from four official monthly bulletins published by the Bureau of National Statistics of the Agency for Strategic Planning and Reforms of the Republic of Kazakhstan (stat.gov.kz):

- February 2024 — `Ж-01-01-М 02 2024 (рус).pdf`
- June 2024 — `Ж-01-М 06 2024 (рус).pdf`
- June 2025 — `Ж-01-М 06 2025 (рус).pdf`
- December 2025 — `Ж-01-М 12 2025 (рус).pdf`

Direct links are listed in the notebook and in the main README. Income and unemployment figures, which the source reports quarterly, were matched to the nearest available quarter for each monthly snapshot — a standard approach when combining indicators published at different frequencies.

No values were estimated, interpolated, or synthetically generated; every number traces back to a specific table in one of the four source PDFs.

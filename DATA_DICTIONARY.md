# data dictionary

file: `data/kazakhstan_regional_retail_trade.csv`
rows: 80 (20 regions x 4 time periods)

| column | type | unit | description |
|---|---|---|---|
| `Region` | string | - | one of Kazakhstan's 20 administrative regions, including the three cities of republican significance (Astana, Almaty city, Shymkent) |
| `Period` | string | YYYY-MM | snapshot month: 2024-02, 2024-06, 2025-06, or 2025-12 |
| `Retail_Trade_mln_KZT` | float | million KZT | **target variable** - retail trade turnover for that single month, not cumulative year-to-date |
| `CPI_YoY_percent` | float | % | consumer price index, year-over-year (e.g. 112.3 means prices were 12.3% above the same month one year prior) |
| `Population_thousand` | float | thousand people | regional population at the start of the period |
| `Investment_mln_KZT` | float | million KZT | fixed capital investment for that single month |
| `Income_per_capita_KZT` | float | KZT/month | average nominal monthly income per capita; income is reported quarterly by the source, so each monthly observation uses the nearest available quarter |
| `Unemployment_rate_percent` | float | % | unemployment rate (ILO methodology); same quarterly matching applies as for income |

---

## provenance

figures were compiled from four official monthly bulletins published by the Bureau of National Statistics of the Republic of Kazakhstan (stat.gov.kz). each bulletin covers all 20 regions for the corresponding month.

| period | direct link |
|---|---|
| february 2024 | [Ж-01-01-М 02 2024](https://stat.gov.kz/upload/iblock/002/iymgccgoyj2adx0lgbbu1ovcapdity11/%D0%96-01-01-%D0%9C%2002%202024%20(%D1%80%D1%83%D1%81).pdf) |
| june 2024 | [Ж-01-М 06 2024](https://stat.gov.kz/upload/iblock/b16/u1k0y8ikgvnv2y0vgxzc0gg6cx4ru9f5/%D0%96-01-%D0%9C%2006%202024%20(%D1%80%D1%83%D1%81).pdf) |
| june 2025 | [Ж-01-М 06 2025](https://stat.gov.kz/upload/iblock/c98/gfrc35fthdnpbtp34vzz4i4xu1dc4cz4/%D0%96-01-%D0%9C%2006%202025%20(%D1%80%D1%83%D1%81).pdf) |
| december 2025 | [Ж-01-М 12 2025](https://stat.gov.kz/upload/iblock/082/mwq0xdd7o5ukymteea8y9n1kyvhg12xn/%D0%96-01-%D0%9C%2012%202025%20(%D1%80%D1%83%D1%81).pdf) |

income and unemployment are published quarterly rather than monthly. for each monthly snapshot the nearest available quarter was used - a standard approach when combining indicators reported at different frequencies.

no values were estimated, interpolated, or synthetically generated. every number in the dataset traces back to a specific table in one of the four source PDFs listed above.

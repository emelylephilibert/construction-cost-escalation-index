# Construction Cost Escalation Index

A lag-based regression model using public Statistics Canada data, with out-of-sample validation.

**Author:** Emely Lephilibert
**Language:** Python
**Data:** Statistics Canada, tables 18-10-0001-01, 18-10-0266-01 and 18-10-0004-01, snapshot of 2026-11-17

## Result

Multi-year construction projects are priced at one point in time, while fuel and material costs keep moving during execution. This project builds an escalation index from three public Statistics Canada tables, tests it on months it was not fitted on, and adapts it to the cost structure of a project.

| Finding | Evidence |
|---------|----------|
| Fuel and construction material prices lead consumer inflation | Strongest correlation with the Montréal CPI at a 1-month lag for fuel (0.634) and an 8-month lag for materials (0.765) |
| A two-input index tracks the CPI with a simple, transparent formula | Construction Market Index (CMI): 15.1% fuel, 84.9% materials, R² 0.6247 |
| The model holds on data it has not seen | April to October 2026: RMSE 0.218 against 0.443 for a naive benchmark, 7 of 7 months inside the 95% interval |
| The index triggers adjustments only in extraordinary periods | 17 months above the review band, all in the post-COVID surge (Nov 2021 to Mar 2023); no trigger in the 2020 shock |
| 2026 shows rising pressure, close to the contract limit | No trigger in the rolling review; a contract with a February 2026 base date reaches +10.16% by November, just over the +10% first-year limit |
| A project layer adapts the index to a project's cost structure | M16 combines fixed, fuel, materials and CMI components with project-specific weights; it records the same number of historical triggers as the CMI, starting one month earlier |

## The indexes

**Construction Market Index (CMI)**, the market reference, the same for every project:

`CMI(t) = 0.151 × CFI(t-1) + 0.849 × IPPI(t-8)`

where the CFI is the average of normalized diesel and gasoline prices in Montréal and the IPPI is the price index for fabricated metal products and construction materials (P63). All components are normalized to January 2018 = 100.

**Project index (M16)**, which adds a project cost breakdown to the CMI. The weights below are illustrative; on a real project they come from the project's estimate:

`M16(t) = 0.15 × 100 + 0.10 × CFI(t) + 0.15 × IPPI(t) + 0.60 × CMI(t)`

## Method

1. **Data audit and preparation:** extraction on exact labels, integrity checks, normalization to January 2018 = 100 and a dated snapshot.
2. **Trends and regimes:** year-over-year and monthly volatility across four economic regimes.
3. **Association and lead time:** correlations by regime, 36-month rolling correlations and a lag analysis from 0 to 12 months, with a common-sample robustness check.
4. **Index modeling:** thirteen OLS specifications, multicollinearity (VIF), constrained regression on the fuel share and regime regressions with blending scenarios. Three candidates are kept: M12 (15.1% fuel), M15 (20%) and M14 (30%).
5. **In-sample evaluation:** tracking against the CPI, a sensitivity check on the March 2026 fuel spike (Cook's distance) and a trigger simulation on a hypothetical \$100M contract with a review band of +12% / -8%.
6. **Out-of-sample backtest:** the frozen model is tested on April to October 2026, and a forecast of the November 2026 CPI is recorded before its publication.
7. **From market index to project index:** M12 becomes the CMI, and the project index M16 is built and tested on the same rules.

## Limitations

The CMI is calibrated to the CPI, not to construction costs, so it should not be read as a cost structure. The M16 weights are illustrative and the index is only as accurate as the cost breakdown behind it. Some periods are short (10 months for the COVID shock, 7 for the out-of-sample test), consecutive year-over-year changes overlap, and recent Statistics Canada values can be revised. The contract values are hypothetical and the dollar simulation is simplified. The full list is in section 7.3 of the notebook.

## Repository

    construction-cost-escalation-index/
    ├── 01_construction_cost_escalation_index.ipynb
    ├── README.md
    ├── requirements.txt
    └── data/
        ├── raw/
        │   └── statcan_series_2026-11-17.csv.gz
        └── processed/
            └── master_dataset_2026-11-17.parquet

- `data/raw/statcan_series_2026-11-17.csv.gz`: the original Statistics Canada rows of the five series used (3,457 rows).
- `data/processed/master_dataset_2026-11-17.parquet`: the monthly master dataset, January 2010 to October 2026 (202 months × 12 columns).

## How to reproduce

1. Install the dependencies: `pip install -r requirements.txt` (the notebook was run with Python 3.13).
2. Download the three full tables as CSV zip files from Statistics Canada and place them in `data/raw/`:
   - Monthly average retail prices for selected products: table 18-10-0001-01 (`18100001-eng.zip`)
   - Industrial product price index, by product: table 18-10-0266-01 (`18100266-eng.zip`)
   - Consumer Price Index, monthly, not seasonally adjusted: table 18-10-0004-01 (`18100004-eng.zip`)
3. Open the notebook from the repository folder and run all cells. Outside Colab, the notebook reads the data from `data/`; in Colab, it reads it from Google Drive.

New downloads may include revised values that change some results. The two saved files keep the exact values used in this analysis, so any difference can be traced to a revision. The model estimation period stays fixed at March 2026; new months extend the out-of-sample test.

## Data source

Statistics Canada. Contains information licensed under the Statistics Canada Open Licence.

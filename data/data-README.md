# Data

The raw data files are not tracked in this repository. All twelve are freely available from two public sources and should be placed in `data/raw/`.

## Kenneth French Data Library

Available at https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html. Download the CSV version of each, unzip, and rename to match:

| File | Library entry |
|------|---------------|
| `25_Portfolios_5x5_Daily.csv` | 25 Portfolios Formed on Size and Book-to-Market (5 x 5), daily |
| `25_Portfolios_5x5.csv` | 25 Portfolios Formed on Size and Book-to-Market (5 x 5), monthly |
| `100_Portfolios_ME_OP_10x10.csv` | 100 Portfolios Formed on Size and Operating Profitability (10 x 10) |
| `F-F_Research_Data_5_Factors_2x3_daily.csv` | Fama/French 5 Factors (2x3), daily |
| `F-F_Research_Data_5_Factors_2x3.csv` | Fama/French 5 Factors (2x3), monthly |
| `F-F_Momentum_Factor.csv` | Momentum Factor (Mom) |

These files are derived portfolio return series distributed by the data library. Each carries a header line noting the CRSP database vintage used to construct it. They are not redistributed here.

## FRED

Available at https://fred.stlouisfed.org. Search the series ID and download as CSV:

| File | Series ID | Series |
|------|-----------|--------|
| `VIXCLS.csv` | VIXCLS | CBOE Volatility Index |
| `AAA.csv` | AAA | Moody's Seasoned Aaa Corporate Bond Yield |
| `BAA.csv` | BAA | Moody's Seasoned Baa Corporate Bond Yield |
| `GS10.csv` | GS10 | 10-Year Treasury Constant Maturity Rate |
| `TB3MS.csv` | TB3MS | 3-Month Treasury Bill Secondary Market Rate |
| `USREC.csv` | USREC | NBER Based Recession Indicators |

## Sample Period

The analysis runs from January 1998 through December 2025, giving 7,043 trading days. The start date is set by the availability of daily VIX. Download each series over at least that range; the notebook trims to the common sample.

## Vintage Note

The French library files used in the original analysis were built from the 202602 and 202603 CRSP vintages. Later vintages revise historical values slightly, so figures reproduced from a fresh download may differ in the final decimal places.

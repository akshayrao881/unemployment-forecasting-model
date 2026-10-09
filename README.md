# Does the Stock Market Predict Unemployment? Build, Test, Break

**Business question:** How much of U.S. unemployment growth can financial-market, monetary and real-economy indicators explain, and does the "best" model survive diagnostic testing?

A three-part regression study (ECON622, Group 2, University of Delaware) in R. The model is built in stages, then deliberately stress-tested.

## Headline results
- **Project 1:** S&P 500 level alone explains **2.8%** of the variation in the monthly unemployment rate (n = 908, 1949–2024; slope −0.000254, t = −5.10). Statistically significant, practically small.
- **Project 2:** six macro predictors lift adjusted R² on 4-quarter unemployment growth from **0.066** (S&P 500 alone) to **0.743** (n = 245 quarters). The added variables are jointly significant (F = 129.3, p < 1e-64).
- A 3-variable model (S&P 500, Fed funds, industrial production) fits almost as well (adj. R² 0.738) and has the lowest BIC. Time-split and expanding-window cross-validation favour the multi-variable models over the simple one.
- **Project 3:** diagnostics on the full model find serial correlation (Breusch–Godfrey χ² = 154.9, p = 1.5e-27), non-normal residuals (Shapiro–Wilk p = 6.9e-10), a misspecified functional form (RESET p = 0.0007) and unstable parameters (CUSUM; Chow/Wald break at 2008Q1, p = 1.2e-4; predictive failure at COVID, p = 2e-26).

## Three stages
| Stage | Question | Folder |
|---|---|---|
| Project 1 | Does the stock market alone relate to unemployment? Simple OLS, one regressor | `project1-simple-regression/` |
| Project 2 | Do monetary, trade and real-economy variables help? Six predictors, t/F tests, AIC/BIC, cross-validation | `project2-multiple-regression/` |
| Project 3 | Is the Project 2 model valid? Serial correlation, normality, multicollinearity, functional form, stability | `project3-diagnostics/` |

Each folder holds the R Markdown source, the knitted PDF report, and the CSVs it reads (Project 1 uses its own `SP500.csv`/`UNRATE.csv`, which differ from Project 2–3's `SP500.csv`/`unemp.csv`, so each folder is self-contained).

## Data
| Variable | Measure | Source |
|---|---|---|
| Unemployment rate | `UNRATE` | FRED |
| S&P 500 | month-end index level | Yahoo Finance (^GSPC) |
| Fed funds rate | `FEDFUNDS` | FRED |
| FDI | `ROWFDIQ027S` | FRED |
| CPI | `CPIAUCSL` | FRED |
| Industrial production | `INDPRO` | FRED |
| Gold | historical gold price | Macrotrends |

**Projects 2–3:** the dependent variable is `UNEMP.FG = 100 × [log(unemp at t+4) − log(unemp at t)]`, quarterly. Monthly series are averaged to quarters. Predictors are 4-quarter growth rates (`SP500.G`, `FEDFUNDS.G`, `FDI.G`, `CPI.G`, `INDPRO.G`, `GOLD.G`).

## Project 1 (simple regression)
Regresses the monthly unemployment *rate* on the S&P 500 *level* (not returns).
| Estimate | Value |
|---|---|
| Intercept | 5.92 |
| Slope | −0.000254 (SE 0.0000498, t = −5.10, p ≈ 4e-7) |
| R² / adj. R² | 0.028 / 0.027 |

## Project 2 (model comparison)
| Model | Predictors | Adj. R² | AIC | BIC |
|---|---|---|---|---|
| A: Simple | S&P 500 | 0.066 | 2192.6 | 2203.1 |
| B: Full | all 6 | 0.743 | **1881.0** | 1909.0 |
| C: Parsimonious | S&P 500, Fed funds, INDPRO | 0.738 | 1882.9 | **1900.4** |

Model C drops FDI, CPI and gold (all p > 0.10) at almost no cost to fit. Out-of-sample, 4 time splits (10–25% held out) and 3 expanding windows (75–85% initial training) give lower RMSE/MAE for the multi-variable models than for Model A.

## Project 3 (diagnostics on Model B)
| Test | Result | Reading |
|---|---|---|
| Breusch–Godfrey (order 11) | χ² = 154.92, df 11, p = 1.47e-27 | Serial correlation, so Newey–West errors (lag 11) are used |
| Shapiro–Wilk | W = 0.924, p = 6.9e-10 | Residuals not normal |
| VIF | max 1.55 (INDPRO.G) | No multicollinearity concern |
| Ramsey RESET (NW-robust Wald) | χ² = 14.63, df 2, p = 0.0007 | Linear form rejected (powers 2–4: χ² = 30.8, p = 9e-7) |
| CUSUM | crosses 5% bound | Parameters not stable over the full sample |
| Chow / Wald break at 2008Q1 (NW) | χ² = 29.36, df 7, p = 1.25e-4 | Relationship differs pre/post-2008 (187 vs 57 quarters) |
| Chow predictive failure (pre/post-COVID) | F = 14.46 (16, 222), p = 1.95e-26 | Pre-COVID model fails after 2020 |

Under Newey–West errors, Fed funds growth (p = 2.7e-4) and industrial-production growth (p = 6e-12) stay highly significant. The intercept goes from significant (OLS p = 0.0015) to insignificant (NW p = 0.21). The S&P 500 coefficient is not significant under either OLS (p = 0.053) or NW (p = 0.29).

## Takeaways
1. Macro controls multiply explanatory power roughly 11x over a stock-market-only model, and the gain survives cross-validation.
2. Industrial production and the Fed funds rate carry the model. The S&P 500 adds little once they are included.
3. Good R² and CV do not mean a valid model: the full model fails serial-correlation, normality, functional-form and stability tests, so its coefficients are descriptive, not causal.

## Limitations
- **Timing:** the Project 2–3 code pairs 4-quarter-ahead unemployment growth with predictor growth over the *same* window, so the fit is contemporaneous, not a one-year-ahead forecast. Shifting the predictors back four quarters in a replication cuts adjusted R² from about 0.74 to about 0.25. Read 0.743 as "explains same-year movements", not "forecasts a year out".
- **FDI dates:** `FDI.csv` starts in 1951Q4 but the code declares a 1952Q1 start; the tail-based alignment hides the offset, but it should be fixed.
- COVID-era quarters (2020Q1–2021Q2) are extreme outliers and likely drive part of the non-normality.
- Growth rates of near-zero bases (e.g. Fed funds at the zero lower bound) produce extreme percentages.
- Next steps: a lagged specification, a COVID dummy or sample split, and a nonlinear form.

## Run it
R ≥ 4.2 with a LaTeX install (xelatex).
```r
install.packages(c("tidyverse","readr","knitr","ggplot2","zoo","lubridate",
  "kableExtra","lmtest","sandwich","broom","dplyr","rlang","patchwork",
  "forecast","car","strucchange"))
rmarkdown::render("project1-simple-regression/simple_regression.Rmd")
rmarkdown::render("project2-multiple-regression/multiple_regression.Rmd")
rmarkdown::render("project3-diagnostics/regression_diagnostics.Rmd")
```
Render each file from inside its own folder (or set the working directory there) so the CSVs are found.

## Authors
Nan Gong, Akshay Rao, Shantanu Surendra Singh, Reshma Thomas. ECON622, Group 2.

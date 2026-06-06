# Stochastic Interest Rate Modelling: CIR Implementation & Extension

A complete quantitative finance research notebook implementing, calibrating,
and extending the Cox-Ingersoll-Ross (CIR) short-rate model on real historical
yield curve data.

## Objective

Reconstruct the full yield curve (6M–30Y) using **only the 3M yield** as input,
achieving an out-of-sample R² > 0.85.

**Result: Overall R² = 0.9006 ✓**

## Project Structure

```
Stochastic-Interest-Rate-Modeling-And-Prediction/
├── data/
│   ├── train_data.csv        # 1,976 days of yield data (9 maturities)
│   ├── test_data.csv         # 495 days (3M–2Y maturities)
│   └── test_data_3M.csv      # Test period 3M yields only
├── notebook/
│   └── CIR_Yield_Curve_Modelling.ipynb
├── problem/
│   └── Problem_statement.pdf
├── .gitignore
└── README.md
```

## Dataset

Daily zero-coupon bond yields across 9 maturities:
`3M, 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, 30Y`

| Split | Date Range | Observations | Maturities Available |
|-------|-----------|-------------|---------------------|
| Train | 2016-05-19 → 2024-04-26 | 1,976 days | All 9 (3M–30Y) |
| Test  | 2024-04-29 → 2026-04-29 | 495 days   | 5 (3M–2Y) |

**Core constraint**: only the 3M yield may be used at test time for prediction.

## Models Implemented

| Model | Description | Overall R² |
|-------|-------------|-----------|
| 1F-CIR OLS | Time-series OLS → CIR bond pricing formula | ~0.874 |
| 1F-CIR MLE | Exact MLE via non-central chi-squared density | ~0.881 |
| 1F-CIR Cross-Section | Full yield surface calibration | 0.896 |
| 2F-CIR Kalman Filter | Two-factor model with linear Kalman filter | ~0.820 |
| **Enhanced CIR** | Cross-sectional regression + momentum feature | **0.9006** |

## Mathematical Framework

**CIR Stochastic Differential Equation:**

$$dr_t = \kappa(\theta - r_t)\,dt + \sigma\sqrt{r_t}\,dW_t$$

**Parameters:**
- κ — speed of mean reversion
- θ — long-run mean interest rate
- σ — volatility coefficient
- W_t — standard Brownian motion

**Feller Condition:** 2κθ ≥ σ² ensures rates remain strictly positive

**Bond pricing (closed form):**

$$P(t,T) = A(\tau)\,e^{-B(\tau)\,r_t}$$

**Yield formula (linear in short rate):**

$$y(\tau) = \frac{B(\tau)}{\tau}\cdot r_t + \frac{-\ln A(\tau)}{\tau}$$

This linearity is the core property enabling single-input yield curve reconstruction.

**Two-Factor Extension:**

$$r_t = x_t + y_t$$

$$dx_t = \kappa_1(\theta_1 - x_t)\,dt + \sigma_1\sqrt{x_t}\,dW_1 \quad \text{(slow / level factor)}$$

$$dy_t = \kappa_2(\theta_2 - y_t)\,dt + \sigma_2\sqrt{y_t}\,dW_2 \quad \text{(fast / slope factor)}$$

## Key Findings

- **CIR linearity** — yields are linear in r_t — is the fundamental property
  enabling single-input yield curve reconstruction from just the 3M rate
- **Cross-sectional calibration** outperforms time-series calibration (OLS/MLE)
  for yield reconstruction because it directly minimises yield error
- **6M, 9M, 1Y** are predicted with R² > 0.93; highly correlated with 3M rate
- **2Y** is the hardest maturity (R² ≈ 0.41) because it incorporates
  forward-looking monetary policy expectations not spanned by today's 3M rate
- **Feller condition** (2κθ ≥ σ²) satisfied across all calibrations
- **Two-Factor CIR** captures level/slope dynamics via Kalman filter but is
  limited at test time since only one observable (3M) is available

## Production Code — Class Structure

| Class | Responsibility |
|-------|---------------|
| `DataProcessor` | Loading, forward-fill, winsorisation, rate floor, feature engineering |
| `CIRModel` | OLS / MLE / cross-sectional calibration; B(τ), lnA(τ), yield surface |
| `TwoFactorCIRModel` | Two-factor CIR with linear Kalman filter for state estimation |
| `CrossSectionalCIRModel` | Enhanced regression model; fits α(τ), β(τ) directly from yield surface |
| `Evaluator` | RMSE (bps), MAE (bps), R² per maturity and pooled overall |
| `Visualizer` | Yield fan plots, actual vs predicted scatter, RMSE heatmaps |

## Notebook Phases

| Phase | Content |
|-------|---------|
| 1 | Exploratory Data Analysis (distributions, correlations, PCA, stationarity) |
| 2 | Data Cleaning & Engineering (DataProcessor class) |
| 3 | Base CIR: mathematics, OLS calibration, MLE calibration |
| 4 | Base CIR: yield curve reconstruction and evaluation |
| 5 | Two-Factor CIR with Kalman Filter |
| 6 | Enhanced CIR: cross-sectional regression (main model) |
| 7 | Benchmark comparison across all models |
| 8 | Critical analysis: residuals, Feller condition, failure modes |
| 9 | Production OOP code and Visualizer |
| 10 | Conclusions and future extensions |

## How to Run

### Google Colab (recommended for submission)

1. Upload `CIR_Yield_Curve_Modelling.ipynb` to Colab
2. Upload the three CSV files from `data/` via the Files panel (📤 icon)
3. Leave paths as default (`'train_data.csv'` etc.)
4. `Runtime → Run all`

### Local / VS Code

```bash
git clone https://github.com/0xDarkXnight/Stochastic-Interest-Rate-Modeling-And-Prediction/
cd Stochastic-Interest-Rate-Modeling-And-Prediction
pip install numpy pandas matplotlib seaborn scipy scikit-learn statsmodels
```

Open `notebook/CIR_Yield_Curve_Modelling.ipynb` in VS Code or Jupyter.

Paths are already set to `'../data/train_data.csv'` to match the folder structure.

## Dependencies

```
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
statsmodels
jupyter
```

Install all at once:

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn statsmodels jupyter
```

## Evaluation Metric

Out-of-sample R² computed as:

$$R^2_{oos} = 1 - \frac{\text{MSE}_\text{model}}{\text{MSE}_\text{baseline}}$$

where baseline predicts the training mean at every test point.

**Achieved: R² = 0.9006 > 0.85 ✓**

## References

- Cox, Ingersoll & Ross (1985) — *A Theory of the Term Structure of Interest Rates*, Journal of Finance
- Longstaff & Schwartz (1992) — *Interest Rate Volatility and the Term Structure*, Journal of Finance
- Duffie, Pan & Singleton (2000) — *Transform Analysis and Asset Pricing for Affine Jump-Diffusions*, Econometrica
- Brigo & Mercurio (2006) — *Interest Rate Models: Theory and Practice*, Springer
- Campbell & Thompson (2008) — *Predicting Excess Stock Returns Out of Sample*, Review of Financial Studies

## Author

Submitted as part of Finance Club, IIT Roorkee — Open Projects 2026
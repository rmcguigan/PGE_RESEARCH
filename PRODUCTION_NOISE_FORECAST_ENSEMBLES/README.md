# Characterizing Production Forecast Uncertainty Through Production Noise Forecast Ensembles (PNFE)

[![Python](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Reference Article

**Title:** Characterizing Production Forecast Uncertainty Through Production Noise Forecast Ensembles  
**Authors:** Ryan McGuigan, John T. Foster, Michael J. Pyrcz  
**Affiliations:** Hildebrand Department of Petroleum and Geosystems Engineering, Department of Aerospace Engineering and Engineering Mechanics, and Department of Planetary and Earth Sciences, The University of Texas at Austin; Chord Energy, Houston, TX.  
**Status:** Submitted to *Computers & Geosciences* (November 2025), DOI pending.  
**Suggested citation:** McGuigan, R., Foster, J. T., & Pyrcz, M. J. (2025). Characterizing Production Forecast Uncertainty Through Production Noise Forecast Ensembles. *Computers & Geosciences*, in review.

## Abstract

Accurate quantification of production forecast uncertainty is essential for data-driven reservoir management but remains challenging due to noise in measured production data. This study introduces the Production Noise Forecast Ensemble (PNFE) workflow, a computational framework for assessing the impact of stochastic production noise on decline-curve forecasts. The method integrates bootstrap resampling and model bagging to generate ensembles of noisy production realizations, enabling the construction of probabilistic forecast distributions and the evaluation of uncertainty calibration using the goodness score metric. PNFE models production noise as a stationary stochastic process derived from the relative first differences of historical production data, fit through maximum likelihood estimation to a parametric distribution such as the Generalized Error Distribution. Synthetic production histories are generated from the Modified Hyperbolic Decline model and perturbed with noise realizations to quantify forecast certainty as a function of production history length. Results demonstrate that forecast uncertainty decreases nonlinearly with increasing history and stabilizes after one to two years of production data. The workflow, implemented in open-source Python, provides a generalizable computational approach for noise-aware uncertainty quantification applicable to geoscientific time series beyond petroleum production.

## Highlights

- Introduces the PNFE workflow that links empirical noise diagnostics, bootstrap resampling, and uncertainty scoring to quantify forecast reliability under stochastic operational noise.
- Uses three Bakken horizontal wells with 8,621 daily observations (February 2015 – January 2024) to demonstrate heavy-tailed noise (β≈0.5) and operations-driven nugget effects.
- Provides reproducible notebooks that generate synthetic Arps ensembles (qi 400–1,300 bopd, di 60–100%/yr, b 0.5–1.5, dmin 7%/yr) with noise sampled from the fitted GED distributions.
- Reports that calibrated probabilistic forecasts require ~360–720 days of history before goodness scores stabilize, highlighting the value of explicit noise modeling in production forecasting workflows.

## Repository Contents

```
.
├── README.md                              # This documentation
├── DAILY_PRODUCTION.csv                   # Bakken well observations used for noise fitting
├── Forecast_Input_Parameters.xlsx         # Parameter ranges for synthetic Arps ensembles
├── Production_Noise_Forecast_Ensembles.ipynb   # End-to-end PNFE workflow
├── Production_Noise Forecast_Ensembles_copy.ipynb # Working scratch notebook
├── Production_noise_modeling.ipynb        # Noise diagnostics (variograms, GED fitting, ADF)
├── example_variogram.ipynb                # Stand-alone variogram tutorial
└── LICENSE                                # MIT license text
```

## Data & Inputs

### Field observations (`DAILY_PRODUCTION.csv`)

- 8,621 rows of daily oil, gas, and water rates for three Bakken horizontal wells (Wells A–C).  
- Time span: 11 February 2015 – 24 January 2024; zero-rate days removed before analysis.  
- Columns: `WELL_NAME`, `D_DATE`, `OIL`, `GAS`, `GAS_SALES`, `WATER`.  
- Operational context: Wells differ in lift capacity and surface allocation routing, producing distinct noise signatures (stable lift, shared facility adjustments, and intermittent under-lift, respectively).  
- Pre-processing inside the notebooks converts dates to `datetime64`, computes relative/percent changes, rolling statistics (30-day window), and encodes on-stream days.

### Synthetic forecast priors (`Forecast_Input_Parameters.xlsx`)

Defines the Arps-MHD ranges sampled when generating synthetic decline curves. These priors drive the Monte Carlo ensemble used to benchmark the uncertainty model.

### Software environment

- Python 3.8+ with `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`, `tqdm`, and `jupyter`.  
- Recommended hardware: ≥8 GB RAM and ≥500 MB free disk space.  
- Typical installation: 5–10 minutes via `pip install numpy pandas matplotlib seaborn scipy statsmodels tqdm jupyter`.  
- Export `environment.yml` with `conda env export > environment.yml` for archival alongside submissions.

## PNFE Workflow (per manuscript Section 2)

1. **Characterize production noise**  
   - Detrend daily oil rates via relative first differences.  
   - Inspect semi-variograms (Figures 1–3) to quantify nugget effects (20–40% of variance).  
   - Verify weak stationarity with 30-day sliding mean/variance and ADF tests (Table 1).  
   - Treat the stationary relative differences as samples of the nugget effect.  

2. **Fit parametric noise models**  
   - Use Maximum Likelihood Estimation to fit Generalized Error (Generalized Normal) Distributions.  
   - Retain β<1 heavy tails that capture allocation spikes and lift interruptions (Table 2).  

3. **Bootstrap noise onto synthetic histories**  
   - Draw Arps-MHD parameters from Table 3 ranges.  
   - Generate deterministic decline curves, then perturb daily rates with GED noise (β=0.5 baseline).  
   - Build Monte Carlo ensembles (default ≥500 realizations) and optional windowed bootstraps for history matching.

4. **Generate forecast ensembles via bagging**  
   - Regress decline parameters on each noisy realization.  
   - Expand the training window (e.g., 90, 180, 360, … 720 days) to emulate maturing wells.  
   - Aggregate forecasts to form probabilistic EUR distributions.

5. **Evaluate the uncertainty model**  
   - For each probability level p, compute the empirical coverage ξ(p) of symmetric forecast intervals.  
   - Accuracy indicator: a(p)=1 if ξ(p)≥p, else 0.  
   - Accuracy score:  \(A = \int_0^1 a(p)\,dp\).  
   - Precision score: \(P = 1 - 2\int_0^1 a(p)[\xi(p)-p]dp\).  
   - Goodness score (Deutsch, 1997): \(G = 1 - \int_0^1 [3a(p)-2][\xi(p)-p]dp\).  
   - Well-calibrated ensembles approach A≈P≈G≈1; short histories and high noise reduce these metrics.

### Empirical statistics reproduced in the notebooks

**Table 1 – ADF tests on relative first differences**

| Well | ADF statistic | p-value | Decision (5%) |
| --- | --- | --- | --- |
| A | -58.51 | 0.00000 | Reject unit root (stationary) |
| B | -7.85  | 0.00000 | Reject unit root (stationary) |
| C | -18.61 | 0.00000 | Reject unit root (stationary) |

**Table 2 – GED parameters fit to Bakken wells**

| Well | β (shape) | loc | scale | median | std. dev. |
| --- | --- | --- | --- | --- | --- |
| A | 0.54 | 0.0000 | 0.059 | 0.0012 | 0.4628 |
| B | 0.59 | -0.0010 | 0.079 | -0.0021 | 0.4283 |
| C | 0.43 | 0.0000 | 0.021 | 0.0012 | 0.5276 |

**Table 3 – Arps-MHD prior ranges**

| Parameter | Min | Max | Notes |
| --- | --- | --- | --- |
| Initial rate (qi, bopd) | 400 | 1,300 | Sampled uniformly |
| Initial decline (di, %/yr) | 60 | 100 | Effective annual |
| b-factor | 0.5 | 1.5 | Modified hyperbolic |
| Minimum decline (dmin, %/yr) | 7 | 7 | Terminal exponential |
| Abandonment rate (qab, bopd) | 1 | 1 | Floor on production |

## Running the notebooks

1. **`Production_noise_modeling.ipynb`** (≈10 min)  
   - Loads `DAILY_PRODUCTION.csv`, filters on-stream days, and reproduces Figures 1–6.  
   - Produces semi-variograms, ACF/PACF, rolling statistics, and GED fits with diagnostic plots saved under `figures/noise/`.

2. **`Production_Noise_Forecast_Ensembles.ipynb`** (≈15–20 min depending on Monte Carlo size)  
   - Implements the PNFE workflow start-to-finish.  
   - Key configuration cells: `GROUND_TRUTH_PARAMS`, `ERROR_MODEL`, `MC_SETTINGS`, and `BOOTSTRAP_WINDOW`.  
   - Outputs EUR distributions, goodness score curves, and convergence diagnostics under `figures/pnfe/`.

3. **`example_variogram.ipynb`** (≈5 min)  
   - Minimal example showing variogram calculation for generic time series; useful for adapting PNFE to other assets.

All notebooks rely on relative paths, so run them from the repository root with `jupyter notebook <file>.ipynb`.

## Key findings from the manuscript

- Relative first differences of Bakken daily oil rates meet weak-stationarity assumptions and isolate nugget variance linked to operational noise.  
- GED fits with β<1 capture heavy-tailed production noise better than Gaussian assumptions, preventing underestimation of volatility.  
- Noise contributes 20–40% of total variance; ignoring it inflates early EUR forecasts and reduces uncertainty coverage.  
- Bagged ensembles trained on <180 days of history systematically under-cover true EUR, whereas ≥360 days begin to converge toward well-calibrated goodness scores.  
- Production history length exhibits diminishing returns beyond ~720 days—the confidence interval reaches an irreducible width despite additional data.  
- Facility-induced noise signatures (stable lift vs. shared allocation vs. under-lift) can be diagnosed before forecasting, enabling targeted mitigation.

## Reproducibility checklist

- Data source and preprocessing documented (`DAILY_PRODUCTION.csv`, on-stream filters, rolling statistics).  
- Noise model parameters stored in notebook outputs and can be exported to CSV for audit trails.  
- Synthetic priors captured in `Forecast_Input_Parameters.xlsx`.  
- Random seeds (default `np.random.seed(42)`) documented in each notebook.  
- Environment exportable via `conda env export > environment.yml`.  
- Figures and tables reproduced programmatically; rerun the notebooks to regenerate submission-quality plots.

## Troubleshooting

- **Long runtimes / memory pressure:** Reduce `MC_SETTINGS["iterations"]` or increase step size of training windows.  
- **Non-invertible covariance warnings:** Ensure `DAILY_PRODUCTION.csv` retains non-zero variance after filtering; drop wells with insufficient samples.  
- **Optimization failures during decline fitting:** Provide tighter bounds for `qi`, `di`, and `b` or switch the optimizer to `L-BFGS-B` with relaxed tolerances.  
- **Missing data errors:** Confirm that both `DAILY_PRODUCTION.csv` and `Forecast_Input_Parameters.xlsx` reside in the project root.

## Contact

- Open an issue at [github.com/rmcguigan/PGE_RESEARCH/issues](https://github.com/rmcguigan/PGE_RESEARCH/issues).  
- For direct correspondence, refer to the corresponding-author information in the submitted manuscript.

## License

This project is distributed under the MIT License (see `LICENSE`).

## References

1. Arps, J. J. (1945). Analysis of Decline Curves. *Transactions of the AIME*, 160(1), 228–247.  
2. Fetkovich, M. J. (1980). Decline Curve Analysis Using Type Curves. *Journal of Petroleum Technology*, 32(6), 1065–1077.  
3. Deutsch, C. V. (1997). *Geostatistics for Reservoir Characterization*. Oxford University Press.  
4. Hale, P. T. (2016). Methods and Uncertainty in Forecasts and Estimated Ultimate Recovery. In J. Seidle (Ed.), *SPEE Monograph 4*.  
5. Ilk, D., Anderson, D. M., Stotts, G. W. J., Mattar, L., & Blasingame, T. A. (2010). Production-Data Analysis—Challenges, Pitfalls, Diagnostics. *SPE Reservoir Evaluation & Engineering*, 13(3), 538–552.  
6. Sadri, M., Shariatipour, S. M., & Ahmadinia, M. (2019). The Impact of Regular Well Testing on the Accuracy of Allocation Calculations. 37th International North Sea Flow Measurement Workshop.  
7. Alqahtani, M., Baghdadi, F., & Wenrong, M. (2023). Multiphase Flowmeter Comparison in Complex Field. *Offshore Technology Conference*, OTC-32610-MS.  
8. Davison, A. C., & Hinkley, D. V. (1997). *Bootstrap Methods and Their Application*. Cambridge University Press.  
9. Metropolis, N., & Ulam, S. (1949). The Monte Carlo Method. *Journal of the American Statistical Association*, 44(247), 335–341.  
10. Künsch, H. R. (1989). The Jackknife and the Bootstrap for General Stationary Observations. *The Annals of Statistics*, 17(3), 1217–1241.

## Version History

- **v1.0.0** (2025-11-25). Initial public release accompanying the Computers & Geosciences submission.

---

**Note to reviewers:** All figures, tables, and metrics reported in the manuscript are regenerated by the notebooks in this repository. Please rerun the notebooks after cloning the repo to validate the PNFE workflow end-to-end.

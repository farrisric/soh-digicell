# Battery State-of-Health Modelling (DigiCell)

This repository describes a private code base.
The source code, data and trained models are not public.
This README explains what the work does, how it was evaluated and what it found.
Access to the code is available on request.

## Context

The work is part of DigiCell, a Horizon Europe project (Grant Agreement 101135486).
It was developed at LEITAT, Terrassa.
The goal is to grade lithium-ion cells for second-life use from fast, non-destructive measurements.

## What it predicts

- **State of health (SOH):** remaining capacity as a fraction of the original.
- **Absolute capacity (Ah):** from a short partial discharge.
- **Remaining useful life (RUL):** cycles left until a set end-of-life threshold.
- **Uncertainty:** a calibrated 95% interval for each prediction.

## Inputs

- Electrochemical impedance spectroscopy (EIS), resampled to a shared 49-point frequency grid.
- Partial voltage curves (dQ/dV windows and pseudo-OCV shape features).

## What the code base contains

- Parsers for raw cycler and potentiostat files (`.nda`, `.mpr`) and an equivalent-circuit fitter.
- Loaders for in-house data and for five open datasets (KIT, LiBforSecUse, Zhang 2020, Samsung 30Q, Warwick).
- Feature builders for EIS and voltage curves.
- Models: Gaussian-process regression, PLS, LightGBM, TabPFN, 1D CNN, deep kernel learning, ensembles.
- Split-conformal calibration for prediction intervals, with a GUM-style uncertainty export.
- More than 30 numbered, reproducible experiments and a single leaderboard of results.

## How it was evaluated

- Leave-one-cell-out: the test cell is never seen in training.
- Metrics are pooled over all held-out predictions.
- Features derived from the target are banned; one such leak was found and removed.
- Every result on the leaderboard cites the script that produced it.

## Key results

In-house fleet, 15 cells, 7 commercial cell models, SOH:

| Model | Features | R2 | MAE (pp) |
|-------|----------|----|----------|
| GPR (ARD + linear mean) | EIS + voltage shape | 0.90 | 2.0 |
| PLS | EIS + voltage shape | 0.87 | 2.5 |
| TabPFN | EIS only | 0.82 | 2.7 |

Open fleets, SOH from EIS only:

| Dataset | Cells | Best R2 | Best MAE (pp) |
|---------|-------|---------|---------------|
| KIT (LG HG2) | 228 | 0.98 | 1.7 |
| LiBforSecUse (NMC811) | 33 | 0.98 | 1.4 |

Other results:

- Absolute capacity from EIS plus a partial discharge: 1.8% MAPE.
- RUL on Zhang 2020: R2 0.77, against a baseline MAE more than twice as large.
- Tracking against an intake measurement of the same cell: R2 0.95, MAE 1.7 pp.
- Three EIS frequencies carry almost all of the SOH signal, which supports a faster test.
- Eleven pulse-resistance features recover about 95% of the EIS accuracy.

## Lessons

- A linear mean function was the largest single gain for GP models (+0.14 R2).
- Models fail on chemistries whose degradation range is absent from training; the cause is label range, not chemistry.
- Negative results are kept on the leaderboard next to positive ones.

## Tech stack

Python, PyTorch, GPyTorch, scikit-learn, LightGBM, TabPFN, pandas, Parquet, pytest.

## Contact

Riccardo Farris, LEITAT Technological Center.

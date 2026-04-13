# Project Context: MUSE LAE Pipeline

## Overview

This is a PhD research pipeline for characterising galactic outflows in high-redshift Lyman-alpha emitters (LAEs) and connecting those outflow properties to physical galaxy characteristics. The project uses integral field spectroscopy data from the VLT/MUSE instrument, targeting LAEs strongly lensed (for magnification) by foreground galaxy clusters.

The pipeline extracts spectra, fits spectral lines, applies radiative transfer models, and performs statistical analysis to understand the relationship between Lyman-alpha (Lyα) line morphology, ISM/CGM conditions (traced by metal emission and absorption lines), and inferred physical properties such as outflow velocity and HI column density.

## Sample

- **~1000 lensed images** drawn from **~650 individual LAEs** across 13 galaxy clusters
- **Redshift range:** 2.9 < z < 6.7 (set by MUSE wavelength coverage for Lyα)
- **Clusters:** A2744, A370, BULLET, MACS0257, MACS0329, MACS0416NE, MACS0416S, MACS0940, MACS1206, MACS2214, RXJ1347, SMACS2031, SMACS2131
- **Source catalogue:** Richard et al. (2021) — hereafter R21 — which provides the initial source positions, redshifts, and lensing magnifications (µ)
- **Lensing role:** Gravitational lensing is exploited purely for magnification; lens modelling is not a primary analysis component

## Data and Spectra

Two types of extracted spectra are used interchangeably:

| Type | Key | Description |
|------|-----|-------------|
| R21 | `R21` | Pre-extracted spectra from Richard et al. (2021); sky-subtracted variants `weight_skysub`, `noweight_skysub` |
| Aperture | `APER` | Re-extracted from MUSE data cubes using optimised circular apertures (1 or 2× FWHM of the Lyα emission centroid); variants `1fwhm_opt`, `2fwhm_opt` |

Aperture sizes and sky-subtraction methods are encoded in `SPEC_TYPE` strings (e.g. `2fwhm_opt`, `weight_skysub`). The `_opt` suffix indicates the aperture position was optimised by fitting the Lyα emission centroid.

## Pipeline Steps

The pipeline proceeds sequentially through numbered scripts/notebooks:

### Step 01 — Spectral Extraction and Fitting (`step01_spectral_extraction_and_fitting.py`)
- Run per cluster via shell scripts (e.g. `run_pipeline_2fwhm_opt_start.script`)
- Extracts aperture spectra from MUSE cubes (or loads R21 pre-extracted spectra)
- Fits the **Lyα line** first (double-peaked profile: red + blue components), then all other metal lines from the R21 catalogue, and forces fits for key diagnostic lines not in the catalogue (CIV1548, SiII1260, OIII1660, SiIV1394, CIII1907, HeII1640)
- Outputs per-cluster FITS tables: `{CLUSTER}_{SPEC_TYPE}_lines.fits` and `{CLUSTER}_{SPEC_TYPE}_lya.fits`
- Uses `tangelo` — an internal custom Python library for all spectral fitting, IO, and catalogue operations

### Step 02 — Megatable Compilation (`step02_make_megatable.ipynb`)
- Merges per-cluster results from Step 01 into a single `lae_megatab_{SPEC_TYPE}.fits` table
- Removes duplicate spectra from overlapping apertures
- One row per unique source (lensed images of the same source are identified and merged or flagged)

### Step 03 — Quality Control (`step03_quality_control.ipynb`)
- Identifies outliers using **Local Outlier Factor (LOF)** on Lyα fit parameters (DISPR, ASYMR, CONT, FLUXR_ERR)
- Checks for cluster-member contamination: if a similar line can be fitted at the same central velocity in a nearby cluster-member spectrum, the source is flagged `'c'`
- Outputs `lae_megatab_flagged_{SPEC_TYPE}.fits`

### Step 04 — Lensed Counterparts (`step04_lensed_counterparts.ipynb`)
- Identifies lensed multiple images (counterparts) of the same source using Lyα line profiles and the R21 catalogue
- Cleans possible [OII] doublet contamination
- Outputs `lae_megatab_flagged_cpts_{SPEC_TYPE}.fits`

### Step 05 — Radiative Transfer Model Fitting (`step05_fitting_rt_models.ipynb`)
- Fits homogeneous thin expanding shell Lyα radiative transfer (RT) models to each source using the **zELDA** package (Gurung-López et al. 2021)
- MCMC is used to constrain the posterior distributions of shell parameters
- Key zELDA output parameters (stored with `_ZELDA` suffix in megatable):
  - `VEXP_ZELDA` — expansion/outflow velocity of the HI shell (km/s)
  - `LOGN_ZELDA` — log₁₀ of the HI column density (cm⁻²)
  - `TAU_ZELDA` — dust optical depth
- Asymmetric errors stored as `{PARAM}_ERRM_ZELDA` (minus) and `{PARAM}_ERRP_ZELDA` (plus)
- Outputs `lae_megatab_flagged_cpts_refit_zeldamcmc_{SPEC_TYPE}.fits`

### Step 06 — Re-fitting Candidate Metal Lines (`step06_refitting_candidate_lines.ipynb`)
- All putative emission and absorption line detections are re-fitted using bootstrap resampling and autocorrelation-aware error estimation (`fit_mc` method in tangelo)
- Advanced flagging for potential systematics
- Outputs `lae_megatab_flagged_cpts_allrefit_zeldamcmc_{SPEC_TYPE}.fits`

### Step 07 — Systemic Redshifts (`step07_measuring_systemic_redshifts.ipynb`)
- Estimates systemic redshift for each source by stacking optically-thin emission lines (e.g. CIV, CIII], HeII, OIII]) and fitting Gaussian profiles
- `DELTAV_LYA` column: velocity offset of the Lyα red peak from the systemic redshift, a key indicator of outflow kinematics
- Outputs `lae_megatab_flagged_cpts_allrefit_zeldamcmc_sysz_{SPEC_TYPE}.fits`

### Step 08 — Absorption Lines and Outflow Velocities (`step08_absorption_lines.ipynb`)
- Searches for stacked interstellar absorption lines: primarily **SiII1260, CII1334, SiIV1394, SiIV1403**
- Measures centroid velocities of absorption lines relative to systemic redshift to characterise outflow velocities directly
- Outputs `lae_megatab_flagged_cpts_allrefit_zeldamcmc_sysz_absv_{SPEC_TYPE}.fits` — the final science-ready megatable

### Step 09 — Statistics and Correlations (`step09_megatable_statistics.ipynb`)
- Generates histograms, scatter plots, and searches for correlations between:
  - Lyα profile properties (FWHMR, DISPR, ASYMR, blue/red flux ratio BRRATIO, peak separation BRSEP, DELTAV_LYA)
  - Metal emission line properties (EW, FWHM, centroid velocity CVEL) for CIV1548, HeII1640, OIII1660, CIII1907
  - Metal absorption line properties (EW) for SiII1260, CII1334, SiIV1394, SiIV1403
  - zELDA model parameters (VEXP_ZELDA, LOGN_ZELDA)
  - UV continuum (CONT)
- Uses **LinMix** (Kelly 2007) MCMC regression accounting for measurement errors in both axes, plus optional bootstrap resampling for robustness
- Performs exploratory **Factor Analysis** with MICE imputation for handling non-detections

## Megatable Columns and Conventions

### Lyα parameters (from Step 01 fitting)

| Column | Description |
|--------|-------------|
| `FLUXR`, `FLUXR_ERR` | Red peak flux and uncertainty |
| `FLUXB`, `FLUXB_ERR` | Blue peak flux and uncertainty (NaN if not detected) |
| `LPEAKR`, `LPEAKR_ERR` | Observed wavelength of red Lyα peak |
| `LPEAKB`, `LPEAKB_ERR` | Observed wavelength of blue Lyα peak |
| `FWHMR`, `FWHMR_ERR` | FWHM of red peak (observed frame, Å) |
| `DISPR`, `DISPR_ERR` | Gaussian dispersion σ of red peak (Å) |
| `ASYMR`, `ASYMR_ERR` | Asymmetry of red peak |
| `CONT`, `CONT_ERR` | UV continuum level |
| `SNRR`, `SNRB` | Signal-to-noise of red/blue peaks |
| `z` | Redshift from Lyα red peak |
| `DELTAV_LYA` | Velocity offset of Lyα red peak from systemic (km/s) |

### Metal line parameters (per line, with `_{LINE}` suffix)

| Column | Description |
|--------|-------------|
| `FLUX_{LINE}`, `FLUX_ERR_{LINE}` | Line flux and uncertainty |
| `FWHM_{LINE}`, `FWHM_ERR_{LINE}` | FWHM of the line (Å) |
| `LPEAK_{LINE}`, `LPEAK_ERR_{LINE}` | Observed peak wavelength |
| `CONT_{LINE}`, `CONT_ERR_{LINE}` | Continuum at line position |
| `SNR_{LINE}` | Line SNR (negative for absorption) |
| `FLAG_{LINE}` | Quality flag (empty string = clean) |

### Derived quantities

- **Equivalent width (EW):** Calculated as `FLUX / CONT`, rest-frame corrected as `EW / (1 + z)`
- **Absorption EW convention:** Sign-flipped so positive values indicate stronger absorption
- **Instrumental correction:** All FWHM and dispersion measurements are corrected for the MUSE line spread function (LSF) using the `muse_lsf_fwhm_poly` polynomial fit, applied in quadrature: `FWHM_intrinsic = sqrt(FWHM_obs² - FWHM_LSF²)`
- **Centroid velocity (CVEL):** Velocity offset of line centroid from rest-frame wavelength, relative to systemic redshift

### Detection thresholds

- **Significant detection (emission):** `SNR > 3.0`
- **Significant detection (absorption):** `SNR < -3.0`
- **Continuum detection required for EW:** `CONT / CONT_ERR > 3.0`
- **Clean sources require:** `FLAG_{LINE} == ''`

## Key Science Questions

1. Do outflow-sensitive Lyα profile properties (FWHMR, DISPR, DELTAV_LYA, BRSEP) correlate with metal absorption line strengths and velocities?
2. Do radiative transfer model parameters (VEXP_ZELDA, LOGN_ZELDA) correlate with directly-measured absorption line outflow velocities?
3. What is the relationship between the UV continuum (proxy for star formation rate) and outflow signatures?
4. Can Lyα profile morphology be used as a reliable proxy for outflow velocity or HI column density?

## Libraries and Dependencies

- **`tangelo`** — internal custom library for all MUSE spectral extraction, fitting, IO, catalogue operations, quality control, and plotting
- **`astropy`** — FITS IO, table operations
- **`numpy`, `scipy`** — numerical operations, ODR regression, statistical tests
- **`LinMix`** — Bayesian linear regression accounting for measurement errors in x and y (Kelly 2007)
- **`zELDA` (`Lya_zelda`)** — Lyα radiative transfer RT model grids and MCMC fitting (Gurung-López et al. 2021)
- **`sklearn`** — Factor Analysis, MICE imputation (IterativeImputer + BayesianRidge)
- **`matplotlib`** — visualisation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jnianias) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->

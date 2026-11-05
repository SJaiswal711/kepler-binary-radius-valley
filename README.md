# Kepler Binary Hosts and the Small-Planet Radius Valley

This repository contains the notebook working pipeline for comparing the observed radius distribution of small Kepler planets in unresolved binary-host systems with an apparent-single reference population. It also contains the latest host-marginalisation, finite-sample and system-bootstrap figures shared during the analysis.

The six notebooks are numbered in dependency order. Notebook 05 is intentionally run twice: first to freeze the comparison sample, then again after Notebook 06 has generated host probabilities.

> **Research status:** work in progress. The current products describe selected, detected planets. They are not completeness-corrected occurrence rates. The host probabilities are a first-pass density-based model and have not been independently calibrated. The current binary–single difference in the adopted valley-occupancy statistic is statistically unresolved.

---

## 1. Scientific scope

The project asks how the inferred small-planet radius distribution changes when the planet-hosting component of an unresolved Kepler binary is not assumed to be the primary star.

The working analysis combines:

1. The Sullivan et al. (2024) Kepler planet-host binary catalogue.
2. Joint stellar characterisation of both binary components with MIST v1.2 isochrones.
3. Planet-by-planet primary/secondary host probabilities using Kepler transit-derived stellar density.
4. Fulton-like selection of binary-host and apparent-single reference samples.
5. Alternative planet-radius distributions under catalogue, primary-host, secondary-host and probability-marginalised interpretations.
6. Exploratory radius-valley occupancy, system-level subsampling and bootstrap analyses.

The principal scientific question is:

> **How sensitive is the observed small-planet radius distribution—and its apparent radius-valley occupancy—to uncertainty in identifying the planet-hosting component of an unresolved binary?**

This is deliberately narrower than an intrinsic planet-occurrence analysis. Companion dilution can affect both inferred planet radii and Kepler detection efficiency; the current six-notebook workflow does not yet apply a validated binary-aware detection-completeness correction.

## 2. Repository contents

```text
kepler-binary-radius-valley/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── 01_retrieve_kepler_reference_catalogues.ipynb
│   ├── 02_retrieve_sullivan_binary_catalogue.ipynb
│   ├── 03_fit_binary_stellar_components_with_mist.ipynb
│   ├── 04_quality_control_and_finalize_binary_sample.ipynb
│   ├── 05_build_fulton_comparison_and_radius_distributions.ipynb
│   └── 06_infer_probabilistic_planet_hosts.ipynb
├── figures/
│   ├── 01_host_marginalised_radius_distribution.png
│   ├── 02_system_level_subsampling_10000.png
│   ├── 03_system_bootstrap_delta_fvalley.png
│   └── 04_primary_vs_marginalised_distribution.png
├── data/
│   ├── raw/           # downloaded public catalogues; not bundled
│   ├── reference/     # cached VizieR tables; not bundled
│   └── processed/     # generated intermediate tables; not bundled
└── outputs/           # generated analysis products; not bundled
```

The six notebooks are copied from the latest available project snapshots reviewed for this README. Their original analysis logic is preserved; filenames have been simplified by removing timestamp suffixes.

### Important notebook-version note

The comparison notebook (Notebook 05) is a 27-cell snapshot of the Fulton-like catalogue construction, sample cutflow, and host-radius sensitivity plots. The working analysis uses MIST-derived stellar information; the 396-planet/285-system parent described below is nevertheless the **pre-MIST-quality-cut** parent for this comparison branch. Do not confuse it with the separate 396-planet/278-system post-QC input used by Notebook 06.

The `F_valley`, probability-marginalised histogram, 10,000-draw subsampling, and system-bootstrap analyses were supplied and run as follow-on cells after this notebook snapshot. **Before publication, merge those follow-on cells into Notebook 05** so the sample construction and final analysis are reproducible from the repository. The intended run order is Notebook 05 (sample-construction pass), Notebook 06 (host inference), then Notebook 05 again (final host-marginalised analysis).
## 3. Notebook-by-notebook guide

### Notebook 01 — `01_retrieve_kepler_reference_catalogues.ipynb`

**Purpose:** retrieve Kepler and comparison catalogues used by the radius-distribution analysis and run preliminary catalogue checks/plots.

This notebook retrieves:

- The Kepler DR25 KOI table from the NASA Exoplanet Archive TAP service (`q1_q17_dr25_koi`).
- Confirmed-planet parameters from the NASA Exoplanet Archive `pscomppars` table (with non-null planet radii; Kepler discoveries are selected later for comparison).
- CKS 2017 tables: VizieR/CDS `J/AJ/154/108` (stellar table 1 and planet table 4, including a VizieR VOTable version of table 4).
- Fulton et al. (2017): `J/AJ/154/109` table 2 and ReadMe.
- Fulton & Petigura (2018): `J/AJ/156/264` tables 2, 3 and 4 and ReadMe.

The notebook parses the CDS tables and creates preliminary raw-count and normalized-radius comparisons. It is a data-retrieval and exploratory-comparison notebook; Notebook 05 is the principal harmonized Fulton-like comparison.

**Main raw-data paths:**

- `data/raw/nasa/nasa_kepler_dr25_koi.csv`
- `data/raw/nasa/confirmed_planets_pscomppars.csv`
- `data/raw/cks/J_AJ_154_108/`
- `data/raw/fulton/J_AJ_154_109/`
- `data/raw/fulton/J_AJ_156_264/`

### Notebook 02 — `02_retrieve_sullivan_binary_catalogue.ipynb`

**Purpose:** retrieve and validate the Sullivan et al. (2024) binary catalogue, then map planet rows to their binary-system stellar properties.

**Source:** VizieR catalogue `J/AJ/168/129`.

The notebook retrieves four tables and saves them without changing their source content:

| Table | Retrieved shape | Role |
|---|---:|---|
| Stellar sample | 286 × 52 | One row per binary system |
| Auxiliary | 82 × 13 | Supplementary catalogue information |
| Planet table | 404 × 28 | Planet-level rows |
| Notes | 65 × 8 | Catalogue notes |

The identifiers have different meanings:

- `KOI` is the **binary-system** identifier.
- `KOIpl` is the **individual planet** identifier.

The planet table is left-joined to the stellar table using a normalized system-level `KOI` key with a validated many-to-one merge. The notebook preserves the original identifiers and writes matched and unmatched products separately.

**Executed checks:** 286 unique stellar systems; 404 unique planet identifiers; all 404 planet rows mapped; zero unmatched planet rows; 77 systems with multiple planets; maximum five planets in one system.

**Main outputs:**

- `data/raw/sullivan/sullivan_sample_table.csv`
- `data/raw/sullivan/sullivan_auxiliary_table.csv`
- `data/raw/sullivan/sullivan_planet_table.csv`
- `data/raw/sullivan/sullivan_notes_table.csv`
- `data/processed/sullivan2024_planets_with_stellar_properties.csv`
- `data/processed/sullivan2024_planets_with_stellar_match.csv`
- `data/processed/sullivan2024_planets_without_stellar_match.csv`
- `data/processed/sullivan2024_star_planet_mapping_audit.csv`

Notebook 02 does **not** apply the later MIST quality cut.

### Notebook 03 — `03_fit_binary_stellar_components_with_mist.ipynb`

**Purpose:** estimate the stellar properties of both components of the 286 Sullivan binary systems.

**Isochrone input:** MIST v1.2, `vvcrit0.0_basic` files. The grid directory expected by the code is:

```text
MIST_v1.2_vvcrit0.0_basic_isos/
```

The executed notebook found 15 metallicity grids spanning `[Fe/H] = -4.0` to `+0.5`. The requested age interval is 0.1–13.8 Gyr, but the available isochrone files span 0.1–12.589254 Gyr. The fitting uses main-sequence points (`phase == 0`), 80 logarithmically spaced target ages, and a 300-point mass grid from 0.1 to 2.5 solar masses. After collapsing repeated requested ages onto available isochrones, the notebook reports 645 unique MIST model combinations.

For each binary, the two components share a common age and metallicity, while their masses are independently inferred. The likelihood uses the measured effective temperatures and stellar radii of both components. The notebook marginalises over mass for each component, combines the two component evidences for each common evolutionary model, and reports posterior mass quantiles as well as a joint maximum-likelihood solution.

A second, simplified comparison is also calculated using a solar-metallicity, 1-Gyr main-sequence MIST isochrone. This is a baseline comparison, not an independent validation of the joint model.

Stellar density is calculated using the MIST-derived median mass and the catalogue-measured stellar radius:

\[
\rho_\star = 1.408\,\frac{M_{\star,50}/M_\odot}{(R_{\star,\mathrm{catalogue}}/R_\odot)^3}
\quad \mathrm{g\,cm^{-3}}.
\]

The notebook reports all 286 systems as fit status `OK`. It also propagates approximate lower and upper density uncertainties from mass-posterior quantiles and the measured stellar-radius uncertainty.

**Main outputs:**

- `data/processed/sullivan2024_joint_mist_masses_densities.csv`
- `data/processed/sullivan2024_joint_and_solar_mist_comparison.csv`

### Notebook 04 — `04_quality_control_and_finalize_binary_sample.ipynb`

**Purpose:** apply a documented system-level quality-control rule to the MIST fits and build the planet-level handoff table.

The quality check examines four normalized residuals:

1. Primary effective temperature.
2. Primary stellar radius.
3. Secondary effective temperature.
4. Secondary stellar radius.

A system is excluded if any required residual is missing or if any absolute normalized residual exceeds 5. Age- and metallicity-boundary flags are recorded; they are not automatic rejection criteria.

**Executed quality-control results:**

| Quantity | Result |
|---|---:|
| Initial fitted systems | 286 |
| Excluded systems | 8 |
| Retained systems | 278 |
| Boundary-flagged systems before QC | 18 |
| Boundary-flagged systems retained | 14 |
| Boundary-flagged systems excluded | 4 |
| Source Sullivan planet rows | 404 |
| Planets in retained systems | 396 |
| Planets in excluded systems | 8 |
| Unmatched planet rows | 0 |

The notebook also checks that the reported minimum chi-square equals the sum of the squared four normalized residuals. This is an internal arithmetic consistency check, not by itself a goodness-of-fit validation.

The largest retained secondary-star density is not a typical population value: the reported maximum is 244.503 g cm⁻³. Extreme stellar solutions should be inspected before physical interpretation.

**Main outputs:**

- `data/processed/sullivan2024_mist_quality_controlled.csv`
- `data/processed/sullivan2024_mist_excluded_poor_fits.csv`
- `data/processed/sullivan2024_mist_quality_control_summary.csv`
- `data/processed/sullivan2024_planets_with_mist_properties_all.csv`
- `data/processed/sullivan2024_planets_quality_controlled.csv`
- `data/processed/sullivan2024_planets_in_excluded_systems.csv`
- `data/processed/sullivan2024_planets_without_stellar_match.csv`
- `data/processed/sullivan2024_planet_mapping_summary.csv`

### Notebook 05 — `05_build_fulton_comparison_and_radius_distributions.ipynb`

**Purpose:** construct the main observed-radius comparison using DR25 confirmed planets, the published Fulton et al. (2017) selected sample, CKS planet radii, and Sullivan binary-host planets.

**MIST information is part of the Notebook 05 working analysis.** The MIST-derived stellar information is used in the binary-host characterization in this notebook; it is important to distinguish that use from the point at which the Sullivan parent sample is defined and from the later system-level MIST quality-control cut. In particular, using MIST-derived quantities in Notebook 05 does not mean that the initial 396-planet/285-system parent has already passed the same MIST quality cut used in Notebook 04.

The notebook's Sullivan branch is constructed from the Sullivan catalogue after the initial `Rpkep` radius-domain gate. That starting parent contains **396 planet rows in 285 binary systems**. The same MIST system-quality cut is a later operation and must be reported as a separate cutflow stage; do not label the 396/285 parent as the MIST-quality-controlled sample.

The notebook applies the following Fulton-like cuts where the required columns are available:

| Selection | Criterion |
|---|---|
| Selection radius domain | 0.5–20 R⊕ |
| Kepler magnitude | `Kp < 14.2` |
| Impact parameter | `0 <= b < 0.7` |
| Orbital period | `0 < P < 100` days |
| Dwarf/evolved-star boundary | `Rstar <= 10**(0.00025*(Teff - 5500) + 0.20)` R☉ |
| Effective temperature | `4700 < Teff < 6500` K |

Plots are restricted to 0.5–4 R⊕. For the Sullivan sample, the initial radius-domain gate uses the Kepler catalogue radius `Rpkep`; the final host-radius sensitivity plots show `Rpkep`, `Rppri` and `Rpsec` as alternative radius estimates for the same planet rows. MIST-derived component information is used in the Notebook 05 working analysis, while the pre-QC parent count remains 396 planets in 285 systems. The exact MIST-QC result for this separate branch must be taken from Notebook 05's own sequential cut ledger after the MIST cut is applied, rather than copied from Notebook 04/06. DR25 transit metadata and disposition are joined by planet identifier.

The main DR25 apparent-single reference is constructed by removing systems listed in the Sullivan binary catalogue from the DR25 confirmed sample after the Fulton-like cuts. “Apparent single” is intentional terminology: absence from the Sullivan list does not prove that a star is physically single.

**Executed principal comparison:**

| Population | Planets | Systems | Radius used |
|---|---:|---:|---|
| DR25 confirmed non-Sullivan | 520 | 395 | `Rp_DR25` |
| Fulton et al. (2017) published selected sample | 892 | 650 | `Rp_Fulton` |
| CKS Fulton-like | 689 | 527 | `Rp_CKS` |
| Sullivan confirmed-only | 63 | 46 | `Rppri` |
| Sullivan candidate + confirmed | 90 | 66 | `Rppri` |

The 63-planet binary sample is the confirmed-only subset; the 90-planet sample adds 27 candidates. “Confirmed” here means the Kepler DR25 planet disposition, not a confirmation category for the binary itself.

The Fulton 2017 table is already a published filtered sample. The notebook can recheck its available radius and period columns, but cannot independently reapply all original cuts because all required columns are not present. CKS and Fulton are not independent planet populations: the Fulton table is drawn from the CKS programme.

**Main outputs:**

- `outputs/fulton_filtered_distribution_comparison/population_summary.csv`
- `outputs/fulton_filtered_distribution_comparison/fulton_like_cutflow_all_catalogues.csv`
- `outputs/fulton_filtered_distribution_comparison/input_and_selection_manifest.csv`
- `outputs/fulton_filtered_distribution_comparison/dr25_confirmed_single_reference.csv`
- `outputs/fulton_filtered_distribution_comparison/cks_fulton_like_selected.csv`
- `outputs/fulton_filtered_distribution_comparison/fulton_2017_published_selected.csv`
- `outputs/fulton_filtered_distribution_comparison/sullivan_confirmed_only_fulton_like.csv`
- `outputs/fulton_filtered_distribution_comparison/sullivan_candidate_confirmed_fulton_like.csv`
- Figures for catalogue-specific filter evolution, Sullivan host-radius sensitivity, and raw-count histograms.

The notebook's histograms/KDEs describe the selected detected-planet samples. They are not completeness-corrected occurrence distributions.

### Notebook 06 — `06_infer_probabilistic_planet_hosts.ipynb`

**Purpose:** estimate whether each planet in an unresolved binary orbits the primary or secondary component.

The notebook uses the Kepler DR25 transit-derived stellar density, joined to the planet-level Sullivan table by normalized `KOIpl` rather than by system `KOI`. Its current input is the MIST-quality-controlled planet table: 396 planet rows in 278 systems.

The transit-density measurement is `koi_srho`; its asymmetric reported errors are converted to a symmetric approximation using the mean of their absolute magnitudes. The component density hypotheses are `rho1_joint_gcm3` and `rho2_joint_gcm3`. In the current pipeline those quantities are calculated from MIST median mass plus catalogue radius; they are not densities based on an independently MIST-predicted stellar radius.

For each component, the likelihood is a Gaussian approximation in density. It combines transit-density and model-density variances and uses the lower or upper stellar-density uncertainty according to which side of the model density the observed density falls on. The baseline priors are equal:

\[
P(H_1)=P(H_2)=0.5.
\]

The notebook reports `p_pri`, `p_sec`, `host_MAP`, entropy and confidence diagnostics. It also repeats the calculation with stellar-density uncertainty inflation factors of 1, 1.5 and 2.

**Executed results:**

| Diagnostic | Baseline result |
|---|---:|
| Input planets | 396 |
| Valid host probabilities | 361 |
| Invalid rows retained with NaN probabilities | 35 |
| Median primary probability among valid rows | 0.671898 |
| Fraction with `p_pri > 0.9` | 39.34% |
| Fraction with `p_sec > 0.9` | 3.32% |
| Fraction with `0.1 <= p_pri <= 0.9` | 57.34% |

The 35 invalid rows are retained in the output catalogue with NaN probabilities. In the executed run, 33 rows lacked finite transit density and 35 lacked a positive finite transit-density uncertainty; some rows can contribute to both counts.

Increasing the model-density uncertainty reduces the median primary probability and increases the ambiguous fraction. For example, at uncertainty inflation ×2, the median primary probability is 0.639110 and the ambiguous fraction is 63.16%.

**Critical interpretation:** these are probabilities conditional on the adopted likelihood, uncertainty model and equal priors. They are not calibrated 90% guarantees. The current model does not marginalise over eccentricity/photoeccentric effects, include a validated physical primary/secondary prior, fully model transit-fit covariance, or account for all transit-model systematics.

**Execution-order clarification:** Notebook 06 is run *after the first/sample-selection pass of Notebook 05*, as requested for the publication workflow. Its stellar-density inputs still originate upstream from Notebooks 03/04, and its transit-density inputs come from DR25. The host-probability catalogue is then joined back to the frozen Notebook 05 binary sample by normalized planet identifier (`KOIpl`). The current Notebook 06 implementation calculates probabilities for its full 396-planet/278-system MIST-QC input; the Notebook 05 merge must explicitly audit which selected planet IDs receive valid probabilities. The workflow is not a single-pass linear run: Notebook 05 first defines the comparison cohort, Notebook 06 computes host probabilities, and Notebook 05 is then reopened for the host-marginalised analysis.

**Main outputs:**

- `data/processed/binary_host_probability_catalog.csv`
- `data/processed/binary_host_probability_summary.csv`
- `data/processed/binary_host_probability_sensitivity.csv`

## 4. Do not conflate the two 396-planet samples

There are two distinct 396-row parent tables in the workflow. They happen to have the same number of planet rows, but differ in their system membership and selection history:

| Branch | Parent definition | Count |
|---|---|---:|
| Notebook 04/06 | Sullivan catalogue after the MIST system-quality cut | 396 planets in 278 systems |
| Notebook 05 | Sullivan catalogue after the initial `Rpkep` radius-domain gate, before the same MIST quality cut | 396 planets in 285 systems |

**Do not treat these as the same sample.** Notebook 04/06's 396/278 population is the MIST-quality-controlled input used by the current host-inference implementation. Notebook 05 starts from its own 396/285 radius-gated parent and uses MIST-derived stellar information in its working binary characterization; its own MIST-QC step, where applied, is a distinct downstream cutflow stage. Its post-MIST-QC count must be reported from Notebook 05's own ledger. The intended publication sequence is to define and freeze the selected comparison sample in Notebook 05 first, run host inference in Notebook 06 second, and then merge the probabilities back into Notebook 05 by normalized planet ID (`KOIpl`). The merge must report matched, unmatched and invalid-probability rows and must not silently replace Notebook 05's selected sample with Notebook 04/06's 396/278 input population.

## 5. Follow-on analysis added after the Notebook 05 snapshot

The following analyses were developed as additional cells after the uploaded six-notebook snapshot. They use the selected 63-planet/46-system Sullivan confirmed sample and the 520-planet/395-system DR25 apparent-single reference, with the host-probability catalogue produced by Notebook 06.

### 5.1 Period-dependent valley-occupancy statistic

The exploratory estimator currently uses the period-dependent valley centre

\[
R_{\rm valley}(P)=2.25-0.26\log_{10}\left(\frac{P}{\rm day}\right),
\]

and the local occupancy ratio

\[
F_{\rm valley}=\frac{\sum_i w_i\,\mathbf{1}(|R_{p,i}-R_{\rm valley}(P_i)|\leq0.20)}{\sum_i w_i\,\mathbf{1}(|R_{p,i}-R_{\rm valley}(P_i)|\leq0.60)}.
\]

The windows are in linear Earth radii. For a single-host sample, `w_i = 1`; for the expected host-marginalised binary statistic, the primary and secondary radius hypotheses contribute weights `p_pri` and `p_sec`, respectively. This is a selected-sample occupancy statistic, not an intrinsic occurrence-rate estimator. The locus and window widths remain candidates for validation and sensitivity testing; they should not be treated as a universal or final valley definition without further checks.

### 5.2 Probability-marginalised radius distribution

For each planet, the radius distribution combines the two alternative host hypotheses:

\[
p(R_p\mid D_i)=p(H_1\mid D_i)p(R_p\mid H_1,D_i)+p(H_2\mid D_i)p(R_p\mid H_2,D_i).
\]

The expected histogram uses the primary and secondary radius estimates with weights `p_pri` and `p_sec`. Separate discrete host assignments are drawn to show a 16th–84th percentile host-assignment band. The current figure uses logarithmic radius bins from 0.5 to 4 R⊕ and reports normalized density per dex.

**Observed values in the current run:**

| Radius treatment | `F_valley` |
|---|---:|
| Binary catalogue radius | 0.166667 |
| Binary primary-host radius | 0.400000 |
| Binary secondary-host radius | 0.300000 |
| Binary probability-marginalised | 0.380929 |
| DR25 apparent-single reference | 0.320442 |

The probability-marginalised statistic is 0.019071 below the primary-only value (−4.77%). Under this estimator, that small reduction means marginalisation makes the valley slightly more depleted relative to its surrounding window; it does **not** demonstrate that host ambiguity fills or flattens the valley.

### 5.3 System-level finite-sample subsampling

The 10,000-draw experiment samples 46 reference systems without replacement and retains all planets in each selected system. This preserves system clustering and naturally allows the number of planets per draw to vary. It is **not yet covariate matched** in stellar mass, effective temperature, period or multiplicity, and should be described as an initial finite-sample diagnostic rather than a calibrated null test.

| Quantity | Current result |
|---|---:|
| Draws | 10,000 |
| Systems per draw | 46 |
| Median planets per draw | 60 |
| 16th–84th percentile planet count | 56–65 |
| Median subsample `F_valley` | 0.318182 |
| 2.5th–97.5th percentile | 0.136364–0.526316 |
| Observed binary `F_valley` | 0.380929 |
| Empirical two-sided tail fraction reported by the exploratory code | 0.556 |

The 0.556 value is an empirical position/tail diagnostic from this particular subsampling implementation; it is **not** a formal p-value and should not be reported as one.

### 5.4 System-level bootstrap

The current 10,000-realisation bootstrap resamples host systems, not individual planet rows, in the binary and apparent-single samples. A host component is also redrawn for each planet in each binary bootstrap realisation. Planet-radius measurement errors are not yet propagated, and the binary/reference samples are not covariate matched.

| Bootstrap statistic | Median | 68% interval | 95% interval |
|---|---:|---:|---:|
| Binary `F_valley` | 0.379310 | 0.285714 to 0.470588 | 0.200000 to 0.562500 |
| Apparent-single `F_valley` | 0.320000 | 0.284974 to 0.356250 | 0.251429 to 0.392473 |
| Difference, binary − single | +0.057971 | −0.039570 to +0.155051 | −0.133855 to +0.251658 |

The 95% interval for the difference includes zero. The current data therefore do not establish a statistically resolved binary–single difference in valley occupancy.

### 5.5 Latest figures

**Host-marginalised binary distribution compared with the apparent-single reference**

![Host-marginalised radius distribution](figures/01_host_marginalised_radius_distribution.png)

**10,000 system-level subsampling experiments**

![10,000 system-level subsampling experiments](figures/02_system_level_subsampling_10000.png)

**System-level bootstrap distribution of the binary–single contrast**

![System-level bootstrap contrast](figures/03_system_bootstrap_delta_fvalley.png)

**Primary-only versus probability-marginalised binary radius distribution**

![Primary versus marginalised distribution](figures/04_primary_vs_marginalised_distribution.png)

## 6. Reproducing the workflow

### 6.1 Software

The notebooks use Python with Jupyter/IPython and the following libraries:

- NumPy
- pandas
- SciPy
- Matplotlib
- Astropy
- Astroquery
- requests
- tqdm

The MIST code uses `numpy.trapezoid`; use NumPy 2.0 or later. A starting dependency list is provided in `requirements.txt`. The exact package versions used for the original Windows notebook runs were not pinned in the supplied notebooks, so the requirements file is a working environment specification rather than a lockfile.

### 6.2 External data required

The raw catalogues and MIST isochrone files are not included in this repository starter. They are retrieved or supplied as follows:

| Data | Source / identifier | Used by |
|---|---|---|
| Sullivan binary catalogue tables | VizieR `J/AJ/168/129` | Notebook 02, then 03/04/05/06 |
| Kepler DR25 KOI catalogue | NASA Exoplanet Archive TAP, `q1_q17_dr25_koi` | Notebook 01, 05, 06 |
| Confirmed-planet parameters | NASA Exoplanet Archive TAP, `pscomppars` | Notebook 01 exploratory comparison |
| CKS 2017 | VizieR/CDS `J/AJ/154/108`, tables 1 and 4 | Notebook 01, 05 |
| Fulton et al. (2017) | VizieR/CDS `J/AJ/154/109`, table 2 | Notebook 01, 05 |
| Fulton & Petigura (2018) | VizieR/CDS `J/AJ/156/264`, tables 2–4 | Notebook 01 exploratory comparison |
| MIST isochrones | MIST v1.2, `vvcrit0.0_basic` | Notebook 03 |

The MIST directory must be placed at the project root using the path expected by Notebook 03:

```text
MIST_v1.2_vvcrit0.0_basic_isos/
```

### 6.3 Recommended execution order

Run from the repository root so the notebooks' relative `data/` paths resolve. The numbered files are arranged by dependency, but the publication workflow is deliberately **two-pass** for Notebook 05: it must define the comparison cohort before host inference, then be reopened after host inference to complete the probability-marginalised analysis.

1. **Notebook 01 — reference catalogues.** Retrieve Kepler DR25, confirmed-planet parameters, CKS and Fulton comparison tables. These data support the apparent-single reference, Fulton-like comparison and DR25 transit-density join.
2. **Notebook 02 — Sullivan binary catalogue.** Retrieve the Sullivan et al. (2024) tables, validate identifiers and map planet rows to binary-system properties.
3. **Notebook 03 — MIST stellar fitting.** Run the joint MIST fits for both binary components. Requires the MIST `.iso` grid directory; produces component stellar parameters and density inputs.
4. **Notebook 04 — sample quality control.** Apply the residual-based MIST QC and create the 396-planet/278-system MIST-quality-controlled planet table used by the current host-inference implementation.
5. **Notebook 05 — first pass (comparison-sample construction).** Apply the Fulton-like filters, incorporate the relevant MIST-derived stellar information, establish the MIST-QC cutflow for its own 396/285 parent, and freeze the selected binary and apparent-single comparison samples. Save the exact selected planet/system identifiers and the sample manifest.
6. **Notebook 06 — probabilistic host inference.** Calculate primary/secondary host probabilities using the MIST-derived component-density information from Notebooks 03/04 and DR25 transit-derived densities. The current Notebook 06 implementation reads the 396/278 MIST-QC table. Explicitly intersect/audit its output against the frozen Notebook 05 binary planet IDs using normalized `KOIpl`; report matched, unmatched, valid and invalid probability rows. Do not assume equal row counts imply identical planet samples.
7. **Return to Notebook 05 — second pass (final inference and figures).** Load the host-probability output, merge by normalized `KOIpl`, audit the join, and run the primary/secondary/MAP/probability-marginalised radius analyses, `F_valley`, finite-sample subsampling and system-level bootstrap. Save these follow-on cells and outputs in Notebook 05 so a fresh-clone reproduction is complete.

If automatic project-root detection in Notebook 05 does not find the project directory, set `PROJECT_ROOT` manually in its first setup cell.
### 6.4 Data and output policy

Large downloaded catalogues, MIST grids, intermediate CSV files and generated output tables are excluded by `.gitignore`. Keep a record of the catalogue identifiers, retrieval dates, table names and selection manifest when preparing a publication. Do not commit proprietary or access-restricted material. The four current PNG diagnostics are included because they are small and directly document the latest analysis state.

## 7. Current scientific limitations and required work

The following are not yet complete and should not be presented as completed results:

1. **Host-probability calibration.** Equal priors and a Gaussian density likelihood are provisional. Validate against synthetic known-host systems and any independently identified real hosts; test physically justified prior choices. Include eccentricity/photoeccentric effects where the data support them.
2. **Stellar-model sensitivity.** Inspect the extreme MIST density solutions, age/metallicity/mass-grid boundary flags, measurement-error assumptions and residual-cut sensitivity.
3. **Matched comparison.** The 63/46 binary sample and 520/395 apparent-single reference have different selection histories. Add covariate matching or stratification in stellar mass, effective temperature, orbital period and multiplicity where possible.
4. **Radius-error propagation.** The current host-assignment histograms and bootstrap do not propagate the asymmetric planet-radius measurement errors. Add radius draws jointly with host assignment.
5. **Valley-estimator validation.** Test the assumed valley locus and narrow/broad window widths. A shifted valley could otherwise be confused with a change in valley depth.
6. **Selection/completeness.** The current curves describe selected detected planets. A completeness-aware intrinsic occurrence analysis requires Kepler per-target detection efficiencies, companion flux contrasts in the Kepler band, validated crossmatches and a binary-aware detection model. Radius correction and detection-completeness correction are separate tasks.
7. **Reference-sample interpretation.** “Non-Sullivan” means absent from the Sullivan catalogue, not demonstrably single. CKS and Fulton 2017 are overlapping catalogue products, not independent replication samples.

The current strongest defensible statement is that the inferred binary radius distribution is sensitive to the assumed host-radius treatment, while the current binary–single difference in the exploratory `F_valley` statistic is statistically unresolved.

## 8. Repository and publication positioning

The project is currently best described as an **observed-radius-distribution and host-assignment analysis**. It should not claim an intrinsic occurrence-rate result or a confirmed physical difference between binary and single-star populations until the host model, sample matching, radius uncertainties and completeness treatment are strengthened.

A possible future paper question is:

> Does the apparent radius-valley morphology of Kepler planets in binary-host systems persist after propagating uncertainty in the identity of the host component and controlling for the different selection histories of the binary and reference samples?

## 9. Suggested repository names

1. **`kepler-binary-radius-valley`** — recommended; specific, searchable and broad enough for the current radius-distribution project.
2. `kepler-binary-host-inference` — emphasizes the host-component inference method.
3. `binary-host-radius-distributions` — broad and descriptive.
4. `kepler-multiplicity-radius-valley` — emphasizes stellar multiplicity and the radius valley.
5. `host-ambiguity-kepler-planets` — emphasizes the central methodological issue.

---

## Acknowledgement of data sources

The analysis uses public catalogues from VizieR/CDS, the NASA Exoplanet Archive and the MIST stellar-evolution grid. Cite the original catalogue papers and data releases in any manuscript and include the exact data-release or catalogue version used for the final rerun. The Sullivan et al. (2024) catalogue is the principal binary-host input; Kepler DR25 supplies planet disposition, transit and stellar metadata used by the current selection and host-density workflow.

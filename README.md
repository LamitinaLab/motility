# C. elegans Swimming Motility Analysis Pipeline

A Jupyter notebook-based pipeline for quantitative analysis of *C. elegans* swimming behavior from WormLab tracking exports.

The project is designed to keep the biological measurements transparent and reproducible. Wherever practical, the pipeline uses minimally processed WormLab outputs as inputs and calculates motility measurements independently rather than relying on opaque or configurable WormLab-derived behavioral classifications.

---

## Overview

The analysis is organized into three sequential notebooks:

1. **01_Data_Wrangling.ipynb**

   * Imports WormLab feature exports
   * Aligns track and frame data
   * Applies tracking-quality control
   * Calculates XY-derived lookback speed
   * Processes midpoint bending-angle data
   * Produces processed datasets for downstream analysis

2. **02_Analysis_and_Plotting.ipynb**

   * Calculates and summarizes motility measurements
   * Performs Fit-quality analysis
   * Analyzes speed and thrashing
   * Generates statistical summaries and figures
   * Produces files suitable for downstream analysis and Prism import

3. **03_Archiving.ipynb**

   * Packages raw data, processed data, analysis results, metadata, figures, and notebooks
   * Generates reproducibility documentation
   * Creates timestamped archives

The general workflow is:

```text
WormLab exports
    ↓
01 — Data Wrangling
    ↓
Processed frame-level and track-level data
    ↓
02 — Analysis & Plotting
    ↓
Metrics, QC summaries, statistics, and figures
    ↓
03 — Archiving
    ↓
Reproducible experiment archive
```

---

# Scientific Design Principles

## Transparent measurement

WormLab is used primarily as the tracking and feature-extraction system.

Whenever possible, biological measurements are calculated from relatively direct WormLab outputs rather than relying on WormLab-derived classifications or calculations whose algorithms or thresholds are not sufficiently transparent.

For example:

* Translational speed is calculated from exported XY position rather than using WormLab's exported Speed measurement.
* Fit is treated as a quality-control variable rather than a motility measurement.
* WormLab Reversal Mobility, Pirouette, and Omega Bend classifications are not currently required because their definitions may depend on configurable WormLab settings.

---

## Preserve biologically interpretable dimensions

The project does not currently combine measurements into a single "motility score."

The long-term analysis framework preserves distinct biological dimensions:

* Translational locomotion
* Body movement
* Activity pattern
* Behavioral dynamics
* Propulsive efficiency

This makes it possible to distinguish phenotypes such as:

* high translation + low body movement
* high translation + high body movement
* low translation + high body movement
* low translation + low body movement

---

# Current WormLab Inputs

The current minimal-input strategy is based on the following WormLab exports:

## Required

### Position

Provides:

* Frame
* Time
* X position
* Y position

XY Position is the primary input for translational locomotion analysis.

---

### Fit

Provides the WormLab body-fit quality measurement for each tracked worm and frame.

Fit is used as a quality-control variable.

Current default threshold:

```python
FIT_THRESHOLD = 0.90
```

Frames below the threshold are treated as unreliable for speed calculations.

---

### Bending Angle — Mid-Point

Provides a signed body bending-angle time series for each worm.

This signal is used for two distinct measurements: QC-aware body angular rate (movement magnitude/rate) and canonical thrashing frequency (sustained periodic oscillation).

---

## Under evaluation

### Length

Worm length is being evaluated as a secondary quality-control measurement.

Potential uses include detecting abnormal fitted body geometry or frames with implausible length changes.

Length is not currently used as a hard exclusion criterion.

---

# Speed Calculation

## Why WormLab Speed is not used

WormLab's exported Speed measurement is not used as the primary translational speed measurement because the details of its calculation are not sufficiently transparent for the intended quantitative analysis.

Instead, speed is calculated directly from exported XY Position.

---

## Lookback speed

Frame-to-frame centroid displacement can be highly sensitive to tracking and centroid jitter.

The pipeline therefore uses a lookback displacement approach.

Current default:

```python
LOOKBACK_FRAMES = 14
```

At a nominal recording rate of 14 fps, this corresponds to approximately a 1-second displacement interval.

For frame `i` and lookback `k`:

```text
dx = X(i) - X(i-k)
dy = Y(i) - Y(i-k)

displacement = sqrt(dx² + dy²)

elapsed_time = Time(i) - Time(i-k)

speed = displacement / elapsed_time
```

Speed is reported in:

```text
µm/s
```

Actual WormLab timestamps are used for the elapsed-time denominator.

---

# Speed Quality Control

The lookback calculation is deliberately conservative.

For a speed value at frame `i` to be considered valid, the entire interval from frame `i-k` through frame `i` must be valid.

For the current default 14-frame lookback, this means all 15 required frames must satisfy the QC criteria.

A speed value is calculated only when:

1. The endpoint frame exists.
2. The lookback endpoint exists.
3. The endpoint frame numbers differ by exactly `LOOKBACK_FRAMES`.
4. Every integer frame within the lookback interval is present.
5. XY Position is valid for every frame in the interval.
6. Fit is present for every frame in the interval.
7. Fit is greater than or equal to `FIT_THRESHOLD` for every frame in the interval.
8. Actual elapsed WormLab time is positive.

If any condition fails:

```text
Speed = NaN
```

Invalid measurements are never converted to zero.

This preserves the distinction between:

```text
0 speed = valid observation with no translational movement
```

and:

```text
NaN = speed cannot be reliably measured
```

---

## Missing frames and tracking gaps

Missing observations are retained during QC evaluation.

The pipeline does not treat "14 retained rows earlier" as equivalent to "14 video frames earlier."

Speed is therefore never calculated across:

* missing frames
* missing XY observations
* missing Fit observations
* low-Fit frames

Once a new fully valid lookback interval has accumulated, speed calculation resumes.

---

# Current Motility Measurements

## Mean speed

Mean speed is calculated from valid XY-derived lookback speed values.

Invalid speed values (`NaN`) are excluded from the mean.

---

## Body angular rate

Production body movement is calculated from WormLab **Bending Angle ? Mid-Point** using:

```python
BODY_ANGLE_LOOKBACK_FRAMES = 2
```

At the nominal 14 fps rate this is approximately 0.143 seconds. The signed endpoint difference is corrected to the shortest angular displacement in the ?180? range, then its magnitude is divided by actual WormLab elapsed time. Every frame in the 3-frame window must be present, have a midpoint angle, and have Fit greater than or equal to `FIT_THRESHOLD`; invalid intervals remain `NaN`.

The primary track summary is `median_body_angular_rate_deg_s`, representing typical QC-valid body-angle movement. `p90_body_angular_rate_deg_s` is the secondary summary of intermittent upper-end movement intensity. These rate summaries quantify movement whether or not it is periodic and are therefore distinct from thrashing frequency.

---

## Thrashing frequency

Thrashing is estimated from the WormLab:

```text
Bending Angle — Mid-Point
```

signal.

The current implementation estimates periodic body oscillation frequency using autocorrelation of the midpoint bending-angle time series.

The current thrashing method is considered an active area of methodological validation.

Future development will address:

* Fit-aware body-signal QC
* missing-frame handling
* continuous valid segments
* bending amplitude
* oscillation regularity
* distinction between biological inactivity and failed frequency estimation

---

# Removed Legacy Metrics

The following metrics were intentionally removed from the pipeline.

## Fatigue Index

The previous implementation compared early and late swimming speed.

It was removed because a 1-minute swimming assay is not sufficient to interpret this ratio confidently as physiological fatigue.

No replacement fatigue metric is currently used.

---

## Straightness

The previous whole-recording displacement/path-length straightness metric was removed.

Over a 1-minute recording, the measurement can be strongly affected by:

* normal turning
* arena geometry
* boundary interactions
* pauses
* trajectory duration

It is therefore not currently considered a core swimming-motility phenotype.

No replacement persistence or tortuosity metric has yet been introduced.

---

# Data Organization

A typical raw-data directory is organized approximately as:

```text
Data/
└── Raw/
    └── [date]/
        └── Wormlab_processed/
            └── [group]/
                ├── metadata_[group].yaml
                └── [video_id]/
                    ├── BatchExport.csv_Position.csv
                    ├── BatchExport.csv_Fit.csv
                    ├── BatchExport.csv_Bending Angle - Mid-Point.csv
                    └── BatchExport.csv_Length.csv
```

Exact filenames may vary depending on WormLab export settings.

---

# Experimental Hierarchy

The project is intended to preserve the distinction between:

```text
worm
↓
video
↓
experimental replicate
↓
genotype / treatment
```

Individual worms from the same recording should not automatically be assumed to represent independent biological replicates.

Experimental-unit structure will be incorporated explicitly into later statistical development.

---

# Notebook 01 — Data Wrangling

`01_Data_Wrangling.ipynb` currently performs the core data-processing steps.

Major responsibilities include:

* discovering experimental directories
* reading YAML metadata
* importing Position data
* importing Fit data
* separating WormLab tracks
* aligning Fit and Position using actual frame numbers
* calculating QC-aware lookback speed
* calculating retained trajectory metrics
* importing midpoint bending-angle data
* calculating QC-aware 2-frame midpoint body angular rate
* estimating thrashing frequency separately
* exporting processed datasets

---

## Current processed outputs

Saved in:

```text
Data/Processed/
```

### `track_metrics.csv`

Contains per-track summary measurements.

Current retained track metrics include quantities such as:

* mean speed
* path length
* displacement
* number of observations
* duration
* median and P90 body angular rate
* valid body-angular-rate interval count and elapsed-time coverage

Straightness and fatigue index are no longer included.

Some trajectory metrics may be revised as the QC framework is further developed.

---

### `normalized_tracks.csv`

Contains frame-level XY trajectories and calculated speed.

XY coordinates are translated so that each trajectory begins at approximately:

```text
X = 0
Y = 0
```

This output is intended primarily for trajectory visualization and downstream analysis.

Its `Speed` column uses the QC-aware XY-derived lookback calculation.

---

### `fit_data.csv`

Contains frame-level Fit measurements and track metadata.

Fit is retained separately so that tracking quality can be evaluated independently from motility measurements.

---

### `body_angle_data.csv`

Contains frame-level raw midpoint bending angles and the QC-aware 2-frame body angular rate. Missing or QC-invalid angular-rate intervals remain `NaN`.

---

### `thrashing_data.csv`

Contains per-track estimates of midpoint-bending oscillation frequency.

The underlying thrashing methodology remains under active validation.

---

# Notebook 02 — Analysis and Plotting

`02_Analysis_and_Plotting.ipynb` consumes processed data generated by Notebook 01.

Current responsibilities include:

* speed summaries
* Fit-quality summaries
* Fit-versus-speed analyses
* production body-angular-rate summaries and figures
* thrashing analyses kept separate
* group comparisons
* statistical testing
* visualization
* generation of Prism-compatible outputs

Straightness and fatigue analyses have been removed.

---

# Notebook 03 — Archiving

`03_Archiving.ipynb` packages the completed experiment analysis for reproducibility.

The archive may include:

* raw WormLab exports
* processed datasets
* statistical results
* figures
* metadata
* Jupyter notebooks
* archive documentation
* manifest information

Archives are timestamped so that analyses can be reconstructed later.

---

# Installation

## Virtual Environment Setup — Windows PowerShell

Activate the virtual environment:

```powershell
.\venv\Scripts\Activate.ps1
```

For `cmd.exe`:

```cmd
.\venv\Scripts\activate.bat
```

Install required packages:

```powershell
pip install -r requirements.txt
```

---

## Core Python packages

The project currently relies on packages including:

* pandas
* numpy
* matplotlib
* seaborn
* scipy
* pyyaml
* jupyter

See `requirements.txt` for the environment-specific dependency list.

---

# Recommended Workflow

Run the notebooks sequentially.

## 1. Data Wrangling

Open and run:

```text
01_Data_Wrangling.ipynb
```

Confirm that:

* WormLab exports are detected
* tracks are loaded correctly
* Fit and Position align by frame
* processed datasets are produced
* validation checks complete successfully

---

## 2. Analysis and Plotting

Run:

```text
02_Analysis_and_Plotting.ipynb
```

Review:

* speed distributions
* Fit QC
* thrashing results
* sample sizes
* statistical summaries
* figures

---

## 3. Archiving

After analysis is complete, run:

```text
03_Archiving.ipynb
```

Confirm the archive contains the required raw data, processed data, results, metadata, figures, and notebook versions.

---

# Current Analysis Parameters

Important methodological parameters should be explicit in the notebooks.

Current production parameters:

```python
LOOKBACK_FRAMES = 14
BODY_ANGLE_LOOKBACK_FRAMES = 2
FIT_THRESHOLD = 0.90
```

Changes to analysis parameters should be documented and archived with the corresponding experiment.

---

# Current Development Priorities

The project is under active development.

Current priorities include:

1. Stabilizing and validating trajectory QC
2. Evaluating Length as a secondary QC measurement
3. Continuing validation of production midpoint body angular rate
4. Validating the separate thrashing-frequency algorithm
5. Adding additional translational metrics where biologically justified
6. Adding movement/pause bout analysis
7. Developing body-movement amplitude and regularity metrics
8. Relating body oscillation to translation
9. Formalizing experimental-replicate-level statistical analysis
10. Expanding reproducible reporting and archiving

New measurements should be added incrementally and validated before being treated as standard outputs.

---

# Planned Motility Framework

Future development may include the following biologically interpretable measurements.

## Translation

Potential measurements include:

* mean speed
* median speed
* speed variability
* distance traveled
* net displacement
* movement fraction

---

## Body movement

Potential measurements include:

* thrashing frequency
* bending amplitude
* bending variability
* oscillation regularity

---

## Activity dynamics

Potential measurements include:

* percent time moving
* number of movement bouts
* movement-bout duration
* pause duration
* longest movement bout
* longest pause

These require a validated moving/inactive threshold before implementation.

---

## Propulsive efficiency

Future analyses may relate body oscillation to translational movement.

Potential measurements include:

```text
distance per body oscillation
```

or related translation/body-motion relationships.

These measurements will only be implemented after both translational speed and body-oscillation measurements are independently validated.

---

# Quality-Control Philosophy

QC variables should remain distinct from biological motility measurements.

Current philosophy:

```text
Fit → primary tracking-quality QC
Length → candidate secondary body-fit QC
Position → translational measurement
Midpoint bending → body-motion measurement
```

A QC failure should generally produce an invalid measurement (`NaN`) rather than a biological value of zero.

QC rules should be explicit, parameterized, and reproducible.

---

# Reproducibility

Important goals of the pipeline include:

* keeping raw WormLab exports unchanged
* separating raw measurements from derived measurements
* recording analysis parameters
* retaining experiment metadata
* preserving notebook versions
* preserving processed datasets
* preserving statistical outputs and figures
* generating reproducible experiment archives

---

# Troubleshooting

## Position data do not load

Check:

* directory structure
* WormLab export filenames
* CSV header structure
* expected Position feature export

---

## Speed contains many NaN values

Possible reasons include:

* first `LOOKBACK_FRAMES` frames of a track
* missing XY measurements
* missing frames
* missing Fit measurements
* Fit below `FIT_THRESHOLD`
* insufficient continuous valid frames after a tracking interruption

This behavior is intentional when the lookback interval does not meet QC requirements.

---

## Thrashing data are missing or unstable

Check:

* whether the Mid-Point Bending Angle export exists
* whether the relevant track contains sufficient valid bending data
* whether the signal contains missing observations
* whether the estimated oscillation falls within the supported analysis range

Thrashing methodology is still being actively validated.

---

# Citation

If this pipeline is used in a publication or shared analysis, document:

* WormLab software and version
* MBF Bioscience
* analysis notebook version
* analysis parameters
* Fit threshold
* lookback interval
* experiment metadata

The project-specific citation format can be finalized when the software reaches a stable release.

---

# License

This pipeline is currently intended for research use.

Add the appropriate software license before public distribution.

---

# Project Status

**Active development**

The current validated core is centered on:

```text
Position
    ↓
Fit-aware continuous lookback interval
    ↓
XY-derived speed
```

alongside midpoint bending-angle analysis for swimming body motion.

Additional motility measurements will be added only after their definitions, assumptions, artifacts, and validation criteria have been explicitly established.

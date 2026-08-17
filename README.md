# Motility Analysis

## Purpose

This project quantifies *C. elegans* swimming and body-wave behavior from WormLab exports. The three-notebook pipeline preserves transparent biological measurements, eligibility states, acquisition-cluster context, and worm-level traceability.

The current dataset represents one experimental run. Worm-level comparisons describe this experiment; independent experimental repeats remain necessary for generalized biological inference.

## Current workflow

1. WormLab CSV exports and YAML metadata are placed under `Data/Raw/`.
2. `01_Data_Wrangling.ipynb` aligns the six export sources, audits sampling, applies Fit-aware QC, and writes processed tables.
3. `02_Analysis_and_Plotting.ipynb` produces phenotype summaries, traveling-wave validation, PCA, robustness analyses, Prism exports, and prospective sample-size planning.
4. `03_Archiving.ipynb` performs the established archival workflow.

See [WORKFLOW.md](WORKFLOW.md) for operating instructions, [OUTPUT_GUIDE.md](OUTPUT_GUIDE.md) for file families, and [QUICK_START.md](QUICK_START.md) for the short run sequence.

## Required WormLab exports

Each acquisition requires aligned exports for Position, Fit, Length, Bending Angle — Mid-Point, Bending Angles — Multiple, and Center Points. Current spatial sampling expects 11 Multiple Angles and 17 Center Points. Center Points are the preferred geometric source for traveling-wave phenotypes; Multiple Angles remain a validation precursor and agreement reference.

## Seven primary phenotypes

| Phenotype | Biological interpretation |
|---|---|
| Mean swimming speed | Translation through space |
| Median body angular rate | Typical posture-change intensity |
| Center-Point wave frequency | Oscillatory timing |
| Center-Point wavelength | Traveling-wave geometry |
| Phase-linearity R² | Organization of spatial phase progression |
| Center-Point wave angular amplitude | Bend magnitude within coherent waves |
| Conditional coherent-wave duty | Persistence of coherent-wave behavior when sufficient continuous data exist |

Movement performance/timing comprises speed, body angular rate, and wave frequency. Wave structure/organization comprises wavelength, phase-linearity R², and amplitude. Conditional duty represents behavioral persistence. These phenotypes are biologically distinct but not statistically independent: speed and body angular rate show the strongest redundancy, wave frequency also contributes to the shared timing axis, and amplitude and conditional duty add especially distinct information.

## Measurement essentials

Mean speed uses Euclidean endpoint displacement across the validated 14-frame lookback divided by actual elapsed WormLab time; it is not accumulated path length.

Body angular rate uses midpoint bending angle, a 2-frame lookback (about 0.14 s at 14 fps), actual timestamps, and wrapping across ±180°. The median describes typical intensity and P90 the upper movement-intensity tail.

Canonical thrashing uses Fit-valid midpoint-angle data in unbridged continuous segments of at least 5 s. An autocorrelation-based repeating rhythm must fall within 0.5–3.0 Hz. It does not use an explicit literal “more than one counted cycle” rule, and isolated or nonperiodic movement receives no frequency estimate.

Center-Point waves use 17 points, 15 interior signed local turning angles computed with `atan2(cross, dot)`, cumulative centerline arc length, normalized body position, and actual WormLab time. No equal-point-spacing or anatomical head/tail assumption is required. Reversing stored point order changes signed phase direction and mirrors the visualization, while preserving frequency, absolute phase slope, wavelength magnitude, wave-speed magnitude, phase-linearity R², and amplitude. Local turning angle is not labeled curvature.

## Experimental design

```text
experimental run
→ genotype / treatment condition
→ video acquisition cluster
→ tracked worm
```

Worms are organism-level observations. Worms within a video share an acquisition cluster; videos are not biological replicates. Current recordings are approximately 60 s at about 14 fps (roughly 840 frames). Duration sensitivity supports retaining 60 s for the complete phenotype framework, particularly wave eligibility and conditional duty; this recommendation is assay-specific.

## Eligibility

- Broad eligibility: mean speed and median body angular rate.
- Wave-conditional: frequency, wavelength, phase-linearity R², and amplitude.
- Eligibility-aware persistence: conditional coherent-wave duty.

Three states must remain distinct:

1. A valid coherent wave is detected: wave properties are measurable.
2. Enough continuous qualifying data exist but no coherent wave is detected: wave properties are `NaN`; conditional duty may legitimately be 0.
3. Continuous qualifying data are insufficient: wave properties and conditional duty are `NaN`.

Missing does not mean zero.

## PCA

The primary PCA is unsupervised, worm-level, complete-case, and fit to seven standardized variables without genotype. It currently includes 202 of 281 worms, mainly limited by coherent-wave eligibility. Consequently, it summarizes the wave-eligible subset rather than all tracked worms without bias.

PCA summarizes covariance; it is not an “overall motility score.” Genotype geometry is highly stable when speed or body angular rate is removed, when frequency plus wavelength are replaced by wave speed, when U/c is added, and under leave-one-video-out perturbation.

## Sample-size planning

All power analyses are prospective planning estimates, not post-hoc observed power. Minimum detectable difference (MDE) is reported in biological units as well as standardized effect size. Intraclass correlation coefficient (ICC) scenarios approximate within-video clustering: 0, 0.05 as the primary conservative assumption, and 0.10 as a stress test.

Planning includes a pooled-SD broad conservative benchmark, a WT-like optimistic assay-sensitivity scenario, a typical-mutant heteroscedastic scenario preferred for routine N2-versus-mutant planning, and a high-mutant-variability conservative scenario.

About 30 worms/group primarily detects large effects. Roughly 40–50 raw worms/genotype/run is a reasonable routine target for many current phenotypes, but it is not universal: exact requirements depend on endpoint, raw-unit difference, eligibility, mutant variability, clustering, and desired power. Subtle effects can require far larger samples. Independent runs matter more for generalizability than indefinitely increasing worms in one run.

## Outputs

- `Data/Processed/`: Notebook 1 processed inputs and ingestion/QC audits.
- `results/qc/`: diagnostic, validation, PCA, duration, clustering, and sample-size outputs.
- `results/prism/`: current seven-phenotype Prism-ready tables and traceability master.

See [OUTPUT_GUIDE.md](OUTPUT_GUIDE.md) for current, diagnostic, presentation, and retained legacy output families.

## Environment

The current development environment is a Conda environment named `wormlab`:

```bash
conda activate wormlab
python -m pip install -r requirements.txt
```

The environment name and interpreter path are local examples, not requirements. The repository may reside on a local, mapped, or network drive. Plotly is optional: if installed, Notebook 2 writes an interactive 3D PCA HTML file; static PCA outputs do not require it.

## Reproducibility caveat

All current variance, eligibility, clustering, duration, PCA, and power estimates derive from one experimental run. Future independent runs may differ, and stronger biological conclusions require independent replication.

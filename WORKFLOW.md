# Motility Analysis Workflow

This is the detailed operating procedure. For a short checklist, use [QUICK_START.md](QUICK_START.md); for output interpretation, use [OUTPUT_GUIDE.md](OUTPUT_GUIDE.md).

## 1. Prepare WormLab exports

Use repository-relative organization such as:

```text
Data/Raw/<date>/Wormlab_processed/<group>/
├── metadata_<group>.yaml
└── <video_id>/
    ├── ...Position.csv
    ├── ...Fit.csv
    ├── ...Length.csv
    ├── ...Bending Angle - Mid-Point.csv
    ├── ...Bending Angles - Multiple.csv
    └── ...Center Points.csv
```

The repository may live on a mapped or network drive. Required sources are Position, Fit, Length, midpoint Bending Angle, Multiple Bending Angles, and Center Points. Current sampling expects 11 Multiple Angles and 17 Center Points. Preserve YAML genotype/treatment metadata and video identifiers.

## 2. Activate the Python environment

```bash
conda activate wormlab
which python
```

The current development interpreter is `/opt/anaconda3/envs/wormlab/bin/python`. Other machines may use different paths or environment names. A cross-platform alternative is to create an environment and install `requirements.txt`.

## 3. Run Notebook 1 — data wrangling

Open `01_Data_Wrangling.ipynb`, save it, and run top-to-bottom after raw-data changes. It ingests and aligns six source families; audits duplicates, alignment, and sampling density; preserves missing observations; applies Fit-aware interval QC; calculates production speed, body angular rate, and canonical thrashing; and writes `Data/Processed/`.

Center Points and Multiple Angles are retained separately. Center Points are the preferred geometric wave source downstream; Multiple Angles provide validation.

## 4. Run Notebook 2 — analysis and plotting

`02_Analysis_and_Plotting.ipynb` contains:

- production mean speed and body angular movement;
- canonical segment-aware thrashing and eligibility;
- Fit and experimental-unit QC;
- Multiple-Angle validation history;
- Center-Point traveling-wave frequency, wavelength, phase organization, speed, amplitude, and duty;
- Center-Point/Multiple-Angle agreement and kymograph validation;
- translation-to-wave-speed ratio diagnostics;
- seven-phenotype eligibility and redundancy audits;
- complete-case PCA and representation sensitivities;
- video-cluster and leave-one-video-out robustness;
- recording-duration sensitivity;
- Prism-ready exports and presentation figures;
- standardized, raw-unit, and control-anchored prospective sample-size planning.

The notebook does not convert missing wave estimates to zero and does not treat PCA as a composite motility score.

## 5. Review QC

Review Fit coverage, source alignment, broad versus wave-conditional eligibility, continuous-segment and wave validity, Center-Point validation, thrashing coverage, PCA complete-case retention, video clustering, leave-one-video-out stability, duration sensitivity, and sample-size assumptions.

The hierarchy is experimental run → genotype/treatment → video acquisition cluster → worm. Worms are organism-level observations; videos are not biological replicates.

## 6. Prism exports

Current wide tables and the traceability master are in `results/prism/`. Each wide file has one genotype per column, retains individual worms, and preserves ineligible values as blank/`NaN`. Older GraphPad-oriented directories remain for compatibility.

## 7. Archive

After reviewing outputs, run `03_Archiving.ipynb` and confirm the archive manifest and contents.

## 8. Git workflow

```bash
git status
git add <specific intentional files>
git commit -m "descriptive message"
git push
```

Avoid `git add .` unless every modified and untracked file has been reviewed. Do not force-add raw data or large result trees casually.

## 9. Full-rerun strategy

Notebook 2 is computationally expensive. On the current development machine a full execution takes roughly 15–20 minutes, varying by storage, machine, and environment.

- Use targeted cells during development.
- Do not repeatedly rerun frame-level wave, duration, PCA, or simulation blocks for Markdown-only work.
- Before finalizing a scientific or structural change, save, reopen, syntax-check, run top-to-bottom once, and inspect stored errors.
- The control-anchored planning block is lightweight relative to the complete notebook.

## 10. Interpretation reminders

- Sixty seconds remains recommended for this complete assay framework.
- Missing wave estimates are not zeros.
- Current p-values describe worms within one run.
- Independent experimental repeats remain necessary.
- A universal worm-count target is imperfect; use endpoint-specific raw-unit planning.

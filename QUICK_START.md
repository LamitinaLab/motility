# Motility Analysis — Quick Start

## 1. Activate an environment

```bash
conda activate wormlab
which python
```

The development interpreter resembles `/opt/anaconda3/envs/wormlab/bin/python`; another machine may use a different path or environment name. Install dependencies with `python -m pip install -r requirements.txt` if needed.

## 2. Confirm raw WormLab exports

Under `Data/Raw/`, confirm Position, Fit, Length, Bending Angle — Mid-Point, Bending Angles — Multiple, and Center Points. Current sampling expects 11 Multiple Angles and 17 Center Points.

## 3. Run Notebook 1

Run `01_Data_Wrangling.ipynb` top-to-bottom after raw-data changes. It aligns sources, preserves missing data, audits sampling, calculates production metrics, and writes `Data/Processed/`.

## 4. Run Notebook 2

Run `02_Analysis_and_Plotting.ipynb` for phenotypes, wave validation, PCA, QC/robustness, Prism exports, duration validation, and prospective sample-size planning.

Notebook 2 is expensive. Use targeted cells during development. Before finalizing, save, reopen, run top-to-bottom once, and confirm no stored errors. Current full execution is approximately 15–20 minutes on the development machine; runtime varies.

## 5. Review outputs

- Core processed data: `Data/Processed/`
- QC/diagnostics: `results/qc/`
- Prism-ready data: `results/prism/`

See [OUTPUT_GUIDE.md](OUTPUT_GUIDE.md).

## 6. Run Notebook 3

Run `03_Archiving.ipynb` and review the archive manifest.

## 7. Git checkpoint

```bash
git status
git add <specific intentional files>
git commit -m "descriptive message"
git push
```

Do not use `git add .` unless every change has been reviewed.

## 8. Interpretation reminders

- Missing wave estimates are `NaN`, not zero.
- Videos are acquisition clusters, not biological replicates.
- Primary PCA describes the wave-eligible complete-case subset.
- The current dataset is one experimental run.
- Independent experimental repeats remain necessary.

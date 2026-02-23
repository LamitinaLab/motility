# Quick Start Guide - C. elegans Motility Analysis

## Prerequisites
1. Activate virtual environment:
   ```powershell
   .\venv\Scripts\Activate.ps1
   ```

2. Ensure your data is organized:
   ```
   Data/Raw/[date]/Wormlab_processed/[genotype]/[video]/
   ```

## Workflow

### Step 1: Data Wrangling (Notebook 01)
**Goal:** Process raw WormLab data into analyzable metrics

**What it does:**
- Loads Position.csv files (X,Y coordinates)
- Loads CurvatureMap files (body curvature)
- Reads metadata from YAML files
- Calculates speeds, straightness, fatigue
- Extracts thrashing frequency
- Normalizes all tracks to origin (0,0)

**Run:** Execute all cells in `01_Data_Wrangling.ipynb`

**Output:** 3 CSV files in `Data/Processed/`:
- `track_metrics.csv` - Per-track summary metrics
- `thrashing_data.csv` - Thrashing frequencies
- `normalized_tracks.csv` - XY coordinates

**Time:** ~1-5 minutes depending on data size

---

### Step 2: Analysis & Plotting (Notebook 02)
**Goal:** Statistical analysis and visualization

**What it does:**
- Calculates group statistics (mean ± SEM)
- Performs ANOVA and pairwise t-tests
- Creates publication-quality figures:
  - Speed comparisons
  - XY trajectory plots
  - Straightness index
  - Fatigue index
  - Thrashing frequency
  - Multi-panel summary

**Run:** Execute all cells in `02_Analysis_and_Plotting.ipynb`

**Output:** Multiple files in `results/`:
- Summary CSV files
- Statistical test results
- Figures (PNG + PDF) in `results/figures/`

**Time:** ~2-5 minutes

---

### Step 3: Archiving (Notebook 03)
**Goal:** Package and document analysis results

**What it does:**
- Creates timestamped archive directory
- Copies all processed data, results, figures
- Generates comprehensive summary report
- Creates README and manifest
- Optional: ZIP compression

**Run:** Execute all cells in `03_Archiving.ipynb`

**Output:** Archive in `Data/Archive/archive_[timestamp]/`
- Complete copy of all results
- Summary report
- Documentation
- Optional ZIP file

**Time:** ~1-2 minutes

---

## Key Metrics Explained

### Speed (μm/s)
Average velocity of worm movement. Higher = faster locomotion.

### Straightness Index (0-1)
Ratio of displacement to path length. 
- 1 = perfectly straight path
- <1 = curved/wandering path
- Lower values indicate more exploratory behavior

### Fatigue Index
Ratio of late-phase speed to early-phase speed.
- 1 = no change
- <1 = slowing down (fatigue)
- >1 = speeding up

### Thrashing Frequency (Hz)
Oscillations per second from body curvature.
Higher frequency = more vigorous thrashing.

---

## Common Questions

**Q: What if my Position files are named differently?**
A: The pipeline handles Position.csv, Position-1.csv, Position-2.csv, etc.

**Q: Can I analyze only some genotypes?**
A: Yes, the pipeline auto-detects all genotypes in your data structure.

**Q: How do I add more metrics?**
A: Edit the helper functions in Notebook 01, particularly `calculate_path_metrics()`.

**Q: Can I change the statistical tests?**
A: Yes, in Notebook 02, modify `perform_pairwise_tests()` to use Mann-Whitney U instead of t-test.

**Q: What if I don't have curvature data?**
A: The pipeline will skip thrashing analysis but still process all other metrics.

---

## File Checklist

Before running, ensure you have:
- [ ] Position files in each video folder
- [ ] Metadata YAML files for each genotype
- [ ] Virtual environment activated
- [ ] All packages installed (`pip install -r requirements.txt`)

After running, you should have:
- [ ] Processed data in `Data/Processed/`
- [ ] Results and figures in `results/`
- [ ] Archive in `Data/Archive/`

---

## Troubleshooting

**Error: "No Position file found"**
→ Check your data directory structure

**Error: "KeyError: 'genotype'"**
→ Ensure YAML metadata files exist

**Empty plots**
→ Verify CSV files loaded correctly (check first few rows)

**Statistical tests fail**
→ Need at least 2 groups with data

---

## Next Steps

After completing the analysis:

1. Review figures in `results/figures/`
2. Check statistical results in `results/`
3. Read the summary report in the archive
4. Use archived ZIP for sharing/backup

---

**Need help?** Check the full README.md for detailed documentation.

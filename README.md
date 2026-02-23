# C. elegans Motility Analysis Pipeline

A comprehensive Jupyter notebook-based pipeline for analyzing C. elegans motility data from WormLab tracking software.

## Overview

This pipeline processes WormLab tracking data to extract quantitative motility metrics, perform statistical comparisons, and generate publication-quality figures. The analysis is organized into three sequential notebooks:

1. **01_Data_Wrangling.ipynb** - Data loading, processing, and metric calculation
2. **02_Analysis_and_Plotting.ipynb** - Statistical analysis and visualization
3. **03_Archiving.ipynb** - Data archiving and documentation

## Features

### Metrics Calculated
- **Mean Speed** - Average movement speed (μm/s)
- **Straightness Index** - Ratio of displacement to path length (0-1)
- **Fatigue Index** - Ratio of late-phase to early-phase speed
- **Path Length** - Total distance traveled
- **Displacement** - Straight-line distance from start to end
- **Thrashing Frequency** - Body oscillation frequency from curvature data (Hz)

### Visualizations
- Speed comparison bar plots with individual data points
- XY track trajectories (all tracks normalized to origin)
- Straightness index comparisons
- Fatigue index comparisons
- Thrashing frequency comparisons
- Multi-panel summary figures

### Statistical Analysis
- Group summaries (mean ± SEM)
- One-way ANOVA for overall group differences
- Pairwise t-tests between genotypes
- Automated statistical reporting

## Installation

### Virtual Environment Setup (Windows PowerShell)

Activate the virtual environment:

```powershell
.\venv\Scripts\Activate.ps1
```

Or with cmd.exe:

```cmd
.\venv\Scripts\activate.bat
```

Install required packages:

```powershell
pip install -r requirements.txt
```

### Required Packages
- pandas
- numpy
- matplotlib
- seaborn
- scipy
- pyyaml
- jupyter

## Data Structure

Your data should be organized as follows:

```
Data/
└── Raw/
    └── [date]/
        └── Wormlab_processed/
            └── [genotype]/
                ├── metadata_[genotype].yaml
                └── [video_id]/
                    ├── Position.csv (or Position-1.csv)
                    ├── Fit.csv
                    └── CurvatureMap_Track_*.csv
```

### File Formats
- **Position.csv**: Track X,Y coordinates over time
  - Row 1: Skip (track labels)
  - Row 2: Headers (Frame, Time, X/Y pairs for each track)
  - Data: Frame number, time (s), X/Y positions (μm)

- **CurvatureMap_Track_*.csv**: Body curvature along worm length
  - Row 1: Skip
  - Row 2: Headers (Frame, Time, curvature values)
  - Data: Curvature values along body at each timepoint

- **metadata_[genotype].yaml**: Experimental metadata
  - Strain, genotype, treatment conditions
  - Imaging parameters, frame rate
  - Experimenter information

## Usage

### Step 1: Data Wrangling
Run `01_Data_Wrangling.ipynb`:
- Loads all Position and CurvatureMap files
- Extracts individual track data
- Calculates movement metrics
- Normalizes tracks to origin (0,0)
- Saves processed data to `Data/Processed/`

**Outputs:**
- `track_metrics.csv` - Per-track metrics
- `thrashing_data.csv` - Thrashing frequencies
- `normalized_tracks.csv` - XY coordinates

### Step 2: Analysis and Plotting
Run `02_Analysis_and_Plotting.ipynb`:
- Performs statistical analysis
- Generates comparison plots
- Creates publication-quality figures
- Saves results to `results/`

**Outputs:**
- `*_group_summary.csv` - Summary statistics
- `*_pairwise_tests.csv` - Statistical test results
- `stats_summary.json` - Comprehensive statistics
- Figures (PNG and PDF) in `results/figures/`

### Step 3: Archiving
Run `03_Archiving.ipynb`:
- Creates timestamped archive
- Copies all processed data and results
- Generates summary reports
- Creates ZIP archive (optional)

**Outputs:**
- Archive directory: `Data/Archive/archive_[timestamp]/`
- Summary report and documentation
- Compressed ZIP file

## Output Files

### Processed Data
Located in `Data/Processed/`:
- `track_metrics.csv` - All track-level metrics
- `thrashing_data.csv` - Thrashing frequency data
- `normalized_tracks.csv` - Normalized XY coordinates

### Results
Located in `results/`:
- `speed_group_summary.csv`
- `straightness_group_summary.csv`
- `fatigue_group_summary.csv`
- `thrashing_group_summary.csv`
- `*_pairwise_tests.csv` - Statistical comparisons
- `stats_summary.json` - Complete statistical summary

### Figures
Located in `results/figures/` (PNG and PDF formats):
- `mean_speed_comparison.*`
- `straightness_comparison.*`
- `fatigue_comparison.*`
- `thrashing_frequency_comparison.*`
- `track_trajectories_all.*`
- `track_trajectories_by_genotype.*`
- `summary_figure.*` - Multi-panel summary

## Customization

### Adjusting Metrics
Edit functions in Notebook 01:
- `calculate_fatigue_index()` - Change window size for fatigue calculation
- `calculate_thrashing_frequency()` - Adjust peak detection parameters
- `calculate_path_metrics()` - Add custom metrics

### Plotting Style
Edit Notebook 02:
- Color schemes: Modify `sns.color_palette()`
- Figure sizes: Adjust `figsize` parameters
- Statistical tests: Switch between t-test and Mann-Whitney U

### Archive Settings
Edit Notebook 03:
- Enable/disable ZIP compression
- Configure automatic cleanup of old archives
- Customize report templates

## Tips

1. **Large Datasets**: Process data in batches if you have many videos
2. **Quality Control**: Check track quality by visualizing individual trajectories
3. **Statistical Power**: Ensure adequate sample sizes (n ≥ 5 per group recommended)
4. **Metadata**: Complete YAML metadata files for better documentation

## Troubleshooting

**Error loading Position files**
- Check file naming (Position.csv vs Position-1.csv)
- Verify CSV format (skip row 1, headers in row 2)

**Missing thrashing data**
- Ensure CurvatureMap files exist
- Check file naming: `CurvatureMap_Track_*.csv`

**Statistical tests fail**
- Verify sufficient sample size per group
- Check for NaN values in data

## Citation

If using this pipeline, please cite:
- WormLab software: MBF Bioscience (https://www.mbfbioscience.com/wormlab)
- This pipeline: Custom C. elegans motility analysis notebooks

## License

This pipeline is provided as-is for research use.

## Contact

For questions or issues, refer to the experimental metadata or contact the lab personnel listed in the YAML files


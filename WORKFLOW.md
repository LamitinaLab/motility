# Analysis Pipeline Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         C. ELEGANS MOTILITY ANALYSIS                        │
│                              WORKFLOW DIAGRAM                                │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────┐
                              │  RAW DATA    │
                              │              │
                              │ WormLab CSV  │
                              │ + Metadata   │
                              └──────┬───────┘
                                     │
                ┌────────────────────┴────────────────────┐
                │                                         │
                ▼                                         ▼
       ┌─────────────────┐                    ┌─────────────────┐
       │ Position.csv    │                    │CurvatureMap.csv │
       │                 │                    │                 │
       │ • Frame         │                    │ • Frame         │
       │ • Time          │                    │ • Time          │
       │ • X,Y coords    │                    │ • Curvature     │
       └────────┬────────┘                    └────────┬────────┘
                │                                      │
                └──────────────────┬───────────────────┘
                                   │
                                   ▼
        ╔══════════════════════════════════════════════════════╗
        ║  NOTEBOOK 01: DATA WRANGLING                         ║
        ╠══════════════════════════════════════════════════════╣
        ║                                                      ║
        ║  1. Load all Position files                          ║
        ║  2. Extract individual tracks                        ║
        ║  3. Calculate instantaneous speeds                   ║
        ║  4. Normalize tracks to origin (0,0)                 ║
        ║  5. Calculate path metrics:                          ║
        ║     • Path length                                    ║
        ║     • Displacement                                   ║
        ║     • Straightness index                             ║
        ║     • Mean speed                                     ║
        ║     • Fatigue index                                  ║
        ║  6. Load CurvatureMap files                          ║
        ║  7. Calculate thrashing frequency                    ║
        ║  8. Save processed data                              ║
        ║                                                      ║
        ╚══════════════════════╦═══════════════════════════════╝
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
       ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
       │track_       │ │thrashing_   │ │normalized_  │
       │metrics.csv  │ │data.csv     │ │tracks.csv   │
       └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
              │               │               │
              └───────────────┴───────────────┘
                              │
                              ▼
        ╔══════════════════════════════════════════════════════╗
        ║  NOTEBOOK 02: ANALYSIS & PLOTTING                    ║
        ╠══════════════════════════════════════════════════════╣
        ║                                                      ║
        ║  1. Load processed data                              ║
        ║  2. Calculate group statistics:                      ║
        ║     • Mean ± SEM for each genotype                   ║
        ║  3. Statistical tests:                               ║
        ║     • One-way ANOVA                                  ║
        ║     • Pairwise t-tests                               ║
        ║  4. Generate plots:                                  ║
        ║     • Speed comparison (bar + scatter)               ║
        ║     • XY trajectory plots                            ║
        ║     • Straightness index                             ║
        ║     • Fatigue index                                  ║
        ║     • Thrashing frequency                            ║
        ║     • Multi-panel summary figure                     ║
        ║  5. Save results and figures                         ║
        ║                                                      ║
        ╚══════════════════════╦═══════════════════════════════╝
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
       ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
       │ Statistics  │ │  Figures    │ │   JSON      │
       │   (CSV)     │ │ (PNG + PDF) │ │  Summary    │
       └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
              │               │               │
              └───────────────┴───────────────┘
                              │
                              ▼
        ╔══════════════════════════════════════════════════════╗
        ║  NOTEBOOK 03: ARCHIVING                              ║
        ╠══════════════════════════════════════════════════════╣
        ║                                                      ║
        ║  1. Create timestamped archive directory             ║
        ║  2. Copy processed data                              ║
        ║  3. Copy results & figures                           ║
        ║  4. Copy metadata files                              ║
        ║  5. Generate summary report                          ║
        ║  6. Create README & manifest                         ║
        ║  7. Optional: Create ZIP archive                     ║
        ║                                                      ║
        ╚══════════════════════╦═══════════════════════════════╝
                               │
                               ▼
                  ┌─────────────────────────┐
                  │  ARCHIVE DIRECTORY      │
                  │  archive_[timestamp]/   │
                  ├─────────────────────────┤
                  │ • processed_data/       │
                  │ • results/              │
                  │ • figures/              │
                  │ • metadata/             │
                  │ • README.md             │
                  │ • summary_report.txt    │
                  │ • manifest.json         │
                  │ • [optional] .zip       │
                  └─────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           KEY METRICS CALCULATED                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SPEED (μm/s)                                                               │
│  └─ Instantaneous velocity from consecutive XY positions                   │
│                                                                             │
│  STRAIGHTNESS INDEX (0-1)                                                   │
│  └─ displacement / path_length                                             │
│     Higher = straighter path                                               │
│                                                                             │
│  FATIGUE INDEX                                                              │
│  └─ speed_late / speed_early                                               │
│     < 1 = slowing down (fatigue)                                           │
│     > 1 = speeding up                                                      │
│                                                                             │
│  THRASHING FREQUENCY (Hz)                                                   │
│  └─ Oscillations per second from body curvature                            │
│     Detected by peak finding in midbody curvature                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATA FLOW SUMMARY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT:    Raw WormLab tracking data (CSV + YAML)                           │
│            └─ Multiple genotypes, multiple videos per genotype             │
│                                                                             │
│  PROCESS:  Extract tracks → Calculate metrics → Normalize coordinates      │
│                                                                             │
│  ANALYZE:  Group statistics → Statistical tests → Visualizations            │
│                                                                             │
│  ARCHIVE:  Package results → Document → Long-term storage                  │
│                                                                             │
│  OUTPUT:   Publication-ready figures + statistical results                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                        ESTIMATED RUNTIME                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Notebook 01:  1-5 minutes  (depends on number of tracks)                  │
│  Notebook 02:  2-5 minutes  (statistical analysis + plotting)              │
│  Notebook 03:  1-2 minutes  (archiving and documentation)                  │
│                                                                             │
│  TOTAL:        ~5-15 minutes for complete analysis pipeline                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

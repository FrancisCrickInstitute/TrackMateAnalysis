# TrackMateAnalysis

[![Python 3.14](https://img.shields.io/badge/python-3.14-blue.svg)](https://www.python.org/downloads/) [![Built with Pixi](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/prefix-dev/pixi/main/assets/badge/v0.json)](https://pixi.sh) ![Commit activity](https://img.shields.io/github/commit-activity/y/FrancisCrickInstitute/TrackMateAnalysis?style=plastic) ![GitHub](https://img.shields.io/github/license/FrancisCrickInstitute/TrackMateAnalysis?color=green&style=plastic) [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/FrancisCrickInstitute/TrackMateAnalysis/main?urlpath=%2Fdoc%2Ftree%2Ftrackmate_analysis_intro.ipynb) [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/FrancisCrickInstitute/TrackMateAnalysis/blob/main/trackmate_analysis_intro.ipynb)

A gentle, **generic** introduction to analysing the output of a
[TrackMate](https://imagej.net/plugins/trackmate/) tracking run with Python and
[Jupyter](https://jupyter.org/).

TrackMate is a [Fiji](https://imagej.net/software/fiji/)/ImageJ plugin that
detects objects ("spots") in each frame of a time-lapse and links them into
"tracks". This repository teaches you how to take its CSV exports and turn them
into insights: track statistics, trajectories, instantaneous speeds, and
mean-squared displacement (MSD).

The analysis is deliberately **not tied to any particular experiment**; it only
relies on the standard columns TrackMate always writes, so it works with any
TrackMate export.

## Repository contents

| File | Purpose |
|---|---|
| `trackmate_analysis_intro.ipynb` | The main tutorial notebook |
| `pixi.toml` / `pixi.lock` | Reproducible environment definition ([pixi](https://pixi.sh/)) |

## Prerequisites

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) (recommended)
  **or** [pixi](https://pixi.sh/latest/#installation) - either manages Python
  and the packages below
- TrackMate CSV exports (see
  [Exporting data from TrackMate](#exporting-data-from-trackmate)). The notebook
  expects TrackMate's CSV format, not arbitrary CSVs.

The notebook needs `pandas`, `numpy`, `matplotlib`, `seaborn` and `jupyter`.
Both setup methods below install these for you, so you do not need to install
them by hand.

## Setup

### With Miniconda (recommended)

Create and activate an environment, then launch Jupyter Lab:

```sh
conda create -n trackmate-analysis -c conda-forge python=3.14 pandas numpy matplotlib seaborn jupyterlab
conda activate trackmate-analysis
jupyter lab
```

Then open `trackmate_analysis_intro.ipynb` and select the kernel for your
`trackmate-analysis` environment.

### With pixi

From the repository root:

```sh
pixi install      # create the environment and install all dependencies
pixi run jupyter  # start Jupyter Lab
```

Then open `trackmate_analysis_intro.ipynb` and select the `trackmate-analysis`
kernel.

## Try it with Binder

[Binder](https://mybinder.org/) builds and launches a live Jupyter environment
from this repository in your browser, so you can run the notebook without
installing anything locally.

Click the **Binder** badge at the top of this page to open
`trackmate_analysis_intro.ipynb` directly. Binder creates the environment from
[`requirements.txt`](requirements.txt) (and supplies Jupyter automatically), so
the notebook's `pandas`, `numpy`, `matplotlib` and `seaborn` dependencies are
already present.

Binder is great for quickly trying the tutorial, but for your own data you will
still want a local environment (see [Setup](#setup)) so you can point the
notebook at your own TrackMate exports.

## Try it in Google Colab

[Google Colab](https://colab.research.google.com/) runs the notebook for free in
your browser with no local install. Click the **Open In Colab** badge at the top
of this page.

Colab does not include the example CSVs, so the notebook downloads them
automatically the first time it runs. To use your own data instead, upload your
TrackMate CSVs in Colab's file browser, then set `DATA_DIR` in the **YOUR
SETTINGS** cell to that folder (for example `/content/`).

## Your first analysis (5 minutes)

1. Export your TrackMate tables as CSVs (see
   [Exporting data from TrackMate](#exporting-data-from-trackmate)) and put
   them in a folder.
2. Open `trackmate_analysis_intro.ipynb` and, in the **YOUR SETTINGS** cell near
   the top, set `DATA_DIR` to that folder.
3. Run the notebook top-to-bottom (**Kernel -> Run All**).
4. Scroll to the **Try it yourself** section and change the metric name inside a
   `plot_track_metric(...)` or `scatter_track_metrics(...)` call to explore your
   data.

You only need to edit two things: the **settings** at the top, and the **metric
names** in the `plot_track_metric(...)` / `scatter_track_metrics(...)` calls.
The analysis code itself can stay as it is.

## Exporting data from TrackMate

In Fiji, after running TrackMate, use the analysis panel to export the tables as
CSV:

1. Export the **spots** table, saving it with a name ending in `_spots.csv`.
2. Export the **tracks** table, saving it with a name ending in `_tracks.csv`.

Put both files in a folder (e.g. `TrackMate_Outputs/`) and point the notebook's
`DATA_DIR` variable at it.

## The TrackMate CSV format (read this before you write code)

TrackMate writes CSVs with **four header rows** before the data:

1. feature names (e.g. `TRACK_ID`, `TRACK_MEAN_SPEED`),
2. human-readable names,
3. short names,
4. units.

`pandas.read_csv` cannot handle this automatically. The notebook uses
`skiprows=[1, 2, 3]` to drop the last three header rows and keep row 0 (the
feature names) as column headers. Remember this if you write your own loading
code.

Two tables are exported:

- **`*_tracks.csv`**: one row per track, with summary measurements such as
  `TRACK_DURATION`, `TRACK_DISPLACEMENT`, `TRACK_MEAN_SPEED` and
  `CONFINEMENT_RATIO`.
- **`*_spots.csv`**: one row per detection, with `TRACK_ID`,
  `POSITION_X`/`POSITION_Y`/`POSITION_Z`, `FRAME`, `POSITION_T` (time), plus
  channel intensities and shape features when available.

## Notebook overview

The notebook walks through, in order:

1. Pointing the notebook at your data and optionally filtering out short or
   low-quality tracks.
2. Loading TrackMate's spots/tracks CSVs and understanding their layout.
3. Setting spatial and temporal calibration (pixel size and frame interval).
4. Plotting any track metric from a friendly menu (duration, mean speed,
   confinement ratio, and more).
5. Reconstructing trajectories from the spots table.
6. Computing instantaneous speed between consecutive frames.
7. Computing mean-squared displacement (MSD) per track.
8. Exploring per-spot shape features such as area and circularity.
9. A "Try it yourself" section with ready-to-edit examples.

Each section builds a reusable function (e.g. `read_trackmate_csv`,
`track_msd`, `plot_track_metric`) that you can copy into your own analysis.

## Where do I change things?

| I want to... | Do this |
|---|---|
| Use my own data | Change `DATA_DIR` in the **YOUR SETTINGS** cell |
| Ignore very short tracks | Set `MIN_TRACK_DURATION` (seconds) in the settings cell |
| Ignore low-quality tracks | Set `MIN_TRACK_QUALITY` in the settings cell |
| Plot a different metric | Change the name in `plot_track_metric("...")` |
| Compare two metrics | Change the names in `scatter_track_metrics("...", "...")` |
| Colour a scatter by another metric | Add `color_metric="..."` to the scatter call |

## Next steps

The notebook's **Try it yourself** section is the best place to start
experimenting. From there, the summary cell points to further ideas: comparing
conditions, filtering tracks by duration/quality, plotting channel intensities
over time, and statistical testing.

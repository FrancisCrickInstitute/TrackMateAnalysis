# TrackMateAnalysis

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

- [pixi](https://pixi.sh/latest/#installation) (manages Python and the
  packages below in one command)
- A TrackMate export (see [Exporting data from TrackMate](#exporting-data-from-trackmate))

The notebook needs `pandas`, `numpy`, `matplotlib`, `seaborn` and `jupyter`;
these are declared in `pixi.toml`, so you do not need to install them by hand.

## Setup

From the repository root:

```sh
pixi install      # create the environment and install all dependencies
pixi run jupyter  # start Jupyter Lab
```

Then open `trackmate_analysis_intro.ipynb` and select the `trackmate-analysis`
kernel.

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

1. Loading TrackMate's spots/tracks CSVs and understanding their layout.
2. Track-level statistics: duration, mean speed and confinement-ratio
   distributions.
3. Reconstructing trajectories from the spots table.
4. Computing instantaneous speed between consecutive frames.
5. Computing mean-squared displacement (MSD) per track.
6. Exploring per-spot shape features such as area and circularity.

Each section builds a reusable function (e.g. `read_trackmate_csv`, `track_msd`)
that you can copy into your own analysis.

## Next steps

The notebook ends with pointers for extending the analysis, including comparing
conditions, filtering tracks by duration/quality, plotting channel intensities
over time, and statistical testing.

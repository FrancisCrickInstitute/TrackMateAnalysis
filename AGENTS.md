# AGENTS.md

Guidance for AI agents (and contributors) working in this repository.

## Project overview

This is an educational repository: a single Jupyter notebook
(`trackmate_analysis_intro.ipynb`) that teaches how to analyse
[TrackMate](https://imagej.net/plugins/trackmate/) tracking exports with Python.
It covers track statistics, trajectories, instantaneous speed, and
mean-squared displacement (MSD).

The analysis is deliberately **generic**: it only relies on the standard
columns TrackMate always writes, so it must work with any TrackMate export, not
a specific experiment.

## Tech stack

- **Python** (managed by [pixi](https://pixi.sh/))
- **pandas**, **numpy**, **matplotlib**, **seaborn**, **jupyter** (declared in
  `pixi.toml` as PyPI dependencies)

## Common commands

From the repository root:

```sh
pixi install      # create the environment and install all dependencies
pixi run jupyter  # start Jupyter Lab
```

Then open `trackmate_analysis_intro.ipynb` and select the `trackmate-analysis`
kernel.

There is no test suite or lint configuration. Verify notebook changes by
running the notebook end-to-end (or the affected cells) under `pixi run jupyter`.

## Repository layout

| Path | Purpose |
|---|---|
| `trackmate_analysis_intro.ipynb` | The main tutorial notebook (the bulk of the content) |
| `pixi.toml` / `pixi.lock` | Reproducible environment definition; `pixi.lock` is generated, do not edit by hand |
| `example_input/` | Example TrackMate CSVs used as notebook input (`*_spots.csv`, `*_tracks.csv`, `*_edges.csv`) |
| `README.md` | Human-facing overview and setup instructions |
| `.gitattributes` | Marks `pixi.lock` as generated/binary to prevent 3-way merges |

## TrackMate CSV format (critical when writing loading code)

TrackMate writes CSVs with **four header rows** before the data:

1. feature names (e.g. `TRACK_ID`, `TRACK_MEAN_SPEED`) — row 0,
2. human-readable names — row 1,
3. short names — row 2,
4. units — row 3.

`pandas.read_csv` cannot parse this directly. Load with:

```python
pd.read_csv(path, skiprows=[1, 2, 3])
```

This keeps row 0 (feature names) as column headers and drops the rest.

Three tables can be exported:

- **`*_tracks.csv`**: one row per track, with summary measurements such as
  `TRACK_DURATION`, `TRACK_DISPLACEMENT`, `TRACK_MEAN_SPEED`, `CONFINEMENT_RATIO`.
- **`*_spots.csv`**: one row per detection, with `TRACK_ID`, `POSITION_X` /
  `POSITION_Y` / `POSITION_Z`, `FRAME`, `POSITION_T` (time), plus channel
  intensities and shape features when available.
- **`*_edges.csv`**: one row per link, with `TRACK_ID`, `SPOT_SOURCE_ID`,
  `SPOT_TARGET_ID`, `SPEED`, `DISPLACEMENT`, `EDGE_TIME`, and directional data.

The notebook discovers inputs by recursively globbing `DATA_DIR` for
`*_tracks.csv` and `*_spots.csv`; it defaults to `./example_input`.

## Conventions and style

- The notebook is pedagogical: each section introduces a **reusable function**
  (e.g. `read_trackmate_csv`, `track_msd`) with explanatory prose, so keep code
  readable and self-contained rather than optimised.
- Keep analysis **generic and experiment-agnostic**; avoid hard-coding dataset
  assumptions (column names, thresholds, or channel counts) beyond what
  TrackMate itself guarantees.
- Preserve the notebook's narrative flow: markdown explanation, then code, then
  (where present) rendered output. Do not strip outputs arbitrarily.
- Keep the Python environment defined in `pixi.toml` in sync with the packages
  the notebook actually imports. Re-generate `pixi.lock` via pixi rather than
  editing it by hand.

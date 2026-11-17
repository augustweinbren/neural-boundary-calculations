Install guide:
1. Open VS Code
2. Open the Terminal within VS Code
3. Type and run within the terminal: `python -m venv .venv`
5. Type and run within the terminal: `source .venv/bin/activate`
6. Type and run within the terminal: `pip install -r requirements.txt` (or `uv pip install -r requirements.txt` if this is too slow)
7. Select Python Interpreter as `.venv` within VS Code
8. Change the data paths within the Settings cell (the first cell with code in it) within the recently added .ipynb file to match where the data is located
9. Look through the remaining settings in the Settings cell and change as you see fit
10. Run all the cells, or run them one at a time within VS Code

<details>
<summary>Claude Opus 5.5 written instructions</summary>
<br>
`dynamic_distance_from_boundary.ipynb` measures how GFP-labelled cells move relative to the top and
bottom tissue edges in timelapse movies, and compares FN (fibronectin) with PBS (control) slices.

## What you need

- **VS Code** with the **Python** and **Jupyter** extensions (install them from the Extensions panel on
  the left of VS Code).
- **Python 3** (tested with Python 3.12).
- This folder, which contains `dynamic_distance_from_boundary.ipynb` and `requirements.txt`.

## One-time setup

You only need to do this once per computer.

1. Open VS Code.
2. Open this folder: **File → Open Folder…**, then choose the folder that contains
   `dynamic_distance_from_boundary.ipynb`. Opening the folder (not just the notebook file) makes the
   terminal start in the right place.
3. Open a terminal inside VS Code: **Terminal → New Terminal**.
4. Create a virtual environment. This is a private copy of Python for this project, stored in a
   folder called `.venv`. Type the line for your computer and press Enter:

   | macOS / Linux | Windows |
   |---|---|
   | `python3 -m venv .venv` | `py -m venv .venv` |

   If a `.venv` folder already exists here, skip this step.
5. Activate it:

   | macOS / Linux | Windows |
   |---|---|
   | `source .venv/bin/activate` | `.venv\Scripts\activate` |

   The terminal prompt now starts with `(.venv)`. If it doesn't, the next step will install the packages
   in the wrong place.
6. Install the packages the notebook needs:

   ```
   pip install -r requirements.txt
   ```

   If this is slow and you have [uv](https://docs.astral.sh/uv/) installed, use
   `uv pip install -r requirements.txt` instead.
7. Open `dynamic_distance_from_boundary.ipynb`. Click **Select Kernel** in the top-right corner of the
   notebook, then **Python Environments**, then the one marked `.venv`. VS Code remembers this choice
   for next time.

## Preparing your data

The notebook finds samples by their file names. For each sample (for example `18951_FN 1`) it expects:

| File | Contents |
|---|---|
| `18951_FN 1_brightfield.tif` | the timelapse movie, with a brightfield channel |
| `18951_FN 1_spots.csv` | TrackMate spots export |
| `18951_FN 1_Coordinates_Top.csv` | hand-drawn top boundary (X, Y columns) |
| `18951_FN 1_Coordinates_Bottom.csv` | hand-drawn bottom boundary (X, Y columns) |

- Put **FN** or **PBS** in the sample name, so the sample appears in the FN vs PBS comparison.
- The files can be in one folder or spread over subfolders (e.g. `Timelapse 11/`, `Timelapse 12/`).
- A sample name must not appear in two folders.
- If a file has a different name, either rename it or add it by hand under `EXTRA_DATASETS` in the
  settings cell.
- Don't use a **macOS Finder alias** to give a file a second name. The notebook can't read through
  aliases. Use a symbolic link instead, made in the terminal, e.g.
  `ln -s "FN 3-MaxIP.tif" "18951_FN 3_brightfield.tif"`.

## Running the analysis

1. Go to the **Settings** cell (the first code cell) and set `DATA_DIR` to the folder that holds your
   data. Examples:

   ```python
   DATA_DIR = Path("/Users/yourname/Desktop/ale-collab")     # macOS / Linux
   DATA_DIR = Path(r"C:\Users\yourname\ale-collab")          # Windows (keep the r before the quotes)
   ```

2. Check the other settings in the same cell. Each one has a comment explaining it. The ones you are
   most likely to change:

   | Setting | What it does |
   |---|---|
   | `ONLY_SAMPLES` | Analyse only some samples, e.g. `["18951_FN 3"]` or `["18951_*"]`. `[]` = all. |
   | `TESTING_MODE` | `True` = only the first 5 frames of each movie. Use it for a quick first check, then set `False` for real results. |
   | `SHOW_ACTIVE_CONTOUR_BUILD_PLOTS` | `True` draws every fitted boundary. Useful for checking; set `False` for full runs, which would otherwise make hundreds of figures. |
   | `OUTPUT_DIR` | Where results and caches are saved. Defaults to `DATA_DIR`. |

3. Click **Run All** at the top of the notebook (or run the cells one at a time with Shift+Enter).
4. Check the table printed by **section 3 (Check input files)** before waiting for the rest. Every
   sample you expect should be listed, with `found` under `image`, `polyline` and `spots`. Samples
   marked `MISSING` or `BAD COLUMNS` are skipped (see Troubleshooting below).

**How long it takes:** the first full run (`TESTING_MODE = False`) is slow, and can take hours for
several samples, because the notebook fits the tissue boundary on every frame of every movie. The
results are saved to `OUTPUT_DIR`, so later runs with the same files and settings load them in
seconds.

## Where the results go

Inside `OUTPUT_DIR`:

- `dynamic_distance_analysis_export/`: all result tables (CSV) and a record of the settings used.
- `dynamic_contour_cache/`: the fitted boundaries, reused by later runs.

To force everything to be recalculated, set `REFRESH_ENDPOINT_CACHE = True` for one run.

## Troubleshooting

| Problem | What to do |
|---|---|
| `ModuleNotFoundError: No module named ...` | The notebook is not using the `.venv` kernel. Repeat setup step 7, or activate `.venv` and repeat step 6. |
| No samples are found | Check `DATA_DIR` and that the file names match the pattern above. The input check lists files it ignored. |
| `MISSING` in the input-check table | That file wasn't found under the expected name. Check its spelling and the `_brightfield` / `_spots` / `_Coordinates_Top` / `_Coordinates_Bottom` ending. |
| `BAD COLUMNS` or `unreadable file` | The CSV isn't the expected export, or the file is a Finder alias rather than the real file. |
| Windows: `activate` is "not digitally signed" or "disabled on this system" | Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once in the terminal, then try step 5 again. |
</details>

# Scorecards

`Run_verobs_all` can generate a WebgraF `Scorecards` entry from Monitor's joint
significance calculation when `SCORECARDS=T` is set in the experiment
environment. Ordinary verification runs skip this optional stage. Scorecards
require `SIGN_TEST_JOINT=T`, `GEN` in the plot selection for each enabled
domain, and the Python renderer dependencies. The stage fails clearly if a run
requests scorecards but required exports are missing.

It creates a `Surface` and an `Upper-Air` page for each comparison against
`CONTROL_EXP_NR` (default `1`). The plotted value is
reference RMSE minus comparison RMSE, after Monitor's normalization, so a
positive value means the comparison has lower RMSE. A black outline marks a
difference significant at the configured confidence level; a gray outline marks
a non-significant difference. The color scale shows the difference as a
percentage of normalized RMSE and is based on significant values only; values
beyond that range saturate at the strongest color. If no values are significant,
all values set the scale. The CSV rows include paired-case counts for each lead.
The footer also counts positive and negative squares, both among significant
values and among all values; exact zeros are excluded from those counts.

## Python setup

Python 3 and Matplotlib are needed only at run time when `SCORECARDS=T`; they
are not needed to compile Monitor or run verification without scorecards. If
you enable scorecards, install the pinned dependency in a virtual environment
and point the run script at that interpreter:

```bash
python3 -m venv .venv-scorecards
source .venv-scorecards/bin/activate
python -m pip install -r src/python/requirements-scorecards.txt
export SCORECARD_PYTHON="$PWD/.venv-scorecards/bin/python"
```

Run Monitor as usual from `scr/`:

```bash
./Run_verobs_all Env_exp_test0
```

When scorecards are enabled, the Fortran code writes versioned `joint_scores`
text exports in each verification working directory. `Run_verobs` copies them
to temporary staging and removes them before sending the Surface/Temp entries
to WebgraF. The Python stage selects the all-station aggregate
(`station_scope=0`), `ALL` selection, and `ALL` initial-time group. It keeps
the selected exports under `WebgraF/<PROJECT>/Scorecards/data/` along with the
rendered PNG and CSV files. For upper air the default selected pressure levels
are 300, 500, 700, and 850 hPa.
Override this list with `SCORECARD_LEVELS`, for example
`SCORECARD_LEVELS="925,850,700,500"`.

Set `CONTROL_EXP_NR` in the experiment environment file to compare every other
experiment against a different reference. Keep `EXP` and `DISPLAY_EXP` in the
same order. If `PERIOD_TYPE` produces multiple periods, WebgraF gets a Period
selector and the files retain their period in their names.

The parser accepts the versioned Monitor format and the supplied legacy
headerless sample. The renderer was written for this repository; no external
renderer source was copied.

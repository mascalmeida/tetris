# Tetris

This repository is dedicated to the publication **[Tetris: An SLA-aware application placement strategy in the Edge–Cloud Continuum](https://www.sciencedirect.com/science/article/pii/S0140366426002689)**, published in *Computer Communications*. The work grew out of **Lucas Mascarenhas Almeida's master's research** in the [Graduate Program in Computer Science (PGCOMP)](https://pgcomp.ufba.br/) at the Federal University of Bahia (UFBA), with the [Ubiquitous Computing Laboratory (UCL)](https://ucl.ufba.br/).

<p align="center">
  <a href="https://pgcomp.ufba.br/"><img src="docs/assets/pgcomp.png" alt="PGCOMP — Federal University of Bahia" width="280" align="middle"></a>
  &nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://ucl.ufba.br/"><img src="docs/assets/ucl.png" alt="UCL — Ubiquitous Computing Laboratory" width="48" height="48" align="middle"></a>
  &nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://www.sciencedirect.com/science/article/pii/S0140366426002689"><img src="docs/assets/computer-communications.gif" alt="Computer Communications journal cover" height="90" align="middle"></a>
</p>

<!-- Official image sources:
PGCOMP: https://pgcomp.ufba.br/sites/pgcomp.ufba.br/files/logomarca_novo_template_-_sites_da_pos-graduacao_0.png
UCL: https://ucl.ufba.br/assets/images/favicon.png
Computer Communications: https://ars.els-cdn.com/content/image/1-s2.0-S0140366426X20126-cov200h.gif
-->

**L. Almeida, B. Ramprasad, E. de Lara, and M. Peixoto.** *Computer Communications*, 259 (2026), 108678. [DOI: 10.1016/j.comcom.2026.108678](https://doi.org/10.1016/j.comcom.2026.108678).

This README explains the proposed algorithm and, above all, provides a **practical tutorial for reproducing the paper's experiments**: setting up Python, running the simulations, analyzing their outputs, and locating the results associated with each figure and table.

## What is Tetris?

Tetris is an SLA-aware application placement strategy for heterogeneous edge and cloud servers. It prioritizes application requests by urgency and packs services onto feasible servers using the geometric mean of the remaining CPU, memory, and storage capacity. Its goal is to reduce resource fragmentation, latency SLA violations, and application drops.

The main benchmark implementation in [`nb_tetris.ipynb`](notebooks/nb_tetris.ipynb) orders applications by their remaining latency SLA margin and considers servers in **ascending** residual-capacity order. The weighted formulation used for the paper's sensitivity study is implemented separately in [`nb_hyper_sensitivity_analysis.ipynb`](notebooks/nb_hyper_sensitivity_analysis.ipynb).

## 1. Explore the repository

```text
tetris/
├── datasets/                         # Five input scenarios in JSON format
├── notebooks/
│   ├── nb_tetris.ipynb                # Tetris simulation
│   ├── nb_thea.ipynb                  # Thea simulation
│   ├── nb_*.ipynb                     # Baselines and additional studies
│   ├── nb_results_*.ipynb             # Statistical analysis and plots
│   ├── results/                      # Simulation CSVs and study summaries
│   ├── logs/                         # Simulator logs in MessagePack format
│   └── *.csv, *.png                   # Exported clustering data and figures
├── docs/assets/                      # Images used in this README
└── requirements.txt                  # Pinned environment for this tutorial
```

The implementations live in the notebooks; there is no separate command-line application to install. Input scenarios and the CSV files required by the main analysis notebooks are already included.

| Scenario ID | Input file in `datasets/` | Purpose |
| --- | --- | --- |
| `sample` | `dataset_sample.json` | Preliminary comparison of six strategies |
| `2ec_low_on` | `dataset_2ec_low_on.json` | Low workload, cloud enabled |
| `2ec_low_off` | `dataset_2ec_low_off.json` | Low workload, cloud disabled |
| `2ec_high_on` | `dataset_2ec_high_on.json` | High workload, cloud enabled |
| `2ec_high_off` | `dataset_2ec_high_off.json` | High workload, cloud disabled; stress scenario |

Choose a route after setup:

- **Analyze the included results:** go directly to [Section 4](#4-reproduce-the-analyses).
- **Rerun the experiments:** complete [Section 3](#3-run-the-simulations), then Section 4. Use [Section 5](#5-reproduce-sensitivity-and-execution-overhead) for the additional sensitivity and timing studies.

## 2. Set up the environment

### Clone `main`

Install Git and Python **3.10** first. The notebooks record Python **3.10.20**, also used to check this tutorial.

```bash
git clone --branch main https://github.com/mascalmeida/tetris.git
cd tetris
```

### Create and activate a virtual environment

**Windows PowerShell:**

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Linux or macOS:**

```bash
python3.10 -m venv .venv
source .venv/bin/activate
```

If PowerShell blocks activation, use the environment's interpreter directly, for example `.\.venv\Scripts\python.exe -m pip install -r requirements.txt`.

### Install packages and the notebook kernel

From the repository root, with the environment active:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip check
python -m ipykernel install --sys-prefix --name tetris --display-name "Python (Tetris)"
```

The paper reports **EdgeSimPy 1.1.0**. The [requirements file](requirements.txt) pins a specific [EdgeSimPy 1.1.0 source revision](https://github.com/EdgeSimPy/EdgeSimPy/tree/76eb5ead74596bb4240759fa4336f1d6f190c70a), along with the analysis packages and Jupyter tools. These are the versions checked for this guide; the repository does not provide a complete historical lockfile for the publication environment.

### Start Jupyter in the correct directory

```bash
cd notebooks
python -c "from pathlib import Path; Path('results').mkdir(exist_ok=True)"
python -m jupyterlab
```

Open a notebook, select **Python (Tetris)**, and use **Kernel → Restart Kernel and Run All Cells** to execute it from a clean state.

**The kernel's working directory must be `notebooks/`.** Simulations load `../datasets/`, while analyses read `results/`. To check this, run:

```python
from pathlib import Path
print(Path.cwd())
assert Path("../datasets/dataset_sample.json").is_file()
assert Path("results").is_dir()
```

## 3. Run the simulations

Run one simulation notebook at a time. Each run rewrites the SLA values in its scenario JSON and overwrites CSVs with the same algorithm, scenario, and seed. Use a separate clone if you want to preserve the included results for comparison.

### Check one small run

In `nb_tetris.ipynb`, find the final cell, starting with `# run the experiment many times`. Temporarily replace its `seed_list` and `scenario_list` assignments with:

```python
seed_list = [428956419]
scenario_list = ['sample']
```

Restart the kernel and run all cells. A successful run produces these files in `notebooks/results/`:

```text
tetris_logsserver_sample_428956419.csv
tetris_logsservice_sample_428956419.csv
tetris_logsuser_sample_428956419.csv
```

This checks the installation only. **Restore the original 30 seeds and five scenarios before running the full experiment.**

### Run the full benchmark

Run every notebook below from top to bottom, keeping its default configuration:

| Order | Notebook | Default scenarios | Runs |
| --- | --- | --- | ---: |
| 1 | [`nb_tetris.ipynb`](notebooks/nb_tetris.ipynb) | `sample` + all four factorial scenarios | 150 |
| 2 | [`nb_thea.ipynb`](notebooks/nb_thea.ipynb) | `sample` + all four factorial scenarios | 150 |
| 3 | [`nb_lix.ipynb`](notebooks/nb_lix.ipynb) | `sample` | 30 |
| 4 | [`nb_edf.ipynb`](notebooks/nb_edf.ipynb) | `sample` | 30 |
| 5 | [`nb_mcf.ipynb`](notebooks/nb_mcf.ipynb) | `sample` | 30 |
| 6 | [`nb_edfmcf.ipynb`](notebooks/nb_edfmcf.ipynb) | `sample` | 30 |
| 7 | [`nb_argos.ipynb`](notebooks/nb_argos.ipynb) | `sample`, `2ec_high_off` | 60 |
| 8 | [`nb_faticanti.ipynb`](notebooks/nb_faticanti.ipynb) | `sample`, `2ec_high_off` | 60 |

These defaults execute **540 simulations** and write **1,620 CSV files**. Argos and Faticanti also run `sample`, although the preliminary six-strategy analysis does not consume those outputs.

The controls are `algo_name` in the first cell and `seed_list` / `scenario_list` in the final execution cell. Preserve algorithm names because analyses use them in filenames. There are no generic `scenario_path` or `output_path` configuration variables.

For the full experiment, retain:

- The original **30 seeds**, their order, and each notebook's scenario order. Random generators are seeded before the scenario loop; removing or reordering scenarios can change the sampled SLA values.
- SLA sampling with mean/standard deviation **40/2.5 ms** for `sample` and **15/5 ms** for the factorial scenarios.
- One-second simulation ticks. `sample` runs for 60 steps; factorial runs stop when all services are provisioned or at 60 steps.

Each run writes:

```text
results/<algorithm>_logsserver_<scenario>_<seed>.csv
results/<algorithm>_logsservice_<scenario>_<seed>.csv
results/<algorithm>_logsuser_<scenario>_<seed>.csv
logs/algorithm=<algorithm>/seed=<seed>/scenario=<scenario>/
```

### Optional: execute from a terminal

From `notebooks/`, with the same environment active:

```bash
python -m jupyter nbconvert --to notebook --execute nb_tetris.ipynb --ExecutePreprocessor.kernel_name=tetris --ExecutePreprocessor.timeout=-1 --output nb_tetris.executed.ipynb
```

Replace the input and output names for each notebook in the tables. Run them sequentially. The separate `.executed.ipynb` retains cell outputs; simulation CSVs still go to `results/`.

## 4. Reproduce the analyses

These notebooks read the included CSVs or the files regenerated in Section 3. Keep their default `seed_list`, `scenario_list`, and `algo_list` consistent with the simulations. Restart the kernel and run all cells in each notebook.

| Order | Analysis notebook | Required simulation outputs | Paper results / output |
| --- | --- | --- | --- |
| 1 | [`nb_results_preliminary_exp.ipynb`](notebooks/nb_results_preliminary_exp.ipynb) | Tetris, Thea, LI-X, EDF, MCF, EDF + MCF on `sample` | Figure 9: preliminary comparison |
| 2 | [`nb_results_all_scenarios_thea_tetris.ipynb`](notebooks/nb_results_all_scenarios_thea_tetris.ipynb) | Tetris and Thea on all four factorial scenarios | Figures 11–14 and 16; Table 8: metrics, confidence intervals, and statistical comparisons |
| 3 | [`nb_results_high_off_argos_faticanti.ipynb`](notebooks/nb_results_high_off_argos_faticanti.ipynb) | Tetris, Thea, Argos, Faticanti on `2ec_high_off` | Figure 15: stress-scenario comparison |
| 4 | [`nb_results_experimental_design.ipynb`](notebooks/nb_results_experimental_design.ipynb) | Eight aggregate rows from analysis 2; see below | Figure 10: factor effects and interactions |
| 5 | [`nb_results_all_scenarios_kmeans.ipynb`](notebooks/nb_results_all_scenarios_kmeans.ipynb) | Tetris and Thea on all four factorial scenarios | Figures 17–19; Tables 9–13: correlations, PCA, clusters, and regressions |

The comparison notebooks have a cell labeled `# RESULTS TO USE ON OVERLEAF PLOTS`. It prints the numerical summaries used to build publication figures. Running these notebooks regenerates statistics; it does not automatically export every figure in the paper's final LaTeX layout. Figures 1, 2, and 4–8 are explanatory illustrations rather than notebook-generated experiment outputs.

### Feed current results into the factorial notebook

`nb_results_experimental_design.ipynb` contains a fixed `all_experiments_input` string. Running it unchanged analyzes those embedded values, even after the simulation CSVs have been regenerated.

In `nb_results_all_scenarios_thea_tetris.ipynb`, insert the following cell **immediately after** `# RESULTS TO USE ON OVERLEAF PLOTS`, before the later analyses overwrite some summary variables. Run the notebook from the beginning to export the nine response metrics in the required order:

```python
from pathlib import Path

metrics = [
    stats_sla, stats_drop, stats_delay, stats_power,
    stats_rii, stats_rfc, stats_rfcpu, stats_rfram, stats_rfi,
]
table = pd.concat(
    [frame.set_index(["algorithm", "scenario"])["mean"] for frame in metrics],
    axis=1,
)
order = [
    (algorithm, scenario)
    for algorithm in ["tetris", "thea"]
    for scenario in ["2ec_low_on", "2ec_low_off", "2ec_high_on", "2ec_high_off"]
]
text = "; ".join(
    ",".join(str(value) for value in row)
    for row in table.loc[order].to_numpy()
)
Path("results/factorial_input.txt").write_text(text, encoding="utf-8")
```

In the factorial notebook, add `from pathlib import Path` to the cell starting with `import pandas as pd`. Inside its `while` loop, replace the fixed assignment with:

```python
all_experiments_input = Path("results/factorial_input.txt").read_text(encoding="utf-8")
```

Restart the kernel and run all cells, keeping `k = 3` and `v = 9`. The row order above matches the notebook's cloud coding, **On = -1, Off = +1**. The paper uses the opposite cloud coding: signed effects involving Cloud reverse, while their absolute percentage influence remains unchanged. For Figure 10, use the effect-normalized plotting cells; the earlier printed sum-of-squares percentages use a different normalization.

The plotting cells display figures inline. To save one, create `plots/` and enable the corresponding `plt.savefig(...)` call before `plt.show()`.

### Locate clustering outputs

The clustering notebook builds **240 observations**: two algorithms × four scenarios × 30 seeds. It standardizes the metrics, computes PCA, evaluates cluster counts from 2 to 10, and fits a five-cluster model. Keep `random_number = seed_list[6]` and the existing label remapping for the paper's cluster numbering.

It writes these files directly under `notebooks/`:

```text
final_dataset.csv
list_of_coordinates.csv
kmeans_clusters_pca_to_paper.png
cluster*_data.csv
cluster*_linear_model_with_equation.png
```

Some regression filenames retain earlier cluster numbers. The dataframe filters and remapped labels identify the clusters used in the paper.

## 5. Reproduce sensitivity and execution overhead

### Hyperparameter sensitivity: Figure 3

Run [`nb_hyper_sensitivity_analysis.ipynb`](notebooks/nb_hyper_sensitivity_analysis.ipynb) from top to bottom. Its first code cell defines the 30 seeds, `2ec_high_off`, seven baseline weights of `1.0`, and `weight_values = np.linspace(0.25, 2.0, 8)`.

Keep these defaults for **1,680 simulations**: seven parameters × eight values × 30 seeds. Each parameter changes independently while the others remain at baseline. This notebook retains raw logs in memory until the simulations finish, so it can require substantially more time and memory than the installation check.

Outputs in `notebooks/results/`:

```text
tetris_hyper_sensitivity_server_raw.csv
tetris_hyper_sensitivity_user_raw.csv
tetris_hyper_sensitivity_by_seed.csv
tetris_hyper_sensitivity_summary.csv
tetris_hyper_sensitivity.png
```

The summary CSVs and figure are included. The two large raw CSVs are excluded by `.gitignore` and must be generated by running the notebook.

### Execution overhead: Table 3

Run [`nb_tetris_complexity.ipynb`](notebooks/nb_tetris_complexity.ipynb), then [`nb_thea_complexity.ipynb`](notebooks/nb_thea_complexity.ipynb), including the final summary cell in each. Their defaults cover 30 seeds and all five scenarios: **300 simulations** in total.

The timing wrapper measures only each placement algorithm call using `time.perf_counter_ns()`. Once both algorithms' step-level CSVs exist, the final cell also creates the combined summaries:

```text
results/tetris_overhead_steps.csv
results/tetris_overhead_summary.csv
results/thea_overhead_steps.csv
results/thea_overhead_summary.csv
results/overhead_summary_tetris_thea.csv
results/overhead_overall_tetris_thea.csv
```

These notebooks also overwrite the standard Tetris/Thea simulation CSVs. Measured times depend on hardware and system load; compare the methodology and summary statistics rather than expecting identical millisecond values.

## 6. Check your run

A complete baseline run produces the **1,620 simulation CSVs** listed by the naming pattern in Section 3. The analysis notebooks load the required algorithm/scenario/seed combinations and report missing files if any are absent. File presence alone does not distinguish included data from a new run.

| Problem | What to check |
| --- | --- |
| `ModuleNotFoundError` | Select **Python (Tetris)** and install packages using that environment's `python -m pip`. |
| A dataset JSON cannot be found | The kernel must run in `notebooks/`, with `../datasets/` available. |
| A results CSV cannot be found | Run the corresponding simulation with all required seeds and scenarios, then restart the analysis kernel. |
| Results differ after a shortened run | Restore both lists and their original order; avoid mixing partial runs with included CSVs. |
| Factorial plots still use earlier values | Replace `all_experiments_input` using the export procedure in Section 4. |
| Dependency or simulator API errors | Recreate the Python 3.10 environment using `requirements.txt`. |

Record `git rev-parse HEAD` and save `python -m pip freeze` alongside your results. Compare aggregate metrics and confidence intervals with the paper; library versions can affect numerical results, and hardware affects runtime measurements.

## Citation and contact

```bibtex
@article{almeida2026tetris,
  title   = {Tetris: An SLA-aware application placement strategy in the Edge--Cloud Continuum},
  author  = {Almeida, L. and Ramprasad, B. and de Lara, E. and Peixoto, M.},
  journal = {Computer Communications},
  volume  = {259},
  pages   = {108678},
  year    = {2026},
  doi     = {10.1016/j.comcom.2026.108678}
}
```

**Lucas Mascarenhas Almeida** · [lucasmascarenhas@ufba.br](mailto:lucasmascarenhas@ufba.br)

## Branch disclaimer

**Always run the experiments from the `main` branch.** It is the most stable and reliable version of this repository and the reference branch for this reproduction tutorial. Other branches may contain experimental or incomplete changes.

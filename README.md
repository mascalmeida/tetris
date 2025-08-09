# Tetris: An SLA-aware Application Placement Strategy in the Edge-Cloud Continuum

This repository contains the full implementation, simulation experiments, and analysis for **Tetris**, a heuristic algorithm designed to manage SLA-compliant modular application placement in edge-cloud environments. This project was developed as part of my Master's thesis in Computer Science at the Federal University of Bahia (UFBA), Brazil.

## 🎯 Overview

Tetris is an innovative placement strategy that optimizes the distribution of application tasks across edge and cloud servers. It balances latency, resource availability, and energy consumption while minimizing SLA violations and resource fragmentation.

## 📌 Features

- SLA-aware modular application placement
- Deadline- and capacity-aware scheduling
- Resource fragmentation avoidance
- Drop and latency violation mitigation
- Integration with [EdgeSimPy](https://github.com/EdgeSimPy/EdgeSimPy)
- Fully reproducible experiments

## 📁 Project Structure

```bash
tetris/
│
├── datasets/ # Input configurations and results from simulations
├── notebooks/ # Jupyter notebooks for running experiments and data analysis
├── requirements.txt
├── .gitignore
└── README.md
```

## 🚀 Getting Started

### Prerequisites

- Python 3.9+
- `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`
- [EdgeSimPy simulator](https://github.com/EdgeSimPy/EdgeSimPy)

### Installation

Clone the repository:

```bash
git clone https://github.com/mascalmeida/tetris.git
cd tetris
pip install -r requirements.txt
```

Install the simulator:

```bash
pip install -q git+https://github.com/EdgeSimPy/EdgeSimPy.git
```

### Running Experiments

Before running experiments, make sure you have the required simulator and dependencies installed (see the Installation section).

#### 1. Choose the algorithm
Available algorithms include: tetris, thea, argos, faticanti, edf, mcf, edf+mcf

#### 2. Edit the corresponding notebook
Each algorithm has a notebook file following the pattern: nb_<algorithm>.ipynb

For example: nb_tetris.ipynb, nb_thea.ipynb, nb_edf.ipynb.

#### 3. Set experiment parameters
Inside the notebook, modify the configuration section to set:

algorithm_name: the name of the strategy (e.g., "tetris")

scenario_path: the input topology/scenario file (from datasets/)

output_path: where results will be saved

seeds: list of seeds for repetitions

Any algorithm-specific parameters (e.g., SLA)

#### 4. Run all cells
Open the notebook with Jupyter:

Then run all cells (Runtime > Run All), and the experiment will execute and store results in the appropriate directory.

#### 5. Analyze Results
Use the output CSV files or use the nb_results notebooks to generate the same visualizations from the paper.


👨‍💻 Author

Lucas Mascarenhas Almeida

MSc in Computer Science, UFBA

📧 lucasmascalmeida@gmail.com

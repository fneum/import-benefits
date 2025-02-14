# Energy Imports and Infrastructure in a Carbon-Neutral European Energy System

Code for preprint article [2404.03927](https://arxiv.org/abs/2404.03927) currently under review.

**Abstract:**

> Importing renewable energy to Europe may offer many potential benefits, including reduced energy costs, lower pressure on infrastructure development, and less land-use within Europe. However, there remain many open questions: on the achievable cost reductions, how much should be imported, whether the energy vector should be electricity, hydrogen or hydrogen derivatives like ammonia or steel, and their impact on Europe's domestic energy infrastructure needs. This study integrates the TRACE global energy supply chain model with the sector-coupled energy system model for Europe, PyPSA-Eur, to explore net-zero emission scenarios with varying import volumes, costs, and vectors. We find system cost reductions of 1-11%, within import cost variations of +/- 20% around our central estimate, with diminishing returns for larger import volumes and a preference for methanol, steel and hydrogen imports. Keeping some domestic power-to-X production is beneficial for integrating variable renewables, leveraging local sustainable carbon sources and utilising some waste heat from fuel synthesis. Across scenarios, power grid reinforcements are more stable than hydrogen pipeline expansion. Our findings highlight the need for coordinating import strategies with infrastructure policy and reveal maneuvering space for incorporating non-cost decision factors.


## Installation

Clone the `git` repository from Github and enter the directory:

```sh
git clone --recurse-submodules git@github.com:fneum/import-benefits
cd import-benefits
```

Consider adding `git clone --depth=1 --shallow-submodules` to skip to only retrieve the latest commits and not the full git history.

Use `conda` or a similar alternative to install the necessary **Python environments** for running TRACE and PyPSA-Eur:

```sh
conda env create -f workflow/trace/envs/environment.20240826-z1.yaml
conda env create -f workflow/pypsa-eur/envs/environment.20240826-z1.yaml
```

The installation process should take only a few minutes; if it takes longer, consider updating your `conda` installation before retrying to create the environment:

```sh
conda deactivate
conda update conda
```

Additionally, install a **solver** of your choice, e.g. Gurobi (requires license, e.g. from https://www.gurobi.com/academia/academic-program-and-licenses):

```sh
conda activate trace-20240826-z1
conda install -c gurobi gurobi"==11.0.1"
conda activate pypsa-eur-20240826-z1
conda install -c gurobi gurobi"==11.0.1"
```

Both workflows can be executed on Linux, Mac or Windows (WSL). There are no futher specific non-standard hardware requirements.

## Usage of TRACE

TRACE uses the [`snakemake` workflow management tool](https://snakemake.readthedocs.io/en/stable/).

Running a limited number of scenarios is possible on a local machine.

To run the workflow and reproduce the model runs presented in the paper, execute the following commands from the `root` directory of the project:

```sh
cd workflow/trace
conda activate trace-20240826-z1
snakemake -call all_scenario_results -n # dry-run
snakemake -call all_scenario_results
```

The results are stored in `results/results.parquet` (and in alternative file format as `results/results.csv`).

Two files are required for the PyPSA-Eur workflow and should be transferred to
`.workflow/pypsa-eur/data/imports` before continuing. From the `root` of this
repository, run:

```sh
mkdir -p workflow/pypsa-eur/data/imports
cp workflow/trace/results/{results.parquet,combined_weighted_generator_timeseries.nc} workflow/pypsa-eur/data/imports
```

Alternatively, in case you want to reproduce PyPSA-Eur runs without re-running
TRACE, download the following two files into the
`./workflow/pypsa-eur/data/imports` directory.

```sh
mkdir -p workflow/pypsa-eur/data/imports
wget -P workflow/pypsa-eur/data/imports \
  https://zenodo.org/records/14xxx/files/combined_weighted_generator_timeseries.nc \
  https://zenodo.org/records/14xxx/files/results.parquet
```

## Usage of PyPSA-Eur

PyPSA-Eur also uses the [`snakemake` workflow management tool](https://snakemake.readthedocs.io/en/stable/).

Apart from solving the optimisation problem of a high-resolution model configuration, the workflow can be executed on a local machine. The preprocessing can take 1-2 hours and may require 16-24 GB of RAM.

To run the workflow and reproduce the model runs presented in the paper, execute the following commands from the `root` directory of the project:

```sh
cd workflow/pypsa-eur
conda activate pypsa-eur-20240826-z1
snakemake -call make_global_summary --configfile=config/config.20240826-z1.yaml -n # dry-run
snakemake -call make_global_summary --configfile=config/config.20240826-z1.yaml
```

These larger runs with higher temporal resolution require a high-performance computing setup with at least 125-150 GB RAM available and access to multiple cores is beneficial for runtime improvements but not strictly necessary. However, the runtime for a single scenario can exceed 48-72 hours. As there are around 350 scenarios, it is recommended to use the [`snakemake` cluster integration](https://snakemake.readthedocs.io/en/v7.19.1/executing/cluster.html) to parallelize scenarios. Additional `snakemake` settings are required depending on the compute infrastructure available.

The `yaml` files referenced above can be used to edit and configure the scenarios, e.g. by adding additional cases, changing some assumptions or reducing complexity.

More detailed usage instructions for the underlying PyPSA-Eur model can be found
at https://pypsa-eur.readthedocs.io/en/v0.13.0/.

## Outputs and Inspection

Running the workflow will create various intermediate resources and results files, including `.csv` files with summary data, `.pdf` files with summary plots and `.nc` files, most of which are PyPSA network files before and after solving.

These files are placed in the `workflow/pypsa-eur/resources` and `workflow/pypsa-eur/results` directories.

PyPSA networks can be read and inspected with Python in the following way:

```py
import pypsa
fn = f"workflow/pypsa-eur/results/20240826-z1/postnetworks/elec_s_115_lvopt__imp_2050.nc"
n = pypsa.Network(fn)
n.statistics()
n.statistics.capex() + n.statistics.opex()
n.statistics.energy_balance()
n.buses_t.marginal_price
```

## License

[MIT](LICENSE)

# Energy Imports and Infrastructure in a Carbon-Neutral European Energy System

Code for preprint article [2404.03927](https://arxiv.org/abs/2404.03927) currently under review.

**Abstract:**

> Importing renewable energy to Europe may offer many potential benefits, including reduced energy costs, lower pressure on infrastructure development, and less land-use within Europe. However, there remain many open questions: on the achievable cost reductions, how much should be imported, whether the energy vector should be electricity, hydrogen or hydrogen derivatives like ammonia or steel, and their impact on Europe's domestic energy infrastructure needs. This study integrates the TRACE global energy supply chain model with the sector-coupled energy system model for Europe, PyPSA-Eur, to explore net-zero emission scenarios with varying import volumes, costs, and vectors. We find system cost reductions of 1-10%, within import cost variations of +-20% around our central estimate, with diminishing returns for larger import volumes and a preference for methanol, steel and hydrogen imports. Keeping some domestic power-to-X production is beneficial for integrating variable renewables, leveraging local sustainable carbon sources and utilising some waste heat from fuel synthesis. Across scenarios, power grid reinforcements are more stable than hydrogen pipeline expansion. Our findings highlight the need for coordinating import strategies with infrastructure policy and reveal maneuvering space for incorporating non-cost decision factors. 

## Installation

Clone the `git` repository from Github and enter the directory:

```sh
git clone --recurse-submodules git@github.com:fneum/import-benefits
cd import-benefits
```

Use `conda` or a similar alternative to install the necessary **Python environment**:

```sh
conda env create -f workflow/pypsa-eur/envs/environment.yaml -n pypsa-eur-imports
```

The installation process should take only a few minutes; if it takes longer, consider updating your `conda` installation before retrying to create the environment:

```sh
conda deactivate
conda update conda
```

Additionally, install a **solver** of your choice, e.g. Gurobi:

```sh
conda activate pypsa-eur-imports
conda install -c gurobi gurobi
```

Finally, there is a data requirement that is not included in the repository and not downloaded automatically.

Download the following two files into the `./workflow/pypsa-eur/data/imports` directory.

```sh
cd workflow/pypsa-eur/data
mkdir imports
wget https://tubcloud.tu-berlin.de/s/Zpntz6kB7HDdP7m/download/combined_weighted_generator_timeseries.nc
wget https://tubcloud.tu-berlin.de/s/T4CfYwfWZX8Xa7E/download/results.parquet
```

The workflow can be executed on Linux, Mac or Windows. There are no futher specific non-standard hardware requirements.

## Usage

This repository uses the [`snakemake` workflow management tool](https://snakemake.readthedocs.io/en/stable/).

Apart from solving the optimisation problem of a high-resolution model configuration, the workflow can be executed on a local machine. The preprocessing can take 1-2 hours and may require 16-24 GB of RAM.

To run a test workflow resulting in a model with reduced temporal resolution with a runtime limited to around 2 hours so that it is suitable for local execution, execute the following commands from the `root` directory of the project:

```sh
cd workflow/pypsa-eur
conda activate pypsa-eur-imports
snakemake -call --configfile=config/config.small.yaml
```

To reproduce the model runs presented in the paper, execute the following commands from the `root` directory of the project:

```sh
cd workflow/pypsa-eur
conda activate pypsa-eur-imports
snakemake -call --configfile=config/config.20240826-z1.yaml
```

These larger runs with higher temporal resolution require a high-performance computing setup with at least 125-150 GB RAM available and access to multiple cores is beneficial for runtime improvements but not strictly necessary. However, the runtime for a single scenario can exceed 48-72 hours. As there are around 250 scenarios, it is recommended to use the [`snakemake` cluster integration](https://snakemake.readthedocs.io/en/v7.19.1/executing/cluster.html) to parallelize scenarios.

The `yaml` files referenced above can be used to edit and configure the scenarios, e.g. by adding additional cases or changing some assumptions.

More detailed usage instructions for the underlying PyPSA-Eur model can be found
at https://pypsa-eur.readthedocs.io/.

## Outputs and Inspection

Running the workflow will create various intermediate resources and results files, including `.csv` files with summary data, `.pdf` files with summary plots and `.nc` files, most of which are PyPSA network files before or after solving.

These files are placed in the `workflow/pypsa-eur/resources` and `workflow/pypsa-eur/results` directories.

PyPSA networks can be read and inspected with Python in the following way:

```py
import pypsa
fn = "workflow/pypsa-eur/results/small/postnetworks/elec_s_110_lvopt__imp_2050.nc"
n = pypsa.Network(fn)
n.statistics()
n.statistics.energy_balance()
n.buses_t.marginal_price
```

## License

[MIT](LICENSE)

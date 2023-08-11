# Infrastructure Planning and Energy Imports into Europe

[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/fneum/import-benefits/main.svg?badge_token=mAxzRKyvSDiYmZYY0ZnVLg)](https://results.pre-commit.ci/latest/github/fneum/import-benefits/main?badge_token=mAxzRKyvSDiYmZYY0ZnVLg)

## Installation

```sh
git clone --recurse-submodules git@github.com:fneum/import-benefits
```

Using `micromamba` or similar alternative, install with command:

```sh
micromamba create -f workflow/pypsa-eur/envs/environment.yaml -n pypsa-eur-20230811
```

cf. https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html#conda-yaml-spec-files

If you use Gurobi:

```sh
mm activate pypsa-eur-20230811
mm install -c gurobi gurobi
```

## Usage

From `root` of project:

```sh
snakemake --profile slurm plot_summary --configfile=config/config.main.yaml
```

## License
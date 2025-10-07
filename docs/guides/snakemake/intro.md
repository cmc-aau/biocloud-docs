# Getting Started

## Introduction
Snakemake is a workflow management system that simplifies the specification and execution of complex data processing and analysis pipelines using both self-made rules (tasks or steps) and/or reusable community-made [wrappers](https://snakemake-wrappers.readthedocs.io/en/stable/). It's an open-source tool, designed to scale from simple to highly complex workflows, providing both flexibility and reproducibility for any computing environment. In comparison to shell scripts Snakemake is much more efficient because it automatically calculates rule dependencies between the individual rules depending on expected inputs and outputs as well as the input data itself, which greatly simplifies parallel computing. Snakemake workflows are written in Python in a standardized format, but has full support for any scripting language. It also has advanced features that automates the installation of required software tools ensuring complete portability and compatibility. Last, but not least, Snakemake has inherent support for [cluster execution](https://snakemake.readthedocs.io/en/stable/tutorial/additional_features.html#cluster-execution) where workflow steps are translated and submitted as separate batch jobs with individually specified resource requirements to maximize resource utilization, while also managing the submission process, job completion monitoring, and much more.

Snakemake provides detailed (and quite messy) documentation at [snakemake.readthedocs.io](https://snakemake.readthedocs.io/en/stable/), however the purpose of this guide is to boil things down a bit to help you get started on the BioCloud HPC for common use-cases and provide a standardized starting point for everyone. Let's get started.

## Preparing project folder for Snakemake
The easiest way to get started with a new project is to create a new git repository on GitHub from our [template GitHub repository](https://github.com/cmc-aau/snakemake_project_template). You of course need a GitHub account first, but that will also come in handy in many other situations. The template repository includes a minimal example workflow which will be explained on the [next page](tutorial.md), so that you can follow along. Clone the repo afterwards to your home folder somewhere and start filling in after this guide. According to the authors of Snakemake, the ideal way to [structure a Snakemake project folder](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility) is like this (with a few additions):


```shell
project/
├── .gitignore
├── LICENSE
├── README.md
├── analysis/
├── config/
│   ├── README.md
│   └── config.yaml
├── data/
├── logs/
├── profiles/
├── results/
└── workflow/
    ├── envs/
    │   ├── tool1.yml
    │   └── tool2.yml
    ├── notebooks/
    │   ├── notebook1.ipynb
    │   └── notebook2.ipynb
    ├── report/
    │   ├── plot1.rst
    │   └── plot2.rst
    ├── rules/
    │   ├── rule1.smk
    │   └── rule2.smk
    ├── scripts/
    │   ├── script1.R
    │   ├── script2.sh
    │   └── script3.py
    └── Snakefile

```

At first it might seem like a lot of files and folders, but workflows can grow quickly, so it's nice with a proper structure from the beginning. You can of course also put everything in a separate subfolder if developing a workflow is not the main goal of the project.

## Installation
To setup Snakemake use the [`environment.yaml`](https://github.com/cmc-aau/snakemake_project_template/blob/main/environment.yml) file provided in the template repository to create a [conda environment from a file](../../software/conda.md#creating-an-environment) for the project with `conda env create -f environment.yml`. It's always best practice to note software dependencies down somewhere in a file, but alternatively you can also load the prebuilt [software module](../../software/modules.md) using:
```
module load snakemake/7.18.2-foss-2020b
```

or create and activate a conda environment for the project on the command line:
```
conda create -c conda-forge -c bioconda -n snakemake snakemake==7.18.2
conda activate snakemake
```

???- "Note: The Snakemake version matters"
      Snakemake keeps track of files but also the Snakemake version used to run the workflow. Therefore it is important to use the same version of snakemake when re-running a workflow. If you try to run a workflow with a different version of snakemake, it will re-run all samples already processed. Therefore be explicit about the version installed and ensure that it's written down somewhere, fx in the project `README` file or a conda `environment.yml` file.

If you use [Visual Studio Code](../../access/ssh.md#visual-studio-code), it can be handy to install the `snakemake` extension to enable syntax highlighting, linting, and autocompletion when developing workflows.

## Workflow catalogs
Before you go ahead and create a new workflow from scratch, it's a good idea to have a look in the public [Snakemake workflow catalog](https://snakemake.github.io/snakemake-workflow-catalog/), because someone might have already made something similar you could use instead. Also have a look at [WorkflowHub](https://workflowhub.eu/) for workflows made with Snakemake, but also other workflow tools.

The template repository is also configured with a few standard [GitHub Actions](https://github.com/features/actions) to automatically test the workflow, and passing some tests is required if you also wish to publish it to the catalog. As long as your repository is public and the `README.md` file contains the two words **snakemake** and **workflow** (and a few other [rules for inclusion](https://snakemake.github.io/snakemake-workflow-catalog/?rules=true)) it will show up in the catalogue automatically. You don't have to do anything specific for it to be listed there, they simply scan GitHub at regular intervals. This is the best way to share your workflow for others to use, but it's optional.

Lastly, if you stick to the standardized folder structure above and keep the workflow in a GitHub repository (public or private) you can easily reuse your workflow across multiple projects using [snakedeploy](https://snakedeploy.readthedocs.io/en/latest/). This is also how workflows are reused from the catalog.

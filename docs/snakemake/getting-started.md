# Getting Started

## Installation
There are two ways to set up snakemake on biocloud. The first is to use the module system:
```bash
module load snakemake/7.18.2-foss-2020b
```

The second is to create a conda environment:
```bash
mamba create -c conda-forge -c bioconda -n snakemake snakemake
conda activate snakemake
```

> **Note:** Snakemake keeps track of files but also the snakemake version, which ran the workflow. Therefore it is important to use the same version of snakemake when re-running a workflow. If you try to run a workflow with a different version of snakemake, it will try to re-run all samples already processed. *Therefore, use the same version of snakemake at all times*.


## Visual Studio Code
If you are using Visual Studio Code, it is possible to install the snakemake extension. This extension will give you syntax highlighting, linting, and autocompletion. To install the extension, simply search for `snakemake` in the extensions tab and install it.

## Setting up a workflow
According to the author of snakemake, the best way to structure a workflow is like so:

```
.
├── config
│   ├── config.yaml
│   └── README.md
├── LICENSE
├── README.md
└── workflow
    ├── envs
    │   ├── env_1.yaml
    │   └── env_2.yaml
    ├── rules
    │   ├── module_1.smk
    │   └── module_2.smk
    └── Snakefile
```

This template can be found at snakemakes github page [here](https://github.com/snakemake-workflows/snakemake-workflow-template).

In essence, the workflow is split into two parts: the `Snakefile` and the `rules`. The `Snakefile` is the main file, which specifies all outputs, and thereby which rules to be executed. The `rules` folder contains the rules, which are the individual steps of the workflow. The `config` folder contains the configuration file, which contains the parameters for the workflow. The `LICENSE` and `README.md` files are self-explanatory.

### Snakefile
The `Snakefile` is the main file of the workflow. A typical `Snakefile` looks like this:

```python
import os
# import other modules if needed

# load config file
SNAKEDIR = os.path.dirname(workflow.snakefile) # get the directory of the Snakefile

configfile: os.path.join(SNAKEDIR, "..", "config", "config.yaml") 

# include the rules
include: "rules/module_1.smk"
include: "rules/module_2.smk"

# define the output files
rule all:
    input:
        expand("results/{sample}.txt", sample=config["samples"])
```

Alot of things are going on here. Firstly, the `configfile` is loaded. Once it is loaded, the `config` variable is available. Often the config file will output directories, database paths, parameters for tools etc. A typical config file could look like this:

```yaml
samples:
    - A
    - B
output_dir: "results"
tmp_dir: "tmp"
db_path: "data/db.sqlite"
```


Next, we import all the rules, which contains the steps from getting from the input to the output. 

For example the `module_1.smk` file could look like this: (give it a better name of course)

```python
rule module_1:
    input:
        "data/{sample}.txt"
    output:
        "tmp/{sample}.txt"
    shell:
        "cat {input} > {output}"
```

and the `module_2.smk` file could look like this:

```python
rule module_2:
    input:
        "tmp/{sample}.txt"
    output:
        "results/{sample}.txt"
    shell:
        "cat {input} > {output}"
```

From these two rules, we can see that the `module_1` rule takes the input from the `data` folder and outputs it to the `tmp` folder. The `module_2` rule takes the input from the `tmp` folder and outputs it to the `results` folder.


Going back the `Snakefile`, there is the `rule all`, which is a special rule in snakemake. It should always be the first rule in a `Snakefile`, and its main purpose is to specify the output files of the workflow and set the `wildcards` variable. In the above example, the `rule all` specifies a file called `results/{sample}.txt`. This might look a bit weird, but it is actually a very powerful feature of snakemake. The `{sample}` part is a `wildcard`, which means that snakemake will populate it with whatever you specify sample to be.

In the above example we would expect the configfile to look like this:

```yaml
samples:
    - A
    - B
```

However, it is also possible to specify the samples directly in the `Snakefile` like so:

```python
rule all:
    input:
        expand("results/{sample}.txt", sample=["A", "B"])
```

will give the output files:

```
results/A.txt
results/B.txt
```

or another example:

```python
rule all:
    input:
        expand("results/{sample}.txt", sample=["MFD1", "MFD2", "MFD3"])
```

will give the output files (provided that the input for creating these files are present):

```
results/MFD1.txt
results/MFD2.txt
results/MFD3.txt
```

The only thing that the `expand` function does is to create a list of strings by expanding the `{sample}` wildcard. This is how snakemake knows which files to create. `expand` creates the list `["results/A.txt", "results/B.txt"]` and by looking at the inputs and outputs of the rules, snakemake knows that it needs to run `module_1` and `module_2` to get from the input to the output.


> **Note:** It is possible to set the input to rule all manually. This is just to show how the `expand` function works. The following code will also work: 
> ```python
> rule all:
>     input:
>         ["results/A.txt", "results/B.txt"]
> ```



## Running a workflow
### Dry run
Before running a workflow, it is a good idea to do a dry run. This will show you what snakemake will do, without actually doing anything. To do a dry run, simply run the following command:

```bash
snakemake -s <path to Snakefile> --dryrun
```
or
```bash
snakemake -s <path to Snakefile> -n
```
Include the `-p` flag to print the shell commands that will be executed.
```bash
snakemake -s <path to Snakefile> -np
```

### Running a workflow

To run a workflow, simply run the following command:

```bash
snakemake -s <path to Snakefile> --cores <number of cores>
```

### Cluster execution
Snakemake can also be run on a cluster. This is done by specifying the `--cluster` flag. For instance, to run on the cluster, simply run the following command:

```bash
snakemake -s <path to Snakefile> --cluster "sbatch --particion {resources.partition} --time={resources.walltime} --mem={resources.mem} --cpus-per-task={threads} --ntasks=1--output=logs/slurm_output_{rule}_{wildcards.sample}_%j.out”" --jobs 100
```


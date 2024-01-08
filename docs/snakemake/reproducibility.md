# Making Snakemake Reproducible
## Overview
A key part of snakemake is reproducibility, which is a major problem in bioinformatics. One thing to ensure reproducibility, is to keep track of the software versions used in the analysis. Snakemake has a built-in way of doing this, which we will cover in this section.

## Conda environments
Snakemake can easily be integrated with conda. All that is required is a `yaml` file located in the `envs` folder ([see getting-started](getting-started.md#setting-up-a-workflow)).


The `yaml` file should look something like this:

```yaml
name: <tool>
channels:
  - bioconda
  - conda-forge
dependencies:
  - <tool>=<version>
```

For example, if we want to use `samtools` version `1.12`, the `yaml` file should look like this:

```yaml
name: samtools
channels:
  - bioconda
  - conda-forge
dependencies:
  - samtools=1.12
```

We can then specify for each rule, which conda environment it should use. For example, if we want to use the `samtools` environment for the `samtools_index` rule, we can do it like so:

```python
rule samtools_index:
    input:
        "data/{sample}.bam"
    output:
        "data/{sample}.bam.bai"
    conda:
        "../envs/samtools.yaml"
    shell:
        "samtools index {input}"
```

Notice the `conda` keyword.

We can now run the workflow like so:

```bash
snakemake --cores 1 --use-conda 
```

The `--use-conda` flag tells snakemake to use conda environments for each rule if specified. First time you run the workflow, snakemake will first create the conda environment (using mamba), and then it will reuse it for subsequent runs.


### Using a already existing conda environment
It is also possible to specify an existing conda environment. For example, if we have a conda environment called `samtools_env` that we want to use for the `samtools_index` rule, we can do it like so:

```python
rule samtools_index:
    conda:
        "samtools_env"
    input:
        "data/{sample}.bam"
    output:
        "data/{sample}.bam.bai"
    shell:
        "samtools index {input}"
```

In order for this to work, the `samtools_env` environment must be installed and show up when running `conda env list`.

```bash
conda env list
```

```
# conda environments:
#
base                  *  /home/username/miniconda3
samtools_env             /home/username/miniconda3/envs/samtools_env
```


## Envmodules
It is also possible to use the module system on the bioservers, however, this is not recommended, as it is not as reproducible as conda environments. However, if you want to use the module system, you can do it like so:

```python
rule samtools_index:
    input:
        "data/{sample}.bam"
    output:
        "data/{sample}.bam.bai"
    envmodules:
        "samtools/1.12"
    shell:
        "samtools index {input}"
```

And run it like so:

```bash
snakemake --cores 1 --use-envmodules
```

> I would not recommend this. Use conda environments instead.
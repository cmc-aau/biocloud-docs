# Wildcards, resources, logging, and configuration
## Wildcards
In the getting started section we saw how to create a simple workflow. However, in the real world, we often have multiple samples, and writing them in the configfile can be a bit tedious. Here I will show you another way to set the wildcards.

Given the following `Snakefile`:

```python
import os
# import other modules if needed


# define the output files
rule all:
    input:
        expand("results/{sample}.txt", sample=<SAMPLES>)



rule module_1:
    input:
        "data/{sample}.txt"
    output:
        "tmp/{sample}.txt"
    shell:
        "cat {input} > {output}"

```


We want to populate the `<SAMPLES>` such that snakemake knows, which samples to run. Let's say that our data folder looks like this:

```
data/
    sample1.txt
    sample2.txt
    sample3.txt
    .
    .
    .
    sample1000.txt
```

Specifying these in a configfile would not make sense.

Instead we can use the `glob` or `glob_wildcards` to populate the `<SAMPLES>` variable. The `glob_wildcards` function takes a pattern and a directory and returns a list of all the files that match the pattern in the directory. In our case we want all the files in the `data` folder that ends with `.txt`. We can do this like this:

### Using glob_wildcards
```python
import os
from snakemake.io import glob_wildcards
# import other modules if needed

SAMPLES = glob_wildcards("data/{sample}.txt").sample
```

### Using glob
```python
import os
import glob
# import other modules if needed

FILES = glob.glob("data/*.txt")
SAMPLES = [os.path.basename(f).split(".")[0] for f in FILES]
```

The `FILES` variable will now contain a list of all the files in the `data` folder that ends with `.txt`. The `SAMPLES` variable will contain a list of all the files in the `data` folder that ends with `.txt` without the `.txt` extension. The `SAMPLES` variable will look like this:

```python
["sample1", "sample2", "sample3", ..., "sample1000"]
```

In any case, we can now use the `SAMPLES` variable in the `rule all`:

```python
import os
import glob
# import other modules if needed

FILES = glob.glob("data/*.txt")
SAMPLES = [os.path.basename(f).split(".")[0] for f in FILES]

# or
# 
# from snakemake.io import glob_wildcards
# SAMPLES = glob_wildcards("data/{sample}.txt").sample

# define the output files
rule all:
    input:
        expand("results/{sample}.txt", sample=SAMPLES)



rule module_1:
    input:
        "data/{sample}.txt"
    output:
        "tmp/{sample}.txt"
    shell:
        "cat {input} > {output}"
```


Another way, which I like to use, is to specify all samples in a TSV file. This is especially useful if you have metadata for each sample. Let's say that we have a TSV file that looks like this:

```
sample  metadata
sample1 metadata1
sample2 metadata2
sample3 metadata3
.
.
.
sample1000 metadata1000
```

We can then use the `pandas` module to read the TSV file and extract the samples:

```python
import os
import pandas as pd
# import other modules if needed

df = pd.read_csv("metadata.tsv", sep="\t")
SAMPLES = df["sample"].tolist()
```
It is as simple as that. 


This is especially useful if a sample has multiple files associated with it. For example, if we have paired-end sequencing data, we could have a TSV file that looks like this:

```
sample  R1  R2
sample1 <path-to>/sample1_R1.fastq.gz <path-to>/sample1_R2.fastq.gz
sample2 <path-to>/sample2_R1.fastq.gz <path-to>/sample2_R2.fastq.gz
sample3 <path-to>/sample3_R1.fastq.gz <path-to>/sample3_R2.fastq.gz
.
.
.
sample1000 <path-to>/sample1000_R1.fastq.gz <path-to>/sample1000_R2.fastq.gz
```

Let's say that we want to process these files with `fastp`, which requires both the R1 and R2 files. We can then use the `pandas` module to read the TSV file and extract the samples:

```python
import os
import pandas as pd
# import other modules if needed

df = pd.read_csv("metadata.tsv", sep="\t")
SAMPLES = df["sample"].tolist()

# define the output files
rule all:
    input:
        expand("results/{sample}_R1_qc.fastq.gz", sample=SAMPLES)
        expand("results/{sample}_R2_qc.fastq.gz", sample=SAMPLES)


rule fastp:
    input:
        R1 = lambda wildcards: df[df["sample"] == wildcards.sample]["R1"].tolist()[0],
        R2 = lambda wildcards: df[df["sample"] == wildcards.sample]["R2"].tolist()[0]
    output:
        R1 = "results/{sample}_R1_qc.fastq.gz",
        R2 = "results/{sample}_R2_qc.fastq.gz"
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2}
        """
```

Here we use the dataframe to extract the R1 and R2 files for each sample as input for the `fastp` rule. We also use the dataframe to specify the output files for the `rule all`. Having a metadata file makes the pipeline feel more as a tool, which can be used for multiple projects.

## Temporary files and protected files
Often in workflows, we need to create temporary files. These files are not needed after the workflow is done, and can be deleted. Snakemake has a built-in feature for this. Going back to the getting started workflow.

```python
rule module_1:
    input:
        "data/{sample}.txt"
    output:
        "tmp/{sample}.txt"
    shell:
        "cat {input} > {output}"

rule module_2:
    input:
        "tmp/{sample}.txt"
    output:
        "results/{sample}.txt"
    shell:
        "cat {input} > {output}"
```

We see there is a temporary file. We can tell snakemake that this file is temporary by adding the `temp` keyword to the rule. This will tell snakemake to delete the file, when it is not needed anymore:

```python
rule module_1:
    input:
        "data/{sample}.txt"
    output:
        temp("tmp/{sample}.txt")
    shell:
        "cat {input} > {output}"

rule module_2:
    input:
        "tmp/{sample}.txt"
    output:
        "results/{sample}.txt"
    shell:
        "cat {input} > {output}"
```

After executing the `module_2` rule, snakemake will delete the temporary file.

Additionally, we can also tell snakemake to protect a file. This way we cannot accidentally overwrite the file. We can do this by adding the `protected` keyword to the rule. This will tell snakemake to not overwrite the file, if it already exists:

```python
rule module_2:
    input:
        "tmp/{sample}.txt"
    output:
        protected("results/{sample}.txt")
    shell:
        "cat {input} > {output}"
```

## Adding resources to rules
### Threads
So far, all rules have had the basic structure:

```python
rule <rule_name>:
    input:
        <input_files>
    output:
        <output_files>
    shell:
        <shell_command>
```

An input, output, and command. However, the default behavior of snakemake is that unless specified, each rule will be executed with 1 thread. This is not optimal, as most tools can utilize multiple threads. We can specify the number of threads for each rule by adding the `threads` keyword to the rule. As an example, let's say that we want to run `fastp` with 8 threads. We can do this like so:

```python
rule fastp:
    input:
        R1 = <R1>,
        R2 = <R2>
    output:
        R1 = <R1>,
        R2 = <R2>
    threads: 8
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2} \
        --thread {threads}
        """
```

The reason this works is that `fastp` takes an argument called `--thread` which specifies the number of threads to use. By using the `{threads}` keyword in the `shell` command, snakemake will populate it with the number of threads specified in the `threads` keyword. This is a very useful feature of snakemake, as it allows us to easily specify the number of threads for each rule.

We can then run snakemake with 8 threads like so:

```bash
snakemake --cores 8
```
> 
or if we have multiple samples, we can run it like so:

```bash
snakemake --cores 40
```

This will allow snakemake to run 5 jobs in parallel, each with 8 threads.

> **Note:** running snakemake with fewer threads (`--cores 1`) will downscale the threads argument. For example, if we run snakemake with 1 thread, the `fastp` rule will be executed with 1 thread.

## Logging
Another powerful feature of snakemake is the logging, which is is useful for debugging. We can specify the log file for each rule by adding the `log` keyword to the rule. For example, if we want to log the output of the `fastp` rule to a file called `fastp.log`, we can do this like so:

```python
rule fastp:
    input:
        R1 = <R1>,
        R2 = <R2>
    output:
        R1 = <R1>,
        R2 = <R2>
    threads: 8
    log: 
        stdout = "results/{sample}/logs/fastp_stdout.log",
        stderr = "results/{sample}/logs/fastp_stderr.log"
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2} \
        --thread {threads} \
        > {log.stdout} 2> {log.stderr}
        """
```

In the above example, we log the stdout and stderr of the `fastp` rule to the `fastp_stdout.log` and `fastp_stderr.log` files respectively. The `>` and `2>` redirects the stdout and stderr to the specified files. 


## Cluster configuration
When running snakemake on a cluster, we also need to specify the resources each rule needs. Think of resources as the exact same as when you request resources on the slurm cluster.
For example:
    
```bash
salloc --ntasks 1 --cpus-per-task 10 --mem 30G --time 200
```

Here we request 1 task, 10 cpus, 30GB of memory, and 200 minutes of runtime. If we want to run snakemake on the cluster, we need to specify these resources for each rule. We can do this by adding the `resources` keyword to the rule. 


### Memory
To add memory to the `fastp` rule, we can do this like so:

```python
rule fastp:
    input:
        R1 = <R1>,
        R2 = <R2>
    output:
        R1 = <R1>,
        R2 = <R2>
    threads: 8
    resources:
        mem = "16GB"
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2} \
        --thread {threads}
        """
```

Here we request 16GB of memory for the `fastp` rule. 


### Time
Another requirement for slurm is the time. We can add this to the `fastp` rule like so:

```python
rule fastp:
    input:
        R1 = <R1>,
        R2 = <R2>
    output:
        R1 = <R1>,
        R2 = <R2>
    threads: 8
    resources:
        mem = "16GB",
        time = "02:00:00"
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2} \
        --thread {threads}
        """
```

Here we request 2 hours of runtime for the `fastp` rule.

### Partition
We can also specify the partition to run on. This is useful because we have multiple partitions on the cluster. We can add this to the `fastp` rule like so:

```python
rule fastp:
    input:
        R1 = <R1>,
        R2 = <R2>
    output:
        R1 = <R1>,
        R2 = <R2>
    threads: 8
    resources:
        mem = "16GB",
        time = "02:00:00",
        partition = "general"
    shell:
        """
        fastp \
        --in1 {input.R1} --in2 {input.R2} \
        --out1 {output.R1} --out2 {output.R2} \
        --thread {threads}
        """
```

If the rule requires a gpu, just add `partition = "gpu"` to the `resources` keyword.


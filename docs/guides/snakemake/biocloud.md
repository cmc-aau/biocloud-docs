# Running Snakemake workflows on SLURM clusters
Please first ensure you understand the basics of [submitting SLURM jobs](../../slurm/jobsubmission.md) before running Snakemake workflows (or anything else!) on the BioCloud. A workflow can either run **within a single SLURM job**, or in **cluster mode**, where Snakemake submits each task as a separate SLURM job.

???+ warning "When to use cluster mode"
    Cluster mode is a great way to give each step exactly the resources it needs, but keep these points in mind:

    - If the workflow spawns hundreds or thousands of jobs, don't use cluster mode. The queue and scheduling overhead will often take longer than the tasks themselves.
    - If all steps are efficient, cluster mode gains you nothing.
    - Cluster mode requires you to test the memory usage of every rule, which can vary with the input data. This is tedious and often not worth it.

## Dry run for inspection
Before running the workflow, inspect the [DAG](tutorial.md#the-directed-acyclic-graph-dag) or perform a "dry run" to list all tasks to be run without running anything:

```
srun --ntasks 1 --cpus-per-task 1 --mem 1G snakemake -n > workflow_dryrun.txt
```

## Running the workflow within a single job
Submit a [batch job](../../slurm/jobsubmission.md#batch-jobs-non-interactive-jobs) with enough resources for the whole workflow, and let Snakemake use them:

```shell
#!/usr/bin/bash -l
#SBATCH --job-name=<snakemake_template>
#SBATCH --output=job_%j_%x.out
#SBATCH --cpus-per-task=32
#SBATCH --mem=64G
#SBATCH --time=1-00:00:00
#SBATCH --mail-type=END,FAIL,TIME_LIMIT_90
#SBATCH --mail-user=abc@bio.aau.dk

set -eu
mamba activate <snakemake_template>

snakemake \
  --cores "$SLURM_CPUS_PER_TASK" \
  --resources mem_mb="$SLURM_MEM_PER_NODE" \
  --keep-going \
  --rerun-incomplete
```

## Cluster mode
Requires Snakemake 8+ and the [SLURM executor plugin](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) (`snakemake-executor-plugin-slurm`, included in the template `environment.yml`). Define `threads`, `mem_mb`, and `runtime` (minutes) for every rule. **Never** set a partition or nodes anywhere, the partition is [selected automatically](../../slurm/partitions.md#automatic-partition-selection).

```shell
#!/usr/bin/bash -l
#SBATCH --job-name=<snakemake_template>
#SBATCH --output=job_%j_%x.out
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=1-00:00:00
#SBATCH --mail-type=END,FAIL,TIME_LIMIT_90
#SBATCH --mail-user=abc@bio.aau.dk

set -eu
mamba activate <snakemake_template>

snakemake \
  --executor slurm \
  --jobs 50 \
  --default-resources mem_mb=1024 runtime=60 \
  --keep-going \
  --rerun-incomplete
```

The warning `You are running snakemake in a SLURM job context` can be ignored. Use `--slurm-efficiency-report` to check the resource usage of each rule.

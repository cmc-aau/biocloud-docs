# Running Snakemake workflows on SLURM clusters
Please first ensure you understand the basics of [submitting SLURM jobs](../../slurm/jobsubmission.md) before running Snakemake workflows (or anything else!) on the BioCloud. It's **highly recommended** that you use a profile like the one [provided below](#biocloud-snakemake-profile) to properly allow Snakemake to start tasks as individual SLURM jobs and **not** run Snakemake itself in a large resource allocation (=job). Snakemake itself hardly requires any resources, 1CPU and 1GB memory is plenty. It's always important when developing Snakemake workflows to make sure that reasonable resource requirements are defined for the individual rules in the workflow (listed under `resources:` and `threads:`). Then it's only a matter of letting Snakemake be aware that it's being used on a HPC cluster and it will do things properly for you.

## Dry run for inspection
Before running the workflow, the [DAG visualization](tutorial.md#the-directed-acyclic-graph-dag) mentioned on the previous page is a very useful way to quickly get an overview of exactly which tasks will be run and the dependencies between them. It can become quite large though, so it can also be useful to perform a "dry run", where Snakemake will output all of the tasks to be run without actually running anything. This can be done in a small [interactive job](../../slurm/jobsubmission.md#interactive-jobs) and the output piped to a file with fx:

```
srun --ntasks 1 --cpus-per-task 1 --mem 1G snakemake -n > workflow_dryrun.txt
```

If you have used the recommended folder structure mentioned earlier, Snakemake will automatically detect and use the `workflow/Snakefile`, but you may have more than one, in which case you must also supply `-s <path to Snakefile>`. 

## Submit the workflow
When you have inspected the DAG or output from the dry run and you are ready to submit the full-scale workflow, you can do this in a [non-interactive SLURM job](../../slurm/jobsubmission.md#non-interactive-jobs) using a batch script like this one:

```shell
#!/usr/bin/bash -l
#SBATCH --job-name=<snakemake_template>
#SBATCH --output=job_%j_%x.out
#SBATCH --cpus-per-task=1
#SBATCH --time=1-00:00:00
#SBATCH --mem=1G
#SBATCH --mail-type=END,FAIL,TIME_LIMIT_90
#SBATCH --mail-user=abc@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# Activate conda environment with only snakemake
mamba activate <snakemake_template>

# Start workflow using resources defined in the profile. Snakemake itself 
# requires nothing, 1CPU + 1G mem is enough

# Render a DAG to visualize the workflow (optional)
snakemake --dag | dot -Tsvg > results/dag.svg

# Main workflow
snakemake --profile biocloud

# Generate a report once finished (optional)
snakemake --report results/report.html
```

From this job Snakemake will submit individual SLURM jobs on your behalf for each task with the resources defined for each rule. This can sometimes start hundreds of jobs (of course within your [limits](../../slurm/accounting.md#qos-info-and-limitations)) depending on the workflow and data - so please **ensure you have defined a reasonable amount of resources for each rule** and that the individual tasks utilize them as close to 100% as possible, especially CPUs. Often resource requirements depend on the exact input data or database being used, so it's also possible to dynamically scale the requirements using simple python lambda functions to calculate appropriate values for each resource, see [Snakemake docs](https://snakemake.readthedocs.io/en/latest/snakefiles/rules.html#dynamic-resources) for details.

## BioCloud Snakemake profile
When using Snakemake to submit SLURM jobs the command with which to start the workflow can easily become quite long (as many as 20+ arguments). Using [Snakemake profiles](https://snakemake.readthedocs.io/en/latest/executing/cli.html#profiles) instead is an easy way to avoid this, where any command line options are simply written to a `config.yaml` file instead, which is then read by Snakemake using the `--profile` argument. On the BioCloud login and compute nodes there is already an optimized default profile pre-installed for all users in `/etc/xdg/snakemake/biocloud/config.yaml`, which will be automatically found by snakemake by name if you just type `snakemake --profile biocloud`. If you need to adjust this profile to suit your needs, you must copy it and place it in either the project folder from where snakemake will be run, or in the user standard location `~/.config/snakemake/biocloud/config.yaml` to avoid copying it around several times for each project. You must supply a name or a path to a folder in which a `config.yaml` is placed, not the path to the file itself. The default profile, which is optimized for our particular setup, is also shown below.

```yaml
#command with which to submit tasks as SLURM jobs
cluster:
  mkdir -p logs/{rule}/ &&
  sbatch
    --parsable
    --qos={resources.qos}
    --cpus-per-task={threads}
    --mem={resources.mem_mb}
    --gpus={resources.gpus}
    --time={resources.runtime}
    --job-name=smk-{rule}-{wildcards}
    --output=logs/{rule}/{rule}-{wildcards}-%j.out
#if rules don't have resources set, use these default values.
#Note that "mem" will be converted to "mem_mb" under the hood, so mem_mb is prefered
default-resources:
  - qos="normal"
  - threads=1
  - mem_mb=512
  - gpus=0
  - runtime="0-01:00:00"
#max threads per job/rule. Will take precedence over anything else. Adjust this
#before submitting to SLURM and leave threads settings elsewhere untouched
max-threads: 32
use-conda: False
use-singularity: False
printshellcmds: False
jobs: 50
local-cores: 1
latency-wait: 120
restart-times: 0
max-jobs-per-second: 10 #don't touch
keep-going: True
rerun-incomplete: True
scheduler: greedy
max-status-checks-per-second: 5
cluster-cancel: scancel
#script to get job status for snakemake, unfortunately neccessary
cluster-status: /etc/xdg/snakemake/biocloud/slurm-status.sh
```

Imagine writing all these settings on the command line every time - just no!

The `slurm-status.sh` script on the last line is available from the template repository [here](https://github.com/cmc-aau/snakemake_project_template/blob/main/extras/slurm-status.sh) or at `/etc/xdg/snakemake/biocloud/slurm-status.sh`.
# Executing snakemake on the bioservers

![snake-slurm-img](media/slurmwsnake.png)

## The basics

The exact way to run snakemake depends on the structure of your workflow. The important thing to know is that snakemake will always work at the directory where you run the command. This means that if you have a directory structure that looks like this:

```
.
pipeline/
    workflow/
        Snakefile
        rules/
            rule1.smk
            rule2.smk
            rule3.smk
data/
    sample1.txt
    sample2.txt
    sample3.txt
    .
    .
    .
    sample1000.txt
```

Then you can run the workflow like this:

```bash
snakemake -s pipeline/workflow/Snakefile --cores <number of cores>
```

Even though the `Snakefile` exists in another folder, snakemake will still work, because it will always work at the directory where you run the command.

## Running snakemake on the bioservers
### Batch job without cluster

```bash
#!/usr/bin/bash -l
#SBATCH --job-name=snakemake-simple
#SBATCH --output=job_%j_%x.out
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --ntasks-per-node=1
#SBATCH --partition=general
#SBATCH --cpus-per-task=<CPUS>
#SBATCH --mem=<MEMORY>
#SBATCH --time=<TIME>
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<EMAIL>@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# load software modules or environments
# activate conda environment
## configure shell
eval "$(conda shell.bash hook)"
conda activate snakemake

# Get number of CPUs from SLURM allocation.
# This only works with single-node jobs
max_threads="$(nproc)"

# run one or more commands as part a full pipeline script or call scripts from elsewhere
snakemake -s pipeline/workflow/Snakefile --cores <CORES> --use-conda 
```

This will execute the pipeline like any other batch script. In this case the pipeline will not take advantage of the SLURM system, but this is often fine for small pipelines.
The important thing to change here is:

- `--cpus-per-task`: The number of CPUs to use. This should be the same as the number of cores you specify in the `snakemake <CORES>` command.
- `--mem`: The memory to use. This should be the expected peak RAM usage of the pipeline.

### Batch job with cluster
To execute snakemake in cluster mode, you need to specify the `--cluster` flag. This is done like so:

```bash
#!/usr/bin/bash -l
#SBATCH --job-name=snakemake-simple
#SBATCH --output=job_%j_%x.out
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --ntasks-per-node=1
#SBATCH --partition=general
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=<TIME>
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<EMAIL>@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# load software modules or environments
# activate conda environment
## configure shell
eval "$(conda shell.bash hook)"
conda activate snakemake

# Get number of CPUs from SLURM allocation.
# This only works with single-node jobs
max_threads="$(nproc)"

# run one or more commands as part a full pipeline script or call scripts from elsewhere
snakemake -s pipeline/workflow/Snakefile --use-conda --jobs 50 \
    --cluster "sbatch --partition={resources.partition} --time={resources.walltime} --mem={resources.mem} --cpus-per-task={threads} --ntasks=1 --output=logs/slurm_output_{rule}_{wildcards.sample}_%j.out" \
    --cluster-cancel "scancel" 
```

The important differences here are:

For the `SBATCH` parameters:

- `--cpus-per-task`: In this case we only use 1 CPU per task. This is because we are running snakemake in cluster mode, which does not require more than 1 CPU per task.
- `--mem`: The memory is set to 1G. This is because snakemake will automatically request the memory needed for each rule.

For the `snakemake` command:

- `--jobs`: This specifies the number of jobs to run in parallel. In this case we run 50 job at a time. 
- `--cluster`: This specifies the command to run on the cluster. This is the command that will be run for each job. The important thing here is that we use the `--partition`, `--time`, `--mem`, `--cpus-per-task` and `--output` flags. These are the flags that snakemake will populate with the correct values for each rule. This is how snakemake knows how much memory to request for each rule.
- `--cluster-cancel`: This specifies the command to cancel a job on the cluster. **It doesn't always work as intended..**


## Useful commands
### Dry run
Before running a workflow, it is a good idea to do a dry run. This will show you what snakemake will do, without actually doing anything. To do a dry run, simply run the following command:

```bash
snakemake -s <path to Snakefile> --dryrun
```

### rerun failed jobs
If a job fails, you can rerun it like so:

```bash
snakemake -s <path to Snakefile> --rerun-incomplete
```

### Unlocking a killed workflow
If a workflow is killed, snakemake will still think a job is running. This will prevent you from restarting the workflow. To unlock the workflow, simply run the following command:

```bash
snakemake -s <path to Snakefile> --unlock
```

### Keeping temporary files
If you want to keep the temporary files generated by snakemake, you can do so by adding the `--notemp` flag. This will prevent snakemake from deleting the temporary files.

```bash
snakemake -s <path to Snakefile> --notemp
```

### Keep going
Sometimes a rule might fail for various reasons - maybe the rule exceeded the available RAM. When one job fails snakemake will by default stop. This is 
# TLDR
Here's a quick "too long, didn't read" guide for those who are too scared to read all the details and just want to get started quickly. There is nothing here you don't need to know!

## How to submit a SLURM job
Start with [logging in](../access/ssh.md) to one of the login nodes through SSH.

### Interactive
If you just need an interactive shell to experiment with some quick commands use `salloc`, for example:
```
salloc --cpus-per-task 2 --mem 4G --time 0-3:00:00
```

This will start a shell on a compute node within a SLURM allocation (here 2 CPUs, 4GB memory for 3 hours), so that you don't run anything on the login node itself.

!!! Important
    **Do NOT** run anything on the login nodes except SLURM commands to submit jobs! You can edit code, browse/move files around, install software, etc, but nothing else. Everything else should be run on a dedicated compute node through SLURM allocations using `sbatch` or `salloc`. If you run things on the login nodes, people can be blocked from doing anything at all if the login nodes crash, because they are (deliberately) quite small.

### Non-interactive
For larger jobs that will run for several hours or days, you need to submit SLURM batch jobs using `sbatch`:

 - Write a shell script (using `nano` or whatever editor you want) named for example `submit.sh` which contains the command(s) you need to run, and let SLURM know how many resources your job needs by filling in the `#SBATCH` comments ([more options here](jobsubmission.md#most-essential-options)) at the top, for example:

```
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.out
#SBATCH --partition=shared
#SBATCH --cpus-per-task=10
#SBATCH --mem=10G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=START,END,FAIL
#SBATCH --mail-user=abc@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# load software modules or environments
module load minimap2

# Get number of CPUs from the SLURM allocation (SLURM will sometimes 
# give you more than what you have requested to optimize efficiency).
# This only works with single-node jobs
max_threads="$(nproc)"

# run one or more commands as part a full pipeline script or call scripts from elsewhere
minimap2 -t "$max_threads" database.fastq input.fastq > out.file
```
This will request 10 CPUs and 10GB memory for a maximum of 2 days on one of the compute nodes within the `shared` compute node partition (see [hardware overview](../index.md#slurm-partitions)). If you need to use a GPU details are [here](jobsubmission.md#requesting-one-or-more-gpus).

 - Submit the job to the queue by typing the command `sbatch submit.sh`
 - Check the job status using `squeue --me` (or the convenient shorter alias `sq`). The job will start once a compute node has enough available resources
 - Once the job starts, you can follow the output from the command(s) by inspecting the log file using for example `tail -F job_xxx.out` (the job will run in the current working directory if not configured otherwise)
 - If needed cancel the job using `scancel <jobid>`

## Optimize efficiency for next time
The job allocation is **entirely yours**, which means you cannot affect other people's jobs in any way, neither can they affect yours. But this also means that you don't want to waste resources which other users could have used instead, especially CPUs (=time). How many resources your job needs is trial and error, but ideally what you request should be utilized to the greatest extend possible. Therefore:

!!! warning "Always inspect and optimize efficiency for next time!"

    When the job completes or fails, **!!!ALWAYS!!!** inspect the CPU and memory usage of the job in either the notification email received or using [these commands](accounting.md#job-efficiency-summary) and adjust the next job accordingly! This is essential to avoid wasting resources which other people could have used.

## Choosing the right compute node partition
If the CPU and memory efficiency/usage of a job was >75%, you are allowed to submit to other partitions (`general`, or `high-mem` if you need lots of memory, see [hardware overview](partitions.md)) than the `shared` partition, which can potentially get things done quicker - a win for everyone. But if not, you will waste resources which could have been used by others. The `shared` partition has less memory available per CPU, however the physical CPUs are shared across jobs to help keep them as busy as possible at all times. The allocated amount of memory will always be yours and yours alone, on the other hand. Many processes will not use all CPUs available for the full duration. This depends heavily on the particular software/tools in use and how they are implemented, as well as the actual input data. Sometimes there is just nothing you can do, but often there is. More details [here](jobsubmission.md#how-many-resources-should-i-request-for-my-jobs). 

# Job control and usage accounting
Below are some nice to know commands for controlling and checking up on running jobs, current and past.

## Get job status info
Use [`squeue`](https://slurm.schedmd.com/squeue.html), for example:
```
$ squeue
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   24 biocloud- interact ksa@bio.  R       2:15      1 bio-oscloud04
```

??? "Job state codes (ST)"
      | Status	Code | Explaination |
      | --- | --- |
      | COMPLETED | CD | The job has completed successfully. |
      | COMPLETING | CG | The job is finishing but some processes are still active. |
      | FAILED | F | The job terminated with a non-zero exit code and failed to execute. |
      | PENDING | PD | The job is waiting for resource allocation. It will eventually run. |
      | PREEMPTED | PR | The job was terminated because of preemption by another job. |
      | RUNNING | R | The job currently is allocated to a node and is running. |
      | SUSPENDED | S | A running job has been stopped with its cores released to other jobs. |
      | STOPPED | ST | A running job has been stopped with its cores retained. |

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/squeue.html#lbAG)

??? "Job reason codes (REASON )"
      | Reason Code | Explaination |
      | --- | --- |
      | Priority | One or more higher priority jobs is in queue for running. Your job will eventually run. |
      | Dependency | This job is waiting for a dependent job to complete and will run afterwards. |
      | Resources | The job is waiting for resources to become available and will eventually run. |
      | InvalidAccount | The job’s account is invalid. Cancel the job and rerun with correct account. |
      | InvaldQoS | The job’s QoS is invalid. Cancel the job and rerun with correct account. |
      | QOSGrpCpuLimit | All CPUs assigned to your job’s specified QoS are in use; job will run eventually. |
      | QOSGrpMaxJobsLimit | Maximum number of jobs for your job’s QoS have been met; job will run eventually. |
      | QOSGrpNodeLimit | All nodes assigned to your job’s specified QoS are in use; job will run eventually. |
      | PartitionCpuLimit | All CPUs assigned to your job’s specified partition are in use; job will run eventually. |
      | PartitionMaxJobsLimit | Maximum number of jobs for your job’s partition have been met; job will run eventually. |
      | PartitionNodeLimit | All nodes assigned to your job’s specified partition are in use; job will run eventually. |
      | AssociationCpuLimit | All CPUs assigned to your job’s specified association are in use; job will run eventually. |
      | AssociationMaxJobsLimit | Maximum number of jobs for your job’s association have been met; job will run eventually. |
      | AssociationNodeLimit | All nodes assigned to your job’s specified association are in use; job will run eventually. |

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/squeue.html#lbAF)

## Cancel a job
With `sbatch` you won't be able to just hit CTRL+c to stop what's running like you're used to in a terminal. Instead you must use `scancel`. Get the job ID from `squeue -u $(whoami)`, then use [`scancel`](https://slurm.schedmd.com/scancel.html) to cancel a running job, for example:
```
$ scancel 24
```

If the particular job doesn't stop and doesn't respond, consider using [`skill`](https://slurm.schedmd.com/skill.html) instead.

## Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/scontrol.html) to control your own jobs, for example suspend a running job:
```
$ scontrol suspend 24
```

Resume again with
```
$ scontrol resume 24
```

## Adjust allocated ressources
It's also possible to adjust allocated ressources to free them up for others to use without having to stop anything, for example:
```
$ scontrol update JobId=24 NumNodes=1 NumTasks=1 CPUsPerTask=1
```

## Job status information
Use [`sstat`](https://slurm.schedmd.com/sstat.html) to show the status and usage accounting information of running jobs.
```
$ sstat
```

With additional details:
```
sstat --jobs=your_job-id --format=jobid,avecpu,maxrss,ntasks
```

??? "Useful format variables"
      | Variable | Description |
      | --- | --- |
      | avecpu | Average CPU time of all tasks in job. |
      | averss | Average resident set size of all tasks. |
      | avevmsize | Average virtual memory of all tasks in a job. |
      | jobid | The id of the Job. |
      | maxrss | Maximum number of bytes read by all tasks in the job. |
      | maxvsize | Maximum number of bytes written by all tasks in the job. |
      | ntasks | Number of tasks in a job. |
      
      For all variables see the [SLURM documentation](https://slurm.schedmd.com/sstat.html#SECTION_Job-Status-Fields)

## Job usage accounting
To view the status of past jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/sacct.html).
```
$ sacct
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
7            biobank_d+ biocloud-+ compute-a+        180  COMPLETED      0:0 
7.batch           batch            compute-a+        180  COMPLETED      0:0 
7.extern         extern            compute-a+        180  COMPLETED      0:0 
8            interacti+ biocloud-+ compute-a+          1     FAILED      2:0 
8.extern         extern            compute-a+          1  COMPLETED      0:0 
9            interacti+ biocloud-+ compute-a+          1  COMPLETED      0:0 
9.extern         extern            compute-a+          1  COMPLETED      0:0
```

There are a huge number of other options to show, see [SLURM docs](https://slurm.schedmd.com/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use `sacct --long > file.txt` and dump it into a file or else it's too much for the terminal.
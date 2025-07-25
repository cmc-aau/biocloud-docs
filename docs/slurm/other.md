# Other commands / FAQ
Below are some nice to know commands with example output and some common problems. This will continously be populated as people ask for certain things. **Your question here!**

## I'm not allowed to submit jobs
```
salloc: error: Job submit/allocate failed: Invalid account or account/partition combination specified
```

This error means you have not been associated with any usage account yet, so you must contact an administrator to add your user to the correct account.

## My job is pending with a "requeued held" status
This means something went wrong when the job started on a compute node, so the job went back into the queue to avoid draining the node. It will stay in this state forever until manually restarted. Try running a `scontrol release <job_id>` and if that doesn't work contact an administrator.

## Show busy/free cores for the entire cluster
Example output (A=allocated, I=idle, O=other, T=total):
```
$ sinfo -o "%C"
CPUS(A/I/O/T)
620/596/240/1456
```

## Show current reservations
Reservations will also be used for scheduled maintenance, so that SLURM simply won't allow any jobs to start if they have a `time` limit set that spans into the reservation.
```
$ sinfo -T
RESV_NAME       STATE           START_TIME             END_TIME     DURATION  NODELIST
maintenance  INACTIVE  2023-12-18T23:00:00  2023-12-20T01:00:00   1-02:00:00  bio-oscloud[02-09]
```

For more details about one or all reservations use
```
$ scontrol show reservations
ReservationName=amplicon StartTime=2024-11-04T08:00:00 EndTime=2024-11-18T08:00:00 Duration=14-00:00:00
   Nodes=bio-oscloud03 NodeCnt=1 CoreCnt=192 Features=(null) PartitionName=general Flags=
     NodeName=bio-oscloud03 CoreIDs=0-191
   TRES=cpu=192
   Users=abc@bio.aau.dk Groups=(null) Accounts=(null) Licenses=(null) State=ACTIVE BurstBuffer=(null) Watts=n/a
   MaxStartDelay=(null)
```

## Show details about the whole cluster configuration
```
$ scontrol show config
```

## SLURM environment variables
SLURM jobs will have a variety of environment variables set within job allocations, which might become handy programmatically. Below is an overview of relevant ones. For all of them refer to the SLURM documentation for [input environment variables](https://slurm.schedmd.com/archive/slurm-24.11.4/sbatch.html#SECTION_INPUT-ENVIRONMENT-VARIABLES) and [output environment variables](https://slurm.schedmd.com/archive/slurm-24.11.4/sbatch.html#SECTION_OUTPUT-ENVIRONMENT-VARIABLES). Some may not be present for your particular job, so to list only those currently available within a job run for example `env | grep -iE 'SLURM|SBATCH'`.

| Variable(s) | Description |
| :--- | :--- |
| `SLURM_ARRAY_TASK_COUNT` | Total number of tasks in a job array |
| `SLURM_ARRAY_TASK_ID` | Job array ID (index) number |
| `SLURM_ARRAY_TASK_MAX` | Job array's maximum ID (index) number |
| `SLURM_ARRAY_TASK_MIN` | Job array's minimum ID (index) number |
| `SLURM_ARRAY_TASK_STEP` | Job array's index step size |
| `SLURM_ARRAY_JOB_ID` | Job array's master job ID number |
| `SLURM_CLUSTER_NAME` | Name of the cluster on which the job is executing |
| `SLURM_CPUS_ON_NODE` | Number of CPUS on the allocated node |
| `SLURM_CPUS_PER_TASK` | Number of cpus requested per task. Only set if the --cpus-per-task option is specified. |
| `SLURM_JOB_ACCOUNT` | Account name associated of the job allocation |
| `SLURM_JOBID`, `SLURM_JOB_ID` | The ID of the job allocation |
| `SLURM_JOB_CPUS_PER_NODE` | Count of processors available to the job on this  |node.
| `SLURM_JOB_DEPENDENCY` | Set to value of the --dependency option |
| `SLURM_JOB_NAME` | Name of the job |
| `SLURM_NODELIST`, `SLURM_JOB_NODELIST` | List of nodes allocated to the job |
| `SLURM_NNODES`, `SLURM_JOB_NUM_NODES` | Total number of different nodes in the job's resource allocation |
| `SLURM_MEM_PER_NODE` | Takes the value of --mem if this option was specified. |
| `SLURM_MEM_PER_CPU` | Takes the value of --mem-per-cpu if this option was specified. |
| `SLURM_NTASKS`, `SLURM_NPROCS` | Same as -n or --ntasks if either of these options was specified. |
| `SLURM_NTASKS_PER_NODE` | Number of tasks requested per node. Only set if the --ntasks-per-node option is specified. |
| `SLURM_NTASKS_PER_SOCKET` | Number of tasks requested per socket. Only set if the --ntasks-per-socket option is specified. |
| `SLURM_SUBMIT_DIR` | The directory from which sbatch was invoked |
| `SLURM_SUBMIT_HOST` | The hostname of the computer from which sbatch was invoked |
| `SLURM_TASK_PID` | The process ID of the task being started |
| `SLURMD_NODENAME` | Name of the node running the job script |
| `SLURM_JOB_GPUS` | GPU IDs allocated to the job (if any). |



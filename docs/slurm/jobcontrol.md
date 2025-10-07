# Job control and useful commands
Below are some nice-to-know commands for controlling and checking up on running or queued jobs.

## Overall cluster status
This will normally show some colored bars for each partition, which unfortunately doesn't render here.
```
Cluster allocation summary per partition or individual nodes (-n).
(Numbers are reported in free/allocated/total(OS factor)).

Partition    |                CPUs                 |           Memory (GB)           |       GPUs        |
========================================================================================================
shared       | 1436 196                 /1632 (3x) | 2091 268                 /2359  |           
general      |  395 373                 /768       | 2970 731                 /3701  |           
high-mem     |  233 199                 /432       | 1803 1936                /3739  |           
gpu          |   24 40                  /64        |   29 180                 /209   |    1 1         /2    
--------------------------------------------------------------------------------------------------------
Total:       | 2088 808                 /2896      | 6894 3115                /10009 |    1 1         /2    

Jobs running/pending/total:
  26 / 1 / 27

Use sinfo or squeue to obtain more details.

```

## Get job status info
Use [`squeue`](https://slurm.schedmd.com/archive/slurm-24.11.4/squeue.html), for example:
```
$ squeue
squeue
      JOBID             NAME       USER ACCOUNT        TIME   TIME_LEFT CPU MIN_ME ST PRIO  PARTITION NODELIST(REASON)
    1275175    RStudioServer user01@bio     acc1       0:00  3-00:00:00  32     5G PD    4    general (QOSMaxCpuPerUserLimit)
    1275180       sshdbridge user02@bio     acc2       7:14     7:52:46   8    40G  R    6    general bio-oscloud03
    1275170   VirtualDesktop user03@bio     acc2      35:54     5:24:06   2    10G  R    6    general bio-oscloud05
```

To show only your own jobs use `squeue --me`. This is used quite often so `sq` has been made an alias of `squeue --me`. You can for example also append `--partition`, `--nodelist`, `--reservation`, and more, to only show the queue for those select partitions, nodes, or reservations.

You can also get an estimated start time for pending jobs by using `squeue --start`. Jobs will in most cases start earlier than this time, as the calculation is based on the time limit set for current running jobs, and most finish in time. They can only start later if new jobs with a higher priority get submitted to the queue before they start:

```
$ squeue --start
    JOBID PARTITION     NAME     USER ST          START_TIME  NODES SCHEDNODES           NODELIST(REASON)
  1306529   general smk-mapp user01@b PD 2025-01-24T12:25:15      1 bio-oscloud02        (Resources)
  1306530   general smk-mapp user01@b PD 2025-01-24T12:25:15      1 bio-oscloud04        (Priority)
  1309386   general cq_EV_me user02@b PD 2025-01-24T12:25:15      1 bio-oscloud05        (Priority)
  1306531   general smk-mapp user01@b PD 2025-01-24T12:27:02      1 bio-oscloud03        (Priority)
  1306532   general smk-mapp user01@b PD 2025-01-25T14:46:56      1 (null)               (Priority)
  1306533   general smk-mapp user01@b PD 2025-01-25T14:46:56      1 (null)               (Priority)
...
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

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/archive/slurm-24.11.4/squeue.html#lbAG)

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

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/archive/slurm-24.11.4/squeue.html#lbAF)

The columns to show can be customized using the `--format` option, but can also be set with the environment variable `SQUEUE_FORMAT` to avoid typing it every time. You can always override this to suit your needs in your `.bashrc` file. The default format is currently:

```
SQUEUE_FORMAT="%.12i %.16j %.10u %.10M %.12L %.3C %.6m %.2t %.9P %R"
```

See a full list [here](https://slurm.schedmd.com/archive/slurm-24.11.4/squeue.html#OPT_format).

## Prevent pending job from starting
Pending jobs can be marked in a "hold" state to prevent them from starting
```
scontrol hold <job_id>
```

To release a queued job from the ‘hold’ or 'requeued held' states:
```
scontrol release <job_id>
```

To cancel and rerun (requeue) a particular job:
```
scontrol requeue <job_id>
```

## Cancel a job
With `sbatch` you won't be able to just hit CTRL+c to stop what's running like you're used to in a terminal. Instead you must use `scancel`. Get the job ID from `squeue --me`, then use [`scancel`](https://slurm.schedmd.com/archive/slurm-24.11.4/scancel.html) to cancel a running job, for example:
```
$ scancel <job_id>

# cancel ALL your jobs
$ scancel --me
```

If the particular job doesn't stop and doesn't respond, consider using [`skill`](https://slurm.schedmd.com/archive/slurm-24.11.4/skill.html) instead.

## Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/archive/slurm-24.11.4/scontrol.html) to control your own jobs, for example suspend a running job:
```
$ scontrol suspend <job_id>
```

Resume again with
```
$ scontrol resume <job_id>
```

## Show details about a running or queued job
```
scontrol show jobid=<jobid>
```

If needed, you can also obtain the batch script used to submit a job:
```
scontrol write batch_script <jobid>
```

## Modifying job attributes
Only a few job attributes can be changed after a job is submitted and **NOT** running yet. These attributes include:

 - time limit
 - job name
 - job dependency
 - partition or QOS
 - nice value

For example:
```
$ scontrol update JobId=<jobid> timelimit=<new timelimit>
$ scontrol update JobId=<jobid> partition=high-mem
```

If the job is already running, adjusting the time limit must be done by an administrator.

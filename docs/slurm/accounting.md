# Usage accounting and priority
All users belong to an account (usually their PI) where all usage is tracked on a per-user basis, which directly impact the priority of future jobs. Additionally, limitations and extra high priorities can be obtained by submitting jobs to different SLURM "Quality of Service"s (QOS). By default, all users are only allowed to submit jobs to the `normal` QOS with equal resource limits and base priority for everyone. Periodically users may submit to the `fastq` QOS instead, which has higher resource [usage limits](#usage-limits-and-qos) and a base priority higher than everyone else (and therefore the usage is also billed 3x), however this must first be discussed among the owners of the hardware (PI's), and then you can contact an administrator to grant your user permission to submit jobs to it for a limited period of time.

## Job scheduling
BioCloud uses the first-in-first-out (FIFO) scheduling algorithm with **backfilling enabled**, which means that occasionally lower priority jobs may start before higher priority jobs to maximize the utilization of the cluster, since smaller (usually shorter) jobs may be able to finish before the larger jobs are scheduled to start. To increase the chance of backfilling, it is therefore important to set a realistic timelimit for your jobs that is as short as possible, but long enough for it to finish.

![FIFO-Backfill](img/fifo-backfill.svg)

???+ info "Plan ahead - Make reservations!"
    To avoid any queue time altogether, resources can be reserved in advance for a user or account to ensure that jobs will start immediately when submitted, even if the partition or cluster is fully utilized. Data processing- and analysis time should already be an integral part of your project planning, so you might as well make a corresponding SLURM reservation too to avoid delays. Utilizing reservations can also help distribute the cluster workload over time more evenly, which is beneficial for everyone. There are several different types of reservations. They can be made by contacting an administrator.

## Job priority
When a job is submitted a priority value between 1-1000 is calculated based on several factors, where a higher number indicates a higher priority in the queue. This does not impact running jobs, and the effect of prioritization is only noticable when the cluster is operating near peak capacity, or when the compute node partition to which the job has been submitted is fully allocated by running jobs. Otherwise jobs will usually start immediately as long as there are resources available and you haven't reached any [usage limits](#usage-limits-and-qos).

Different weights are given to the individual priority factors, where the most significant ones are the account [fair-share factor](#the-fair-share-factor) and the [QOS](#usage-limits-and-qos). All factors are normalized to a value between 0-1, then weighted by an adjustable scalar, which may be adjusted be occasionally by an administrator depending on the overall cluster usage over time. Users can also be nice to other users and reduce the priority of their own jobs by setting a "nice" value using `--nice` when submitting for example less time-critical jobs. Job priorities are then calculated according to the following formula:

```
Job_priority =
	(PriorityWeightQOS) * (QOS_factor)
	(PriorityWeightAge) * (age_factor) +
	(PriorityWeightFairshare) * (fair-share_factor) +
	(PriorityWeightJobSize) * (job_size_factor) -
	- nice_factor
```

To obtain the current configured weights use `sprio -w`:
```
$ sprio -w
  JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
Weights                               1        100        800       904        1000
```

The priority of pending jobs will be shown in the job queue when running `squeue`. To see the exact contributions of each factor to the priority of a pending job use `sprio -j <jobid>`:
```
$ sprio -j 1282256
  JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
1282256 general          586          0        129        619        138          0
```

The job **age** and **size** factors are important to avoid the situation where large jobs can get stuck in the queue for a long time because smaller jobs will always fit in everywhere much more easily. The age factor will max out to `1.0` when 3 days of queue time has been accrued for any job. The job size factor is directly proportional to the number of CPUs requested, regardless of the time limit, and is normalized to the total number of CPUs in the cluster. Therefore the weight of it will always be configured to be equal to the total number of (physical) CPUs available in the cluster.

### The fair-share factor
As the name implies, the fair-share factor is used to ensure that users within each account have their fair share of computing resources made available to them over time. Because the individual research groups have contributed with different amounts of hardware to the cluster, the overall share of computing resources made available to them should match accordingly. Secondly, the resource usage of individual users within each account is important to consider as well, so that users who may recently have vastly overused their shares within each account will not have the highest priority. The goal of the fair-share factor is to balance the usage of all users by adjusting job priorities, so that it's possible for everyone to use their fair share of computing resources over time. The fair-share factor is calculated according to the [fair-tree algorithm](https://slurm.schedmd.com/archive/slurm-24.11.4/fair_tree.html), which is an integrated part of the SLURM scheduler. At the time of writing, it has been configured with a **usage decay half-life of 2 weeks**, and the usage data is never reset.

To see the current fair-share factor for your user and the amount of shares available for each account, you can run `sshare`:

```
$ sshare
Account                    User  RawShares  NormShares    RawUsage  EffectvUsage  FairShare 
-------------------- ---------- ---------- ----------- ----------- ------------- ---------- 
root                                          0.000000  2113260049      0.000000            
 ao                                  20000    0.007035      678087      0.000321            
 cms                                134195    0.047205           0      0.000000            
 jln                                464042    0.163233   159497849      0.075475            
 kln                                 20000    0.007035   127893108      0.060520            
 kt                                 136788    0.048117   620619813      0.293680            
 ma                                 747468    0.262932   547028231      0.258856            
 md                                 321172    0.112977    38170820      0.018063            
 ml                                  20000    0.007035     1786141      0.000845            
 mms                                136788    0.048117   205651655      0.097312            
 mrr                                145084    0.051035           1      0.000000            
 mto                                 20000    0.007035           0      0.000000            
 ndj                                 20000    0.007035     2512949      0.001189            
 pc                                 290168    0.102070           0      0.000000            
 phn                                214114    0.075317   406460415      0.192339            
  phn                abc@bio.a+          1    0.062500     7130776      0.017544   0.072072 
 pk                                  20000    0.007035           0      0.000000            
 rw                                  20000    0.007035       28203      0.000013            
 sb                                  20000    0.007035      660663      0.000313            
 sss                                 33000    0.011608      527458      0.000250            
 students                            20000    0.007035      128587      0.000061            
 tnk                                 20000    0.007035           0      0.000000            
 ts                                  20000    0.007035       45927      0.000022            
```

  - `RawShares`: the amount of "shares" assigned to each account (in our setup simply the number of CPUs each account has contributed with)
  - `NormShares`: the fraction of shares given to each account normalized to the total shares available across all accounts, e.g. a value of 0.33 means an account has been assigned 33% of all the resources available in the cluster.
  - `RawUsage`: usage of all jobs charged to the account or user. The value will decay over time depending on the usage decay half-life configured. The `RawUsage` for an account is the sum of the `RawUsage` for each user within the account, thus indicative of which users have contributed the most to the account’s overall score.
  - `EffectvUsage`: `RawUsage` divided by the **total** `RawUsage` for the cluster, hence the column always sums to `1.0`. `EffectvUsage` is therefore the percentage of the total cluster usage the account has actually used (in relation to the total usage, NOT the total capacity). In the example above, the `ma` account has used `26.5%` of the total cluster usage since the last usage reset, if any.
  - `FairShare`: The fair-share score calculated using the following formula `FS = 2^(-EffectvUsage/NormShares)`. The `FairShare` score can be interpreted by the following intervals: 
    - 1.0: **Unused**. The account has not run any jobs since the last usage reset, if any.
    - 0.5 - 1.0: **Underutilization**. The account is underutilizing their granted share. For example a value of 0.75 means the account has underutilized their share 1:2
    - 0.5: **Average utilization**. The account on average is using exactly as much as their granted share.
    - 0.0 - 0.5: **Over-utilization**. The account is overusing their granted share. For example a value of 0.75 means the account has recently overutilized their share 2:1
    - 0: No share left. The account is vastly overusing their granted share and users will get the lowest possible priority.

For more details about job prioritization see the [SLURM documentation](https://slurm.schedmd.com/archive/slurm-24.11.4/priority_multifactor.html) and this [presentation](https://slurm.schedmd.com/SLUG19/Priority_and_Fair_Trees.pdf).

## Usage limits and QOS
Usage limits are set at the QOS level, where the QOS's listed below are present in the cluster. The `normal` QOS is the default, and the `fastq` QOS is only available to users who have been granted permission by an administrator for a limited period of time to get a higher priority than everyone else and higher usage limits. The `interactive` QOS is only available on the `interactive` partition and will be enforced on all jobs submitted to the [`interactive` partition](partitions.md#the-interactive-partition).

Please note that the following limits may occasionally be adjusted without further notice to balance usage, so this table may not always be completely up to date. You can always use the `sacctmgr show qos` command to see the current limits for each QOS.

| | `interactive` | `normal` | `fastq` |
| ---: | :---: | :---: | :---: |
| [UsageFactor](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_UsageFactor) <br> *Usage accounting will be multiplied by this value* | 1.0 | 1.0 | 3.0 |
| [Priority](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_Priority_2) <br> *Add this number to the calculated [job priority](#job-priority)* | | | +1000 |
| [MinPrioThres](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MinPrioThreshold) <br> *Minimum priority required for the job to be scheduled* | | 100 |  |
| [MaxTRESPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESPerUser) <br> *Maximum number of [TRES](https://slurm.schedmd.com/archive/slurm-24.11.4/tres.html) (trackable resources) each* **user** *is able to use at once* | cpu=32,mem=128G | cpu=576 |  |
| [MaxJobsPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxJobsPerUser) <br> *Maximum number of jobs each* **user** *is able to have running at once* | 10 | 200 | 500 |
| [MaxSubmitPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxSubmitJobsPerUser) <br> *Maximum number of jobs each* **user** *is able to submit at once (running+pending)* | 20 | 500 | 1000 |
| [MaxTRESPA](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESPerAccount) <br> *Maximum number of [TRES](https://slurm.schedmd.com/archive/slurm-24.11.4/tres.html) (trackable resources) each* **account** *is able to use at once* | | cpu=1728 | cpu=1728 |
| [MaxTRESRunMinsPA](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESRunMinsPerAccount) <br> *Maximum number of TRES\*minutes each* **account** *is able to have running at once (e.g. 100CPU's for 1 hour corresponds to 6000 CPU minutes)* | | cpu=17500000 | cpu=30000000 |
| [MaxTRESRunMinsPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESRunMinsPerUser) <br> *Maximum number of TRES\*minutes each* **user** *is able to have running at once (e.g. 100CPU's for 1 hour corresponds to 6000 CPU minutes)* | | cpu=6000000 |  |

To see details about account associations, allowed QOS's, limits set at the user level, and more, for your user, use the following command:
```
# your user
$ sacctmgr show user withassoc where name=$USER
      User   Def Acct     Admin    Cluster    Account  Partition     Share   Priority MaxJobs MaxNodes  MaxCPUs MaxSubmit     MaxWall  MaxCPUMins                  QOS   Def QOS
---------- ---------- --------- ---------- ---------- ---------- --------- ---------- ------- -------- -------- --------- ----------- ----------- -------------------- ---------
abc@bio.a+       root      None   biocloud       root                    1          1                                                                  fastq,normal    normal

# all users
$ sacctmgr show user withassoc | less
```

### Undergraduate students
Undergraduate students (up to but NOT including master students) share resources within the `students` account and only their combined usage is limited. View current limitations with for example:

```
$ sacctmgr list account students -s format="account,priority,MaxCPUs,grpcpus,qos" | head -n 3
   Account   Priority  MaxCPUs  GrpCPUs                  QOS 
---------- ---------- -------- -------- -------------------- 
  students                          192               normal
```

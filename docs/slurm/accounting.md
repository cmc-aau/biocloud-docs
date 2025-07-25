# Usage accounting and priority
All users belong to an account (usually their PI) where all usage is tracked on a per-user basis, which directly impact the priority of future jobs. Additionally, limitations and extra high priorities can be obtained by submitting jobs to different SLURM "Quality of Service"s (QOS). By default, all users are only allowed to submit jobs to the `normal` QOS with equal resource limits and base priority for everyone. Periodically users may submit to the `highprio` QOS instead, which has higher resource [usage limits](#usage-limits-and-qos) and a base priority higher than everyone else (and therefore the usage is also billed 3x), however this must first be discussed among the owners of the hardware (PI's), and then you must contact an administrator to grant your user permission to submit jobs to it for a period of time.

## Job scheduling
BioCloud uses the first-in-first-out (FIFO) scheduling algorithm with **backfilling enabled**, which means that occasionally lower priority jobs may start before higher priority jobs to maximize the utilization of the cluster, since smaller (usually shorter) jobs may be able to finish before the larger jobs are scheduled to start. To increase the chance of backfilling, it is therefore important to set a realistic timelimit for your jobs that is as short as possible, but long enough for it to finish.

![FIFO-Backfill](img/fifo-backfill.svg)

## Job priority
When a job is submitted a priority value is calculated based on several factors, where a higher number indicates a higher priority in the queue. This does not impact running jobs, and the effect of prioritization is only noticable when the cluster is operating near peak capacity, or when the compute node partition to which the job has been submitted is fully allocated by running jobs. Otherwise jobs will usually start immediately as long as there are resources available and you haven't reached the [maximum CPU's per user limit](#qos-info-and-limitations), if any.

Different weights are given to the individual priority factors, where the most significant ones are the account fair-share factor (described in more detail below) and the QOS, as described above. All factors are normalized to a value between 0-1, then weighted by an adjustable scalar, which may be adjusted occasionally depending on the overall cluster usage. Users can also be nice to other users and reduce the priority of their own jobs by setting a "nice" value using `--nice` when submitting for example less time-critical jobs. Job priorities are then calculated according to the following formula:

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
As the name implies, the fair-share factor is used to ensure that users within each account have their fair share of computing resources made available to them over time. Because the individual research groups have contributed with different amounts of hardware to the cluster, the overall share of computing resources made available to them should match accordingly. Secondly, the resource usage of individual users within each account is important to consider as well, so that users who may recently have vastly overused their shares within each account will not have the highest priority. The goal of the fair-share factor is to balance the usage of all users by adjusting job priorities, so that it's possible for everyone to use their fair share of computing resources over time. The fair-share factor is calculated according to the [fair-tree algorithm](https://slurm.schedmd.com/archive/slurm-24.11.4/fair_tree.html), which is an integrated part of the SLURM scheduler. At the time of writing, it has been configured with a usage decay half-life of 2 weeks, and the usage data is never reset.

To see the current fair-share factor for your user and the amount of shares available for each account, you can run `sshare`:

```
$ sshare
Account                    User  RawShares  NormShares    RawUsage  EffectvUsage  FairShare 
-------------------- ---------- ---------- ----------- ----------- ------------- ---------- 
root                                          0.000000   699456285      1.000000            
 root                abc@bio.a+          1    0.000548     1136501      0.001625   0.115183 
 ao                                     16    0.008762           0      0.000000            
 jln                                   272    0.148959   148170579      0.211843            
 kln                                    16    0.008762    48720153      0.069656            
 kt                                     48    0.026287    60884827      0.087049            
 ma                                    624    0.341731   185384914      0.265043            
 md                                    259    0.141840    60612473      0.086659            
 ml                                     16    0.008762     2789545      0.003974            
 mms                                    48    0.026287    92148476      0.131738            
 mto                                    16    0.008762           0      0.000000            
 ndj                                    16    0.008762     5181213      0.007408            
 phn                                   381    0.208653    62047356      0.088711            
 pk                                     16    0.008762           1      0.000000            
 rw                                     16    0.008762    18346721      0.026231            
 sss                                    48    0.026287    13392398      0.019147            
 students                               16    0.008762      499066      0.000714            
 ts                                     16    0.008762      142053      0.000203            
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

???+ tip "The fair-share factor and CPU efficiency"
      The value of the fair-share factor is calculated based on CPU usage in units of **allocation seconds** and **not** CPU seconds, which is normally the unit used for CPU usage reported by the `sreport` and `sacct` commands. Therefore, this also means that the CPU efficiency of past jobs directly impacts how much actual work can be done by the allocated CPUs for each user within each account before their fair share of resources is consumed for the period.

For more details about job prioritization see the [SLURM documentation](https://slurm.schedmd.com/archive/slurm-24.11.4/priority_multifactor.html) and this [presentation](https://slurm.schedmd.com/SLUG19/Priority_and_Fair_Trees.pdf).

## Usage limits and QOS
Currently, the following QOS's are available:

| | `normal` | `highprio` |
| ---: | :---: | :---: |
| [UsageFactor](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_UsageFactor) | 1.0 | 3.0 |
| [Priority](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_Priority_2) | +0 | +1000 |
| [MinPrioThres](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MinPrioThreshold) | 100 |  |
| [MaxTRESPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESPerUser) | cpu=576 |  |
| [MaxJobsPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxJobsPerUser) | 100 | 100 |
| [MaxSubmitPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxSubmitJobsPerUser) | 200 | 200 |
| [MaxTRESPA](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESPerAccount) | cpu=1728 | cpu=1728 |
| [MaxTRESRunMinsPA](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESPerAccount) | cpu=17500000 | cpu=30000000 |
| [MaxTRESRunMinsPU](https://slurm.schedmd.com/archive/slurm-24.11.4/sacctmgr.html#OPT_MaxTRESRunMinsPerUser) | cpu=6000000 |  |

To see details about account associations, allowed QOS's, limits set at the user level, and more, for your user, use the following command:
```
# your user
$ sacctmgr show user withassoc where name=$USER
      User   Def Acct     Admin    Cluster    Account  Partition     Share   Priority MaxJobs MaxNodes  MaxCPUs MaxSubmit     MaxWall  MaxCPUMins                  QOS   Def QOS
---------- ---------- --------- ---------- ---------- ---------- --------- ---------- ------- -------- -------- --------- ----------- ----------- -------------------- ---------
abc@bio.a+       root      None   biocloud       root                    1          1                                                                  highprio,normal    normal

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

## Job resource usage summary
### Running jobs
Use [`sstat`](https://slurm.schedmd.com/archive/slurm-24.11.4/sstat.html) to show the status and live usage accounting information of only **running** jobs. For batch scripts you need to add `.batch` to the job ID, for example:
```
$ sstat <job_id>.batch
```

This will print EVERY metric, so it's nice to select only a few most relevant ones, for example:

```
$ sstat --jobs <job_id>.batch --format=jobid,avecpu,maxrss,ntasks
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
      
      For all variables see the [SLURM documentation](https://slurm.schedmd.com/archive/slurm-24.11.4/sstat.html#SECTION_Job-Status-Fields)

### Past jobs
To view the status of **past** jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/archive/slurm-24.11.4/sacct.html). `sacct` will return **everything** accounted for by default which is very inconvenient to view in a terminal window, so the below command will show the most essential information:
```
$ sacct -o jobid,jobname,start,end,NNodes,NCPUS,ReqMem,CPUTime,AveRSS,MaxRSS --user=$USER --units=G -j 138
JobID           JobName               Start                 End   NNodes      NCPUS     ReqMem    CPUTime     AveRSS     MaxRSS 
------------ ---------- ------------------- ------------------- -------- ---------- ---------- ---------- ---------- ---------- 
138          interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16        20G   00:02:56                       
138.interac+ interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56          0          0 
138.extern       extern 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56      0.00G      0.00G 

```

There are a huge number of other options to show, see [SLURM docs](https://slurm.schedmd.com/archive/slurm-24.11.4/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use `sacct --long > file.txt` and dump it into a file or else it's too much for the terminal.

## Reservation usage
Show reservation usage in CPU hours and percent of the reservation total used
```
$ sreport reservation utilization -t hourper
--------------------------------------------------------------------------------
Reservation Utilization 2024-11-04T00:00:00 - 2024-11-04T23:59:59
Usage reported in TRES Hours/Percentage of Total
--------------------------------------------------------------------------------
  Cluster      Name               Start                 End      TRES Name                     Allocated                          Idle 
--------- --------- ------------------- ------------------- -------------- ----------------------------- ----------------------------- 
 biocloud  amplicon 2024-11-04T08:00:00 2024-11-05T15:18:55            cpu                  1154(19.20%)                  4858(80.80%) 
```

## Job efficiency summary
### Individual jobs
To view the efficiencies of individual jobs use `seff`, for example:

```
$ seff 2357
Job ID: 2357
Cluster: biocloud
User/Group: <username>
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 96
CPU Utilized: 60-11:06:29
CPU Efficiency: 45.76% of 132-02:48:00 core-walltime
Job Wall-clock time: 1-09:01:45
Memory Utilized: 383.42 GB
Memory Efficiency: 45.11% of 850.00 GB
```

This information will also be shown in notification emails when jobs finish.

### Multiple jobs
Perhaps a more useful way to use `sacct` is through the [reportseff](https://github.com/troycomi/reportseff) tool (pre-installed), which can be used to calculate the CPU, memory, and time efficiencies of past jobs, so that you can optimize future jobs and ensure resources are utilized to the max (**2TM**). For example:
```
$ reportseff -u $(whoami) --format partition,jobid,state,jobname,alloccpus,reqmem,elapsed,cputime,CPUEff,MemEff,TimeEff -S r,pd,s --since d=4
  Partition     JobID        State                       JobName                    AllocCPUS     ReqMem   Elapsed        CPUTime      CPUEff   MemEff   TimeEff 
    interactive      306282     COMPLETED                     midasok                        2         8G       00:13:07       00:26:14      4.1%    15.4%     10.9%  
   general      306290     COMPLETED           smk-map2db-sample=barcode46             16         24G      00:01:38       00:26:08     90.6%    18.7%     2.7%   
   general      306291     COMPLETED           smk-map2db-sample=barcode47             16         24G      00:02:14       00:35:44     92.8%    18.4%     3.7%   
   general      306292     COMPLETED           smk-map2db-sample=barcode58             16         24G      00:02:32       00:40:32     80.3%    19.0%     4.2%   
   general      306293     COMPLETED           smk-map2db-sample=barcode34             16         24G      00:02:16       00:36:16     78.1%    18.7%     3.8%   
   general      306294     COMPLETED           smk-map2db-sample=barcode22             16         24G      00:02:38       00:42:08     81.1%    19.0%     4.4%   
   general      306295     COMPLETED           smk-map2db-sample=barcode35             16         24G      00:02:26       00:38:56     79.0%    19.0%     4.1%   
   general      306296     COMPLETED           smk-map2db-sample=barcode82             16         24G      00:01:39       00:26:24     68.9%    17.8%     2.8%   
   general      306297     COMPLETED           smk-map2db-sample=barcode70             16         24G      00:02:04       00:33:04     76.0%    19.5%     3.4%   
   general      306298     COMPLETED           smk-map2db-sample=barcode94             16         24G      00:01:44       00:27:44     70.1%    17.9%     2.9%   
   general      306331     COMPLETED           smk-map2db-sample=barcode59             16         24G      00:04:00       01:04:00     87.2%    19.6%     6.7%   
   general      306332     COMPLETED           smk-map2db-sample=barcode83             16         24G      00:02:13       00:35:28     76.3%    19.0%     3.7%   
   general      306333     COMPLETED           smk-map2db-sample=barcode11             16         24G      00:02:49       00:45:04     81.6%    19.4%     4.7%   
   general      306334     COMPLETED           smk-map2db-sample=barcode23             16         24G      00:03:33       00:56:48     61.6%    19.0%     5.9%   
   general      306598     COMPLETED      smk-mapping_overview-sample=barcode46         1         4G       00:00:09       00:00:09     77.8%     0.0%     1.5%   
   general      306601     COMPLETED      smk-mapping_overview-sample=barcode82         1         4G       00:00:09       00:00:09     77.8%     0.0%     1.5%   
   general      306625     COMPLETED      smk-mapping_overview-sample=barcode94         1         4G       00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306628     COMPLETED      smk-concatenate_fastq-sample=barcode71        1         1G       00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306629     COMPLETED      smk-concatenate_fastq-sample=barcode70        1         1G       00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306630     COMPLETED      smk-concatenate_fastq-sample=barcode34        1         1G       00:00:07       00:00:07     57.1%     0.0%     1.2%   
```

In the example above, way too much memory was requested for all the jobs in general, and also the time limits were way too long. The most important is the CPU efficiency, however, which was generally good except for one job, but it was a very small job.

## Usage reports
`sreport` can be used to summarize usage in many different ways, below are some examples.

### Account usage by user
```
$ sreport cluster AccountUtilizationByUser format=account%8,login%23,used%10 -t hourper start="now-1week"
--------------------------------------------------------------------------------
Cluster/Account/User Utilization 2024-03-13T12:00:00 - 2024-03-19T23:59:59 (561600 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
 Account                   Login               Used 
-------- ----------------------- ------------------ 
    root                             154251(63.71%) 
    root              <username>      37176(15.35%) 
     jln                                3988(1.65%) 
     jln              <username>        2010(0.83%) 
     jln              <username>           0(0.00%) 
     jln              <username>        1978(0.82%) 
     kln                                   7(0.00%) 
     kln              <username>           7(0.00%) 
      ma                               13460(5.56%) 
      ma              <username>        6152(2.54%) 
      ma              <username>        2504(1.03%) 
      ma              <username>        3710(1.53%) 
      ma              <username>         963(0.40%) 
      ma              <username>         131(0.05%) 
      md                              52921(21.86%) 
      md              <username>       18690(7.72%) 
      md              <username>       22443(9.27%) 
      md              <username>       11788(4.87%) 
     mms                               17036(7.04%) 
     mms              <username>         600(0.25%) 
     mms              <username>       16436(6.79%) 
     phn                                6799(2.81%) 
     phn              <username>         114(0.05%) 
     phn              <username>          31(0.01%) 
     phn              <username>        6654(2.75%) 
     phn              <username>           0(0.00%) 
     sss                               22081(9.12%) 
     sss              <username>       22081(9.12%) 
students                                 638(0.26%) 
students              <username>         638(0.26%) 
```

### User usage by account
```
$ sreport cluster UserUtilizationByAccount format=login%23,account%8,used%10 -t hourper start="now-1week"
--------------------------------------------------------------------------------
Cluster/User/Account Utilization 2024-03-13T12:00:00 - 2024-03-19T23:59:59 (561600 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
                  Login  Account               Used 
----------------------- -------- ------------------ 
             <username>     root      37176(15.35%) 
             <username>       md       22443(9.27%) 
             <username>      sss       22081(9.12%) 
             <username>       md       18690(7.72%) 
             <username>      mms       16436(6.79%) 
             <username>       md       11788(4.87%) 
             <username>      phn        6654(2.75%) 
             <username>       ma        6152(2.54%) 
             <username>       ma        3710(1.53%) 
             <username>       ma        2504(1.03%) 
             <username>      jln        2010(0.83%) 
             <username>      jln        1978(0.82%) 
             <username>       ma         963(0.40%) 
             <username> students         638(0.26%) 
             <username>      mms         600(0.25%) 
             <username> compute+         145(0.06%) 
             <username>       ma         131(0.05%) 
             <username>      phn         114(0.05%) 
             <username>      phn          31(0.01%) 
             <username>      kln           7(0.00%) 
             <username>      phn           0(0.00%) 
             <username>      jln           0(0.00%) 
```

### Top users
```
$ sreport user top topcount=20 format=login%21,account%8,used%10 start="now-1week" -t hourper
--------------------------------------------------------------------------------
Top 20 Users 2024-03-13T11:00:00 - 2024-03-19T23:59:59 (565200 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
                Login  Account               Used 
--------------------- -------- ------------------ 
          <username>      root      37176(15.26%) 
          <username>        md       22698(9.32%) 
          <username>       sss       22082(9.06%) 
          <username>        md       18850(7.74%) 
          <username>       mms       16436(6.75%) 
          <username>        md       12038(4.94%) 
          <username>       phn        6654(2.73%) 
          <username>        ma        6167(2.53%) 
          <username>        ma        3780(1.55%) 
          <username>        ma        2504(1.03%) 
          <username>       jln        2383(0.98%) 
          <username>       jln        1978(0.81%) 
          <username>        ma         963(0.40%) 
          <username>  students         648(0.27%) 
          <username>       mms         600(0.25%) 
          <username>        kt         146(0.06%) 
          <username>        ma         133(0.05%) 
          <username>       phn         114(0.05%) 
          <username>       kln          98(0.04%) 
          <username>       phn          31(0.01%) 
```

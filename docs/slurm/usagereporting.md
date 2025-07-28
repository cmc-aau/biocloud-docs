# Usage reporting
Every month users get an automated usage report sent by email. The report contains only basic, but important, information such as total CPU allocation hours and an efficiency summary of all jobs run in the past month. The latter is essential information to minimize wasted resources and thus also the average queue time for all users. For additional data you can use some of the commands below.

## Running jobs
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

## Past jobs
To view the status of **past** jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/archive/slurm-24.11.4/sacct.html). `sacct` will return **everything** accounted for by default which is very inconvenient to view in a terminal window, so the below command will show the most essential information:
```
$ sacct -o jobid,jobname,start,end,NNodes,NCPUS,ReqMem,CPUTime,AveRSS,MaxRSS --user=$USER --units=G -j 138
JobID           JobName               Start                 End   NNodes      NCPUS     ReqMem    CPUTime     AveRSS     MaxRSS 
------------ ---------- ------------------- ------------------- -------- ---------- ---------- ---------- ---------- ---------- 
138          interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16        20G   00:02:56                       
138.interac+ interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56          0          0 
138.extern       extern 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56      0.00G      0.00G 

```

There is a large number of other options to show, see [SLURM docs](https://slurm.schedmd.com/archive/slurm-24.11.4/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use for example `sacct --long | less -S`.

## Reservations
Show current reservations in the system and reservation usage of the reservation total
```
# show current reservations in the system
$ sinfo -T
RESV_NAME       STATE           START_TIME             END_TIME     DURATION  NODELIST
maintenance  INACTIVE  2023-12-18T23:00:00  2023-12-20T01:00:00   1-02:00:00  bio-oscloud[02-09]

# show details about all current reservations
$ scontrol show reservations
ReservationName=amplicon StartTime=2024-11-04T08:00:00 EndTime=2024-11-18T08:00:00 Duration=14-00:00:00
   Nodes=bio-oscloud03 NodeCnt=1 CoreCnt=192 Features=(null) PartitionName=general Flags=
     NodeName=bio-oscloud03 CoreIDs=0-191
   TRES=cpu=192
   Users=abc@bio.aau.dk Groups=(null) Accounts=(null) Licenses=(null) State=ACTIVE BurstBuffer=(null) Watts=n/a
   MaxStartDelay=(null)

# show reservation utilization in CPU hours and percent of the reservation total used
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

# Other commands / FAQ
Below are some nice to know commands and example output. This will continously be populated as people ask for certain things. Your question here!

## How can I get a more detailed overview of the job queue and their requested ressources
```
squeue -o "%.18i %Q  %.8j %.8u %.2t %.10M %.10L %.6C %m %R"
             JOBID PRIORITY      NAME     USER ST       TIME  TIME_LEFT   CPUS MIN_MEMORY NODELIST(REASON)
                 9 4294901751      test user01@b PD       0:00 14-00:00:00     40 300G (Resources)
                10 4294901750  minimap2 user02@b PD       0:00    1:00:00      4 40G (Nodes required for job are DOWN, DRAINED or reserved for jobs in higher priority partitions)
                11 4294901749      test user01@b PD       0:00 14-00:00:00      4 10G (Priority)
                 8 4294901752      test user01@b  R      20:35 13-23:39:25     40 0 bio-oscloud04
```

## Show busy/free cores for the entire cluster
Example output (A=allocated, I=idle, O=other, T=total):
```
$ sinfo -o "%C"
CPUS(A/I/O/T)
10/374/0/38
```

## Generate a usage report
```
$ sreport cluster UserUtilizationByAccount -t Hours  Users=$USER
--------------------------------------------------------------------------------
Cluster/User/Account Utilization 2023-11-20T00:00:00 - 2023-11-20T23:59:59 (86400 secs)
Usage reported in CPU Hours
--------------------------------------------------------------------------------
  Cluster     Login     Proper Name         Account     Used   Energy 
--------- --------- --------------- --------------- -------- -------- 
 biocloud abc@bio.+             ksa compute-account        6        0 

```

## Show reserved compute nodes
```
$ sinfo -T
RESV_NAME      STATE           START_TIME             END_TIME     DURATION  NODELIST
group1       ACTIVE  2021-08-31T10:23:01  2021-12-15T09:23:01  106-00:00:00  node[01,02]
```

## Show details about a particular job
```
$ scontrol show job 24
JobId=24 JobName=interactive
   UserId=ksa@bio.aau.dk(101632) GroupId=ksa@bio.aau.dk(101632) MCS_label=N/A
   Priority=4294901738 Nice=0 Account=compute-account QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:02:00 TimeLimit=14-00:00:00 TimeMin=N/A
   SubmitTime=2023-11-01T11:20:01 EligibleTime=2023-11-01T11:20:01
   AccrueTime=Unknown
   StartTime=2023-11-01T11:20:01 EndTime=2023-11-15T11:20:01 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2023-11-01T11:20:01 Scheduler=Main
   Partition=general AllocNode:Sid=bio-ospikachu02:340145
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=bio-ospikachu05
   BatchHost=bio-ospikachu05
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=10G,node=1,billing=1
   AllocTRES=cpu=1,mem=10G,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:1 CoreSpec=*
   MinCPUsNode=1 MinMemoryNode=10G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/bin/bash
   WorkDir=/user_data/ksa/projects/slurmtest
   Power=
```

## Show details about the whole cluster configuration
```
$ scontrol show config
```
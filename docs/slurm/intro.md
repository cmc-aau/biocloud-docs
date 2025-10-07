# Introduction to SLURM
SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters of any size, from a single server to tens of thousands. SLURM manages resources on an HPC cluster by dividing similar compute nodes into [partitions](partitions.md). Users submit jobs with specified resource requirements to these partitions from a login-node, and then the SLURM controller schedules and allocates resources to those jobs based on available resources. SLURM also stores detailed usage information of all jobs in a usage accounting database, which allows enforcement of fair-share policies and priorities for job scheduling for each partition.

## BioCloud SLURM cluster overview
![SLURM overview](img/slurm-overview-inverted.png)

(Note: the exact partitions in the figure may be outdated, but the setup is the same)

## Getting started
Start with obtaining shell access to one of the login nodes `bio-fe[01-02].srv.aau.dk`, as described on the [SSH access](../access/ssh.md) page. To start with it's always nice to get an overview of the cluster, it's partitions, and how many resources that are currently allocated. This is achieved with the `sinfo` command, example output:

```
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
general*     up 14-00:00:0      3    mix bio-oscloud[02-04]
general*     up 14-00:00:0      2   idle bio-oscloud[05-06]
high-mem     up 28-00:00:0      1  boot^ bio-oscloud07
high-mem     up 28-00:00:0      1  alloc bio-oscloud08
gpu          up 14-00:00:0      1    mix bio-oscloud09
```

To get an overview of running jobs use `squeue`, example output:
```
# everything
$ squeue
 JOBID         NAME       USER       TIME    TIME_LEFT CPU MIN_ME ST PARTITION NODELIST(REASON)
  2380         dRep user01@bio 9-00:31:24   4-23:28:36  80   300G  R   general bio-oscloud02
  2389        dramv user02@bio 8-16:14:07   5-07:45:53  16   300G  R   general bio-oscloud02
  3352       METAGE user03@cs. 1-13:00:45     10:59:15  10   125G  R   general bio-oscloud03
  3359      ar-gtdb user02@bio 1-00:04:32   5-23:55:28  32   300G  R   general bio-oscloud03
  3361     bac_gtdb user02@bio 1-00:03:05   5-23:56:55  32   500G  R   general bio-oscloud04
  3366  interactive user04@bio    2:03:42   2-00:56:18  60   128G  R   general bio-oscloud02
  3426       blastn user05@bio      41:56   3-23:18:04  96   200G  R   general bio-oscloud03
  3430  interactive user06@bio       7:23      3:52:37   1     2G  R   general bio-oscloud02
  3333 as-predictio user07@bio 2-19:42:49   6-04:17:11   5    16G  R       gpu bio-oscloud09
  3372      checkm2 user02@bio   21:37:50   6-02:22:10 192  1800G  R  high-mem bio-oscloud08

# your own jobs only
$ squeue --me
 JOBID         NAME       USER       TIME    TIME_LEFT CPU MIN_ME ST PARTITION NODELIST(REASON)
  3333 as-predictio user07@bio 2-19:42:49   6-04:17:11   5    16G  R       gpu bio-oscloud09
```

Or get a more detailed overview per compute node of current resource allocations and which jobs are running etc. This will normally show some colored bars, but they are not visible here.
```
$ sstatus
Cluster allocation summary per partition or individual nodes (-n).
(Numbers are reported in free/allocated/total(OS factor)).

Partition    |                CPUs                 |           Memory (GB)           |       GPUs        |
==========================================================================================================
shared       |  838 218                 /1056 (3x) | 1056 368                 /1424  |
general      |  715 245                 /960       | 3870 765                 /4635  |
high-mem     |  190 242                 /432       | 1608 2131                /3739  |
gpu          |   44 20                  /64        |   84 125                 /209   |    1 1         /2
----------------------------------------------------------------------------------------------------------
Total:       | 1787 725                 /2512      | 6620 3389                /10009 |    1 1         /2

Jobs running/pending/total:
  23 / 1 / 24

Use sinfo or squeue to obtain more details.
```

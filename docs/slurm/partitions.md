# Compute node partitions
The compute nodes are divided into separate partitions based on their hardware configuration. This is to ensure that for example CPU's from different manufacturer generations can be billed differently in the usage accounting (newer CPU's are faster), nodes with more memory (per CPU) are only used for jobs that actually require more memory, and GPU nodes are only used for jobs that require GPU's, etc. BioCloud is a **heterogeneous cluster** because nodes are purchased at different times, hence their hardware configuration is different. Furthermore, the number of partitions will only increase in the future as more nodes are added to the cluster at different times, which increases complexity, making it difficult or confusing to submit jobs to the most appropriate partition(s). This can result in an inefficient cluster with longer queue times and wasted computing resources.

Below is a list of the current compute node partitions, their hardware configuration, and the billing factor for each partition. 

???+ info "Partition selection is automatic"
      To make it as simple as possible for you, and to increase the overall cluster efficiency by ensuring that the most appropriate hardware is used for each job, the partition for your job(s) is set automatically by the SLURM scheduler. The resulting partition is selected based on several factors, where the most important are the requested **memory per CPU ratio** and the required [**node features**](#node-features), if any. Therefore, submitting jobs to specific partitions using the `--partition` option will have no effect, as it will be overwritten.

## CPU partitions
All CPUs are AMD EPYC, all except one are dual socket. Interactive

| Partition | Hostname | EPYC model | CPUs/Threads | Memory | Scratch space |
| ---: | :--- | :---: | :---: | :---: | :---: |
| `interactive` | `node1` <br> `axomamma`| 7552 <br> 7713 | 48C / 96T <br> 128C / 256T | 0.5 TB <br> 1.0 TB | <br> 3.5 TB |
| `slim-zen3` | `node[2-6]` | 7643 |  96C / 192T | 1.0 TB | 18 TB (node5) |
| `slim-zen5` | `node[7-9]` | 9565 | 144C / 288T | 1.5 TB | |
| `fat-zen3` | `node10` <br> `node11` | 7643 <br> 7713 | 96C / 192T <br> 128C / 256T | 2.0 TB <br> 2.0 TB | |
| `fat-zen5` | `node[12-13]` | 9565 | 144C / 288T | 2.3 TB | 12.8 TB |

| Partition | Usage billing factor |
| ---: | :--- |
| `interactive` | 0.5x |
| `slim-zen3` | 1.0x |
| `slim-zen5` | 1.5x |
| `fat-zen3` | 1.5x |
| `fat-zen5` | 2.0x |
| `gpu-a10` | 1.0x / GPU 32x|

## GPU partitions
| Partition | Hostname | EPYC model | CPUs/Threads | Memory | Scratch space | GPU |
| ---: | :--- | :---: | :---: | :---: | :---: | :---: |
| `gpu-a10` | `node14`| 7313 | 32C / 64T | 256 GB | 3.0 TB | NVIDIA A10 |

## Billing
```
$ scontrol show partition | grep -e '^PartitionName' -e 'TRESBillingWeights'
PartitionName=interactive
   TRESBillingWeights=cpu=0.5
PartitionName=slim-zen3
   TRESBillingWeights=cpu=1.0
PartitionName=slim-zen5
   TRESBillingWeights=cpu=1.5
PartitionName=fat-zen3
   TRESBillingWeights=cpu=1.5
PartitionName=fat-zen5
   TRESBillingWeights=cpu=2.0
PartitionName=gpu-a10
   TRESBillingWeights=cpu=1.0,gres/gpu=32.0
```

## Node features
Use `--prefer` or `--constraint`.
```
$ sinfo
  PARTITION AVAIL  TIMELIMIT  CPUS(A/I/O/T)     STATE       REASON  NODES NODELIST AVAIL_FEATURES
interactiv*    up 1-00:00:00      0/40/0/40      idle         none      1 bio-comp1 zen3,scratch
  slim-zen3    up 14-00:00:0      0/40/0/40      idle         none      1 bio-comp2 zen3
  slim-zen5    up 14-00:00:0      0/40/0/40      idle         none      1 bio-comp4 zen5
   fat-zen3    up 14-00:00:0      0/40/0/40      idle         none      1 bio-comp3 zen3,scratch
   fat-zen5    up 14-00:00:0      0/40/0/40      idle         none      1 bio-comp5 zen5,scratch
    gpu-a10    up 14-00:00:0      0/40/0/40      idle         none      1 bio-comp6 zen3,scratch,a10
```

# Compute node partitions
The compute nodes are divided into separate partitions based on their hardware configuration. This is to allow that for example CPU's from different manufacturer generations can be set up with a different [billing factor](https://slurm.schedmd.com/archive/slurm-24.11.4/slurm.conf.html#OPT_TRESBillingWeights) to ensure fair usage accounting (newer CPU's are faster), nodes with more memory (per CPU) are only used for jobs that actually require more memory, and GPU nodes are only used for jobs that require GPU's, etc. BioCloud is a quite **heterogeneous cluster** because nodes are purchased at different times, hence their hardware configuration is also different. Furthermore, the number of partitions will only increase in the future as more nodes are added to the cluster at different times, which increases complexity, making it difficult or confusing to submit jobs to the most appropriate partition(s). This can result in an inefficient cluster with longer queue times and wasted computing resources.

Below is a list of the current compute node partitions, as well as the hardware configuration and unique features of each node.

???+ info "Partition selection is automatic"
      To make it as simple as possible for you, and to increase the overall cluster efficiency by ensuring that the most appropriate hardware is used for each job, the partition for your job(s) is assigned automatically by the SLURM scheduler. The resulting partition is selected based on several factors, where the most important are the requested **memory per CPU ratio** and any required **node features**. Therefore, submitting jobs to specific partitions using the `--partition` option will have no effect, as it will be overwritten. In very specific scenarios the automatically assigned partition may not be ideal, in which case exceptions can be made, just contact an administrator.

## CPU partitions
Below is a brief overview of all CPU partitions. Details about the exact CPU model, scratch space and special features for each compute node are listed further down.

### Overview
| Partition | Nodes | Total CPUs | Total memory | Billing factor |
| ---: | :--: | :--: | :--: | :--- |
| `interactive` | 2 | 352T (x2) | 1.5 TB | 0.5x |
| `slim-zen3` | 5 | 960T | 5.0 TB | 1.0x |
| `slim-zen5` | 3 | 864T | 4.5 TB | 1.5x |
| `fat-zen3` | 2 | 576T | 4.0 TB | 1.5x |
| `fat-zen5` | 2 | 576T | 4.6 TB | 2.0x |
| **TOTAL** | **14** | **3328T (3680)** | **19.6 TB** | |

### The `interactive` partition
This partition is reserved for interactive jobs for people to be able to do data analysis (usually produced from batch jobs) without having to wait for hours or days due to queue time.

The `interactive` partition is set up with an over-subscription factor of 2, which means each CPU can be used by up to 2 jobs at once to maximize CPU utilization because interactive jobs are usually very inefficient.

???+ info "Memory per CPU on the interactive partition"
      Because the CPU's on the `interactive` partition are over-subscribed, the SLURM scheduler will sometimes allocate more CPUs than you have initially requested for your job until the max memory per CPU setting of **2.0 GB** is satisfied. This is to avoid idle CPU's due to exhausted memory (memory cannot be shared) and to avoid that nodes crash because they run out of memory, [details here](https://slurm.schedmd.com/archive/slurm-24.11.4/slurm.conf.html#OPT_MaxMemPerCPU). It is therefore ideal to detect the number of CPUs available dynamically in your scripts and commands using for example `nproc` or from the `SLURM_JOB_CPUS_PER_NODE` environment variable.

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node01`| 2x AMD EPYC 7713 | 128C / 256T | 1.0 TB | 3.5 TB NVMe | `zen3` <br>`scratch` |
| `bio-node02` | 1x AMD EPYC 7552P | 48C / 96T | 0.5 TB | | `zen3` |

### Batch job partitions
These partitions are dedicated to non-interactive and efficient batch jobs that can run for a long time. The `slim-*` nodes generally have less memory per CPU, while the `fat-*` nodes have more memory per CPU, which is useful for jobs that require a lot of memory. The `zen3` and `zen5` features indicate the generation of AMD EPYC CPUs used in the nodes.

**`slim-zen3`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node[03-06]` | 2x AMD EPYC 7643 | 96C / 192T | 1.0 TB | | `zen3` |
| `bio-node07` | 2x AMD EPYC 7643 | 96C / 192T | 1.0 TB | 18 TB NVMe | `zen3`<br>`scratch` |

**`slim-zen5`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node08` | 2x AMD EPYC 9565 | 144C / 288T | 1.5 TB | | `zen5` |

**`fat-zen3`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node09` | 2x AMD EPYC 7643 | 96C / 192T | 2.0 TB | | `zen3` |
| `bio-node10` | 2x AMD EPYC 7713 | 128C / 256T | 2.0 TB | 12.8 TB NVMe | `zen3`<br>`scratch` |

**`fat-zen5`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `node[12-13]` | 2x AMD EPYC 9565 | 144C / 288T | 2.3 TB | 12.8 TB NVMe | `zen5`<br>`scratch` |

## GPU partitions

**`gpu-a10`**

| Hostname | CPU model | CPUs | Memory | Scratch space | GPU | Features |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| `node14`| 2x AMD EPYC 7313 | 32C / 64T | 256 GB | 3.0 TB NVMe | NVIDIA A10 | `zen3`<br>`scratch`<br>`a10` |

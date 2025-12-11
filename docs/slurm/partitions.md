# Compute node partitions
The compute nodes are divided into separate partitions based on their hardware configuration. This is to allow that for example CPU's from different manufacturer generations can be set up with a different [billing factor](https://slurm.schedmd.com/archive/slurm-24.11.4/slurm.conf.html#OPT_TRESBillingWeights) to ensure fair usage accounting (newer CPU's are faster), nodes with more memory (per CPU) are only used for jobs that actually require more memory, and GPU nodes are only used for jobs that require GPU's, etc.

## Automatic partition selection
BioCloud is a quite **heterogeneous cluster** because nodes are purchased at different times, hence their hardware configuration is also different. Furthermore, the number of partitions will only increase in the future as more nodes are added to the cluster at different times, which increases complexity, making it difficult or confusing to submit jobs to the most appropriate partition(s). This can result in an inefficient cluster with longer queue times and wasted computing resources. Therefore, the most appropriate partition for **batch jobs** is automatically assigned by the SLURM scheduler according to custom logics defined for our specific setup. Manually specifying a partition using the `--partition` option will have no effect, as it will be overridden. Interactive jobs will **always** be assigned the `interactive` partition.

The most appropriate partition is determined by several factors, the most significant of which are the requested **memory per CPU ratio** and any specified [**node features**](jobsubmission.md#requesting-compute-nodes-with-special-features). Partitions are allocated according to the defined [priority tiers](https://slurm.schedmd.com/archive/slurm-24.11.4/slurm.conf.html#OPT_PriorityTier) shown in the table below, ensuring that newer (and faster) compute nodes are always selected first before older nodes. Secondly, if all general compute nodes happen to be fully allocated, jobs with only modest memory requirements will still be able to run on compute nodes with extra memory instead, if available. Conversely, jobs that require extra memory per CPU will not be able to run on a general compute node as this can result in unavailable CPUs due to fully allocated memory. In certain situations, the automatically assigned partition may not be optimal, in which case manual intervention by an administrator may be necessary and exceptions can be made if it makes sense depending on the situation.

## CPU partitions
Below is a brief overview of all CPU partitions. Details about the exact CPU model, scratch space and node features for each compute node are listed further down.

### Overview
| Partition | Nodes | Total CPUs | Total memory | Billing factor | Priority tier |
| ---: | :--: | :--: | :--: | :--: | :--: |
| `interactive` | 2 | 512T | 3.0 TB | 0.5x | - |
| `zen5` | 3 | 864T | 4.5 TB | 1.0x | 1st |
| `zen3` | 8 | 1312T | 6.5 TB | 0.5x | 2nd |
| `zen5x` | 2 | 576T | 4.6 TB | 1.5x | 3rd |
| `zen3x` | 2 | 448T | 4.0 TB | 1.0x | 4th |
| **TOTAL** | **17** | **3712** | **19.6 TB** | | |

### The `interactive` partition
This partition is reserved for short and small interactive jobs, where users can do data analysis, quick testing, and day-to-day work without having to wait for hours or even days due to queue time. Therefore, no batch jobs will be able to run here, and there is a max CPUs per job limit of `32` to ensure high availability. Ideally, the `interactive` partition should never be fully utilized. Furthermore, it is optimized for interactive jobs, which are usually very inefficient (e.i. the allocated CPU's do absolutely nothing when you are just typing or clicking around).

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node[16-17]` | 2x AMD EPYC 9535 | 128C / 256T | 1.5 TB | | `zen5`<br>`epyc9535` |

### Batch job partitions
These partitions are dedicated to non-interactive and efficient batch jobs that can potentially run for a long time. The `slim-*` nodes generally have less memory per CPU, while the `fat-*` nodes have more memory per CPU, which is useful for jobs that require a lot of memory. The `zen3` and `zen5` features indicate the generation of AMD EPYC CPUs used in the nodes.

**`zen3`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node01`| 2x AMD EPYC 7713 | 128C / 256T | 1.0 TB | 3.5 TB NVMe | `zen3`<br>`epyc7713`<br>`scratch` |
| `bio-node02` | 1x AMD EPYC 7552P | 48C / 96T | 0.5 TB | | `zen3`<br>`epyc7552p` |
| `bio-node[03,04,06,07]` | 2x AMD EPYC 7643 | 96C / 192T | 1.0 TB | | `zen3`<br>`epyc7643` |
| `bio-node05` | 2x AMD EPYC 7643 | 96C / 192T | 1.0 TB | 18 TB NVMe | `zen3`<br>`epyc7643`<br>`scratch` |

**`zen3x`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node08` | 2x AMD EPYC 7643 | 96C / 192T | 2.0 TB | | `zen3`<br>`epyc7643` |
| `bio-node09` | 2x AMD EPYC 7713 | 128C / 256T | 2.0 TB | 12.8 TB NVMe | `zen3`<br>`epyc7713` |

**`zen5`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node[11-13]` | 2x AMD EPYC 9565 | 144C / 288T | 1.5 TB | | `zen5`<br>`epyc9565` |

**`zen5x`**

| Hostname | CPU model | CPUs | Memory | Scratch space | Features |
| ---: | :---: | :---: | :---: | :---: | :---: |
| `node[14-15]` | 2x AMD EPYC 9565 | 144C / 288T | 2.3 TB | 12.8 TB NVMe | `zen5`<br>`epyc9565`<br>`scratch` |

## GPU partitions

**`gpu-a10`**

| Hostname | CPU model | CPUs | Memory | Scratch space | GPU | Features |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| `bio-node10`| 2x AMD EPYC 7313 | 32C / 64T | 256 GB | 3.0 TB NVMe | NVIDIA A10 | `zen3`<br>`epyc7313`<br>`scratch`<br>`a10` |

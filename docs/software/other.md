# Other software
In rare cases, some software tools are not readily available through conda or biocontainers. Below are some guides on how to run some them.

## ARB 7
ARB version 6.0 can be installed through conda from the [bioconda](https://anaconda.org/bioconda/arb-bio) channel, however it is not maintained anymore, and the latest version 7.0 is not available. To run version 7.0 on BioCloud, you need to run a wrapper script that runs ARB 7.0 from a custom built container image. You can do that in two ways, either in a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) started from the web portal, or through an [interactive shell session](../slurm/jobsubmission.md#graphical-apps-gui) using `salloc` from a login node. ARB 7 is then available by simply typing `arb7` in the terminal.

Each method has its own pros and cons. If you run ARB from within a virtual desktop, it will keep running even if you shut down your computer, but it is not as responsive and it can be inconvenient with many windows open at the same time. If you run through an interactive shell session, windows will instead show up on your own computer. As mentioned in the [job submission guide](../slurm/jobsubmission.md#graphical-apps-gui) ensure that you have X11 forwarding enabled for your SSH connection if you want to run things through SSH using `salloc`. Regardless of the method, everything will run in an isolated SLURM job on a BioCloud compute node.

## CLC
CLC is licensed software tied to specific machines and is only available through a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) on nodes with the `clc` [node feature](../slurm/partitions.md#the-interactive-partition) listed. So ensure that you write `--constraint clc` in the "Additional job options" field. You can then find CLC in the menus.

## AlphaFold
[AlphaFold](https://github.com/google-deepmind/alphafold) is quite complex to install and run. It runs through containers, but through a python wrapper script. Copy the SLURM sbatch script from `/shared_software/biocloud-software/alphafold_singularity/sbatch_example.sh` and adjust to suit your needs. `AlphaFold` benefits from GPU-accelerated servers, but can also run on regular CPUs. So submit to the appropriate partition.

To make folders available inside the container you need to bind/mount them, as described [here](containers.md#binding-mounting-folders-from-the-host-to-the-container).

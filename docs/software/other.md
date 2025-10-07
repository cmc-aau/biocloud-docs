# Other software
In rare cases, some software tools are not readily available through conda or biocontainers. Below are some guides on how to run some them.

## ARB 7
ARB version 6.0 can be installed through conda from the [bioconda](https://anaconda.org/bioconda/arb-bio) channel, however it is not maintained anymore, and the latest version 7.0 is not available. To run version 7.0 on BioCloud, you need to run a wrapper script that runs ARB 7.0 from a custom built container image. You can do that through either a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) started from the web portal, or through an [interactive shell session](../slurm/jobsubmission.md#graphical-gui-apps) using `salloc` from a login node. ARB 7 is then available by simply typing `arb7` in the terminal.

## CLC
As of 2026, CLC is licensed on a per-user basis and is no longer tied to specific machines. CLC is installed only on nodes in the `interactive` partition. You can find CLC in the menus in a [virtual desktop](../guides/webportal/apps/virtualdesktop.md).

## AlphaFold
[AlphaFold](https://github.com/google-deepmind/alphafold) is quite complex to install and run. It runs through containers, but through a python wrapper script. Copy the SLURM sbatch script from `/software/biocloud-software/alphafold_singularity/sbatch_example.sh` and adjust to suit your needs. `AlphaFold` benefits from GPU-accelerated servers, but can also run on regular CPUs. So submit to the appropriate partition.

To make folders available inside the container you need to bind/mount them, as described [here](containers.md#binding-mounting-folders-from-the-host-to-the-container).

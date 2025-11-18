# Other software
In rare cases, some software tools are not readily available through conda or biocontainers. Below are some guides on how to run some them.

## ARB 7
ARB version 6.0 can be installed through conda from the [bioconda](https://anaconda.org/bioconda/arb-bio) channel, however it is not maintained anymore, and the latest version 7.0 is not available. To run version 7.0 on BioCloud, you need to run a wrapper script that runs ARB 7.0 from a custom built container image. You can do that through either a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) started from the web portal, or through an [interactive shell session](../slurm/jobsubmission.md#graphical-gui-apps) using `salloc` from a login node. ARB 7 is then available by simply typing `arb7` in the terminal.

## CLC
CLC version 25 is currently only installed on (and licensed to) the `bio-node11` node in the `interactive` partition (currently the only node there). You should be able to find CLC in the menus when starting a [virtual desktop](../guides/webportal/apps/virtualdesktop.md). As of November 2025, newer versions of CLC are licensed on a per-user basis and no longer tied to specific machines.

## AlphaFold
[AlphaFold](https://github.com/google-deepmind/alphafold) is quite complex to install and run. It runs within a container, but it must be started using a python wrapper script. Copy the SLURM sbatch script from `/software/lib/biocloud-software/alphafold_singularity/sbatch_example.sh` and adjust to suit your needs. `AlphaFold` benefits from GPU-accelerated servers, but can also run on regular CPUs. So submit to the appropriate partition.

To make folders available inside the container you need to bind/mount them, as described [here](containers.md#binding-mounting-folders-from-the-host-to-the-container).

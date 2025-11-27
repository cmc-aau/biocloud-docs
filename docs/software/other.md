# Other software
In rare cases, some software tools are not readily available through conda or biocontainers, or they are problematic to get to run properly. Below are some guides on how to run some them. Most are installed under `/software/lib/biocloud-software`.

## ARB 7
ARB version 6.0 can be installed through conda from the [bioconda](https://anaconda.org/bioconda/arb-bio) channel, however it is not maintained anymore, and the latest version 7.0 is not available. To run version 7.0 on BioCloud, you need to run a wrapper script that runs ARB 7.0 from a custom built container image. You can do that through either a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) started from the web portal, or through an [interactive shell session](../slurm/jobsubmission.md#graphical-gui-apps) using `salloc` from a login node. ARB 7 is then available by simply typing `arb7` in the terminal.

## CLC
CLC version 25 is currently only installed on (and licensed to) the `bio-node11` node in the `interactive` partition (currently the only node there). You should be able to find CLC in the menus when starting a [virtual desktop](../guides/webportal/apps/virtualdesktop.md). As of November 2025, newer versions of CLC are licensed on a per-user basis and no longer tied to specific machines.

## AlphaFold
[AlphaFold](https://github.com/google-deepmind/alphafold) is quite complex to install and run. It runs within a container, but it must be started using a python wrapper script. It has been bundled up under `/software/lib/biocloud-software/alphafold_singularity/`. Just Copy the SLURM sbatch script from `/software/lib/biocloud-software/alphafold_singularity/sbatch_example.sh` and adjust to suit your needs. `AlphaFold` benefits from GPU-accelerated servers, but can also run on regular CPUs depending on the size of the input data.

## ESMFold
[ESMFold](https://github.com/facebookresearch/esm) can be run through a custom-built container by simply typing `esm-fold`.

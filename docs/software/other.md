# Other software
In rare cases, some software tools are not readily available through conda or biocontainers. Below are some guides on how to run some them.

## ARB
ARB version 6.0 can be installed through conda from the [bioconda](https://anaconda.org/bioconda/arb-bio) channel, however it is not maintained anymore, and the latest version 7.0 is not available. To run version 7.0 on BioCloud, you need to run a wrapper script that runs ARB 7.0 from a custom container image. You can do that in a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) started from the web portal, or through an [interactive shell session](../slurm/jobsubmission.md#graphical-apps-gui) using `salloc` or `srun`. The convenient thing about the latter is that windows will show up on your own computer, but everything is running inside a SLURM job on a BioCloud compute node:

```
srun --cpus-per-task 2 --mem 4G --time 0-3:00:00 --x11 bash /software/biocloud-software/arb/run.sh
```

As mentioned in the [job submission guide](../slurm/jobsubmission.md#graphical-apps-gui) ensure that you have X11 forwarding enabled for your SSH connection if you want to run things through SSH using `salloc` or `srun`.

To run things in a virtual desktop, simply open a terminal and run `bash /software/biocloud-software/arb/run.sh`.

## CLC
CLC is licensed software tied to specific machines and is only available through a [virtual desktop](../guides/webportal/apps/virtualdesktop.md) on nodes with the `clc` [node feature](../slurm/partitions.md#the-interactive-partition) listed. So ensure that you write `--constraint clc` in the "Additional job options" field. You can then find CLC in the menus.

## AlphaFold
[AlphaFold](https://github.com/google-deepmind/alphafold) is quite complex to install and run. It runs through containers, but through a python wrapper script. Copy the SLURM sbatch script from `/shared_software/biocloud-software/alphafold_singularity/sbatch_example.sh` and adjust to suit your needs. `AlphaFold` benefits from GPU-accelerated servers, but can also run on regular CPUs. So submit to the appropriate partition.

To make folders available inside the container you need to bind/mount them, as described [here](containers.md#binding-mounting-folders-from-the-host-to-the-container).

## usearch11
`usearch11` is made available on all servers by simply running `usearch11` or `usearch`.

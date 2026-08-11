## Transferring data to/from BioCloud
To move files between your personal computer and BioCloud, you can use the file transfer support built into VS Code or MobaXterm. Other GUI tools such as [FileZilla](https://filezilla-project.org/download.php) or [WinSCP](https://winscp.net/eng/index.php) also work well. From a terminal, common SSH-based tools include `scp`, `rsync`, `rclone`, and `sftp`.

For smaller transfers, the [interactive web portal](webportal.md) is another convenient option.

For larger or longer transfers, especially from external sources like another HPC system, you can use either a persistent session on a login node or a non-interactive SLURM job to run the transfer on a compute node. On a login node, use [`tmux`](https://github.com/tmux/tmux/wiki/Getting-Started) or [`screen`](https://linuxize.com/post/how-to-use-linux-screen/#starting-a-screen-session) to keep the transfer running after you disconnect. For transferring through a compute node, submit a simple 1-CPU SLURM job like the example below. Running the transfer on a compute node also helps spread the network load across more interfaces than the login nodes alone.

Example `rsync` batch script:

```
#!/usr/bin/bash -l
#SBATCH --job-name=transfer
#SBATCH --output=job_%j_%x.out
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=END,FAIL,TIME_LIMIT_80
#SBATCH --mail-user=abc@bio.aau.dk

rsync -avP user@externalsrc:/path/to/src/folder/ local/biocloud/destination
```

Note the trailing `/` in both `src` and `dest`: it changes the behavior of `rsync`. With the trailing slash, `rsync` copies the contents of the source folder into the target folder. Without the trailing slash, it copies the source folder itself into the target location. If you want the two directories to be synchronized, omit the trailing slash on both paths.

Note that a non-interactive SLURM job cannot prompt for a password, so it will likely fail unless you configure public-key authentication for the external host, just as you would do for BioCloud logins, [see below](#ssh-public-key-authentication).

Avoid using `rsync` as a proxy through a third host whenever possible. A proxy transfer adds an extra network hop, which usually makes the transfer slower and increases load. Because BioCloud is not directly exposed to the internet, you will normally need to run transfers from BioCloud itself when the external source is publicly reachable.

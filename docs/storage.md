# Storage
All nodes are connected to a Ceph network storage cluster through a few different mount points for different purposes, listed below. Data is stored with 3x replication to protect against data loss or corruption due to hardware and disk failures. The data is **NOT backed up** anywhere, however, which means it's not protected against human errors. Therefore, **if you delete data - it's gone** and cannot be restored! Some mount points are therefore mounted read-only, but your own data is your responsibility alone. So be careful, also when sharing data with other users when doing shared projects etc.

Data on the Ceph network storage cluster is lightly compressed at rest, so manually compressing files is not needed, however it can be beneficial to reduce the number of files.

## Network storage mount points

| Mount point | Contents |
| --- | --- |
| `/home` | All data for individual projects and generally most things should go here |
| `/projects` | All data for shared projects should go here |
| `/databases` | Various databases required for bioinformatic tools |
| `/raw_data` | Raw data (mostly from DNA sequencing) |

Please always organize EVERYTHING under `/raw_data/` and `/projects` by `PI/project/(sub-project)` subfolders.

???+ "Max file size"
     There is a max file size of 4TB on the Ceph network cluster, which cannot be increased, [details here](https://docs.ceph.com/en/latest/cephfs/administration/#maximum-file-sizes-and-performance).

## Local scratch space
For jobs requiring heavy I/O where large amounts of temporary data needs to be written, it's important to avoid using the network storage and instead write to a local harddrive. This will avoid overburdening the network (which other jobs also need) and the storage cluster itself, but it can also be much faster in some cases. When deciding whether you need local scratch space, both the size of the data as well as the number of files are important, however the ladder is by far the most important because it has the biggest impact on the performance of the storage cluster overall, which can impact every other user, [more info below](#avoid-numerous-small-files-at-all-costs).

To use local scratch space, you must first ensure to submit your job(s) using the `--constraint scratch` option to the `srun`, `sbatch`, and `salloc` commands, which will ensure that your job is run on a [compute node](slurm/partitions.md) with local scratch space available. Then, simply ensure that your job writes to the `/tmp` folder, which will automatically be mounted to the local scratch space on the allocated compute node. The `/tmp` folder is private to each user and their jobs, so you can write whatever you want there without worrying about other jobs overwriting your files.

???+ warning 
      All data owned by your user anywhere on local scratch space on a particular compute node is **automatically deleted** after the last SLURM job run by your user has finished. Therefore, ensure to move files elsewhere as the last step of the job, otherwise it will be lost.

## Temporary space
Each job will have a separate and entirely private mount point for temporary data (default location is usually `/tmp` or `/var/tmp`), which will be automatically deleted after each job. On compute nodes with extra local scratch space it will automatically be mounted somewhere in there instead, so that there is a lot more space available for temporary files on those nodes (refer to [compute node partitions](slurm/partitions.md)).

If you would need more space for temporary data on compute nodes that have no extra local scratch space, or you need even more temporary space than there's available on local scratch space, it's possible to place it on the Ceph network storage as well. However, if you choose to do so, please see the [best practices](#avoid-numerous-small-files-at-all-costs) below. It can simply be done by for example setting the environment variable `TMPDIR` early in the batch script by adding a line, fx `export TMPDIR=${HOME}/tmp`. Ensure no conflicts can occur within the folder(s) if you run multiple jobs on multiple different compute nodes at once.

## Storage policy
**Your data - your responsibility!**

The Ceph network storage is **NOT** a long-term storage or backup solution. **EVERYTHING IS TEMPORARY, SO CLEAN UP AFTER YOURSELF IMMEDIATELY AFTER EACH JOB!!** You will likely forget about it very quickly after you've done your work, so make it a habit to cleanup shortly after every job. Once studies are published, delete all the data and ensure that for example raw DNA sequencing data is uploaded to public archives and that the corresponding code to produce results in the studies is available somewhere, preferably in a GitHub repository.

Furthermore, if you are no longer employed or studying at AAU, your home folder will be **deleted without notice**, and you have the sole responsibility to hand on any data that should be saved for longer by passing it on to other users, for example your PI, **before you leave**.

## Moving large amounts of data around
If you need to move large amounts of data (or numerous files at once regardless of size) it will happen in an instant regardless of the size if you make sure you are doing it on the same filesystem/mount, for example from `/projects/xxx` to `/projects/yyy`. However, if you need to move things across mount points, for example from `/home/xxx` to `/projects/yyy` please ask an administrator to do it for you, since moving between mount points will happen over the network and cause unneccessary traffic and load on the storage cluster, let alone be very slow. An administrator can move anything instantly anywhere on the storage cluster, but if you need to transfer external files there is no way around it, it will be limited by the network speed.

## Best practices with Ceph storage
### Avoid numerous small files at all costs
**Whereever possible always try to write fewer but larger files instead of many small ones**. The Ceph storage cluster uses a file metadata cache server that holds an entry for every single open file, so the cache memory usage is directly proportional to the number of open files (meaning being accessed in any way). If there are too many open files (across **ALL** connected clients/nodes at once) the metadata server will ask clients to release some pressure on the cache and if they fail to do so in time they will simply be [evicted](https://docs.ceph.com/en/reef/cephfs/eviction/) without notice (meaning the Ceph cluster will force an unmount). This will happen on any node that tips the iceberg and is thus not a result of the exact storage actions of individual clients but rather that of all nodes connected to the Ceph cluster across the whole university at any one time. See [Ceph best usage practices](https://docs.ceph.com/en/reef/cephfs/app-best-practices/) for additional details.

### Avoid using `ls -l`
When listing directories, it's common to use `ls -l` to list things vertically, however this will also request various other information like permissions, file size, owner, group, access time etc. This will burden the metadata servers, especially if used in loops in scripts on many files, so if you don't need all this extra information and just want to list the contents vertically instead of horizontally, just use `ls -1` instead and make that a habit. Likewise, don't use `stat` on many files if not neccessary.

### Obtaining the total size of folders
To obtain the total disk space used of all files inside a folder it's common to use the `du -sh /some/folder` command. Doing this at a large folder is quite similar to performing a [DDoS attack](https://en.wikipedia.org/wiki/Denial-of-service_attack) on the Ceph storage cluster, so please never use `du` on folders, only on individual files. It will likely never finish anyways if the folder contains many files. The best way to obtain the size of a folder is to instead obtain the information in the form of storage quota attributes directly from the Ceph metadata servers using the custom `storagequota` command as demonstrated below, which is both instant and will not cause any stress on the cluster:

```
# home folder
$ storagequota
Storage quota used for '/home/bio.aau.dk/abc': 3.46TB / 9.09TB (38.06%)

# specific folder
$ storagequota /projects
Storage quota used for '/projects': 397.54TB

# multiple folders, sorted by size
storagequota /projects/* | sort -rt ':' -k2 -n | head -n 5
Storage quota used for '/projects/microflora_danica':   	208.40TB
Storage quota used for '/projects/MiDAS':   	125.75TB
Storage quota used for '/projects/NNF_AD_2023':   	30.66TB
Storage quota used for '/projects/glomicave':   	8.84TB
Storage quota used for '/projects/dark_science':   	7.19TB
```

The `storagequota` command is simply a convenient wrapper script around the `getfattr` command, which retrieves attributes of files and folders directly from the Ceph MDS servers. It only retrieves the size of the folder, but there are also a few other attributes that may be of interest, for example the number of files, see examples below.

```
$ getfattr -n ceph.dir.rfiles /projects 
getfattr: Removing leading '/' from absolute path names
# file: projects
ceph.dir.rfiles="219203126"
```

??? "Additional Ceph attributes"
      | Attribute | Explaination |
      | --- | --- |
      | ceph.dir.entries | |
      | ceph.dir.files | Number of files in folder (non-recursive) |
      | ceph.dir.subdirs | Number of subdirs (non-recursive) |
      | ceph.dir.rentries | |
      | ceph.dir.rfiles | Number of files in folder (recursive) |
      | ceph.dir.rsubdirs | Number of folders in folder (recursive) |
      | ceph.dir.rbytes | Size of folder (recursive) |
      | ceph.dir.rctime | Recently changed time (non-recursive) |

      There are no descriptions on these in the Ceph documentation or anywhere else, I've simply found them in the Ceph source code! This is all there is.


## Shared folders
If you need to give other users write access to a file/folder that you own, you need to set the group ownership of the folder to the `bio_server_users@bio.aau.dk` group and set the [setGID](https://www.geeksforgeeks.org/setuid-setgid-and-sticky-bits-in-linux-file-permissions/) bit on folders (to ensure child files/folders will inherit the ownership of a parent folder), see the example below. This will give **everyone** with access to the BioCloud servers full control of the files. If you only want a specific group of people to have write access, there is only one way to do that, which is to contact the university IT services to create an email address group for the specific users, and then follow the same steps below, but instead use the new email of that group.

**Never** use `chmod 777`. It's a major security weakness that can allow anyone (even users outside of the university authentication system, human or not) to do anything they want with a file/folder and potentially place hidden payload there. Folders will be scanned regularly for insecure permissions and corrected without notice. Only `owner` and `group` should have the execute bit set, never `other`. See [permissions in Linux](https://www.geeksforgeeks.org/permissions-in-linux/) to learn about file and folder permissions on Linux systems.

```
folder="some_folder/"

# create the folder if it doesn't exist already
mkdir -p "$folder"

# set group ownership
chown -R :bio_server_users@bio.aau.dk "$folder"

# If there are already files with weak permissions, correct them with:
chmod -R o-x "$folder"

# Ensure group can edit files/folders:
find "$folder" -type f -exec chmod 664 {} \;
find "$folder" -type d -exec chmod 775 {} \;

# set the setGID sticky bit to ensure new files and folders inherit group ownership
chmod 2775 "$folder"
```

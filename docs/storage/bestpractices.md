# Best practices with Ceph storage

### Moving large amounts of data around
If you need to move large amounts of data (or numerous files at once regardless of size) it will happen in an instant regardless of the size if you make sure you are doing it within the same [mount point](intro.md), for example from `/projects/xxx` to `/projects/yyy`. However, if you need to move things across mount points, for example from `/home/xxx` to `/projects/yyy` please ask an administrator to do it for you, since moving between mount points will happen over the network and cause unneccessary traffic and load on the storage cluster, let alone be very slow. An administrator can move anything instantly anywhere on the storage cluster.


???+ info "Use symbolic links instead of copying data"
      To avoid data duplication and unnecessary use of storage space, please use symbolic links (using the `ln -s` command) instead of copying data (using the `cp` command) whenever possible. This is especially important when working with large files. Symbolic links are simply pointers to the original file.


### Avoid numerous small files at all costs
**Whereever possible always try to write fewer but larger files instead of many small ones**. The Ceph storage cluster uses a file metadata cache server (MDS server) that holds an entry for every single open file, so the cache memory usage is directly proportional to the number of open files (being accessed in any way). If there are too many open files (across **ALL** connected clients/nodes at once) the metadata server will ask clients to release some pressure on the cache and if they fail to do so in time they will simply be [evicted](https://docs.ceph.com/en/reef/cephfs/eviction/) without notice (meaning the Ceph cluster will force an unmount to protect itself). This will happen on any node that tips the iceberg and is thus not a result of the exact storage actions of individual clients, but rather that of all nodes connected to the Ceph cluster at any one time. See [Ceph best usage practices](https://docs.ceph.com/en/reef/cephfs/app-best-practices/) for additional details.

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
      | Attribute | Explanation |
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


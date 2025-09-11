# Mount points
All nodes are connected to a 2PB [CephFS](https://ceph.io/en/discover/) network storage cluster through a few different mount points for different purposes. The same mount points are used everywhere, so everything will have the same locations regardless of which node you are accessing from.

It is important to stress that the storage should be considered as **temporary** working storage only. It is **NOT** a long-term storage solution for cold data, so please clean up after yourself immediately after each job! See the [storage policy](policy.md) for more details.

???+ warning "Data is NOT backed up"
      Data is stored with 3x replication across multiple disks and storage nodes to protect against data loss or corruption due to hardware and/or disk failures. The data is **NOT backed up** anywhere, however, which means it's not protected against human errors. Therefore, **if you delete data - it's gone forever** and cannot be restored! 

???+ info "Max file size"
     There is a max file size of 4TB on the CephFS network storage cluster, which cannot be increased, [details here](https://docs.ceph.com/en/latest/cephfs/administration/#maximum-file-sizes-and-performance).
     
## Mount points

| Mount point | Contents |
| --- | --- |
| `/home` | Data for individual projects and generally most things should go here. |
| `/projects` | Data for shared projects where multiple people work on the same project(s) should go here. |
| `/databases` | **(read-only)** Various databases required for bioinformatic tools. |
| `/raw_data` | Raw data archive (mostly from DNA sequencing). |

???+ info "Large databases"
      To avoid data duplication and unnecessary use of storage space, please always ask an administrator to download large databases (100GB+), so that everyone else can also benefit from them. Do NOT store large databases in your home or project folders, unless you need to modify or customize them.

## Organization and naming of data
At a university people come and go all the time due to time-limited employments. It is therefore important to organize and name things conveniently, especially witin shared folders, to be able to easily identify who is responsible for what when people leave the university. Therefore, please always organize EVERYTHING under the `/raw_data/` and `/projects` mount points with the top-level structure:

`/<mountpoint>/PI/project/(sub-project)`

where **PI** is an acronym or the initials of the principal investigator (or research group leader) for the project, **project** is the full name of the project, and **sub-project** is optional and can be used if there are multiple sub-projects within a larger project.

Secondly, naming folders clearly and unambigously is very important, so please avoid using acronyms and abbreviations that only make sense to you. Instead, use full user- or project names, preferably including a date or at least a year, so that it's always clear who is responsible for what. For example:

```
/projects/John_Doe/2025-Microbiome_of_Danish_Water_Systems
/raw_data/Jane_Smith/Metagenomics_of_Soil_Fungi
```
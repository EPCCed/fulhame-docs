# Filesystems

Fulhame three filesystems: NFS (where ```/home``` and ```/opt``` are mounted), Lustre-SSD (mounted as ```/work-ssd```) and Lustre-HDD (mounted as ```/work-hdd```). All three are available on both compute and login nodes. None are backed up and reponsbility for maintaining copies of critical data lies with the user.

## NFS

The NFS filesystem is where ```/home``` is mounted and is available on both compute and login nodes. It is not a parallel file system so big builds and intensive IO jobs running against this filesystem will impact all others users very quickly.

User files are stored at:
```
/scratch3/home/<proj>/<proj>/<username>
```
Where ```<username>``` is your username and ```<proj>``` is the project code for your user (usually one of fh1..5 or mXXYY with XXYY representing the academic year group for students).

## Lustre-HDD

This filesystem runs Lustre on top of spinning disks (HDD). User files are stored at:
```
/work-hdd/<proj>/<proj>/<username>
```
Where ```<proj>``` and ```<username>``` have the same meaning as above.

## Lustre-SSD

This filesystem runs Lustre on top of solid state disks (SSD). User files are stored at:
```
/work-ssd/<proj>/<proj>/<username>
```
Where ```<proj>``` and ```<username>``` have the same meaning as above.

##  Notes on Lustre

Fulhame was the first production-ready HPC cluster to run a fully homogenous Lustre system on ARM processors. Both client (compute/login nodes) and server (MDS/MGS/OST/OSS) run on nodes with the same configuration as the compute nodes.
# Volume Shadow Copy with EXPRESSCLUSTER

VSS (Volume Shadow Copy) is one of the features of Windows to take snapshots of a drive. It enables to guarantee the consistency of application data across backup operation.
User takes snapshots of a drive (like x: drive) and can retrieve files from any snapshots taken in the past.
However, in the use with EC, the snapshot taken at node#1 could not be referenced at node#2 after failover.  
It was found that the snapshots could be referenced after the failover if the data partition of MD (mirror disk) resource was dismounted then mounted. This document describes how to implement the solution which enables the reference to VSS snapshots on a MD resource.

----

## System configuration

The configuration is "2 nodes data-mirroring type cluster" has tested. shared-disk and hybrid-disk type can be tested (expecting the same way is applicable).  
The following version of OS and EXPRESSCLUSTER are evaluated.

  - Windows Server 2012 R2 / 2016 / 2019
  - EXPRESSCLUSTER X 3.2 / 3.3 / 4.0 / 4.1

  ```
  LAN
   |     +------------------------+
   +-----+ Node#1                 |
   |     | - Windows Server 2019  |
   |  +--+ - EXPRESSCLUSTER X 4.1 |
   |  |  +------------------------+
   |  |
   |  |  +------------------------+
   +--|--+ Node#2                 |
   |  |  | - Windows Server 2019  |
   |  +--+ - EXPRESSCLUSTER X 4.1 |
   |     +------------------------+
  ```

## Setup steps

- Prepare a cluster to have MD resource
- Add a Script Resource and configure it to depend on the MD resource and edit *start.bat* as following sample
  - In the sample,  *x:\\*  represents the mount point (drive letter) of the MD resource.

	```
	rem ***************************************
	rem *              start.bat              *
	rem ***************************************

	rem Check startup attributes
	IF "%CLP_EVENT%" == "RECOVER" GOTO EXIT

	rem ***************************************
	rem Parameter

	rem Mount point for MD/SD/HD resrouce
	set mnt=x:\
	rem ***************************************

	for /f "usebackq" %%i in (`mountvol %mnt% /l`) do @set volname=%%i
	mountvol %mnt% /p
	mountvol %mnt% %volname%

	:EXIT
	```

## How to test
1. Start the failover group on node#1.
1. Update some files and take a snapshot.
1. Move the failover group to node#2.
1. Check if the snapshot taken on node#1 is available.
1. Update some files and take a snapshot.
1. Shutdown node#2.
1. Check if the snapshot taken on node#2 is available on node#1 after failover.

<!--
## Enable Shadow Copies with EXPRESSCLUSTER
Enable shadow copies with a Disk/Hybrid Disk/Mirror Disk Resource of EXPRESSCLUSTER to take over a history of shadow copies. For more detail, please check [EnableShadowCopies.md](https://github.com/EXPRESSCLUSTER/Volume-Shadow-Copy-Service/blob/master/EnableShadowCopies.md).
-->

## References

- [Best Practices for Shadow Copies of Shared Folders](http://technet.microsoft.com/en-us/library/cc753975.aspx)
- [How to: VSS Tracing](https://docs.microsoft.com/en-us/archive/blogs/supportingwindows/how-to-vss-tracing)


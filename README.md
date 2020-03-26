# Volume Shadow Copy with EXPRESSCLUSTER

VSS (Volume Shadow Copy) is one of the features of Windows to take snapshots of a drive. It enables to guarantee the consistency of application data across backup operation.
User takes snapshots of a drive (like x: drive) and can retrieve files from any snapshots taken in the past.
However, in the use with EC, the snapshot taken at node#1 could not be referenced at node#2 after failover.  
It was found that the snapshots could be referenced after the failover if the data partition of MD (mirror disk) resource was dismounted then mounted. This document describes how to implement the solution which enables the reference to VSS snapshots on a MD resource.

----

## System configuration

The configuration is "2 nodes data-mirroring type cluster".  
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

- Install EC on Windows
- Configure a cluster to have MD resource
- Obtain the *volume name* for the data partition of MD resource from both nodes 

	e.x. *x:* is drive letter for the data partition of the MD resource.  
	start the failover group on node#1. Open cmd.exe on node#1

	```
	C:\> mountvol x:\ /l
	\\?\Volume{c348f10a-0000-0000-0000-104000000000}\
	```

	move the failover group to node#2, open cmd.exe on node#2

	```
	C:\> mountvol x:\ /l
	\\?\Volume{ea5b5c1c-0000-0000-0000-104000000000}\
	```


- Add a Script Resource

  Configure it to depend on the MD resource and edit start.bat as following sample

	- *x:* is data-partition of the MD resource. 
	- *\\\\?\\Volume{\*}\\* are volume-name which were obtained in above. The upper one is that of node#1 and lowere one is that of node#2

	```
	rem ***************************************
	rem *              start.bat              *
	rem ***************************************
	
	IF "%CLP_EVENT%" == "RECOVER" GOTO EXIT

	mountvol x:\ /p

	IF "%CLP_SERVER%" == "OTHER" GOTO ON_OTHER1
	
	mountvol x:\ \\?\Volume{c348f10a-0000-0000-0000-104000000000}\
	GOTO EXIT
	
	:ON_OTHER1
	mountvol x:\ \\?\Volume{ea5b5c1c-0000-0000-0000-104000000000}\
	GOTO EXIT
	
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


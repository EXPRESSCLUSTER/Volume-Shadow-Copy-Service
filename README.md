# Volume-Shadow-Copy-Service
## Index
* [Disclaimer](#disclaimer)
* [About This Page](#about-this-page)
* [Evaluation Environment](#evaluation-environment)
* [Prerequisites for EXPRESSCLUSTER X](#prerequisites-for-expresscluster-x)
* [Install EXPRESSCLUSTER X](#install-expresscluster)
* [Create a Cluster](#create-a-cluster)
  * [Save Data and Snapshots on the Same Volume](#save-data-and-snapshots-on-the-same-volume) 
  * [Save Data and Snapshots on Different Volumes](#save-data-and-snapshots-on-different-volumes) 
* [Add a Volume Shadow Copy Storage](#add-a-volume-shadow-copy-storage)
* [Edit the Script Resource to Take Over Snapshots](#edit-the-script-resource-to-take-over-snapshots)
* [Failover Test](#failover-test)

<!---
* [Appendix](#appendix)
  * [Notes](#notes)
-->

## Disclaimer
**The contents of this document are subject to change without notice. The author assumes no esponsibility for technical or editorial mistakes in or omissions from this clustering procedure.**

## About This Page
This page describes how to enable shadow copies with EXPRESSCLUSTER X.

## Evaluation Environment
The following version of OS and EXPRESSCLUSTER are evaluated.
- OS
  - Windows Server 2016 
  - Windows Server 2012 R2
- EXPRESSCLUSTER
  -  EXPRESSCLUSTER X 4.0
  -  EXPRESSCLUSTER X 3.3
  -  EXPRESSCLUSTER X 3.2
<!--
```
+--------------------------+
| Active Directory         |
| - Windows Server 2012 R2 |
+--------------------------+
 |
 |  +--------------------------+
 +--| Cluster Node #1          |
 |  | - Windows Server 2012 R2 |
 |  | - EXPRESSCLUSTER X 3.2   |
 |  +--------------------------+
 |
 |  +--------------------------+
 +--| Cluster Node #2          |
 |  | - Windows Server 2012 R2 |
 |  | - EXPRESSCLUSTER X 3.2   |
 |  +--------------------------+
 |
 |  +--------------------------+
 +--| Client Machine           |
    | - Windows Server 2012 R2 |
    +--------------------------+
```
-->
## Prerequisites for EXPRESSCLUSTER X
If you use **Mirror Disk Resource** or **Hybrid Disk Resource**, you have to install **EXPRESSCLUSTER X 2.1 or later**. For more information, please visit [NEC website](http://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html?) and refer to **Getting Started Guide** and **Installation and Configuration Guide**.

|Resource                      |Supported Version            |
|:-----------------------------|:----------------------------|
|Disk Resource (sd)            |EXPRESSCLUSTER X 1.0 or later|
|Mirror Disk Resource (md)     |EXPRESSCLUSTER X 2.1 or later|
|Hybrid Disk Resource (hd)     |EXPRESSCLUSTER X 2.1 or later|

## Install EXPRESSCLUSTER
1. Install EXPRESSCLUSTER on both the primary and secondary node following **Installation and Configuration Guide**.
1. Register the licenses.
1. Restart both nodes.

## Create a Cluster
Here is a sample configuration using a mirror disk resource. You can also use a disk resource and hybrid disk resource using the following steps.

### Save Data and Snapshots on the Same Volume
1. Setup a failover group.
1. Add a script resource.
   1. Right click the failover group and click **Add Resource**.
   1. Select **script resource** for **Type** and set a resource name (ex. script-vss1) for **Name**. Then, click **Next**.
   1. Uncheck the box of **Follow the default dependency** and click **Next**.
   1. Check **Retry Count** or **Failover Threshold** and click **Next**.
   1. Click **Finish**.
1. Add a mirror disk resource.
   1. Right click the failover group and click **Add Resource**.
   1. Select **mirror disk resource** for **Type** and set a resource name (ex. md1) for **Name**. Then, click **Next**.
   1. Uncheck the box of **Follow the default dependency**. Then, select the script resource (ex. script-vss1) and click **Add**. After that, click **Next**.
   1. Check **Retry Count** or **Failover Threshold** and click **Next**.
   1. Set a data partition (ex. M: drive) and a cluster partition and click **Finish**.
1. Add other resources if you need.

|Depth|Resource                                   |Name       |Remarks |
|:----|:------------------------------------------|:----------|:-------|
|0    |Script resource                            |script-vss1|NULL    |
|1    |Mirror disk resource for data and snapshots|md1        |M: drive|

### Save Data and Snapshots on Different Volumes
<!---
**Need to fix the procedure. For detail, please refer to https://github.com/fukunagt/Volume-Shadow-Copy-Service/issues/1 .**
-->
1. Setup a failover group.
1. Add a script resource.
   1. Right click the failover group and click **Add Resource**.
   1. Select **script resource** for **Type** and set a resource name (ex. script-vss1) for **Name**. Then, click **Next**.
   1. Uncheck the box of **Follow the default dependency** and click **Next**.
   1. Check **Retry Count** or **Failover Threshold** and click **Next**.
   1. Click **Finish**.
1. Add a mirror disk resource to save snapshots.
   1. Right click the failover group and click **Add Resource**.
   1. Select **mirror disk resource** for **Type** and set a resource name (ex. md-vss1) for **Name**. Then, click **Next**.
   1. Uncheck the box of **Follow the default dependency**. Then, select the script resource (ex. script-vss1) and click **Add**. After that, click **Next**.
   1. Check **Retry Count** or **Failover Threshold** and click **Next**.
   1. Set a data partition (ex. M: drive) and a cluster partition and click **Finish**.
1. Add a mirror disk resource to save data.
   1. Right click the failover group and click **Add Resource**.
   1. Select **mirror disk resource** for **Type** and set a resource name (ex. md-data1) for **Name**. Then, click **Next**.
   1. Uncheck the box of **Follow the default dependency**. Then, select the mirror disk resource to save snapshots (ex. md-vss1) and click **Add**. After that, click **Next**.
   1. Check **Retry Count** or **Failover Threshold** and click **Next**.
   1. Set a data partition (ex. N: drive) and a cluster partition and click **Finish**.
1. Add other resources if you need.

|Depth|Resource                          |Name       |Remarks |
|:----|:---------------------------------|:----------|:-------|
|0    |Script resource                   |script-vss1|NULL    |
|1    |Mirror disk resource for snapshots|md-vss1    |M: drive|
|2    |Mirror disk resource for data     |md-data1   |N: drive|

* **WARNING:** If the only md-vss1 has been activated for **30 minutes**, Volume Shadow Copy histories will be deleted. In this configuration, the both md-vss1 and md-data1 should be running simultaneously.

## Add a Volume Shadow Copy Storage
1. Start the failover group on the primary server.
1. Add a Volume Shadow Copy Storage using GUI or vssadmin command. If you use vssadmin command, you need to add a schedule for Volume Shadow Copy manually.
1. Move the failover group to the secondary server.
1. Add a Volume Shadow Copy Storage using GUI or vssadmin command, again.

Ex. Save data and snapshots on the same volume (M: drive)
```bat
C:\> vssadmin add shadowstorage /For=M: /On=M: /MaxSize=UNBOUNDED
```

Ex. Save data and snapshots on different volumes (M: and N: drive)
```bat
C:\> vssadmin add shadowstorage /For=N: /On=M: /MaxSize=UNBOUNDED
```

## Edit the Script Resource to Take Over Snapshots
1. Right click the script resource (ex. script-vss1) and click **Properties**.
1. Click **Details** tab.
1. Select **stop.bat** and **Edit** button.
1. Add following lines on the bottom of the script.

Ex. Save data and snapshots on the same volume (M: drive)
```bat
vssadmin list shadows
armsleep 1
vssadmin list shadows
armsleep 1
mountvol M: /P
```

Ex. Save data and snapshots on different volumes (M: and N: drive)
```bat
vssadmin list shadows
armsleep 1
vssadmin list shadows
armsleep 1
mountvol N: /P
mountvol M: /P
```

## Failover Test
1. Start the failover group on the primary server.
1. Update some files and take a snapshot.
1. Move the failover group to the secondary server.
1. Check if the snapshot you took on the primary server is available.
1. Update some files and take a snapshot, again.
1. Shutdown the secondary server.
1. Check if the snapshot you took on the secondary server is available.

<!---
## Appendix
### Notes 
FIXME
* If you save data and snapshots on different volumes, you may not get **Shadow Copy Volume** with **vssadmin** command. However, you can see it with GUI.

```
Contents of shadow copy set ID: {a976aee3-ffcf-4902-82cd-ca93ad7aa380}
   Contained 1 shadow copies at creation time: 4/16/2015 5:30:56 PM
      Shadow Copy ID: {6b0dde5b-4830-494a-8a74-268f42ffa613}
         Original Volume: (P:)\\?\Volume{ac25e9d6-b365-11e4-80e0-00155d459086}\
         Shadow Copy Volume: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy397
:
```
--->

{id="chap:4"}
# Managing a Hadoop Cluster 

In this chapter, we will cover:

- Managing HDFS cluster
- Configuring SecondaryNameNode
- Managing MapReduce cluster
- Managing TaskTracker
- Decommissioning DataNode
- Replacing a slave node
- Managing MapReduce jobs
- Checking job history from the web UI
- Importing data to HDFS
- Manipulating files on HDFS
- Configuring HDFS quota
- Configuring CapacityScheduler
- Configuring fair scheduler
- Configuring Hadoop daemon logging
- Configuring Hadoop audit logging
- Upgrading Hadoop

## Introduction 

From the perspective of functionality, a Hadoop cluster is composed of
an HDFS cluster and a MapReduce cluster. The HDFS cluster consists of
the default filesystem for Hadoop. It has one or more NameNodes to keep
track of the filesystem metadata, while actual data blocks are stored on
distributed slave nodes managed by DataNode. Similarly, a MapReduce
cluster has one JobTracker on the master node and a number of
TaskTrackers on the slave nodes. The JobTracker manages the life cycle
of MapReduce jobs. It splits jobs into smaller tasks and schedules the
tasks to run by the TaskTrackers. A TaskTracker executes tasks assigned
by the JobTracker in parallel by forking one or a number of JVM
processes. As a Hadoop cluster administrator, you will be responsible
for managing both the HDFS cluster and the MapReduce cluster.

In general, system administrators should maintain the health and
availability of the cluster. More specifically, for a HDFS cluster, it
means the management of the NameNodes and DataNodes and the management
of the JobTrackers and TaskTrackers for MapReduce. Other administrative
tasks include the management of Hadoop jobs, for example configuring job
scheduling policy with schedulers.

At the end of this chapter, we will cover topics for configuring Hadoop
logging and doing system upgrade. Logging provides insights for
diagnosing cluster failure or performance problems, and system upgrade
plays an important role in keeping the software up to date.

### Managing HDFS cluster 

The health of HDFS is critical for a Hadoop-based Big Data platform.
HDFS problems can negatively affect the efficiency of the cluster. Even
worse, it can make the cluster not function properly. For example,
DataNode unavailability caused by network segmentation can lead to some
under-replicated data blocks. When this happens, HDFS will automatically
replicate those data blocks, which will bring a lot of overhead to the
cluster and cause the cluster to be too unstable to be available for
use. In this recipe, we will show commands to manage a HDFS cluster.

### Getting ready 

Before getting started, we assume that our Hadoop cluster has been
properly configured and all the daemons are running without any
problems. Log in to the master node from the administrator machine with
the following command: ssh hduser@master

### How to do it... 

Use the following steps to check the status of a HDFS cluster with
`hadoop fsck`:

Check the status of the root filesystem with the following command:

    $ hadoop fsck /
    FSCK started by hduser from /10.147.166.55 for path / at Thu Feb 28 17:14:11 EST 2013
    ..
    /user/hduser/.staging/job_201302281211_0002/job.jar:  Under replicated blk_-665238265064328579_1016. Target Replicas is 10 but found 5 replica(s).
    .................................Status: HEALTHY
     Total size:    14420321969 B
     Total dirs:    22
     Total files:   35
     Total blocks (validated):      241 (avg. block size 59835360 B)
     Minimally replicated blocks:   241 (100.0 %)
     Over-replicated blocks:        0 (0.0 %)
     Under-replicated blocks:       2 (0.8298755 %)
     Mis-replicated blocks:         0 (0.0 %)
     Default replication factor:    2
     Average block replication:     2.0248964
     Corrupt blocks:                0
     Missing replicas:              10 (2.0491803 %)
     Number of data-nodes:          5
     Number of racks:               1
    FSCK ended at Thu Feb 28 17:14:11 EST 2013 in 28 milliseconds

    The filesystem under path '/' is HEALTHY

The output shows that some percentage of data blocks is under
replicated. But because HDFS can automatically make duplication for
those data blocks, the HDFS filesystem and the `/` directory are both
`HEALTHY`.

Check the status of all the files on HDFS with the following command:

    $ hadoop fsck / -files
    FSCK started by hduser from /10.147.166.55 for path / at Thu Feb 28 17:40:35 EST 2013
    / <dir>
    /home <dir>
    /home/hduser <dir>
    /home/hduser/hadoop <dir>
    /home/hduser/hadoop/tmp <dir>
    /home/hduser/hadoop/tmp/mapred <dir>
    /home/hduser/hadoop/tmp/mapred/system <dir>
    /home/hduser/hadoop/tmp/mapred/system/jobtracker.info 4 bytes, 1 block(s):  OK
    /user <dir>
    /user/hduser <dir>
    /user/hduser/randtext <dir>
    /user/hduser/randtext/_SUCCESS 0 bytes, 0 block(s):  OK
    /user/hduser/randtext/_logs <dir>
    /user/hduser/randtext/_logs/history <dir>
    /user/hduser/randtext/_logs/history/job_201302281451_0002_1362090421087_hduser_random-text-writer 23995 bytes, 1 block(s):  OK
    /user/hduser/randtext/_logs/history/job_201302281451_0002_conf.xml 22878 bytes, 1 block(s):  OK
    /user/hduser/randtext/part-00001 1102231864 bytes, 17 block(s):  OK
    Status: HEALTHY

Hadoop will scan and list all the files in the cluster.

This command scans all files on HDFS and prints the size and status.

Check the locations of file blocks with the following command:

	$ hadoop fsck / -files -locations

The output will be similar to the following [figure](#fig:hdfs.block.locations). 

{id="fig:hdfs.block.locations"}
![Block locations on HDFS](images/5163os_04_07.png)

The first line tells us that file `part-00000` has 17 blocks in total
and each block has 2 replications (replication factor has been set to
2). The following lines list the location of each block on the DataNode.
For example, block `blk_6733127705602961004_1127` has been replicated on
hosts `10.145.231.46` and `10.145.223.184`. The number `50010` is the port
number of the DataNode.

Check the locations of file blocks containing rack information with the
following command:

	$ hadoop fsck / -files -blocks -racks

Delete corrupted files with the following command:

	$ hadoop fsck -delete

Move corrupted files to `/lost+found` with the following command:

	$ hadoop fsck -move

Use the following steps to check the status of a HDFS cluster with
`hadoop dfsadmin`. 

Report the status of each slave node with the following command:

    $ hadoop dfsadmin -report
    Configured Capacity: 422797230080 (393.76 GB)
    Present Capacity: 399233617920 (371.82 GB)
    DFS Remaining: 388122796032 (361.47 GB)
    DFS Used: 11110821888 (10.35 GB)
    DFS Used%: 2.78%
    Under replicated blocks: 0
    Blocks with corrupt replicas: 0
    Missing blocks: 0

    -------------------------------------------------
    Datanodes available: 5 (5 total, 0 dead)

    Name: 10.145.223.184:50010
    Decommission Status : Normal
    Configured Capacity: 84559446016 (78.75 GB)
    DFS Used: 2328719360 (2.17 GB)
    Non DFS Used: 4728565760 (4.4 GB)
    DFS Remaining: 77502160896(72.18 GB)
    DFS Used%: 2.75%
    DFS Remaining%: 91.65%
    Last contact: Thu Feb 28 20:30:11 EST 2013

    ...

The first section of the output shows the summary of the HDFS cluster,
including the configured capacity, present capacity, remaining capacity,
used space, number of under replicated data blocks, number of data
blocks with corrupted replicas, and number of missing blocks.

The following sections of the output information show the status of each
HDFS slave node, including the name (`ip:port`) of DataNode machine,
commission status, configured capacity, HDFS and non-HDFS used space
amount, HDFS remaining space, and the time that the slave node contacted
the master.

Refresh all the DataNodes using the following command:

	$ hadoop dfsadmin -refreshNodes

Check the status of the safe mode using the following command:

    $ hadoop dfsadmin -safemode get
    Safe mode is OFF

The output tells us that the NameNode is not in safe mode. In this case,
the filesystem is both readable and writable. If the NameNode is in safe
mode, the filesystem will be read-only (write protected).

Manually put the NameNode into safe mode using the following command:

	$ hadoop dfsadmin -safemode enter

This command is useful for system maintenance.

Make the NameNode to leave safe mode using the following command:

	$ hadoop dfsadmin -safemode leave

If the NameNode has been in safe mode for a long time or it has been put
into safe mode manually, we need to use this command to let the NameNode
leave this mode.

Wait until NameNode leaves safe mode using the following command:

	$ hadoop dfsadmin -safemode wait
	
This command is useful when we want to wait until HDFS finishes data
block replication or wait until a newly commissioned DataNode to be
ready for service.

Save the metadata of the HDFS filesystem with the following command:

	$ hadoop dfsadmin -metasave meta.log

The meta.log file will be created under the directory
`$HADOOP_HOME/logs`. Its content will be similar to the following:

    21 files and directories, 88 blocks = 109 total
    Live Datanodes: 5
    Dead Datanodes: 0
    Metasave: Blocks waiting for replication: 0
    Metasave: Blocks being replicated: 0
    Metasave: Blocks 0 waiting deletion from 0 datanodes.
    Metasave: Number of datanodes: 5
    10.145.223.184:50010 IN 84559446016(78.75 GB) 2328719360(2.17 GB) 2.75% 77502132224(72.18 GB) Thu Feb 28 21:43:52 EST 2013
    10.152.166.137:50010 IN 84559446016(78.75 GB) 2357415936(2.2 GB) 2.79% 77492854784(72.17 GB) Thu Feb 28 21:43:52 EST 2013
    10.145.231.46:50010 IN 84559446016(78.75 GB) 2048004096(1.91 GB) 2.42% 77802893312(72.46 GB) Thu Feb 28 21:43:54 EST 2013
    10.152.161.43:50010 IN 84559446016(78.75 GB) 2250854400(2.1 GB) 2.66% 77600096256(72.27 GB) Thu Feb 28 21:43:52 EST 2013
    10.152.175.122:50010 IN 84559446016(78.75 GB) 2125828096(1.98 GB) 2.51% 77724323840(72.39 GB) Thu Feb 28 21:43:53 EST 2013
    21 files and directories, 88 blocks = 109 total
    ...

### How it works... 

The HDFS filesystem will be write-protected when NameNode enters safe
mode. When an HDFS cluster is started, it will enter safe mode first.
The NameNode will check the replication factor for each data block. If
the number of replicas for a data block is less than the configured
replication factor, which is **3** by default, the data block will be
marked as under replicated. Finally, an under replication factor, which
is the percentage of under replicated data blocks, will be calculated.
If the percentage number is larger than the threshold value, the
NameNode will stay in safe mode until enough new replicas are created
for the under replicated data blocks so as to make the under replication
factor lower than the threshold.

We can get the usage of the fsck command using:

    $ hadoop fsck
    Usage: DFSck <path> [-move | -delete | -openforwrite] [-files [-blocks [-locations | -racks]]]
            <path>  start checking from this path
            -move   move corrupted files to /lost+found
            -delete delete corrupted files
            -files  print out files being checked
            -openforwrite   print out files opened for write
            -blocks print out block report
            -locations      print out locations for every block
            -racks print out network topology for data-node locations
             By default fsck ignores files opened for write, use -openforwrite to report such files. They are usually tagged CORRUPT or HEALTHY depending on their block allocation status.

We can get the usage of the `dfsadmin` command using:

    $ hadoop dfsadmin
    Usage: java DFSAdmin
               [-report]
               [-safemode enter | leave | get | wait]
               [-saveNamespace]
               [-refreshNodes]
               [-finalizeUpgrade]
               [-upgradeProgress status | details | force]
               [-metasave filename]
               [-refreshServiceAcl]
               [-refreshUserToGroupsMappings]
               [-refreshSuperUserGroupsConfiguration]
               [-setQuota <quota> <dirname>...<dirname>]
               [-clrQuota <dirname>...<dirname>]
               [-setSpaceQuota <quota> <dirname>...<dirname>]
               [-clrSpaceQuota <dirname>...<dirname>]
               [-setBalancerBandwidth <bandwidth in bytes per second>]
               [-help [cmd]]

### There's more ... 

Besides using command line, we can use the web UI to check the status of
an HDFS cluster. For example, we can get the status information of HDFS
by opening the link [DFS health page](http://master:50070/dfshealth.jsp).

We will get a web page that shows the summary of the HDFS cluster such
as the configured capacity and remaining space. For example, the web
page will be similar to the following [figure](#fig:hdfs.summary).

{id="fig:hdfs.summary"}
![Summary of a HDFS cluster](images/5163os_04_08.png)

By clicking on the Live Nodes link, we can check the status of each
DataNode. The web page is similar to the following
[figure](#fig:datanode.status). 

{id="fig:datanode.status"}
![HDFS DataNode status](images/5163os_04_09.png)

By clicking on the link of each node, we can browse the directory of the
HDFS filesystem. The web page will be similar to the following
[figure](#fig:content.hdfs). 

{id="fig:content.hdfs"}
![Content of an HDFS directory](images/5163os_04_10.png)

The web page shows that file `/user/hduser/randtext` has been split into
five partitions. We can browse the content of each partition by clicking
on the **part-0000x** link.

### See also 

- The Validating Hadoop installation recipe in [Chapter 3](#chap:3),
  Configuring a Hadoop cluster
- The Decommissioning DataNode recipe
- The Manipulating files on HDFS recipe

## Configuring SecondaryNameNode

Hadoop NameNode is a single point of failure. By configuring
SecondaryNameNode, the filesystem image and edit log files can be backed
up periodically. And in case of NameNode failure, the backup files can
be used to recover the NameNode. In this recipe, we will outline steps
to configure SecondaryNameNode.

### Getting ready 

We assume that Hadoop has been configured correctly.

Log in to the master node from cluster administration machine using the
following command:

	$ ssh hduser@master

### How to do it... 

Perform the following steps to configure SecondaryNameNode:

Stop the cluster using the following command:

	$ stop-all.sh

Add or change the following into the file
`$HADOOP_HOME/conf/hdfs-site.xml`:

```xml
<property>
    <name>fs.checkpoint.dir</name>
    <value>/hadoop/dfs/namesecondary</value>
</property>
```

If this property is not set explicitly, the default checkpoint directory
will be `${hadoop.tmp.dir}/dfs/namesecondary`.

Start the cluster using the following command:

	$ start-all.sh

The tree structure of NameNode data directory will be similar to the
following:

    .
    |-- current
    |   |-- VERSION
    |   |-- edits
    |   |-- fsimage
    |   `-- fstime
    |-- image
    |   `-- fsimage
    |-- in_use.lock
    `-- previous.checkpoint
        |-- VERSION
        |-- edits
        |-- fsimage
        `-- fstime

    3 directories, 10 files

The tree structure of the SecondaryNameNode will be similar to that of
the NameNode.

### There's more... 

To increase redundancy, we can configure NameNode to write filesystem
metadata on multiple locations. For example, we can add an NFS shared
directory for backup by changing the following property in the file
`$HADOOP_HOME/conf/hdfs-site.xml`:

```xml
<property>
  <name>dfs.name.dir</name>
  <value>/hadoop/dfs/name,/nfs/name</value>
</property>
```

`/nfs/name` is an NFS shared directory on a remote machine.

### See also 

- The Managing HDFS cluster recipe
- The Decommissioning DataNode recipe

## Managing the MapReduce cluster

A typical MapReduce cluster is composed of one master node that runs the
JobTracker and a number of slave nodes that run TaskTrackers. The task
of managing a MapReduce cluster includes maintaining the health as well
as the membership between TaskTrackers and the JobTracker. In this
recipe, we will outline commands to manage a MapReduce cluster.

### Getting ready 

We assume that the Hadoop cluster has been properly configured and
running. Log in to the master node from the cluster administration
machine using the following command: `ssh hduser@master`.

### How to do it... 

Perform the following steps to manage a MapReduce cluster:

List all the active TaskTrackers using the following command:

	$ hadoop -job -list-active-trackers
	
This command can help us check the registration status of the
TaskTrackers in the cluster.

Check the status of JobTracker safe mode using the following command:

    $ hadoop mradmin -safemode get
    Safe mode is OFF

The output shows that the JobTracker is not in safe mode. We can
submit jobs to the cluster. If the JobTracker is in safe mode, no jobs
can be submitted to the cluster.

Manually let the JobTracker enter safe mode using the following
command:

	$ hadoop mradmin -safemode enter

This command is handy when we want to maintain the cluster.

Let the JobTracker leave safe mode using the following command:

	$ hadoop mradmin -safemode leave

When maintenance tasks are done, you need to run this command.

If we want to wait for safe mode to exit, the following command can be
used:

	$ hadoop mradmin -safemode wait

Reload the MapReduce queue configuration using the following command:

	$ hadoop mradmin -refreshQueues

Reload active TaskTrackers using the following command:

	$ hadoop mradmin -refreshNodes

### How it works... 

Get the help for the mradmin command using the following command:

    $ hadoop mradmin
    The usage information will be similar to the following:
    Usage: java MRAdmin
               [-refreshServiceAcl]
               [-refreshQueues]
               [-refreshUserToGroupsMappings]
               [-refreshSuperUserGroupsConfiguration]
               [-refreshNodes]
               [-safemode <enter | leave | get | wait>]
               [-help [cmd]]
    ...

The meaning of the command options is listed below:

- `-refreshServiceAcl` Force JobTracker to reload service ACL.
- `-refreshQueues` Force JobTracker to reload queue configurations.
- `-refreshUserToGroupsMappings` Force JobTracker to reload user group mappings.
- `-refreshSuperUserGroupsConfiguration` Force JobTracker to reload super user group mappings.
- `-refreshNodes` Force JobTracker to refresh the JobTracker hosts.
- `-help [cmd]` Show the help info for a command or all commands.

### See also 

- The Configuring SecondaryNameNode recipe
- The Managing MapReduce jobs recipe

## Managing TaskTracker

TaskTrackers are MapReduce daemon processes that run on slave nodes.
They accept tasks assigned by the JobTracker on the master node and fork
JVM processes/threads to run the tasks. TaskTracker is also responsible
for reporting the progress of the tasks as well as its health status
using heartbeat.

Hadoop maintains three lists for TaskTrackers: blacklist, gray list, and
excluded list. TaskTracker black listing is a function that can
blacklist a TaskTracker if it is in an unstable state or its performance
has been downgraded. For example, when the ratio of failed tasks for a
specific job has reached a certain threshold, the TaskTracker will be
blacklisted for this job. Similarly, Hadoop maintains a gray list of
nodes by identifying potential problematic nodes.

Sometimes, excluding certain TaskTrackers from the cluster is desirable.
For example, when we debug or upgrade a slave node, we want to separate
this node from the cluster in case it affects the cluster. Hadoop
supports the live decommission of a TaskTracker from a running cluster.

### Getting ready 

We assume that Hadoop has been properly configured. MapReduce and HDFS
daemons are running without any issues.

Log in to the cluster master node from the administrator machine using
the following command:

	$ ssh hduser@master

List the active trackers with the following command on the master node:

    $ hadoop job -list-active-trackers
    And the output should be similar to the following:
    tracker_slave5:localhost/127.0.0.1:55590
    tracker_slave1:localhost/127.0.0.1:47241
    tracker_slave3:localhost/127.0.0.1:51187
    tracker_slave4:localhost/127.0.0.1:60756
    tracker_slave2:localhost/127.0.0.1:42939

### How to do it... 

Perform the following steps to configure the heartbeat interval:

Stop a MapReduce cluster with the following command:

	$ stop-dfs.sh

Open the file `$HADOOP_HOME/conf/mapred-site.xml` with your favorite
text editor and add the following content into the file:

```xml
<property>
  <name>mapred.tasktracker.expiry.interval</name>
  <value>600000</value>
</property>
```

The value is in milliseconds.

Copy the configuration into the slave nodes using the following command:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo 'Copying mapred-site.xml to slave node ' $host
		sudo scp $HADOOP_HOME/conf/mapred-site.xml hduser@$host:$HADOOP_HOME/conf
	done

Start the MapReduce cluster with the following command:

	$ start-mapred.sh

Perform the following steps to configure TaskTracker blacklisting:

Stop the MapReduce cluster with the following command:

	$ stop-mapred.sh

Set the number of task failures for a job to blacklist a TaskTracker by
adding or changing the following property in the file
`$HADOOP_HOME/conf/hdfs-site.xml`:

```xml
<property>
  <name>mapred.max.tracker.failures</name>
  <value>10</value>
</property>
```

Set the maximum number of successful jobs that can blacklist a
TaskTracker by adding or changing the following property in the file
`$HADOOP_HOME/conf/hdfs-site.xml`:

```xml
<property>
  <name>mapred.max.tracker.blacklists</name>
  <value>5</value>
</property>
```

Copy the configuration file to the slave nodes using the following
commands:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo 'Copying hdfs-site.xml to slave node ' $host
		sudo scp $HADOOP_HOME/conf/hdfs-site.xml hduser@$host:$HADOOP_HOME/conf
	done

Start the MapReduce cluster using the following command:

	$ start-mapred.sh

List blacklisted TaskTrackers using the following command:

	$ hadoop job -list-blacklisted-trackers

Perform the following steps to decommission TaskTrackers:

Set the TaskTracker exclude file by adding the following properties into
the file:

```xml
<property>
  <name>mapred.hosts.exclude</name>
  <value>$HADOOP_HOME/conf/mapred-exclude.txt</value>
</property>
```

The `$HADOOP_HOME/conf/mapred-exclude.txt` file will contain the
excluding TaskTracker hostnames one per line. For example, the file
should contain the following two lines if we want to exclude `slave1`
and `slave3` from the cluster:

    slave1
    slave3

Force the JobTracker to reload the TaskTracker list with the following
command:

	$ hadoop mradmin -refreshNodes

List all the active trackers again using the following command:

    $ hadoop job -list-active-trackers
    tracker_slave5:localhost/127.0.0.1:55590
    tracker_slave4:localhost/127.0.0.1:60756
    tracker_slave2:localhost/127.0.0.1:42939

### How it works... 

TaskTrackers on slave nodes contact the JobTracker on the master node
periodically. The interval between two consecutive contact
communications is called a heartbeat. More frequent heartbeat
configurations can incur higher load to the cluster. The value of the
heartbeat property should be set based on the size of the cluster.

The JobTracker uses TaskTracker blacklisting to remove those unstable
TaskTrackers. If a TaskTracker is blacklisted, all the tasks currently
running on the TaskTracker can still finish and the TaskTracker will
continue the connection with JobTracker through the heartbeat mechanism.
But the TaskTracker will not be scheduled for running future tasks. If a
blacklisted TaskTracker is restarted, it will be removed from the
blacklist.

The total number of blacklisted TaskTrackers should not exceed 50
percent of the total number of TaskTrackers.

### See also 

- The Managing the MapReduce cluster recipe
- The Managing MapReduce jobs recipe
- The Decommissioning DataNode recipe
- The Replacing a slave node recipe

## Decommissioning DataNode

Similar to TaskTracker, there are situations when we need to temporarily
disable a DataNode from the cluster, for example, because the storage
space of the DataNode has been used up. In this recipe, we will outline
steps to decommission a DataNode from a live Hadoop cluster.

### Getting ready 

We assume that our Hadoop has been configured properly.

Log in to the master node from the cluster administrator machine with
the following command:

	$ ssh hduser@master

For illustration purpose, we assume to decommission DataNode on host
slave1 from our running Hadoop cluster.

### How to do it... 

Perform the following steps to decommission a live DataNode:

Create the file `$HADOOP_HOME/conf/dfs-exclude.txt` with the following
content:

	slave1

The `dfs-exclude.txt` file contains the DataNode hostnames, one per
line, that are to be decommissioned from the cluster.

Add the following property to the file
`$HADOOP_HOME/conf/hdfs-site.xml`:

```xml
<property>
  <name>dfs.hosts.exclude</name>
  <value>$HADOOP_HOME/conf/dfs-exclude.txt</value>
</property>
```

Force the NameNode to reload the active DataNodes using the following
command:

	$ hadoop dfsadmin -refreshNodes

Get a description report of each active DataNode:

	$ hadoop dfsadmin -report

### How it works... 

Cluster administrators can use the dfsadmin command to manage the
DataNodes. We can get the usage of this command using the following:

    $ hadoop dfsadmin
    Usage: java DFSAdmin
               [-report]
               [-safemode enter | leave | get | wait]
               [-saveNamespace]
               [-refreshNodes]
               [-finalizeUpgrade]
               [-upgradeProgress status | details | force]
               ...

### See also 

- The Managing the HDFS cluster recipe
- The Configuring SecondaryNameNode recipe
- The Managing TaskTracker recipe
- The Replacing a slave node recipe

## Replacing a slave node

Sometimes, we need to replace a slave node with new hardware, because of
reasons such as the slave node is not stable, more storage space or more
powerful CPUs are desired, and so on. In this recipe, we will outline
the steps to replace a slave node.

### Getting ready 

We assume that replacement hardware is ready for use. And for
illustration purposes, we suppose `slave2` needs to be replaced in this
book.

### How to do it... 

Perform the following steps to replace a slave node:

Decommission the TaskTracker on the slave node with the steps outlined
in the Managing TaskTracker recipe of this chapter.

Decommission the DataNode on the slave node with the steps outlined in
the Decommission DataNode recipe of this chapter.

Power off the slave node and replace it with the new hardware.

Install and configure the Linux operating system on the new node with
the steps outlined in the Installing the Linux operating system,
Installing Java and other tools, and Configuring SSH recipes of Chapter
2, Preparing for Hadoop Installation.

Install Hadoop on the new node by copying the Hadoop directory and
configuration from the master node with the following commands:

	$ sudo scp -r /usr/local/hadoop-1.1.2 hduser@slave2:/usr/local/
	$ sudo ssh hduser@slave2 -C "ln -s /usr/local/hadoop-1.1.2 /usr/local/hadoop"
	$ sudo scp ~/.bashrc hduser@slave2:~/

Log in to `slave2` and start the DataNode and TaskTracker using the
following commands:

	$ ssh hduser@slave2 -C "hadoop DataNode &"
	$ ssh hduser@slave2 -C "Hadoop TaskTracker &"

Refresh the DataNodes with the following command:

	$ hadoop dfsadmin -refreshNodes

Refresh the TaskTracker with the following command:

	$ hadoop mradmin -refreshNodes

Report the status of the live DataNodes with the following command:

	$ hadoop dfsadmin -report

Get all the active TaskTrackers with the following command:

	$ hadoop job -list-active-trackers

### See also 

- The Installing Linux on node recipe of [Chapter 2](#chap:2), Preparing for
  Hadoop Installation 
- The Installing Java and other tools recipe of [Chapter 2](#chap:2), Preparing
  for Hadoop Installation
- The Configuring SSH recipe of [Chapter 2](#chap:2), Preparing for Hadoop
  installation
- The Configuring Hadoop in Fully-distributed Mode recipe of
  [Chapter 3](#chap:3), Configuring a Hadoop Cluster
- The Managing TaskTracker recipe
- The Decommissioning DataNode recipe

## Managing MapReduce jobs

The Hadoop Big Data platform accepts jobs submitted by clients. In a
multiuser environment, multiple jobs can be submitted and run
simultaneously. The tasks of managing Hadoop jobs include checking job
status, changing job priority, killing a running job, and so on. In this
recipe, we will outline the steps to do these job management tasks.

### Getting ready 

We assume that our Hadoop cluster has been configured properly and all
the Hadoop daemons are running without any issues. We also assume that a
regular user can submit Hadoop jobs to the cluster.

Log in to the master node from the cluster administrator machine with
the following command:

	$ ssh hduser@master

### How to do it... 

Perform the following steps to check the status of Hadoop jobs:

List all the running jobs using the following command:

    $ hadoop job -list
    1 jobs currently running
    JobId   State   StartTime   UserName        Priority        SchedulingInfo
    job_201302152353_0001   4   1361405786524   hduser  NORMAL  NA

The output message tells us that currently one job with JobId
`job_201302152353_0001` is running on the cluster.

List all the submitted jobs since the start of the cluster with the
following command:

    hadoop job -list all
    2 jobs submitted
    States are:
      Running : 1     Succeded : 2    Failed : 3      Prep : 4
    JobId   State   StartTime   UserName        Priority        SchedulingInfo
    job_201302152353_0001   2   1361405786524   hduser  NORMAL  NA
    job_201302152353_0002   4   1361405860611   hduser  NORMAL  NA

The State column of the output message shows the status of jobs. For
example, in the preceding output, two jobs have been submitted, the
first job with JobId `job_201302152353_0001` is in the succeeded state
and the second job with JobId `job_201302152353_0002` is in the
preparation state. Both jobs have normal priority and don't have
scheduling information.

We can check the status of the default queue with the following command:

    $ hadoop queue -list
    Queue Name : default
    Queue State : running
    Scheduling Info : N/A

Hadoop manages jobs using queues. By default, there is only one default
queue. The output of the command shows the cluster has only one default
queue, which in the running state with no scheduling information.

Check the status of a queue ACL with the following command:

    $ hadoop queue -showacls
    Queue acls for user :  hduser

    Queue  Operations
    =====================
    default  submit-job,administer-jobs

The output shows that the user hduser can submit and administer jobs in
the default queue.

Show all the jobs in the default queue using the following command:

    $ hadoop queue -info default -showJobs
    Queue Name : default
    Queue State : running
    Scheduling Info : N/A
    Job List
    JobId   State   StartTime   UserName   Priority   SchedulingInfo
    job_201302152353_0001   2   1361405786524   hduser  NORMAL  NA

Check the status of a job with the following command:

    $ hadoop job -status job_201302152353_0001
    Job: job_201302152353_0001
    file: hdfs://master:54310/user/hduser/.staging/job_201302152353_0001/job.xm                l
    tracking URL: http://master:50030/jobdetails.jsp?jobid=job_201302152353_000                1
    map() completion: 1.0
    reduce() completion: 1.0

    Counters: 31
            Job Counters
                    Launched reduce tasks=1
                    SLOTS_MILLIS_MAPS=87845
                    Total time spent by all reduces waiting after reserving slots (ms)=0
                    Total time spent by all maps waiting after reserving slots (ms)= 0
                    Rack-local map tasks=8
                    Launched map tasks=10
                    Data-local map tasks=2
                    SLOTS_MILLIS_REDUCES=16263
            File Input Format Counters
                    Bytes Read=1180
            File Output Format Counters
                    Bytes Written=97
            FileSystemCounters
                    FILE_BYTES_READ=226
                    HDFS_BYTES_READ=2440
                    FILE_BYTES_WRITTEN=241518
                    HDFS_BYTES_WRITTEN=215
            Map-Reduce Framework
                    Map output materialized bytes=280
                    Map input records=10
                    Reduce shuffle bytes=252
                    Spilled Records=40
                    Map output bytes=180
                    Total committed heap usage (bytes)=2210988032
                    CPU time spent (ms)=9590
                    Map input bytes=240
                    SPLIT_RAW_BYTES=1260
                    Combine input records=0
                    Reduce input records=20
                    Reduce input groups=20
                    Combine output records=0
                    Physical memory (bytes) snapshot=2033074176
                    Reduce output records=0
                    Virtual memory (bytes) snapshot=5787283456
                    Map output records=20

Change the status of a job by performing the following steps:

Set the job `job_201302152353_0001` to be on high priority using the
following command:

	$ hadoop job -set-priority job_201302152353_0003 HIGH

Available priorities, in descending order, include:
`VERY_HIGH, HIGH, NORMAL, LOW`, and `VERY_LOW`.

The priority of the job will be *HIGH* as shown in the following output:

    4 jobs submitted
    States are:
            Running : 1     Succeded : 2    Failed : 3      Prep : 4
    JobId   State   StartTime   UserName        Priority        SchedulingInfo
    job_201302152353_0001   2   1361405786524   hduser  NORMAL  NA
    job_201302152353_0002   2   1361405860611   hduser  NORMAL  NA
    job_201302152353_0003   1   1361408177470   hduser  HIGH    NA

Kill the job `job_201302152353_0004` using the following command:

	$ hadoop job -kill job_201302152353_0004

With the job status command, we will get the following output:

    3 jobs submitted
    States are:
     Running : 1   Succeded : 2  Failed : 3   Prep : 4   Killed : 5
    JobId   State   StartTime   UserName        Priority        SchedulingInfo
    job_201302152353_0001   2   1361405786524   hduser  NORMAL  NA
    job_201302152353_0002   2   1361405860611   hduser  NORMAL  NA
    job_201302152353_0003   1   1361408177470   hduser  HIGH    NA
    job_201302152353_0004   5   1361407740639   hduser  NORMAL  NA

The Killed : 5 information is not in the original output, I put it there
according to the state of the killed job `job_201302152353_0004`.

Perform the following steps to submit a MapReduce job:

Create the job configuration file, `job.xml`, with the following
content:

    <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <configuration>
      <property>
        <name>mapred.input.dir</name>
        <value>randtext</value>
      </property>

      <property>
        <name>mapred.output.dir</name>
        <value>output</value>
      </property>

      <property>
        <name>mapred.job.name</name>
        <value>wordcount</value>
      </property>

      <property>
        <name>mapred.mapper.class</name>
        <value>org.apache.hadoop.mapred.WordCount$Map</value>
      </property>

      <property>
        <name>mapred.combiner.class</name>
        <value>org.apache.hadoop.mapred.WordCount$Reduce</value>
      </property>

      <property>
        <name>mapred.reducer.class</name>
        <value>org.apache.hadoop.mapred.WordCount$Reduce</value>
      </property>

      <property>
        <name>mapred.input.format.class</name>
        <value>org.apache.hadoop.mapred.TextInputFormat</value>
      </property>

      <property>
        <name>mapred.output.format.class</name>
        <value>org.apache.hadoop.mapred.TextOutputFormat</value>
      </property>

    </configuration>


The `job.xml` file is an XML file that specifies the configuration of a
job. In this job configuration file, we specified the job name, the
mapper class, the reducer class, the combiner class, the input format,
and output format for the job. We have used wordcount as an example, so
we also need to make sure `$HADOOP_HOME/hadoop-examples*.jar` is
available in `CLASSPATH`.

Submit the job with the following command:

    $ hadoop job -submit job.xml
    13/03/01 11:55:53 WARN mapred.JobClient: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
    13/03/01 11:55:53 INFO util.NativeCodeLoader: Loaded the native-hadoop library
    13/03/01 11:55:53 WARN snappy.LoadSnappy: Snappy native library not loaded
    13/03/01 11:55:53 INFO mapred.FileInputFormat: Total input paths to process : 5
    Created job job_201302281451_0012

### How it works... 

The queue command is a wrapper command for the JobQueueClient class, and
the job command is a wrapper command for the JobClient class.

We can get the usage of the queue command with the following:

    $ hadoop queue
    Usage: JobQueueClient <command> <args>
            [-list]
            [-info <job-queue-name> [-showJobs]]
            [-showacls]

Similarly, we can get the usage of the job command with the following:

    $ hadoop job
    Usage: JobClient <command> <args>
            [-submit <job-file>]
            [-status <job-id>]
            [-counter <job-id> <group-name> <counter-name>]
            [-kill <job-id>]
            [-set-priority <job-id> <priority>]. Valid values for priorities are: VERY_HIGH HIGH NORMAL LOW VERY_LOW
            [-events <job-id> <from-event-#> <#-of-events>]
            [-history <jobOutputDir>]
            [-list [all]]
            [-list-active-trackers]
            [-list-blacklisted-trackers]
            [-list-attempt-ids <job-id> <task-type> <task-state>]
            [-kill-task <task-id>]
            [-fail-task <task-id>]

### There's more... 

We discussed the most useful commands for Hadoop job management.
Actually, there are even more commands that are related to job
management, and alternatively, we can use the web UI to manage Hadoop
jobs.

#### More job management commands 

Get the value of a counter:

	$ hadoop job -counter <job-id> <group-name> <counter-name>

For example, we can get the counter `HDFS_BYTES_WRITTEN` of the counter
group FileSystemCounters for the job `job_201302281451_0002` with the
following command:

	$ hadoop job -counter job_201302281451_0002 FileSystemCounters HDFS_BYTES_WRITTEN

Query events of a MapReduce job with the following command:

	$ hadoop job -events <job-id> <from-event-#> <#-of-events>

For example, we can query the first 10 events of the job
`job_201302281451_0002` using the following command:

	$ hadoop job -events job_201302281451_0002 0 10

The output will be similar to the [following figure](#fig:task.events).

{id="fig:task.events"}
![MapReduce task completion events](images/5163os_04_13.png)

Get the job history including job details, failed and killed jobs, and
so on with the following command:

    $hadoop job -history
    Hadoop job: 0012_1362156954465_hduser
    =====================================
    Job tracker host name: job
    job tracker start time: Tue May 18 17:18:01 EDT 1976
    User: hduser
    JobName: wordcount
    JobConf: hdfs://master:54310/user/hduser/.staging/job_201302281451_0012/job.xml
    Submitted At: 1-Mar-2013 11:55:54
    Launched At: 1-Mar-2013 11:55:54 (0sec)
    Finished At: 1-Mar-2013 11:56:43 (48sec)
    Status: FAILED
    Counters:

    |Group Name |Counter name|Map Value |Reduce Value |Total Value|
    -------------------------------------------------------
    =====================================

    Task Summary
    ============================
    Kind Total Successful Failed Killed StartTime   FinishTime

    Setup 1 1 0 0 1-Mar-2013 11:55:46  1-Mar-2013 11:55:49 (2sec)
    Map 45 0 38 7 1-Mar-2013 11:55:49  1-Mar-2013 11:56:34 (44sec)
    Reduce  0   0   0   0
    Cleanup 1 1 0 0 1-Mar-2013 11:56:33 1-Mar-2013 11:56:36 (3sec)
    ============================

    No Analysis available as job did not finish

    KILLED SETUP task list for 0012_1362156954465_hduser
    TaskId          StartTime       FinishTime      Error
    ====================================================

    FAILED MAP task list for 0012_1362156954465_hduser
    TaskId          StartTime       FinishTime      Error   InputSplits
    ====================================================
    task_201302281451_0012_m_000000 1-Mar-2013 11:55:58  1-Mar-2013 11:56:33 (35sec)  /default-rack/slave2,/default-rack/slave1

    ...
    FAILED task attempts by nodes
    Hostname        FailedTasks
    ===============================
    slave1  task_201302281451_0012_m_000000, task_201302281451_0012_m_000001, task_201302281451_0012_m_000002, task_201302281451_0012_m_000004, task_201302281451_0012_m_000005, task_201302281451_0012_m_000008, task_201302281451_0012_m_000010, task_201302281451_0012_m_000013,
    slave2  task_201302281451_0012_m_000000, task_201302281451_0012_m_000005, task_201302281451_0012_m_000008, task_201302281451_0012_m_000010, task_201302281451_0012_m_000019, task_201302281451_0012_m_000034, task_201302281451_0012_m_000036, task_201302281451_0012_m_000039,
    slave3  task_201302281451_0012_m_000000, task_201302281451_0012_m_000001, task_201302281451_0012_m_000002, task_201302281451_0012_m_000003, task_201302281451_0012_m_000004, task_201302281451_0012_m_000010, task_201302281451_0012_m_000012, task_201302281451_0012_m_000019,
    slave4  task_201302281451_0012_m_000000, task_201302281451_0012_m_000001, task_201302281451_0012_m_000003, task_201302281451_0012_m_000004, task_201302281451_0012_m_000010, task_201302281451_0012_m_000012, task_201302281451_0012_m_000013, task_201302281451_0012_m_000034,
    slave5  task_201302281451_0012_m_000002, task_201302281451_0012_m_000003, task_201302281451_0012_m_000005, task_201302281451_0012_m_000008, task_201302281451_0012_m_000012, task_201302281451_0012_m_000013,

    KILLED task attempts by nodes
    Hostname        FailedTasks
    ===============================
    slave1  task_201302281451_0012_m_000003, task_201302281451_0012_m_000012,
    slave2  task_201302281451_0012_m_000002, task_201302281451_0012_m_000013,
    slave3  task_201302281451_0012_m_000005,
    slave5  task_201302281451_0012_m_000001, task_201302281451_0012_m_000004,

## Managing tasks

We will show you how to kill tasks, check task attempts, and so on.

Kill a task with the following command:

	$ hadoop job -kill-task <task-id> 

For example, to kill the task `task_201302281451_0013_m_000000`, we can
use the following command:

	$ hadoop job -kill-task task_201302281451_0013_m_000000

After the task is killed, the JobTracker will restart the task on a
different node. The killed tasks can be viewed through the web UI as
shown in [figure](#fig:mapred.killed.tasks).

{id="fig:mapred.killed.tasks"}
![Status of killed tasks](images/5163os_04_14.png)

Hadoop JobTracker can automatically kill tasks in the following
situations:

- A task does not report progress after timeout
- Speculative execution can run one task on multiple nodes, if one of
  these task is succeeded, other attempts of the same task will be
  killed, because the attempt results for those attempts will be
  useless
- Job/Task schedulers such as fair scheduler and capacity scheduler
  need empty slots for other pools or queues

In many situations, we need a task to fail, which can be done with the
following command:

	$ hadoop job -fail-task <task-id> 

For example, to fail the task `task_201302281451_0013_m_000000`, we can
use the following command:

	$ hadoop job -fail-task task_201302281451_0013_m_000000

List task attempts with the following command:

	$ hadoop job -list-attempt-ids <job-id> <task-type> <task-state>

In this command, available task types are `map, reduce, setup`, and
`clean`; available task states are running and completed. For example,
to list all the completed map attempts for the job
`job_201302281451_0014`, the following command can be used:

    $ hadoop job -list-attempt-ids job_201302281451_0014 map completed
    attempt_201302281451_0014_m_000000_0
    attempt_201302281451_0014_m_000001_0
    attempt_201302281451_0014_m_000002_0
    attempt_201302281451_0014_m_000009_0
    attempt_201302281451_0014_m_000010_0
    ...

## Managing jobs through the web UI

We will show job management from the web UI.

Check the status of a job by opening the JobTracker URL:
<master:50030/jobtracker.jsp>.

We will get a web page similar to the following [figure](#fig:mapred.status).

{id="fig:mapred.status"}
![MapReduce Job and Scheduling information](images/5163os_04_11.png)

From this web page, we can get the cluster summary information, the
scheduling information, the running jobs list, the completed jobs list,
and the retired jobs list. By clicking on a specific job link, we can
check the details of a job, or we can open the URL,
<http://master:50030/jobdetails.jsp?jobid=job_201302281451_0004&refresh=30>.
By specifying the refresh parameter, we can tell the web page to refresh
every 30 seconds.

Kill a job by opening the URL,
<master:50030/jobdetails.jsp?jobid=job_201302281451_0007&action=kill>.

After a while, the killed job will be listed in the [Failed Jobs list](#fig:failed.job).

{id="fig:failed.job"}
![Information of a Failed job](images/5163os_04_12.png)

Change the job priority to be HIGH by opening the URL,
<master:50030/jobdetails.jsp?jobid=job_201302281451_0007&action=changeprio&prio=HIGH>.

### See also 

- The Validating Hadoop installation recipe of [Chapter 3](#chap:3),
  Configuring a Hadoop Cluster
- The Managing the HDFS cluster recipe
- The Managing MapReduce cluster recipe
- Refer to [MapReduce
  tutorial](http://hadoop.apache.org/docs/r1.1.2/mapred_tutorial.html).

## Checking job history from the web UI

Hadoop keeps track of all the submitted jobs in the logs directory. The
job history logs contain information for each job such as the total run
time and the run time of each task. In this section, we will show you
how to check the job history logs through a web UI.

### Getting ready 

We assume that our Hadoop cluster has been properly configured and all
daemons are running without any issues.

### How to do it... 

Perform the following steps to check job history logs from web UI:

Open the job history URL, <http://master:50030/jobhistoryhome.jsp> show in
the following [figure](#fig:history.jobs).

{id="fig:history.jobs"}
![History jobs](images/5163os_04_01.png)

On the web UI, we can filter jobs based on the username and job name in
the format **username:jobname** as shown in [figure](#fig:history.jobs).
username should be the username that runs a certain job, and job name
should contain keywords of Hadoop jobs.

From the web UI, we will be able to get a list of jobs in the Available
Jobs in History section. By clicking on the Job Id link of a job, show in
[figure](#fig:specific.job.history). 

{id="fig:specific.job.history"}
![History of a specific job](images/5163os_04_02.png)

This web page shows the details of the job, including task information
such as total, successful, failed, and killed tasks. The information
also includes the start time and end time of four phases of a Hadoop job
including setup, map, reduce, and cleanup phases.

The web page also contains information of counters of the job as shown
in the lower part of [figure](#fig:specific.job.history).

In addition to the summary of job information, the web UI provides
interface for us to analyze a job. By clicking on the link **Analyze This
Job**, we will get [figure](#fig:task.statistics).

{id="fig:task.statistics"}
![Statistics of tasks for a job](images/5163os_04_03.png)

The web page contains information of simple time analytics for each
task, for example the best performing tasks that take the shortest time,
the worse performing tasks, and the average time taken by all the tasks.

To further check the information of a task, we can click on the link for
the task, shown in [figure](#fig:task.info).

{id="fig:task.info"}
![Information of a task](images/5163os_04_04.png)

We can get the counters of a task by clicking on the Counters field of
the task as shown in the [figure](#fig:task.info), or we can get the same web
page by opening URL
<http://master:50030/taskstats.jsp?tipid=task_201302281211_0001_m_000000>.

In this URL, `task_201302281211_0001_m_000000` is the task ID we want to
get counters id, shown in [figure](#fig:task.counters).

{id="fig:task.counters"}
![Task Counters](images/5163os_04_05.png)

In addition to all these web services, the web UI provides a graphical
display of the progress of Hadoop jobs and each [phase](#fig:mapreduce.progress).

{id="fig:mapreduce.progress"}
![Progress of map and reduce tasks from the web UI](images/5163os_04_06.png)

This screenshot shows the progress of each map and reduce task. The
reduce task is composed of three phases, the shuffle phase, the sort
phase, and the reduce phase, with each phase composing of {$$}\frac{1}{3}{/$$}
of the total reduce task.

### How it works... 

The meaning of the job history URL, <master:50030/jobhistoryhome.jsp>.

- `master` Hostname of machine that runs the JobTracker daemon.
- `50030` The port number of the JobTracker embedded web server.
- `jobhistoryhome.jsp` The `.jsp` file name that provides the job
history service.

The web UI can be automatically updated every five seconds; this
interval can be modified by changing the
`mapreduce.client.completion.pollinterval` property in the
`$HADOOP_HOME/conf/mapred-site.xml` file similar to the following:

```xml
<property>
   <name>mapreduce.client.completion.pollinterval</name>
   <value>5000</value>
</property>
```

The following list shows the summary of URIs we can use to check the
status of jobs, tasks, and attempts.

- `master:50030/jobtracker.jsp`  JobTracker.
- `master:50030/jobhistoryhome.jsp`  Job history.
- [master:50030/jobtasks.jsp?jobid=\<jobID\>&type=map&pagenum=1](master:50030/jobtasks.jsp?jobid=<jobID>&type=map&pagenum=1)
 List of all map tasks.
- [master:50030/jobtasks.jsp?jobid=\<jobID\>&type=reduce&pagenum=1](master:50030/jobtasks.jsp?jobid=<jobID>&type=reduce&pagenum=1)
 List of all reduce tasks.
- [master:50030/taskdetails.jsp?tipid=\<taskID\>](master:50030/taskdetails.jsp?tipid=<taskID>)
 Task attempt details.
- [master:50030/taskstats.jsp?attemptid=\<attempID\>](master:50030/taskstats.jsp?attemptid=<attempID>)
 Attempt counters.

The following [table](#tbl:ids) lists the naming examples for jobID, taskID, and
attemptID.

{id="tbl:ids"}
|   **ID**   | **Example**                            |
|:-----------|:---------------------------------------|
|   jobID    | `job_201302281451_0001`                |
|   taskID   | `task_201302281451_0001_m_000000`      |
|  attemptID | `attempt_201302281451_0001_m_000000_0` |

### See also 

- The Validating Hadoop installation recipe of [Chapter 3](#chap:3),
  Configuring a Hadoop Cluster
- The Managing MapReduce jobs recipe

## Importing data to HDFS

If our Big Data is on the local filesystem, we need to move it to HDFS.
In this section, we will list steps to move data from the local
filesystem to the HDFS filesystem.

### Getting ready 

We assume that our Hadoop cluster has been properly configured and all
the Hadoop daemons are running without any issues. And we assume that
the data on the local system is in the directory `/data`.

### How to do it... 

Perform the following steps to import data to HDFS:

Use the following command to create a data directory on HDFS:

	$ hadoop fs -mkdir data

This command will create a directory
[/user/hduser/data](/user/hduser/data) in the HDFS filesystem.

Copy the data file from the local directory to HDFS using the following
command:

	$ hadoop fs -cp file:///data/datafile /user/hduser/data

Alternatively, we can use the command:

	hadoop fs -put /data/datafile /user/hduser/data

Verify the data file on HDFS with the following command:

	$ hadoop fs -ls /user/hduser/data

Move the data file from the local directory to HDFS with command:

	$ hadoop fs -mv file:///data/datafile /user/hduser/data

The local copy will be deleted if you use this command.

Use distributed copy to copy the large data file to HDFS:

	$ hadoop distcp file:///data/datafile /user/hduser/data

This command will initiate a MapReduce job with a number of mappers to
run the copy task in parallel.

### There's more... 

To copy multiple files from the local directory to HDFS, we can use the
following command:

	$ hadoop fs -copyFromLocal src1 src2 data

This command will copy two files src1 and src2 from the local directory
to the data directory on HDFS.

Similarly, we can move files from the local directory to HDFS. Its only
difference from the previous command is that the local files will be
deleted.

	$ hadoop fs -moveFromLocal src1 src2 data

This command will move two files, **src1** and **src2**, from the local
directory to HDFS.

Although distributed copy can be faster than the simple data importing
commands, it can incur large load to the node that the data resides on,
because of the possibly of high data transfer requests. distcp will be
more useful when copying data from one HDFS location to another. For
example:

	$ hadoop distcp hdfs:///user/hduser/file hdfs:///user/hduser/file-copy

### How it works... 

We can get the usage of the fs command with the following:

    $ hadoop fs
    Usage: java FsShell
               [-ls <path>]
               [-lsr <path>]
               [-du <path>]
               [-dus <path>]
               [-count[-q] <path>]
               [-mv <src> <dst>]
               [-cp <src> <dst>]
               [-rm [-skipTrash] <path>]
               [-rmr [-skipTrash] <path>]
               [-expunge]
               [-put <localsrc> ... <dst>]
               [-copyFromLocal <localsrc> ... <dst>]
               [-moveFromLocal <localsrc> ... <dst>]
               [-get [-ignoreCrc] [-crc] <src> <localdst>]
               [-getmerge <src> <localdst> [addnl]]
               [-cat <src>]
               [-text <src>]
               [-copyToLocal [-ignoreCrc] [-crc] <src> <localdst>]
               [-moveToLocal [-crc] <src> <localdst>]
               [-mkdir <path>]
               [-setrep [-R] [-w] <rep> <path/file>]
               [-touchz <path>]
               [-test -[ezd] <path>]
               [-stat [format] <path>]
               [-tail [-f] <file>]
               [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
               [-chown [-R] [OWNER][:[GROUP]] PATH...]
               [-chgrp [-R] GROUP PATH...]
               [-help [cmd]]

The `<src>` and `<dst>` parameters for these data import commands use
different default filesystem schema if no one has been explicitly
specified in the command.

The default `<src>` filesystem schema for `-cp` and `-mv` is `hdfs:///`,
which is configured with the *fs.default.name* property in the file
`$HADOOP_HOME/conf/core-site.xml`. While the default `<src>` filesystem
schema for `-put`, `-copyFromLocal`, and `-moveFromLocal` is `file:///`.

The default `<dst>` filesystem schema for all these commands is
`hdfs:///`.

### See also 

- The Managing the HDFS cluster recipe
- The Manipulating files on HDFS recipe

## Manipulating files on HDFS

Besides commands to copy files from the local directory, HDFS provides
commands to operate on files. In this section, we will show you how to
operate files, such as downloading files from HDFS, checking the content
of files, and removing files from HDFS.

### Getting ready 

We assume that our Hadoop cluster has been properly configured and all
the daemons are running without any issues.

### How to do it... 

Perform the following steps to check the status of files and directory
on HDFS:

List files of the user's home directory on HDFS using the following
command:

    $ hadoop fs -ls .
    Found 7 items
    drwx------ - hduser supergroup   0 2013-02-21 22:17 /user/hduser/.staging
    -rw-r--r-- 2 hduser supergroup 646 2013-02-21 22:28 /user/hduser/file1
    -rw-r--r-- 2 hduser supergroup 848 2013-02-21 22:28 /user/hduser/file2
    ...

To recursively list files in the home directory, we can use the command
`hadoop fs -lsr ...`

Check the space usage of files and folders in the home directory with
the following command:

    $ hadoop fs -du .
    Found 7 items
    648521        hdfs://master:54310/user/hduser/.staging
    646           hdfs://master:54310/user/hduser/file1
    3671517       hdfs://master:54310/user/hduser/file2
    ...

The first column shows the size of the file in bytes and the second
column shows the location of files on HDFS.

Sometimes, we can get a summarized usage of a directory with the command
`hadoop fs -dus ..` It will show us the total space usage of the directory
rather than the sizes of individual files and folders in the directory.
For example, we can get a one-line output similar to the following:

	$ hdfs://master:54310/user/hduser    109810605367

Check the content of a file with the following command:

	$ hadoop fs -cat file1

This command is handy to check the content of small files. But when the
file is large, it is not recommended. Instead, we can use the command
`hadoop fs -tail file1` to check the content of the last few lines.

Alternatively, we can use the command `hadoop fs -text file1` to show the
content of file1 in text format.

Use the following commands to test if file1 exists, is empty, or is a
directory:

    $ hadoop fs -test -e file1
    $ hadoop fs -test -z file1
    $ hadoop fs -test -d file1

Check the status of file1 using the following command:

	$ hadoop fs -stat file1

Perform the following steps to manipulate files and directories on HDFS:

Empty the trash using the following command:

	$ hadoop fs -expunge

Merge files in a directory dir and download it as one big file:

	$ hadoop fs -getmerge dir file1

This command is similar to the cat command in Linux. It is very useful
when we want to get the MapReduce output as one file rather than several
smaller partitioned files.

For example, the command can merge files `dir/part-00000`,
`dir/part-00001`, and so on to file1 to the local filesystem.

Delete file1 under the current directory using the following command:

	$ hadoop fs -rm file1

Note that this command will not delete a directory. To delete a
directory, we can use the command `hadoop fs -rmr` dir. It is very similar
to the Linux command `rm -r`, which will recursively delete everything in
the directory dir and the directory itself. So use it with caution.

Download file1 from HDFS using the following command:

	$ hadoop fs -get file1

The file1 file under the directory `/user/hduser` will be downloaded to
the current directory on the local filesystem.

Change the group membership of a regular file with the following
command:

	$ hadoop fs -chgrp hadoop file1

In this command, we are assuming group hadoop exists.

Also, we can use the command `hadoop fs -chgrp -R <hadoop-dir>` to
change the group membership of a directory dir recursively.

Change the ownership of a regular file with the following command:

	$ hadoop fs -chown hduser file1

Similarly, we can use the command `hadoop fs -chown hdadmin -R <dir>` to
change the ownership of a directory dir recursively.

Change the mode of a file with the following command:

	$ hadoop fs -chmod 600 file1

The mode of files and directories under HDFS follows a similar rule as
the mode under Linux.

Set the replication factor of file1 to be 3 using the following
command:

	$ hadoop fs -setrep -w 3 file1

Create an empty file using the following command:

	$ hadoop fs -touchz 0file

### How it works... 

We can get the usage of the fs command with the following command:

    $ hadoop fs
    Usage: java FsShell
               [-ls <path>]
               [-lsr <path>]
               [-du <path>]
               [-dus <path>]
               [-count[-q] <path>]
               [-mv <src> <dst>]
               [-cp <src> <dst>]
               [-rm [-skipTrash] <path>]
               [-rmr [-skipTrash] <path>]
               [-expunge]
               [-put <localsrc> ... <dst>]
               [-copyFromLocal <localsrc> ... <dst>]
               [-moveFromLocal <localsrc> ... <dst>]
               [-get [-ignoreCrc] [-crc] <src> <localdst>]
               [-getmerge <src> <localdst> [addnl]]
               [-cat <src>]
               [-text <src>]
               [-copyToLocal [-ignoreCrc] [-crc] <src> <localdst>]
               [-moveToLocal [-crc] <src> <localdst>]
               [-mkdir <path>]
               [-setrep [-R] [-w] <rep> <path/file>]
               [-touchz <path>]
               [-test -[ezd] <path>]
               [-stat [format] <path>]
               [-tail [-f] <file>]
               [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
               [-chown [-R] [OWNER][:[GROUP]] PATH...]
               [-chgrp [-R] GROUP PATH...]
               [-help [cmd]]

To get help for each individual command, we can use the -help option.
For example, we can get the help of the list command with the following:

    $ hadoop fs -help ls
    -ls <path>:     List the contents that match the specified file pattern. If
                    path is not specified, the contents of /user/<currentUser>
                    will be listed. Directory entries are of the form
                            dirName (full path) <dir>
                    and file entries are of the form
                            fileName(full path) <r n> size
                    where n is the number of replicas specified for the file
                    and size is the size of the file, in bytes.

## Configuring HDFS quota

In a multiuser environment, quota can enforce the fair share of
computing resources. HDFS supports quota for users and directories. In
this recipe, we will list steps to configure HDFS quota.

### Getting ready 

We assume that the Hadoop cluster has been configured properly and all
the daemons are running without any issues.

### How to do it... 

Perform the following steps to manage HDFS quota:

Set name quota on the home directory with the following command:

	$ hadoop dfsadmin -setQuota 20 /usr/hduser

This command will set name quota on the home directory to 20, which
means at most 20 files, including directories, can be created under the
home directory.

If we reach the quota, we will get an error message:

    put: org.apache.hadoop.hdfs.protocol.NSQuotaExceededException: The NameSpace quota (directories and files) of directory \verb|/user/hduser| is exceeded: quota=20 file count=141

Set space quota of the current user's home directory to be **100000000**
with the following command:

	$ hadoop dfsadmin -setSpaceQuota 100000000 /user/hduser

If the space usage under the directory `/user/hduser` exceeds the
specified quota, we will get an error message similar to the following:

    put: org.apache.hadoop.hdfs.protocol.DSQuotaExceededException: The DiskSpace quota of /user/hduser is exceeded: quota=100000000 diskspace consumed=204.5g

Check the quota status with the following command:

	$ hadoop fs -count -q /user/hduser

We will get output similar to the following before setting quota:

    none inf  none inf 13  127  109810605367 hdfs://master:54310/user/hduser

And we can get the following quota has been set:

    100  -40  100000000 -219525889438  13  127  1098106  05367 hdfs://master:54310/user/hduser

The meaning of output columns are

    DIR_COUNT FILE_COUNT
    CONTENT_SIZE FILE_NAME
    QUOTA
    REMAINING_QUATA SPACE_QUOTA
    REMAINING_SPACE_QUOTA
    DIR_COUNT
    FILE_COUNT
    CONTENT_SIZE
    FILE_NAME

Clear the name quota with the following command:

	$ hadoop dfsadmin -clrQuota /user/hduser

Clear the space quota with the following command:

	$ hadoop dfsadmin -clrSpaceQuota /user/hduser

### How it works... 

We can get the usage of the `hadoop fs` command with the following
command:

    $ hadoop dfsadmin
    Usage: java DFSAdmin
               [-report]
               [-safemode enter | leave | get | wait]
               [-saveNamespace]
               [-refreshNodes]
               [-finalizeUpgrade]
               [-upgradeProgress status | details | force]
               [-metasave filename]
               [-refreshServiceAcl]
               [-refreshUserToGroupsMappings]
               [-refreshSuperUserGroupsConfiguration]
               [-setQuota <quota> <dirname>...<dirname>]
               [-clrQuota <dirname>...<dirname>]
               [-setSpaceQuota <quota> <dirname>...<dirname>]
               [-clrSpaceQuota <dirname>...<dirname>]
               [-setBalancerBandwidth <bandwidth in bytes per second>]
               [-help [cmd]]

The generic usage of `-count` command is:

	$ hadoop fs -count -q <path>

In this command `-q` specifies the directory to query.

## Configuring CapacityScheduler

Hadoop CapacityScheduler is a pluggable MapReduce job scheduler. The
goal is to maximize the Hadoop cluster utilization by sharing the
cluster among multiple users. CapacityScheduler uses queues to guarantee
the minimum share of each user. It has features of being secure,
elastic, operable, and supporting job priority. In this recipe, we will
outline steps to configure CapacityScheduler for a Hadoop cluster.

### Getting ready 

We assume that our Hadoop cluster has been properly configured and all
the daemons are running without any issues. Log in to the master node
from the cluster administrator machine using the following command:

	$ ssh hduser@master

### How to do it... 

Configure CapacityScheduler with the following steps:

Configure Hadoop to use CapacityScheduler by adding the following lines
into the file `$HADOOP_HOME/conf/mapred-site.xml`:

```xml
<property>
    <name>mapred.jobtracker.taskScheduler</name>
    <value>org.apache.hadoop.mapred.CapacityTaskScheduler </value>
</property>
```

Define a new queue, hdqueue, by adding the following lines into the file
`$HADOOP_HOME/conf/mapred-site.xml`:

```xml
<property>
    <name>mapred.queue.names</name>
    <value>default,hdqueue</value>
  </property>
```

By default, a Hadoop cluster has only one default queue.

Configure `CapacityScheduler` queues by adding the following lines into
the file `$HADOOP_HOME/conf/capacity-scheduler.xml`:

```xml
<property>
  <name>mapred.capacity-scheduler.queue.hdqueue.capacity</name>
  <value>20</value>
</property>

<property>
  <name>mapred.capacity-scheduler.queue.default.capacity</name>
  <value>80</value>
</property>

<property>
  <name>mapred.capacity-scheduler.queue.hdqueue.minimum-user-limit-percent</name>
  <value>20</value>
</property>

<property>
  <name>mapred.capacity-scheduler.maximum-system-jobs</name>
  <value>10</value>
</property>

<property>
  <name>mapred.capacity-scheduler.queue.hdqueue.maximum-initialized-active-tasks</name>
  <value>500</value>
</property>

<property>
  <name>mapred.capacity-scheduler.queue.hdqueue.maximum-initialized-active-tasks-per-user</name>
  <value>100</value>
</property>

<property>
  <name>mapred.capacity-scheduler.queue.hdqueue.supports-priority</name>
  <value>true</value>
</property>
```

Restart the MapReduce cluster with the following commands:

    $ stop-mapred.sh
    $ start-mapred.sh

From the JobTracker web UI, we can get a queue
[scheduling information](#fig:mapred.scheduling). 

{id="fig:mapred.scheduling"}
![Hadoop MapReduce job Scheduling information](images/5163os_04_17.png)

Alternatively, we can use the command `hadoop queue -list` to get the same
information.

Get the schedule details of each queue by opening the URL,
`master:50030/scheduler`.

{id="fig:job.queues"}
![Job Scheduler Queues](images/5163os_04_18.png)

[The figure](#fig:job.queues) shows the status of each queue in the cluster
including the numbers of running jobs, pending jobs, and so on.

Test the queue configuration by submitting an example wordcount job to
the queue hdqueue using the following command:

	$ hadoop jar $HADOOP_HOME/hadoop-examples-1.1.2.jar wordcount -Dmapred.job.queue.name=hdqueue randtext wordcount.out

From the job information web UI, we can get the job scheduling
information similar to the following:

    Job Scheduling information: 8 running map tasks using 8 map slots. 0 additional slots reserved. 1 running reduce tasks using 1 reduce slots. 0 additional slots reserved.

### How it works... 

CapacityScheduler is available as a JAR file under the
`$HADOOP_HOME/lib` directory. For example, in our Hadoop distribution,
the JAR file is `$HADOOP_HOME/lib/hadoop-capacity-scheduler-1.1.2.jar`.

A few important queue configuration parameters are described below.

- `mapred.capacity-scheduler.queue.hdqueue.capacity` The percentage share
  of total number of slots for the hdqueue queue.
- `mapred.capacity-scheduler.queue.default.capacity` The percentage share
  of total number of slots for default queue.
- `mapred.capacity-scheduler.queue.hdqueue.minimum-user-limit-percent` The
  percentage of minimum resources allocated for each user in the queue
  hdqueue.
- `mapred.capacity-scheduler.maximum-system-jobs` The maximum number of
  jobs that can be initialized concurrently by CapacityScheduler.
- `mapred.capacity-scheduler.queue.hdqueue.maximum-initialized-active-tasks` 
  The maximum number of concurrently initialized tasks across all jobs in the queue hdqueue.
- `mapred.capacity-scheduler.queue.hdqueue.maximum-initialized-active-tasks-per-user`
  The maximum number of concurrently initialized tasks across all jobs in
  the queue hdqueue for each user. 
- `mapred.capacity-scheduler.queue.hdqueue.supports-priority`Whether to
  support job priority for job scheduling or not.

### There's more... 

Hadoop supports access control on the queue using queue ACLs. Queue ACLs
control the authorization of MapReduce job submission to a queue. More
information about queue ACLs can be found at
<http://hadoop.apache.org/docs/r1.1.2/cluster_setup.html#Configuring+the+Hadoop+Daemons>.

### See also 

- The Managing MapReduce jobs recipe
- The Checking job history from the web UI recipe
- The Configuring Fair Scheduler recipe
- The Configuring job authentication with ACL recipe of
  [Chapter 5](#chap:5), Hardening a Hadoop Cluster 

- Refer to
    <http://hadoop.apache.org/docs/r1.1.2/capacity_scheduler.html>

## Configuring Fair Scheduler

Similar to CapacityScheduler, Fair Scheduler was designed to enforce
fair shares of cluster resources in a multiuser environment. In this
recipe, we will outline steps to configure Fair Scheduler for a Hadoop
cluster.

### Getting ready 

We assume that our Hadoop cluster has been configured properly and all
the daemons are running without any problems. Log in to the master node
from the Hadoop administrator machine using the following command:

	$ ssh hduser@master

### How to do it... 

Perform the following steps to configure Hadoop Fair Scheduler:

Enable fair scheduling by changing the following property in the file
`$HADOOP_HOME/conf/mapred-site.xml`:

```xml
  <property>
    <name>mapred.jobtracker.taskScheduler</name>
    <value>org.apache.hadoop.mapred.FairScheduler</value>
  </property>
```

Create the Fair Scheduler configuration file,
`$HADOOP_HOME/conf/fair-scheduler.xml`, with content similar to the
following:

```xml
<?xml version="1.0"?>
<allocations>
 <pool name="hduser">
    <minMaps>5</minMaps>
    <minReduces>5</minReduces>
    <maxMaps>90</maxMaps>
    <maxReduces>20</maxReduces>
    <weight>2.0</weight>
  </pool>
  <user name="hduser">
    <maxRunningJobs>1</maxRunningJobs>
  </user>
  <userMaxJobsDefault>3</userMaxJobsDefault>
</allocations>
```

Restart the MapReduce cluster with the following commands:

	$ stop-mapred.sh
	$ start-mapred.sh

Verify the setting of Fair Scheduler by opening the URL
<http://master:50030/scheduler>, as shown in the
[following figure](#fig:fairscheduler). 

{id="fig:fairscheduler"}
![Status of fair scheduler](images/5163os_04_16.png)

### How it works... 

The Hadoop Fair Scheduler schedules jobs in such a way that all jobs can
get an equal share of computing resources. Jobs are organized with
scheduling pools. A pool can be configured for each Hadoop user. If the
pool for a user is not configured, the default pool will be used. A pool
specifies the amount of resources a user can share on the cluster, for
example the number of map slots, reduce slots, the total number of
running jobs, and so on.

`minMaps` and `minReduces` are used to ensure the minimum share of
computing slots on the cluster for a pool. The minimum share guarantee
can be useful when the required number of computing slots is larger than
the number of configured slots. In case if the minimum share of a pool
is not met, JobTracker will kill tasks on other pools and assign the
slots to the starving pool. In such cases, the JobTracker will restart
the killed tasks on other nodes and thus, the job will take longer time
to finish.

Besides computing slots, the Fair Scheduler can limit the number of
concurrently running jobs and tasks on a pool. So, if a user submits
more jobs than the configured limit, some jobs have to in-queue until
other jobs finish. In such a case, higher priority jobs will be
scheduled by the Fair Scheduler to run earlier than lower priority jobs.
If all jobs in the waiting queue have the same priority, the Fair
Scheduler can be configured to schedule these jobs with either Fair
Scheduler or FIFO Scheduler.

Following [table](#tbl:fairscheduler) shows the properties supported by fair
scheduler:

{id="tbl:fairscheduler"}
| **Property**                     | **Value** | **Description**																					  |
|:---------------------------------|:----------|:-----------------------------------------------------------------------------------------------------|	   	
| minMaps                          | Integer   | Minimum map slots for a pool.																		  |
| maxMaps                          | Integer   | Maximum map slots for a pool.																		  |
| minReduces                       | Integer   | Minimum reduce slots for a pool.																	  |
| minReduces                       | Integer   | Maximum reduce slots for a pool.																	  |
| schedulingMode                   | Fair/FIFO | Pool internal scheduling mode, fair or fifo.														  |
| maxRunningJobs                   | Integer   | Maximum number of concurrently running jobs for a pool. Default value is unlimited.				  |
| weight                           | Float     | Value to control non proportional share of cluster resource. The default value is 1.0.				  |
| minSharePreemptionTimeout        | Integer   | Seconds to wait before killing other pool's tasks if a pool's share is under minimum share.		  |
| maxRunningJobs                   | Integer   | Maximum number of concurrent running jobs for a user. Default is unlimited.						  |
| poolMaxJobsDefault               | Integer   | Default maximum number of concurrently running jobs for a pool.									  |
| userMaxJobsDefault               | Integer   | Default maximum number of concurrently running jobs for a user.									  |
| defaultMinSharePreemptionTimeout | Integer   | Default seconds to wait before killing other pool's tasks when a pool's share is under minimum share.|
| fairSharePreemptionTimeout       | Integer   | Pre-emption time when a job's resource is below half of the fair share.							  |
| defaultPoolSchedulingMode        | Fair/FIFO | Default in-pool scheduling mode.																	  |
																																					   
### See also 																																		   
																																					   
- The Configuring CapacityScheduler recipe																											   
- Refer to [the FairScheduler																														   
    documentation](http://hadoop.apache.org/docs/r1.1.2/fair_scheduler.html).

## Configuring Hadoop daemon logging

System logging plays an important role in dealing with performance and
security problems. In addition, the logging information can be used
analytically to tune the performance of a Hadoop cluster. In this
recipe, we will show you how to configure Hadoop logging.

### Getting ready 

We assume that our Hadoop cluster has been properly configured.

### How to do it... 

Perform the following steps to configure Hadoop logging:

Log in to the master node with the following command from the Hadoop
administrator machine:

	$ ssh hduser@master

Check the current logging level of JobTracker with the following
command:

    hadoop daemonlog -getlevel master:50030 org.apache.hadoop.mapred.JobTracker
    Connecting to http://master:50030/logLevel?log=org.apache.hadoop.mapred.JobTracker
    Submitted Log Name: org.apache.hadoop.mapred.JobTracker
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Effective level: INFO

Tell Hadoop to only log error events for JobTracker using the following
command:

    $ hadoop daemonlog -setlevel master:50030 org.apache.hadoop.mapred.JobTracker ERROR
    Connecting to http://master:50030/logLevel?log=org.apache.hadoop.mapred.JobTracker&level=ERROR
    Submitted Log Name: org.apache.hadoop.mapred.JobTracker
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Submitted Level: ERROR
    Setting Level to ERROR ...
    Effective level: ERROR

Now, the logging status of the JobTracker daemon will be similar to the
following:

    Connecting to http://master:50030/logLevel?log=org.apache.hadoop.mapred.JobTracker
    Submitted Log Name: org.apache.hadoop.mapred.JobTracker
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Effective level: ERROR

Get the log levels for TaskTracker, NameNode, and DataNode with the
following commands:

    hadoop daemonlog -getlevel master:50030 org.apache.hadoop.mapred.TaskTracker hadoop daemonlog -getlevel master:50070 org.apache.hadoop.dfs.NameNode
    hadoop daemonlog -getlevel master:50070 org.apache.hadoop.dfs.DataNode
    Connecting to http://master:50030/logLevel?log=org.apache.hadoop.mapred.TaskTracker
    Submitted Log Name: org.apache.hadoop.mapred.TaskTracker
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Effective level: WARN

    Connecting to http://master:50070/logLevel?log=org.apache.hadoop.dfs.NameNode
    Submitted Log Name: org.apache.hadoop.dfs.NameNode
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Effective level: INFO

    Connecting to http://master:50070/logLevel?log=org.apache.hadoop.dfs.DataNode
    Submitted Log Name: org.apache.hadoop.dfs.DataNode
    Log Class: org.apache.commons.logging.impl.Log4JLogger
    Effective level: INFO

### How it works... 

By default, Hadoop sends log messages to Log4j, which is configured in
the file `$HADOOP_HOME/conf/log4j.properties`. This file defines both
what to log and where to log. For applications, the default root logger
is INFO,console , which logs all messages at level INFO and above the
console's stderr. Log files are named
`$HADOOP_LOG_DIR/hadoop-$HADOOP_IDENT_STRING-<hostname>.log`.

Hadoop supports a number of log levels for different purposes. The log
level should be tuned based on the purpose of logging. For example, if
we are debugging a daemon, we can set its logging level to be DEBUG
rather than something else. Using a verbose log level can give us more
information, while on the other hand will incur overhead to the cluster.

The following [table](#tbl:log4j) shows all the logging levels provided by Log4j:

{id="tbl:log4j"}
| **Log level**    | **Description** 															|
|:-----------------|:---------------------------------------------------------------------------|
| `ALL`            | The lowest logging level, all loggings will be turned on.					|
| `DEBUG`          | Logging events useful for debugging applications.							|
| `ERROR`          | Logging error events, but application can continue to run.					|
| `FATAL`          | Logging very severe error events that will abort applications.				|
| `INFO`           | Logging informational messages that indicate the progress of applications. |
| `OFF`            | Logging will be turned off.												|
| `TRACE`          | Logging more finger-grained events for application debugging.				|
| `TRACE_INT`      | Logging in TRACE level on integer values.									|
| `WARN`           | Logging potentially harmful events.										|
																								 
We can get the usage of daemonlog with the following command:									 
																								 
    $ hadoop daemonlog																			 
    USAGES:																						 
    java org.apache.hadoop.log.LogLevel -getlevel <host:port> <name>							 
    java org.apache.hadoop.log.LogLevel -setlevel <host:port> <name> <level>					 
																								 
### There's more... 																			 
																								 
Other than configuring Hadoop logging on the fly from command line, we							 
can configure it using configuration files. The most important file that						 
we need to configure is `$HADOOP/conf/hadoop-env.sh`.											 
																								 
Sometimes, audit logging is desirable for corporate auditing purposes.							 
Hadoop provides audit logging through Log4j using the INFO logging								 
level. We will show you how to configure Hadoop audit logging in the							 
next recipe.																					 

The cluster needs to be restarted for the configuration to take effect.

#### Configuring Hadoop logging with hadoop-env.sh 

Open the file `$HADOOP_HOME/conf/hadoop-env.sh` with a text editor and
change the following line:

	# export HADOOP_LOG_DIR=${HADOOP_HOME}/logs

We change the preceding command to the following:

	export HADOOP_LOG_DIR=${HADOOP_HOME}/logs

Configure the logging directory to `/var/log/hadoop` by changing the
following line:

	export HADOOP_LOG_DIR=/var/log/hadoop

Additionally, following [table](#tbl:loggingenv) shows other environment variables
we can configure for Hadoop logging:

{id="tbl:loggingenv"}
|  **Variable name**       | **Description**																		 |
|:-------------------------|:----------------------------------------------------------------------------------------|
| `HADOOP_LOG_DIR`         | Directory for log files.																 |
| `HADOOP_PID_DIR`         | Directory to store the PID for the servers.											 |
| `HADOOP_ROOT_LOGGER`     | Logging configuration for hadoop.root.logger. Default value: "INFO,console".			 |
| `HADOOP_SECURITY_LOGGER` | Logging configuration for `hadoop.security.logger`. Default value: "INFO,NullAppender". |
| `HDFS_AUDIT_LOGGER`      | Logging configuration for hdfs.audit.logger. Default value: "INFO,NullAppender".		 |
																													  
## Configuring Hadoop security logging																				  
																													  
Security logging can help Hadoop cluster administrators to identify													  
security problems. It is enabled by default.																		  

The security logging configuration is located in the file
`$HADOOP_HOME/conf/log4j.properties`. By default the security logging
information is appended to the same file as NameNode logging. We can
check the security logs with the following command:

    grep security $HADOOP_HOME/logs/hadoop-hduser-namenode-master.log
    2013-02-28 13:36:01,008 ERROR org.apache.hadoop.security.UserGroupInformation: PriviledgedActionException as:hduser cause:org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create file/user/hduser/test. Name node is in safe mode.

The error message tells that the NameNode is in safe mode, so the file
`/user/hduser/test` cannot be created. Similar information can give us a
very useful hint to figure out operation errors.

#### Hadoop logging file naming conventions 

Hadoop logs files are kept under the directory `$HADOOP_HOME/logs`.

The folder contains one .log file and one .out file for each Hadoop
daemon, for example, NameNode, SecondaryNameNode, and JobTracker on the
master node and TaskTracker and DataNode on a slave node. The .out file
is used when a daemon is being started. Its content will be emptied
after the daemon has started successfully. The .log files contain all
the log messages for a daemon, including startup logging messages.

On the master node, the logs directory contains a history folder that
contains logs of the MapReduce job history. Similarly, on a slave node,
the logs directory contains a userlogs directory, which maintains the
history information of the tasks that ran on the node.

In Hadoop, the names of logging files are using the following format:

	$ hadoop-<username>-<daemonname>-<hostname>.log

### See also 

- The Configuring Hadoop audit logging recipe
- Refer to <http://wiki.apache.org/hadoop/HowToConfigure>

## Configuring Hadoop audit logging

Audit logging might be required for data processing systems such as
Hadoop. In Hadoop, audit logging has been implemented using the Log4j
Java logging library at the INFO logging level. By default, Hadoop audit
logging is disabled. This recipe will guide you through the steps to
configure Hadoop audit logging.

### Getting ready 

We assume that our Hadoop cluster has been configured properly. Log in
to the master node from the administrator machine using the following
command:

	$ ssh hduser@master

### How to do it... 

Perform the following steps to configure Hadoop audit logging:

Enable audit logging by changing the following line in the
`$HADOOP_HOME/conf/log4j.properties` file from:

    log4j.logger.org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit=WARN

to the following:

    log4j.logger.org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit=INFO

Try making a directory on HDFS with the following command:

	$ hadoop fs -mkdir audittest

Check the audit log messages in the NameNode log file with the following
command:

    $ grep org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit $HADOOP_HOME/logs/hadoop-hduser-namenode-master.log
    2013-02-28 13:38:04,235 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit: ugi=hduser    ip=/10.0.0.1  cmd=mkdirs   src=/user/hduser/audittest  dst=null        perm=hduser:supergroup:rwxr-xr-x

The Hadoop NameNode is responsible for managing audit logging messages,
which are forwarded to the NameNode logging facility. So what we have
seen so far is that the audit logging message has been mixed with the
normal logging message.

We can separate the audit logging messages from the NameNode logging
messages by configuring the file `$HADOOP_HOME/conf/log4j.properties`
with the following content:

    # Log at INFO level, SYSLOG appenders
    log4j.logger.org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit=INFO

    # Disable forwarding the audit logging message to the NameNode logger.
    log4j.additivity.org.apache.hadoop.hdfs.server.namenode.FSNamesystem.audit=false

    ################################
    # Configure logging appender
    ################################
    #
    # Daily Rolling File Appender (DRFA)
    log4j.appender.DRFAAUDIT=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.DRFAAUDIT.File=$HADOOP_HOME/logs/audit.log
    log4j.appender.DRFAAUDIT.DatePattern=.yyyy-MM-dd
    log4j.appender.DRFAAUDIT.layout=org.apache.log4j.PatternLayout
    log4j.appender.DRFAAUDIT.layout.ConversionPattern=%d{ISO8601} %p %c: %m%n

### How it works... 

Hadoop logs auditing messages of operations, such as creating, changing,
or deleting files into a configured log file. By default, audit logging
is set to `WARN`, which disables audit logging. To enable it, the
logging level need be changed to `INFO`.

When a Hadoop cluster has many jobs to run, the log file can become
large very quickly. Log file rotation is a function that periodically
rotates a log file to a different name, for example, by appending the
date to the filename, so that the original log file name can be used as
an empty file.

### See also 

- The Configuring Hadoop daemon logging recipe

## Upgrading Hadoop

A Hadoop cluster needs to be upgraded when new versions with bug fixes
or new features are released. In this recipe, we will outline steps to
upgrade a Hadoop cluster to a newer version.

### Getting ready 

Download the desired Hadoop release from an Apache mirror site:
<http://www.apache.org/dyn/closer.cgi/hadoop/common/>. In this book, we
assume to upgrade Hadoop from version `1.1.2` to version `1.2.0`, which
is still in the beta state when writing this book.

We assume that there are no running or pending MapReduce jobs in the
cluster.

In the processing of upgrading a Hadoop cluster, we want to minimize the
damage to the data stored on HDFS, and this procedure is the cause of
most of the upgrade problems. The data damages can be caused by either
human operation or software and hardware failures. So, a backup of the
data might be necessary. But the sheer size of the data on HDFS can be a
headache for most of the upgrade experience.

A more practical way is to only back up the HDFS filesystem metadata on
the master node, while leaving the data blocks intact. If some data
blocks are lost after upgrade, Hadoop can automatically recover it from
other backup replications.

Log in to the master node from the administrator machine with the
following command:

	$ ssh hduser@master

### How to do it... 

Perform the following steps to upgrade a Hadoop cluster:

Stop the cluster with the following command:

	$ stop-all.sh

Back up block locations of the data on HDFS with the fsck command:

	$ hadoop fsck / -files -blocks -locations > dfs.block.locations.fsck.backup

The resulting file, `dfs.block.locations.fsck.backup`, will contain the
locations of each data block on the HDFS filesystem.

Save the list of all files on the HDFS filesystem with the following
command:

	$ hadoop dfs -lsr / > dfs.namespace.lsr.backup

Save the description of each DataNode in the HDFS cluster with the
following command:

	$ hadoop dfsadmin -report > dfs.datanodes.report.backup

Copy the checkpoint files to a backup directory with the following
commands:

	$ sudo cp dfs.name.dir/edits /backup
	$ sudo cp dfs.name.dir/image/fsimage /backup

Verify that no DataNode daemon is running with the following command:

	for node in `cat $HADOOP_HOME/conf/slaves`
		do
		echo 'Checking node ' $node
		ssh $node -C "jps"
	done

If any DataNode process is still running, kill the process with the
following command:

	$ ssh $node -C "jps | grep 'DataNode' | cut -d'\t' -f 1 | xargs kill -9 "

A still running DataNode can fail an update if it is not killed because
the old version DataNode might register with the newer version NameNode,
causing compatibility problems.

Decompress the Hadoop archive file with the following commands:

	$ sudo mv hadoop-1.2.0.tar.gz /usr/local/
	$ sudo tar xvf hadoop-1.2.0.tar.gz

Copy the configuration files from the old configuration directory to the
new one using the following command:

	$ sudo cp $HADOOP_HOME/conf/* /usr/local/hadoop-1.2.0/conf/*
	
You can make changes to the configuration files if necessary.

Update the Hadoop symbolic link to the Hadoop version with the following
command:

	$ sudo rm -rf /usr/local/hadoop
	$ sudo ln -s /usr/local/hadoop-1.2.0 /usr/local/hadoop

Upgrade in the slave nodes with the following commands:

	for host in `cat $HADOOP_HOME/conf/slaves`
		do
		echo 'Configuring hadoop on slave node ' $host
		sudo scp -r /usr/local/hadoop-1.2.0 hduser@$host:/usr/local/
		echo 'Making symbolic link for Hadoop home directory on host ' $host
		sudo ssh hduser@$host -C "ln -s /usr/local/hadoop-1.2.0 /usr/local/hadoop"
	done

Upgrade the NameNode with the following command:

	$ hadoop namenode -upgrade

This command will convert the checkpoint to the new version format. We
need to wait to let it finish.

Start the HDFS cluster using the following commands:

	$ start-dfs.sh

Get the list of all files on HDFS and compare its difference with the
backed up one using the following commands:

	$ hadoop dfs -lsr / > dfs.namespace.lsr.new
	$ diff dfs.namespace.lsr.new dfs.namespace.lsr.backup

The two files should have the same content if there is no error in the
upgrade.

Get a new report of each DataNode in the cluster and compare the file
with the backed up one using the following command:

	$ hadoop dfsadmin -report > dfs.datanodes.report.new
	$ diff dfs.datanodes.report.1.log dfs.datanodes.report.backup

The two files should have the same content if there is no error.

Get the locations of all data blocks and compare the output with the
previous backup using the following commands:

	$ hadoop fsck / -files -blocks -locations > dfs.block.locations.fsck.new
	$ diff dfs.locations.fsck.backup dfs.locations.fsck.new

The result of this command should tell us that the data block locations
should be the same.

Start the MapReduce cluster using the following command:

	$ start-mapred.sh

Now, we can check the status of the cluster either by running a sample
MapReduce job such as teragen and terasort, or by using the web user
interface.

### How it works... 

We can use the following command to get the usage of HDFS upgrade
commands:

    $ hadoop dfsadmin

    Usage: java DFSAdmin
               [-report]
               [-safemode enter | leave | get | wait]
               [-saveNamespace]
               [-refreshNodes]
               [-finalizeUpgrade]
               [-upgradeProgress status | details | force]
               [-metasave filename]
               [-refreshServiceAcl]
               [-refreshUserToGroupsMappings]
               [-refreshSuperUserGroupsConfiguration]
               [-setQuota <quota> <dirname>...<dirname>]
               [-clrQuota <dirname>...<dirname>]
               [-setSpaceQuota <quota> <dirname>...<dirname>]
               [-clrSpaceQuota <dirname>...<dirname>]
               [-setBalancerBandwidth <bandwidth in bytes per second>]
               [-help [cmd]]

Following [table](#tbl:dfsadmin) shows the meanings of the command options:

{id="tbl:dfsadmin"}																															
| **Option**       | **Description**																										|
|:-----------------|:-----------------------------------------------------------------------------------------------------------------------|  
| -report          | Reports filesystem information and statistics.																			|
| -saveNamespace   | Save a snapshot of the filesystem metadata into configured directories.												|
| -finalizeUpgrade | Finalize the upgrade of HDFS; this command will cause the DataNode to delete the previous version working directories. |
| -metasave        | Saves the metadata of the HDFS cluster.																				|
																																			 
### See also 																																 
																																			 
- The Configuring Hadoop in pseudo-distributed mode recipe of
  [Chapter 3](#chap:3), Configuring a Hadoop Cluster 
- The Configuring Hadoop in fully-distributed mode recipe of
  [Chapter 3](#chap:3), Configuring a Hadoop Cluster 
- The Validating Hadoop installation recipe of [Chapter 3](#chap:3), Configuring
  a Hadoop Cluster 

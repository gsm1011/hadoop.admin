# Configuring a Hadoop Cluster <a name="#chap:3"></a>

In this chapter, we will cover:

- Choosing a Hadoop version
- Configuring Hadoop in pseudo-distributed mode
- Configuring Hadoop in fully-distributed mode
- Validating Hadoop installation
- Installing ZooKeeper
- Installing HBase
- Installing Hive
- Installing Pig
- Installing Mahout

## Choosing a Hadoop version

As an open source project, Hadoop has been under active development over
the past a few years. New versions are being released regularly. These
new releases either fix bugs contributed by the community, leading to a
more stable Hadoop software stack or add new features for the purpose of
more full-fledged, enterprise class distribution.

In this section, we are going to review the release history of Hadoop,
pointing out features of these releases. More importantly, we will give
tips on choosing a proper Hadoop distribution.

### Getting ready <a name="#getting-ready-8"></a>

In general, the release version number of a Hadoop distribution consists
of three parts: the version number, the major revision number and the
minor revision number. A Hadoop release version can be described in
Figure [fig:hadoop.release].

![Structure of a Hadoop release version<span
data-label="fig:hadoop.release"></span>](figs/5163os_03_01.png)

Sometimes, the revision number can have a fourth part, for example
`0.20.203.0`, but this is relatively rare.

### How to do it... <a name="#how-to-do-it...-1"></a>

Table [tbl:compare.mrv1.mrv2] shows features of major Hadoop releases:

<span>l|l|l|l|l</span> **Feature or Version** & **2.x.y** & **1.1.x** &
**0.23.x** & **0.20.x**\
Stable & & Yes & & Yes\
MRv1 & & Yes & & Yes\
MRv2 & Yes & & Yes &\
Kerberos Security & Yes & Yes & Yes &\
HDFS federation & Yes & & Yes &\
NameNode HA & Yes & & Yes &\
HDFS append & Yes & Yes & Yes &\
HDFS symbolic Links & Yes & Yes & Yes &\

The table tells us that Hadoop is evolving rapidly, with new features
such as security, HDFS federation and NameNode HA being added over time.
Another lesson we can learn from the table is that the most recent
stable release version 1.1.x does not contain all the features. And
although release version 2.0.x is the most feature rich Hadoop release,
it is still in alpha state requiring further improvements.

So, which version should I choose for my deployment? Generally, we need
to consider the following two properties: stability and features. For a
production deployment, we definitely want to deploy a stable release and
we also want to use the release that contains all the required features.
Clearly, our current optimal and only choice is version 1.1.x, or
specifically version 1.1.2 as of this book writing.

### See also <a name="#see-also-5"></a>

- More information about Hadoop releases can be found at [Hadoop @
    Apache](http://hadoop.apache.org/releases.html)

## Configuring Hadoop in pseudo-distributed mode

Pseudo-distributed mode refers to a Hadoop cluster configuration that
contains only one node. This mode can be helpful for debugging and
validation purposes. In this recipe, we will outline steps to configure
Hadoop in pseudo-distributed mode.

### Getting ready <a name="#getting-ready-9"></a>

Before configuring Hadoop in pseudo-distributed mode, we assume that we
have a machine, for example the master node of the Hadoop cluster, with
Linux installed. And all the necessary tools have been installed and
properly configured.

The most important dependent software is Java, which is the programming
language and library that Hadoop is based on. To check that Java has
been properly installed, we can use the following command:

	$ java -version
    java version "1.7.0_13"
    Java(TM) SE Runtime Environment (build 1.7.0_13-b20)
    Java HotSpot(TM) 64-Bit Server VM (build 23.7-b01, mixed mode)
    If you have installed OpenJDK other than the Oracle's official Java, the output will be similar to the following:
    Java version "1.7.0_09-icedtea"
    OpenJDK Runtime Environment (fedora-2.3.4.fc17-x86_64)
    OpenJDK 64-Bit Server VM (build 23.2-b09, mixed mode)

If you have installed OpenJDK, please refer to recipe *Installing Java
and other tools* of Chapter 2, Preparing for Hadoop installation.

Download the desired Hadoop distribution. In this book, we assume to use
Hadoop release 1.1.2.

To download a Hadoop release from one [mirror
site](http://www.apache.org/dyn/closer.cgi/hadoop/common/), choose the
proper mirror site (or use the suggested link on top of the mirror).
Start download by clicking the proper Hadoop release. We suggest
downloading a gzip archived file with file name ending with tar.gz.

Alternatively, we can download a Hadoop release with the following
command under Linux:

	$ wget http://mirror.quintex.com/apache/hadoop/common/hadoop-1.1.2/hadoop-1.1.2.tar.gz -P ~

Last, we assume that ssh password-less login has been properly
configured.

### How to do it... <a name="#how-to-do-it...-2"></a>

Use the following recipe to configure Hadoop in pseudo-distributed mode:

Copy the Hadoop archive to `/usr/local` directory:

	sudo cp hadoop-1.1.2.tar.gz /usr/local

Decompress the Hadoop package archive:

	cd /usr/local
	sudo tar xvf hadoop-1.1.2.tar.gz

The uncompressed archive file will contain the following files and
folders:

    CHANGES.txt  c++                     hadoop-examples-1.1.2.jar     lib
    LICENSE.txt  conf                    hadoop-minicluster-1.1.2.jar  libexec
    NOTICE.txt   contrib                 hadoop-test-1.1.2.jar         sbin
    README.txt   hadoop-ant-1.1.2.jar    hadoop-tools-1.1.2.jar        share
    bin          hadoop-client-1.1.2.jar ivy                           src
    build.xml    hadoop-core-1.1.2.jar   ivy.xml                       webapps

The folder contains several jar files and folders such as bin, sbin and
conf. Jar files hadoop-core-1.1.2.jar and hadoop-tools-1.1.2.jar contain
the core classes of Hadoop. File hadoop-examples-1.1.2.jar and
hadoop-test-1.1.2.jar contains sample MapReduce jobs.

Folder conf contains cluster configuration files, folder bin contains
commands and scripts to start and stop a cluster and folder sbin
contains scripts to do specific tasks.

Make a soft link for Hadoop root directory.

	$ sudo ln -s hadoop-1.1.2 hadoop

Use your favorite text editor to open file  /.bashrc and add the
following contents:

    export JAVA_HOME=/usr/java/latest
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$JAVA_HOME/bin:HADOOP_HOME/bin

We are assuming Oracle Java has been installed under directory
/usr/java/latest.\
Source file  /.bashrc with command:

	$ . ~/.bashrc

Use your favorite text editor to open file
`$HADOOP_HOME/conf/hadoop-env.sh` and change the `JAVA_HOME` environment
variable to:

	export JAVA_HOME=/usr/Java/latest

Use your favorite text editor to open file
`$HADOOP_HOME/conf/core-site.xml` and add the following content:

```xml
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:54310</value>
  </property>

  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:54311</value>
  </property>

  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop/tmp/</value>
  </property>
</configuration>
```

Use your favorite text editor to open file
`$HADOOP_HOME/conf/hdfs-site.xml` and add the following content to the
file:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>

  <property>
    <name>dfs.data.dir</name>
    <value>/hadoop/data/</value>
  </property>
</configuration>
```

Use your favorite text editor to open file
`$HADOOP_HOME/conf/mapred-site.xml` and add the following content:

```xml
<configuration>
  <property>
    <name>mapred.system.dir</name>
    <value>/hadoop/mapred</value>
  </property>
</configuration>
```

Ask localhost to run SecondaryNameNode daemon with command:

	$ sudo echo 'localhost' > $HADOOP_HOME/conf/masters

Configure localhost as the single slave node with command:

	$ sudo echo "localhost" > $HADOOP_HOME/conf/slaves

Use the following steps to start and stop a Hadoop cluster:

Format the HDFS filesystem from NameNode with the following command:

	$ hadoop namenode -format

We will get output similar to the following:

    13/02/14 01:43:12 INFO namenode.NameNode: STARTUP_MSG:
    /************************************************************
    STARTUP_MSG: Starting NameNode
    STARTUP_MSG:   host = localhost/127.0.0.1
    STARTUP_MSG:   args = [-format]
    STARTUP_MSG:   version = 1.1.2
    STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.0 -r 1393290; compiled by 'hortonfo' on Wed Oct  3 05:13:58 UTC 2012
    ************************************************************/
    13/02/14 01:43:13 INFO util.GSet: VM type       = 64-bit
    13/02/14 01:43:13 INFO util.GSet: 2% max memory = 17.77875 MB
    13/02/14 01:43:13 INFO util.GSet: capacity      = 2^21 = 2097152 entries
    13/02/14 01:43:13 INFO util.GSet: recommended=2097152, actual=2097152
    13/02/14 01:43:13 INFO namenode.FSNamesystem: fsOwner=shumin
    13/02/14 01:43:13 INFO namenode.FSNamesystem: supergroup=supergroup
    13/02/14 01:43:13 INFO namenode.FSNamesystem: isPermissionEnabled=true
    13/02/14 01:43:13 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
    13/02/14 01:43:13 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
    13/02/14 01:43:13 INFO namenode.NameNode: Caching file names occuring more than 10 times
    13/02/14 01:43:13 INFO common.Storage: Image file of size 112 saved in 0 seconds.
    13/02/14 01:43:14 INFO common.Storage: Storage directory /hadoop/tmp/dfs/name has been successfully formatted.
    13/02/14 01:43:14 INFO namenode.NameNode: SHUTDOWN_MSG:
    /************************************************************
    SHUTDOWN_MSG: Shutting down NameNode at localhost/127.0.0.1
    ************************************************************/

Start the HDFS daemons with command:

	$ start-dfs.sh

We will get output similar to the following:

    starting namenode, logging to /usr/local/hadoop/libexec/../logs/hadoop-hduser-namenode-localhost.out
    localhost: starting datanode, logging to /usr/local/hadoop/Hadoop/libexec/../logs/hadoop-hduser-datanode-localhost.out
    localhost: starting secondarynamenode, logging to /usr/local/hadoop/libexec/../logs/hadoop-hduser-secondarynamenode-localhost.out

The output shows that the following HDFS daemons have been started:
*NameNode*, *DataNode* and *SecondaryNameNode*.

Start the MapReduce daemons with command:

	$ start-mapred.sh

The output will be similar to:

    starting jobtracker, logging to /usr/local/hadoop/libexec/../logs/hadoop-hduser-jobtracker-localhost.out
    localhost: starting tasktracker, logging to /usr/local/hadoop/libexec/../logs/hadoop-hduser-tasktracker-localhost.out

The output shows that the following MapReduce daemons have been started:
JobTracker and TaskTracker.

With the jps command, we can get a list of all running daemons:

    10984 SecondaryNameNode
    11272 TaskTracker
    11144 JobTracker
    26966 NameNode
    10855 DataNode
    27183 Jps

So far, all the Hadoop daemons have been started.

Stop the MapReduce daemons with command:

	$ stop-mapred.sh

Stop the HDFS daemons with command:

	$ stop-hdfs.sh

### How it works... <a name="#how-it-works..."></a>

Under Unix-like operating systems, system runtime configurations and
environment variables are specified via plain text files. These files
are called run configuration file, meaning that they provide
configurations when the program runs. For example, file .bashrc under a
user’s home directory is the run configuration file for bash shell. It
will be sourced (loaded) automatically every time when a bash terminal
is opened. So, in this file, we can specify commands and environment
variables for a running bash environment.

`.bashrc` OR `.bash_profile`

Under Linux, the bash shell has two run configuration files for a user,
`.bashrc` and `.bash_profile`. The difference between the two files is
that `.bash_profile` is executed for login shells, while `.bashrc` for
interactive non-login shells. More specifically, when we login to the
system by entering username and password either locally or from a remote
machine `.bash_profile` will be executed and a bash shell process
initialized. On the other hand, if we open a new bash terminal after
logged into a machine or type the bash command from command line, file
`.bashrc` will be used for initialization before we see the command
prompt on the terminal window. In this recipe, we used file `.bashrc`,
so that new configurations will be available after opening a new bash
process. Alternatively, we can manually source a configuration file
after it is created or changed with the source command.

Table [s]hows configuration files for configuring a Hadoop cluster in
pseudo-distributed mode:

<span>ll</span> **File** & **Description**\
hadoop-env.sh & Configures environment variable used by Hadoop.\
core-site.xml & Configures parameters for the whole Hadoop cluster.\
hdfs-site.xml & Configures parameters for HDFS and its clients.\
mapred-site.xml & Configures parameters for MapReduce and its clients.\
masters & Configures host machines for SecondaryNameNode.\
slaves & Configures a list of slave node hosts.\

- *hadoop-env.sh* specifies environment variables for running Hadoop.
    For example, the home directory of Java installation `JAVA_HOME` and
    those related to Hadoop runtime options and cluster logging etc.

- *core-site.xml* specifies the URI of HDFS NameNode and MapReduce
    JobTracker. Value <hdfs://localhost:54310> of the fs.default.name
    property specifies the location of the default file system as HDFS
    on localhost using port 54310. We can specify other file system
    schemes such as local file system with <file:///home/hduser/hadoop>
    and amazon web service S3 with <s3://a-bucket/hadoop> etc. Value
    localhost:54311 of the mapred.job.tracker property specifies the URI
    of the cluster’s JobTracker.

- *hdfs-site.xml* specifies the HDFS related configurations. For
    example, dfs.replication configures the replication factor of data
    blocks on HDFS. For example, the value 2 specifies that each data
    block will be replicated twice on the file system. Property
    dfs.data.dir specifies the location of the data directory on the
    host Linux file system.

- *mapred-site.xml* specifies configurations for the MapReduce
    framework. For example, we can configure the total number of jvm
    tasks, the number of map slots and reduce slots on a slave node and
    the amount of memory for each task etc.

- *masters* file specifies hosts that will run a SecondaryNameNode
    daemon. In our single node configuration, we put localhost in this
    file. And a SecondaryNameNode daemon will be started on localhost,
    which has been verified with the jps command.

- slaves file specifies slave nodes that run tasks controlled by task
    trackers. In our pseudo-distributed mode configuration, localhost is
    the only slave node in the cluster.

Hadoop provides a number of bash scripts for convenience of starting and
stopping a cluster. Table [tbl:start.stop.scripts] shows these scripts.

<span>p<span>.18</span>p<span>.73</span></span> **Script** &
**Description**\
**start-dfs.sh** & Script to start HDFS daemons including NameNode,
SecondaryNameNode and DataNode. A PID file will be created for each
daemon process under default folder \$<span>hadoop.tmp.dir</span>. For
example, if user hduser is used to run the script, file
/hadoop/tmp/hadoop-hduser-namenode.pid will be created for the NameNode
daemon process.\
**stop-dfs.sh** & Script to stop HDFS daemons. This command will try to
find the PIDs of the HDFS daemons and kill the processes with the PIDs.
So, if the PID file is missing, this script will not work.\
**start-mapred.sh** & Script to start MapReduce daemons, including the
JobTracker and TaskTrackers. Similar to start-hdfs.sh script, PIDs files
will be created for each daemon process.\
**stop-mapred.sh** & Script to stop Hadoop MapReduce daemons. Similar to
stop-dfs.sh script, the script will try to find the PID files and then
kill those processes.\
**start-all.sh** & Equals to start-dfs.sh plus start-mapred.sh.\
**stop-all.sh** & Equals to stop-dfs.sh plus stop-mapred.sh.\

### There’s more... <a name="#theres-more...-3"></a>

Currently, Hadoop is also available in rpm format. So we can use the
following command to install Hadoop:

	$ sudo rpm -ivh http://www.poolsaboveground.com/apache/hadoop/common/stable/hadoop-1.1.2-1.x86_64.rpm

The locations of installed files will be different from the tar ball
method. And we can check the file layout with command:

	$ rpm -ql hadoop

Then we can use the following command to configure a Hadoop cluster in
single node:

	$ sudo hadoop-setup-single-node.sh

### See also <a name="#see-also-6"></a>

- Configuring Hadoop in fully distributed mode

- Validating Hadoop installation in Chapter

## Configuring Hadoop in fully-distributed mode

To configure a Hadoop cluster in fully-distributed mode, we need to
configure all the master and slave machines. Although different from the
pseudo-distributed mode, the configuration experience will be similar.
In this recipe, we will outline steps to configure Hadoop in
fully-distributed mode.

### Getting ready <a name="#getting-ready-10"></a>

In this book, we propose to configure a Hadoop cluster with 1 master
node and 5 slave nodes. The hostname of the master node is 1 and the
hostnames of the slave nodes are: **slave1, slave2, slave3, slave4**,
and **slave5**.

Before getting started, we assume that Linux has been installed on all
the cluster nodes and we should validate password-less login with the
following commands on the master node:

	$ ssh hduser@slave1
	$ ssh hduser@slave2
	...

Unlike the pseudo-distributed mode, configuring a Hadoop cluster in
fully-distributed mode requires the successful configuration of all the
nodes in the cluster. Otherwise, the cluster will not work as expected.

We should be cautious about the interconnection of the cluster nodes.
Connection problems might be caused by configurations of firewalls,
network and so on. Assuming file `$HADOOP_HOME/conf/slaves` contains
hostnames of the slave nodes, we can use the following command to check
the password-less login to all slave nodes from the master node:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo "Testing ssh from master to node" $host
		ssh hduser@$host
	done

### How to do it... <a name="#how-to-do-it...-3"></a>

Use the following recipe to configure Hadoop in fully-distributed mode:\
Login to the master node from administrator machine with command:

	$ ssh hduser@master

Copy the Hadoop archive to the `/usr/local` directory:

	$ sudo cp hadoop-1.1.2.tar.gz /usr/local

Decompress the Hadoop archive:

	$ cd /usr/local
	$ sudo tar xvf hadoop-1.1.2.tar.gz

Make proper soft link for Hadoop root directory.

	$ sudo ln -s hadoop-1.1.2 hadoop

Use your favorite text editor to open file `~/.bashrc` and add the
following content:

	export JAVA_HOME=/usr/java/latest
	export HADOOP_HOME=/usr/local/Hadoop
	export PATH=$PATH:$JAVA_HOME/bin:HADOOP_HOME/bin

Open file `$HADOOP_HOME/conf/hadoop-env.sh` with your favorite text
editor and add the following content:

	export JAVA_HOME=/usr/java/latest

Open file `$HADOOP_HOME/conf/core-site.xml` with your favorite text
editor and add the following content:

```xml
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://master:54310</value>
  </property>

  <property>
    <name>mapred.job.tracker</name>
    <value>master:54311</value>
  </property>
</configuration>
```

Open file `$HADOOP_HOME/conf/hdfs-site.xml` with your favorite text
editor and add the following content into the file:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>

  <property>
    <name>dfs.data.dir</name>
    <value>/hadoop/data/</value>
  </property>

  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop/tmp/</value>
  </property>
</configuration>
```

Open file `$HADOOP_HOME/conf/mapred-site.xml` with your favorite text
editor and add the following content:

```xml
<configuration>
  <property>
    <name>mapred.tasktracker.map.tasks.maximum</name>
    <value>6</value>
  </property>

  <property>
    <name>mapred.tasktracker.reduce.tasks.maximum</name>
    <value>6</value>
  </property>

  <property>
    <name>mapred.map.child.java.opts</name>
    <value>-Xmx512m</value>
  </property>

  <property>
    <name>mapred.reduce.child.java.opts</name>
    <value>-Xmx512m</value>
  </property>

</configuration>
```

Configure file `$HADOOP_HOME/conf/masters` with command:

	$ sudo echo "master" > $HADOOP_HOME/conf/masters

This will configure the master node to run SecondaryNameNode.\
Open file `$HADOOP_HOME/conf/slaves` with your favorite text editor and
add all the slave node hostnames into the file similar to the following:

    slave1
    slave2
    slave3
    ...

Copy the configured Hadoop directory to all the slave nodes with
command:

	for host in `cat $HADOOP_HOME/conf/slaves`
		do
		echo "Configuring hadoop on slave node " $host
		sudo scp -r /usr/local/hadoop-1.1.2 hduser@$host:/usr/local/
		echo "Making symbolic link for Hadoop home directory on host " $host
		sudo ssh hduser@$host -C "ln -s /usr/local/hadoop-1.1.2 /usr/local/hadoop"
	done

The for-loop command will recursively copy the `/usr/local/hadoop-1.1.2`
directory to each node specified in file `$HADOOP_HOME/conf/slaves`. And
a symbolic link is made on each node for the Hadoop directory. We can
get the following output information:

    Configuring hadoop on slave node slave1
    Making symbolic link for Hadoop home directory on host host slave1
    Configuring hadoop on slave node slave2
    Making symbolic link for Hadoop home directory on host host slave2
    Configuring hadoop on slave node slave3
    Making symbolic link for Hadoop home directory on host host slave3
    Configuring hadoop on slave node slave4
    Making symbolic link for Hadoop home directory on host host slave4
    ...

Copy the bash configuration file to each slave node with command:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo "Copying local bash run configuration file to host " $host
		sudo cp ~/.bashrc $host:~/
	done

The for-loop command copies the bash run configuration file from the
master node to all the slave nodes in the cluster. We can get the
following output message:

    Copying local bash run configuration file to host slave1
    Copying local bash run configuration file to host slave2
    Copying local bash run configuration file to host slave3
    ...

Use the following recipe to start a Hadoop cluster:

Format the HDFS filesystem on the master node with command:

	$ hadoop namenode -format

If this is the first time to format the HDFS, the command should finish
automatically. If you are reformatting an existing filesystem, it will
ask you for permission to format the filesystem. For example, the output
information will contain message similar to the following:

`Re-format filesystem in /tmp/hadoop-shumin/dfs/name ? (Y or N)`

In such a case, we need to type “Y” to confirm the reformatting of the
filesystem. Be cautious that all the data will be wiped out after you
hit the “Enter” key.

Check the directory structure of the formatted NameNode with command:

	$ tree /hadoop/dfs/

The output will be similar to the following:

    /hadoop/dfs/
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

The tree listing shows the directory structure of a formatted HDFS
filesystem which contains the filesystem image (in folder
`/hadoop/dfs/name/image` directory) and the current live image (mirrored
to folder `/hadoop/dfs/name/current`) in main memory.

Start HDFS cluster daemons with command:

	start-dfs.sh

And we will get output similar to the following:

    starting namenode, logging to /usr/local/hadoop/logs/hadoop-hduser-namenode-master.out
    slave1: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-sslave1.out
    slave2: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-slave2.out
    slave3: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-slave3.out
    slave4: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-slave4.out
    slave5: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-slave5.out
    master: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hduser-secondarynamenode-hadoop-master.out

The output message shows that a NameNode and a SecondaryNameNode are
started on the master node. And a DataNode is started on each slave
node.\
Start the MapReduce cluster daemons with command:

	$ start-mapred.sh

The output similar to the following:

    starting jobtracker, logging to /usr/local/hadoop/logs/hadoop-hduser-jobtracker-master.out
    slave1: starting tasktracker, logging to /usr/local/Hadoop/logs/hadoop-hduser-tasktracker-slave1.out
    slave2: starting tasktracker, logging to /usr/local/Hadoop/logs/hadoop-hduser-tasktracker-slave2.out
    slave3: starting tasktracker, logging to /usr/local/Hadoop/logs/hadoop-hduser-tasktracker-slave3.out
    slave4: starting tasktracker, logging to /usr/local/Hadoop/logs/hadoop-hduser-tasktracker-slave4.out
    slave5: starting tasktracker, logging to /usr/local/Hadoop/logs/hadoop-hduser-tasktracker-slave5.out

The output message shows that a JobTracker is started on the master node
and a TaskTracker is started on each slave node.

On the master node, check the status of the Hadoop daemons with command:

    $ jps
    19512 NameNode
    19930 JobTracker
    19708 SecondaryNameNode
    20276 Jps

On a slave node, we can check the status of the daemon processes with
the same command and the output will be similar to the following:

    3949 Jps
    3639 TaskTracker
    3501 DataNode

The highlighted daemons in the previous two steps must be present.
Otherwise there are be configuration problems. You can review recipe
Validating Hadoop installation for troubleshooting and debugging
suggestions.

List all the available TaskTrackers with command:

    $ hadoop job -list-active-trackers
    tracker_slave1:slave1/10.0.0.2:38615
    tracker_slave2:slave2/10.0.0.3:39618
    tracker_slave3:slave3/10.0.0.4:48228
    tracker_slave4:slave4/10.0.0.5:42954
    tracker_slave5:slave5/10.0.0.6:43858

Check the status of each node in the HDFS cluster with command:

    $ hadoop dfsadmin -report
    Configured Capacity: 13500319031296 (12.28 TB)
    Present Capacity: 12015141961728 (10.93 TB)
    DFS Remaining: 4067084627968 (3.7 TB)
    DFS Used: 7948057333760 (7.23 TB)
    DFS Used%: 66.15%
    Under replicated blocks: 0
    Blocks with corrupt replicas: 0
    Missing blocks: 0

    -------------------------------------------------
    Datanodes available: 5 (5 total, 0 dead)

    Name: 192.168.1.14:50010
    Decommission Status : Normal
    Configured Capacity: 964306395136 (898.08 GB)
    DFS Used: 590553788416 (550 GB)
    Non DFS Used: 97300185088 (90.62 GB)
    DFS Remaining: 276452421632(257.47 GB)
    DFS Used%: 61.24%
    DFS Remaining%: 28.67%
    Last contact: Sat Feb 16 00:34:17 EST 2013

    ...

    Name: 192.168.1.17:50010
    Decommission Status : Normal
    Configured Capacity: 964262363136 (898.04 GB)
    DFS Used: 617057673216 (574.68 GB)
    Non DFS Used: 81531011072 (75.93 GB)
    DFS Remaining: 265673678848(247.43 GB)
    DFS Used%: 63.99%
    DFS Remaining%: 27.55%
    Last contact: Sat Feb 16 00:34:15 EST 2013

The output shows that there are 5 DataNodes in the cluster. And the
status of each DataNode such as capacity and percentage of usage is
reported.

Use the following two steps to stop a running Hadoop cluster:

Stop the MapReduce daemons with command on the master node:

    $ stop-mapred.sh
    stopping jobtracker
    slave3: stopping tasktracker
    slave2: stopping tasktracker
    slave5: stopping tasktracker
    slave4: stopping tasktracker
    slave1: stopping tasktracker

The output shows that the JobTracker on the master node and TaskTrackers
on the slave nodes are being stopped. Stop the HDFS daemons with command
on the master node:

    $ stop-dfs.sh
    stopping namenode
    slave3: stopping datanode
    slave4: stopping datanode
    slave2: stopping datanode
    slave1: stopping datanode
    slave5: stopping datanode
    localhost: stopping secondarynamenode

The output shows that the NameNode and SecondaryNameNode daemons on the
master node and the DataNode daemons on the slave nodes are being
stopped. Alternatively, we can use command `stop-all.sh` to stop all the
running Hadoop daemons.

### How it works <a name="#how-it-works-9"></a>

The following table shows the properties used in this recipe:

`fs.default.name` The URI of the default filesystem.

`mapred.job.tracker` The URI of the JobTracker, for example
`localhost:54310`

`dfs.replication` How many nodes a block should be replicated
to. The default value of this property is 3.

`dfs.data.dir` The local storage directory of data blocks on
DataNodes.

`hadoop.tmp.dir` A base directory for a number of other
directories.

`mapred.tasktracker.map.tasks.maximum` Max number of parallel
map tasks that a TaskTracker can run.

`mapred.tasktracker.reduce.tasks.maximum` Max number of
parallel reduce tasks that a TaskTracker can run.

`mapred.map.child.java.opts` The Java options for the map
task child processes.

`mapred.reduce.child.java.opts` The Java options for the
reduce task child processes.

### There’s more... <a name="#theres-more...-4"></a>

Alternatively, we can use the following steps to configure a
fully-distributed Hadoop cluster: 

Download Hadoop rpm package on the administrator machine with command:

	$ wget http://www.poolsaboveground.com/apache/hadoop/common/stable/hadoop-1.1.2-1.x86_64.rpm -P ~/repo

Login to the master node with command:

	$ ssh hduser@master

Use the following command to install Hadoop on all nodes:

	for host in master slave1 slave2 slave3 slave4 slave5; do
		echo "Installing Hadoop on node: " $host
		sudo rpm -ivh ftp://hadoop.admin/repo/hadoop-1.1.2-1.x86_64.rpm
	done

Configure the Hadoop cluster by modifying the configuration files
located in folder `/etc/hadoop`.

### See also <a name="#see-also-7"></a>

- Configuring Hadoop in pseudo-distributed mode
- Validating Hadoop installation

## Validating Hadoop installation

The configuration of a Hadoop cluster is not done before the validation
step. Validation plays an important role in the configuration of a
Hadoop cluster, for example, it can help us figure out configuration
problems.

The most straightforward way to validate a Hadoop cluster configuration
is to run a MapReduce job from the master node. Alternatively, there are
two methods to validate the cluster configuration. One is from web
interface and the other is from the command line. In this recipe, we
will list steps to validate the configuration a Hadoop cluster.

### Getting ready <a name="#getting-ready-11"></a>

To validate the configuration from the web interface, a web browser such
as Firefox, Google Chrome etc. is needed. Sometimes if a GUI web browser
is not available, we can use a command line based web browser such as
elinks and lynx etc. In this book, we assume to use elinks for
illustration purpose.

We assume that elinks has been installed with command:

	$ sudo yum install elinks

Start all the Hadoop daemons with commands:

	$ start-dfs.sh
	$ start-mapred.sh

### How to do it... <a name="#how-to-do-it...-4"></a>

Use the following steps to run a MapReduce job:

Login to the master node with command:

	$ ssh hduser@master

Run a sample MapReduce job with command:

	$ hadoop jar $HADOOP_HOME/hadoop-examples*.jar pi 20 100000

In this command, `hadoop-examples*jar` is a jar file that contains a few
sample MapReduce jobs such as $\pi$. Option 20 is the number of tasks to
run and 100000 specifies the size of the sample for each task.

If this job finishes without any problem, we can say that the Hadoop
cluster is working. But this is not enough, because we also need to make
sure all the slave nodes are available for running tasks.

Use the following recipe to validate Hadoop cluster configuration
through web user interface:

Open URL `master:50030/jobtracker.jsp` with a web browser. The webpage
will be similar to Figure [fig:jobtracker.webui].

![Hadoop Jobtracker Web UI<span
data-label="fig:jobtracker.webui"></span>](figs/5163os_03_02.png)

The webpage shows that the Hadoop cluster contains 5 active slave nodes.

Check the status each slave node by clicking the link, which lead us to
a webpage similar to Figure [fig:active.trackers].

![List of active TaskTrackers<span
data-label="fig:active.trackers"></span>](figs/5163os_03_03.png)

From Figure [fig:active.trackers], we can easily check the status of the
active TaskTrackers on the slave nodes. For example, we can see the
count of failed tasks, the number of MapReduce slots and the heart beat
seconds etc.

Check the status of slave DataNodes by opening URL master:50070. The
webpage will be similar to Figure [fig:namenode.webui].

![Hadoop HDFS NameNode Web UI<span
data-label="fig:namenode.webui"></span>](figs/5163os_03_04.png)

The webpage shows that the cluster is configured with 5 active nodes.

By clicking the `Live Nodes` link we can see the details of
each node as shown in Figure [fig:hdfs.datanodes].

![List of HDFS live DataNodes<span
data-label="fig:hdfs.datanodes"></span>](figs/5163os_03_05.png)

This webpage shows the status of each slave node including node
capacity, percentage of usage and how many blocks are hosted in the node
etc.

Run an example teragen job to generate 10GB data on the HDFS with
command:

	$ hadoop jar $HADOOP_HOME/hadoop-examples-1.1.2.jar teragen $((1024*1024*1024* 10/100)) teraout

In this command, hadoop-examples-1.1.2.jar is the Java archive file
which provides a number of Hadoop examples. The option
`$((1024*1024*1024* 10/100))` tells us how many lines of data will be
generated with the total data size 10GB.

When the job is running, we can check the status of the job by opening
URL
[here](http://master:50030/jobdetails.jsp?jobid=job_201302160219_0003&refresh=30).

In this URL, `job_201302160219_0003` is the job ID and refresh=30 tells
how often the webpage should be refreshed.

The job status webpage will be similar to Figure [fig:mapreduce.job].

![Information about a MapReduce job from the Web UI<span
data-label="fig:mapreduce.job"></span>](figs/5163os_03_06.png)

Figure [fig:mapreduce.job] tells us that the Hadoop cluster has been
configured successfully!

After the teragen job finishes, we can check the node storage space
usage by opening URL
<http://master:50070/dfsnodelist.jsp?whatNodes=LIVE>. The webpage will
be similar to Figure [fig:hdfs.storage].

![Storage information of DataNodes from the Web UI<span
data-label="fig:hdfs.storage"></span>](figs/5163os_03_07.png)

The webpage shows that a certain percentage of storage space has been
used on each slave node.

Sometimes, a command line based web browser can be more handy than a GUI
browser, for example, we can use command `elinks master:50030` to check
the status of MapReduce on the master node and use command
`elinks master:50070` to check the status and health of HDFS.

Use the following recipe to validate the configuration a Hadoop cluster
from command line:

List all available TaskTrackers with command:

    $ hadoop job -list-active-trackers
    Example output is similar to the following:
    tracker_slave1:localhost/127.0.0.1:53431
    tracker_slave4:localhost/127.0.0.1:52644
    tracker_slave3:localhost/127.0.0.1:37775
    tracker_slave2:localhost/127.0.0.1:56074
    tracker_slave5:localhost/127.0.0.1:43541

The output confirms that all the configured TaskTrackers are active in
the Hadoop cluster.

Check the status of HDFS cluster with command:

    $ hadoop fsck /
    FSCK started by hduser from /10.0.0.1 for path / at Sat Feb 16 03:03:44 EST 2013
    ...............................Status: HEALTHY
     Total size:    7516316665 B
     Total dirs:    15
     Total files:   31
     Total blocks (validated):      125 (avg. block size 60130533 B)
     Minimally replicated blocks:   125 (100.0 %)
     Over-replicated blocks:        0 (0.0 %)
     Under-replicated blocks:       0 (0.0 %)
     Mis-replicated blocks:         0 (0.0 %)
     Default replication factor:    2
     Average block replication:     2.0
     Corrupt blocks:                0
     Missing replicas:              0 (0.0 %)
     Number of data-nodes:          5
     Number of racks:               1
    FSCK ended at Sat Feb 16 03:03:44 EST 2013 in 12 milliseconds


    The filesystem under path '/' is HEALTHY

The output gives us the same information as from the web interface. And
the last line tells us that the root filesystem is *HEALTHY*.

### How it works <a name="#how-it-works-10"></a>

Hadoop provides commands and web interfaces for system administrators to
check the status of the cluster. When we start Hadoop daemons, a
build-in web server will be started and a number of pre-written *jsp*
script files are used to respond to user’s requests from a web browser.
The jsp files can be found under the `$HADOOP_HOME/webapps` directory.
If you have programming experience, you can take advantage of the jsp
files to develop personalized Hadoop cluster management tools.

### There’s more... <a name="#theres-more...-5"></a>

In this part, we list a few typical Hadoop configuration problems and
give suggestions on dealing with these problems.

#### Can’t start HDFS daemons <a name="#cant-start-hdfs-daemons"></a>

There are many possible reasons that can cause this problem. For
example, the NameNode on the master node has not been formatted, in
which case, we can format the HDFS before starting the cluster with
command:

	$ hadoop namenode -format

Warning!\
Be cautious when formatting the filesystem with this command. It will
erase all the data on the file system. Always try other methods before
using this one.

More generically, to troubleshoot this problem, we need to check if HDFS
has been properly configured and daemons are running. This can be done
with the following command:

	$ jps

If the output of this command *does not* contain the NameNode and
SecondaryNameNode daemons, we need to check the configuration of HDFS.

To troubleshoot the HDFS startup problem, we can open a new terminal and
monitor the NameNode log file on the master node with the following
command:

	$ tail -f $HADOOP_HOME/logs/hadoop-hduser-namenode-master.log

This command will show the content of the log file in a dynamic way when
new log is appended to the file. If error happens, we can get error
messages similar to the following:

    2013-02-16 11:44:29,860 ERROR org.apache.hadoop.hdfs.server.namenode.NameNode: java.net.UnknownHostException: Invalid hostname for server: master1
            at org.apache.hadoop.ipc.Server.bind(Server.java:236)
            at org.apache.hadoop.ipc.Server$Listener.<init>(Server.java:302)
            at org.apache.hadoop.ipc.Server.<init>(Server.java:1488)
            at org.apache.hadoop.ipc.RPC$Server.<init>(RPC.java:560)
            at org.apache.hadoop.ipc.RPC.getServer(RPC.java:521)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:295)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:529)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1403)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1412)

    2013-02-16 11:44:29,865 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: SHUTDOWN_MSG:
    /************************************************************
    SHUTDOWN_MSG: Shutting down NameNode at master/10.144.150.104
    ************************************************************/

Alternatively, the following command will give the same error:

	$ hadoop jobtracker

The message above shows that the hostname of the NameNode is wrong. It
should be master instead of master1.

#### Cluster is missing slave nodes <a name="#cluster-is-missing-slave-nodes"></a>

Most probably, this problem is caused by hostname resolution. To
confirm, we can check the content of file `/etc/hosts` with:

    $ cat /etc/hosts
    10.0.0.1	master
    10.0.0.2	slave1
    10.0.0.3	slave2
    10.0.0.4	slave3
    10.0.0.5	slave4
    10.0.0.6	slave5

If the IP address and hostname mapping does not exist or has been
erroneously specified in this file, correcting the error can solve this
problem.

#### MapReduce daemons can’t be started <a name="#mapreduce-daemons-cant-be-started"></a>

The following two reasons can cause this problem:

The HDFS daemons are not running, which can cause the MapReduce daemons
to ping the NameNode daemon at a regular interval, which can be
illustrated with the following log output:

    13/02/16 11:32:19 INFO ipc.Client: Retrying connect to server: master/10.0.0.1:54310. Already tried 0           time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1 SECONDS)
    13/02/16 11:32:20 INFO ipc.Client: Retrying connect to server: master/10.0.0.1:54310. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1 SECONDS)
    13/02/16 11:32:21 INFO ipc.Client: Retrying connect to server: master/10.0.0.1:54310. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1 SECONDS)
    13/02/16 11:32:22 INFO ipc.Client: Retrying connect to server: master/10.0.0.1:54310. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1 SECONDS)
    13/02/16 11:32:23 INFO ipc.Client: Retrying connect to server: master/10.0.0.1:54310. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1 SECONDS).

To troubleshoot this problem, we can refer to tips for **can’t start
HDFS daemons**.

*Configuration problems of MapReduce*. Recall that we have
configurations for the number of map slots and reduce slots as well as
memory amount in the `$HADOOP_HOME/conf/mapred-site.xml` file. Before
starting a cluster, we need to make sure that the total amount of
configured memory should be smaller than the total amount of system
memory.

For example, suppose a slave host has 4GB of memory, and we have
configured 6 map slots and 6 reduce slots with memory 512MB for each
slot. So we can compute the total configured task memory with the
following formula: $6 \time 512 + 6 \times 512 = 6GB$.

As 6GB is larger than the system memory 4GB, the system will not start.
To clear this problem, we can decrease the number of map slots and
reduce slot from 6 to 3. This configuration gives us a total configured
memory of 3GB, which is smaller than the system total memory 4GB, thus
the MapReduce daemons should be able to start successfully.

### See also <a name="#see-also-8"></a>

- Configuring Hadoop in pseudo-distributed mode

- Configuring Hadoop in fully-distributed mode

## Configuring ZooKeeper

ZooKeeper provides highly reliable centralized service for maintaining
configuration information, naming and providing distributed
synchronization and group services. In this recipe, we will outline
steps to install ZooKeeper.

### Getting ready <a name="#getting-ready-12"></a>

Make sure Hadoop has been properly configured. Please refer to the
previous recipes in this chapter about installation of Hadoop on a
cluster.

Login to the master node from the Hadoop administrator machine as hduser
with command:

	$ ssh hduser@master

Download ZooKeeper archive file with commands:

	$ wget http://www.gtlib.gatech.edu/pub/apache/zookeeper/stable/zookeeper-3.4.5.tar.gz -P ~/repo

### How to do it... <a name="#how-to-do-it...-5"></a>

Use the following recipe to configure ZooKeeper:

Login to the master node with command:

	$ ssh hduser@master

Copy the downloaded archive to /usr/local with command:

	$ sudo wget ftp://hadoop.admin/repo/zookeeper-3.4.5.tar.gz -P /usr/local

De-compress the file with command:

	$ cd /usr/local/
	$ sudo tar xvf zookeeper-3.4.5.tar.gz

Create symbolic link with command:

	$ sudo ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper

Open file `~/.bashrc` and add the following lines:

    ZK_HOME=/usr/local/zookeeper
    export PATH=$ZK_HOME/bin:$PATH

Load the configuration file with:

	$ . ~/.bashrc

Create data and log directories for ZooKeeper with command:

	$ sudo mkdir -pv /hadoop/zookeeper/{data,log}

Create Java configuration file `$ZK_HOME/conf/java.env` with the
following content:

    JAVA_HOME=/usr/java/latest
    export PATH=$JAVA_HOME/bin:$PATH

The file name *java.env* is mandatory. It will be loaded by zookeeper.

Create file `$ZK_HOME/conf/zookeeper.cfg` and add the following lines:

    tickTime=2000
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=master:2888:3888
    server.2=slave1:2888:3888
    server.3=slave2:2888:3888
    server.4=slave3:2888:3888
    server.5=slave4:2888:3888
    server.6=slave5:2888:3888
    dataDir=/hadoop/zookeeper/data
    dataLogDir=/hadoop/zookeeper/log

The highlighted section makes every node know the other nodes in the
ZooKeeper ensemble.

Configure ZooKeeper on all slave nodes with command:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo "Configuring ZooKeeper on " $host
		scp ~/.bashrc hduser@$host:~/
		sudo scp -r /usr/local/zookeeper-3.4.5 hduser@$host:/usr/local/
		echo "Making symbolic link for ZooKeeper home directory on " $host
		sudo ssh hduser@$host -C "ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper"
	done
	
Start ZooKeeper on master node with command:

	$ zkServer.sh start

Verify ZooKeeper configuration with command:

	$ zkCli.sh -server master:2181

Stop ZooKeeper with command:

	$ zkServer.sh stop

### See also <a name="#see-also-9"></a>

- Installing HBase

- Get more documentation about ZooKeeper from [ZooKeeper Admin
    Documents](http://zookeeper.apache.org/doc/r3.4.5/zookeeperAdmin.html)

## Installing HBase

HBase is the database based on Hadoop. It is a distributed, scalable Big
Data storage system. In this section, we are going to list steps about
installing HBase in our Hadoop cluster.

### Getting ready <a name="#getting-ready-13"></a>

To install HBase, we assume Hadoop has been configured without any
issues.

Download HBase from a mirror site. Similar to downloading Hadoop, HBase
is hosted on mirrors all over the world. Visit link
<http://www.apache.org/dyn/closer.cgi/hbase/> and select the nearest
mirror (the suggested mirror on the top is the optimal choice). After
selecting the mirror, follow the link to select the HBase version, we
suggest the stable version, for example, follow the link
<http://mirror.quintex.com/apache/hbase/stable/> and can see the
downloadable files as shown in Figure [fig:hbase.download].

![Download a stable HBase release from a mirror site<span
data-label="fig:hbase.download"></span>](figs/5163os_03_08.png)

Click the file link hbase-0.94.5.tar.gz to download the file to the
administrator machine. Then, copy the file to the FTP repository with
command:

	$ cp hbase-0.94.5.tar.gz ~/repo

Alternatively, we can download the file with command:

	$ wget http://mirror.quintex.com/apache/hbase/stable/hbase-0.94.5.tar.gz -P ~/repo

### How to do it... <a name="#how-to-do-it...-6"></a>

Use the following recipe to install HBase:\
Login to the master node from administrator machine with command:

	$ ssh hduser@master

De-compress the HBase archive with commands:

	$ cd /usr/local
	$ sudo wget ftp://hadoop.admin/repo/hbase-0.94.5.tar.gz -P /usr/local
	$ sudo tar xvf hbase-0.94.5.tar.gz

Create symbolic link with command:

	$ ln -s hbase-0.94.5 hbase

Use your favorite text editor to open file  /.bashrc and append the
following lines into the file:

    export HBASE_HOME=/usr/local/hbase
    export PATH=$HBASE_HOME/bin:$PATH

Open file `$HBASE_HOME/conf/hbase-env.sh` and set `JAVA_HOME` as:

	$ export JAVA_HOME=/usr/java/latest

Open file `$HBASE_HOME/conf/hbase-site.xml` with your favorite text
editor and add the following contents to the file:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:54310/hbase</value>
  </property>

  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <property>
    <name>hbase.tmp.dir</name>
    <value>/hadoop/hbase</value>
  </property>

  <property>
    <name>hbase.ZooKeeper.quorum</name>
    <value>master</value>
  </property>

  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/hadoop/zookeeper</value>
  </property>
</configuration>
```

Open file `$HBASE_HOME/conf/regionservers` and add the following lines:

    slave1
    slave2
    slave3
    slave4
    slave5

Link the HDFS configure file to HBase configuration directory with
command:

	$ sudo ln -s $HADOOP_HOME/conf/hdfs-site.xml $HBASE_HOME/conf/hdfs-site.xml

Replace the dependent jar files for HBase with command:

	$ rm -i $HBASE_HOME/lib/hadoop-core*.jar $HBASE_HOME/lib/zookeeper-*.jar
	$ cp -i $HADOOP_HOME/hadoop-core*.jar $HADOOP_HOME/lib/commons-*.jar $ZK_HOME/zookeeper-*.jar $HBASE_HOME/lib/

Configure all the slave nodes with command:

	for host in `cat $HBASE_HOME/conf/regionservers`; do
		echo "Configuring HBase on " $host
		scp ~/.bashrc hduser@$host:~/
		sudo scp -r /usr/local/hbase-0.94.5 hduser@$host:/usr/local/
		echo "Making symbolic link for HBase home directory on " $host
		sudo ssh hduser@$host -C "ln -s /usr/local/hbase-0.94.5 /usr/local/hbase"
		echo "Making symbolic link for hdfs-site.xml to the HBase configuration directory on " $host
		sudo ssh hduser@$host -C "ln -s /usr/local/hadoop-1.1.2/conf/hdfs-site.xml /usr/local/hbase-0.94.5/conf/hdfs-site.xml"
	done

Start HBase daemons with command:

	$ start-hbase.sh

Connect to the running HBase with command:

	$ hbase shell

Verify the HBase installation with the following HBase shell commands:

    hbase(main):001:0> create 'test', 'c'
    0 row(s) in 0.2410 seconds

    hbase(main):001:0> put 'test', 'r1', 'c:a', 'v1'
    0 row(s) in 0.0320 seconds

    hbase(main):003:0> scan 'test'
    ROW COLUMN+CELL row1 column=c:a, timestamp=124455459102, value=v1 r1
    1 row(s) in 0.2130 seconds

    hbase(main):006:0> disable 'test'
    0 row(s) in 9.4210 seconds

    hbase(main):007:0> drop 'test'
    0 row(s) in 8.3412 seconds

    hbase(main):010:0> exit

To stop HBase, use command:

    $ stop-hbase.sh
    stopping hbase...............

### How it works <a name="#how-it-works-11"></a>

In the configuration, property hbase.rootdir specifies the root
directory of the HBase data storage. And property
*hbase.zookeeper.property.dataDir* specifies the root directory of the
ZooKeeper data storage.

### There’s more... <a name="#theres-more...-6"></a>

- Installing ZooKeeper in Chapter 3, Configuring a Hadoop cluster

- More documentation about HBase can be found
    [here](http://wiki.apache.org/hadoop/Hbase)

## Installing Hive

As a top level abstraction language, Hive provides a handy tool for
manipulating data storage on HDFS with SQL like language. In this
section, we will talk about installing Hive on our Hadoop cluster.

### Getting ready <a name="#getting-ready-14"></a>

Before we install Hive, we need to make sure Hadoop has been properly
installed. Please refer to the previous sections about the configuration
of a Hadoop cluster.\
Download Hive from a mirror site with command similar to the following
on the administrator machine:

	$ wget http://apache.osuosl.org/hive/stable/hive-0.9.0.tar.gz -P ~/repo

### How to do it... <a name="#how-to-do-it...-7"></a>

Use the following steps to install Hive:

Login to the master node from the Hadoop administrator machine as hduser
with command:

	$ ssh hduser@master

Copy the archive to /usr/local with command:

	$ sudo wget ftp://hadoop.admin/repo/hive-0.9.0.tar.gz /usr/local

De-compress the Hive archive with command:

	$ cd /usr/local
	$ tar xvf hive-0.9.0.tar.gz

Create symbolic link with command:

	$ ln -s /usr/local/hive-0.9.0 /usr/local/hive

Use your favorite text editor to open file `~/.bashrc` and add the
following lines to this file:

	export HIVE_HOME=/usr/local/hive
	export PATH=$HIVE_HOME/bin:$PATH

Start Hive with command:

	$ hive

### There’s more... <a name="#theres-more...-7"></a>

- Installing Pig in Chapter 3, Configuring a Hadoop cluster

- Get more documentation about Hive from [Hive
    Home](https://cwiki.apache.org/confluence/display/Hive/Home)

## Installing Pig

Similar to Hive, Pig provides a handy tool for manipulating Hadoop data.
In this recipe, we are going to discuss the installation of Apache Pig.

### Getting ready <a name="#getting-ready-15"></a>

Before we install Pig, we need to make sure Hadoop has been properly
installed. Please refer to the previous sections about the configuration
of a Hadoop cluster.

Download Pig the archive file from a mirror site with command on the
administrator machine:

	$ wget http://www.motorlogy.com/apache/pig/stable/pig-0.10.1.tar.gz ~/repo

### How to do it... <a name="#how-to-do-it...-8"></a>

Use the following steps to configure Pig:

Login to the master node from the Hadoop administrator machine as hduser
with command:

	$ ssh hduser@master

Copy the archive to `/usr/local` with the following command:

	$ sudo wget ftp://hadoop.admin/repo/pig-0.10.1.tar.gz /usr/local

De-compress the Pig archive file with command:

	$ cd /usr/local
	$ sudo tar xvf pig-0.10.1.tar.gz

Create symbolic link to the Pig directory:

	$ sudo ln -s /usr/local/pig-0.10.1 /usr/local/pig

Open file  /.bashrc with your favorite text editor and add the following
lines into the file:

	export PIG_HOME=/usr/local/pig
	export PATH=$PIG_HOME/bin:$PATH

Run Pig in local mode with command:

	$ pig -x local

Run Pig in MapReduce mode with command:

	$ pig

Alternatively, we can use the following command:

	$ pig -x mapreduce
	
Pig that runs in MapReduce mode will utilizes the power of distributed
computing provided by Hadoop.

### There’s more... <a name="#theres-more...-8"></a>

- Installing Hive in Chapter 3, Configuring a Hadoop cluster

- More documentation about Pig can be obtained from [Pig @
    Apache](http://pig.apache.org/docs/r0.10.0/)

## Installing Mahout

Apache Mahout is a machine learning library that scales machine learning
algorithms on Big Data. It is implemented on top of the Hadoop Big Data
stack. It already has implementation of a wide range of machine learning
algorithms. In this recipe, we will outline steps to configure Apache
Mahout.

### Getting ready <a name="#getting-ready-16"></a>

Before we install Mahout, we need to make sure Hadoop has been properly
installed.

Download Mahout from the mirror site with the following command on the
master node:

	$ wget http://www.eng.lsu.edu/mirrors/apache/mahout/0.7/mahout-distribution-0.7.tar.gz -P ~/repo

### How to do it... <a name="#how-to-do-it...-9"></a>

Use the following the recipe to install Mahout: Login to the master node
from the Hadoop administrator machine as hduser with command:

	$ ssh hduser@master

Copy the archive to /usr/local with command:

	$ sudo wget ftp://hadoop.admin/repo/mahout-distribution-0.7.tar.gz /usr/local

De-compress the Mahout archive with command:

	$ cd /usr/local
	$ sudo tar xvf mahout-distribution-0.7.tar.gz
	
Create symbolic link to the Mahout directory with command:

	$ sudo ln -s /usr/local/mahout-distribution-0.7 /usr/local/mahout

Open file `/.bashrc` with your favorite text editor and add the following
lines to the file:

	export MAHOUT_HOME=/usr/local/pig
	export PATH=$MAHOUT_HOME/bin:$PATH

Load the configuration with command:

	$ . ~/.bashrc

Install Maven with command:

	$ sudo yum install maven

Compile and install Mahout core with the following commands:

	$ cd $MAHOUT_HOME
	$ sudo mvn compile
	$ sudo mvn install

The install command will run all the tests by default, we can ignore the
tests to speed up the installation process with command sudo
mvn -DskipTests install.

Compile the Mahout examples with commands:

	$ cd examples
	$ sudo mvn compile

Use the following steps to verify Mahout configuration:

Download sample data with command:

	$ wget http://archive.ics.uci.edu/ml/databases/synthetic_control/synthetic_control.data -P ~/

Start the Hadoop cluster with commands:

	$ start-dfs.sh
	$ start-mapred.sh

Put the downloaded data into HDFS with commands:

	$ hadoop fs -mkdir testdata
	$ hadoop fs -put ~/synthetic_control.data testdata

Run kmeans clustering with command:

	$ mahout org.apache.mahout.clustering.syntheticcontrol.kmeans.Job

### There’s more... <a name="#theres-more...-9"></a>

- More documentation about Mahout can be obtained from the [Mahout
    Wiki](https://cwiki.apache.org/confluence/display/MAHOUT/Mahout+Wiki)

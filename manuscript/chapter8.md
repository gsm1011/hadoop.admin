# Building Hadoop Cluster with Amazon EC2 and S3 

In this chapter, we will cover:

- Registering with Amazon Web Services (AWS)
- Managing AWS security credentials
- Preparing local machine for EC2 connection
- Creating an Amazon Machine Image (AMI)
- Using S3 to host data
- Configuring a Hadoop cluster with the new AMI

## Introduction 

Amazon Elastic Cloud Computing (EC2) and Simple Storage Service (S3) are
cloud computing web services provided by Amazon Web Services (AWS). EC2
offers platform as a service (PaaS), with which we can start up
theoretically unlimited number of servers on the cloud. S3 offers
storage services on the cloud. More information about AWS, EC2 and S3
can be obtained from [aws.amazon.com](aws.amazon.com).

From the previous chapters of this book, we know that the configuration
of a Hadoop cluster requires a big amount of hardware investment. For
example, to setup a Hadoop cluster, a number of computing nodes and
networking devices are required. Comparatively, with the help of AWS
cloud computing, especially EC2, we can setup a Hadoop cluster with
minimum cost and much less efforts.

In this chapter, we are going to discuss topics of configuring a Hadoop
cluster in the amazon cloud. We will guide you through the recipes of
registering with AWS, creating Amazon Machine Image (AMI) and
configuring a Hadoop cluster with the new AMI and so on.

## Registering with Amazon Web Services (AWS)

To use AWS, registration is required. The steps to register with AWS are
straightforward. In this recipe, we will outline steps to do this.

### Getting ready 

We assume to use a GUI web browser for AWS registration. So, we are
assuming you already have a web browser with internet access. In
addition, personal information needs to be prepared to fill the online
registration forms.

### How to do it... 

We can use the following steps to register with AWS:

Use a web browser to open the following link: <http://aws.amazon.com/>.

Click the ’**Sign Up**’ button on the upper right corner of the window.

You will be directed to a webpage as shown in Figure [fig:aws.signin].

![Sign in or Create and AWS Account<span
data-label="fig:aws.signin"></span>](figs/5163os_08_01.png)

Fill in the email address in the text field with label ’**My e-mail
address is**:’ and select the ’**I am a new user**’ radio button as
shown in the screenshot.

Click the ’**Sign in using our secure server**’ button at the bottom as
shown in the screenshot.

Fill the login credentials form, which includes name, email and password
as shown in Figure [fig:login.credentials].

![The Login Credentials Form<span
data-label="fig:login.credentials"></span>](figs/5163os_08_02.png)

Click the ’**Continue**’ button at the bottom as shown in the
screenshot.

Fill in the ’**Contact Information**’, ’**Security Check**’ and ’**AWS
Customer Agreement**’ form as shown in Figure [fig:aws.personal.info].

![AWS Signin Personal Information Page<span
data-label="fig:aws.personal.info"></span>](figs/5163os_08_03.png)

Click the ’**Create Account and Continue**’ button at the bottom as
shown in the screenshot.

So far, an AWS account has been created. Now we can login to AWS with
the newly created account by using the ’**returning user**’ option as
shown in Figure [fig:aws.signin.return].

![AWS signin with Returning User<span
data-label="fig:aws.signin.return"></span>](figs/5163os_08_04.png)

By clicking the ’**Sign in using our secure server**’ button on the
bottom of the window, we will be able to login to the AWS management
console page with EC2 and S3 service available as shown in Figure
[fig:aws.management.console].

![The AWS Management Console<span
data-label="fig:aws.management.console"></span>](figs/5163os_08_05.png)

Till here, we finished the registration step successfully.

### See also 

- <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html?r=1874>

## Managing AWS security credentials

Security credentials are critical for web services such as EC2 and S3.
They are used for remote access of the cloud servers on AWS. For
example, in this chapter, we will use these credentials to login to the
servers remotely from a client machine.

AWS provides web interface to manage security credentials, this recipe
will guide you through the steps to create, download and manage these
security credentials.

### Getting ready 

Before getting started, we assume that you have successfully registered
with AWS. Otherwise, you need to follow the steps in the previous recipe
to register with AWS.

We also assume that we have a client machine with Linux (such as CentOS)
installed. The machine should be able to access the internet and has at
least one GUI web browser installed.

Create a directory for storing AWS credentials with command:

	$ mkdir -v ~/.ec2

### How to do it... 

Use the following recipe to manage AWS security credentials:

Open a web browser and go to URL [aws.amazon.com](aws.amazon.com).

Click the ’**My Account / Console**’ dropdown button on the upper left
of the window as shown in Figure [fig:secu.select].

![Choose the Security Credentials button<span
data-label="fig:secu.select"></span>](figs/5163os_08_07.png)

Click the ’**Security Credentials**’ in the dropdown list as shown in
the previous screenshot.

If you have logged in to AWS previously, you will be able to visit the
security credentials management page. Otherwise, a login window will
appear. You need to type in the user name and password and login as a
’**returning user**’.

Currently, Amazon AWS has a few types of credentials as shown in Figure
[fig:aws.credentials.types].

![AWS Types of Security Credentials<span
data-label="fig:aws.credentials.types"></span>](figs/5163os_08_08.png)

Access Credentials include Access Keys, X.509 Certificates and Key pairs
as shown in Figure [fig:aws.access.credentials].

![AWS Access Credentials<span
data-label="fig:aws.access.credentials"></span>](figs/5163os_08_09.png)

By clicking the ’**Make Inactive**’ link on the status column of the
access keys table, we can make the access keys inactive. Inactive access
keys can be made active again and can be deleted from the list.

![X.509 Certificates<span
data-label="fig:aws.x509certificates"></span>](figs/5163os_08_11.png)

Similarly, we can make a X.509 certificate inactive. Inactive
certificates can be made active again or deleted from the list.

By clicking the ’**Create a new Certificate**’ link we will be able to
create a new certificate as shown in Figure [fig:aws.create.x509].

![Create X.509 Certificates<span
data-label="fig:aws.create.x509"></span>](figs/5163os_08_10.png)

We need to download the private key file as well as the X.509
certificate by clicking on the buttons as shown in the screenshot. These
files should be kept security and never shared with any other people.

Key pairs used for EC2 can be managed from the management console as
shown in Figure [fig:aws.keypair.management].

![AWS Managing Key Pairs<span
data-label="fig:aws.keypair.management"></span>](figs/5163os_08_14.png)

New key pairs can be created by clicking the ’**Create Key Pair**’
Button on the top of the window. A pop up window will be used to type in
the name of the key pair. And the newly created key pair will be
downloaded to local machine. Use the following command to copy the
downloaded key pair to `~/.ec2` folder. `$ cp *.pem ~/.ec2/`

### How it works 

Table [tbl:awssecurity] show the usage of each security credential:

<span>lll</span> **Security Credential** & & **Used for**\
Access Credentials & Access Keys & Secure REST or Query request access
to AWS service APIs.\
& X.509 Certificates & Make SOAP protocol requests to AWS service APIs.\
& Key Pairs & Launch and securely access EC2 instances.\
Sign-In Credentials & & Login to AWS from the web portal.\
Account Identifiers & & Share resources between accounts.\

## Preparing local machine for EC2 connection

A local client machine is required for accessing EC2. For example, we
can use the local client machine to launch EC2 instances, login to the
instances on EC2 and so on. In this recipe we will list steps to
configure a local machine for EC2 connection.

### Getting ready 

Before getting started, we assume that we have registered with AWS and
security credentials have been created. We also assume that a machine
with Linux has been installed.

### How to do it... 

Use the following recipe to configure a local machine for EC2 remote
access:

Get the Access Key ID and the secret access key from the security
credentials web page as shown in Figure [fig:aws.access.keys].

![AWS Access Keys<span
data-label="fig:aws.access.keys"></span>](figs/5163os_08_15.png)

Move the key pair to the .ec2 directory with command:

	$ mv <key-pair-name>.pem ~/.ec2/

Move the private key file to the .ec2 directory with command:

	$ mv pk-*.pem ~/.ec2/

Move the certificate file to the .ec2 directory with command:

	$ mv cert-*.pem ~/.ec2/` Download EC2 command line tools from URL
<http://aws.amazon.com/developertools/351>.

Decompress the zip file with command:

	$ unzip ec2-ami-tools.zip

Add the following content into file `~/.profile`:

    export EC2_HOME=~/ec2/
    export AWS_ACCOUNT_ID=example@mail.com
    export EC2_PRIVATE_KEY=~/.ec2/pk-WXP232J7PL4TR5JKMUBJGH65DFNCYZGY.pem
    export EC2_CERT=~/.ec2/cert-WXP232J7PL4TR5JKMUBJGH65DFNCYZGY.pem
    export AWS_ACCESS_KEY_ID=AKIAJ7GAQT52MZKJA4WQ
    export AWS_SECRET_ACCESS_KEY=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
    export PATH=$PATH:$EC2_HOME/bin

Add the following content into file `~/.bashrc`:

	$ . ~/.profile

Test the configuration with command:

	$ ec2-describe-images

If the configuration has no problem, we will get a list of AMIs.
Otherwise, we will be able to get error similar to the following:

    Client.AuthFailure: AWS was not able to validate the provided access credentials.

## Creating an Amazon Machine Image (AMI)

Amazon Machine Image (AMI) is the machine image used by EC2. An AMI is a
template that contains configuration for operating system and software
packages. We can start EC2 instances from pre-existing or personalized
AMIs. AWS offers a large number of public AMIs that can be used for
free.

Generally, there are types of AMIs, one is EBS (Elastic Block Storage)
backed AMI and the other is instance store backed AMI. In this recipe,
we will first outline steps to create an instance store backed AMI and
briefly introduce how to create an EBS backed AMI.

### Getting ready 

Before getting started, we assume that you have successfully registered
with AWS. And we also assume that a client machine has been configured
properly to connect to AWS.

Login to the local machine and install the MAKEDEV utility with
command:

	$ sudo yum install -y MAKEDEV

Install the Amazon AMI tools with command:

    $sudo rpm -ivh  http://s3.amazonaws.com/ec2-downloads/ec2-ami-tools.noarch.rpm
    Retrieving http://s3.amazonaws.com/ec2-downloads/ec2-ami-tools.noarch.rpm
    Preparing...                #################################### [100%]
       1:ec2-ami-tools          #################################### [100%]

### How to do it... 

Use the following recipe to create an instance store backed AMI:

Create an image file with command:

	$ dd if=/dev/zero of=centos.img bs=1M count=1024

This command will emit the following message:

    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 10.5981 s, 101 MB/s

In this command, if specifies the input of for the data, `/dev/zero` is
a special device on Linux systems; of specifies the output of the
command, here we specify a file name as the image name; bs specifies the
size of the blocks and count is the number of blocks for input to
output. The size of the output file **centos.img** is determined by the
block size and count. For example, the above command creates an image
file of size $1M \times 1024$ which is around $1.0GB$.

Check the size of the image file with command:

	$ ls -lh centos.img
	-rw-rw-r--. 1 shumin shumin 1.0G May  3 00:14 centos.img

Create a root file system inside the image file with command:

    $ mke2fs -F -j centos.img
    mke2fs 1.42.3 (14-May-2012)
    Discarding device blocks: done
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    655360 inodes, 2621440 blocks
    131072 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=2684354560
    80 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done

Create a directory under the `/mnt` directory with command:

	$ sudo mkdir -v /mnt/centos

Mount the image file to the folder with command:

	$ sudo mount -o loop centos.img /mnt/centos

Create the `/dev` directory under the root directory of the mounted file
system with command:

	$ sudo mkdir -v /mnt/centos/dev

Create a minimal set of devices with commands:

	$ sudo /sbin/MAKEDEV -d /mnt/centos/dev -x console
	$ sudo /sbin/MAKEDEV -d /mnt/centos/dev -x null
	$ sudo /sbin/MAKEDEV -d /mnt/centos/dev -x zero

These commands will give us the following output:

    MAKEDEV: mkdir: File exists
    MAKEDEV: mkdir: File exists
    MAKEDEV: mkdir: File exists

The reason for these warning messages is because the parent directories
already exists, when the MAKEDEV command tries to create the folder with
the mkdir command, it will fail with this warning.

Create the fstab configuration file with command:

	$ sudo mkdir -pv /etc/fstab

Put the following content into the file:

    /dev/sda1  /         ext3    defaults        1 1
    none       /dev/pts  devpts  gid=5,mode=620  0 0
    none       /dev/shm  tmpfs   defaults        0 0
    none       /proc     proc    defaults        0 0
    none       /sys      sysfs   defaults        0 0

Create the proc directory under the root file system of the image file
with command:

	$ sudo mkdir -pv /mnt/centos/proc

Mount a proc file system to the `/mnt/centos/proc` directory with
command:

	$ sudo mount -t proc none /mnt/centos/proc

Create the CentOS yum repository file `/etc/yum.repos.d/centos.repo`
with following content:

    [centos]
    name=centos
    #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
    #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
    baseurl=http://mirror.centos.org/centos-6/6.4/os/x86_64/
    gpgkey=http://mirror.centos.org/centos-6/6.4/os/x86_64/RPM-GPG-KEY-CentOS-6
    gpgcheck=1
    protect=1

Install the latest CentOS 6.3 operating system with command:

	$ sudo yum --disablerepo=* --enablerepo=centos  --installroot=/mnt/centos -y groupinstall Base

The `--disablerepo` option means to disable all the available
repositories and `--enablerepo` option enables only the CentOS
repository specified in the previous step.

This command will start the installation of CentOS 6.3 on the mounted
directory, which will take a while depending on the network speed and
host system hardware configurations.

When the installation completes, we can verify the installation with
command:

	$ ls -lh /mnt/centos/

The directory structure of the installed operating system should be the
same as the directory structure of a regularly installed Linux. For
example, the output will be similar to the following:

    total 108K
    dr-xr-xr-x.  2 root root 4.0K May  3 01:12 bin
    dr-xr-xr-x.  3 root root 4.0K May  3 01:13 boot
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 dev
    drwxr-xr-x. 73 root root 4.0K May  3 02:00 etc
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 home
    dr-xr-xr-x. 10 root root 4.0K May  3 01:12 lib
    dr-xr-xr-x.  9 root root  12K May  3 01:11 lib64
    drwx------.  2 root root  16K May  3 00:17 lost+found
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 media
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 mnt
    drwxr-xr-x.  3 root root 4.0K May  3 01:12 opt
    dr-xr-xr-x.  2 root root 4.0K Sep 23  2011 proc
    dr-xr-x---.  2 root root 4.0K Sep 23  2011 root
    dr-xr-xr-x.  2 root root  12K May  3 01:12 sbin
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 selinux
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 srv
    drwxr-xr-x.  2 root root 4.0K Sep 23  2011 sys
    drwxrwxrwt.  2 root root 4.0K May  3 01:13 tmp
    drwxr-xr-x. 13 root root 4.0K May  3 01:02 usr
    drwxr-xr-x. 19 root root 4.0K May  3 01:12 var

We can configure the system with the following steps:

Create the network adapter configuration file
[/mnt/centos/etc/sysconfig/network-scripts/ifcfg-eth0](/mnt/centos/etc/sysconfig/network-scripts/ifcfg-eth0)
with the following content:

    DEVICE=eth0
    BOOTPROTO=dhcp
    ONBOOT=yes
    TYPE=Ethernet
    USERCTL=yes
    PEERDNS=yes
    IPV6INIT=no

In this configuration, BOOTPROTO specifies to use DHCP IP address
assignment.

Enable networking by adding or changing the NETWORKING option in the
network configuration file `/mnt/centos/etc/sysconfig/network`:

	NETWORKING=yes

Add the following content into file `/mnt/centos/etc/fstab`:

    /dev/sda2  /mnt      ext3    defaults        0 0
    /dev/sda3  swap      swap    defaults        0 0

These two lines configure the mount points for the swap and root
partitions.

Configure to start necessary services with commands:

	$ sudo chroot /mnt/centos /bin/sh
	$ chkconfig --level 345 network on
	$ exit

Umount the image file with command:

	$ sudo umount /mnt/centos/proc
	$ sudo umount -d /mnt/centos

Copy the private key and X.509 certificate file into the instance with
command:

	$ scp -i shumin.guo ~/.ec2/pk-*pem ~/.ec2/cert-*pem root@ec2-58-214-29-104.compute-1.amazonaws.com:~/.ec2/

Login to the instance with command:

	$ ssh -i ~/.ec2/shumin.guo.pem root@ec2-58-214-29-104.compute-1.amazonaws.com

In this command, `ec2-58-214-29-104.compute-1.amazonaws.com` is the
public domain name of the instance.

Configure password-less login with commands:

	$ ssh-keygen
	$ mkdir -v /mnt/centos/root/.ssh
	$ sudo cp ~/.ssh/id_* /mnt/centos/root/.ssh

When you are prompted to enter the paraphrase, leave it empty by
pressing the ’**Enter**’ key.

Copy the public key to the `authorized_keys` file with command:

	$ cat /mnt/centos/root/.ssh/id_rsa.pub >> /mnt/centos/root/.ssh/authorized_keys

Copy the local Java installation to the image folder with command:

	$ sudo cp -r /usr/java /mnt/centos/usr

Download the latest Hadoop distribution from official mirror
<http://www.apache.org/dyn/closer.cgi/hadoop/common/>.

Use the following command to decompress the Hadoop package and create
the symbolic link:

	$ sudo tar xvf hadoop-*.tar.gz -C /mnt/centos/usr/local/
	$ sudo ln -s /mnt/centos/usr/local/hadoop-* /mnt/centos/usr/local/hadoop

Add the following environment variables to file *.bashrc*:

    export JAVA_HOME=/usr/java/latest
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin

Add the following content into file `$HADOOP_HOME/conf/core-site.xml`:

```xml
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:54310</value>
  </property>
<configuration>
```

Add the following content into file `$HADOOP_HOME/conf/mapred-site.xml`:

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
    <value>/hadoop/tmp/hadoop-${user.name}/</value>
  </property>

</configuration>
```

Add the following content into file `$HAOOP_HOME/conf/hdfs-site.xml`:

```xml
<configuration>

  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:54311</value>
  </property>

  <property>
    <name>mapred.system.dir</name>
    <value>/hadoop/mapred</value>
  </property>

</configuration>
```

Download and install the other ecosystem components with recipes in
Chapter 3.

We can use the following recipe to bundle, upload and register an AMI:\
Bundle the loopback image file with command:

	$ ec2-bundle-image -i centos.img -k .ec2/pk-*.pem -c .ec2/cert-*.pem -u 123412341234

Option *-i* specifies the image file name, *-k* specifies the private
key file, *-c* specifies the certificate file and -u specifies the user
Account Number which is a 12 digit numeric string.

We can get the Account number from the web user interface through the
following URL: <https://portal.aws.amazon.com/gp/aws/manageYourAccount>?
The account number is on the upper left of the window as shown in Figure
[fig:aws.account.number].

![AWS Account Number<span
data-label="fig:aws.account.number"></span>](figs/5163os_08_16.png)

The command will ask for the architecture of the image, then it will
bundle the image with user’s security credentials and split the bundled
image file into smaller files.

The output message will be similar to the following:

    Please specify a value for arch [x86_64]:
    Bundling image file...
    Splitting /tmp/centos.img.tar.gz.enc...
    Created centos.img.part.00
    Created centos.img.part.01
    Created centos.img.part.02
    Created centos.img.part.03
    Created centos.img.part.04
    Created centos.img.part.05
    Created centos.img.part.06
    Created centos.img.part.07
    Created centos.img.part.08
    Created centos.img.part.09
    Created centos.img.part.10
    ...
    Generating digests for each part...
    Digests generated.
    Creating bundle manifest...
    ec2-bundle-image complete.

Create a bucket from the S3 web interface as shown in Figure
[fig:aws.s3.bucket.create].

![AWS Creating S3 Bucket<span
data-label="fig:aws.s3.bucket.create"></span>](figs/5163os_08_17.png)

Type in the bucket name and select the region based on your location as
shown in Figure [fig:aws.s3.bucket.region].

![Choose Region for the Bucket<span
data-label="fig:aws.s3.bucket.region"></span>](figs/5163os_08_18.png)

Click the ’Create’ button, the bucket will be successfully created as
shown in Figure [fig:aws.s3.buckets].

![AWS S3 Buckets<span
data-label="fig:aws.s3.buckets"></span>](figs/5163os_08_19.png)

Upload the bundled file into S3 with command:

	$ ec2-upload-bundle -b packt-bucket -m /tmp/centos.img.manifest.xml -a AKIAJ7GAQT52MZKJA4WQ -s QDHHZ0/Mj5pDYFWKpqEzXhwjqM1UB+cqjGQQ6l3S

This command will upload the bundled image parts to the specified bucket
(packt-bucket in this case), which is specified with the *-b* option.
Option *-m* specifies the location of the manifest file, option *-a*
specifies the access key and *-s* specifies the secret key. Note that
for security purposes, the manifest file will be encrypted with the
public key before being uploaded.

The command will give output similar to the following:

    Uploading bundled image parts to the S3 bucket packt-bucket ...
    Uploaded centos.img.part.00
    Uploaded centos.img.part.01
    Uploaded centos.img.part.02
    Uploaded centos.img.part.03
    Uploaded centos.img.part.04
    Uploaded centos.img.part.05
    Uploaded centos.img.part.06
    Uploaded centos.img.part.07
    Uploaded centos.img.part.08
    Uploaded centos.img.part.09
    Uploaded centos.img.part.10
    ...
    Uploading manifest ...
    Uploaded manifest.
    Bundle upload completed.

When the upload completes, we can check the content of the bucket by
clicking on the bucket name. The bucket should now contain all the image
parts as well as the manifest file as shown in Figure
[fig:aws.bucket.content].

![Files of a S3 Bucket<span
data-label="fig:aws.bucket.content"></span>](figs/5163os_08_20.png)

Register the AMI with command:

	$ ec2-register packt-bucket/image.manifest.xml -n packt-centos-6.4-x64 -O AKIAJ7GAQT52MZKJA4WQ -W QDHHZ0/Mj5pDYFWKpqEzXhwjqM1UB+cqjGQQ6l3S

The command will give us an ID for the newly registered AMI similar to
the following: `IMAGE   ami-9f422ff6`

Warning!\
The AMI registration step is required in order for EC2 to find the AMI
and run instances with it. Note that once changes are made on the image
part files stored on S3, re-registration is required in order for the
changes to take effect.

We can check the details of the new AMI with command:

	$ ec2-describe-images ami-9f422ff6
	IMAGE   ami-9f422ff6    869345430376/packt-centos-6.4-x64       869345430376    available       private             x86_64  machine                         instance-store  paravirtual     xen

The meaning of each column is:

- The IMAGE identifier
- The ID of the image
- The source of the image
- The ID of the image owner
- The status of the image
- The visibility of the image (public or private)
- The product codes, if any, that are attached to the instance
- The architecture of the image (i386 or `x86_64`)
- The image type (machine, kernel, or ramdisk)
- The ID of the kernel associated with the image (machine images only)
- The ID of the RAM disk associated with the image (machine images only)
- The platform of the image
- The type of root device (ebs or instance-store)
- The virtualization type (paravirtual or hvm)
- The Hypervisor type (xen or kvm)

Once the registration completes, we can start an instance with the new
AMI with command:

	$ ec2-run-instances ami-9f422ff6 -n 1 -k shumin.guo

The command specifies to run instance with our new AMI, option *-n*
specifies the number of instances to start and option *-k* specifies the
key pair to use for logging in to these instances.

The output will be similar to the following:

    RESERVATION     r-ca8919aa      869345430376    default
    INSTANCE        i-0020e06c      ami-9f422ff6    pending shumin.guo      0               m1.small        2013-05-03T08:22:09+0000  us-east-1a                              monitoring-disabled        instance-store     paravirtual     xen             sg-7bb47b12     default false

The first line of the output is the reservation information, and the
meanings of the columns are:

- The RESERVATION identifier
- The ID of the reservation
- The AWS account ID of the instance owner
- The name of each security group the instance is in The second line
    shows the instance information, and the meanings of columns are:
- The INSTANCE identifier
- The ID of the instance
- The AMI ID of the image on which the instance is based
- The public DNS name associated with the instance. This is only
    present for instances in the running state.
- The private DNS name associated with the instance. This is only
    present for instances in the running state.
- The state of the instance
- The key name. If a key was associated with the instance at launch,
    its name will appear.
- The AMI launch index
- The product codes associated with the instance
- The instance type
- The instance launch time
- The Availability Zone
- The ID of the kernel
- The ID of the RAM disk
- The platform (windows or empty)
- The monitoring state
- The public IP address
- The private IP address
- `EC2-VPC` The ID of the VPC
- `EC2-VPC` The subnet ID
- The type of root device (ebs or instance-store)
- The instance lifecycle
- The Spot Instance request ID
- The instance license
- The placement group the cluster instance is in
- The virtualization type (paravirtual or hvm)
- The hypervisor type (xen or kvm)
- The client token
- The ID of each security group the instance is in
- The tenancy of the instance (default or dedicated)
- Whether or not the instance is EBS optimized (true or false)
- The Amazon Resource Name (ARN) of the IAM role

The output message shows that the ID of the instance is **i-0020e06c**.

After waiting for a while, we can check the status of the instance with
command:

    $ ec2-describe-instances i-0020e06c
    RESERVATION     r-ca8919aa      869345430376    default
    INSTANCE        i-0020e06c      ami-9f422ff6    ec2-54-224-240-54.compute-1.amazonaws.com       ip-10-34-102-91.ec2.internal      running shumin.guo      0               m1.small        2013-05-03T08:22:09+0000        us-east-1a               monitoring-disabled      54.224.240.54   10.34.102.91                    instance-store                                  paravirtual       xen             sg-7bb47b12     default false

The instance status tells us that it is in running state.

Alternatively, we can check the status of the instance from the web UI.
For example, we can get the status of the instance similar to Figure
[fig:aws.ec2.instances].

![AWS EC2 Instances<span
data-label="fig:aws.ec2.instances"></span>](figs/5163os_08_21.png)

Login to the instance with command:

	$ ssh -i ~/.ec2/shumin.guo.pem root@ec2-54-224-240-54.compute-1.amazonaws.com

In this command, *-i* specifies the key pair to use for login and
`ec2-54-224-240-54.compute-1.amazonaws.com` is the public domain name of
the EC2 instance.

### There’s more... 

As we mentioned previously, there are other methods to create AMIs. One
method is to create an AMI from existing AMI. Another method is to
create an EBS backed AMI.

#### Creating AMI from an existing AMI 

This recipe shows steps how to create an instance store backed AMI from
an existing AMI. We assume that you have registered with AWS and have
successfully configured the security credentials in a local machine. We
also assume that you have downloaded the key pair and saved it to the
proper location. In this recipe, we assume the private keys,
certificates and key pairs all located in the `~/.ec2` folder.

Start an instance from an existing AMI. For example, we can start an
instance with the new AMI created in the previous recipe with command:

	$ ec2-run-instances ami-9f422ff6 -n 1 -k shumin.guo

This command will start up one instance from the new AMI. Key pair
shumin.guo is used to login to the instance remotely.

Copy the private key and X.509 certificate into the instance with
command:

	$ scp -i shumin.guo ~/.ec2/pk-*pem ~/.ec2/cert-*pem root@ec2-58-214-29-104.compute-1.amazonaws.com:~/.ec2/

Login to the instance with command:

	$ ssh -i ~/.ec2/shumin.guo.pem root@ec2-58-214-29-104.compute-1.amazonaws.com

Configure password-less login with commands:

	$ ssh-keygen

You will be prompted to enter the paraphrase, leave it empty by pressing
the ’Enter’ key.

	$ ssh-copy-id localhost

Download and install Java with recipe installing Java and other tools of
Chapter 2.

Download the latest Hadoop distribution from
<http://www.apache.org/dyn/closer.cgi/hadoop/common/>.

Use the following command to decompress the Hadoop package and create
the symbolic link:

	$ tar xvf hadoop-*.tar.gz -C /usr/local/
	$ ln -s /usr/local/hadoop-* /usr/local/hadoop

Add the following environment variables to file .bashrc:

	$ export JAVA_HOME=/usr/java/latest
	$ export HADOOP_HOME=/usr/local/hadoop
	$ export PATH=$PATH:$HADOOP_HOME/bin

Add the following content into file `$HADOOP_HOME/conf/core-site.xml`:

```xml
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:54310</value>
  </property>
<configuration>
```

Add the following content into file `$HADOOP_HOME/conf/mapred-site.xml`:

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
    <value>/hadoop/tmp/hadoop-${user.name}/</value>
  </property>

</configuration>
```

Add the following content into file `$HAOOP_HOME/conf/hdfs-site.xml`:

```xml
<configuration>

  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:54311</value>
  </property>

  <property>
    <name>mapred.system.dir</name>
    <value>/hadoop/mapred</value>
  </property>

</configuration>
```

Download and install all other Hadoop ecosystem components by using the
recipes in Chapter 3.

Install the AMI tools package with command:

	$ rpm -ivh http://s3.amazonaws.com/ec2-downloads/ec2-ami-tools.noarch.rpm

Disable SELinux with command:

	$ setenforce 0

Disable iptables with command:

	$ iptables -F
	$ chkconfig iptables off

Bundle the image with command:

	$ ec2-bundle-vol -e ~/.ec2 -k pk-*pem -c cert-*.pem -u 123412341234

In this command, *-k* specifies the name of the file that contains the
private key, *-c* specifies the file that contains the X.509
certificate, *-u* specifies the 12 to 15 digit count ID without dashes
of the current user and *-e* specifies the **location/directory** that
contains the private key file and certificate file.

Upload the bundled AMI to S3 with command:

	$ ec2-upload-bundle -b packt-bucket -m /tmp/image.manifest.xml\index{image.manifest.xml} -a AKIAJ7GAQT52MZKJA4WQ -p QDHHZ0/Mj5pDYFWKpqEzXhwjqM1UB+cqjGQQ6l3S

In this command, *-b* specifies the name of the bucket on S3, *-m*
specifies the location of the manifest file, *-a* option specifies the
access key string and *-p* option specifies the secret key string.

Register the AMI with command:

	$ ec2-register packt-bucket/image.manifest.xml -n centos-hadoop-1.0 -O AKIAJ7GAQT52MZKJA4WQ -W QDHHZ0/Mj5pDYFWKpqEzXhwjqM1UB+cqjGQQ6l3S

In this command, the first parameter specifies the location of the
manifest file in the S3 bucket, the *-n* option specifies the name of
the AMI, *-O* specifies the access key string and -W specifies the
secret key string.

#### Creating an EBS backed AMI 

Creating an EBS backed AMI is straightforward from the web management
console. This recipe will guide you through the steps to create an EBS
backed AMI from a running EC2 instance. For more information, you can
visit Amazon official document at:
<http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html#process_creating-an-ami-ebs>.

Use the following recipe to create an EBS backed AMI:

Go to <https://console.aws.amazon.com/ec2/v2/home#Images:>, filter the
AMIs with conditions similar to Figure [fig:aws.ec2.ami.filter].

![Filtering EC2 AMIs<span
data-label="fig:aws.ec2.ami.filter"></span>](figs/5163os_08_29.png)

Start an instance by right clicking on one of the AMIs and then click
’Launch’ as shown in Figure [fig:aws.launch.instance].

![Launching EC2 Instance<span
data-label="fig:aws.launch.instance"></span>](figs/5163os_08_30.png)

When the instance is running, login to the instances, make changes
according to your requirements. Then, from the web management console,
right click on the running instance and then select ’Create Image (EBS
AMI)’ as shown in Figure [fig:aws.create.ebsami].

![Create EBS AMI from Running Instance<span
data-label="fig:aws.create.ebsami"></span>](figs/5163os_08_31.png)

Go to the AMIs tab of the AWS web console, and select ’Owned By Me’, we
will see that the EBS AMI is being created as shown in Figure
[fig:aws.owned.ami].

![Owned AMI<span
data-label="fig:aws.owned.ami"></span>](figs/5163os_08_33.png)

EC2 creates a snapshot for the new AMI. Similar to the image part files
stored in S3, the snapshot stores the physical image of the EBS backed
AMI.

### See also 

- Installing HBase in Chapter [chap:3], Configuring a Hadoop cluster
- Installing Hive in Chapter [chap:3], Configuring a Hadoop cluster
- Installing Pig in Chapter [chap:3], Configuring a Hadoop cluster
- Installing ZooKeeper in Chapter [chap:3], Configuring a Hadoop
    cluster
- Installing Mahout in Chapter [chap:3], Configuring a Hadoop cluster
- <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Tutorial_CreateImage.html>
- <http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/command-reference.html>

## Using S3 to host data

Simple Storage Service (S3) provides a convenient online data store.
Users can use it to store and retrieve data. More information about S3
can be obtained from <http://aws.amazon.com/s3/>.

This recipe will outline steps to configure S3 as the distributed data
storage system for MapReduce.

### Getting ready 

Before getting started, we assume that you have successfully registered
with AWS and the client machine has been successfully configured to
access the AWS.

### How to do it... 

Use the following recipe to configure S3 for data storage:

Stop the Hadoop cluster with commands:

	$ stop-all.sh

Open file `$HADOOP_HOME/conf/core-site.xml` and add the following
contents into the file:

```xml
<property>
  <name>fs.default.name</name>
  <!-- value>master:54310</value-->
  <value>s3n://packt-bucket</value>
</property>

<property>
  <name>fs.s3n.awsAccessKeyId</name>
  <value>AKIAJ7GAQT52MZKJA4WQ</value>
</property>

<property>
  <name>fs.s3n.awsSecretAccessKey</name>
  <value>QDHHZ0/Mj5pDYFWKpqEzXhwjqM1UB+cqjGQQ6l3S</value>
</property>
```

The first property configures Hadoop to use S3 as distributed
filesystem.

Start the cluster with commands:

	$ start-mapred.sh

As we are using S3 instead of HDFS as the data storage filesystem, there
is no need to start the HDFS cluster any more.

Check the configuration with S3 with command:

    hadoop fs -ls /
    Found 49 items
    -rwxrwxrwx   1       8560 2013-05-03 03:18 /centos.img.manifest.xml
    -rwxrwxrwx   1   10485760 2013-05-03 03:17 /centos.img.part.00
    -rwxrwxrwx   1   10485760 2013-05-03 03:17 /centos.img.part.01
    -rwxrwxrwx   1   10485760 2013-05-03 03:17 /centos.img.part.02
    ...

## Configuring a Hadoop cluster with the new AMI

Starting a Hadoop cluster with the new AMI is simple and
straightforward. This recipe will list steps to start up a Hadoop
cluster with the new AMI.

### Getting ready 

Before getting started, we assume that you have registered with AWS and
have successfully created a new AMI with Hadoop properly configured.

### How to do it... 

Use the following recipe to configure a Hadoop cluster with EC2:

Run a number of instances either from the command line or from the web
interface.

After the instances are all in running state, run the following command
to get the internal hostname of these instances:

	$ ec2-describe-instances | grep running | egrep -o 'ip.*?internal' | sed -e 's/.ec2.internal//g' > nodes.txt

File nodes.txt will have contents similar to the following:

    ip-10-190-81-210
    ip-10-137-11-196
    ip-10-151-11-161
    ip-10-137-48-163
    ip-10-143-160-5
    ip-10-142-132-17

We are assuming to use the highlighted node as the master node and the
public domain name of this node is:
`ec2-174-129-127-90.compute-1.amazonaws.com`.

Copy file nodes.txt to the master node with command from the local
machine:

	$ scp -i ~/.ec2/shumin.guo.pem nodes.txt ec2-user@ec2-174-129-127-90.compute-1.amazonaws.com:~/

Login to the new instance with command:

	$ ssh -i ~/.ec2/shumin.guo.pem root@ec2-user@ec2-174-129-127-90.compute-1.amazonaws.com

Use the following commands to create a hosts file:

	$ cp nodes.txt nodes.ip.txt
	$ cp nodes.txt slaves
	$ sed -i 's/ip-//g' nodes.ip.txt
	$ sed -i 's/-/./g' nodes.ip.txt
	$ sed -i '1d' slaves
	$ paste nodes.ip.txt nodes.txt > hosts

The hosts file should have the following content:

    10.190.81.210   ip-10-190-81-210
    10.137.11.196   ip-10-137-11-196
    10.151.11.161   ip-10-151-11-161
    10.137.48.163   ip-10-137-48-163
    10.143.160.5    ip-10-143-160-5
    10.142.132.17   ip-10-142-132-17

Move the hosts file to `/etc/hosts` with command:

	for hosts in `cat nodes.txt`; do
		echo 'Configuring /etc/hosts file for host : ' $host
		scp hosts $hosts:/etc/hosts
	done

Configure the slaves file with command:

	$ cp slaves $HADOOP_HOME/conf/slaves

Open file `$HADOOP_HOME/conf/core-site.xml` with a text editor and
change the `fs.default.name` similar to the following:

```xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://ip-10-190-81-210:54310</value>
</property>
```

Open file `$HADOOP_HOME/conf/mapred-site.xml` with a text editor and
change the *mapred.job.tracker* property similar to the following:

```xml
<property>
  <name>mapred.job.tracker</name>
  <value>ip-10-190-81-210:54311</value>
</property>
```

Copy the configurations to all the slave nodes with command:

	for host in `cat $HADOOP_HOME/conf/slaves`; do
		echo 'Copying Hadoop conifugration files to host: ' $host
		scp $HADOOP_HOME/conf/{core,mapred}-site.xml $host:$HADOOP_HOME/conf
	done

Start the cluster with command:

	$ start-dfs.sh
	$ start-mapred.sh

When the cluster is running, we can start to submit jobs to the cluster
from the master node.

### There’s more... 

An alternative method of running a MapReduce with the amazon cloud is to
use Amazon Elastic MapReduce (EMR). Amazon EMR provides an elastic
parallel computing platform based on EC2 and S3. Data and results can be
stored on S3. EMR computing is handy for ad-hoc data processing
requirements.

## Data processing with Amazon Elastic MapReduce

Before using EMR, we assume that you have registered with AWS. A S3
bucket (for example packt-bucket) has been created using the S3 web
console. In the following recipe, we will use wordcount job, which is
shipped with the Hadoop examples jar package, as an example.

We can use the following recipe to use EMR for data processing:

Create the input directory (with name input) and Java library directory
(with name jars) under the bucket from the S3 web management console.

Upload data into the input folder from the web console as shown in
Figure [fig:aws.s3.upload].

![Uploading data into S3<span
data-label="fig:aws.s3.upload"></span>](figs/5163os_08_22.png)

Upload required jar file (hadoop-examples-\*.jar in this example) into
the jars directory.

If you have configured S3 using the command in the previous recipe, you
can also use the following commands to finish the above steps:

	$ hadoop fs -mkdir /jars /input
	$ hadoop fs -put $HADOOP_HOME/hadoop-examples-*.jar /jars
	$ hadoop fs -put words.txt /input

*words.txt* contains the input data for the wordcount job.

Open URL: <https://console.aws.amazon.com/elasticmapreduce>.

Click the ’*Create New Job Flow*’ button as shown in Figure
[fig:aws.elasticmapred.createflow].

![Create New Elastic MapReduce Job Flow<span
data-label="fig:aws.elasticmapred.createflow"></span>](figs/5163os_08_23.png)

Next, enter the ’Job Flow Name’, select the ’Hadoop Version’ and select
the job flow type as shown in Figure [fig:aws.elasticmapred.config].

![Configure Elastic MapReduce Job Flow<span
data-label="fig:aws.elasticmapred.config"></span>](figs/5163os_08_24.png)

To test a simple job flow, you can choose ’Run a sample application’
instead.

Click ’Continue’ button at the bottom, the next window asks for the
location of the jar file and the parameters for running the Hadoop
MapReduce job as shown in Figure [fig:aws.elasticmapred.specify.param].

![Specify Job Flow Parameters<span
data-label="fig:aws.elasticmapred.specify.param"></span>](figs/5163os_08_01.png)

In this step, we need to specify the location of the jar file and the
arguments to run a job. The specifications should be similar to option
specifications from the command line with the only difference that all
the files should be specified using the S3 scheme.

Click ’Continue’, we need to configure EC2 instances. By default, there
will be one m1.small instance as the master node and two m1.small
instances as the slave nodes. You can configure the instance type and
the number of instances based on the job properties (for example, big or
small input data size, data intensive or computation intensive etc.).
This step is shown in Figure [fig:aws.elasticmapred.config.instance].

![Configure EC2 Instances<span
data-label="fig:aws.elasticmapred.config.instance"></span>](figs/5163os_08_26.png)

Click the ’Continue’ button and we will go to the ’Advanced options’
window. This window asks for instance boot options such as security key
pairs etc. In this step, we can choose the key pair and use all others
as default and click ’Continue’.

We will go to the ’Bootstrap actions’ window. We can simply use default
in this step and click ’Continue’.

The ’Review’ window shows the options we have configured, if there is no
problem, we can click the ’Create Job Flow’ button to create an EMR job
flow. This step is shown in Figure [fig:aws.elasticmapred.flow.review].

![Review Job Flow Configuration<span
data-label="fig:aws.elasticmapred.flow.review"></span>](figs/5163os_08_27.png)

The job flow will be started and we can check the output when it
completes. We can get its status from the web console as shown in Figure
[fig:aws.elasticmapred.flow.status].

![Status of an Elastic MapReduce Job Flow<span
data-label="fig:aws.elasticmapred.flow.status"></span>](figs/5163os_08_28.png)

### See also 

- Chapter [chap:3], Configuring a Hadoop cluster

[^1]: HDD stands for Hard Disk Drive and SSD stands for Solid State
    Drive

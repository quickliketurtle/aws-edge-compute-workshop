# Workshop - Module 3

## Greengrass installation

<a name="1.1"></a>
## 1.1 Preparing for the Greengrass Installation

In this section we are preparing the *EC2* instance to be able to run *Greengrass* on it.

1. We need to create a user account `ggc_user` which will be the acocunt used to run *Greengrass* in.

	<details>
		<summary>Command: `sudo adduser --system ggc_user`</summary>
	
		Output:
		
		Adding system user `ggc_user' (UID 112) ...
		Adding new user `ggc_user' (UID 112) with group `nogroup' ...
		Creating home directory `/home/ggc_user' ...
	</details>

1. We will also create a group `ggc_group` used for *Greengrass*.

	<details>
		<summary>Command: `sudo addgroup --system ggc_group`</summary>
	
		Output:
		
		Adding group `ggc_group' (GID 116) ...
		Done.
	</details>

1. We now need to change into a fodler on the file system in order to adjust the configuration of the instance to support cgroup with the necessary filesystem protection used by *Greengrass*.

	Command: `cd /etc/sysctl.d`
	
1. To list the contents of the folder we changed into use the following command.

	<details>
		<summary>Command: `ls`</summary>
	
		Output:
		
		10-console-messages.conf   10-lxd-inotify.conf       10-zeropage.conf
		10-ipv6-privacy.conf       10-magic-sysrq.conf       99-cloudimg-ipv6.conf
		10-kernel-hardening.conf   10-network-security.conf  99-sysctl.conf
		10-link-restrictions.conf  10-ptrace.conf            README
	</details>

1. We now edit (if it exists) or create a file to store the configuration parameters for the filesystem.

	Command: `sudo nano 00-defaults.conf`

	Add to the file:

	```
	fs.protected_hardlinks = 1
	fs.protected_symlinks = 1
	```

	`CTRL+O` to save, then `CTRL+X` to exit the editor
	
1. We can now reboot the EC2 instance to pick-up the configuration changes.

	<details>
		<summary>Command: `sudo reboot`</summary>
	
		Output:
		
		Connection to ec2-34-222-222-74.us-west-2.compute.amazonaws.com closed by remote host.
		Connection to ec2-34-222-222-74.us-west-2.compute.amazonaws.com closed.
	</details>
	
	![13_1](../images/13_1.png)
	
	**NB**: your SSH session will be disconnnected automatically
	
1. Once the reboot is finished connect back to the EC2 instance using SSH.

	<details>
		<summary>Command: `ssh -i "~/Downloads/SBE_Workshop.pem" ubuntu@ec2-34-222-222-74.us-west-2.compute.amazonaws.com`</summary>
	
		Output:
		
		Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-1067-aws x86_64)
		
		 * Documentation:  https://help.ubuntu.com
		 * Management:     https://landscape.canonical.com
		 * Support:        https://ubuntu.com/advantage
	
	  	Get cloud support with Ubuntu Advantage Cloud Guest:
	​    http://www.ubuntu.com/business/services/cloud
	
		6 packages can be updated.
		6 updates are security updates.
		
		New release '18.04.1 LTS' available.
		Run 'do-release-upgrade' to upgrade to it.
		
		Last login: Fri Oct 26 14:44:13 2018 from 196.32.238.241
	</details>

1. Using the following command we can check whether the filesystem protection parameters have been successfully picked up during the reboot by the operating system.

	<details>
		<summary>Command: `sudo sysctl -a | grep fs.protected`</summary>
	
		Output:
		
		fs.protected_hardlinks = 1
		fs.protected_symlinks = 1
		sysctl: reading key "net.ipv6.conf.all.stable_secret"
		sysctl: reading key "net.ipv6.conf.default.stable_secret"
		sysctl: reading key "net.ipv6.conf.ens3.stable_secret"
		sysctl: reading key "net.ipv6.conf.lo.stable_secret"
	</details>

1. Now we need to download a shell script file in order to add support for cgroup-mount on the EC2 instance.

	<details>
		<summary>Command: `curl https://raw.githubusercontent.com/tianon/cgroupfs-mount/951c38ee8d802330454bdede20d85ec1c0f8d312/cgroupfs-mount > cgroupfs-mount.sh`</summary>

		Output:
		
		  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
		                                 Dload  Upload   Total   Spent    Left  Speed
		100  1275  100  1275    0     0   7271      0 --:--:-- --:--:-- --:--:--  7285
	</details>

1. In order to execute the shell script and install the support we need to make the script executable by adjusting its filesystem permissions.

	Command: `chmod +x cgroupfs-mount.sh`
	
1. Then we can execute the shell script, make sure you use super user permissions for the execution.

	Command: `sudo bash ./cgroupfs-mount.sh`

1. To be able to execute local Lambda function we need to ascertain that the necessary programming runtimes are installed and availble on the system. In thcase we install Python 2.7 and the Python package manager PIP. If you want tot used Javascript or Java for your local Lambda function you need to make sure you install those environments on the EC2 instance as well and configure them correctly.
	
	<details>
		<summary>Command: `sudo apt install python2.7 python-pip -y`</summary>
	
		Output:
		
		Reading package lists... Done
		Building dependency tree
		Reading state information... Done
		The following additional packages will be installed:
		  binutils build-essential cpp cpp-5 dpkg-dev fakeroot g++ g++-5 gcc gcc-5
		  libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl
		  libasan2 libatomic1 libc-dev-bin libc6-dev libcc1-0 libcilkrts5 libdpkg-perl
		  libexpat1-dev libfakeroot libfile-fcntllock-perl libgcc-5-dev libgomp1
		  libisl15 libitm1 liblsan0 libmpc3 libmpx0 libpython-all-dev libpython-dev
		  libpython-stdlib libpython2.7 libpython2.7-dev libpython2.7-minimal
		  libpython2.7-stdlib libquadmath0 libstdc++-5-dev libtsan0 libubsan0
		  linux-libc-dev make manpages-dev python python-all python-all-dev python-dev
		  python-minimal python-pip-whl python-pkg-resources python-setuptools
		  python-wheel python2.7-dev python2.7-minimal
		Suggested packages:
		  binutils-doc cpp-doc gcc-5-locales debian-keyring g++-multilib
		  g++-5-multilib gcc-5-doc libstdc++6-5-dbg gcc-multilib autoconf automake
		  libtool flex bison gdb gcc-doc gcc-5-multilib libgcc1-dbg libgomp1-dbg
		  libitm1-dbg libatomic1-dbg libasan2-dbg liblsan0-dbg libtsan0-dbg
		  libubsan0-dbg libcilkrts5-dbg libmpx0-dbg libquadmath0-dbg glibc-doc
		  libstdc++-5-doc make-doc python-doc python-tk python-setuptools-doc
		  python2.7-doc binfmt-support
		The following NEW packages will be installed:
		  binutils build-essential cpp cpp-5 dpkg-dev fakeroot g++ g++-5 gcc gcc-5
		  libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl
		  libasan2 libatomic1 libc-dev-bin libc6-dev libcc1-0 libcilkrts5 libdpkg-perl
		  libexpat1-dev libfakeroot libfile-fcntllock-perl libgcc-5-dev libgomp1
		  libisl15 libitm1 liblsan0 libmpc3 libmpx0 libpython-all-dev libpython-dev
		  libpython-stdlib libpython2.7 libpython2.7-dev libpython2.7-minimal
		  libpython2.7-stdlib libquadmath0 libstdc++-5-dev libtsan0 libubsan0
		  linux-libc-dev make manpages-dev python python-all python-all-dev python-dev
		  python-minimal python-pip python-pip-whl python-pkg-resources
		  python-setuptools python-wheel python2.7 python2.7-dev python2.7-minimal
		0 upgraded, 57 newly installed, 0 to remove and 3 not upgraded.
		Need to get 72.9 MB of archives.
		After this operation, 209 MB of additional disk space will be used.
		Get:1 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython2.7-minimal amd64 2.7.12-1ubuntu0~16.04.3 [340 kB]
		Get:2 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 python2.7-minimal amd64 2.7.12-1ubuntu0~16.04.3 [1,261 kB]
		Get:3 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 python-minimal amd64 2.7.12-1~16.04 [28.1 kB]
		Get:4 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython2.7-stdlib amd64 2.7.12-1ubuntu0~16.04.3 [1,880 kB]
		Get:5 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 python2.7 amd64 2.7.12-1ubuntu0~16.04.3 [224 kB]
		Get:6 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython-stdlib amd64 2.7.12-1~16.04 [7,768 B]
		Get:7 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 python amd64 2.7.12-1~16.04 [137 kB]
		Get:8 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libmpc3 amd64 1.0.3-1 [39.7 kB]
		Get:9 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 binutils amd64 2.26.1-1ubuntu1~16.04.7 [2,309 kB]
		Get:10 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libc-dev-bin amd64 2.23-0ubuntu10 [68.7 kB]
		Get:11 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 linux-libc-dev amd64 4.4.0-138.164 [859 kB]
		Get:12 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libc6-dev amd64 2.23-0ubuntu10 [2,079 kB]
		Get:13 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libisl15 amd64 0.16.1-1 [524 kB]
		Get:14 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 cpp-5 amd64 5.4.0-6ubuntu1~16.04.10 [7,671 kB]
		Get:15 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 cpp amd64 4:5.3.1-1ubuntu1 [27.7 kB]
		Get:16 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libcc1-0 amd64 5.4.0-6ubuntu1~16.04.10 [38.8 kB]
		Get:17 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libgomp1 amd64 5.4.0-6ubuntu1~16.04.10 [55.1 kB]
		Get:18 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libitm1 amd64 5.4.0-6ubuntu1~16.04.10 [27.4 kB]
		Get:19 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libatomic1 amd64 5.4.0-6ubuntu1~16.04.10 [8,888 B]
		Get:20 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libasan2 amd64 5.4.0-6ubuntu1~16.04.10 [264 kB]
		Get:21 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 liblsan0 amd64 5.4.0-6ubuntu1~16.04.10 [105 kB]
		Get:22 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libtsan0 amd64 5.4.0-6ubuntu1~16.04.10 [244 kB]
		Get:23 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libubsan0 amd64 5.4.0-6ubuntu1~16.04.10 [95.3 kB]
		Get:24 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libcilkrts5 amd64 5.4.0-6ubuntu1~16.04.10 [40.1 kB]
		Get:25 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libmpx0 amd64 5.4.0-6ubuntu1~16.04.10 [9,764 B]
		Get:26 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libquadmath0 amd64 5.4.0-6ubuntu1~16.04.10 [131 kB]
		Get:27 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libgcc-5-dev amd64 5.4.0-6ubuntu1~16.04.10 [2,228 kB]
		Get:28 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 gcc-5 amd64 5.4.0-6ubuntu1~16.04.10 [8,426 kB]
		Get:29 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 gcc amd64 4:5.3.1-1ubuntu1 [5,244 B]
		Get:30 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libstdc++-5-dev amd64 5.4.0-6ubuntu1~16.04.10 [1,426 kB]
		Get:31 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 g++-5 amd64 5.4.0-6ubuntu1~16.04.10 [8,319 kB]
		Get:32 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 g++ amd64 4:5.3.1-1ubuntu1 [1,504 B]
		Get:33 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 make amd64 4.1-6 [151 kB]
		Get:34 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libdpkg-perl all 1.18.4ubuntu1.4 [195 kB]
		Get:35 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 dpkg-dev all 1.18.4ubuntu1.4 [584 kB]
		Get:36 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 build-essential amd64 12.1ubuntu2 [4,758 B]
		Get:37 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libfakeroot amd64 1.20.2-1ubuntu1 [25.5 kB]
		Get:38 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 fakeroot amd64 1.20.2-1ubuntu1 [61.8 kB]
		Get:39 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libalgorithm-diff-perl all 1.19.03-1 [47.6 kB]
		Get:40 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libalgorithm-diff-xs-perl amd64 0.04-4build1 [11.0 kB]
		Get:41 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libalgorithm-merge-perl all 0.08-3 [12.0 kB]
		Get:42 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libexpat1-dev amd64 2.1.0-7ubuntu0.16.04.3 [115 kB]
		Get:43 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial/main amd64 libfile-fcntllock-perl amd64 0.22-3 [32.0 kB]
		Get:44 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython2.7 amd64 2.7.12-1ubuntu0~16.04.3 [1,070 kB]
		Get:45 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython2.7-dev amd64 2.7.12-1ubuntu0~16.04.3 [27.8 MB]
		Get:46 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 libpython-dev amd64 2.7.12-1~16.04 [7,840 B]
		Get:47 http://us-west-2.ec2.archive.ubuntu.com/ubuntu xenial-updates/main amd64 update-alternatives: using /usr/bin/fakeroot-sysv to provide /usr/bin/fakeroot (fakeroot) in auto mode
		Setting up libalgorithm-diff-perl (1.19.03-1) ...
		Setting up libalgorithm-diff-xs-perl (0.04-4build1) ...
		Setting up libalgorithm-merge-perl (0.08-3) ...
		Setting up libexpat1-dev:amd64 (2.1.0-7ubuntu0.16.04.3) ...
		Setting up libfile-fcntllock-perl (0.22-3) ...
		Setting up libpython2.7:amd64 (2.7.12-1ubuntu0~16.04.3) ...
		Setting up libpython2.7-dev:amd64 (2.7.12-1ubuntu0~16.04.3) ...
		Setting up libpython-dev:amd64 (2.7.12-1~16.04) ...
		Setting up libpython-all-dev:amd64 (2.7.12-1~16.04) ...
		Setting up manpages-dev (4.04-2) ...
		Setting up python-all (2.7.12-1~16.04) ...
		Setting up python2.7-dev (2.7.12-1ubuntu0~16.04.3) ...
		Setting up python-dev (2.7.12-1~16.04) ...
		Setting up python-all-dev (2.7.12-1~16.04) ...
		Setting up python-pip-whl (8.1.1-2ubuntu0.4) ...
		Setting up python-pip (8.1.1-2ubuntu0.4) ...
		Setting up python-pkg-resources (20.7.0-1) ...
		Setting up python-setuptools (20.7.0-1) ...
		Setting up python-wheel (0.29.0-1) ...
		Processing triggers for libc-bin (2.23-0ubuntu10) ...
	</details>

1. We also install the version management tool `git` to be available locally.

	<details>
		<summary>Command: `sudo apt-get install git -y`</summary>
	
		Output:
		
		Reading package lists... Done
		Building dependency tree
		Reading state information... Done
		git is already the newest version (1:2.7.4-0ubuntu1.5).
		0 upgraded, 0 newly installed, 0 to remove and 3 not upgraded.
	</details>

1. Let's now clone the git repo that contains the AWS Greengrass samples.

	<details>
		<summary>Command: `git clone https://github.com/aws-samples/aws-greengrass-samples.git`</summary>

		Output:
		
		Cloning into 'aws-greengrass-samples'...
		remote: Enumerating objects: 141, done.
		remote: Total 141 (delta 0), reused 0 (delta 0), pack-reused 141
		Receiving objects: 100% (141/141), 91.08 KiB | 0 bytes/s, done.
		Resolving deltas: 100% (64/64), done.
		Checking connectivity... done.
	</details>

1. To be able to make use of the samples let's change into the cloned repo.

	Command: `cd aws-greengrass-samples/greengrass-dependency-checker-GGCv1.6.0/`

1. Run the dependency checker to ascertain that the EC2 instance is correctly configured for the installation of *Greengrass*. Here you can also see the available runtimes for the local execution of *Lambda* functions.

	<details>
   		<summary>Command: `sudo ./check_ggc_dependencies`</summary>

	   	Output:
	   	
	   	==========================Checking script dependencies==============================
	   	The device has all commands required for the script to run.
	   	
	   	========================Dependency check report for GGC v1.6=========================
	   	System configuration:
	   	Kernel architecture: x86_64
	   	Init process: /lib/systemd/systemd
	   	Kernel version: 4.4
	   	C library: Ubuntu GLIBC 2.23-0ubuntu10
	   	C library version: 2.23
	   	Directory /var/run: Present
	   	/dev/stdin: Found
	   	/dev/stdout: Found
	   	/dev/stderr: Found
	   	
	   	--------------------------------Kernel configuration--------------------------------
	   	Kernel config file: /boot/config-4.4.0-1067-aws
	   	
	   	Namespace configs:
	   	CONFIG_IPC_NS: Enabled
	   	CONFIG_UTS_NS: Enabled
	   	CONFIG_USER_NS: Enabled
	   	CONFIG_PID_NS: Enabled
	   	
	   	Cgroup configs:
	   	CONFIG_CGROUP_DEVICE: Enabled
	   	CONFIG_CGROUPS: Enabled
	   	CONFIG_MEMCG: Enabled
	   	
	   	Other required configs:
	   	CONFIG_POSIX_MQUEUE: Enabled
	   	CONFIG_OVERLAY_FS: Enabled
	   	CONFIG_HAVE_ARCH_SECCOMP_FILTER: Enabled
	   	CONFIG_SECCOMP_FILTER: Enabled
	   	CONFIG_KEYS: Enabled
	   	CONFIG_SECCOMP: Enabled
	   	
	   	------------------------------------Cgroups check-----------------------------------
	   	Cgroups mount directory: /sys/fs/cgroup
	   	
	   	Devices cgroup: Enabled and Mounted
	   	Memory cgroup: Enabled and Mounted
	   	
	   	----------------------------Commands and software packages--------------------------
	   	Python version: 2.7.12
	   	NodeJS 6.10: Not found
	   	Java 8: Not found
	   	OpenSSL version: 1.0.2
	   	wget: Present
	   	realpath: Present
	   	tar: Present
	   	readlink: Present
	   	basename: Present
	   	dirname: Present
	   	pidof: Present
	   	df: Present
	   	grep: Present
	   	umount: Present
	   	
	   	---------------------------------Platform security----------------------------------
	   	Hardlinks_protection: Enabled
	   	Symlinks protection: Enabled
	   	
	   	-----------------------------------User and group-----------------------------------
	   	ggc_user: Present
	   	ggc_group: Present
	   	
	   	------------------------------------Results-----------------------------------------
	   	Note:
	   	1. It looks like the kernel uses 'systemd' as the init process. Be sure to set the
	   	'useSystemd' field in the file 'config.json' to 'yes' when configuring Greengrass core.
	   	
	   	Missing optional dependencies:
	   	1. Could not find the binary 'nodejs6.10'.
	   	
	   	If NodeJS 6.10 or later is installed on the device, name the binary 'nodejs6.10' and
	   	add its parent directory to the PATH environment variable. NodeJS 6.10 or later is
	   	required to execute NodeJS lambdas on Greengrass core.
	   	
	   	2. Could not find the binary 'java8'.
	   	
	   	If Java 8 or later is installed on the device name the binary 'java8' and add its
	   	parent directory to the PATH environment variable. Java 8 or later is required to
	   	execute Java lambdas on Greengrass core.
	
	
	   ​	
	   	----------------------------------Exit status---------------------------------------
	   	You can now proceed to installing the Greengrass core 1.6 software on the device.
	   	Please reach out to the AWS Greengrass support if issues arise.
   </details>

<a name="1.4"></a>
## 1.2 Greengrass creation in the AWS Management Console

Before we install *Greengrass* we need to create the necessary certificates to be able to authenticate our *Greengrass* installation against *AWS Iot Core*. We need to create the *Grengrass* certificates in the AWS console.

1. In your browser, bring up the AWS Management Console, change to the AWS IoT Core service and select the Greengrass category and select to **Get Started** in `Define a Greengrass Group` section.
	![14_1](../images/14_1.png)

1. Select **Use easy creation** button.
	![14_2](../images/14_2.png)

1. Name your group `sbe_workshop` and select **Next**.
	![14_3](../images/14_3.png)

1. Confirm the name of the Greengrass Group's Core is `sbe_workshop_Core` and select **Next**.
	![14_4](../images/14_4.png)

1. Select to **Create Group and Core**.
	![14_5](../images/14_5.png)
	
1. Select to **Download these resources as a tar.gz**
	![14_6](./images/14_6.png)
	![14_7](./images/14_7.png)
	
1. Scroll down the page and download the correct Greengrass binary for your system, in our case this should be `x86_64  Ubuntu 14.04 - 16.04  Linux` select the **Download** link next to it.
	![14_8](../images/14_8.png)
	
1. Confirm that the Greengrass Group was successfully created.
	![14_9](../images/14_9.png)

### 1.5 Setting up Greengrass on the EC2 instance

Now we have created the *Greengrass* group definition and have downloaded the certificates for *Greengrass* as well as the correct *Greengrass* binary.

1. Copy tarball with the certificate and configuration information from your computer to the EC2 instance.

	<details>
		<summary>Command: `scp -i "~/Downloads/SBE_Workshop.pem" ~/Downloads/7400e5b0bd-setup.tar.gz ubuntu@ec2-34-222-222-74.us-west-2.compute.amazonaws.com:/home/ubuntu`</summary>
	
		Output:
		
		7400e5b0bd-setup.tar.gz                       100% 2743     7.8KB/s   00:00
	</details>

1. Copy the Greengrass binary from your computer to the EC2 instance.

	<details>
		<summary>Command: `scp -i "~/Downloads/SBE_Workshop.pem" ~/Downloads/greengrass-ubuntu-x86-64-1.6.0.tar.gz ubuntu@ec2-34-222-222-74.us-west-2.compute.amazonaws.com:/home/ubuntu`</summary>
	
		Ouput:
		
		greengrass-ubuntu-x86-64-1.6.0.tar.gz         100% 9364KB  64.0KB/s   02:26
	</details>
	
1. On the EC2 instance extract the Greengrass binary.

	<details>
		<summary>Command: `sudo tar -xzvf greengrass-ubuntu-x86-64-1.6.0.tar.gz -C /`</summary>
	
		Output:
		
		greengrass/
		greengrass/certs/
		greengrass/certs/README
		greengrass/ggc/
		greengrass/ggc/core
		greengrass/ggc/packages/
		greengrass/ggc/packages/1.6.0/
		greengrass/ggc/packages/1.6.0/lambda/
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGShadowSyncManager
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGShadowService
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGCloudSpooler:1
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGTES
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGConnManager
		greengrass/ggc/packages/1.6.0/lambda/GreengrassSystemComponents/
		greengrass/ggc/packages/1.6.0/lambda/GreengrassSystemComponents/greengrassSystemComponents
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGIPDetector:1/
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGIPDetector:1/ipdetector
		greengrass/ggc/packages/1.6.0/lambda/arn:aws:lambda:::function:GGDeviceCertificateManager
		greengrass/ggc/packages/1.6.0/release_notes_1_6_0.html
		greengrass/ggc/packages/1.6.0/runtime/
		greengrass/ggc/packages/1.6.0/runtime/java8/
		greengrass/ggc/packages/1.6.0/runtime/java8/aws-greengrass-ipc-java-sdk-1.0.jar
		greengrass/ggc/packages/1.6.0/runtime/java8/aws-greengrass-java-common-1.0.jar
		greengrass/ggc/packages/1.6.0/runtime/java8/aws-greengrass-java-lambda-runtime-1.0.jar
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/try.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/localWatchLogger.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/functionArnFields.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/retry.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/index.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/versionParser.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/encodingType.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/config.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-common-js/envVars.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-ipc-sdk-js/
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-ipc-sdk-js/ipcclient.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/node_modules/aws-greengrass-ipc-sdk-js/index.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/redirect.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/start.js
		greengrass/ggc/packages/1.6.0/runtime/nodejs6.10/lambda_nodejs_runtime.js
		greengrass/ggc/packages/1.6.0/runtime/python2.7/
		greengrass/ggc/packages/1.6.0/runtime/python2.7/lambda_runtime.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/ipc_client.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/__init__.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/__init__.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/utils/
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/utils/__init__.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/utils/__init__.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/utils/exponential_backoff.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/utils/exponential_backoff.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_ipc_python_sdk/ipc_client.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/parse_version.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/env_vars.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/common_log_appender.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/__init__.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/parse_version.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/__init__.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/env_vars.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/encoding_type.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/common_log_appender.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/greengrass_message.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/greengrass_message.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/function_arn_fields.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/function_arn_fields.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/greengrass_common/encoding_type.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/__init__.pyc
		greengrass/ggc/packages/1.6.0/runtime/python2.7/__init__.py
		greengrass/ggc/packages/1.6.0/runtime/python2.7/lambda_runtime.py
		greengrass/ggc/packages/1.6.0/runtime/executable/
		greengrass/ggc/packages/1.6.0/runtime/executable/libaws-greengrass-core-sdk-c.so
		greengrass/ggc/packages/1.6.0/bin/
		greengrass/ggc/packages/1.6.0/bin/daemon
		greengrass/ggc/packages/1.6.0/LICENSE/
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_docker_docker_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_seccomp_libseccomp_golang_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_godbus_dbus_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_pquerna_ffjson_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_coreos_go_systemd_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_huin_gobinarytest_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_syndtr_gocapability_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_huin_mqtt_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_docker_go_units_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_fsnotify_fsnotify_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_opencontainers_runc_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_opencontainers_runtime_spec_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_Sirupsen_logrus_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/curl_haxx_se_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_urfave_cli_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_vishvananda_netlink_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/sqlite_org_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_jmespath_go_jmespath_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_paho_mqtt_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_golang_protobuf_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/Golang_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_aws_aws_sdk_go_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/libb64_sourceforge_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_mattn_go_sqlite3_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_jeffallen_mqtt_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_go_ini_ini_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/attributions/github_nu7hatch_gouuid_License.txt
		greengrass/ggc/packages/1.6.0/LICENSE/Greengrass AWS SW License (IoT additional) vr6.txt
		greengrass/ggc/packages/1.6.0/greengrassd
		greengrass/config/
		greengrass/config/config.json
		greengrass/ota/
		greengrass/ota/ota_agent_v1.0.0/
		greengrass/ota/ota_agent_v1.0.0/ggc-ota
		greengrass/ota/ota_agent_v1.0.0/LICENSE/
		greengrass/ota/ota_agent_v1.0.0/LICENSE/attributions/
		greengrass/ota/ota_agent_v1.0.0/LICENSE/attributions/github_davegamble_cjson_License.txt
		greengrass/ota/ota_agent_v1.0.0/LICENSE/attributions/github_eclipse_mosquitto_License.txt
		greengrass/ota/ota_agent_v1.0.0/LICENSE/Greengrass AWS SW License vr6.txt
		greengrass/ota/ota_agent
	</details>
	
1. Extract the certificate and configuration information into the `/greengrass` directory.

	<details>
		<summary>Command: `sudo tar -xzvf 7400e5b0bd-setup.tar.gz -C /greengrass`</summary>
	
		Output:
		
		certs/7400e5b0bd.cert.pem
		certs/7400e5b0bd.private.key
		certs/7400e5b0bd.public.key
		config/config.json
	</details>
	
1. Let's change into the directory on the EC2 instance that holds all the certificates used for authentication.

	Command: `cd /greengrass/certs/`
	
1. On the EC2 instance we now have certificates created for *Greengrass* but we still need the root certificate used for validation. We can download that with the following command.

	<details>
		<summary>Command: `sudo wget -O root.ca.pem http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem`</summary>
	
		Output:
		
		--2018-10-26 15:31:55--  http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
		Resolving www.symantec.com (www.symantec.com)... 23.195.225.59, 2600:1409:0:58c::145b, 2600:1409:0:595::145b
		Connecting to www.symantec.com (www.symantec.com)|23.195.225.59|:80... connected.
		HTTP request sent, awaiting response... 200 OK
		Length: 1758 (1.7K) [text/plain]
		Saving to: ‘root.ca.pem’
		
		root.ca.pem         100%[===================>]   1.72K  --.-KB/s    in 0s
		
		2018-10-26 15:31:55 (151 MB/s) - ‘root.ca.pem’ saved [1758/1758]
	</details>

1. Verify that the certificate is not empty.

	<details>
		<summary>Command: `cat root.ca.pem`</summary>
	
		Output:
		
		-----BEGIN CERTIFICATE-----
		MIIE0zCCA7ugAwIBAgIQGNrRniZ96LtKIVjNzGs7SjANBgkqhkiG9w0BAQUFADCB
		yjELMAkGA1UEBhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQL
		ExZWZXJpU2lnbiBUcnVzdCBOZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJp
		U2lnbiwgSW5jLiAtIEZvciBhdXRob3JpemVkIHVzZSBvbmx5MUUwQwYDVQQDEzxW
		ZXJpU2lnbiBDbGFzcyAzIFB1YmxpYyBQcmltYXJ5IENlcnRpZmljYXRpb24gQXV0
		aG9yaXR5IC0gRzUwHhcNMDYxMTA4MDAwMDAwWhcNMzYwNzE2MjM1OTU5WjCByjEL
		MAkGA1UEBhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQLExZW
		ZXJpU2lnbiBUcnVzdCBOZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJpU2ln
		biwgSW5jLiAtIEZvciBhdXRob3JpemVkIHVzZSBvbmx5MUUwQwYDVQQDEzxWZXJp
		U2lnbiBDbGFzcyAzIFB1YmxpYyBQcmltYXJ5IENlcnRpZmljYXRpb24gQXV0aG9y
		aXR5IC0gRzUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCvJAgIKXo1
		nmAMqudLO07cfLw8RRy7K+D+KQL5VwijZIUVJ/XxrcgxiV0i6CqqpkKzj/i5Vbex
		t0uz/o9+B1fs70PbZmIVYc9gDaTY3vjgw2IIPVQT60nKWVSFJuUrjxuf6/WhkcIz
		SdhDY2pSS9KP6HBRTdGJaXvHcPaz3BJ023tdS1bTlr8Vd6Gw9KIl8q8ckmcY5fQG
		BO+QueQA5N06tRn/Arr0PO7gi+s3i+z016zy9vA9r911kTMZHRxAy3QkGSGT2RT+
		rCpSx4/VBEnkjWNHiDxpg8v+R70rfk/Fla4OndTRQ8Bnc+MUCH7lP59zuDMKz10/
		NIeWiu5T6CUVAgMBAAGjgbIwga8wDwYDVR0TAQH/BAUwAwEB/zAOBgNVHQ8BAf8E
		BAMCAQYwbQYIKwYBBQUHAQwEYTBfoV2gWzBZMFcwVRYJaW1hZ2UvZ2lmMCEwHzAH
		BgUrDgMCGgQUj+XTGoasjY5rw8+AatRIGCx7GS4wJRYjaHR0cDovL2xvZ28udmVy
		aXNpZ24uY29tL3ZzbG9nby5naWYwHQYDVR0OBBYEFH/TZafC3ey78DAJ80M5+gKv
		MzEzMA0GCSqGSIb3DQEBBQUAA4IBAQCTJEowX2LP2BqYLz3q3JktvXf2pXkiOOzE
		p6B4Eq1iDkVwZMXnl2YtmAl+X6/WzChl8gGqCBpH3vn5fJJaCGkgDdk+bW48DW7Y
		5gaRQBi5+MHt39tBquCWIMnNZBU4gcmU7qKEKQsTb47bDN0lAtukixlE0kF6BWlK
		WE9gyn6CagsCqiUXObXbf+eEZSqVir2G3l6BFoMtEMze/aiCKm0oHw0LxOXnGiYZ
		4fQRbxC1lfznQgUy286dUV4otp6F01vvpX1FQHKOtw5rDgb7MzVIcbidJ4vEZV8N
		hnacRHr2lVz2XTIIM6RUthg/aFzyQkqFOFSDX9HoLPKsEdao7WNq
	</details>
	
	*(Optional)* In order to go further with your validation of the certificate you can also use the following command: `openssl x509 -text -noout -in ./root.ca.pem`
	
1. Now it is time to start *Greengrass* for the first time, in order to do this we need to change into the right directory.

	Command: `cd /greengrass/ggc/core/`

1. Then using the correct permission by assuming `superuser` rights we can start the *Greengrass* daemon `greengrassd`.

	<details>
		<summary>Command: `sudo ./greengrassd start`</summary>
		Output:
	
		Setting up greengrass daemon
		Validating hardlink/softlink protection
		Found cgroup subsystem:  blkio
		Found cgroup subsystem:  hugetlb
		Found cgroup subsystem:  perf_event
		Found cgroup subsystem:  devices
		Found cgroup subsystem:  net_cls
		Found cgroup subsystem:  net_prio
		Found cgroup subsystem:  memory
		Found cgroup subsystem:  freezer
		Found cgroup subsystem:  pids
		Found cgroup subsystem:  cpu
		Found cgroup subsystem:  cpuacct
		Found cgroup subsystem:  cpuset
		Found cgroup subsystem:  name=systemd
		
		Greengrass successfully started with PID:  7452
	</details>

1. In order to verify that the Greengrass daemon is running we can use the `ps` command.

	<details>
		<summary>Command: `ps aux | grep greengrassd`</summary>
	
		Output:
		
		root      7452  0.4  0.0 646800 20832 pts/0    Sl   15:38   0:00 /greengrass/ggc/packages/1.6.0/bin/daemon -core-dir /greengrass/ggc/packages/1.6.0 -greengrassdPid 7447
		ubuntu    7570  0.0  0.0  12944   968 pts/0    S+   15:38   0:00 grep --color=auto greengrassd
	</details>
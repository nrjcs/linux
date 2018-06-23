# Linux Operating Systems for Hadoop Administration

**NOTE:** This guide uses CentOS operating system. Other linux flavours might have slightly different options.

There are different installation options.

1. Minimal install - comes with console only
2. Compute Node - used for installation on cloud
3. Infrastructure Server - used to form infrastructure of different servers
4. File and Print Server - using server just to keep files and send to printers
5. Basic web Server - comes with minimal install and apache web server.
6. Virtualization Host - virtualization layer set up
7. Server with GUI - server infrastructure with GUI to enable learning.
8. GNOME Desktop - working desktop installation
9. KDE Plasma workspaces - similar to last one but only graphics are different
10. Development and Creative Workstation - used for designers

ls - list
touch <filename> - create new blank file
man <command> - see the manual for a given command
ls -lh - list the file in human readable format.

### Setting Kernel Parameters

Linux Kernel is the core of the OS. It is the main process that talks to the hardware. When user send command to shell, it sends it to kernel and communicates the same to hardware. Kennel parameters allows the hardware in certain manner.

`ping <ip_address>` will give replies from the ip address when system is available on the network. We can also disable this by changing kernel parameters. We can set kernel parameters to not respond to USB drives.

Linux boot process involves: power on the SMPS, BIOS (basic input output system) contains information about all hardware components and it fires them up and let the OS know which components are connected. If BIOS fails to load Grub, it would report 'Missing Operating System'
Load stage 1 Grub from MBR. Linux has bootloader. Before Kernel comes in bootloader talks to hardware. This is also called Grub. It loads from Master boot record, available in first sector of harddisk. It reads partition table, knows the partition.
It reaches stage 1.5 where it displays menulist - here it lists which OS are available. If something goes wrong, it would not move forward.

If everything is fine, then it would list all available OS. (menu.list)
After this it loads Kernel image and initial RAM disk. If there is something wrong, it would report 'File not found'
After this Kernel mounts the root file system and runs `init`. If mounting of root file system fails, it would just freeze.
`init` runs scripts to start user-level services.

User can interact with Shell and shell interacts with Kernel. Shell is an interpreter for user commands.

`echo $SHELL` shows which shell we are using.

`sysctl` is an interface that allows us to make changes to running linux kernel. At the time of booting again, the settings would disappear. To keep these settings, we need to write our settings to `/etc/sysctl.conf`.

Real time kernel parameter data is stored in `/proc` directory. Particularly `/proc/sys` is where you can find all the information about devices, drivers and kernel features.

```shell
man sysctl
vi /etc/sysctl.conf
sysctl -a # list all kernel settings
sysctl -a | wc -l # count number of lines for these parameters
cat /proc/sys/net/ipv4/ip_forward # shows ip forwarding option
echo "0" > /proc/sys/net/ipv4/ip_forward # turn off ip forwarding
# To make this setting permanent, we have to write this option in `/etc/sysctl.conf` file.
sysctl -p # reloads the `sysctl.conf` options.

# To stop accepting ping requests
vi /etc/sysctl.conf
net.ipv4.icmp_echo_ignore_all=1 # ignore ipv4 requests
sysctl -p # now ping will not work for this machine.
# To revert, set it to 0 again.
```

## Package Manager

Linux uses Package manager softwares for installation of different software versions.
Package managers allow packages to be downloaded with their dependencies. In Linux, developers create bundle of softwares that can be installed with package mangager.
RPM = Redhat package manager

Tar files allow us to build from source files.

`ssh user@ip_address` is used to connect to given machine using given username with SSH protocol.

RPM is free and released under GPL. It keeps the information of all the installed packages under `/var/lib/rpm` database. RPM deals with `.rpm` files which contains the actual information about the packages to be installed along with versions. We get packages from yum repos. Apart from that we can also find online libraries like:

[http://rpmfind.net](http://rpmfind.net)
[http://www.redhat.com](http://www.redhat.com)
[http://freshrpms.net](http://freshrpms.net)
[http://rpm.pbone.net](http://rpm.pbone.net)

```shell
man rpm
cat /etc/redhat-release
uname -a # parameters of the kernel
wget http://download-link.com/download.file # download the file from web
# Install rpm package
# i = install, v = verbose, h = hash checking and progress view
rpm -ivh some-package.rpm

# remove rpm package
rpm -e some-package # erase package
rpm -qa # list rpms that are present in a system, q = query, a = all
rpm -qa > rpms-list.txt # redirect the list of rpms to a file, create a file
rpm -ql somepackage # query list the files related to this package has been installed.
rpm -qpA somepackage.rpm # show libraries that this rpm uses to extract or install
rpm -Uvh some-package.rpm # upgrade this package

# yum manages the dependencies and installs required dependencies for a given package.
yum install httpd # yellowdog updater modified
yum remove httpd # uninstall a package (httpd)
netstat -tpnl # netstat shows the port and address where our machine is listening
yum whatprovides netstat # what package provides given command
```

**IO Redirection for error logging**

We can redirect the output using redirection operator.

```shell
1 - standard output
2 - standard error stream

ls nosuchdir 2>&1 # redirect the output of error stream to stdout
ls nosuchdir 2>filename.txt # output the error to given filename
ls nosuchdir 2>>filename.txt # redirect and append the error to filename
0< FILENAME # accept input from the FILENAME
```

## System Services

OS comprises set of services that are started when kernel loads. These services run in the background to keep the OS up and alive. Process is a program that has been loaded long-term storage device, usually a HDD into System RAM and is currently being processed by the CPU on motherboard.There are various types of processes. _User process_ is a process created by users.

`ps aux` gives the output of various processes running. It lists user, PID, command, CPU, memory, etc. When user executes a command, it creates a process in the background. For example, running `ping` creates a separate process. Use `ps aux | grep ping` to get ping process information.

_System Process_ are processes that run as daemon. Every service running in a server run daemon in the background. These are not invoked by the user but run in the background.

```shell
yum install httpd
systemctl start httpd # start httpd daemon
systemctl status httpd # show the status of httpd daemon
systemctl list-unit-files # list all system services
systemctl list-unit-files | grep disabled
systemctl enable <process_name> # enable a process
systemctl disable httpd # disable and run only when invoked manually
systemctl stop httpd # stop the service
systemctl restart httpd # restart the service
systemctl --failed # show the failed services

ps aux | grep httpd # there might be many processes running which might be child processes for apache daemon

top # shows the linux processes. It provides real time view of the running system.
# We can see a lot of details about the processes. Zombie processes are the processes where the parent process has died but they haven't sent the shutdown signal to child processes.
# Swap is the memory which is used when we are low on memory. It uses these harddisk memory.
```

PID - Process ID
USER - name of the user that owns the process
PR - priority assigned to the user. This can be used to allocate priority in case of memory crunch situations
NI - Nice value of the process. It is also priority
VIRT - Virtual memory used by the process, Virtual memory is actually the part of the hard disk.
RES - The amount of physical RAM the process is using.
SZ - Size of the process
WCHAN - the name of the kernel function in which the process is sleeping

```shell
ps # shows PID, TTY, TIME, CMD
ps -ef # we get more information
# STAT includes the state of the proces, D - Uninterruptible sleep, R- running, S - Interruptible sleep, T - Stopped or traced, Z - Zombied
```

**CRON** is a daemon to execute scheduled tasks. It is automatically started with `/etc/init.d` on entering multi user runlevels.
_Scheduling of a process_ can be done using `cron` command.

Let's create a script.

```shell
vi test-script.sh
#!/bin/sh
echo "Starting script"
ping -c 2 google.com
echo script ran at date >> /workspace/test-script.log
echo "Stopping Script"
exit
:wq!

chmod 755 starting.sh
bash test-script.sh

# To make this script run automatically at a scheduled time, we can use cron command.
crontab -e # edit the crontab
```

* Cron job has five columns and each column sets the time of the day.
  From left 1st one is minute (0-59)
  2nd one is Hour (0-23)
  3rd Day of the month (1-31)
  4th Month (1 - 12)
  5th - Day of week (0-6)(sunday=0)

```shell
crontab -e
*/1 * * * * /workspace/test-script.sh # run this script every minute, every hour, every hour, every month

crontab -e # run at specific time 00:00AM
00 00 * * * # run at 00.00AM everyday

00 00 1 4 * # run on every april 1st, every year
```

## User Management

We can manually add users using `etc/passwd` and `etc/shadow` file.
Use `useradd` or `adduser` command to create a new user depending on the Linux flavour.

`useradd -g <group> -d <home_directory> -c <comment> -s <shell> login-name`

Use `groupadd` to create a new group
`groupadd <group_name>`
`passwd login-name`

In GUI, we can go to `Applications -> Settings -> Users and Groups`

Applications => Settings => User => Unlock => Click + => Enter full Name, Username, Allow user to set a password when they next login

**Using Shell**

```shell
vi /etc/passwd # lists usernames
useradd piyush # create user
passwd piyush # create password for user named piyush
useradd -h # help with arguments
useradd -b /home/piyush.patel/ piyush # create user with home directory with different directory than username
useradd -G manager username # put username to group manager
vi /etc/passwd
# Above file shows details like login name, password, user id(uid), group id(gid), comment, home directory, shell
vi /etc/shadow # shows login name, encrypted password, date since unix timestamp count the password was changed, date before password may not be changed, etc.
ssh piyush@<ip_address> # login using ssh with new user

useradd mary
passwd mary
# change home directory of user
usermod -d /home/testnew <test_user>

# change user group
usermod -G <group> <user>

groupadd <group_name> # create new group
usermod -g <group> <user> # change primary group of user

usermod -L <username> # lock the user from acessing
usermod -U <username> # unlock the user

usermod -s /bin/bash <username> # change shell for a given user

userdel <username> # delete a user by username

pgrep -u <username> # grep the process by username
kill -9 <pid> <pid2> # kill given process ids
```

## File System

In UNIX, everything is file; device drivers, programs, folders, etc. Linux filesystem has different types of partitions.

`df -Th` # see how file system is partitioned
which ls # shows where the command file is located
`/bin` contains user binaries
/sbin`contains system binaries`/etc`- configuration files for packages installed`/dev`- devices logically mapped and defined in this pc`/proc`- process information, contains kernel parameters that change real time`/var`- variable file`/tmp`- to store temporary files`/usr`- user programs`/home`- home directory for the user`/boot`- Boot loader files`/lib`- System libraries`/opt`- Optional add-on Applications`/mnt`- Mount directories`/media`- Removable directories`/srv` - Service data

**Commands**

```shell
ls # list the files and directories
ls -l # long listing
ls -a # all files including hidden files
pwd # print working directory
cat filename # shows the file content of the file
# big files can be opened with less command
cat filename | less
tail filename  # shows last 10 lines
head filename # shows first 10 lines
tail -n 2 filename # shows last 2 lines

vi filename # create  and open file
i # enter insert mode in vi
Esc -> :wq! # save and exit the file
:w -> # save and continue writing
Esc -> :q! # exit the file
:x! -> save and exit

grep -i -R "hi" /dirname/ # find all files containing word hi in a given directory

find / -name passwd # file filenamed 'passwd' in / directory recursively

locate <filename> # shows files which containe the word in name

cp file1 /tmp/file2 # copy file1 content to file2
mv file1 /tmp/file2 # move the file1 to file2 location
rm file1 # remove file1
rm -rf dirname # remove directory forcefully recursively

fdisk -l # shows different devices
mount /dev/cdrom /media # mount a filesystem
umount /media # unmount cdrom
mount <name_of_device> <media_location>

tar czvf etc.tar.gz /etc # tar archive /etc directory to etc.tar.gz
clear # shortcut Ctrl+L

df # show disk system disk space usage
df -Th # human readable format
du -sh /etc/ # disk usage
fdisk -l # lists devices, HDD available, external devices connected, etc.

ls -l # lists files with access permissions, the first says whether it is directory or file, next three are for user, then group and the next three are for other users.
# Then it lists username, group
```

# Oracle Database 12c Installation on CentOS 7

# 1. Introduction
This guide presents how to deploy Oracle Database 12c (12.1.0.2.0) on CentOS 7.1 (64-bit) using quick installation features.

Reference System: 

```
[root@centos7 ~]# hostnamectl
   Static hostname: centos7.example.com
         Icon name: computer
           Chassis: n/a
        Machine ID: 583b4d69eaea465ea4bb96ac3b891e15
           Boot ID: 931ed1af622046ebbde071a87844a7d5
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-229.11.1.el7.x86_64
      Architecture: x86_64
```

# 2. Prerequisites

After a successful OS installation, verify the hostname and register it in your DNS. Alternatively add your hostname/IP to the /etc/hosts. 


```
[root@centos7 ~]# cat /etc/hostname
centos7.example.com
```

Leave the SELinux in enforcing mode as well as the firewall enabled 

```
[root@centos7 ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

```
[root@centos7 ~]# firewall-cmd --state
running
```
Update the CentOS system with the latest packages 

```
[root@centos7 ~]# yum update -y
```

Download the Oracle Database 12c for Linux x86-64 software:

http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html


# 3. Installation Steps

Create required OS users and groups for Oracle Database. 


```
[root@centos7 ~]# groupadd oinstall
[root@centos7 ~]# groupadd dba
[root@centos7 ~]# useradd -g oinstall -G dba oracle
[root@centos7 ~]# passwd oracle
```

Add the following kernel parameters to the /etc/sysctl.conf

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 1987162112
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```


Check and apply the new values. 


```
[root@centos7 ~]# sysctl -p
[root@centos7 ~]# sysctl -a
```

Specify limits for oracle user in the /etc/security/limits.conf


```
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
```

Extract the zipped Oracle Database Software archives (linuxamd64_12102_database_1of2.zip, linuxamd64_12102_database_2of2.zip) to the /stage folder.


```
[root@centos7 ~]# yum install -y zip unzip
[root@centos7 ~]# unzip linuxamd64_12102_database_1of2.zip -d /stage/
[root@centos7 ~]# unzip linuxamd64_12102_database_2of2.zip -d /stage/
```


Modify permissions on /stage

```
[root@centos7 ~]# chown -R oracle:oinstall /stage/
```


Create /u01 directory for Oracle software and /u02 for database files. 

```
[root@centos7 ~]# mkdir /u01
[root@centos7 ~]# mkdir /u02
[root@centos7 ~]# chown -R oracle:oinstall /u01
[root@centos7 ~]# chown -R oracle:oinstall /u02
[root@centos7 ~]# chmod -R 775 /u01
[root@centos7 ~]# chmod -R 775 /u02
[root@centos7 ~]# chmod g+s /u01
[root@centos7 ~]# chmod g+s /u02
```


Install required packages: 

```
[root@centos7 ~]# yum install -y binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.i686 glibc.x86_64 \
glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 libaio.i686 libaio.x86_64 libaio-devel.i686 libaio-devel.x86_64 \
libgcc.i686 libgcc.x86_64 libstdc++.i686 libstdc++.x86_64 libstdc++-devel.i686 libstdc++-devel.x86_64 libXi.i686 libXi.x86_64 \
libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64
```
Also install the "X Window System" package group. 

```
[root@centos7 ~]# yum groupinstall -y "X Window System"
```

Because the Oracle installation requires GUI, there are two simple ways how to provide it.

# Solution 1.

Login remotely from a graphical Linux machine via SSH.



```
ssh -X oracle@centos7.example.com
```
# Solution 2.

Use a Microsoft Windows desktop with an SSH Client (PuTTY) and an X-Windows terminal emulator (Xming).

The following article describes how to install Xming on Windows systems.

* Xming X-Windows terminal emulator for Microsoft Windows computers

Use a solution above to login as "oracle" user and run the Oracle Installer: 


```
[oracle@centos7 ~]$ /stage/database/runInstaller
Starting Oracle Universal Installer...
```

# 4. Oracle Installer Screens

----------------------------
# STEP 1 - Security Updates 
----------------------------

If you don't wish to receive emails from Oracle Support then deselect the check box and click Next.

Another window opens click YES on it. 

-------------------------------
# STEP 2 - Installation Option 
-------------------------------

Select Create and configure a database and click Next

------------------------
# STEP 3 - System Class 
------------------------

Select Desktop Class for a simple default installation of the Oracle Database. 

--------------------------------
# STEP 4 - Typical Installation 
--------------------------------

On the Typical Install Configuration screen, specify the key features. 


------------------------------------------------------------------
Oracle base              /u01/app/oracle 

Software location        /u01/app/oracle/product/12.1.0/dbhome_1 

Database file location   /u02 

Global database name     orcl.example.com 
------------------------------------------------------------------




Also specify the suitable Database edition as well as the Character set. Provide a strong password for database administration and finally deselect the Create as Container database option. 

----------------------------
# STEP 5 - Create Inventory 
----------------------------

Accept the default /u01/app/oraInventory and click Next. 


--------------------------------
# STEP 6 - Prerequisite Checks  
--------------------------------


The Installer automatically verifies all required OS packages and OS kernel settings.

-------------------
# STEP 7 - Summary 
-------------------



Final chance to edit any installation features. Click Install.

-----------------------------------------
# STEP 8 - Execute Configuration Scripts
-----------------------------------------


When a request window appears, login as root and execute two scripts: 


```
[root@centos7 ~]# /u01/app/oraInventory/orainstRoot.sh
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.
Changing groupname of /u01/app/oraInventory to oinstall.
The execution of the script is complete.
```


```
[root@centos7 ~]# /u01/app/oracle/product/12.1.0/dbhome_1/root.sh
Performing root user operation.
The following environment variables are set as:
    ORACLE_OWNER= oracle
    ORACLE_HOME=  /u01/app/oracle/product/12.1.0/dbhome_1
Enter the full pathname of the local bin directory: [/usr/local/bin]: <PRESS ENTER>
   Copying dbhome to /usr/local/bin ...
   Copying oraenv to /usr/local/bin ...
   Copying coraenv to /usr/local/bin ...
Creating /etc/oratab file...
Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
You can follow the installation in a separated window.
```

Both scripts must run as root. 

---------------------------------
# STEP 9 - Installation progress 
---------------------------------

Another window opens to show you the installation progress. Do not close this window. 

------------------------------------------------
# STEP 10 - Installation completed successfully 
------------------------------------------------

The last screen inform that the installation is done and displays the Oracle Enterprise Manager URL.

https://localhost:5500/em

Click OK to close the Installer. 


# 5. Post Installation Tasks

- 5.1. Firewall

Login as root and verify the active zones 

```
[root@centos7 ~]# firewall-cmd --get-active-zones
public
  interfaces: eth0
```

Open the related ports 

```
[root@centos7 ~]# firewall-cmd --zone=public --add-port=1521/tcp --add-port=5500/tcp --add-port=5520/tcp --add-port=3938/tcp \ 
--permanent
success
```

```
[root@centos7 ~]# firewall-cmd --reload
success
```

```
[root@centos7 ~]# firewall-cmd --list-ports
1521/tcp 3938/tcp 5500/tcp 5520/tcp
```


- 5.2. Oracle Environment


Login as oracle user and add the following values to the /home/oracle/.bash_profile


```
TMPDIR=$TMP; export TMPDIR
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/12.1.0/dbhome_1; export ORACLE_HOME
ORACLE_SID=orcl; export ORACLE_SID
PATH=$ORACLE_HOME/bin:$PATH; export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/lib64; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
```


Reload the bash_profile to apply the new settings: 

```
[oracle@centos7 ~]$ . .bash_profile
```


- 5.3. Login to the database

Finally login to the database: 

```
[oracle@centos7 ~]$ sqlplus system@orcl
... output omitted ...
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
SQL>
```

Manage database with Oracle Enterprise Manager: 


https://<hostname>:5500/em 


# Have fun with nu11secur1ty =)








































































































































































































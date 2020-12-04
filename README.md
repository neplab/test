---
title: Installing Oracle 12C Database for OIM (Silently)
author: Identity Manager
date: 2020-11-22
hero: ./images/hero.jpg
excerpt: In this blog, we'll install required database server for oim 12c silently
categories:
  - database
  - oracle
---

### Specs of VM used

CentOS 7.8
Ram: 8 GB

### Required Files - Database installer file

linuxx64_12201_database.zip
source: (https://download.oracle.com/otn/linux/oracle12c/122010/linuxx64_12201_database.zip)

_make sure to copy this file in /tmp using winscp or upload it to cloud bucket._

## Installation Steps

### Install required packages - as root

```bash
yum install -y binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 \
glibc.i686 glibc.x86_64 glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 \
libaio.i686 libaio.x86_64 libaio-devel.i686 libaio-devel.x86_64 libgcc.i686 \
libgcc.x86_64 libstdc++.i686 libstdc++.x86_64 libstdc++-devel.i686 \
libstdc++-devel.x86_64 libXi.i686 libXi.x86_64 libXtst.i686 libXtst.x86_64 \
make.x86_64 sysstat.x86_64 unzip zip xauth xclock xdpyinfo vim xorg-x11-xauth
```

### Creating swap file - as root

```bash
sudo dd if=/dev/zero of=/swapfile bs=8192 count=1048576
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Adding users and groups - as root

```bash
groupadd oinstall
groupadd dba
useradd oracle
usermod -g oinstall oracle
usermod -G dba oracle
```

### Creating oracle dir - as root

```bash
mkdir /u01/
mkdir /u01/software/
mkdir /u01/app/
```

### Getting the installer file - as root

Assuming you've uploaded your installer file in "/tmp" already, let's copy it in place

```bash
cp /tmp/linuxx64_12201_database.zip /u01/software/

```

### Setting Permission - as root

```bash
chown -R oracle:oinstall /u01
```

### Setting /etc/sysctl.conf - as root

Please use vi command to write following content

```bash
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 3835977728
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
fs.aio-max-nr = 1048576
```

Let's print the configuration to make sure its there

```bash
sysctl -p
```

Now that it's been verified, let's apply the configuration

```bash
sysctl -a
```

### Setting /etc/security/limits.conf - as root

Please use vi command to write following content

```bash
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
```

### stop firewall - as root

We need to disable firewall on server level

```bash
service firewalld stop
chkconfig firewalld off
```

### switch to user oracle at this point

```bash
sudo su - oracle
```

### Setting required variables - as oracle

Let's set ORACLE_HOME to "/u01/app/oracle/product/12.2.0/dbhome_1" (yet to be created)

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1" >> ~/.bashrc
export PATH=$PATH:/u01/app/oracle/product/12.2.0/dbhome_1/bin" >> ~/.bashrc
```

Also, let's set ORACLE_SID=testdb

```bash
export ORACLE_SID=iamdev" >> ~/.bashrc
```

After writing so many thing to ~/.bashrc, let's source it to make sure configuration loads in current shell

```bash
source ~/.bashrc
```

### Unzipping the installer - as oracle

Let's make sure the installer zip file is executable for oracle first

```bash
chmod +x /u01/software/linuxx64_12201_database.zip
```

We should be able to unzip the file at this point

```bash
unzip /u01/software/linuxx64_12201_database.zip
```

### Editing response file for database installation

Please use vi editor to write following content to the file "/u01/software/database/response/db_install.rsp"

```bash
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v12.2.0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=oimdb.us-central1-a.c.iam-tools-295120.internal
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oraInventory
SELECTED_LANGUAGES=en
ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=dba
oracle.install.db.OSBACKUPDBA_GROUP=dba
oracle.install.db.OSDGDBA_GROUP=dba
oracle.install.db.OSKMDBA_GROUP=dba
oracle.install.db.OSRACDBA_GROUP=dba
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true
oracle.installer.autoupdates.option=SKIP_UPDATES
```

### Begin installation

```bash
sh /u01/software/database/runInstaller -ignoreSysPrereqs -showProgress -silent -responseFile /u01/software/database/response/db_install.rsp
```

### Verify installation

### What's next

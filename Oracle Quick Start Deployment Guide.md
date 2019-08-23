基于金山云云服务部署高可用Oracle数据库服务

根据不同需求，基于金山云云服务器、专属云、物理主机、云硬盘和托管服务自建Oracle数据库，通过Oracle Data Guard实现高可用，实现主备数据复制、有计划切换(Switchover)和故障切换(Failover)，并利用对象存储归档数据库历史数据。具体的架构示意图如下：
![基于金山云的高可用Oracle数据库服务部署架构](https://raw.githubusercontent.com/ksc-sbt/qs-oracle/master/images/ksc-oracle-ha-architecture.png)

本文介绍采用金山云云服务器搭建高可用Oracle数据库（采用版本是Oracle Database 12c Release 1, 12.1.0.2）的过程，如果需要更稳定的性能和隔离性，可选择金山云的专属云或云物理主机。也可以把专有的Oracle RAC托管到金山云托管区，并通过内部专线实现和公有云服务互通。本文包括如下内容：

* 准备部署环境

* 安装Oracle主服务器(Primary Server)

* 安装Oracle备服务器(Standby Server)

* 配置Data Guard

* 备份Oracle数据，并归档到金山云对象存储

# 1. 准备部署环境
本文基于金山云北京6区部署Oracle数据库服务。具体的环境配置如下：

## 1.1 VPC配置信息

|  网络资源   | 名称  | CIDR  |
|  ----  | ----  | ----  |
| VPC  | sbt-vpc |	10.34.0.0/16 |

## 1.2 子网配置信息

| 子网名称 | 所属VPC |可用区 | 子网类型  | CIDR  | 说明|
|  ----  | ----  | ----  |----  |----  |----|
| private_a  | sbt-vpc |可用区A | 普通子网| 10.34.0.0/20|用于管理Oracle主云服务器和备云服务器|

## 1.3 安全组配置信息
在创建Oracle数据库云服务器实例时，需要关联安全组。下面是安全组(oracle-sg)的配置规则：

|协议|行为|起始端口|结束端口|源IP| 备注|
|----|----|----|----|----|---|
|TCP|允许|1521|1521|0.0.0.0/0| oracle|
|ICMP|允许|全部协议|全部协议|0.0.0.0/0|ping|
|TCP|允许|22|22|0.0.0.0/0|ssh|

## 1.4 NAT配置信息
为了能从公网下载Oracle安装软件，Oracle数据库云服务器所处的子网需要关联NAT实例。具体的NAT实例配置信息如下。

|名称|所属VPC|作用范围|类型|所绑定的子网|
|  ----  | ----  | ----  |----  |----  |
|Ksc_Nat|sbt-vpc|绑定的子网|公网|private_a|

## 1.5 Oracle数据库云服务器配置
本指南采用两台金山云云服务器，具体的配置信息如下。其中orasrv01是主服务器，orasrv02是备服务器。

|服务器名称|数据中心|可用区|子网|镜像|云服务器类型|系统盘|数据盘|
|----|----|----|----|----|----|----|----|
|orasrv01|北京6区(VPC)|可用区A|private_a|centos-7.6-201903131627|2核 4G（通用型N3）|云硬盘3.0(SSD)(50G)|云硬盘3.0(SSD)(100G)|
|orasrv02|北京6区(VPC)|可用区A|private_a|centos-7.6-201903131627|2核 4G（通用型N3）|云硬盘3.0(SSD)(50G)|云硬盘3.0(SSD)(100G)|
 
 为了能规范管理多台云服务器的配置，可提前创建实例启动模板，让后基于实例启动模板创建云服务器实例。下图是基于上述配置创建的云服务器实例启动模板。
![云服务器实例启动模板](https://raw.githubusercontent.com/ksc-sbt/qs-oracle/master/images/kec-template.png)

在创建完云服务器后，通过如下步骤完成安装Oracle所需的一些准备工作。

### 1.5.1 主机配置
修改主服务器机器名：
```bash
hostnamectl set-hostname orasrv01
```
修改备服务器机器名：
```bash
hostnamectl set-hostname orasrv02
```
修改主、被服务器的/etc/hosts，增加机器名和IP的解析记录。
```text
127.0.0.1 localhost
10.34.0.24 orasrv01
10.34.0.27 orasrv02
```
在主、备服务器上更新操作系统安装包，如果希望通过图形化界面安装Oracle，则需安装Linxu图形化安装包，并配置默认启动图形化界面。本文采用静默方式安装Oracle，可不用安装图形化界面。
```bash
yum update -y 
yum groupinstall -y "GNOME Desktop"
systemctl set-default graphical.target
```
如果希望恢复命令模式启动，可执行如下命令：
```bash
systemctl set-default multi-user.target
```

重新启动服务器后，通过金山云控制点击"连接实例"连接云服务器。
![控制台连接实例](https://raw.githubusercontent.com/ksc-sbt/qs-oracle/master/images/ksyun-console.png)

浏览器会弹出一个新的窗口，能看到图形化登录界面，表示云服务器的图形化程序包安装成功。
![Linux云服务器图形化界面](https://raw.githubusercontent.com/ksc-sbt/qs-oracle/master/images/kec-gui.png)

### 1.5.2 初始化数据盘
通过ssh登录到云服务器，完成云硬盘的初始化。
首先执行如下命令，获得块设备信息。
```bash
fdisk -l
```
在显示信息中，能查看块设备名称为/dev/vdb
```text
[root@orasrv01 ~]# fdisk -l

Disk /dev/vda: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a2d0a

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   104857566    52427759+  83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
执行如下命令，完成对/dev/vdb块存储的分区、格式化，并挂载到/u01目录下。
```bash
echo -e 'o\nn\np\n1\n\n\nw' | fdisk /dev/vdb
mkdir -p /u01
mkfs -t ext4 /dev/vdb1
echo '/dev/vdb1 /u01   ext4    defaults,noatime  1   1'>>/etc/fstab
mount /u01
```
初始化完成后，能看到如下文件系统信息，其中文件系统/dev/vdb1已经mount到/u01下。
```text
[root@orasrv01 ~]# df -m
Filesystem     1M-blocks  Used Available Use% Mounted on
/dev/vda1          50268  5188     42812  11% /
devtmpfs            1880     0      1880   0% /dev
tmpfs               1895     1      1895   1% /dev/shm
tmpfs               1895    10      1886   1% /run
tmpfs               1895     0      1895   0% /sys/fs/cgroup
tmpfs                379     1       379   1% /run/user/988
tmpfs                379     0       379   0% /run/user/0
/dev/vdb1         100664    61     95468   1% /u01
```
### 1.5.3 增加操作系统swap空间
新创建的云服务器实例的swap空间是0，根据Oracle手册中对swap空间大小的建议[https://docs.oracle.com/database/121/LADBI/pre_install.htm#LADBI7505](https://docs.oracle.com/database/121/LADBI/pre_install.htm#LADBI7505)，设置swap空间大小和内存大小一样。
```
[root@orasrv01 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        465M        2.6G         10M        681M        3.0G
Swap:            0B          0B          0B
```
通过如下命令增加swap空间。
 ```bash
mkdir -p /swap
dd if=/dev/zero of=/swap/swapfile.001 bs=1MB count=4096
chmod 600 /swap/swapfile.001
mkswap -f /swap/swapfile.001
echo '/swap/swapfile.001 swap   swap defaults,noatime  1   1'>>/etc/fstab
swapon /swap/swapfile.001
```
执行如下命令，显示swap空间已经增加成功。
```
[root@orasrv01 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        468M        142M         10M        3.1G        3.0G
Swap:          3.8G          0B        3.8G
```

### 1.5.4 修改Oracle所需的操作系统参数
首先修改/etc/sysctl.conf文件。
```bash
cat <<EOF >> /etc/sysctl.conf 
fs.aio-max-nr = 1048576  
fs.file-max = 6815744   
kernel.shmall = 409600000   
kernel.shmmax = 80530636000 
kernel.shmmni = 4096   
kernel.sem = 250 32000 100 128   
net.ipv4.ip_local_port_range = 9000 65500   
net.core.rmem_default = 262144  
net.core.rmem_max = 4194304  
net.core.wmem_default = 262144   
net.core.wmem_max = 1048576 
EOF
```
然后执行如下命令，使得配置生效。
```bash
sysctl -p
```
修改/etc/security/limits.conf
```bash
cat <<EOF >> /etc/security/limits.conf 
oracle soft nproc 2047   
oracle hard nproc 16384   
oracle soft nofile 1024   
oracle hard nofile 65536   
oracle soft stack 10240 
EOF
```
### 1.5.5 安装Oracle所需程序包
执行如下命令，安装所需要系统程序包。
```bash
yum install -y \
binutils \
compat-libcap1 \
compat-libstdc++-33 \
elfutils-libelf \
elfutils-libelf-devel \
gcc \
gcc-c++ \
glibc \
glibc-common \
glibc-devel \
glibc-headers \
ksh \
libaio \
libaio-devel \
libgcc \
libstdc++ \
libstdc++-devel \
make \
libXext \
libXtst \
libX11 \
libXau \
libxcb \
libXi \
sysstat \
unixODBC \
unixODBC-devel
```
### 1.5.5 初始化Oracle所需用户
创建Oracle组和账户oracle，并设置口令为Passw0rd。
```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
useradd -u 54321 -g oinstall -G dba,oper oracle
echo -e 'Passw0rd\nPassw0rd\n' | passwd oracle
chown -R oracle:oinstall /u01
```
### 1.5.6 修改sshd配置，启动密码登录
在创建云服务器时选择密钥登录，则在用ssh访问云服务器时，如果不带密钥文件就会访问失败。下面是错误信息。
```
[root@orasrv01 ~]# ssh oracle@orasrv01
The authenticity of host 'orasrv01 (10.34.0.24)' can't be established.
ECDSA key fingerprint is SHA256:b57v8qjP9hALvsugO/7oC93QIlXLkeD++W8fAAuUCJ4.
ECDSA key fingerprint is MD5:83:25:c1:93:1d:b1:9b:47:d3:9a:39:ee:73:da:f2:da.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'orasrv01,10.34.0.24' (ECDSA) to the list of known hosts.
Permission denied (publickey).
```
执行命令修改/etc/ssh/sshd_config，设置允许密码登录，并重启sshd服务。
```bash
sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd
```
执行如下命令，如果提示需要输入口令，表示sshd启用密码登录成功。
```
[root@orasrv01 ~]# ssh oracle@orasrv01
Password: 
```

### 1.5.7 下载Oracle安装包
通过访问Oracle网站（https://www.oracle.com/database/technologies/database12c-linux-downloads.html）可下载Oracle安装程序。为了缩短下载时间，安装包已存储在金山云对象存储上，可通过如下命令快速下载。从金山云云主机访问金山云对象存储，是通过金山云内网，下载速度能达到每秒30MB。
```bash
wget https://ks3-cn-beijing.ksyun.com/ksc-sbt-software/oracle/linuxamd64_12102_database_1of2.zip
wget https://ks3-cn-beijing.ksyun.com/ksc-sbt-software/oracle/linuxamd64_12102_database_2of2.zip
```
下面是下载后的文件。
```text
[oracle@orasrv01 ~]$ ls -l
total 2625088
-rw-r--r-- 1 oracle oinstall 1673544724 Aug 20 17:04 linuxamd64_12102_database_1of2.zip
-rw-r--r-- 1 oracle oinstall 1014530602 Aug 20 17:18 linuxamd64_12102_database_2of2.zip
[oracle@orasrv01 ~]$ 
```
执行unzip把文件解压缩。
```bash
unzip -q linuxamd64_12102_database_1of2.zip
unzip -q linuxamd64_12102_database_2of2.zip
```
此外，本文所静默安装Oracle的响应文件存储在github上，可通过如下命令在服务器上获得文件。
```bash
git clone https://github.com/ksc-sbt/qs-oracle.git
```

# 2 安装Oracle主服务器(Primary Server)

# 2.1 安装Oracle软件

文件/home/oracle/qs-oracle/install-soft.rsp是安装Oracle的配置文件，该文件有如下重要参数可能需要调整：
```
ORACLE_HOSTNAME=orasrv01
```
然后进入Oracle安装程序目录，执行如下命令进行静默安装。
```bash
./runInstaller  -silent -ignorePrereq -responsefile /home/oracle/qs-oracle/install-soft.rsp 
```
整个安装过程大约持续5分钟，下面是最后的输出信息。
```text
[oracle@orasrv02 database]$ ./runInstaller  -silent -ignorePrereq -responsefile /home/oracle/qs-oracle/install-soft.rsp 
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 500 MB.   Actual 33494 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 3906 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2019-08-23_11-22-28AM. Please wait ...[oracle@orasrv02 database]$ You can find the log of this install session at:
 /u01/app/oraInventory/logs/installActions2019-08-23_11-22-28AM.log
 The installation of Oracle Database 12c was successful.
Please check '/u01/app/oraInventory/logs/silentInstall2019-08-23_11-22-28AM.log' for more details.

As a root user, execute the following script(s):
	1. /u01/app/oraInventory/orainstRoot.sh
	2. /u01/app/oracle/product/12.1.0/dbhome_1/root.sh
Successfully Setup Software.
 ```
按上面输出信息，切换到root用户，执行如下命令。
```bash
/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/12.1.0/dbhome_1/root.sh
```
下面是命令的输出信息。
```text
[root@orasrv02 ~]# /u01/app/oraInventory/orainstRoot.sh
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/app/oraInventory to oinstall.
The execution of the script is complete.
[root@orasrv02 ~]# /u01/app/oracle/product/12.1.0/dbhome_1/root.sh
Check /u01/app/oracle/product/12.1.0/dbhome_1/install/root_orasrv02_2019-08-23_11-27-38.log for the output of root script
[root@orasrv02 ~]# 
```

修改oracle环境的环境变量
```bash
cat <<EOF >> /home/oracle/.bash_profile 
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
export ORACLE_SID=orcl
export PATH=/usr/sbin:$PATH
export PATH=/u01/app/oracle/product/12.1.0/dbhome_1/bin:$PATH
export LD_LIBRARY_PATH=/u01/app/oracle/product/12.1.0/dbhome_1/lib:/lib:/usr/lib
export CLASSPATH=/u01/app/oracle/product/12.1.0/dbhome_1/jlib:/u01/app/oracle/product/12.1.0/dbhome_1/rdbms/jlib
EOF
```
安装完成后，执行如下命令，表明Oracle软件安装成功。
```
[oracle@orasrv01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 11:59:34 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> 

```

# 2.2 创建Oracle数据库

文件/home/oracle/qs-oracle/dbca-orcl.rsp是创建Oracle数据库的参数配置文件，该文件有如下重要参数可能需要调整。

```
GDBNAME = "orcl"
SID = "orcl"
SYSPASSWORD = "Passw0rd"
SYSTEMPASSWORD = "Passw0rd"
```
运行dbca创建数据库orcl。
```
dbca -silent -responseFile  dbca-orcl.rsp
```
显示信息如下：
```text
[oracle@orasrv01 qs-oracle]$ dbca -silent -responseFile  dbca-orcl.rsp
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/orcl/orcl.log" for further details.
```
通过执行如下命令，表明数据库安装成功。
```
[oracle@orasrv01 qs-oracle]$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 12:13:51 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> select status from v$instance;

STATUS
------------
OPEN
```

下面是创建一个Oracle用户user001，并授权，然后用user001创建表并增加记录的过程，验证数据库能正常访问。
```
[oracle@orasrv01 qs-oracle]$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 12:15:24 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> create user user001 identified by user001;

User created.

SQL> grant connect, dba to user001;

Grant succeeded.

SQL> create table t001(id number(10), name varchar2(10));

Table created.

SQL> insert into t001 values(1, 'name001');

1 row created.

SQL> commit;

Commit complete.

SQL> select * from t001;  

	ID NAME
---------- ----------
	 1 name001

SQL> 
```

# 3 安装Oracle备服务器(Standby Server)

在备服务器上，只需安装Oracle，不用安装数据库。文件/home/oracle/qs-oracle/install-soft-sb.rsp是安装Oracle的配置文件，该文件有如下重要参数可能需要调整：
```
ORACLE_HOSTNAME=orasrv02
```
然后进入Oracle安装程序目录，执行如下命令进行静默安装。
```bash
./runInstaller  -silent -ignorePrereq -responsefile /home/oracle/qs-oracle/install-soft-sb.rsp 
```
后续过程和"2.1 安装Oracle软件"相同。

# 4. 配置Data Guard

## 4.1 配置主、备服务器的listener.ora
主服务器的/u01/app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora内容如下：
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv01)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl_DGMGRL)
      (ORACLE_HOME = /u01/app/oracle/product/12.1.0/dbhome_1)
      (SID_NAME = orcl)
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle
```
备服务器的/u01/app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora内容如下：
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv02)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl_stby_DGMGRL)
      (ORACLE_HOME = /u01/app/oracle/product/12.1.0/dbhome_1)
      (SID_NAME = orcl)
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle
```
然后启动listener。
```bash
lsnrctl stop
lsnrctl start
```
修改主、备服务器上的/u01/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora，内容如下：
```
orcl =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv01)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = orcl)
    )
  )

orcl_stby =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv02)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SID = orcl)
    )
  )
```
最后通过tnsping命令验证配置是否成功。
```
[oracle@orasrv02 ~]$ tnsping orcl

TNS Ping Utility for Linux: Version 12.1.0.2.0 - Production on 23-AUG-2019 15:24:23

Copyright (c) 1997, 2014, Oracle.  All rights reserved.

Used parameter files:


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv01)(PORT = 1521))) (CONNECT_DATA = (SID = orcl)))
OK (0 msec)
[oracle@orasrv02 ~]$ tnsping orcl_stby

TNS Ping Utility for Linux: Version 12.1.0.2.0 - Production on 23-AUG-2019 15:24:33

Copyright (c) 1997, 2014, Oracle.  All rights reserved.

Used parameter files:


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = orasrv02)(PORT = 1521))) (CONNECT_DATA = (SID = orcl)))
OK (0 msec)
[oracle@orasrv02 ~]$ 

```
## 4.2 配置主服务器
检查主服务器是否处于归档模式。
```
SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
NOARCHIVELOG
```
发现数据库不处于归档模式，执行如下命令启用。
```oracle-sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
```
启用强制日志模式：
```oracle-sql
ALTER DATABASE FORCE LOGGING;
ALTER SYSTEM SWITCH LOGFILE;
```
在主服务器上创建备用redo日志。
```oracle-sql
ALTER DATABASE ADD STANDBY LOGFILE SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE SIZE 50M;
ALTER DATABASE ADD STANDBY LOGFILE SIZE 50M;
```
启用flashback数据库，并设置自动管理备份文件。
```oracle-sql
ALTER DATABASE FLASHBACK ON;
ALTER SYSTEM SET STANDBY_FILE_MANAGEMENT=AUTO;

```
## 4.2 配置备服务器

### 4.2.1 准备复制

为备服务器创建参数文件/tmp/initorcl_stby.ora，内容如下：
```
*.db_name='orcl'
```
创建必要的目录：
```bash
mkdir -p /u01/app/oracle/oradata/orcl/pdbseed
mkdir -p /u01/app/oracle/oradata/orcl/pdb1
mkdir -p /u01/app/oracle/fast_recovery_area/orcl
mkdir -p /u01/app/oracle/admin/orcl/adump
```
为备服务器创建password文件，中SYS用户的password必须和主服务器设置的SYS用户password相同。
```bash
orapwd file=/u01/app/oracle/product/12.1.0/dbhome_1/dbs/orapworcl password=Passw0rd entries=10
```
### 4.2.2 通过复制创建备数据库

利用临时的配置文件启动数据库。
```text
$ sqlplus / as sysdba

SQL> STARTUP NOMOUNT PFILE='/tmp/initorcl_stby.ora';
```

运行rman备份管理工具。
```bash
rman TARGET sys/Passw0rd@orcl AUXILIARY sys/Passw0rd@orcl_stby
```
输出信息如下：
```
[oracle@orasrv02 ~]$ rman TARGET sys/Passw0rd@orcl AUXILIARY sys/Passw0rd@orcl_stby

Recovery Manager: Release 12.1.0.2.0 - Production on Fri Aug 23 13:17:35 2019

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCL (DBID=1544838615)
connected to auxiliary database: ORCL (not mounted)

RMAN> 
```
然后执行DUPLICATE命令。
```oracle-sql
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  SPFILE
    SET db_unique_name='orcl_stby' COMMENT 'Is standby'
  NOFILENAMECHECK;
```
命令执行完成最后将显示如下信息：
```text
archived log for thread 1 with sequence 6 is already on disk as file /u01/app/oracle/fast_recovery_area/ORCL_STBY/archivelog/2019_08_23/o1_mf_1_6_goyxv6vg_.arc
archived log for thread 1 with sequence 7 is already on disk as file /u01/app/oracle/fast_recovery_area/ORCL_STBY/archivelog/2019_08_23/o1_mf_1_7_goyxv7x2_.arc
archived log file name=/u01/app/oracle/fast_recovery_area/ORCL_STBY/archivelog/2019_08_23/o1_mf_1_6_goyxv6vg_.arc thread=1 sequence=6
archived log file name=/u01/app/oracle/fast_recovery_area/ORCL_STBY/archivelog/2019_08_23/o1_mf_1_7_goyxv7x2_.arc thread=1 sequence=7
media recovery complete, elapsed time: 00:00:00
Finished recover at 23-AUG-19
Finished Duplicate Db at 23-AUG-19
RMAN> 
```
执行sqlplus, 验证备库是否创建成功。
```
[oracle@orasrv02 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 13:25:23 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> select status from v$instance;

STATUS
------------
MOUNTED

SQL> show parameter db_name

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_name 			     string	 orcl
SQL> show parameter db_unique_name

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_unique_name			     string	 orcl_stby
SQL> 
```

# 4.3 启用Data Guard Broker
通过sqlplus，在主、备服务器上启用Data Guard Broker。
```
ALTER SYSTEM SET dg_broker_start=true;
```
然后检查是否启用。
```
SQL> show parameter dg_broker_start;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
dg_broker_start 		     boolean	 TRUE
SQL> 
```
然后重新启动Oracle Listener，并检查启动状态。在下面输出中，看到orcl_stby_DGB部署，表示Broker已经启动成功。
```
[oracle@orasrv02 ~]$ lsnrctl reload

LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 23-AUG-2019 14:13:59

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=orasrv02)(PORT=1521)))
The command completed successfully
[oracle@orasrv02 ~]$ lsnrctl services

LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 23-AUG-2019 14:14:02

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=orasrv02)(PORT=1521)))
Services Summary...
Service "orclXDB" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
    Handler(s):
      "D000" established:0 refused:0 current:0 max:1022 state:ready
         DISPATCHER <machine: orasrv02, pid: 11109>
         (ADDRESS=(PROTOCOL=tcp)(HOST=orasrv02)(PORT=15540))
Service "orcl_stby" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
    Handler(s):
      "DEDICATED" established:0 refused:0 state:ready
         LOCAL SERVER
Service "orcl_stby_DGB" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
    Handler(s):
      "DEDICATED" established:0 refused:0 state:ready
         LOCAL SERVER
Service "orcl_stby_DGMGRL" has 1 instance(s).
  Instance "orcl", status UNKNOWN, has 1 handler(s) for this service...
    Handler(s):
      "DEDICATED" established:0 refused:0
         LOCAL SERVER
The command completed successfully
[oracle@orasrv02 ~]$ 
```

在主服务器上，把orcl数据库注册为主数据库。
```
[oracle@orasrv01 ~]$ dgmgrl sys/Passw0rd@orcl
DGMGRL for Linux: Version 12.1.0.2.0 - 64bit Production

Copyright (c) 2000, 2013, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected as SYSDBA.
DGMGRL> CREATE CONFIGURATION my_dg_config AS PRIMARY DATABASE IS orcl CONNECT IDENTIFIER IS orcl;
Configuration "my_dg_config" created with primary database "orcl"
DGMGRL>
```
增加备数据库，并启用配置信息。
```
DGMGRL> ADD DATABASE orcl_stby AS CONNECT IDENTIFIER IS orcl_stby MAINTAINED AS PHYSICAL;
Database "orcl_stby" added
DGMGRL> ENABLE CONFIGURATION;
Enabled.
DGMGRL>
```
执行如下命令检查当前的配置信息。
```
DGMGRL> SHOW CONFIGURATION;

Configuration - my_dg_config

  Protection Mode: MaxPerformance
  Members:
  orcl      - Primary database
    orcl_stby - Physical standby database 

Fast-Start Failover: DISABLED

Configuration Status:
SUCCESS   (status updated 26 seconds ago)

DGMGRL> SHOW DATABASE orcl;

Database - orcl

  Role:               PRIMARY
  Intended State:     TRANSPORT-ON
  Instance(s):
    orcl

Database Status:
SUCCESS

DGMGRL> SHOW DATABASE orcl_stby;

Database - orcl_stby

  Role:               PHYSICAL STANDBY
  Intended State:     APPLY-ON
  Transport Lag:      0 seconds (computed 0 seconds ago)
  Apply Lag:          0 seconds (computed 0 seconds ago)
  Average Apply Rate: 4.00 KByte/s
  Real Time Query:    ON
  Instance(s):
    orcl

Database Status:
SUCCESS
DGMGRL> 
```

# 4.4 测试复制是否成功
连接主服务器，创建表t002，并插入数据。
```
[oracle@orasrv01 ~]$ sqlplus user001/user001@orcl

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 14:21:17 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Last Successful login time: Fri Aug 23 2019 13:30:00 +08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> create table t002(id number(10), name varchar2(10));

Table created.

SQL> insert into t002 values(1, 'name001');

1 row created.

SQL> commit;

Commit complete.

```
连接到备服务器，发现表中已经有主服务器中插入的数据。但如果直接对备库执行数据修改操作，则提示read-only提示。
```
oracle@orasrv02 ~]$ sqlplus user001/user001@orcl_stby

SQL*Plus: Release 12.1.0.2.0 Production on Fri Aug 23 14:21:47 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Last Successful login time: Fri Aug 23 2019 14:21:17 +08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> select * from t002;

	ID NAME
---------- ----------
	 1 name001

SQL> 

SQL> insert into t002 values(2, 'name002');
insert into t002 values(2, 'name002')
            *
ERROR at line 1:
ORA-16000: database or pluggable database open for read-only access

```

# 5. Oracle数据备份，并归档到金山云对象存储
在备数据库上，首先运行rman进行数据库备份。
```
[oracle@orasrv02 ~]$ rman target / 

Recovery Manager: Release 12.1.0.2.0 - Production on Fri Aug 23 14:35:23 2019

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCL (DBID=1544838615)

RMAN> backup as compressed backupset database;

Starting backup at 23-AUG-19
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=26 device type=DISK
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00001 name=/u01/app/oracle/oradata/orcl/system01.dbf
input datafile file number=00003 name=/u01/app/oracle/oradata/orcl/sysaux01.dbf
input datafile file number=00004 name=/u01/app/oracle/oradata/orcl/undotbs01.dbf
input datafile file number=00006 name=/u01/app/oracle/oradata/orcl/users01.dbf
channel ORA_DISK_1: starting piece 1 at 23-AUG-19
channel ORA_DISK_1: finished piece 1 at 23-AUG-19
piece handle=/u01/app/oracle/fast_recovery_area/ORCL_STBY/backupset/2019_08_23/o1_mf_nnndf_TAG20190823T143534_goz29pqc_.bkp tag=TAG20190823T143534 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:35
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
including current control file in backup set
including current SPFILE in backup set
channel ORA_DISK_1: starting piece 1 at 23-AUG-19
channel ORA_DISK_1: finished piece 1 at 23-AUG-19
piece handle=/u01/app/oracle/fast_recovery_area/ORCL_STBY/backupset/2019_08_23/o1_mf_ncsnf_TAG20190823T143534_goz2btts_.bkp tag=TAG20190823T143534 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
Finished backup at 23-AUG-19
RMAN> 
```
上输出得知，备份文件位于/u01/app/oracle/fast_recovery_area/ORCL_STBY/backupset/2019_08_23/目录。
然后利用金山云对象存储数据迁移工具KS3Up-tool，可快速把备份数据导入对对象存储中，从而实现大量历史数据对归档备份。详细信息参考：[https://docs.ksyun.com/documents/895](https://docs.ksyun.com/documents/895)。

#6. 小结
本文介绍了如何安装Oracle数据库，并利用Oracle Data Guard实现数据库服务器高可用的过程。金山云通过其云服务器、物理主机等服务器，帮助客户实现基于Oracle数据库的应用上云，或者建立云灾备环境。



```text


oracle@dbsrv002 ~]$ dgmgrl /
DGMGRL for Linux: Version 12.1.0.2.0 - 64bit Production

Copyright (c) 2000, 2013, Oracle. All rights reserved.

Welcome to DGMGRL, type "help" for information.
Connected as SYSDG.
DGMGRL> show configuration
ORA-16525: The Oracle Data Guard broker is not yet available.

Configuration details cannot be determined by DGMGRL
DGMGRL> 


Oracle RMAN Backup and Restore

You can use the Oracle Recovery Manager (RMAN) to back up your data, send the backup files to AWS through AWS Snowball, or by using VPN or AWS Direct Connect, and restore your database on AWS.

For more information about Oracle RMAN, see the Oracle documentation.
Oracle Data Pump

You can use Oracle Data Pump to perform network export/import operations, or send your dump file to the Oracle machines or to Amazon S3 for import operation.

For more information about Oracle Data Pump, see the Oracle documentation. 



BACKUP DATABASE
FORMAT '/u01/oraclebck/%U'
TAG TESTDB
KEEP UNTIL TIME 'SYSDATE+1'
RESTORE POINT TESTDB06;


RMAN> BACKUP DATABASE
FORMAT '/u01/oraclebck/%U'
TAG TESTDB
KEEP UNTIL TIME 'SYSDATE+1'
RESTORE POINT TESTDB06;2> 3> 4> 5> 

Starting backup at 20-AUG-19

using channel ORA_DISK_1
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================
RMAN-03002: failure of backup command at 08/20/2019 13:30:45
RMAN-06149: cannot BACKUP DATABASE in NOARCHIVELOG mode

RMAN> 


CONFIGURE CHANNEL DEVICE TYPE DISK FORMAT '/u01/oraclebck/full_%u_%s_%p';


CREATE TABLESPACE USER_DATA 
DATAFILE'$ORACLE_BASE/oradata/orcl/user_data01.dbf'  
SIZE 1024M AUTOEXTEND ON NEXT 100M EXTENT MANAGEMENT LOCAL UNIFORM SIZE 
256K; 

alter tablespace USER_DATA add datafile '$ORACLE_BASE/oradata/orcl/user_data02.dbf'  
SIZE 1024M AUTOEXTEND ON NEXT 100M maxsize 30G; 
alter tablespace USER_DATA add datafile '$ORACLE_BASE/oradata/orcl/user_data03.dbf'  
SIZE 1024M AUTOEXTEND ON NEXT 100M maxsize 30G; 

CREATE USER c##udadmin IDENTIFIED BY test1234 DEFAULT TABLESPACE USER_DATA 
TEMPORARY TABLESPACE temp; 
GRANT connect,dba to c##udadmin; 


sqlplus c##udadmin/test1234 as sysdba 

create user

create table dept(  
       deptno number(3) primary key,  
       dname varchar2(10),  
       loc varchar2(13)   
       );  
       
              
insert into  t001 values(1, 'name001');
       
insert into  dept values(1, 'name001', 'loc001');
insert into  dept values(2, 'name002', 'loc002');




create table TestTable as  
select rownum as id, 
               to_char(sysdate + rownum/24/3600, 'yyyy-mm-dd hh24:mi:ss') as inc_datetime, 
               trunc(dbms_random.value(0, 100)) as random_id, 
               dbms_random.string('x', 20) random_string 
          from dual 
        connect by level <= 100; 


RMAN> backup as compressed backupset database;

Starting backup at 20-AUG-19
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=13 device type=DISK
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00012 name=/u01/app/oracle/oradata/orcl/user_data01.dbf
input datafile file number=00013 name=/u01/app/oracle/oradata/orcl/user_data02.dbf
input datafile file number=00014 name=/u01/app/oracle/oradata/orcl/user_data03.dbf
input datafile file number=00001 name=/u01/app/oracle/oradata/orcl/system01.dbf
input datafile file number=00003 name=/u01/app/oracle/oradata/orcl/sysaux01.dbf
input datafile file number=00004 name=/u01/app/oracle/oradata/orcl/undotbs01.dbf
input datafile file number=00006 name=/u01/app/oracle/oradata/orcl/users01.dbf
channel ORA_DISK_1: starting piece 1 at 20-AUG-19
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 08/20/2019 13:19:40
ORA-19809: limit exceeded for recovery files
ORA-19804: cannot reclaim 67108864 bytes disk space from 4781506560 limit
continuing other job steps, job failed will not be re-run
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00011 name=/u01/app/oracle/oradata/orcl/pdborcl/example01.dbf
input datafile file number=00009 name=/u01/app/oracle/oradata/orcl/pdborcl/sysaux01.dbf
input datafile file number=00008 name=/u01/app/oracle/oradata/orcl/pdborcl/system01.dbf
input datafile file number=00010 name=/u01/app/oracle/oradata/orcl/pdborcl/SAMPLE_SCHEMA_users01.dbf
channel ORA_DISK_1: starting piece 1 at 20-AUG-19
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 08/20/2019 13:19:41
ORA-19809: limit exceeded for recovery files
ORA-19804: cannot reclaim 67108864 bytes disk space from 4781506560 limit
continuing other job steps, job failed will not be re-run
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00007 name=/u01/app/oracle/oradata/orcl/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/u01/app/oracle/oradata/orcl/pdbseed/system01.dbf
channel ORA_DISK_1: starting piece 1 at 20-AUG-19
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 08/20/2019 13:19:42
ORA-19809: limit exceeded for recovery files
ORA-19804: cannot reclaim 67108864 bytes disk space from 4781506560 limit

RMAN> 

```

SQL> startup open
ORACLE instance started.

Total System Global Area 1593835520 bytes
Fixed Size		    2924880 bytes
Variable Size		 1023413936 bytes
Database Buffers	  553648128 bytes
Redo Buffers		   13848576 bytes
Database mounted.
Database opened.
SQL> create user c##user001 identified by user001;

User created.

SQL> grant connect,dba to c##user001;

Grant succeeded.

SQL> 



[oracle@dbsrv002 ~]$ sqlplus c##user001/user001@orcl

SQL*Plus: Release 12.1.0.2.0 Production on Mon Aug 19 18:54:03 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> 


# 安装VNC server


yum install tigervnc-server -y



[oracle@db-srv ~]$ netca -silent -responseFile ~/netca.rsp 

Parsing command line arguments:
    Parameter "silent" = true
    Parameter "responsefile" = /home/oracle/netca.rsp
Done parsing command line arguments.
Oracle Net Services Configuration:
Profile configuration complete.
Oracle Net Listener Startup:
    Running Listener Control: 
      /u01/app/oracle/product/12.1.0/db_1/bin/lsnrctl start LISTENER
    Listener Control complete.
    Listener started successfully.
Listener configuration complete.
Oracle Net Services configuration successful. The exit code is 0
[oracle@db-srv ~]$ 



[oracle@db-srv ~]$ more /u01/app/oracle/cfgtoollogs/dbca/ORAC12C.log

The Oracle system identifier(SID) "orcl12c" already exists. Specify another SID.

/u01/ has enough space. Required space is 7665 MB , available space is 79532 MB.
File Validations Successful.
[oracle@db-srv ~]$ more /etc/oratab
#



https://confluence.atlassian.com/doc/confluence-5-9-5-release-notes-803603792.html?_ga=2.267466382.1691481261.1566188397-2125458091.1566188397


# This file is used by ORACLE utilities.  It is created by root.sh
# and updated by either Database Configuration Assistant while creating
# a database or ASM Configuration Assistant while creating ASM instance.

# A colon, ':', is used as the field terminator.  A new line terminates
# the entry.  Lines beginning with a pound sign, '#', are comments.
#
# Entries are of the form:
#   $ORACLE_SID:$ORACLE_HOME:<N|Y>:
#
# The first and second fields are the system identifier and home
# directory of the database respectively.  The third field indicates
# to the dbstart utility that the database should , "Y", or should not,
# "N", be brought up at system boot time.
#
# Multiple entries with the same $ORACLE_SID are not allowed.
#
#
orcl12c:/u01/app/oracle/product/12.1.0/db_1:N
[oracle@db-srv ~]$ 


ORACLE instance shut down.
SQL> startup mount
ORACLE instance started.

Total System Global Area 1073741824 bytes
Fixed Size		    2932632 bytes
Variable Size		  713031784 bytes
Database Buffers	  352321536 bytes
Redo Buffers		    5455872 bytes
ORA-01102: cannot mount database in EXCLUSIVE mode


SQL> 


create user user001 identified by “user001” temporary tablespace temp default tablespace users;


[root@db-srv dbs]# pwd
/u01/app/oracle/product/12.1.0/db_1/dbs
[root@db-srv dbs]# ls
hc_ORA12C.dat   init.ora        lkORAC12C     spfileorcl12c.ora
hc_orcl12c.dat  initORA12C.ora  orapworcl12c
[root@db-srv dbs]# 




# 1 准备环境



oracle@db-srv ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.1.0.2.0 Production on Mon Aug 19 12:12:57 2019

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORA-01078: failure in processing system parameters
LRM-00109: could not open parameter file '/u01/app/oracle/product/12.1.0/db_1/dbs/initORA12C.ora'


[oracle@db-srv ~]$ cp /u01/app/oracle/admin/ORAC12C/pfile/init.ora.7192019115012 /u01/app/oracle/product/12.1.0/db_1/dbs/initORA12C.ora
[oracle@db-srv ~]$ 





[root@db-srv pfile]# vi init.ora.7192019115012 
[root@db-srv pfile]# pwd
/u01/app/oracle/admin/ORAC12C/pfile



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


[oracle@ora-srv database]$ ./runInstaller -silent -responseFile ~/db_install.rsp 
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 500 MB.   Actual 11128 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 7812 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2019-08-16_06-11-17PM. Please wait ...[oracle@ora-srv database]$ [FATAL] [INS-32021] Insufficient disk space on this volume for the selected Oracle home.
   CAUSE: The selected Oracle home was on a volume without enough disk space.
   ACTION: Choose a location for Oracle home that has enough space (minimum of 6,553MB) or free up space on the existing volume.
A log of this session is currently saved as: /tmp/OraInstall2019-08-16_06-11-17PM/installActions2019-08-16_06-11-17PM.log. Oracle recommends that if you want to keep this log, you should move it from the temporary location.



INFO: Adding ExitStatus VAR_VALIDATION_FAILURE to the exit status set
WARNING: A log of this session is currently saved as: /tmp/OraInstall2019-08-16_06-04-35PM/installActions2019-08-16_06-04-35PM.log. Oracle recommends that if you want to keep this log, you should move it from the temporary location.
INFO: Finding the most appropriate exit status for the current application
INFO: Exit Status is -2
INFO: Shutdown Oracle Database 12c Release 1 Installer

#------------------------------------------------------------------------------
DECLINE_SECURITY_UPDATES=true


确认操作系统版本

```text
[root@vm10-34-0-7 ~]# uname -r
3.10.0-862.14.4.el7.x86_64
```



下载文件
 ```bash

wget https://ks3-cn-beijing.ksyun.com/ks3-tools/ks3util-1.1.3-dist.zip
unzip ks3util-1.1.3-dist.zip 
PATH=$PATH:$HOME/bin:/root/ks3util-1.1.3/bin
chmod +x /root/ks3util-1.1.3/bin/ks3util
```

修改.ks3utilconfig
```properties
ks3.ak=xxx
ks3.sk=xxx
ks3.endpoint=ks3-cn-beijing-internal.ksyun.com
ks3.protocol=http
```
ks3util ls -b ksc-sbt-software -k list.txt
ks3util multi-get -b  ksc-sbt-software -f fail.txt -k list.txt -s .

 




3.6 Installing Oracle Linux with Oracle Linux Yum Server Support

Use the following procedure to install Oracle Linux and configure your Linux installation for security errata or bug fix updates using the Oracle Linux yum server:

    Obtain Oracle Linux DVDs from Oracle Store, or download Oracle Linux from the Oracle Software Delivery Cloud:

    Oracle Store:

    https://shop.oracle.com

    Oracle Software Delivery Cloud website:

    https://edelivery.oracle.com/linux

    Install Oracle Linux from the ISO or DVD image.

    Log in as root

    Download the yum repository file for your Linux distribution from http://yum.oracle.com/, using the instructions you can find on the Oracle Linux yum server website. For example:

    # cd /etc/yum.repos.d/
    # wget http://yum.oracle.com/public-yum-ol6.repo

    Ensure that the olrelease_latest file (for example, ol6_latest for Oracle Linux 6) is enabled, as this is the repository that contains the Oracle Preinstallation RPM.

    (Optional) Edit the repo file to enable other repositories. For example, enable the repository ol6_UEK_latest by setting enabled=1 in the file with a text editor.

    Run the command yum repolist to verify the registered channels.

    Start a terminal session and enter the following command as root, depending on your platform. For example:

    Oracle Linux 6 and Oracle Linux 7:

    yum install oracle-rdbms-server-12cR1-preinstall



# 准备安装文件

```bash
wget https://ks3-cn-beijing.ksyun.com/ks3-tools/ks3util-1.1.3-dist.zip
yum install -y java-1.8.0-openjdk.x86_64

 

```




# 1 None Data Guard


物理机： 56核 128G， 1.60TB 
233.33/天

```bash
[root@ymq-epc-oracle_1 ~]# yum install fio.x86_64  -y
```


云服务器配置调整后Oracle系统配置需要进行的调整

* SWAP区调整
* SGA区调整

系统盘：
Windows 50GB
Linux 20GB

本地数据盘，作为共享存储空间
0-100G( 最大48核，96G, 2000GB), 但本地盘不能调整。



0.54元/G/月

/vdb: 作为共享存储空间



    规格
    提供10GB到16000GB的容量，步长为10GB
    性能
    单块SSD云硬盘最高可提供20000随机读写IOPS、256M/s的吞吐量性能。
    
    30*600 = 18000
    
    最大盘600G

类型 	随机IOPS 	吞吐量 	时延 	功能特性
SSD EBS2.0 	IOPS=min{1800+30*容量（GB），20000} 	min{80+0.5*容量（GB），256}MBps 	小于3ms 	不支持快照
SSD EBS3.0 	IOPS=min{1800+30*容量（GB），20000} 	min{80+0.5*容量（GB），256}MBps 	小于2ms 	支持快照

    适用场景
    适用于高IO、高吞吐量的读写密集型的高可靠应用场景，如NoSQL、MongDB、SQL Server等大中型关系型数据库，以及针对TB、PB级别的分布式数据分析，挖掘等场景。


EBS云盘（10G-16,000G)


0.76元/G/月

122.4
 50.4
 
72.GB


/vdc: swap
/vdd: Oracle DATA
/vde: Oracle REDO


```bash
 yum install -y fio.x86_64
```
```text
[root@vm10-34-16-20 ~]# fio --version
fio-3.1
```


Orion for Oracle Administrators
https://docs.oracle.com/database/121/TGDBA/pfgrf_iodesign.htm#TGDBA95226


fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/vdb -name=Rand_Write_Testing
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/vdb

Most Oracle Database production systems in domains such as ERP (Enterprise Resource Planning), CRM (Customer Relationship Management) are in the range of 3,000–30,000 IOPS. Your individual application might have different IOPS requirements. A performance test environment’s IOPS needs are generally identical to those of production environments, but for other test and development environments, the range is usually 200–2,000 IOPS. Some online transaction processing (OLTP) systems use up to 60,000 IOPS. There are Oracle databases that use more than 60,000 IOPS, but that is unusual. If your environment shows numbers outside these parameters, you should complete further analysis to confirm your numbers.


```bash

[root@ymq-epc-oracle_1 ~]# yum install fio -y

[root@vm10-34-16-20 ~]# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/vdc -name=Rand_Write_Testing
Rand_Write_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=78.3MiB/s][r=0,w=20.0k IOPS][eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=10784: Thu Aug 15 12:05:40 2019
  write: IOPS=19.8k, BW=77.3MiB/s (81.1MB/s)(1024MiB/13246msec)
    slat (usec): min=2, max=9617, avg= 4.19, stdev=19.89
    clat (usec): min=412, max=66689, avg=6460.68, stdev=12638.36
     lat (usec): min=418, max=66692, avg=6465.34, stdev=12638.38
    clat percentiles (usec):
     |  1.00th=[  873],  5.00th=[ 1647], 10.00th=[ 1991], 20.00th=[ 2311],
     | 30.00th=[ 2540], 40.00th=[ 2737], 50.00th=[ 2933], 60.00th=[ 3163],
     | 70.00th=[ 3490], 80.00th=[ 4015], 90.00th=[ 5997], 95.00th=[49546],
     | 99.00th=[59507], 99.50th=[61604], 99.90th=[63177], 99.95th=[64226],
     | 99.99th=[65274]
   bw (  KiB/s): min=76024, max=80176, per=100.00%, avg=79868.00, stdev=785.40, samples=26
   iops        : min=19006, max=20044, avg=19967.00, stdev=196.35, samples=26
  lat (usec)   : 500=0.04%, 750=0.51%, 1000=0.90%
  lat (msec)   : 2=8.76%, 4=69.74%, 10=12.38%, 20=0.91%, 50=1.86%
  lat (msec)   : 100=4.89%
  cpu          : usr=4.64%, sys=12.80%, ctx=59263, majf=0, minf=27
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=77.3MiB/s (81.1MB/s), 77.3MiB/s-77.3MiB/s (81.1MB/s-81.1MB/s), io=1024MiB (1074MB), run=13246-13246msec

Disk stats (read/write):
  vdc: ios=42/261328, merge=0/0, ticks=37/1673608, in_queue=1674822, util=99.49%
You have new mail in /var/spool/mail/root
[root@vm10-34-16-20 ~]# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/vdd -name=Rand_Write_Testing
Rand_Write_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=10.2MiB/s][r=0,w=2599 IOPS][eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=10796: Thu Aug 15 12:07:54 2019
  write: IOPS=2401, BW=9607KiB/s (9837kB/s)(1024MiB/109150msec)
    slat (usec): min=2, max=8220, avg= 4.88, stdev=27.40
    clat (usec): min=463, max=333213, avg=53287.46, stdev=47202.90
     lat (usec): min=468, max=333218, avg=53292.82, stdev=47202.48
    clat percentiles (usec):
     |  1.00th=[  1663],  5.00th=[  2114], 10.00th=[  2343], 20.00th=[  2737],
     | 30.00th=[  3130], 40.00th=[  3785], 50.00th=[ 94897], 60.00th=[ 96994],
     | 70.00th=[ 96994], 80.00th=[ 98042], 90.00th=[ 98042], 95.00th=[ 99091],
     | 99.00th=[101188], 99.50th=[102237], 99.90th=[110625], 99.95th=[181404],
     | 99.99th=[333448]
   bw (  KiB/s): min= 8336, max=10888, per=100.00%, avg=9609.58, stdev=132.81, samples=218
   iops        : min= 2084, max= 2722, avg=2402.33, stdev=33.20, samples=218
  lat (usec)   : 500=0.01%, 750=0.05%, 1000=0.11%
  lat (msec)   : 2=3.40%, 4=38.12%, 10=4.58%, 20=0.36%, 50=0.17%
  lat (msec)   : 100=51.15%, 250=2.05%, 500=0.01%
  cpu          : usr=0.63%, sys=1.79%, ctx=62184, majf=0, minf=27
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=9607KiB/s (9837kB/s), 9607KiB/s-9607KiB/s (9837kB/s-9837kB/s), io=1024MiB (1074MB), run=109150-109150msec

Disk stats (read/write):
  vdd: ios=40/262136, merge=0/0, ticks=41/13949648, in_queue=13950275, util=100.00%
[root@vm10-34-16-20 ~]# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/vdb -name=Rand_Write_Testing
Rand_Write_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=187MiB/s][r=0,w=47.9k IOPS][eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=11071: Thu Aug 15 12:09:01 2019
  write: IOPS=46.2k, BW=181MiB/s (189MB/s)(1024MiB/5670msec)
    slat (usec): min=2, max=164, avg= 4.24, stdev= 2.76
    clat (usec): min=841, max=84224, avg=2761.25, stdev=1519.69
     lat (usec): min=855, max=84229, avg=2765.99, stdev=1519.66
    clat percentiles (usec):
     |  1.00th=[ 1205],  5.00th=[ 1319], 10.00th=[ 1516], 20.00th=[ 1778],
     | 30.00th=[ 2008], 40.00th=[ 2212], 50.00th=[ 2737], 60.00th=[ 3163],
     | 70.00th=[ 3392], 80.00th=[ 3556], 90.00th=[ 3785], 95.00th=[ 4015],
     | 99.00th=[ 5932], 99.50th=[ 7504], 99.90th=[16188], 99.95th=[26870],
     | 99.99th=[57934]
   bw (  KiB/s): min=124784, max=219224, per=99.36%, avg=183751.27, stdev=25360.48, samples=11
   iops        : min=31196, max=54806, avg=45937.73, stdev=6340.13, samples=11
  lat (usec)   : 1000=0.05%
  lat (msec)   : 2=29.62%, 4=65.14%, 10=4.93%, 20=0.18%, 50=0.07%
  lat (msec)   : 100=0.02%
  cpu          : usr=9.79%, sys=28.49%, ctx=19629, majf=0, minf=27
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=181MiB/s (189MB/s), 181MiB/s-181MiB/s (189MB/s-189MB/s), io=1024MiB (1074MB), run=5670-5670msec

Disk stats (read/write):
  vdb: ios=42/261027, merge=0/0, ticks=41/707738, in_queue=708095, util=98.54%

```



Response File 	Description

db_install.rsp
	

Silent installation of Oracle Database 12c

grid_install.rsp
	

Silent installation of Oracle Grid Infrastructure

dbca.rsp
	

Silent installation of Database Configuration Assistant

netca.rsp
	

Silent installation of Oracle Net Configuration Assistant


```bash

./orion -run simple


RION VERSION 11.1.0.7.0

Commandline:
-run simple

This maps to this test:
Test: orion
Small IO size: 8 KB
Large IO size: 1024 KB
IO Types: Small Random IOs, Large Random IOs
Simulated Array Type: CONCAT
Write: 0%
Cache Size: Not Entered
Duration for each Data Point: 60 seconds
Small Columns:,      0
Large Columns:,      0,      1,      2
Total Data Points: 8

Name: /dev/vdb  Size: 21474836480
1 FILEs found.

Maximum Large MBPS=1554.54 @ Small=0 and Large=2
Maximum Small IOPS=19019 @ Small=5 and Large=0
Minimum Small Latency=0.26 @ Small=5 and Large=0


ORION VERSION 11.1.0.7.0

Commandline:
-run simple

This maps to this test:
Test: orion
Small IO size: 8 KB
Large IO size: 1024 KB
IO Types: Small Random IOs, Large Random IOs
Simulated Array Type: CONCAT
Write: 0%
Cache Size: Not Entered
Duration for each Data Point: 60 seconds
Small Columns:,      0
Large Columns:,      0,      1,      2
Total Data Points: 8

Name: /dev/vdc  Size: 751619276800
1 FILEs found.

Maximum Large MBPS=255.88 @ Small=0 and Large=2
Maximum Small IOPS=14741 @ Small=5 and Large=0
Minimum Small Latency=0.33 @ Small=4 and Large=0


ORION VERSION 11.1.0.7.0

Commandline:
-run simple

This maps to this test:
Test: orion
Small IO size: 8 KB
Large IO size: 1024 KB
IO Types: Small Random IOs, Large Random IOs
Simulated Array Type: CONCAT
Write: 0%
Cache Size: Not Entered
Duration for each Data Point: 60 seconds
Small Columns:,      0
Large Columns:,      0,      1,      2
Total Data Points: 8

Name: /dev/vdd  Size: 21474836480
1 FILEs found.

Maximum Large MBPS=90.25 @ Small=0 and Large=1
Maximum Small IOPS=2401 @ Small=4 and Large=0
Minimum Small Latency=0.44 @ Small=1 and Large=0

ORION VERSION 11.1.0.7.0

Commandline:
-run simple

This maps to this test:
Test: orion
Small IO size: 8 KB
Large IO size: 1024 KB
IO Types: Small Random IOs, Large Random IOs
Simulated Array Type: CONCAT
Write: 0%
Cache Size: Not Entered
Duration for each Data Point: 60 seconds
Small Columns:,      0
Large Columns:,      0,      1,      2
Total Data Points: 8

Name: /dev/sdb  Size: 479559942144
1 FILEs found.

Maximum Large MBPS=362.46 @ Small=0 and Large=2
Maximum Small IOPS=22817 @ Small=5 and Large=0
Minimum Small Latency=0.19 @ Small=2 and Large=0
~                                                     

```
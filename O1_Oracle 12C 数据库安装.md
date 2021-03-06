# O1_Oracle安装

- 安装指导 https://docs.oracle.com/en/database/index.html
- 推荐网站 https://oracle-base.com/
- 推荐安装指导 https://oracle-base.com/articles/12c/oracle-db-12cr2-installation-on-oracle-linux-6-and-7

简介：

```wiki
12C版本：
12C 12.1.0.1 
```

内核优化：

```shell
kernel.shmmall 总内存/4
kernel.shmmax  全备内容或者80%
```

配置在线源
```shell
[root@foundation0 isos]# rpm -ivh http://epel.mirrors.arminco.com/7/x86_64/Packages/e/epel-release-7-12.noarch.rpm
```

安装步骤:

```shell
vim /etc/hosts
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.56.107  ol7-122.localdomain  ol7-122

vim /etc/sysctl.conf
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 1073741824   #根据实际情况做修改
kernel.shmmax = 4398046511104  #根据实际情况做修改
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500

/sbin/sysctl -p

vim /etc/security/limits.d/oracle-database-server-12cR2-preinstall.conf
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc    16384
oracle   hard   nproc    16384
oracle   soft   stack    10240
oracle   hard   stack    32768
oracle   hard   memlock    134217728
oracle   soft   memlock    134217728

#安装依赖包
# OL6 and OL7 (RHEL6 and RHEL7)
yum install binutils -y
yum install compat-libcap1 -y
yum install compat-libstdc++-33 -y
yum install compat-libstdc++-33.i686 -y
yum install glibc -y
yum install glibc.i686 -y
yum install glibc-devel -y
yum install glibc-devel.i686 -y
yum install ksh -y
yum install libaio -y
yum install libaio.i686 -y
yum install libaio-devel -y
yum install libaio-devel.i686 -y
yum install libX11 -y
yum install libX11.i686 -y
yum install libXau -y
yum install libXau.i686 -y
yum install libXi -y
yum install libXi.i686 -y
yum install libXtst -y
yum install libXtst.i686 -y
yum install libgcc -y
yum install libgcc.i686 -y
yum install libstdc++ -y
yum install libstdc++.i686 -y
yum install libstdc++-devel -y
yum install libstdc++-devel.i686 -y
yum install libxcb -y
yum install libxcb.i686 -y
yum install make -y
yum install nfs-utils -y
yum install net-tools -y
yum install smartmontools -y
yum install sysstat -y
yum install unixODBC -y
yum install unixODBC-devel -y
yum install gcc-c++ -y
yum install xorg-x11-font* xorg-x11-xauth

#创建用户组
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper

#创建用户
useradd -u 54321 -g oinstall -G dba,oper oracle

#配置密码
passwd oracle

#selinux 关掉
vim /etc/selinux/config
SELINUX=permissive

#防火墙关掉
systemctl stop firewalld
systemctl disable firewalld

# vim /home/oracle/.bash_profile
# Oracle Settings
export TMP=/tmp
export TMPDIR=$TMP

export ORACLE_HOSTNAME=ol7-122.localdomain
export ORACLE_UNQNAME=cdb1
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/12.2.0.1/db_1
export ORACLE_SID=cdb1

export PATH=/usr/sbin:/usr/local/bin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH

export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
```
创建安装数据库软件的目录：
```shell
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01/app
```

已配置vncserver的情况下：

```shell
./runInstaller #安装实例
netca #启动监听
dbca #安装数据库
```
没有配置vncserver的情况下:
```shell
yum install -y tigervnc-server tigervnc
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
vim /etc/systemd/system/vncserver@:1.service
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
ExecStart=/usr/sbin/runuser -l oracle -c "/usr/bin/vncserver %i"
PIDFile=/home/oracle/.vnc/%H%i.pid
ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

[Install]
WantedBy=multi-user.target


systemctl daemon-reload
systemctl enable vncserver@:1.service

vncserver
ps -ef|grep vnc
export DISPLAY=:1
xdpyinfo | grep name
printenv | grep DIS  #查看当前DISPLAY信息
xhost + 

su - oracle
export DISPLAY=:1
cd /u01/soft/database
./runInstaller

[root@foundation0 oracle]# ps -ef | grep java |awk '{print $2}'
16848
17465
17854
20124
20703
20969
21184
21527
21748
22097
22695
23300
23644
25715
26709
[root@foundation0 oracle]# for i in `ps -ef | grep java |awk '{print $2}'`;do kill -9 $i;done
```

swap分区默认分配7G需要拓展
```shell
free -h
dd if=/dev/zero of=/home/swap bs=1024 count=7340032
mkswap /home/swap
swapon /home/swap

[root@foundation0 oracle]# free -h
              total        used        free      shared  buff/cache   available
Mem:            15G        1.8G        6.0G         13M        7.8G         13G
Swap:          7.9G          0B        7.9G
[root@foundation0 oracle]# free -h
              total        used        free      shared  buff/cache   available
Mem:            15G        1.8G        6.0G         13M        7.8G         13G
Swap:          7.9G          0B        7.9G
[root@foundation0 oracle]# dd if=/dev/zero of=/home/swap bs=1024 count=7340032
7340032+0 records in
7340032+0 records out
7516192768 bytes (7.5 GB) copied, 28.3909 s, 265 MB/s
[root@foundation0 oracle]# mkswap /home/swap
Setting up swapspace version 1, size = 7340028 KiB
no label, UUID=d280e8ee-a22c-4669-9853-c10f81e0e580
[root@foundation0 oracle]# swapon /home/swap
swapon: /home/swap: insecure permissions 0644, 0600 suggested.
[root@foundation0 oracle]# free -h
              total        used        free      shared  buff/cache   available
Mem:            15G        1.8G        855M         13M         12G         13G
Swap:           14G          0B         14G

```

执行配置脚本：
```shell
[root@foundation0 oracle]# /u01/app/oraInventory/orainstRoot.sh
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/app/oraInventory to oinstall.
The execution of the script is complete.

[root@foundation0 oracle]# /u01/app/oracle/product/12.2.0.1/db_1/root.sh 
Performing root user operation.

The following environment variables are set as:
    ORACLE_OWNER= oracle
    ORACLE_HOME=  /u01/app/oracle/product/12.2.0.1/db_1

Enter the full pathname of the local bin directory: [/usr/local/bin]: 
   Copying dbhome to /usr/local/bin ...
   Copying oraenv to /usr/local/bin ...
   Copying coraenv to /usr/local/bin ...


Creating /etc/oratab file...
Entries will be added to the /etc/oratab file as needed by
Database Configuration Assistant when a database is created
Finished running generic part of root script.
Now product-specific root actions will be performed.
Do you want to setup Oracle Trace File Analyzer (TFA) now ? yes|[no] : 
ys^H
Oracle Trace File Analyzer (TFA - User Mode) is available at :
    /u01/app/oracle/product/12.2.0.1/db_1/suptools/tfa/release/tfa_home/bin/tfactl

OR

Oracle Trace File Analyzer (TFA - Daemon Mode) can be installed by running this script :
    /u01/app/oracle/product/12.2.0.1/db_1/suptools/tfa/release/tfa_home/install/roottfa.sh

```

远程连接：

```shell
lsnrctl status

[oracle@Oracle01 ~]$ lsnrctl status

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 22-SEP-2019 13:49:06

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=Oracle01)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                22-SEP-2019 13:23:08
Uptime                    0 days 0 hr. 26 min. 1 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/12.2.0.1/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/Oracle01/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=Oracle01)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Services Summary...
Service "931ed7877f0586f5e055288c95e95258" has 1 instance(s).
  Instance "cdb1", status READY, has 1 handler(s) for this service...
Service "cdb1" has 1 instance(s).
  Instance "cdb1", status READY, has 1 handler(s) for this service...
Service "cdb1XDB" has 1 instance(s).
  Instance "cdb1", status READY, has 1 handler(s) for this service...
Service "orclpdb" has 1 instance(s).
  Instance "cdb1", status READY, has 1 handler(s) for this service...
The command completed successfully

```

配置服务名：

```shell
cd $ORACLE_HOME/network/admin
vim tnsnames.ora
LISTENER_CDB1 =
  (ADDRESS = (PROTOCOL = TCP)(HOST = Oracle01)(PORT = 1521))


CDB1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = Oracle01)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = cdb1)
    )
  )

PDB1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = Oracle01)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orclpdb)
    )
  )

```




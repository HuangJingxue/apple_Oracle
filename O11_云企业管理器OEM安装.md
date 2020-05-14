安装注意事项：

- 先安装OCM软件需要的Oracle CDB数据库emcdb和pdb emrep
- 再安装OCM软件

1. 创建目录
```
 mkdir -p /u01/app/oracle/middleware_booboo
 mkdir -p /u01/app/oracle/agent_booboo
 chown -R oracle. /u01/app/oracle/middleware_booboo
 chown -R oracle. /u01/app/oracle/agent_booboo
 ```
 
2. 准备安装软件包
```shell
SOFTWARE_DIR=/u01/software
cd ${SOFTWARE_DIR}
mkdir -p em12cr5
unzip -oqd em12cr5 em12105_linux64_disk1.zip
unzip -oqd em12cr5 em12105_linux64_disk2.zip
unzip -oqd em12cr5 em12105_linux64_disk3.zip
cd em12cr5
```
3. 准备应答文件
```shell
su - oracle

# Set parameters.
ORACLE_BASE=/u01/app/oracle
ORA_INVENTORY=/u01/app/oraInventory
ORACLE_HOSTNAME=${HOSTNAME}
PDB_NAME=emrep
SYS_PASSWORD=WLS3Gg5_2
UNIX_GROUP_NAME=oinstall
MW_HOME=${ORACLE_BASE}/middleware_booboo
OMS_HOME=${MW_HOME}/oms
GC_INST=${ORACLE_BASE}/gc_inst
AGENT_BASE=${ORACLE_BASE}/agent_booboo
WLS_USERNAME=weblogic
WLS_PASSWORD=WLS3Gg5_2
SYSMAN_PASSWORD=${WLS_PASSWORD}
AGENT_PASSWORD=${WLS_PASSWORD}
SOFTWARE_LIBRARY=${ORACLE_BASE}/swlib
DATABASE_HOSTNAME=ocm
LISTENER_PORT=1521
SOFTWARE_DIR=/u01/software

# Create Response file.
cat > /tmp/install.rsp <<EOF
RESPONSEFILE_VERSION=2.2.1.0.0
UNIX_GROUP_NAME=${UNIX_GROUP_NAME}
INVENTORY_LOCATION=${ORA_INVENTORY}
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true
INSTALL_UPDATES_SELECTION=skip
ORACLE_MIDDLEWARE_HOME_LOCATION=${MW_HOME}
ORACLE_HOSTNAME=${ORACLE_HOSTNAME}
AGENT_BASE_DIR=${AGENT_BASE}
WLS_ADMIN_SERVER_USERNAME=${WLS_USERNAME}
WLS_ADMIN_SERVER_PASSWORD=${WLS_PASSWORD}
WLS_ADMIN_SERVER_CONFIRM_PASSWORD=${WLS_PASSWORD}
NODE_MANAGER_PASSWORD=${WLS_PASSWORD}
NODE_MANAGER_CONFIRM_PASSWORD=${WLS_PASSWORD}
ORACLE_INSTANCE_HOME_LOCATION=${GC_INST}
CONFIGURE_ORACLE_SOFTWARE_LIBRARY=true
SOFTWARE_LIBRARY_LOCATION=${SOFTWARE_LIBRARY}
DATABASE_HOSTNAME=${DATABASE_HOSTNAME}
LISTENER_PORT=${LISTENER_PORT}
SERVICENAME_OR_SID=${PDB_NAME}
SYS_PASSWORD=${SYS_PASSWORD}
SYSMAN_PASSWORD=${SYSMAN_PASSWORD}
SYSMAN_CONFIRM_PASSWORD=${SYSMAN_PASSWORD}
DEPLOYMENT_SIZE=SMALL
AGENT_REGISTRATION_PASSWORD=${AGENT_PASSWORD}
AGENT_REGISTRATION_CONFIRM_PASSWORD=${AGENT_PASSWORD}
PLUGIN_SELECTION={}
b_upgrade=false
EM_INSTALL_TYPE=NOSEED
CONFIGURATION_TYPE=LATER
CONFIGURE_SHARED_LOCATION_BIP=false
FROM_LOCATION="${SOFTWARE_DIR}/em12cr5/oms/Disk1/stage/products.xml"
EOF
```
4. 开始静默安装
```shell
./runInstaller -silent -responseFile /tmp/install.rsp -waitforcompletion
具体安装过程

20:28~20:45

[oracle@oracle01 em12cr5]$ ./runInstaller -silent -responseFile /tmp/install.rsp -waitforcompletion
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 400 MB.   Actual 15262 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 1955 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2019-11-02_11-36-50PM. Please wait ...Installer has detected  host name as ol7-122.localdomain, you have changed the host name to oracle01.

Installing ORACLE_HOME /u01/app/oracle/middleware_booboo/jdk16
Installation in progress
................
Install successful
Linking in progress

Link successful
Setup in progress

Setup successful
Installing Oracle WebLogic Server

Installing ORACLE_HOME /u01/app/oracle/middleware_booboo/Oracle_BI1
Installation in progress
.................
Install successful
Linking in progress

Link successful
Setup in progress

Setup successful

Installing ORACLE_HOME /u01/app/oracle/middleware_booboo/oms
Installation in progress
........Installation in progress
..Unable to find product oracle.swd.jre[1.5.0.0.0, 9.9.9] in Oracle Inventory
..
Install successful
Linking in progress
Error in invoking target 'install' of makefile '/u01/app/oracle/middleware_booboo/oms/sqlplus/lib/ins_sqlplus.mk'. See '/u01/app/oraInventory/logs/installActions2019-11-02_11-45-26-PM.log' for details.

Link successful
Setup in progress
.
Setup successful
Copying plug-ins to Management Service Oracle Home
You can find the log of this install session at:
 /u01/app/oraInventory/logs/installActions2019-11-02_11-53-31-PM.log

Installing ORACLE_HOME /u01/app/oracle/agent_booboo/core/12.1.0.5.0
Installation in progress
..........
Install successful
Linking in progress

Link successful
Setup in progress
......
Setup successful
You can find the log of this install session at:
 /u01/app/oraInventory/logs/cloneActions2019-11-02_11-55-46-PM.log

Installing ORACLE_HOME /u01/app/oracle/agent_booboo/sbin
Installation in progress

Install successful
Linking in progress

Link successful
Setup in progress

Setup successful
Extracting WT.zip
You can find the log of this install session at:
 /u01/app/oraInventory/logs/cloneActions2019-11-02_11-55-52-PM.log

Installing ORACLE_HOME /u01/app/oracle/middleware_booboo/oracle_common
Installation in progress
.............
Install successful
Linking in progress

Link successful
Setup in progress
...
Setup successful
You can find the log of this install session at:
 /u01/app/oraInventory/logs/cloneActions2019-11-02_11-59-27-PM.log

Installing ORACLE_HOME /u01/app/oracle/middleware_booboo/Oracle_WT
Installation in progress
................
Install successful
Linking in progress
Error in invoking target 'install' of makefile '/u01/app/oracle/middleware_booboo/Oracle_WT/webcache/lib/ins_calypso.mk'. See '/u01/app/oraInventory/logs/cloneActions2019-11-02_11-59-27-PM.log' for details.

Link successful
Setup in progress

Setup successful
Applying the required oneoff patches.
Warning: The following configuration scripts need to be executed as the "root" user
  /u01/app/oracle/middleware_booboo/oms/allroot.sh
To execute the configuration scripts:
 1. Open a new  terminal window
 2. Login in as "root"
 3. Run the scripts

Enterprise Manager Cloud Control Installation has finished.

```

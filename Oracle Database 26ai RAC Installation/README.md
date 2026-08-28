# Oracle Database 26ai RAC Installation on Oracle Linux 10
## Step 1: Operating System Preparation
Perform the following steps on all nodes.
### Update the Operating System
```bash
sudo dnf update -y
```
### Install Required Packages
```bash
dnf install -y bc
dnf install -y binutils
dnf install -y elfutils-libelf
dnf install -y fontconfig
dnf install -y glibc
dnf install -y glibc-devel
dnf install -y glibc-headers
dnf install -y ksh
dnf install -y libaio
dnf install -y libgcc
dnf install -y libibverbs
dnf install -y libstdc++
dnf install -y libvirt-libs
dnf install -y libxcb
dnf install -y libX11
dnf install -y libXau
dnf install -y libXi
dnf install -y libXrender
dnf install -y libXtst
dnf install -y libxcrypt-compat
dnf install -y make
dnf install -y policycoreutils
dnf install -y policycoreutils-python-utils
dnf install -y smartmontools
dnf install -y sysstat
dnf install -y ipmiutil
dnf install -y net-tools
dnf install -y nfs-utils
dnf install -y libnsl
dnf install -y libnsl2
dnf install -y libnsl2-devel

dnf install -y gcc
dnf install -y unixODBC
```
### Disable Avahi Daemon
```bash
systemctl status avahi-daemon
systemctl stop avahi-daemon
systemctl disable avahi-daemon
```

## Step 2: Create Required Groups and Users
### Create Oracle Groups
```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
groupadd -g 54324 backupdba
groupadd -g 54325 dgdba
groupadd -g 54326 kmdba
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54329 asmadmin
groupadd -g 54330 racdba
```
### Create Users
```bash
useradd -m -u 54321 -g oinstall -G dba,oper,backupdba,dgdba,kmdba,asmdba,asmoper,asmadmin,racdba oracle
useradd -m -u 54331 -g oinstall -G dba,asmadmin,asmdba,asmoper,racdba grid

```
### Set Passwords
```bash
passwd oracle
passwd grid
```
## Step 3: Configure Kernel Parameters
```bash
vi /etc/sysctl.conf

[root@ms-ol-node-01 ~]# cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).

# oracle-ai-database-preinstall-26ai setting for fs.file-max is 6815744
fs.file-max = 6815744

# oracle-ai-database-preinstall-26ai setting for kernel.sem is '250 32000 100 128'
kernel.sem = 250 32000 100 128

# oracle-ai-database-preinstall-26ai setting for kernel.shmmni is 4096
kernel.shmmni = 4096

# oracle-ai-database-preinstall-26ai setting for kernel.shmall is 1073741824 on x86_64
kernel.shmall = 1073741824

# oracle-ai-database-preinstall-26ai setting for kernel.shmmax is 4398046511104 on x86_64
kernel.shmmax = 4398046511104

# oracle-ai-database-preinstall-26ai setting for kernel.panic_on_oops is 1 per Orabug 19212317
kernel.panic_on_oops = 1

# oracle-ai-database-preinstall-26ai setting for net.core.rmem_default is 262144
net.core.rmem_default = 262144

# oracle-ai-database-preinstall-26ai setting for net.core.rmem_max is 4194304
net.core.rmem_max = 4194304

# oracle-ai-database-preinstall-26ai setting for net.core.wmem_default is 262144
net.core.wmem_default = 262144

# oracle-ai-database-preinstall-26ai setting for net.core.wmem_max is 1048576
net.core.wmem_max = 1048576

# oracle-ai-database-preinstall-26ai setting for net.ipv4.conf.all.rp_filter is 2
net.ipv4.conf.all.rp_filter = 2

# oracle-ai-database-preinstall-26ai setting for net.ipv4.conf.default.rp_filter is 2
net.ipv4.conf.default.rp_filter = 2

# oracle-ai-database-preinstall-26ai setting for fs.aio-max-nr is 1048576
fs.aio-max-nr = 1048576

# oracle-ai-database-preinstall-26ai setting for vm.hugetlb_shm_group is gid of primary group of 'oracle' user
vm.hugetlb_shm_group = 54321

# oracle-ai-database-preinstall-26ai setting special parameters BEGIN
# oracle-ai-database-preinstall-26ai setting for kernel.panic is 10
kernel.panic = 10

# oracle-ai-database-preinstall-26ai setting for net.ipv4.ip_local_port_range is 9000 65535
net.ipv4.ip_local_port_range = 9000 65535

# oracle-ai-database-preinstall-26ai setting special parameters END
[root@ms-ol-node-01 ~]#
```
Apply the changes
```bash
sysctl -p
```
## Step 4: Configure Resource Limits
Edit /etc/security/limits.conf
```bash
vi /etc/security/limits.conf

oracle soft nofile 1024
oracle hard nofile 65536
oracle soft nproc 16384
oracle hard nproc 16384
oracle soft stack 10240
oracle hard stack 32768
oracle soft memlock 134217728
oracle hard memlock 134217728

grid soft nofile 1024
grid hard nofile 65536
grid soft nproc 16384
grid hard nproc 16384
grid soft stack 10240
grid hard stack 32768
grid soft memlock 134217728
grid hard memlock 134217728
```

## Step 5: Configure Network (/etc/hosts)
Edit /etc/hosts on all nodes:
```bash
vi /etc/hosts

[root@ms-ol-node-01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

#Public IP
10.xxx.200.111 ms-ol-node-01.localdomain ms-ol-node-01
10.xxx.200.112 ms-ol-node-02.localdomain ms-ol-node-02

#Private IP
192.xxx.60.111 ms-ol-node-01-priv.localdomain ms-ol-node-01-priv
192.xxx.60.112 ms-ol-node-02-priv.localdomain ms-ol-node-02-priv


#VIP IP
10.xxx.200.113 ms-ol-node-01-vip.localdomain ms-ol-node-01-vip
10.xxx.200.114 ms-ol-node-02-vip.localdomain ms-ol-node-02-vip

#scan IP
10.xxx.200.118 ms-ol-node-scan.localdomain ms-ol-node-scan
10.xxx.200.119 ms-ol-node-scan.localdomain ms-ol-node-scan
10.xxx.200.120 ms-ol-node-scan.localdomain ms-ol-node-scan
[root@ms-ol-node-01 ~]#
```
## Step 6: Configure Shared Storage (ASM)
### Create partitions for the disks
```bash
fdisk /dev/sdc

[root@ms-ol-node-01 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   80G  0 disk
├─sda1        8:1    0  600M  0 part /boot/efi
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0 78.4G  0 part
  ├─ol-root 252:0    0 49.4G  0 lvm  /
  └─ol-swap 252:1    0    5G  0 lvm  [SWAP]
sdb           8:16   0   60G  0 disk
└─sdb1        8:17   0   60G  0 part
  └─db-u01  252:2    0   60G  0 lvm  /u01
sdc           8:32   0   50G  0 disk
└─sdc1        8:33   0   50G  0 part
sr0          11:0    1 13.2G  0 rom  /run/media/root/OL-8-10-0-BaseOS-x86_64
[root@ms-ol-node-01 ~]#
[root@ms-ol-node-01 ~]#

```
### Create udev rule
```bash
vi /etc/udev/rules.d/99-oracle-asm.rules
[root@ms-ol-node-01 ~]# cat /etc/udev/rules.d/99-oracle-asm.rules
KERNEL=="sdc1", OWNER="grid", GROUP="asmdba", MODE="0660"
[root@ms-ol-node-01 ~]#
```
### Reload the udev rules
```bash
udevadm control --reload-rules && udevadm trigger

[root@ms-ol-node-01 soft]# udevadm control --reload-rules
[root@ms-ol-node-01 soft]# udevadm trigger
[root@ms-ol-node-01 soft]#
[root@ms-ol-node-01 soft]# ls -l /dev/sdc1
brw-rw----. 1 grid asmdba 8, 33 Aug 27 14:15 /dev/sdc1
[root@ms-ol-node-01 soft]#
```
## Step 7: Create Directory Structure
```bash
mkdir -p /u01/app/26.0.0/grid
mkdir -p /u01/app/grid
mkdir -p /u01/app/oracle
mkdir -p /u01/app/oracle/product/26.0.0/dbhome_1

chown -R grid:oinstall /u01
chown -R oracle:oinstall /u01/app/oracle
chmod -R 775 /u01
```
## Step 8: Configure Environment Variables
```bash
Grid Node1:
vi .bash_profile
export ORACLE_SID=+ASM1
export ORACLE_BASE=/u01/app/grid
export ORACLE_HOME=/u01/app/26.0.0/grid
export PATH=$ORACLE_HOME/bin:$PATH

Grid Node2:
vi .bash_profile
export ORACLE_SID=+ASM2
export ORACLE_BASE=/u01/app/grid
export ORACLE_HOME=/u01/app/26.0.0/grid
export PATH=$ORACLE_HOME/bin:$PATH

Oracle Node1:
vi .bash_profile
export ORACLE_SID=cdb26ai1
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/26.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH

Oracle Node2:
vi .bash_profile
export ORACLE_SID=cdb26ai2
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/26.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

## Step 9: Disable Firewall and SELinux
```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld

sudo setenforce 0
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```
## Step 10: Configure SSH Equivalence
As grid user:
```bash
ssh-keygen -t rsa
ssh-copy-id ms-ol-node-01
ssh-copy-id ms-ol-node-02
```
As oracle user:
```bash
ssh-keygen -t rsa
ssh-copy-id ms-ol-node-01
ssh-copy-id ms-ol-node-02
```
## Step 11: Install Grid Infrastructure
### Run Cluster Verification
```bash
[grid@ms-ol-node-01 grid]$ ./runcluvfy.sh stage -pre crsinst -n ms-ol-node-01,ms-ol-node-02 -verbose
This software is "230" days old. It is a best practice to update the CRS home by downloading and applying the latest release update. Refer to MOS                                                              note 756671.1 for more details.

Initializing ...

Performing following verification checks ...

  Physical Memory ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  9.704GB (1.017534E7KB)    8GB (8388608.0KB)         passed
    ms-ol-node-01  9.704GB (1.017534E7KB)    8GB (8388608.0KB)         passed
  Physical Memory ...PASSED
  Available Physical Memory ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  7.0911GB (7435600.0KB)    50MB (51200.0KB)          passed
    ms-ol-node-01  7.3741GB (7732308.0KB)    50MB (51200.0KB)          passed
  Available Physical Memory ...PASSED
  Swap Size ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  16.9336GB (1.7756152E7KB)  9.704GB (1.017534E7KB)    passed
    ms-ol-node-01  16.9336GB (1.7756152E7KB)  9.704GB (1.017534E7KB)    passed
  Swap Size ...PASSED
  Free Space: ms-ol-node-02:/usr,ms-ol-node-02:/var,ms-ol-node-02:/etc,ms-ol-node-02:/sbin,ms-ol-node-02:/tmp ...
    Path              Node Name     Mount point   Available     Required      Status
    ----------------  ------------  ------------  ------------  ------------  ------------
    /usr              ms-ol-node-02  /             27.4541GB     25MB          passed
    /var              ms-ol-node-02  /             27.4541GB     5MB           passed
    /etc              ms-ol-node-02  /             27.4541GB     25MB          passed
    /sbin             ms-ol-node-02  /             27.4541GB     10MB          passed
    /tmp              ms-ol-node-02  /             27.4541GB     1GB           passed
  Free Space: ms-ol-node-02:/usr,ms-ol-node-02:/var,ms-ol-node-02:/etc,ms-ol-node-02:/sbin,ms-ol-node-02:/tmp ...PASSED
  Free Space: ms-ol-node-01:/usr,ms-ol-node-01:/var,ms-ol-node-01:/etc,ms-ol-node-01:/sbin,ms-ol-node-01:/tmp ...
    Path              Node Name     Mount point   Available     Required      Status
    ----------------  ------------  ------------  ------------  ------------  ------------
    /usr              ms-ol-node-01  /             24.3516GB     25MB          passed
    /var              ms-ol-node-01  /             24.3516GB     5MB           passed
    /etc              ms-ol-node-01  /             24.3516GB     25MB          passed
    /sbin             ms-ol-node-01  /             24.3516GB     10MB          passed
    /tmp              ms-ol-node-01  /             24.3516GB     1GB           passed
  Free Space: ms-ol-node-01:/usr,ms-ol-node-01:/var,ms-ol-node-01:/etc,ms-ol-node-01:/sbin,ms-ol-node-01:/tmp ...PASSED
  User Existence: grid ...
    Node Name     Status                    Comment
    ------------  ------------------------  ------------------------
    ms-ol-node-02  passed                    exists(54331)
    ms-ol-node-01  passed                    exists(54331)

    Users With Same UID: 54331 ...PASSED
  User Existence: grid ...PASSED
  Group Existence: asmadmin ...
    Node Name     Status                    Comment
    ------------  ------------------------  ------------------------
    ms-ol-node-02  passed                    exists
    ms-ol-node-01  passed                    exists
  Group Existence: asmadmin ...PASSED
  Group Existence: asmdba ...
    Node Name     Status                    Comment
    ------------  ------------------------  ------------------------
    ms-ol-node-02  passed                    exists
    ms-ol-node-01  passed                    exists
  Group Existence: asmdba ...PASSED
  Group Existence: oinstall ...
    Node Name     Status                    Comment
    ------------  ------------------------  ------------------------
    ms-ol-node-02  passed                    exists
    ms-ol-node-01  passed                    exists
  Group Existence: oinstall ...PASSED
  Group Membership: asmdba ...
    Node Name         User Exists   Group Exists  User in Group  Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     yes           yes           yes           passed
    ms-ol-node-01     yes           yes           yes           passed
  Group Membership: asmdba ...PASSED
  Group Membership: asmadmin ...
    Node Name         User Exists   Group Exists  User in Group  Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     yes           yes           yes           passed
    ms-ol-node-01     yes           yes           yes           passed
  Group Membership: asmadmin ...PASSED
  Group Membership: oinstall(Primary) ...
    Node Name         User Exists   Group Exists  User in Group  Primary       Status
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-02     yes           yes           yes           yes           passed
    ms-ol-node-01     yes           yes           yes           yes           passed
  Group Membership: oinstall(Primary) ...PASSED
  Run Level ...
    Node Name     run level                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  5                         3,5                       passed
    ms-ol-node-01  5                         3,5                       passed
  Run Level ...PASSED
  Hard Limit: maximum open file descriptors ...
    Node Name         Type          Available     Required      Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     hard          65536         65536         passed
    ms-ol-node-01     hard          65536         65536         passed
  Hard Limit: maximum open file descriptors ...PASSED
  Soft Limit: maximum open file descriptors ...
    Node Name         Type          Available     Required      Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     soft          1024          1024          passed
    ms-ol-node-01     soft          1024          1024          passed
  Soft Limit: maximum open file descriptors ...PASSED
  Hard Limit: maximum user processes ...
    Node Name         Type          Available     Required      Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     hard          16384         16384         passed
    ms-ol-node-01     hard          16384         16384         passed
  Hard Limit: maximum user processes ...PASSED
  Soft Limit: maximum user processes ...
    Node Name         Type          Available     Required      Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     soft          16384         2047          passed
    ms-ol-node-01     soft          16384         2047          passed
  Soft Limit: maximum user processes ...PASSED
  Soft Limit: maximum stack size ...
    Node Name         Type          Available     Required      Status
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     soft          10240         10240         passed
    ms-ol-node-01     soft          10240         10240         passed
  Soft Limit: maximum stack size ...PASSED
  Architecture ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  x86_64                    x86_64                    passed
    ms-ol-node-01  x86_64                    x86_64                    passed
  Architecture ...PASSED
  OS Kernel Version ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  5.15.0-206.153.7.1.el8uek.x86_64  5.4.17                    passed
    ms-ol-node-01  5.15.0-206.153.7.1.el8uek.x86_64  5.4.17                    passed
  OS Kernel Version ...PASSED
  OS Kernel Parameter: semmsl ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     250           250           250           passed
    ms-ol-node-02     250           250           250           passed
  OS Kernel Parameter: semmsl ...PASSED
  OS Kernel Parameter: semmns ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     32000         32000         32000         passed
    ms-ol-node-02     32000         32000         32000         passed
  OS Kernel Parameter: semmns ...PASSED
  OS Kernel Parameter: semopm ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     100           100           100           passed
    ms-ol-node-02     100           100           100           passed
  OS Kernel Parameter: semopm ...PASSED
  OS Kernel Parameter: semmni ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     128           128           128           passed
    ms-ol-node-02     128           128           128           passed
  OS Kernel Parameter: semmni ...PASSED
  OS Kernel Parameter: shmmax ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     4398046511104  4398046511104  5209774080    passed
    ms-ol-node-02     4398046511104  4398046511104  5209774080    passed
  OS Kernel Parameter: shmmax ...PASSED
  OS Kernel Parameter: shmmni ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     4096          4096          4096          passed
    ms-ol-node-02     4096          4096          4096          passed
  OS Kernel Parameter: shmmni ...PASSED
  OS Kernel Parameter: shmall ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     1073741824    1073741824    1073741824    passed
    ms-ol-node-02     1073741824    1073741824    1073741824    passed
  OS Kernel Parameter: shmall ...PASSED
  OS Kernel Parameter: file-max ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     6815744       6815744       6815744       passed
    ms-ol-node-02     6815744       6815744       6815744       passed
  OS Kernel Parameter: file-max ...PASSED
  OS Kernel Parameter: ip_local_port_range ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     between 9000 & 65535  between 9000 & 65535  between 9000 & 65535  passed
    ms-ol-node-02     between 9000 & 65535  between 9000 & 65535  between 9000 & 65535  passed
  OS Kernel Parameter: ip_local_port_range ...PASSED
  OS Kernel Parameter: rmem_default ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     262144        262144        262144        passed
    ms-ol-node-02     262144        262144        262144        passed
  OS Kernel Parameter: rmem_default ...PASSED
  OS Kernel Parameter: rmem_max ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     4194304       4194304       4194304       passed
    ms-ol-node-02     4194304       4194304       4194304       passed
  OS Kernel Parameter: rmem_max ...PASSED
  OS Kernel Parameter: wmem_default ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     262144        262144        262144        passed
    ms-ol-node-02     262144        262144        262144        passed
  OS Kernel Parameter: wmem_default ...PASSED
  OS Kernel Parameter: wmem_max ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     1048576       1048576       1048576       passed
    ms-ol-node-02     1048576       1048576       1048576       passed
  OS Kernel Parameter: wmem_max ...PASSED
  OS Kernel Parameter: aio-max-nr ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     1048576       1048576       1048576       passed
    ms-ol-node-02     1048576       1048576       1048576       passed
  OS Kernel Parameter: aio-max-nr ...PASSED
  OS Kernel Parameter: panic_on_oops ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     1             1             1             passed
    ms-ol-node-02     1             1             1             passed
  OS Kernel Parameter: panic_on_oops ...PASSED
  OS Kernel Parameter: kernel.panic ...
    Node Name         Current       Configured    Required      Status        Comment
    ----------------  ------------  ------------  ------------  ------------  ------------
    ms-ol-node-01     10            10            at least 1    passed
    ms-ol-node-02     10            10            at least 1    passed
  OS Kernel Parameter: kernel.panic ...PASSED
  Package: binutils-2.30-113.0.2 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  binutils-2.30-123.0.2.el8  binutils-2.30-113.0.2     passed
    ms-ol-node-01  binutils-2.30-123.0.2.el8  binutils-2.30-113.0.2     passed
  Package: binutils-2.30-113.0.2 ...PASSED
  Package: libgcc-8.5.0 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  libgcc(x86_64)-8.5.0-21.0.1.el8  libgcc(x86_64)-8.5.0      passed
    ms-ol-node-01  libgcc(x86_64)-8.5.0-21.0.1.el8  libgcc(x86_64)-8.5.0      passed
  Package: libgcc-8.5.0 (x86_64) ...PASSED
  Package: libstdc++-8.5.0 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  libstdc++(x86_64)-8.5.0-21.0.1.el8  libstdc++(x86_64)-8.5.0   passed
    ms-ol-node-01  libstdc++(x86_64)-8.5.0-21.0.1.el8  libstdc++(x86_64)-8.5.0   passed
  Package: libstdc++-8.5.0 (x86_64) ...PASSED
  Package: compat-openssl10-1.0.2 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  compat-openssl10(x86_64)-1.0.2o-4.el8_10.3  compat-openssl10(x86_64)-1.0.2  passed
    ms-ol-node-01  compat-openssl10(x86_64)-1.0.2o-4.el8_10.3  compat-openssl10(x86_64)-1.0.2  passed
  Package: compat-openssl10-1.0.2 (x86_64) ...PASSED
  Package: fontconfig-2.13.1 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  fontconfig(x86_64)-2.13.1-4.el8  fontconfig(x86_64)-2.13.1  passed
    ms-ol-node-01  fontconfig(x86_64)-2.13.1-4.el8  fontconfig(x86_64)-2.13.1  passed
  Package: fontconfig-2.13.1 (x86_64) ...PASSED
  Package: sysstat-11.7.3 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  sysstat-11.7.3-13.0.3.el8_10  sysstat-11.7.3            passed
    ms-ol-node-01  sysstat-11.7.3-13.0.3.el8_10  sysstat-11.7.3            passed
  Package: sysstat-11.7.3 ...PASSED
  Package: make-4.2.1 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  make-4.2.1-11.el8         make-4.2.1                passed
    ms-ol-node-01  make-4.2.1-11.el8         make-4.2.1                passed
  Package: make-4.2.1 ...PASSED
  Package: glibc-2.28 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  glibc(x86_64)-2.28-251.0.2.el8  glibc(x86_64)-2.28        passed
    ms-ol-node-01  glibc(x86_64)-2.28-251.0.2.el8  glibc(x86_64)-2.28        passed
  Package: glibc-2.28 (x86_64) ...PASSED
  Package: glibc-devel-2.28 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  glibc-devel(x86_64)-2.28-251.0.2.el8  glibc-devel(x86_64)-2.28  passed
    ms-ol-node-01  glibc-devel(x86_64)-2.28-251.0.2.el8  glibc-devel(x86_64)-2.28  passed
  Package: glibc-devel-2.28 (x86_64) ...PASSED
  Package: libaio-0.3.112 (x86_64) ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  libaio(x86_64)-0.3.112-1.el8  libaio(x86_64)-0.3.112    passed
    ms-ol-node-01  libaio(x86_64)-0.3.112-1.el8  libaio(x86_64)-0.3.112    passed
  Package: libaio-0.3.112 (x86_64) ...PASSED
  Package: nfs-utils-2.3.3-51 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  nfs-utils-2.3.3-59.0.1.el8  nfs-utils-2.3.3-51        passed
    ms-ol-node-01  nfs-utils-2.3.3-59.0.1.el8  nfs-utils-2.3.3-51        passed
  Package: nfs-utils-2.3.3-51 ...PASSED
  Package: smartmontools-7.1-1 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  smartmontools-7.1-3.el8   smartmontools-7.1-1       passed
    ms-ol-node-01  smartmontools-7.1-3.el8   smartmontools-7.1-1       passed
  Package: smartmontools-7.1-1 ...PASSED
  Package: net-tools-2.0-0.52 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  net-tools-2.0-0.52.20160912git.el8  net-tools-2.0-0.52        passed
    ms-ol-node-01  net-tools-2.0-0.52.20160912git.el8  net-tools-2.0-0.52        passed
  Package: net-tools-2.0-0.52 ...PASSED
  Package: policycoreutils-2.9-1 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  policycoreutils-2.9-25.0.1.el8  policycoreutils-2.9-1     passed
    ms-ol-node-01  policycoreutils-2.9-25.0.1.el8  policycoreutils-2.9-1     passed
  Package: policycoreutils-2.9-1 ...PASSED
  Package: policycoreutils-python-utils-2.9-1 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  policycoreutils-python-utils-2.9-25.0.1.el8  policycoreutils-python-utils-2.9-1  passed
    ms-ol-node-01  policycoreutils-python-utils-2.9-25.0.1.el8  policycoreutils-python-utils-2.9-1  passed
  Package: policycoreutils-python-utils-2.9-1 ...PASSED
  Users With Same UID: 0 ...PASSED
  Current Group ID ...PASSED
  Root user consistency ...
    Node Name                             Status
    ------------------------------------  ------------------------
    ms-ol-node-02                         passed
    ms-ol-node-01                         passed
  Root user consistency ...PASSED
  Package: psmisc-22.6-19 ...
    Node Name     Available                 Required                  Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  psmisc-23.1-5.el8         psmisc-22.6-19            passed
    ms-ol-node-01  psmisc-23.1-5.el8         psmisc-22.6-19            passed
  Package: psmisc-22.6-19 ...PASSED
  Host name ...PASSED
  Node Connectivity ...
    Hosts File ...
    Node Name                             Status
    ------------------------------------  ------------------------
    ms-ol-node-01                         passed
    ms-ol-node-02                         passed
    Hosts File ...PASSED

Interface information for node "ms-ol-node-02"

  Name   IP Address      Subnet          Gateway         Def. Gateway    HW Address        MTU
  ------ --------------- --------------- --------------- --------------- ----------------- ------
  ens33  10.118.200.112  10.118.200.0    0.0.0.0         192.168.60.254  00:50:56:A6:27:6A 1500
  ens35  192.168.60.112  192.168.60.0    0.0.0.0         192.168.60.254  00:50:56:A6:B1:8C 1500

Interface information for node "ms-ol-node-01"

  Name   IP Address      Subnet          Gateway         Def. Gateway    HW Address        MTU
  ------ --------------- --------------- --------------- --------------- ----------------- ------
  ens33  10.118.200.111  10.118.200.0    0.0.0.0         192.168.60.254  00:50:56:A6:F0:ED 1500
  ens35  192.168.60.111  192.168.60.0    0.0.0.0         192.168.60.254  00:50:56:A6:03:03 1500

Check: MTU consistency of the subnet "192.168.60.0".

    Node              Name          IP Address    Subnet        MTU
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     ens35         192.168.60.112  192.168.60.0  1500
    ms-ol-node-01     ens35         192.168.60.111  192.168.60.0  1500

Check: MTU consistency of the subnet "10.118.200.0".

    Node              Name          IP Address    Subnet        MTU
    ----------------  ------------  ------------  ------------  ----------------
    ms-ol-node-02     ens33         10.118.200.112  10.118.200.0  1500
    ms-ol-node-01     ens33         10.118.200.111  10.118.200.0  1500

    Source                      Destination                 Connected?
    --------------------------  --------------------------  --------------------------
    ms-ol-node-01[ens35:192.168.60.111]  ms-ol-node-02[ens35:192.168.60.112]  yes

    Source                      Destination                 Connected?
    --------------------------  --------------------------  --------------------------
    ms-ol-node-01[ens33:10.118.200.111]  ms-ol-node-02[ens33:10.118.200.112]  yes
    Check that maximum (MTU) size packet goes through subnet ...PASSED
    subnet mask consistency for subnet "192.168.60.0" ...PASSED
    subnet mask consistency for subnet "10.118.200.0" ...PASSED
  Node Connectivity ...PASSED
  Multicast or broadcast check ...
    Checking subnet "10.118.200.0" for multicast communication with multicast
    group "224.0.0.251"

    Subnet        Network Type              Multicast Enabled
    ------------  ------------------------  ------------------------
    10.118.200.0  PRIVATE                   TRUE
  Multicast or broadcast check ...PASSED
  Network Time Protocol (NTP) ...PASSED
  Same core file name pattern ...PASSED
  User Mask ...
    Node Name     Available                 Required                  Comment
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  0022                      0022                      passed
    ms-ol-node-01  0022                      0022                      passed
  User Mask ...PASSED
  User Not In Group "root": grid ...
    Node Name     Status                    Comment
    ------------  ------------------------  ------------------------
    ms-ol-node-02  passed                    does not exist
    ms-ol-node-01  passed                    does not exist
  User Not In Group "root": grid ...PASSED
  Time zone consistency ...PASSED
  Path existence, ownership, permissions and attributes ...
    Path "/var" ...PASSED
    Path "/dev/shm" ...PASSED
  Path existence, ownership, permissions and attributes ...PASSED
  Time offset between nodes ...PASSED
  resolv.conf Integrity ...
    Node Name                             Status
    ------------------------------------  ------------------------
    ms-ol-node-01                         passed
    ms-ol-node-02                         passed

checking response for name "ms-ol-node-01" from each of the name servers
specified in "/etc/resolv.conf"

    Node Name     Source                    Comment                   Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-01  8.8.8.8                   IPv4                      failed

checking response for name "ms-ol-node-02" from each of the name servers
specified in "/etc/resolv.conf"

    Node Name     Source                    Comment                   Status
    ------------  ------------------------  ------------------------  ----------
    ms-ol-node-02  8.8.8.8                   IPv4                      failed
  resolv.conf Integrity ...FAILED (PRVG-10048)
  DNS/NIS name service ...PASSED
  Daemon "avahi-daemon" not configured and running ...
    Node Name     Configured                Status
    ------------  ------------------------  ------------------------
    ms-ol-node-02  no                        passed
    ms-ol-node-01  no                        passed

    Node Name     Running?                  Status
    ------------  ------------------------  ------------------------
    ms-ol-node-02  no                        passed
    ms-ol-node-01  no                        passed
  Daemon "avahi-daemon" not configured and running ...PASSED
  Daemon "proxyt" not configured and running ...
    Node Name     Configured                Status
    ------------  ------------------------  ------------------------
    ms-ol-node-02  no                        passed
    ms-ol-node-01  no                        passed

    Node Name     Running?                  Status
    ------------  ------------------------  ------------------------
    ms-ol-node-02  no                        passed
    ms-ol-node-01  no                        passed
  Daemon "proxyt" not configured and running ...PASSED
  Domain Sockets ...PASSED
  User Equivalence ...PASSED
  RPM Package Manager database ...INFORMATION (PRVG-11250)
  Maximum locked memory check ...PASSED
  /dev/shm mounted as temporary file system ...PASSED
  File system mount option hidepid for proc filesystem ...PASSED
  SCP binary check ...PASSED
  Systemd login manager IPC parameter ...PASSED
  cgroup OS compatibility ...INFORMATION (PRVG-11250)
  ORAchk health score ...INFORMATION (PRVH-1507)

Pre-check for cluster services setup was unsuccessful on all the nodes.


Failures were encountered during execution of CVU verification request "stage -pre crsinst".

resolv.conf Integrity ...FAILED
ms-ol-node-02: PRVG-10048 : Name "ms-ol-node-02" was not resolved to an address
               of the specified type by name servers "8.8.8.8".

ms-ol-node-01: PRVG-10048 : Name "ms-ol-node-01" was not resolved to an address
               of the specified type by name servers "8.8.8.8".

RPM Package Manager database ...INFORMATION
PRVG-11250 : The check "RPM Package Manager database" was not performed because
it needs 'root' user privileges.

Refer to My Oracle Support notes "2548970.1" for more details regarding errors
PRVG-11250".

cgroup OS compatibility ...INFORMATION
PRVG-11250 : The check "cgroup OS compatibility" was not performed because it
needs 'root' user privileges.

Refer to My Oracle Support notes "2548970.1" for more details regarding errors
PRVG-11250".

ORAchk health score ...INFORMATION
PRVH-1507 : ORAchk/EXAchk checks are skipped.


CVU operation performed:      stage -pre crsinst
Date:                         Aug 27, 2026, 4:09:11 PM
CVU version:                  23.26.1.0.0 (010926x8664)
CVU home:                     /u01/app/26.0.0/grid
User:                         grid
Operating system:             Linux5.15.0-206.153.7.1.el8uek.x86_64
[grid@ms-ol-node-01 grid]$

```
## Step 12: Grid Infrastructure Installation 
### 1: Select Configuration Option
<img width="842" height="631" alt="image" src="https://github.com/user-attachments/assets/b3899021-f836-49e7-8065-cca27fe2125a" />

### 2: Select Cluster Configuration
<img width="834" height="629" alt="image" src="https://github.com/user-attachments/assets/ffdc426b-15fa-4bc5-bb2b-4a76bb5d9e3c" />

### 3: Grid Plug and Play Information (SCAN)
<img width="844" height="629" alt="image" src="https://github.com/user-attachments/assets/f3cbf80d-729b-4074-bb3f-37b3dd5a0135" />

### 4: Cluster Node Information
<img width="842" height="628" alt="image" src="https://github.com/user-attachments/assets/d07bf529-bd85-41a8-8b58-006fb254da47" />

### 5: Specify Network Interface Usage
<img width="842" height="629" alt="image" src="https://github.com/user-attachments/assets/7bb5ec73-22f0-4420-b563-33957e1c8b7c" />

### 6: Storage Option Information
<img width="839" height="632" alt="image" src="https://github.com/user-attachments/assets/2174e3fb-c874-44ed-bc13-7d0c45a6339e" />

### 7: Create ASM Disk Group
<img width="832" height="631" alt="image" src="https://github.com/user-attachments/assets/17d7e0dc-84d1-4f0f-9587-25e9ee50096b" />

### 8: Specify ASM Password
<img width="839" height="631" alt="image" src="https://github.com/user-attachments/assets/9a602fc9-dc8b-48a8-9aae-99ae1753f5bc" />

### 9: Automatic Self Correction
<img width="835" height="626" alt="image" src="https://github.com/user-attachments/assets/27c51260-abd6-432b-b33c-21c9e05d0ccf" />

### 10: Failure Isolation Support
<img width="834" height="626" alt="image" src="https://github.com/user-attachments/assets/2ba34a1c-f57c-4ab4-ab3c-851102c3336b" />

### 11: Specify Management Options
<img width="844" height="632" alt="image" src="https://github.com/user-attachments/assets/3fde66b2-9b95-4154-a825-04213996a6c0" />

### 12: Privileged Operating System Groups
<img width="841" height="631" alt="image" src="https://github.com/user-attachments/assets/25eecb79-45a2-4ef5-824c-ca42e363d733" />

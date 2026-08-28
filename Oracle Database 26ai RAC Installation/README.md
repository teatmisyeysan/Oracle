# Grid Infrastructure setup
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
10.118.200.111 ms-ol-node-01.localdomain ms-ol-node-01
10.118.200.112 ms-ol-node-02.localdomain ms-ol-node-02

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
```bash
fdisk /dev/sdb

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
sdd           8:48   0  100G  0 disk
sr0          11:0    1 13.2G  0 rom  /run/media/root/OL-8-10-0-BaseOS-x86_64
[root@ms-ol-node-01 ~]#
[root@ms-ol-node-01 ~]#

```

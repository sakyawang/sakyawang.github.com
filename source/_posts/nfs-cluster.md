---
title: Centos 7.2搭建NFS集群
date: 2019-02-18 11:04:47
description: Cetos 7.2下搭建NFS高可用集群
categories:
- cluster
  
tags:
- nfs
- cluster
- pacemaker
- drbd
- crm
- pcs
---

## 使用到的软件列表

| 名字 | 说明 |
| ---- | ---- |
| DRBD | 基于网络的RAID1实现 |
| NFS |  网络文件系统    |
| pacemaker |  一个群集资源管理器    |
| crmsh |   pacemaker cli |
| pcs |  packmaker web ui + cli   |
| corosync |  集群成员之间通信管理软件   |

## 环境准备

| host  | ip        | vip       | os        | lvm device    | drbd device | nfs directory |
| ----- | --------- | --------- | --------- | ------------- | ----------- | ------------- |
| node1 | 10.2.6.41 | 10.2.6.40 | Cetnos7.2 | /dev/nfs/work | drbr0       | /home/nfs     |
| node2 | 10.2.6.42 | 10.2.6.40 | Cetnos7.2 | /dev/nfs/work | drbd0       | /home/nfs     |

1. 时间同步

    ```shell
    [root@node1 .ssh]# sudo ntp timeserver

    [root@node2 .ssh]# sudo ntp timeserver
    ```

2. hosts配置

    ```shell
    [root@node1 .ssh]# sudo hostnamectl set-hostname node1
    [root@node1 .ssh]# sudo vim /etc/hosts
        node1   10.2.6.41
        node2   10.2.6.42

    [root@node2 .ssh]# sudo hostnamectl set-hostname node2
    [root@node2 .ssh]# sudo vim /etc/hosts
        node1   10.2.6.41
        node2   10.2.6.42
    ```

3. 免密登录配置

    ```shell
    # 生成密钥对
    [root@node1 .ssh]# sudo ssh-keygen
    # 公钥写入authorized_keys
    [root@node1 .ssh]# sudo scp /root/.ssh/id_rsa.pub root@node2:/home

    # 生成密钥对
    [root@node2 .ssh]# sudo ssh-keygen
    # 公钥写入authorized_keys
    [root@node2 .ssh]# sudo scp /root/.ssh/id_rsa.pub root@node1:/home

    [root@node1 .ssh]# sudo cat /home/id_rsa.pub >> /root/.ssh/authorized_keys

    [root@node2 .ssh]# sudo cat /home/id_rsa.pub >> /root/.ssh/authorized_keys
    ```

4. 关闭防火墙

    ```shell
    [root@node1 .ssh]# sudo systemctl stop firewalld 
    [root@node1 .ssh]# sudo systemctl disable firewalld

    [root@node2 .ssh]# sudo systemctl stop firewalld 
    [root@node2 .ssh]# sudo systemctl disable firewalld
    ```    

5. 关闭selinux

    ```shell
    [root@node1 .ssh]# sudo vim /etc/selinux/config
        SELINUX=disabled
    [root@node1 .ssh]# sudo reboot now

    [root@node2 .ssh]# sudo vim /etc/selinux/config
        SELINUX=disabled
    [root@node2 .ssh]# sudo reboot now
    ```

## 环境搭建

### 创建lvm

```shell
# 新建分区/dev/sdb1
fdisk /dev/sdb
n
w
fdisk /dev/sdb
t
8e
w
# 创建物理卷
pvcreate /dev/sdb1
# 创建卷组
vgcreate nfs /dev/sdb1
# 创建lvm
lvcreate -n work -L 50G nfs
```

### 安装配置DRBD

> 无特殊说明以下操作均在两个节点都操作

1. 添加ELRepo仓库

    ```shell
    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm)
    ```
2. 安装DRBD

    ```shell
    yum install drbd90-utils kmod-drbd90

    # 禁止drbd开机启动，由pacemaker托管
    systemctl disable drbd
    ```
3. 检查drbd内核模块

    ```shell
    lsmod | grep drbd

    # 添加到内核
    modprobe drbd

    # 启动加载内核模块
    echo drbd > /etc/modules-load.d/drbd.conf
    ```
4. 配置/etc/drbd.d/drbd0.res
   
    ```shell
    resource drbd0 {
        disk /dev/nfs/work;
        device /dev/drbd0;
        meta-disk internal;
        on node1 {
                address 10.2.6.41:7790;
        }
        on node2 {
                address 10.2.6.42:7790;
        }
    }
    ```
5. 创建元数据
   
   ```shell
   drbdadm create-md drbd0
   ```
6. 启动设备

    ```shell
    drbdadm up drbd0
    ```
7. 保证状态同步

    ```shell
    drbdadm -- --clear-bitmap new-current-uuid drbd0/0
    ```
8. 主节点node1上配置

    ```shell
    drbdadm primary drbd0
    mkfs.xfs /dev/drbd0
    ```
9. 创建挂载点

    ```shell
    mkdir /home/nfs/work
    ```
10. 备节点node2配置

    ```shell
    drbdadm secondary drbd0
    ```

### 安装pcs、pacemaker、corosync

1. 安装
```shell
yum install pacemaker pcs resource-agents
```

2. 设置hacluster用户密码

    ```shell
    passwd hacluster
    ```
3. 创建集群

    ```shell
    pcs cluster auth node1 node2 -u hacluster -p password --force

    pcs cluster setup --force --name nfs_cluster node1 node2

    pcs cluster start --all
    pcs property set no-quorum-policy=ignore
    pcs property set stonith-enabled=false
    pcs cluster enable --all
    pcs status
    ```
### 安装crmsh

1. github下载安装包[CRMSH](https://github.com/ClusterLabs/crmsh)
2. 安装
   
    ```shell
    tar zxf ***.tar
    cd crmsh
    ./autogen.sh
    ./configure
    make
    make install

    python setup.py install

    ```
### 安装nfs

```shell
yum install nfs-utils rpcbind
systemctl enable rpcbind
systemctl start rpcbind
```
### 使用CRM管理配置pacemaker nfs集群

1. 配置DRBD资源

```shell
# 进入crm控制台
crm configure
# 添加drbd资源
primitive p_drbd_r0 ocf:linbit:drbd \
  params drbd_resource=drbd0 \
  op start interval=0s timeout=240s \
  op stop interval=0s timeout=100s \
  op monitor interval=31s timeout=20s role=Slave \
  op monitor interval=29s timeout=20s role=Master
# 创建master/slave
ms ms_drbd_r0 p_drbd_r0 meta master-max=1 \
  master-node-max=1 clone-max=2 clone-node-max=1 notify=true
# 验证
verify
# 提交
commit
# 退出
exit
```
2. 设置nfs使用的文件系统资源
```shell
# 进入crm控制台
crm configure
# 添加文件系统
primitive p_fs_drbd0 ocf:heartbeat:Filesystem \
  params device=/dev/drbd0 directory=/home/nfs fstype=xfs \
  options=noatime,nodiratime \
  op start interval="0" timeout="60s" \
  op stop interval="0" timeout="60s" \
  op monitor interval="20" timeout="40s"
# 添加依赖资源启动顺序
order o_drbd_r0-before-fs_drbd0 \
  inf: ms_drbd_r0:promote p_fs_drbd0:start
# 托管服务配置
colocation c_fs_drbd0-with_drbd-r0 \
  inf: p_fs_drbd0 ms_drbd_r0:Master
# 验证
verify
# 提交
commit
# 退出
exit
```
3. 设置nfs服务资源
```shell 
# 进入crm控制台
crm configure
# 添加nfs服务
primitive p_nfsserver ocf:heartbeat:nfsserver \
  params nfs_shared_infodir=/home/nfs/nfs_shared_infodir nfs_ip=10.2.6.40 \
  op start interval=0s timeout=40s \
  op stop interval=0s timeout=20s \
  op monitor interval=10s timeout=20s
# 添加依赖资源启动顺序
order o_fs_drbd0-before-nfsserver \
  inf: p_fs_drbd0 p_nfsserver
# 托管服务配置
colocation c_nfsserver-with-fs_drbd0 \
  inf: p_nfsserver p_fs_drbd0 
# 验证
verify
# 提交
commit
# 退出
exit
```
4. 设置nfs导出目录配置资源
```shell
mkdir -p /home/nfs/exports/iso
chown nfsnobody:nfsnobody /home/nfs/exports/iso
# 进入crm控制台
crm configure
# 添加nfs服务
primitive p_exportfs_dir1 ocf:heartbeat:exportfs \
  params clientspec=* directory=/home/nfs/exports/iso fsid=1 \
  unlock_on_stop=1 options=rw,sync \
  op start interval=0s timeout=40s \
  op stop interval=0s timeout=120s \
  op monitor interval=10s timeout=20s
# 添加依赖资源启动顺序
order o_nfsserver-before-exportfs-dir1 \
  inf: p_nfsserver p_exportfs_dir1
# 托管服务配置
colocation c_exportfs-with-nfsserver \
  inf: p_exportfs_dir1 p_nfsserver
# 验证
verify
# 提交
commit
# 退出
exit
```
5. 设置虚拟IP资源
```shell
# 进入crm控制台
crm configure
# 添加VIP
primitive p_virtip_dir1 ocf:heartbeat:IPaddr2 \
  params ip=10.2.6.40 cidr_netmask=16 nic=ens160 \
  op monitor interval=20s timeout=20s \
  op start interval=0s timeout=20s \
  op stop interval=0s timeout=20s
# 添加依赖资源启动顺序
order o_exportfs_dir1-before-p_virtip_dir1 \
  inf: p_exportfs_dir1 p_virtip_dir1
# 托管服务配置
colocation c_virtip_dir1-with-exportfs-dir1 \
  inf: p_virtip_dir1 p_exportfs_dir1
# 验证
verify
# 提交
commit
# 退出
exit
```
## 集群验证

pcs WEB UI https://10.2.6.41:2224

在客户端设备上验证

```shell
# 查看nfs挂载目录
showmount -e 10.2.6.40
# 挂载
mkdir /mnt/test_nfs_mount

mount 10.2.6.40:/home/nfs/exports/iso /mnt/test_nfs_mount

dd if=/dev/zero of=/mnt/test_nfs_mount/test.out bs=1M count=1024

# 以上文件写完之前在cluster primary节点操作
echo b > /proc/sysrq-trigger

验证文件是否正常写完
```

## 引用

[redhat ha add-on](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/)

[ClusterLabs](https://clusterlabs.org/)

[DRBD CONFIG](http://yallalabs.com/linux/how-to-install-and-configure-drbd-cluster-on-rhel7-centos7/)

[HA NFS](https://www.linbit.com/downloads/tech-guides/HA_NFS_Cluster_using_Pacemaker_and_DRBD_on_RHEL7.pdf)
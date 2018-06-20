---
title: Falcon部署配置
date: '2018-03-27'
description: Falcon部署配置,集成Gralcon
categories: 
- devops

tags:
- falcon
- gralcon
---

## 环境准备

    Centos7.2
    Golang1.8.1
    redis 3.2.10
    mysql 5.7.21
    open-falcon
    falcon-dashboard
    grafana

## Golang 安装配置

1. 下载安装包
2. 解压缩
    
    tar -C /usr/local -zxf go1.8.1.linux-amd64.tar.gz

3. 配置PATH

    vim /etc/profile
    添加
        export PATH=$PATH:/usr/local/go/bin
    source /etc/profile

4. 配置GOPATH（工作空间）

    vim /root/.bash_profile
    添加
        export GOPATH=$HOME/go
    source /root/.bash_profile

## redis 安装配置

*安装*

```shell
yum update
yum install epel-release
yum install redis
```
*配置*

```shell
 chkconfig redis status
 service redis start
```
>注意：open-falcon暂不支持redis使用密码连接

## mysql安装配置

*安装*

```shell
wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
yum localinstall mysql57-community-release-el7-7.noarch.rpm 
yum remove mariadb-server.x86_64
yum install mysql-community-server
```

*修改密码*

```shell
启动服务：
    service mysqld start
获取初始化密码：
    grep 'temporary password' /var/log/mysqld.log
连接修改密码：
    mysql -h 127.0.0.1 -u root -p password
    alter user 'root'@'localhost' identified by newpassword
```
## open-falcon安装配置

*下载源码*

```shell
mkdir -p $GOPATH/src/github.com/open-falcon
cd $GOPATH/src/github.com/open-falcon
git clone https://github.com/open-falcon/falcon-plus.git
```

*初始化数据库*

```shell
cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema
mysql -h 127.0.0.1 -u root -p < 1_uic-db-schema.sql 
mysql -h 127.0.0.1 -u root -p < 2_portal-db-schema.sql 
mysql -h 127.0.0.1 -u root -p < 3_dashboard-db-schema.sql 
mysql -h 127.0.0.1 -u root -p < 4_graph-db-schema.sql 
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

*编译制作安装包*

```shell
go get -u github.com/kardianos/govendor
cd $GOPATH/src/github.com/open-falcon/falcon-plus/
make all
make agent
make pack
```

*安装*

```shell
cd $GOPATH/src/github.com/open-falcon/falcon-plus
mkdir -p /root/open-falcon
tar -zxf open-falcon-v0.2.1.tar.gz -C /root/open-falcon
```

*配置*

所有模块的cfg.json中的database.addr添加密码，即root:password@

*启动服务*

```shell
./open-falcon start
./open-falcon check   //检查服务是否启动
```

*agent安装配置*

解压缩安装包到指定的机器上。

```shell
vim agent/config/cfg.json

"heartbeat": {
        "enabled": true,
        "addr": "10.2.6.31:6030",
        "interval": 60,
        "timeout": 1000
    },
    "transfer": {
        "enabled": true,
        "addrs": [
            "10.2.6.31:8433"
        ],
        "interval": 60,
        "timeout": 1000
    },

addr修改为open-falcon服务的IP地址
启动agent：
    ./open-falcon start agent
```

## falcon-dashboard安装配置

*安装配置*

```shell
mkdir -p $HOME/open-falcon/
cd $HOME/open-falcon && git clone https://github.com/open-falcon/dashboard.git
cd dashboard

yum install -y python-virtualenv
yum install -y python-devel
yum install -y openldap-devel
yum install -y mysql-devel
yum groupinstall "Development tools"

cd $HOME/open-falcon/dashboard/
virtualenv ./env

./env/bin/pip install -r pip_requirements.txt 
修改数据库配置密码：
    vim rrd/config.py
    PORTAL_DB_HOST = os.environ.get("PORTAL_DB_HOST","127.0.0.1")
    PORTAL_DB_PORT = int(os.environ.get("PORTAL_DB_PORT",3306))
    PORTAL_DB_USER = os.environ.get("PORTAL_DB_USER","root")
    PORTAL_DB_PASS = os.environ.get("PORTAL_DB_PASS","yourpassword")
    PORTAL_DB_NAME = os.environ.get("PORTAL_DB_NAME","falcon_portal")
    
    # alarm database
    # TODO: read from api instead of db
    ALARM_DB_HOST = os.environ.get("ALARM_DB_HOST","127.0.0.1")
    ALARM_DB_PORT = int(os.environ.get("ALARM_DB_PORT",3306))
    ALARM_DB_USER = os.environ.get("ALARM_DB_USER","root")
    ALARM_DB_PASS = os.environ.get("ALARM_DB_PASS","yourpassword")
    ALARM_DB_NAME = os.environ.get("ALARM_DB_NAME","alarms")
运行：
    ./env/bin/python wsgi.py
```

## grafana集成

*安装配置*

```shell
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.0.3-1.x86_64.rpm
yum install initscripts fontconfig
rpm -Uvh grafana-5.0.3-1.x86_64.rpm 
/bin/systemctl enable grafana-server.service
/bin/systemctl start grafana-server.service

service grafana-server start
```

*安装open-falcon插件*

```shell
cd /var/lib/grafana/plugins
git clone https://github.com/open-falcon/grafana-openfalcon-datasource
```

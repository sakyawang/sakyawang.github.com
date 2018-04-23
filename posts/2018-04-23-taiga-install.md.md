---
title: Taiga环境搭建
date: '2018-04-23'
description: Taiga环境搭建
categories: soft

tags:
- scrum
- 敏捷
- taiga
---

# Taiga环境搭建

## 环境配置

    unbuntu 16.4 
    taiga stable

## 安装

1. 创建taiga用户
    
        sudo adduser taiga
        sudo adduser taiga sudo
        sudo su taiga

2. 安装依赖软件

        sudo apt-get update
        sudo apt-get install -y build-essential binutils-doc autoconf flex bison libjpeg-dev
        sudo apt-get install -y libfreetype6-dev zlib1g-dev libzmq3-dev libgdbm-dev libncurses5-dev
        sudo apt-get install -y automake libtool libffi-dev curl git tmux gettext
        sudo apt-get install -y nginx
        sudo apt-get install -y rabbitmq-server redis-server
        sudo apt-get install -y circus

        sudo apt-get install -y postgresql-9.5 postgresql-contrib-9.5
        sudo apt-get install -y postgresql-doc-9.5 postgresql-server-dev-9.5

        sudo apt-get install -y python3 python3-pip python-dev python3-dev python-pip virtualenvwrapper
        sudo apt-get install -y libxml2-dev libxslt-dev
        sudo apt-get install -y libssl-dev libffi-dev

3. 初始化postgresql

        sudo -u postgres createuser taiga
        sudo -u postgres createdb taiga -O taiga --encoding='utf-8' --locale=en_US.utf8 --template=template0


4. 初始化RabbitMQ

        sudo rabbitmqctl add_user taiga PASSWORD_FOR_EVENTS
        sudo rabbitmqctl add_vhost taiga
        sudo rabbitmqctl set_permissions -p taiga taiga ".*" ".*" ".*"

5. 创建日志目录

        mkdir -p ~/logs

6. Taiga backend安装配置
    
        ## 下载代码切换分支
        cd ~
        git clone https://github.com/taigaio/taiga-back.git taiga-back
        cd taiga-back
        git checkout stable

        ## 创建python虚拟环境 
        mkvirtualenv -p /usr/bin/python3.5 taiga

        ## 安装依赖
        pip install -r requirements.txt

        ## 初始化数据库（默认用户 admin/123123）
        python manage.py migrate --noinput
        python manage.py loaddata initial_user
        python manage.py loaddata initial_project_templates
        python manage.py compilemessages
        python manage.py collectstatic --noinput

        ## 配置~/taiga-back/settings/local.py
        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.postgresql',
                'NAME': 'taiga',
                'USER': 'taiga',
                'PASSWORD': 'Cloudmap123',
                'HOST': '',
                'PORT': '',
            }
        }

        SITES = {
            "api": {
               "scheme": "http",
               "domain": "10.2.0.6:8000",
               "name": "api"
            },
            "front": {
               "scheme": "http",
               "domain": "10.2.0.6:9001",
               "name": "front"
            }
        }

        ## 配置~/taiga-back/settings/common.py
        LANGUAGE_CODE = 'zh-hans'  # 修改默认语言为简体中文

        MEDIA_URL = "http://10.2.0.6/media/" # 解决wen界面用户头像url异常为：localhost:8000
        STATIC_URL = "http://10.2.0.6/static/" # 解决后台管理系统样式丢失
        
        ## 配置事件连接秘钥，值取taiga-event的config.json中的secret
        SECRET_KEY = "aw3+t2r(8(0kkrhg8)gx6i96v5^kv%6cfep9wxfom0%7dy0m9e"

        ## 验证配置
        workon taiga
        python manage.py runserver
        # 访问 http://localhost:8000/api/v1/ 有数据返回

7. Taiga frontend安装配置

        ## 下载代码切换分支
        cd ~
        git clone https://github.com/taigaio/taiga-front-dist.git taiga-front-dist
        cd taiga-front-dist
        git checkout stable

        ## 创建配置文件修改配置
        cp ~/taiga-front-dist/dist/conf.example.json ~/taiga-front-dist/dist/conf.json

        {
            "api": "http://10.2.0.6/api/v1/",
            "eventsUrl": "ws://10.2.0.6/events",
            "eventsMaxMissedHeartbeats": 5,
            "eventsHeartbeatIntervalTime": 60000,
            "eventsReconnectTryInterval": 10000,
            "debug": true,
            "debugInfo": false,
            "defaultLanguage": "zh-hans", /**修改默认语言为中文简体**/
            "themes": ["taiga"],
            "defaultTheme": "taiga",
            "publicRegisterEnabled": false,
            "feedbackEnabled": true,
            "supportUrl": "https://tree.taiga.io/support",
            "privacyPolicyUrl": null,
            "termsOfServiceUrl": null,
            "maxUploadFileSize": null,
            "contribPlugins": [],
            "tribeHost": null,
            "importers": [],
            "gravatar": true
        }

8. Taiga event安装配置（可选，用来做消息通知）

        ## 下载代码
        cd ~
        git clone https://github.com/taigaio/taiga-events.git taiga-events
        cd taiga-events

        ## 安装nodejs、npm
        curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
        sudo apt-get install -y nodejs
        sudo apt-get install -y npm

        ## 修改nodejs服务为node，解决npm install找不到node服务异常
        sudo ln -s /usr/bin/nodejs /usr/bin/node

        ## 安装项目nodejs模块依赖
        npm install
        sudo npm install -g coffee-script

        ## 创建配置文件修改配置
        cp config.example.json config.json

        {
            "url": "amqp://taiga:cloudmap@localhost:5672/taiga", /**用户名密码为第四步初始化rabbitmq时设置的**/
            "secret": "aw3+t2r(8(0kkrhg8)gx6i96v5^kv%6cfep9wxfom0%7dy0m9e",/**配置事件连接秘钥，值和taiga-ebackend的ccommon.py的SECRET_KEY一致**/
            "webSocketServer": {
                "port": 8888
            }
        }

        ## 添加配置到circus中/etc/circus/conf.d/taiga-events.ini
        [watcher:taiga-events]
        working_dir = /home/taiga/taiga-events
        cmd = /usr/local/bin/coffee
        args = index.coffee
        uid = taiga
        numprocesses = 1
        autostart = true
        send_hup = true
        stdout_stream.class = FileStream
        stdout_stream.filename = /home/taiga/logs/taigaevents.stdout.log
        stdout_stream.max_bytes = 10485760
        stdout_stream.backup_count = 12
        stderr_stream.class = FileStream
        stderr_stream.filename = /home/taiga/logs/taigaevents.stderr.log
        stderr_stream.max_bytes = 10485760
        stderr_stream.backup_count = 12
        
        ## 重启circusd服务，查看状态 
        sudo service circusd restart
        sudo service circusd status                            

9. 配置taiga启动服务
    
        ## 初始化Taiga的circus配置/etc/circus/conf.d/taiga.ini
        [watcher:taiga]
        working_dir = /home/taiga/taiga-back
        cmd = gunicorn
        args = -w 3 -t 60 --pythonpath=. -b 127.0.0.1:8001 taiga.wsgi
        uid = taiga
        numprocesses = 1
        autostart = true
        send_hup = true
        stdout_stream.class = FileStream
        stdout_stream.filename = /home/taiga/logs/gunicorn.stdout.log
        stdout_stream.max_bytes = 10485760
        stdout_stream.backup_count = 4
        stderr_stream.class = FileStream
        stderr_stream.filename = /home/taiga/logs/gunicorn.stderr.log
        stderr_stream.max_bytes = 10485760
        stderr_stream.backup_count = 4

        [env:taiga]
        PATH = /home/taiga/.virtualenvs/taiga/bin:$PATH
        TERM=rxvt-256color
        SHELL=/bin/bash
        USER=taiga
        LANG=en_US.UTF-8
        HOME=/home/taiga
        PYTHONPATH=/home/taiga/.virtualenvs/taiga/lib/python3.5/site-packages

        ## 重启circusd服务并验证
        sudo service circusd restart    
        circusctl status

10. 配置nginx

        ## 移除默认配置避免和taiga冲突
        sudo rm /etc/nginx/sites-enabled/default

        ## 配置/etc/nginx/conf.d/taiga.conf
        server {
            listen 80 default_server;
            server_name _;

            large_client_header_buffers 4 32k;
            client_max_body_size 50M;
            charset utf-8;

            access_log /home/taiga/logs/nginx.access.log;
            error_log /home/taiga/logs/nginx.error.log;

            # Frontend
            location / {
                root /home/taiga/taiga-front-dist/dist/;
                try_files $uri $uri/ /index.html;
            }

            # Backend
            location /api {
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Scheme $scheme;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://127.0.0.1:8001/api;
                proxy_redirect off;
            }

            # Django admin access (/admin/)
            location /admin {
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Scheme $scheme;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://127.0.0.1:8001$request_uri;
                proxy_redirect off;
            }

            # Static files
            location /static {
                alias /home/taiga/taiga-back/static;
            }

            # Media files
            location /media {
                alias /home/taiga/taiga-back/media;
            }

            # Taiga-events
            location /events {
                proxy_pass http://127.0.0.1:8888/events;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_connect_timeout 7d;
                proxy_send_timeout 7d;
                proxy_read_timeout 7d;
            }
        }

        ## 验证配置文件
        sudo nginx -t

        ## 重启nginx
        sudo service nginx restart
# 安装Docker

> - 下载安装包
>
>   ```shell
>   wget https://download.docker.com/linux/static/stable/aarch64/docker-26.1.3.tgz 
>   ```
>
> - 解压
>
>   ```shell
>   tar -zxvf docker-26.1.3.tgz
>   ```
>
> - 移动解压出来的二进制文件到 /usr/bin 目录中
>
>   ```shell
>   mv docker/* /usr/bin/
>   ```
>
> - 编辑docker的系统服务文件
>
>   `vim /usr/lib/systemd/system/docker.service`
>
>   ```bash
>   [Unit]
>   Description=Docker Application Container Engine
>   Documentation=https://docs.docker.com
>   After=network-online.target firewalld.service
>   Wants=network-online.target
>   [Service]
>   Type=notify
>   ExecStart=/usr/bin/dockerd
>   ExecReload=/bin/kill -s HUP $MAINPID
>   LimitNOFILE=infinity
>   LimitNPROC=infinity
>   TimeoutStartSec=0
>   Delegate=yes
>   KillMode=process
>   Restart=on-failure
>   StartLimitBurst=3
>   StartLimitInterval=60s
>   [Install]
>   WantedBy=multi-user.target
>   ```
>
> - 重新加载和重启docker
>
>   ```shell
>   systemctl daemon-reload
>   systemctl restart docker
>   ```

## 配置docker

> - 创建`docker`群组
>
>   ```shell
>   sudo groupadd docker
>   ```
>
> - 将您的用户添加到`docker`组中。
>
>   ```shell
>   sudo usermod -aG docker $USER
>   ```
>
> - `sudo gpasswd -a bmyeln docker`：将docker账户给与权限
>
> - `sudo service docker restart`：重启docker

## 相关命令

### 重启 docker 服务：systemctl restart  docker

## 部署mysql

> 1. 拉取镜像：`docker pull mysql:8.0.39`
>
> 2. 创建挂载目录
>
>    ```shell
>    mkdir -p /home/app/mysql/conf.d
>    mkdir -p /home/app/mysql/data
>    mkdir -p /home/app/mysql/logs
>    ```
>
> 3. 放入my.cnf
>
>    `vim /home/app/mysql/conf.d/my.cnf`
>
>    ```
>    # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
>    #
>    # This program is free software; you can redistribute it and/or modify
>    # it under the terms of the GNU General Public License as published by
>    # the Free Software Foundation; version 2 of the License.
>    #
>    # This program is distributed in the hope that it will be useful,
>    # but WITHOUT ANY WARRANTY; without even the implied warranty of
>    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
>    # GNU General Public License for more details.
>    #
>    # You should have received a copy of the GNU General Public License
>    # along with this program; if not, write to the Free Software
>    # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
>     
>    #
>    # The MySQL  Server configuration file.
>    #
>    # For explanations see
>    # http://dev.mysql.com/doc/mysql/en/server-system-variables.html
>     
>    [mysqld]
>    pid-file        = /var/run/mysqld/mysqld.pid
>    socket          = /var/run/mysqld/mysqld.sock
>    datadir         = /var/lib/mysql
>    secure-file-priv= NULL
>    # Disabling symbolic-links is recommended to prevent assorted security risks
>    symbolic-links=0
>     
>    # Custom config should go here
>    !includedir /etc/mysql/conf.d/
>    ```
>
> 4. 启动容器
>
>    - ```shell
>      docker run --name mysql -d -p 3306:3306 --restart=always \
>      -v /home/yhc/app/mysql/conf.d/my.cnf:/etc/mysql/my.cnf \
>      -v /home/yhc/app/mysql/logs:/logs \
>      -v /home/yhc/app/mysql/data/mysql:/var/lib/mysql \
>      -e MYSQL_ROOT_PASSWORD=xxxx \
>      mysql:8.0.39
>      ```
>
>    - ```shell
>      --restart=always -> 开机启动容器,容器异常自动重启
>      -d -> 以守护进程的方式启动容器
>      -p 3306:3306 -> 绑定宿主机端口
>      -v /home/yhc/app/mysql/conf.d/my.cnf:/etc/mysql/my.cnf -> 映射配置文件
>      -v /home/yhc/app/mysql/logs:/logs -> 映射日志
>      -v /home/yhc/app/mysql/data/mysql:/var/lib/mysql -> 映射数据
>      --name mysql -> 指定容器名称
>      -e MYSQL_ROOT_PASSWORD=123456 -> 写入配置root密码
>      ```

## 部署nginx

> 1. 拉取镜像：`docker pull nginx:latest`
>
> 2. 创建挂载目录
>
>    ```shell
>    mkdir -p /home/app/mysql/conf.d
>    mkdir -p /home/app/mysql/data
>    mkdir -p /home/app/mysql/logs
>    ```
>
> 3. 放入nginx.conf
>
>    ```nginx
>    user www-data;
>    worker_processes auto;
>    pid /run/nginx.pid;
>    include /etc/nginx/modules-enabled/*.conf;
>    
>    events {
>        worker_connections 768;
>        # multi_accept on;
>    }
>    
>    http {
>        sendfile on;
>        tcp_nopush on;
>        tcp_nodelay on;
>        keepalive_timeout 65;
>        types_hash_max_size 2048;
>    
>        include /etc/nginx/mime.types;
>        default_type application/octet-stream;
>    
>        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
>        ssl_prefer_server_ciphers on;
>    
>        access_log /var/log/nginx/access.log;
>        error_log /var/log/nginx/error.log;
>    
>        gzip on;
>    
>        include /etc/nginx/conf.d/*.conf;
>        include /etc/nginx/sites-enabled/*;
>    
>         server {
>             listen 80;
>             server_name 2darray.primrose.fun;
>             rewrite ^(.*)$ https://${server_name}$1 permanent;
>         }
>    
>        server {
>            listen 443 ssl;
>            server_name 2darray.primrose.fun;
>            ssl_certificate   cert/2darray/app.pem;
>            ssl_certificate_key  cert/2darray/app.key;
>            ssl_session_timeout 5m;
>            ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
>            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
>            ssl_prefer_server_ciphers on;
>            charset utf-8;
>            access_log /var/log/nginx/access.log;
>            error_log /var/log/nginx/error.log;
>    
>            client_max_body_size 10M;
>    
>            location /api {
>                proxy_pass http://127.0.0.1:3000;
>            }
>    
>            location /websocket {
>                proxy_pass http://127.0.0.1:3000;
>                proxy_set_header X-Real-IP $remote_addr;
>                proxy_set_header Host $http_host;
>                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>    
>                proxy_http_version 1.1;
>                proxy_set_header Upgrade $http_upgrade;
>                proxy_set_header Connection "upgrade";
>                proxy_read_timeout  36000s;
>            }
>    
>            location / {
>                root /project/2darray/dist;
>                index index.html;
>                try_files $uri $uri/ /index.html;
>            }
>    
>        }
>    }
>    ```
>
> 
>
> 4. 启动容器
>
>    - ```shell
>      docker run --name nginx -d -p 80:80 --restart=always -v /home/yhc/app/nginx/html:/usr/share/nginx/html -v /home/yhc/app/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/yhc/app/nginx/logs:/var/log/nginx -v /home/yhc/app/nginx/cert:/etc/nginx/cert -v /home/yhc/project:/project nginx:latest
>      ```
>    
>      ```shell
>      docker exec -it nginx /bin/bash
>      ```

## 部署应用

> 1. 编写 dockerfile
>
>    ```dockerfile
>    # 使用适合的基础镜像
>    FROM williamyeh/java8:latest
>    
>    # 设置工作目录
>    WORKDIR /app
>    
>    # 复制多个服务的 JAR 文件
>    COPY /home/yhc/project/2darray/backend/backend-0.0.1-SNAPSHOT.jar /app/backend-0.0.1-SNAPSHOT.jar
>    COPY /home/yhc/project/2darray/backend/matchingsystem-0.0.1-SNAPSHOT.jar /app/matchingsystem-0.0.1-SNAPSHOT.jar
>    COPY /home/yhc/project/2darray/backend/botrunningsystem-0.0.1-SNAPSHOT.jar /app/botrunningsystem-0.0.1-SNAPSHOT.jar
>    
>    # 暴露服务的端口
>    EXPOSE 3000
>    EXPOSE 3001
>    EXPOSE 3002
>    # 复制启动脚本
>    COPY start.sh /app/start.sh
>    
>    # 给启动脚本赋予执行权限
>    RUN chmod +x /app/start.sh
>    
>    # 使用启动脚本来同时运行两个服务
>    ENTRYPOINT ["/app/start.sh"]
>    ```
>
> 2. 启动脚本 `start.sh`
>
>    ```shell
>    #!/bin/bash
>    
>    # 启动service并后台运行
>    java -jar /app/backend-0.0.1-SNAPSHOT.jar &
>    java -jar /app/matchingsystem-0.0.1-SNAPSHOT.jar &
>    java -jar /app/botrunningsystem-0.0.1-SNAPSHOT.jar &
>    # 保持容器持续运行
>    wait
>    ```
>
> 3. 构建和运行容器
>
>    ```
>    docker build -t 2darray .
>    ```
>
>    ```shell
>    docker run --name 2darray -d --restart=always -p 3000:3000 -p 3001:3001 -p 3002:3002 2darray
>    ```
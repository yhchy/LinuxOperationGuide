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
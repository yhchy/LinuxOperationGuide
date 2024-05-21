> - `free -h`: 查看磁盘容量
> - `cat /proc/cpuinfo`：查看cpu配置
> - `uname -p`：查看系统架构，麒麟的鲲鹏是aarch64架构
> - 

## usermod

> 1. 用户在/etc/passwd
> 2. 组 在/etc/group 
> 3. 故获得特定组下所有用户 
> 4. grep '组名' /etc/group 获得组号 
> 5. grep '组号' /etc/passwd 获得组下所有用户
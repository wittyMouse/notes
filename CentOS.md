## CentOS 笔记

Linux 从入门到放弃系列之 CentOS

镜像版本 CentOS-8.3.2011-x86_64

### 通过 VirtualBox 安装 CentOS

首先需要下载 VirtualBox 和 CentOS 镜像，最好在虚拟机的设置页面将网络设置为“桥接网卡”（如果希望本机和虚拟机能够相互访问，虚拟机又可能访问外部网络的话），如果只需要命令行（盲猜下载的 CentOS 镜像是完整版），装系统的时候需要在 INSTALLATION SUMMARY 页面点击 Software Selection 将 Base Environment 修改为 Minimal Install，省略具体安装步骤。

### yum 切换阿里源

如果出现 yum 下载软件速度很慢的问题，可以将 yum 的镜像源切换成阿里源

```shell
# 备份 CentOS-Base.repo
mv /etc/yum.repos.d/CentOS-Linux-BaseOS.repo /etc/yum.repos.d/CentOS-Linux-BaseOS.repo.backup

# 使用 wget 下载（二选一）
wget -O /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-8.repo
# 使用 curl 下载（二选一）
curl -o /etc/yum.repos.d/CentOS-Linux-BaseOS.repo https://mirrors.aliyun.com/repo/Centos-8.repo

# 生成缓存
yum makecache
```

### 常用命令

```shell
# 关机
shutdown now

# 重启
reboot

# 查看系统信息
uname

# 查看系统版本
cat /etc/redhat-release

# 查看内核版本
uname -a
```

### 服务相关命令

```shell
# 开启服务
systemctl start xxx.service

# 关闭服务
systemctl stop xxx.service

# 重启服务
systemctl restart xxx.service

# 开机启动服务
systemctl enable xxx.service

# 关闭开机启动服务
systemctl disable xxx.service
```

### 关闭 SElinux

```shell
# 用文本编辑器打开 /etc/sysconfig/selinux 文件
vim /etc/sysconfig/selinux

# 将 SELINUX=enforcing 修改为 SELINUX=disabled，保存，重启

# 查看 SElinux 状态
sestatus
```

### 防火墙 firewall 相关命令
```shell
# 永久开放
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=80/udp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --add-port=443/udp --permanent

# 永久关闭
firewall-cmd --zone=public --remove-port=80/tcp --permanent

# 配置生效
firewall-cmd --reload

# 查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports

# 关闭防火墙
systemctl stop firewalld.service

# 查看防火墙状态
firewall-cmd --state
```

### 设置静态 IP

```shell
# 用文本编辑器打开 /etc/sysconfig/network-scripts/ 目录下的网卡配置文件，如果只设置了一个网卡，理论上只有一个文件，形如 ifcfg-enp0s3

# 查看网卡名称
nmcli c

# 打开网卡配置文件
vim /etc/sysconfig/network-scripts/ifcfg-enp0s3

# 静态 IP
BOOTPROTO=static
# 在系统启动时是否激活网卡
ONBOOT=yes
# IP 地址
IPADDR=192.168.1.200
# 子网掩码
NETMASK=255.255.255.0
# 网关
GATEWAY=192.168.1.1
# DNS
DNS1=233.5.5.5
DNS2=223.6.6.6

# 重启网络
nmcli c reload
```
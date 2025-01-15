---
published: 2025-01-14
title: Scaleway STARDUST服务器VPS改1GB硬盘安装Alpine Linux 启动Xray 最省钱
description: 1GB硬盘安装Alpine Linux并Xray
draft: false
category: other
tags:
  - VPS
---
要省钱就省钱到底，基于网友公开资料整理。

经过面板进行示例的操作均会收取0.01的费用，

通过CLI操作未知。最好用CLI进行操作，文档参考
https://github.com/scaleway/scaleway-cli/blob/master/docs/commands/instance.md#reboot-server

一般需要用到的命令

```
#重启
scw instance server reboot <实例的InstanceID>
#启动
scw instance server start <实例的InstanceID>
#关机
scw instance server stop <实例的InstanceID>
```

# 步骤

1. 开启一个10GB的IPv6实例，VPS原始系统为 Debian。
2. 关机
3. 点击实例的Attached volumes添加LOCAL STORAGE 1GB
4. 点击实例的Create flexible IP添加一个ipv4地址（可能不是必须，netboot支持ipv6了）
5. 先链接console再开机，需要按esc进行中断进入bios。按照教程启动 https://gist.github.com/karolba/a3f1c5f8d50c67f5a19e6c8f38e53e12
6. netboot.xyz 网络安装 Alpine Linux
7. 执行 setup-alpine 完成后续设置，网络设置为 DHCP，磁盘选 vdb，类型为 sys，设置密码等
8. 控制台advanced-settings设置boot启动盘
9. 在控制台 console 中登录 Alpine Linux并设置ssh密钥
10. 如果没有问题，就可以将 VPS 断电，删除 10GB 磁盘和 IPv4 地址，然后启动 VPS。

# 部署之后

使用控制台 Console 登录到服务器

## 修改DNS

vi /etc/resolv.conf

```
nameserver 2a00:1098:2b::1
nameserver 2a00:1098:2c::1
```

## 修改网络

vi /etc/network/interfaces

```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet6 static
        address <ipv6_address>/64
        gateway <ipv6_gateway>
```

## 修改SSH配置

vi /etc/ssh/sshd_config

取消注释以下选项
PubkeyAuthentication 是否允许使用密钥验证方式登录

AuthorizedKeysFile 允许登录主机的公钥存放文件，默认为用户家目录下的 .ssh/authorized_keys

将自己的私钥放到/root/.ssh/authorized_keys

```
cd /root
mkdir .ssh
touch .ssh/authorized_keys
vi .ssh/authorized_keys
```

修改完重启 reboot 测试一下

# 安装Xray

xray项目：https://github.com/XTLS/Xray-core

适用于已有config配置，但不想使用脚本/ui面板的用户。

其他参考

一键脚本 https://github.com/zxcvos/Xray-script

x-ui https://github.com/MHSanaei/3x-ui

## 下载xray

```
# 创建文件夹xray
mkdir xray

# 移动到xray
cd xray

# 本机属于linux64，链接可能过时，链接为实例，自行到项目获取最新版本
wget https://github.com/XTLS/Xray-core/releases/download/v25.1.1/Xray-linux-64.zip

# 解压
unzip Xray-linux-64.zip
```

## 设置xray服务

### 添加一个xray服务

touch /etc/init.d/xray

vi /etc/init.d/xray

```bash
#!/sbin/openrc-run

# 设置服务名称和描述
name="Xray"
description="Xray with XTLS support"

# 设置环境变量和配置文件路径
: ${env:="XRAY_LOCATION_ASSET=/root/xray"}
: ${confdir:="/root/xray"}

# 设置 Xray 执行文件路径和启动参数
command="/root/xray/xray"
command_args="run -confdir $confdir"
command_user="root"

# 设置 PID 文件路径
pidfile="/run/xray.pid"
command_background=true

# 定义额外的命令
extra_commands="checkconfig"

# 定义依赖项
depend() {
    need net
}

# 检查配置文件是否存在并有效
checkconfig() {
    if [ ! -f "$confdir/config.json" ]; then
        eerror "You need to set up $confdir/config.json first."
        return 1
    fi
    if [ ! -f "$XRAY_LOCATION_ASSET/geoip.dat" ]; then
        eerror "GeoIP database 'geoip.dat' not found in $XRAY_LOCATION_ASSET."
        return 1
    fi
    export $env
    $command $command_args -test
}

# 启动前检查配置文件
start_pre() {
    export XRAY_LOCATION_ASSET="/root/xray"
    checkconfig
}

# 启动服务
start() {
    ebegin "Starting $name"
    start_pre
    start-stop-daemon --start --background --exec $command -- $command_args
    eend $?
}

# 停止服务
stop() {
    ebegin "Stopping $name"
    start-stop-daemon --stop --exec $command
    eend $?
}
```

### 管理xray

```
Enable
# rc-update add xray

Disable
# rc-update del xray

Start
# rc-service xray start

Stop
# rc-service xray stop

Restart
# rc-service xray restart
```

## 系统服务命令

```
安装包
apk add ...

启动/停止/重启 已有服务
rc-service 服务名 start/stop/restart

列出所有可用服务
rc-service --list

服务状态
rc-status
```

## 启动xray

将config.json放入/root/xray的文件夹内使用

```
#启动
rc-service xray start

# 可以查看运行状态
rc-status
```

## 替换并自动更新 geo

创建更新脚本：

```\#
mkdir /usr/local/etc/xray-script
# 打开并编辑更新脚本
vim /usr/local/etc/xray-script/update-dat.sh
```

脚本内容：

```
#!/usr/bin/env bash

set -e

XRAY_DIR="/usr/local/share/xray"

GEOIP_URL="https://github.com/Loyalsoldier/v2ray-rules-dat/raw/release/geoip.dat"
GEOSITE_URL="https://github.com/Loyalsoldier/v2ray-rules-dat/raw/release/geosite.dat"

[ -d $XRAY_DIR ] || mkdir -p $XRAY_DIR
cd $XRAY_DIR

curl -L -o geoip.dat.new $GEOIP_URL
curl -L -o geosite.dat.new $GEOSITE_URL

rm -f geoip.dat geosite.dat

mv geoip.dat.new geoip.dat
mv geosite.dat.new geosite.dat

rc-service xray restart
```

赋予可执行权限：

chmod +x /usr/local/etc/xray-script/update-dat.sh
可以输入以下命令先手动执行一次：

/usr/local/etc/xray-script/update-dat.sh
确认没有问题后使用 crontab 设置定期执行：

执行 crontab -e；
选择使用 vim 打开；
在文件末尾追加 00 23 \* \* 1 /usr/local/etc/xray-script/update-dat.sh >/dev/null 2>&1
说明：可自行决定更新时间，示例中为每周一 23 点执行（注意服务器时区）。

# Warp 实现ipv4出口

项目
https://gitlab.com/fscarmen/warp

```
安装bash
apk add bash

下载并运行脚本
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh

选择为 IPv6 only 添加 WARP IPv4 网络接口 (bash menu.sh 4)
```

# bbr加速

```
echo “tcp_bbr” >> /etc/modules && modprobe tcp_bbr
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
sysctl -p
bbr已经开启成功，可以使用以下方式查看模块：

lsmod | grep bbr
```

# 参考

* https://tabsp.com/posts/vless-reality-vision/
* https://linux.do/t/topic/151670
* https://blog.passall.us/archives/1300

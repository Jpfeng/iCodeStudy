# Shadowsocks

> @ Ubuntu 16.04 LTS

## 部署

### 安装 Shadowsocks

```shell
apt-get install python-pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

### 安装 libsodium

如果使用 `salsa20` 或 `chacha20` 加密方法，需要安装 `libsodium` >= 1.0.0 。

```shell
apt-get install build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.15/libsodium-1.0.15.tar.gz
tar xf libsodium-1.0.15.tar.gz && cd libsodium-1.0.15
./configure && make -j2
sudo make install
sudo ldconfig
```

### 添加配置文件

创建 `/etc/shadowsocks.json` ，填入配置内容。

```json
{
    "server":"[server-ip]",
    "server_port":443,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"[password]",
    "timeout":15,
    "method":"chacha20",
    "fast_open": false,
    "log-file":"/dev/null"
}
```

### 开启实例

开启前台任务：

```shell
ssserver -c /etc/shadowsocks.json
```

开启后台任务：

```shell
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

### 将 Shadowsocks 添加为服务

创建脚本 `/etc/init.d/shadowsocks` ：

```shell
vim /etc/init.d/shadowsocks
```

填入以下内容：

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          shadowsocks
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: start shadowsocks
# Description:       start shadowsocks
### END INIT INFO

start(){
    ssserver -c /etc/shadowsocks.json -d start
}

stop(){
    ssserver -c /etc/shadowsocks.json -d stop
}

case "$1" in
start)
    start
    ;;
stop)
    stop
    ;;
reload)
    stop
    start
    ;;
*)
    echo "Usage: $0 {start|reload|stop}"
    exit 1
    ;;
esac
```

增加这个文件的可执行权限：

```shell
chmod +x /etc/init.d/shadowsocks
```

可以在 shell 中直接运行命令来控制 Shadowsocks 了。

```shell
service shadowsocks {start|reload|stop}
```

### 添加开机自启动

打开 `/etc/rc.d/rc.local`

```shell
vim /etc/rc.d/rc.local
```

在最后添加：

```shell
service shadowsocks start
```

## 优化

### 优化内核参数

打开 `/etc/sysctl.conf`

```shell
vim /etc/sysctl.conf
```

添加如下内容：

```shell
# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for low-latency network
net.ipv4.tcp_congestion_control = htcp
# forward ipv4
net.ipv4.ip_forward = 1
```

保存生效

```shell
sysctl -p
```

其中， `net.ipv4.tcp_congestion_control = htcp` 配置 `htcp` 为 TCP 拥塞控制算法，适用于低延迟的网络，可以显著提高速度。需要通过如下命令开启：

```shell
modprobe tcp_htcp
```

对于高延迟网络，可以使用 `hybla` 算法，需要内核支持。通过如下命令测试内核是否支持：

```shell
sysctl net.ipv4.tcp_available_congestion_control
```

如果输出中有 `hybla` ，则证明内核已开启 `hybla` 。否则，可以通过如下命令开启：

```shell
modprobe tcp_hybla
```

然后在 `/etc/sysctl.conf` 中将 TCP 拥塞控制算法配置为 `hybla` ：

```shell
net.ipv4.tcp_congestion_control = hybla
```

### TCP 优化

打开 `/etc/security/limits.conf`

```shell
vim /etc/security/limits.conf
```

加入如下内容：

```shell
*   soft    nofile  512000
*   hard    nofile  1024000
```

打开 `/etc/profile`

```shell
vim /etc/profile
```

加入如下内容：

```shell
ulimit -SHn 1024000
```

重启服务器并执行

```shell
ulimit -n
```

输出返回 `1024000` 即可。

### TCP Fast Open

适用于服务器端与客户端的系统内核版本均是 Linux 3.7+ 。可以通过如下命令查看内核版本：

```shell
uname -r
```

若符合条件，则服务端与客户端均需进行如下配置：

打开 `/etc/sysctl.conf`

```shell
vim /etc/sysctl.conf
```

加入如下内容：

```shell
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
```

打开 `/etc/shadowsocks.json`

```shell
vim /etc/shadowsocks.json
```

将文件中 `"fast_open": false` 改为

```shell
"fast_open": true
```

### 安装 gevent

安装 `gevent` 可以提高 Shadowsocks 的性能。执行如下命令：

```shell
apt-get install python-dev libevent-dev python-setuptools python-gevent
easy_install greenlet gevent
```

### 防火墙设置

自动调整 MTU

```shell
iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

开启 NAT 。将 `eth0` 改为自己的网卡名

```shell
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

打开 443 端口

```shell
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT -p udp --dport 443 -j ACCEPT
```

重启防火墙 iptables

```shell
service iptables restart
```

### 使用 TCP-BBR

开启 TCP-BBR 需要系统内核版本为 Linux 4.9+ 。查看内核版本：

```shell
uname -r
```

若内核版本不符合要求，可以在这里选择最新的内核版本：

> [Linux 内核](http://kernel.ubuntu.com/~kernel-ppa/mainline/)

下载最新的内核

```shell
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.13.9/linux-image-4.13.9-041309-generic_4.13.9-041309.201710211231_amd64.deb
```

安装内核

```shell
dpkg -i linux-image-4.*.deb
```

*\*可选\** 删除旧内核

```shell
dpkg -l | grep linux-image
apt-get purge [旧内核]
```

更新 grub 系统引导文件并重启

```shell
update-grub
reboot
```

开机后查看内核版本，若为 4.9+ 则支持 TCP-BBR 。执行

```shell
lsmod | grep bbr
```

如果结果中没有 `tcp_bbr` 则需要先执行

```shell
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```

然后执行

```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

保存生效

```shell
sysctl -p
```

最后执行

```shell
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```

如果结果都有 `bbr` , 则证明内核已开启 bbr 。执行

```shell
lsmod | grep bbr
```

若有 `tcp_bbr` 模块即说明 bbr 已启动

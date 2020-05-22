# CentOS配置SS和SSR和BBR
- 环境：CentOS 6

## 配置SS
- shadowsocks服务端[github网址](https://github.com/shadowsocks/shadowsocks/tree/master)
- windows客户端[github网址](https://github.com/shadowsocks/shadowsocks-windows)
- android客户端[github网址](https://github.com/shadowsocks/shadowsocks-android)

### 1.更新源和安装组件
```
yum update
yum install vim
yum install python-setuptools && easy_install pip
```

### 2.下载安装ss
- 通过github下载并安装：
```
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

- 通过yum源下载并安装：
```
pip install shadowsocks
```

- 安装完成后会在系统中生成名为ssserver的服务，可通过这个服务控制ss。
- 可通过`ssserver -h`选项查看所有选项。

### 3.使用方法
- 写配置文件：
```
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

> 文件位置：`/etc/shadowsocks.json`

- 前台运行
```
ssserver -c /etc/shadowsocks.json
```
- 后台运行
```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```
- 日志位置：`/var/log/shadowsocks.log`，可通过`less /var/log/shadowsocks.log`查看。

### 4.可能出现的问题
#### (1)pip安装ss报错
- 通过pip安装shadowsocks时可能出错：
```
_blocking_errnos = {errno.EAGAIN, errno.EWOULDBLOCK} pip
```
- 这时需要手动从github获取pip并安装：
```
wget  https://raw.githubusercontent.com/pypa/get-pip/master/2.6/get-pip.py
python2.6 get-pip.py
```

> 原问题[网址](https://blog.csdn.net/li740207611/article/details/86609917)

#### (2)服务器iptables阻止访问特定端口
- 解决方法：打开对应端口的权限，并保存iptables设置
```
iptables -I INPUT -p tcp --dport 8388 -j ACCEPT
/etc/init.d/iptables save
```
- 重启ss服务：
```
ssserver -c /etc/shadowsocks.json -d restart
```

## 配置SSR
- shadowsocks-r[服务端](https://github.com/shadowsocksrr/shadowsocksr)
- ssr windows[客户端](https://github.com/shadowsocksrr/shadowsocksr-csharp)

### 1.安装组件
```
yum install git
git clone https://github.com/shadowsocksrr/shadowsocksr.git
```

### 2.使用方法
- 进入ssr根目录运行初始化脚本：
```
cd ~/shadowsocksr
bash initcfg.sh
```
- 写配置文件：
```
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,

    "password": "m",
    "method": "aes-128-ctr",
    "protocol": "auth_aes128_md5",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth_compatible",
    "obfs_param": "",
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```
- 设置多端口：
```
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    //"server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
	
    "port_password":{
    	"8388":"password1",
    	"8389":"password2",
    	"8390":"password3"	
    },
    //"password": "m",
    "method": "aes-128-ctr",
    "protocol": "auth_aes128_md5",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth_compatible",
    "obfs_param": "",
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "additional_ports" : {}, // only works under multi-user mode
    "additional_ports_only" : false, // only works under multi-user mode
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```
> 配置文件位置：`~/shadowsocksr/user-config.json`

- 进入ssr根目录中的shadowsocks目录，执行其他操作,这些操作默认使用前面的配置文件。
- 启动服务前台运行：
```
python server.py
/*或使用脚本*/
./run.sh
```
- 后台运行：
```
python server.py -d start
python server.py -d restart
/*或使用脚本*/
./logrun.sh
```
- 停止运行：
```
python server.py -d stop
/*或使用脚本*/
./stop.sh
```
- 后台运行时查看日志：
```
tail -f /var/log/shadowsocksr.log
/*或使用脚本*/
./tail.sh
```
- 其中ssr服务器可以兼容ss客户端，只需要将配置文件中`"protocol"`字段设为`origin`,`"obfs"`字段设为`plain`，其他字段匹配即可。

### 3.可能出现的问题
#### (1)服务器iptables阻止访问特定端口
- 解决方法：参照ss。

### 4.可用的BBR加速
- BBR("Bottleneck Bandwidth and RTT")为google开发的针对linux内核tcp栈的阻塞控制算法。
- BBR[github网址](https://github.com/google/bbr)。
- BBR安装脚本[github网址](https://github.com/teddysun/across)。
- Ubuntu，CentOS 6，CentOS 7的BBR安装[网址](https://www.hi-linux.com/posts/64279.html)。
- 搬瓦工vps提供一些系统版本安装好BBR的服务器。

## 配置BBR
- 系统支持：CentOS 6+，Debian 7+，Ubuntu 12+
- 虚拟技术：OpenVZ 以外的，比如 KVM、Xen、VMware 等
- 内存要求：≥128M

### 关于脚本
- 本脚本已在Vultr上的VPS全部测试通过。
- 当脚本检测到 VPS 的虚拟方式为 OpenVZ 时，会提示错误，并自动退出安装。
- 脚本运行完重启发现开不了机的，打开 VPS 后台控制面板的 VNC, 开机卡在 grub 引导, 手动选择内核即可。

### 使用方法
- [脚本内容](https://github.com/tzz1996/Blog/blob/master/bbr.sh)
- 使用root用户登录，运行以下命令：
```
wget --no-check-certificate https://github.com/tzz1996/Blog/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```
- 安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
- 重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令，显示为最新版本表示成功。
```
uname -r
```
- 使用以下命令检查bbr安装情况：
```
sysctl net.ipv4.tcp_available_congestion_control
```
> 返回值一般为：net.ipv4.tcp_available_congestion_control = bbr cubic reno
> 或者为：net.ipv4.tcp_available_congestion_control = reno cubic bbr
```
sysctl net.ipv4.tcp_congestion_control
```
> 返回值一般为：net.ipv4.tcp_congestion_control = bbr
```
sysctl net.core.default_qdisc
```
> 返回值一般为：net.core.default_qdisc = fq
```
lsmod | grep bbr
```
> 返回值有 tcp_bbr 模块即说明 bbr 已启动。
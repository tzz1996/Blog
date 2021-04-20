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
> CentOS 7和CentOS 8的yum源可能不包含python-setuptools包，或提供python2或python3的更高版本  
> `CentOS 7/8`可通过`yum install python2/python3`来安装python库，和`yum install python2-setuptools`  
> 但`CentOS 7/8`不能运行对应的`easy_install pip`命令  
> 需要手动通过`wget https://bootstrap.pypa.io/pip/2.7/get-pip.py`来下载对应版本的`get-pip.py`文件  
> 然后通过`python/python2/python3 get-pip.py`命令来安装pip

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
> 现网址已更新为`https://bootstrap.pypa.io/pip/2.6/get-pip.py`，其中`2.6`更换为对应版本  
> `python2.6`命令需要根据具体系统`/usr/bin`目录下的python可执行文件名来确定命令和对应的python版本

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

#### (2).1 CentOS 7/8没有service iptables save命令问题
- 对应情况：`service iptables save`出现报错：
```
The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
```
- 解决方法：关闭防火墙
```
systemctl stop firewalld
```
- 通过yum源安装iptables服务：
```
yum install iptables-services
```
- 设置iptables为使能：
```
systemctl enable iptables
```
- 打开iptables：
```
systemctl start iptables
```
- 随后执行`service iptables save`便可保存iptables策略
- 重启iptables服务：
```
service iptables restart
```
> 配置完成后会生成`/etc/syscofig/`目录下的`iptables`可执行文件

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
> 返回值一般为：`net.ipv4.tcp_available_congestion_control = bbr cubic reno`
> 或者为：`net.ipv4.tcp_available_congestion_control = reno cubic bbr`
```
sysctl net.ipv4.tcp_congestion_control
```
> 返回值一般为：`net.ipv4.tcp_congestion_control = bbr`
```
sysctl net.core.default_qdisc
```
> 返回值一般为：`net.core.default_qdisc = fq`
```
lsmod | grep bbr
```
> 返回值有 `tcp_bbr` 模块即说明 bbr 已启动。

# SSTap+SSR+UDPspeeder
- [SSTap](https://sourceforge.net/projects/sstap/)
- [UDPspeeder](https://github.com/wangyu-/UDPspeeder)

## 原理
- 主要原理是通过发冗余数据来对抗网络的丢包，发送冗余数据的方式支持FEC(Forward Error Correction)和多倍发包，其中FEC算法是Reed-Solomon。
- udpspeeder对ssr发出的udp包进行处理，此时ssr客户端被设置为只能收发udp包，再用sstap进行tcp和udp的合并。

## 使用方法
### 服务端配置
- 下载udpspeeder并解压：
```
wget https://github.com/wangyu-/UDPspeeder/releases/download/20180522.0/speederv2_binaries.tar.gz
tar zxvf speederv2_binaries.tar.gz
```

- 此时假设你服务器`ip`为`44.55.66.77`，有一个服务监听在`udp 7777`端口上，比如`ssr`，运行如下命令：
```
./speederv2_x86 -s -l0.0.0.0:4096 -r127.0.0.1:7777   -k"passwd"  -f2:4 --timeout 1 &
```
> -f2:4表示对每2个原始数据发送4个冗余包。-f2:4 和-f 2:4都是可以的，空格可以省略<br/>
> 3倍流量，但是引入的额外延迟更小(<=2ms)，适用与游戏场景<br/>
> `ps -aux |grep 4096`可查看后台运行的udpspeeder进程信息<br/>

### 客户端配置
- 下载[udpspeeder客户端](https://github.com/wangyu-/UDPspeeder/releases)，并进入到客户端目录（windows），运行：
```
speederv2.exe -c -l0.0.0.0:3333 -r44.55.66.77:4096 -k"passwd"  -f2:4 --timeout 1
```
> 现在在Windows上访问本机的3333即相当于访问VPS的7777端口，实现udp加速了。

- 下载sstap，添加`socks5`代理，`ip`设为本地回环`127.0.0.1`，端口设置为ssr的本地端口（默认为1080），附加路由设置为`vps`的`ip`。
- ssr客户端的服务端`ip`设置为`127.0.0.1`，端口设置为udpspeeder监听的`3333`，其他配置与ssr服务端相匹配，系统代理模式设置为直连，表示系统不通过ssr进行代理。
- 此时便可通过sstap代理来达到降低延迟效果。


# SSR+kcptun+udp2raw
- [kcptun](https://github.com/xtaci/kcptun)
- [udp2raw](https://github.com/wangyu-/udp2raw-tunnel)
- udp2raw在windows下运行需要依赖[winpcap库](https://www.winpcap.org/install/bin/WinPcap_4_1_3.exe)

## kcptun配置
### 服务端配置
- 通过[kcptun脚本](https://github.com/kuoruan/shell-scripts/blob/master/kcptun/kcptun.sh)配置服务端
> [其他配置网址](https://ssr.tools/588)
- 下载并运行`kcptun.sh`:
```
wget --no-check-certificate https://github.com/tzz1996/Blog/raw/master/kcptun.sh
chmod +x ./kcptun.sh
./kcptun.sh
```
> 由于`kcptun.sh`文件中写的是通过`python`命令来调用python库，但`CentOS 7/8`中对应的python命令为`python2/python3`  
> 需要修改`/usr/bin`目录下`python2/python3`的文件名为`python`即可，因为此版本`kcptun.sh`支持python2.6版本及以上  
> 需注意：修改`python`命令后需要重新运行`python get-pip.py`来重新编译连接pip命令，否则pip命令无法正常执行

- 配置文件`config.json`如下：
```
{
    "listen": ":4000",
    "target": "127.0.0.1:8388",
    "key": "123456",
    "crypt": "aes",
    "mode": "fast2",
    "mtu": 1200,
    "sndwnd": 512,
    "rcvwnd": 512,
    "datashard": 10,
    "parityshard": 3,
    "dscp": 0,
    "nocomp": true,
    "quiet": false,
    "tcp": false,
    "pprof": false
}
```
> 其中`"target"`为ssr服务端监听端口，`"listen"`为kcptun服务端监听端口，其他参数请参考官方项目文档

#### kcptun常用功能及命令：
- KCPTUN安装目录：`/usr/local/kcptun`
- KCPTUN的参数配置文件：`/usr/local/kcptun/server-config.json`
- 启动：`supervisorctl start kcptun`
- 停止：`supervisorctl stop kcptun`
- 重启：`supervisorctl restart kcptun`
- 状态：`supervisorctl status kcptun`
- 卸载：`./kcptun.sh uninstall`

### 客户端配置
- 通过[下载地址](https://github.com/xtaci/kcptun/releases)下载对应系统和处理器架构的客户端软件
- windows本地运行：
```
client_windows_amd64.exe --localaddr :8888 --remoteaddr 66.42.80.107:29900 --key 123456 --crypt aes --mode fast2 --mtu 1200 --sndwnd 512 --rcvwnd 512 --datashard 10 --parityshard 3 --dscp 0 --nocomp true --quiet false --tcp false
```
> 其中的参数配置和服务端参数匹配<br/>
> `--localaddr :8888`为kcptun客户端为ssr客户端提供的服务端口，`--remoteaddr 66.42.80.107:29900`为服务端ip和kcptun服务端监听端口

- ssr中服务器ip改为`127.0.0.1`，服务器端口改为`8888`，其余参数与ssr服务端参数匹配
- 至此形成一条`ssr client`<->`kcptun client`<---->`kcptun server`<->`ssr server`的数据隧道

## udp2raw配置
### 服务端配置
- 下载udp2raw并解压：
```
wget https://github.com/wangyu-/udp2raw-tunnel/releases/download/20181113.0/udp2raw_binaries.tar.gz
tar zxvf udp2raw_binaries.tar.gz
```
- 运行udp2raw服务端：
```
./udp2raw_x86 -s -l0.0.0.0:8855 -r 127.0.0.1:4000 -k "passwd" --raw-mode faketcp -a &
```
> 其中`4000`为kcptun服务端监听端口，`8855`为udp2raw服务端监听端口

### 客户端配置
- windows下安装winpcap库
- 下载作者提供的windows版[udp2raw client](https://github.com/wangyu-/udp2raw-multiplatform)
- 通过udp2raw客户端提供的自动生成windwos开启端口代码功能打开对应权限：
```
udp2raw_mp_nolibnet.exe -c -r45.66.77.88:8855 -l0.0.0.0:4000 --raw-mode faketcp -k"passwd" -g
```
> `-g`选项将生成windows下的网络命令，手动运行即可
- 以下为windows生成的网络设置命令：
```
netsh advfirewall firewall add rule name=udp2raw protocol=TCP dir=in remoteip=66.42.80.107 remoteport=8855 action=block
netsh advfirewall firewall add rule name=udp2raw protocol=TCP dir=out remoteip=66.42.80.107 remoteport=8855 action=block
```
- 本地运行：
```
udp2raw_mp_nolibnet.exe -c -r45.66.77.88:8855 -l0.0.0.0:4000 --raw-mode faketcp -k"passwd"
```
> 其中`45.66.77.88:8855`为`vps ip`和udp2raw服务端监听端口

## 数据隧道配置
- vps上成功运行ssr
- vps上运行udp2raw服务端：
```
./udp2raw_x86 -s -l0.0.0.0:8855 -r 127.0.0.1:4000 -k "passwd" --raw-mode faketcp -a &
```
- 本地运行udp2raw客户端：
```
udp2raw_mp_nolibnet.exe -c -r45.66.77.88:8855 -l0.0.0.0:4000 --raw-mode faketcp -k"passwd"
```
> 如果一切正常`client`端输出显示`client_ready`,`server`端也会有类似输出,显示`server_ready`
- vps上运行kcptun服务端，监听端口设置为udp2raw服务端的远程端口`4000`
- 本地运行kcptun客户端：
```
client_windows_amd64.exe --localaddr :8888 --remoteaddr 127.0.0.1:4000 --key 123456 --crypt aes --mode fast2 --mtu 1200 --sndwnd 512 --rcvwnd 512 --datashard 10 --parityshard 3 --dscp 0 --nocomp true --quiet false --tcp false
```
> 其中`--remoteaddr 127.0.0.1:4000`为udp2raw客户端监听端口<br/>
> 并且`kcptun client`，`udp2raw client`，`kcptun server`， `udp2raw server`间的通信端口均设为`4000`
- ssr客户端连接kcptun监听的`8888`端口
- 此时完成了一条`ssr client`<->`kcptun client`<->`udp2raw client`<---->`udp2raw server`<->`kcptun server`<->`ssr server`的数据隧道
# CentOS配置SS和SSR
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
git clone https://github.com/shadowsocksr/shadowsocksr.git
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

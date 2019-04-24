#CentOS配置SS和SSR
- 环境：CentOS 6

##配置SS
- shadowsocks服务端[github网址](https://github.com/shadowsocks/shadowsocks/tree/master)
- windows客户端[github网址](https://github.com/shadowsocks/shadowsocks-windows)
- android客户端[github网址](https://github.com/shadowsocks/shadowsocks-android)

###1.更新源和安装组件
```
yum update
yum install python-setuptools && easy_install pip
```

###2.下载安装ss
```
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

- 安装完成后会在系统中生成名为ssserver的服务，可通过这个服务控制ss。
- 可通过`ssserver -h`选项查看所有选项。

###3.使用方法
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


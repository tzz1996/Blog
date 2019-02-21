# Ubuntu下配置wordpress
- 此服务器操作系统为Ubuntu 14.04.5 LTS。
- 使用的环境为php+MySQL+apache。
- 因为在wordpress使用过程中需要通过服务器搭建ftp服务器来进行插件或主题的下载安装，因此还涉及到vsftpd的安装。

## 配置步骤

### 1.安装apache服务器
```
sudo apt-get install apache2
sudo /etc/init.d/apache2 restart
```

- apache2的默认访问路径在`/var/www`中（此处为`/var/www/html`），修改方法略。
- 安装apache2，并重启，之后浏览器访问`http://localhost`,若出现apache的页面则apache2安装成功。

### 2.安装PHP
- 安装php5：`sudo apt-get install php5`
- 安装apache2的php5组件：`sudo apt-get install libapache2-mod-php5`
- 重启apache2使其加载php模块：`sudo /etc/init.d/apache2 restart`或`service apache2 restart`
- 检查php是否安装成功并与apache2关联：在`/var/www/html`中新建`index.php`，其中添加内容：
```
<?php
	phpinfo();
```

- 之后浏览器访问`http://localhost/index.php`，若显示有php信息，则安装成功。

### 3.安装MySQL
- 安装mysql服务器和客户端：`sudo apt-get install mysql-server mysql-client`
- 安装php的mysql关联：`sudo apt-get install php5-mysql`
- 重启apache2服务，并访问`http://localhost/index.php`，若mysql即关联安装成功则显示mysql信息。

### 4.安装php5-gd组件
- 安装php5-gd组件：`sudo apt-get install php5-gd`
- 重启apache2服务。
- 经验证，若没有安装php5-gd组件，则在wordpress中进行上传图片剪裁时会出现`There has been an error cropping your image`错误。（[原问题1](https://wordpress.org/support/topic/there-has-been-an-error-cropping-your-image-4/)，[原问题2](https://wordpress.org/support/topic/there-has-been-an-error-cropping-your-image-5/)）

### 5.下载wordpress并配置
- wordpress[中文官网](https://cn.wordpress.org/)，获取下载链接，此处为`https://cn.wordpress.org/wordpress-5.0.3-zh_CN.tar.gz`。
- 在需要存放的目录中运行：`wget https://cn.wordpress.org/wordpress-5.0.3-zh_CN.tar.gz`
> 或下载英文官方最新版：`wget http://wordpress.org/latest.tar.gz`

- 解压：`tar -zxvf wordpress-5.0.3-zh_CN.tar.gz`
- 移动到apache2的访问目录：`mv wordpress*/ /var/www/html`
- mysql中新建用于wordpress的数据库：
```
mysql -u root -p
sql>create database wordpress;
```
- 浏览器访问`http://localhost/wordpress/wp-admin/install.php`，进行安装。
> 可能出现的问题：
> 若安装或使用过程中出现写入权限的问题，则需要去服务器`/var/www/html/wordpress`中对应的文件或文件夹进行赋权操作
> ，如`sudo chmod 777 wordpress`。

### 6.安装vsftpd
- 新服务器可能需要更新一下源：`sudo apt-get update`
- 从源上安装vsftpd：`sudo apt-get install vsftpd`
- 将`/etc/vsftpd.conf`中的`write_enable`设置为`YES`。
- 将`/etc/ftpusers`中的`root`用户删除。
- 重启vsftpd服务：`service vsfptd restart`
- 本地登录已验证：`ftp localhost`

### 7.安装SSL
- 首先需要注册一个对应自己服务器ip地址的域名，如[freenom](https://my.freenom.com/clientarea.php)网站。
- 将这个域名添加到一个DNS提供商网站上，例如[cloudflare](https://www.cloudflare.com/)网站。在这个过程中会生成两个新的域名，需要到以freenom为例的[更新nameservers界面](https://my.freenom.com/clientarea.php?action=domains)去将两个新生成的nameserver替换进去。全部步骤完成后在cloudflare上会显示正常。
- 通过[certbot](https://certbot.eff.org/lets-encrypt/ubuntutrusty-apache)这里提供的步骤完成在服务器上安装ssl证书。其中需要注意的是，这个过程需要用到你所使用的DNS提供商的API key，以cloudflare为例的[获取API_key界面](https://dash.cloudflare.com/profile)这里获取，使用方法在[certbot提供的cloudflare插件安装文档](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)中。
> 需注意certbot只支持一些DNS提供商，并提供以下插件：
> [certbot-dns-cloudflare](https://certbot-dns-cloudflare.readthedocs.io/en/stable/),
> [certbot-dns-cloudxns](https://certbot-dns-cloudxns.readthedocs.io/en/stable/),
> [certbot-dns-digitalocean](https://certbot-dns-digitalocean.readthedocs.io/en/stable/),
> [certbot-dns-dnsimple](https://certbot-dns-dnsimple.readthedocs.io/en/stable/),
> [certbot-dns-dnsmadeeasy](https://certbot-dns-dnsmadeeasy.readthedocs.io/en/stable/),
> [certbot-dns-google](https://certbot-dns-google.readthedocs.io/en/stable/),
> [certbot-dns-linode](https://certbot-dns-linode.readthedocs.io/en/stable/),
> [certbot-dns-luadns](https://certbot-dns-luadns.readthedocs.io/en/stable/),
> [certbot-dns-nsone](https://certbot-dns-nsone.readthedocs.io/en/stable/),
> [certbot-dns-ovh](https://certbot-dns-ovh.readthedocs.io/en/stable/),
> [certbot-dns-rfc2136](https://certbot-dns-rfc2136.readthedocs.io/en/stable/),
> [certbot-dns-route53](https://certbot-dns-route53.readthedocs.io/en/stable/).

- 到此，已完成在服务器上安装ssl证书，需要把他安装到wordpress中，用wordpress中的ssl安装插件即可。
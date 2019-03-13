---
title: ss-panel配合shadowsocks-manyuser搭建多用户shadowsocks站点
date: 2017-05-30 04:51:22
tags:
---

>想看看shadowsocks多用户使用情况，然后发现这方面已经很成熟了。下面尝试走下这个成熟的流程

# SS-Panel前端站点搭建
>以下流程使用域名ss.alphaboom.cn 作为示例

## PHP环境搭建
配置Nginx环境，使用[lnpm一键安装包](https://blog.linuxeye.cn/31.html)搭建所需要的环境。
安装好环境之后再创建虚拟机，上面链接里都有操作步骤。（没有证书注意不要创建https的）
<!--more-->
## SS-Panel前端搭建
1.进入网站目录

```
cd /data/wwwroot/ss.alphaboom.cn/
```
2.下载代码

```
git clone https://github.com/orvice/ss-panel.git .
```

3.修改文件权限

```
find /data/wwwroot/ss.alphaboom.cn -type f -exec chmod 644 {} \;   #为了安全，将代码里面文件权限设置为644
find /data/wwwroot/ss.alphaboom.cn -type d -exec chmod 755 {} \;   #为了安全，将代码里面目录权限设置为755
chown -R www.www /data/wwwroot/ss.alphaboom.cn/
```
4.nginx 设置
>如果使用上面脚本默认的目录，那么配置文件就在/usr/local/nginx/conf/vhost/ss.alphaboom.cn.conf

在配置文件找到 root 那行 修改成如下形式

```
root /data/wwwroot/ss.alphaboom.cn/public;
```
然后再下方空白处加上如下代码

```
location / {
   try_files $uri $uri/ /index.php$is_args$args;
} 
```
5.导入数据库

```
mysql -u root -p
mysql> create database shadowsocks
mysql> use shadowsocks;
mysql> source db.sql;
mysql> flush privileges;
```
这个地方之后我有遇到`is not allowed to connect to this MySQL server`的错误，可以通过该表法

```
mysql -u root -p
mysql>use mysql;
mysql>update user set host = ‘%’ where user = ‘root’;
mysql>select host, user from user;
```

6.修改配置信息
先复制一份模板，为配置文件

```
cp .env.example .env
```
然后修改配置文件

```
vim .env
```
首先修改数据库内容

```
db_driver = 'mysql'
db_host = 'localhost'
db_port = '3306'
db_database = 'shadowsocks'
db_username = 'root'
db_password = 'password'
db_charset = 'utf8'
db_collation = 'utf8_general_ci'
db_prefix = ''
```
其他一些可以修改的部分

```
// !!! 修改此key为随机字符串确保网站安全 !!!
key = 'jdksjskdkjionqoskz'
env = 'prod'  // 正式环境请保持env为prod确保安全
debug =  'false'  //  正式环境请确保为false
appName = 'Anarchy ss'             //站点名称
baseUrl = 'http://ss.alphaboom.cn'  

// mu key 用于校验ss-go mu的请求
muKey = 'MyMuKey' //一会后端配置也会使用到     
```
7.安装第三方库

```
curl -sS https://getcomposer.org/installer | php
php composer.phar  install
```
8.添加管理员用户

```
php xcat createAdmin  
```
9.前端站点安装完毕
重新加载下Nginx，最好mysql也重启下
```
service nginx reload
service mysql restart
```
# Shadowsocks多用户后端shadowsocks-manyuser

## 获取源码
>[shadowsocks-rm](https://github.com/mengskysama/shadowsocks-rm/tree/manyuser)

1.随便找个目录下载源码

```
git clone -b manyuser https://github.com/mengskysama/shadowsocks-rm.git
```
2.安装pip包管理器

```
yum -y install python-setuptools && easy_install pip
yum install git -y
```
3.安装cymysql

```
pip install cymysql
```
4.进入文件夹

```
cd /shadowsocks-rm/shadowsocks	
```
5.修改配置文件

```
vim config.py
```

```
import logging

#Config
MYSQL_HOST = 'mengsky.net'//修改为前端配置的 这里应为 localhost
MYSQL_PORT = 3306
MYSQL_USER = 'root'
MYSQL_PASS = 'root'//这里修改为真实的密码
MYSQL_DB = 'shadowsocks'//前端配置的就是找个名称所以不用修改

MANAGE_PASS = 'passwd'
#if you want manage in other server you should set this value to global ip
MANAGE_BIND_IP = '127.0.0.1'
#make sure this port is idle
MANAGE_PORT = 23333

PANEL_VERSION = 'V3' # V2 or V3. V2 not support API
API_URL = 'http://domain/mu'//domain修改为自己设定的域名我这里为ss.alphaboom.cn
API_PASS = 'mupass'//修改为前端配置的mukey，我们之前设置叫MyMuKey
NODE_ID = '1'
CHECKTIME = 15
SYNCTIME = 600

#BIND IP
#if you want bind ipv4 and ipv6 '[::]'
#if you want bind all of ipv4 if '0.0.0.0'
#if you want bind all of if only '4.4.4.4'
SS_BIND_IP = '0.0.0.0'
SS_METHOD = 'rc4-md5'

#LOG CONFIG
LOG_ENABLE = False
LOG_LEVEL = logging.DEBUG
LOG_FILE = '/var/log/shadowsocks.log'
```
## 启动服务

先前台启动服务看有没有问题
```
python servers.py
```
如果有问题根据错误信息修改，然后关掉这个进程

```
ps -ef | grep servers.py
kill -9 查询到的pid
```
没有问题，后台开启服务
```
nohup python servers.py &
```

# 结束

http://ss.alphaboom.cn/ 完成

# 补充多节点的配置方式

多节点配置方式就是多个后端使用一个前端，具体步骤如下：

* 在新的服务器上安装Shadowsocks多用户后端shadowsocks-manyuser（参考上面的步骤），需要注意的地方是，数据库的配置。上面后台和前端都在同一个服务器上，这次会有些不同：

```
#Config
MYSQL_HOST = '104.238.135.97'#使用前端使用服务器的ip
MYSQL_PORT = 3306
MYSQL_USER = 'root'
MYSQL_PASS = 'password'
MYSQL_DB = 'shadowsocks'
```

这里是远程连接所以需要在前端的服务器上开启远程连接的权限

```
GRANT ALL PRIVILEGES ON `shadowsocks`.* TO 'test'@'66.66.66.66' IDENTIFIED BY 'qq123456' WITH GRANT OPTION;
```

在新的节点服务上就可以使用名为test，密码为qq123456的用户来操作名为shadowsocks数据库了（这个用户要对应，新节点的配置信息）
之后开放3306端口：

```
iptables -I INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
iptables-save
```

* 然后就可以在前端新增节点了。如果要记录新节点的流量信息要修改,新节点服务器的如下配置

```
API_URL = 'http://domain/mu'//domain修改为自己设定的域名我这里为ss.alphaboom.cn
API_PASS = 'mupass'//修改为前端配置的mukey，我们之前设置叫MyMuKey
NODE_ID = '1'#修改这个ID为新节点的id，增加了一个，所以这里应该改为2，前端后台也可以看到节点id值
CHECKTIME = 15
SYNCTIME = 600
#如果修改无效可以尝试放开下面的注释，默认应该是开启的
#API_ENABLED = True 
```

到这里节点配置完毕

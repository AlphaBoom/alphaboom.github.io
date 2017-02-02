---
title: 配置ShadowSocksR和使用BBR加速
date: 2017-01-13 03:07:18
tags:
---
>最近听说BBR加速效果挺好，正好之前vultr手滑充了10刀，试试速度有没有提升，vultr就是太慢才放弃的换搬瓦工的。搬瓦工又不能使用BBR加速

<!--more-->
## 在Vultr上创建实例
创建了一个日本节点的服务器ubuntu64，之前应该是死慢的 

## 安装ShaowSocksR
每次安装ShadowSocks都是随便找个一键脚本安装，至今不知道都安装了什么..下面试试手动安装...来到[shadowsocks-rss](https://github.com/breakwa11/shadowsocks-rss)进入wiki按照步骤来。新开的缺了好多环境 装python 装mysql 装cymysql  恩 mysql连接报错，貌似mysql配置错误，算了 我还是用一键脚本安装吧[用这个的脚本](https://shadowsocks.be/9.html)

```bash
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```
**卸载方法:**

```bash
./shadowsocksR.sh uninstall
```
安装完成后即已后台启动 ShadowsocksR ，运行：

```bash
/etc/init.d/shadowsocks status
```

可以查看 ShadowsocksR 进程是否已经启动。
本脚本安装完成后，已将 ShadowsocksR 自动加入开机自启动。

**使用命令：**

启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status

配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks

**多用户配置 sample：**

```json
{
"server":"0.0.0.0",
"server_ipv6": "[::]",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
    "8989":"password1",
    "8990":"password2"，
    "8991":"password3"
},
"timeout":300,
"method":"aes-256-cfb",
"protocol": "origin",
"protocol_param": "",
"obfs": "plain",
"obfs_param": "",
"redirect": "",
"dns_ipv6": false,
"fast_open": false,
"workers": 1
}
```
结束....还是一键脚本方便，虽然可能携带私货，只为翻个墙配置环境实在太浪费时间精力了，原谅我这个咸鱼。

## 测试下不加速的速度
先ping下服务器丢包率0延迟130-140，诶怎么看起来还不错，之前不用就是因为丢包率40%+延迟300-500啊，算了..难道这次是花的钱之前只是送的钱的缘故？？刷下youtube 1080p的视频 300-500kib/s 但是搬瓦工的1Mb/s 峰值接近2Mb/s ..这 是因为凌晨3点多的原因么，没人用。。算了继续测试vultr那台。

## 安装BBR
继续找[一建脚本](https://www.8dlive.com/post/415.html)看看
安装运行脚本

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

```bash
uname -r
```
查看内核版本，含有 4.9.0 就表示 OK 了

```bash
sysctl net.ipv4.tcp_available_congestion_control
```
返回值一般为：
`net.ipv4.tcp_available_congestion_control = bbr cubic reno`

```bash
sysctl net.ipv4.tcp_congestion_control
```
返回值一般为：
`net.ipv4.tcp_congestion_control = bbr`

```bash
sysctl net.core.default_qdisc
```

返回值一般为：
`net.core.default_qdisc = fq`

```bash
lsmod | grep bbr
```
返回值有 tcp_bbr 模块即说明bbr已启动。

感谢这些脚本的作者们，下面再来测试下速度吧哇2m-5m好吧应该是到我的带宽上线了我是50m联通的网

## 最后
先暂时删除日本服务器节点，之后可以考虑在搬瓦工到期之后继续用回Vultr的vps了

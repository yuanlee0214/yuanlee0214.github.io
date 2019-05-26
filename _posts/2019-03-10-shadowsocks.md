---
title: "Shadowsocks"
layout: post
date: 2019-3-10 15:00
image: #/assets/images/markdown.jpg
headerImage: false
tag:
- Shadowsocks
category: blog
author: #Yuan Lee
description: #Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="http://localhost:4000/assets/images/profile.jpg" height="20" width="20" align="absmiddle">'
---

## Summary

Shadowsocks 部署过程记录

#### 目录
- [Ⅰ.Shadowsocks Server](#shadowsocks-server)
- [Ⅱ.ChaCha20 Encryption](#chacha20-encryption)
- [Ⅲ.Installation of Pip3](#installation-of-pip3)

---


<h3 id="shadowsocks-server">一、Ubuntu 16.04 下 Shadowsocks服务器端安装及优化</h3>

<h4>前言</h4>

本教程旨在提供简明的Ubuntu 16.04下安装服务器端Shadowsocks。不同于Ubuntu 16.04之前的教程，本文抛弃initd，转而使用Ubuntu 16.04支持的Systemd管理Shadowsocks的启动与停止，显得更为便捷。优化部分包括BBR、TCP Fast Open以及吞吐量优化。

本教程仅适用于Ubuntu 16.04及之后的版本，基于Python 3，支持IPv6。

---

<h4>1.安装pip</h4>
本教程使用Python 3为载体，因Python 3对应的包管理器pip3并未预装，首先安装pip3：
{% highlight raw %}
sudo apt install python3-pip
{% endhighlight %}
PS：此步骤若无效则跳转至本文第三部分([点我立即跳转](#installation-of-pip3))查看具体安装步骤

<h4>2.安装Shadowsocks</h4>

因Shadowsocks作者不再维护pip中的Shadowsocks（定格在了2.8.2），我们使用下面的命令来安装最新版的Shadowsocks：
{% highlight raw %}
pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip
{% endhighlight %}

安装完成后可以使用下面这个命令查看Shadowsocks版本：
{% highlight raw %}
sudo ssserver --version
{% endhighlight %}

目前会显示“Shadowsocks 3.0.0”。

<h4>3.创建配置文件</h4>

创建Shadowsocks配置文件所在文件夹：
{% highlight raw %}
sudo mkdir /etc/shadowsocks
{% endhighlight %}

然后创建配置文件：sudo nano /etc/shadowsocks/config.json复制粘贴如下内容（注意修改密码“password”）：
{% highlight raw %}
{
"server":"::",
"server_port":8388,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"mypassword",
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}
{% endhighlight %}

然后按Ctrl + O保存文件，Ctrl + X退出。

<h4>4.测试Shadowsocks配置首先记录下服务器的IP地址</h4>
{% highlight raw %}
ifconfig
{% endhighlight %}

然后来测试下Shadowsocks能不能正常工作了：
{% highlight raw %}
ssserver -c /etc/shadowsocks/config.json
{% endhighlight %}

在Shadowsocks客户端添加服务器，地址填写IPv4地址或IPv6地址，端口号为8388，加密方法为aes-256-cfb，密码为设置的密码。然后设置客户端使用全局模式，浏览器登录Google试试应该能直接打开了。

这时浏览器登录<http://ip138.com>就会显示Shadowsocks服务器的IP啦！

测试完毕，按Ctrl + C关闭Shadowsocks。

<h4>5.配置Systemd管理Shadowsocks</h4>

新建Shadowsocks管理文件
{% highlight raw %}
sudo nano /etc/systemd/system/shadowsocks-server.service
{% endhighlight %}

复制粘贴：
{% highlight raw %}
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Ctrl + O保存文件，Ctrl + X退出。

启动Shadowsocks：
{% highlight raw %}
sudo systemctl start shadowsocks-server
{% endhighlight %}

设置开机启动Shadowsocks：
{% highlight raw %}
sudo systemctl enable shadowsocks-server
{% endhighlight %}

至此，Shadowsock服务器端的基本配置已经全部完成了！

<h4>6.优化</h4>

这部分属于进阶操作，在你使用Shadowsocks时感觉到延迟较大，或吞吐量较低时，可以考虑对服务器端进行优化。

开启BBR

BBR系Google最新开发的TCP拥塞控制算法，目前有着较好的带宽提升效果，甚至不比老牌的锐速差。

<h5>1) 升级Linux内核</h5>

BBR在Linux kernel 4.9引入。首先检查服务器kernel版本：
{% highlight raw %}
uname -r
{% endhighlight %}

如果其显示版本在4.9.0之下，则需要升级Linux内核，否则请忽略下文。

更新包管理器：
{% highlight raw %}
sudo apt update
{% endhighlight %}

查看可用的Linux内核版本：
{% highlight raw %}
sudo apt-cache showpkg linux-image
{% endhighlight %}

找到一个你想要升级的Linux内核版本，如“linux-image-4.18.0-16-generic”：
{% highlight raw %}
sudo apt install linux-image-4.18.0-16-generic
{% endhighlight %}

等待安装完成后重启服务器：
{% highlight raw %}
sudo reboot
{% endhighlight %}

删除老的Linux内核：
{% highlight raw %}
sudo purge-old-kernels
{% endhighlight %}

<h5>2) 开启BBR</h5>

运行
{% highlight raw %}
lsmod | grep bbr
{% endhighlight %}

如果结果中没有tcp_bbr，则先运行：
{% highlight raw %}
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
{% endhighlight %}

运行：
{% highlight raw %}
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
{% endhighlight %}

运行：
{% highlight raw %}
sysctl -p
{% endhighlight %}

保存生效。运行：
{% highlight raw %}
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
{% endhighlight %}

若均有bbr，则开启BBR成功。

<h5>3) 优化吞吐量</h5>

新建配置文件：
{% highlight raw %}
sudo nano /etc/sysctl.d/local.conf
{% endhighlight %}

复制粘贴：
{% highlight raw %}
# max open files
fs.file-max = 51200
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
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

net.ipv4.tcp_congestion_control = bbr
{% endhighlight %}

运行：
{% highlight raw %}
sysctl --system
{% endhighlight %}

编辑之前的shadowsocks-server.service文件：
{% highlight raw %}
sudo nano /etc/systemd/system/shadowsocks-server.service
{% endhighlight %}

在ExecStart前插入一行，内容为：
{% highlight raw %}
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
{% endhighlight %}

即修改后的shadowsocks-server.service内容为：
{% highlight raw %}
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Ctrl + O保存文件，Ctrl + X退出。

重载shadowsocks-server.service：
{% highlight raw %}
sudo systemctl daemon-reload
{% endhighlight %}

重启Shadowsocks：
{% highlight raw %}
sudo systemctl restart shadowsocks-server
{% endhighlight %}

<h5>4) 开启TCP Fast Open</h5>

TCP Fast Open可以降低Shadowsocks服务器和客户端的延迟。实际上在上一步已经开启了TCP Fast Open，现在只需要在Shadowsocks配置中启用TCP Fast Open。

编辑config.json：
{% highlight raw %}
sudo nano /etc/shadowsocks/config.json
{% endhighlight %}

将fast_open的值由false修改为true。

Ctrl + O保存文件，Ctrl + X退出。

重启Shadowsocks：
{% highlight raw %}
sudo systemctl restart shadowsocks-server
{% endhighlight %}

注意：TCP Fast Open同时需要客户端的支持，即客户端Linux内核版本为3.7.1及以上；你可以在Shadowsocks客户端中启用TCP Fast Open。
至此，Shadowsock服务器端的优化已经全部完成了！

<div class="breaker"></div>


<h3 id="chacha20-encryption">二、Ubuntu 16.04 安装libsodium库支持chacha20加密方式</h3>

<h4><span class="evidence">PS：此步骤并非必须步骤。</span></h4>
<h5>因为chacha20加密方式的安全性与aes-256-cfb相近，但效率比aes-256-cfb高，所以推荐启用chacha20加密。libsodium是给Shadowsocks提供chacha20、salsa20、chacha20-ietf等高级加密所必须的扩展库。</h5>

<h4>1.安装依赖</h4>

Debian 7/8、Ubuntu 14/15/16 及其衍生系列：
{% highlight raw %}
sudo apt-get update
sudo apt-get install build-essential wget -y
{% endhighlight %}

<h4>2.下载 libsodium 最新版本</h4>

可以从libsodium 官网下，也可以从github下载。选择速度最快的下载方式。

<h5>1) 从官网下载：</h5>
{% highlight raw %}
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
{% endhighlight %}

<h5>2) 从 github 下载（其中 1.0.10 是 libusodium 的版本号，可以改成最新的）:</h5>
{% highlight raw %}
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
{% endhighlight %}

<h4>3.解压</h4>

<h5>1) 官网下载的：</h5>
{% highlight raw %}
tar xzvf LATEST.tar.gz
{% endhighlight %}

<h5>2) github 下载的：</h5>
{% highlight raw %}
tar xzvf libsodium-1.0.10.tar.gz
{% endhighlight %}

<h4>4.生成配置文件</h4>
{% highlight raw %}
cd libsodium*
./configure
{% endhighlight %}

<h4>5.编译并安装</h4>
{% highlight raw %}
make -j8 && make install
{% endhighlight %}

<h4>6.添加运行库位置并加载运行库：</h4>
{% highlight raw %}
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
{% endhighlight %}

<h4>7.编辑之前的shadowsocks的config.json文件</h4>
{% highlight raw %}
sudo nano /etc/shadowsocks/config.json
{% endhighlight %}
将加密方式由之前的aes-256-cfb改为chacha20即可，即修改后的shadowsocks/config.json文件的内容为：
{% highlight raw %}
{
"server":"::",
"server_port":8388,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"mypassword",
"timeout":300,
"method":"chacha20",
"fast_open": true
}
{% endhighlight %}

<div class="breaker"></div>


<h3 id="installation-of-pip3">三、Ubuntu 16.04 安装pip3</h3>

系统本身自带python3

查看是否安装了pip3
{% highlight raw %}
pip3 -v
{% endhighlight %}

然而却出现了 E: Unable to locate package python3-pip 错误，原因很简单，就是自带的源没有找到python3-pip这个包，执行下面的语句更新源即可
{% highlight raw %}
# 更新一下源
sudo apt update
{% endhighlight %}

更新好源后，再次执行 sudo apt install python3-pip 便能下载安装了。
到这里pip算是装好，但如果你在执行 pip3 -V 后报以下异常的话，可以执行
{% highlight raw %}
sudo apt install python3-distutils
{% endhighlight %}
试试。

{% highlight raw %}
Traceback (most recent call last):
File "/usr/bin/pip3", line 9, in <module>
from pip import main
File "/usr/lib/python3/dist-packages/pip/__init__.py", line 14, in <module>
from pip.utils import get_installed_distributions, get_prog
File "/usr/lib/python3/dist-packages/pip/utils/__init__.py", line 23, in <module>
from pip.locations import (
File "/usr/lib/python3/dist-packages/pip/locations.py", line 9, in <module>
from distutils import sysconfig
ImportError: cannot import name 'sysconfig'
{% endhighlight %}
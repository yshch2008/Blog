---
date: "2019-4-30+08:00"
publishdate: "2019-4-30+08:00"
lastmod: "2019-4-30+08:00"
draft: false
title: "搭建VPS_谷歌云"
tags: ["修行", "科学上网", "云"]
series: ["技术"]
categories: ["学习"]
toc: true
---

# 申请免费谷歌云试用

1.登录Google Cloud官网并进行账户注册：<https://cloud.google.com/> ,我们在官网首页点击免费试用进行注册：

![谷歌云](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907114315.png)

2.我们进行国家和地区资料填写，我们在右侧栏可以看到12个月有效期的300刀赠额说明。国家地区选择真实所在地，这里我选择中国，勾选同意条款，点击同意并继续：

![谷歌云](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907114412.png)

3.我们接下来进行个人资料和信用卡资料的填写，个人资料和信用卡资料如实填写就好了。

![谷歌云](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907114509.png)

4.账号类型尽量选择个人，个人资料和信用卡信息填写完成之后直接点击开始免费试用选项即可。接着你的信用卡账户消费 1 美金，这是谷歌为了确认你不是机器人用于验证账户的，5 分钟内你会收到账户撤销 1 美金的消费操作，1 美元会返回你的账户，整个过程完全免费。另外除非你主动确认升级成付费账号，300 美金的赠款消费完毕以后并不会直接从你的信用卡账户续费扣款。

![谷歌云](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907114857.png)

5.我们完成付款之后登陆到Google Cloud的控制台，点击左上角的导航菜单按钮，我们选择结算进行赠送额度查看：

![img](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907114958.png)

6.我们进入到结算页面就可以看到相关的额度信息了，如果没有其它问题的话我们会看到赠金300刀以及有效期365天：

![谷歌云](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907115148.png)



# 设置服务器，VPC规则

- 我们直接点击http-server和https-server的名称进入到详细页面并点击修改按钮进行端口的设置，这里的端口号就是我们刚才安装完SSR和BBR之后获得的SSR的端口号，这里在输入的时候采用英文输入模式下进行输入，否则会出现报错。输入完成之后记得点击保存按钮。

 ![谷歌云安装SSR](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907122302.png)

 ![谷歌云安装SSR](https://www.freeluffy.com/wp-content/uploads/2018/09/20180907122421.png)



# yum安装Wget

yum -y install wget  

# 配置BBR加速

**什么是BBR：**

TCP BBR是谷歌出品的TCP拥塞控制算法。BBR目的是要尽量跑满带宽，并且尽量不要有排队的情况。BBR可以起到单边加速TCP连接的效果。

Google提交到Linux主线并发表在ACM queue期刊上的TCP-BBR拥塞控制算法。继承了Google“先在生产环境上部署，再开源和发论文”的研究传统。TCP-BBR已经再YouTube服务器和Google跨数据中心的内部广域网(B4)上部署。由此可见出该算法的前途。

TCP-BBR的目标就是最大化利用网络上瓶颈链路的带宽。一条网络链路就像一条水管，要想最大化利用这条水管，最好的办法就是给这跟水管灌满水。

BBR解决了两个问题：

- 在有一定丢包率的网络链路上充分利用带宽。非常适合高延迟，高带宽的网络链路。
- 降低网络链路上的buffer占用率，从而降低延迟。非常适合慢速接入网络的用户。

Google 在 2016年9月份开源了他们的优化网络拥堵算法BBR，最新版本的 Linux内核(4.9-rc8)中已经集成了该算法。

对于TCP单边加速，并非所有人都很熟悉，不过有另外一个大名鼎鼎的商业软件“锐速”，相信很多人都清楚。特别是对于使用国外服务器或者VPS的人来说，效果更佳。

BBR项目地址：

> <https://github.com/google/bbr>

**升级内核，第一步首先是升级内核到支持BBR的版本：**
1.yum更新系统版本：

> yum update

2.查看系统版本：

```bash
[root@server ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
[root@server ~]# 
```

3.安装elrepo并升级内核：

```bash
[root@server ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@server ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
[root@server ~]# yum --enablerepo=elrepo-kernel install kernel-ml -y
```

4.更新grub文件并重启系统：

```bash
[root@server ~]# egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux 7 Rescue 8619ff5e1306499eac41c02d3b23868e (4.14.14-1.el7.elrepo.x86_64)
CentOS Linux (4.14.14-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-693.11.6.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-c73a5ccf3b8145c3a675b64c4c3ab1d4) 7 (Core)
[root@server ~]# grub2-set-default 0
[root@server ~]# reboot
```

5.重启完成后查看内核是否已更换为4.14版本：

```bash
[root@server ~]# uname -r
4.14.14-1.el7.elrepo.x86_64
[root@server ~]#
```

6.开启bbr：

```bash
[root@server ~]# vim /etc/sysctl.conf    # 在文件末尾添加如下内容
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

7.加载系统参数：

```bash
[root@vultr ~]# sysctl -p
net.ipv6.conf.all.accept_ra = 2
net.ipv6.conf.eth0.accept_ra = 2
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
[root@vultr ~]#
```

如上，输出了我们添加的那两行配置代表正常。

8.确定bbr已经成功开启：

```bash
[root@vultr ~]# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
[root@vultr ~]# lsmod | grep bbr
tcp_bbr                20480  2 
[root@vultr ~]# 
```

输出内容如上，则表示bbr已经成功开启。

# 安装SSR

1. wget –no-check-certificate 
   https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
   && chmod +x shadowsocksR.sh





2. 安装完成后，需要创建shadowsocks的配置文件/etc/shadowsocks.json，编辑内容如下：

```bash
[root@server ~]# vim /etc/shadowsocks.json
{
  "server": "0.0.0.0",
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "port_password": {
    "8080": "填写密码",
    "8081": "填写密码"
  },
  "timeout": 600,
  "method": "aes-256-cfb"
}
```

说明：

- method为加密方法，可选aes-128-cfb, aes-192-cfb, aes-256-cfb, bf-cfb, cast5-cfb, des-cfb, rc4-md5, chacha20, salsa20, rc4, table
- port_password为端口对应的密码，可使用密码生成工具生成一个随机密码

以上两项信息在配置 shadowsocks 客户端时需要配置一致，具体说明可查看 shadowsocks 的帮助文档。

**如果你不需要配置多个端口的话，仅配置单个端口，则可以使用以下配置：**

```
{
  "server": "0.0.0.0",
  "server_port": 8080,
  "password": "填写密码",
  "method": "aes-256-cfb"
}
```

说明：

- server_port为服务监听端口
- password为密码

同样的以上两项信息在配置 shadowsocks 客户端时需要配置一致。

------

# 配置自启动

编辑shadowsocks 服务的启动脚本文件，内容如下：

```bash
[root@server ~]# vim /etc/systemd/system/shadowsocks.service
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```

执行以下命令启动 shadowsocks 服务：

```bash
[root@server ~]# systemctl enable shadowsocks
[root@server ~]# systemctl start shadowsocks
```

检查 shadowsocks 服务是否已成功启动，可以执行以下命令查看服务的状态：

> systemctl status shadowsocks -l

如果服务启动成功，则控制台显示的信息应该类似这样：
![CentOS7.4搭建shadowsocks，以及配置BBR加速](https://s1.51cto.com/images/blog/201801/24/bf2514ac76e124d605b5f6816e2cbd7a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

确认服务启动成功后，配置防火墙规则，开放你配置的端口，不然客户端是无法连接的：

```bash
[root@server ~]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
[root@server ~]# firewall-cmd --zone=public --add-port=8081/tcp --permanent
success
[root@server ~]# firewall-cmd --reload
success
```





# 设定 VPC IP 转发

VPC 网络=>default=>编辑=>动态传送模式设置为全域=>保存

#  负载均衡

网络服务=>负载平衡=>建立负载平衡=>UDP 负载平衡=>从网际网路到我的 VM=>继续

```
##后端设定
# 区域：刚才实例创建所选区域
# 后端：选择创建的现有实例

##前端设定
# IP: 临时
# 端口：500-4500
```

最后填入名称并建立。

## [SS下载](https://github.com/shadowsocks/shadowsocks-windows/releases)


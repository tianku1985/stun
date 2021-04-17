

一、安装stun：
相关组件下载参照：https://github.com/awe1p/stun

cd到openwrt源代码路径
git glone https://github.com/awe1p/stun package/network/stun
make menuconfig，选中新增的stun-client和stund，保存退出
make V=s
镜像编译并烧录后，多了两个指令：stund stun-client。使用方法参照官网所述：https://openwrt.org/docs/guide-user/services/voip/stund

二、环境搭建：
路由器A作为stun服务器接入公网Internet；B为待测NAT，上级接入公网Internet，下级连路由C（或安装了stun的pc）作为stun-client

三、测试指令：
A运行：stund -v & （当然也可以指定interface）
C运行：stun-client {A的公网ip} -v
此时c返回公网ip和NAT类型等详细数据:



根据返回结果可判断出7种类型：
（1）Open Internet
（2）UDP Blocked
（3）Symmetric Firewall
（4）Full Cone NAT
（5）Restricted Cone NAT
（6）Port Restricted Cone NAT
（7）Symmetrict NAT
其中根据后4种NAT的返回结果判断为：
Independent Mapping, Independent Filter = Fullcone NAT
Independent Mapping, Address Dependent Filter = Restricted Cone NAT
Independent Mapping, Port Dependent Filter = Port-Restricted Cone NAT
Dependent Mapping = Symmetric NAT

根据Independent Mapping, Independent Filter可判断此时我的NAT类型为完全锥形NAT
去掉B,C直连公网，会得到Firewall。关掉C的防火墙，会得到open。

（实现Fullcone NAT过程： https://blog.csdn.net/xuzhen5062/article/details/106386098）

四、stun测试原理简述：

STUN协议定义了三类测试过程来检测NAT类型：
Test1：
STUN Client通过端口{IP-c1:Port-c1}向STUN Server{IP-s1:Port-s1}发送一个Binding Request（没有设置任何属性）。STUN Server收到该请求后，通过端口{IP-s1:Port-s1}把它所看到的STUN Client的IP和端口{IP-m1,Port-m1}作为Binding Response的内容回送给STUN Client。
Test1(2)：
STUN Client通过端口{IP-c1:Port-c1}向STUN Server{IP-s2:Port-s2}发送一个Binding Request（没有设置任何属性）。STUN Server收到该请求后，通过端口{IP-s2:Port-s2}把它所看到的STUN Client的IP和端口{IP-m1#2,Port-m1#2}作为Binding Response的内容回送给STUN Client。
Test2：
STUN Client通过端口{IP-c1:Port-c1}向STUN Server{IP-s1:Port-s1}发送一个Binding Request（设置了Change IP和Change Port属性）。STUN Server收到该请求后，通过端口{IP-s2:Port-s2}把它所看到的STUN Client的IP和端口{IP-m2,Port-m2}作为Binding Response的内容回送给STUN Client。
Test3：
STUN Client通过端口{IP-c1:Port-c1}向STUN Server{IP-s1:Port-s1}发送一个Binding Request（设置了Change Port属性）。STUN Server收到该请求后，通过端口{IP-s1:Port-s2}把它所看到的STUN Client的IP和端口{IP-m3,Port-m3}作为Binding Response的内容回送给STUN Client。

顺序为：Test1、Tset2、Tset3、Tset1(2)

流程如下图所示：



根据之前的测试结果图也可以看出每种test的结果。 
原文链接：https://blog.csdn.net/xuzhen5062/article/details/106377911/


## stun extension for openwrt, which inclides stund and stun-client

Compile
---
```
# cd to OpenWrt source path
# Clone this repo
git clone https://github.com/awe1p/stun.git package/network/stun
# Select Network -> stun-client and Network -> stund
make menuconfig
# Compile
make V=s
```

Usage
---
```
# you should have at least two routers as server and client
# run in stun server with two interface connected to the Internet
stund -v
# run in stun client to get your NAT type in response
stun-client {server IP} -v
```
Result
---
```
when you get 
Independent Mapping, Independent Filter = Fullcone NAT
Independent Mapping, Address Dependent Filter = Restricted Cone NAT
Independent Mapping, Port Dependent Filter = Port-Restricted Cone NAT
Dependent Mapping = Symmetric NAT
```

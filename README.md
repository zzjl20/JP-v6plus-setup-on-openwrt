# 日本光回线v6plus设定步骤
记录nanopi安装openwrt系统，为v6plus类型光纤设置的教程。主要参考了[【OpenWrt】MAP-e(v6プラス)の設定全手順、PPPoEとの併用もできる](https://wifi-manual.net/enhikari-map-with-openwrt/)这篇文章.

本文将演示在v6plus类型的网络要如何配置openwrt.

## 前期确认
确认ISP给你的线路类型是v6plus.

确认路由器已经安装好openwrt，以下教程基于openwrt 24.10.2

确认路由器当前可以接通外网，并且有ipv6的地址。可以通过命令行格式下使用`opkg update`来确认，也可以在openwrt的luci界面找IPv6地址。

<img src="/pics/pic0.png" width="600">
<img src="/pics/pic1.png" width="600">
并非所有操作都可以从luci完成，需要terminal。如果没有命令行,也可以在luci中安装`luci-app-ttyd`

本文的路由器使用Nanopi R4S.

为了隐私，将真实ip地址替换为ipv6=2404:1234:5678:90ab:xxxx:xxxx:xxxx:xxxx/64
## 准备工作

### step1 计算MAP-E
在openwrt配置完成之前无法使用浏览器。因此在配置之前需要提前利用下面网站来计算后面所需要的参数。

https://ipv4.web.fc2.com/map-e.html

只需把ipv6地址填入第一行，然后点击“计算”即可，得到的结果保存为图片后用。
<img src="/pics/pic1.png" width="600">
## 正式设置步骤

### step2 安装map
System -> Software, Update lists... -> 在Filter:中填入"map",找到"map"包-> 点击"Install..."。安装完成后，原先“install...”按钮会变成灰色的“Installed”按钮. **注意:** 完成后重启一次这个包才能生效, System -> Reboot.
<img src="/pics/pic1.png" width="600">
V6plus类型将IPv4的地址打包进IPv6格式中，Map包的作用，是解析v6plus的地址，这样才能从中解析出来IPv4的地址。成功之后协议类型会多出一个“MAP/LW4over6.”
<img src="/pics/pic1.png" width="600">
<img src="/pics/pic1.png" width="600">
### step3 配置wan6口

### step4 配置lan口
### step5 进行MAP-E映射

















### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6


#### 标题（用底线的形式）Heading (underline)

This is an H1
=============

This is an H2
-------------

### 字符效果和横线等

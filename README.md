# 日本光回线v6plus设定步骤
记录nanopi安装openwrt系统，为v6plus类型光纤设置的教程。主要参考了[【OpenWrt】MAP-e(v6プラス)の設定全手順、PPPoEとの併用もできる](https://wifi-manual.net/enhikari-map-with-openwrt/)这篇文章.

本文将演示在v6plus类型的网络要如何配置openwrt.

## 前期确认
确认ISP给你的线路类型是v6plus.

确认路由器已经安装好openwrt，以下教程基于openwrt 24.10.2

确认路由器当前可以接通外网，并且有ipv6的地址。可以通过命令行格式下使用`opkg update`来确认，也可以在openwrt的luci界面找IPv6地址。

<img src="/pics/pic0.png" width="600">
<img src="/pics/pic2.png" width="600">
并非所有操作都可以从luci完成，需要terminal。如果没有命令行,也可以在luci中安装`luci-app-ttyd`

本文的路由器使用Nanopi R4S.

为了隐私，将真实ip地址替换为ipv6=2404:1234:5678:90ab:xxxx:xxxx:xxxx:xxxx/64
## 准备工作

### step1 计算MAP-E
在openwrt配置完成之前无法使用浏览器。因此在配置之前需要提前利用下面网站来计算后面所需要的参数。

https://ipv4.web.fc2.com/map-e.html

只需把ipv6地址填入第一行，然后点击“计算”即可，得到的结果保存为图片后用。
<img src="/pics/pic3.png" width="600">
## 正式设置步骤

### step2 安装map
System -> Software, Update lists... -> 在Filter: 中填入"map",找到"map"包-> 点击Install...。安装完成后，原先“install...”按钮会变成灰色的“Installed”按钮. **注意:** 完成后重启一次这个包才能生效, System -> Reboot.
<img src="/pics/pic4.png" width="600">

V6plus类型网络将IPv4的地址打包进IPv6格式中，map包的作用，是解析v6plus的地址，这样才能从中解析出来IPv4的地址。map包安装成功之后协议类型会多出一个“MAP/LW4over6.”

<img src="/pics/pic5.png" width="600">
<img src="/pics/pic6.png" width="600">

### step3 配置wan6口
选择Internet -> Interface, 来到接口菜单。此时未经任何配置的接口应该是这样的。

<img src="/pics/pic7.png" width="600">


| 接口名 | 说明 |
|-------|-----|
| lan | 本地局域网接口|
| wan | 外部ISP分配的IPv4接口，但是v6plus类型网络没有IPv4网址| 
| wan6 | 外部ISP分配的IPv6接口|

首先将外部接入的v6plus解析。可以新建一个新接口(Add new interface...)，也可用现有接口修改一下配置。使用已有的wan6口 -> Edit -> DHCP Server -> Setup DHCP Server -> IPv6 Settings. 按照下图更改。

<img src="/pics/pic8.png" width="600">

| 选项 | 状态 | 说明 |
|--|--|--|
| Designated master | ☑️ | 告诉openwrt此接口为主通路|
| RA-Service | relay mode| |
| DHCPv6-Service | relay mode| |
| NDP-Proxy | relay mode| |
| Learn routes| ☑️ |  |

点击Save返回接口界面。
### step4 配置lan口
同样要把局域网也设置一下关于IPv6相关的功能。在lan口中， -> Edit -> DHCP Server -> IPv6 Settings, 按照如下图更改

<img src="/pics/pic9.png" width="600">

| 选项 | 状态 |
|--|--|
| RA-Service | relay mode|
| DHCPv6-Service | Server mode|
| NDP-Proxy | relay mode|
| Learn routes| ☑️ |

按Save保存更改，回到接口界面。 此时wan6接口和lan口都做了一些更改，但是还没有生效。选择Save & Apply，将配置生效。

如果一切没有问题，那么此时电脑已经可以通过浏览器浏览一些IPv6网站，比如Youtube和Google都支持IPv6访问。

### step5 进行MAP-E映射
但是IPv4仍然无法使用。因为我们还为将v6plus网址中的IPv4地址解析出来。此时的wan6接口应该如下：

<img src="/pics/pic10.png" width="600">
IPv6 地址之下应该还有一个IPv6 PD参数，但此时没有。据说开办v6plus业务同时也开通光电话，那么就会自动出现这个参数，那就可以跳过这一步，到下一步修改wan接口部分。但是如果没有，就需要我们手动加上了。这一步无法在luci界面中完成。

使用ssh工具进入路由器，输入如下代码：
```
uci set network.wan6.ip6prefix="2024:1234:5678:90ab::/64"
uci show network.wan6.ip6prefix
uci commit network
/etc/init.d/network reload
```
其中，**wan6**是接口名，如果主通道是自己新建的接口，那么替换为新接口名。

**2024:1234:5678:90ab::/64** 为IPv6地址的前16位，最后的 **/64** 不可少。

此时再回到luci界面刷新一下，就可以看到出现了IPv6 PD的字样。

<img src="/pics/pic11.png" width="600">

有了PD信息，就可以把IPv4地址解析出来了。可以新建一个接口，也可以在当前接口上进行改造。选择wan -> Edit -> General Settings，在Protocal中选择MAP/LW4over6协议 -> Save 。

### 然后在General Settings选项中按照下图修改：
<img src="/pics/pic12.png" width="600">

那么BR/DMR/AFTR等这些项目，具体填什么内容呢？还记得前面Step1计算的MAP-E的内容吗，此时正派上用场。

<img src="/pics/pic13.png" width="600">
<img src="/pics/pic3.png" width="600">

| 选项 | 内容|
|--|--|
| Protocal | MAP/LW4over6|
| Type | MAP-E|
| BR/DMR/AFTR | option peeraddr|
| IPv4 prefix| option ipaddr | 
| IPv4 prefix length | option ip4prefixlen|
| IPv6 prefix | option ip6prefix|
| IPv6 prefix length | option ip6prefixlen|
| EA-Bits length| option ealen | 
| PSID-bits length | psidlen|
| PSID offset | offset|

General Settings 修改完毕。

### Advanced Settings 中如下修改。
<img src="/pics/pic14.png" width="600">

### Firewall Settings 中如下修改。
<img src="/pics/pic15.png" width="600">

至此，wan6修改完毕，按Save键保存并返回接口窗口。

这样，所有修改已经完成。选择Save & Apply,让修改生效。刷新界面后，会多出一个灰色接口，这是由v6plus地址经由MAP-E转换后自动生成的接口，如果不接入wan口接受不到v6plus地址，则不会生成这个接口。而且lan口下会多出一个IPv6地址，原先的IPv6是管本地环路的，新生成的这个地址才是上网用的。

<img src="/pics/pic16.png" width="600">

至此，电脑已经可以经由路由器正常上网。


title: "校园网 + 华硕 RT-AC66U 配置 IPv6"
date:  2016-08-05 12:00:00
---
从北五环搬到本部的宿舍之后，使用网络遇到了不少问题：

 1. 校园网每个 IP 必须并且只需登录一次，因此入户线插在路由器 WAN 口上就会导致一个人登录后所有人共享同一个账号。解决方法是关掉路由器的 DHCP 服务器，然后把入户线插到 LAN 口上，让路由器变成一台交换机，把 DHCP 交给上游来处理，这样路由器下面的每台设备都有了独立的外网 IP；
 2. 然而校园网账号同时只能在两个 IP 上登录，因此像我这种设备多如牛毛的人就必须再买台路由器挂在交换机的下面，所有设备共享一个外网 IP，只需要登录一个账号即可；
 3. 问题又来了：校园网每个月只有 20GB 免费流量，IPv6 流量不计。然而支持 IPv6 的路由器并不多，千挑万选买了一台入门级的华硕 RT-AC66U 路由器（以下简称 AC66U），但是在管理界面打开 IPv6 选项之后，PC 并没有自动分配到 IPv6 地址。

既然 AC66U 搭载了 AsusWRT 系统，那么完全可以当作一台普通的 Linux 机器，手动地解决 IPv6 的问题。

首先分析问题，路由器在 IPv4 网络下的工作原理是使用 NAT（Network Address Translation，网络地址转换）将一个可路由的外网地址与子网设备的内网地址进行转换。然而 IPv6 地址多得用不完，因此不需要 NAT，所以即使路由器自身成功地获取到了一个外网 IPv6 地址，也没有办法为子网设备分配本地的 IPv6 地址。

不过解决方法还是有的，NDP（Neighbour Discovery Protocol，邻居发现协议）可以让具有可路由外网地址的设备（路由器）告诉上层路由，某个不可路由地址（子网设备）是它的邻居，或者说在它的子网中。NDP 不只可以用于 IPv6，在 IPv4 网络中 NDP 经常可以用作 NAT 穿透的一种手段。那么就在 AC66U 上配置 NDP 吧。

## 刷机

AC66U 自带的 AsusWRT 不支持 SSH 登录，怒刷第三方系统。[AsusWRT-Merlin][1] 是一个强化版的 AsusWRT，虽然不知道究竟强化在哪里，但是很好用。

要注意的是，华硕最新的官方固件不再允许刷入未经认证的第三方系统，因此在路由器管理界面用系统更新功能来刷机可能会失败。这种情况可以把路由器切换到恢复模式（按住重置键开机），然后用华硕提供的 Firmware Restoration 工具刷入新固件。这个工具可以在 [AC66U 的支持页面][2]下载到。

## 安装 Entware

[Entware][3] 是一个用于嵌入式设备的包管理器，安装方法可以看[这里][4]。简单概括：找个 ext2/ext3 的 U 盘插到路由器上；运行 *entware-setup.sh*。

安装完成后，记得用 `opkg update` 更新源。

## 安装和配置 ndppd

[ndppd][5] 全名 NDP Proxy Daemon，顾名思义是一个用来支持 NDP 的守护程序，响应上层路由的邻居发现请求。

用 Entware 安装 ndppd：

```
$ opkg install ndppd
```

查看路由器的外网 IPv6 地址（假设 eth0 是外网网卡。一定要是带有 *Global* 的地址）：

```
$ ifconfig eth0
eth0 Link encap:Ethernet HWaddr FC:...
...
inet6 addr: 2001:da8:aaaa:bbbb:cccc:dddd:eeee:ffff/64 Scope:Global
...
```

地址的前缀长度是 64，因此我们的子网前缀要长于 64 才可以，以 80 为例，取地址的前 5 段作为前缀，即（*aaaa*、*bbbb* 和 *cccc* 请根据实际地址替换）：

```
2001:da8:aaaa:bbbb:cccc::/80
```

修改 ndppd 的配置文件（*/opt/etc/ndppd.conf*）：

```
...
proxy eth0 {
    ...
    rule 2001:da8:aaaa:bbbb:cccc::/80 {
        auto
    }
}
```

保存修改，运行 ndppd：

```
$ ndppd -d -p /opt/var/run/ndppd.pid
```

## 配置内网地址

ndppd 运行之后，就可以响应上层路由对 2001:da8:aaaa:bbbb:cccc::/80 子网段的请求，不过我们还没有为子网内的设备分配可路由的 IP 地址。

首先设置网关地址，假设 `br0` 是内网网卡：

```
$ ip -6 addr add 2001:da8:aaaa:bbbb:cccc::1/80 dev br0
```

然后为设备手动分配的 IPv6 地址。地址填写 2001:da8:aaaa:bbbb:cccc::/80 网段内的任意一个地址，默认网关填写 2001:da8:aaaa:bbbb:cccc::1。这样应该就可以成功连接 IPv6 网络了。

## 开机启动 ndppd

AsusWRT-Merlin 的开机启动脚本位于 */jffs/scripts*，详细的文档在这里。其中 *services-start* 脚本用于在开机之后启动（或以不同的配置重新启动）服务，在这个脚本里加上：

```
ip_addr=`ip addr show eth0 | grep “inet6\b.*global.*” | awk ‘{print $2}’ | cut -d ‘/’ -f 1`
addr1=`echo $ip_addr | cut -d “:” -f 1`
addr2=`echo $ip_addr | cut -d “:” -f 2`
addr3=`echo $ip_addr | cut -d “:” -f 3`
addr4=`echo $ip_addr | cut -d “:” -f 4`
addr5=`echo $ip_addr | cut -d “:” -f 5`
router_addr=$addr1":”$addr2":”$addr3":”$addr4":”$addr5"::1/80"
ip -6 addr add $router_addr dev br0

pkill ndppd
ndppd -d -p /opt/var/run/ndppd.pid
```

第一段代码获取 *eth0* 的外部 IPv6 地址，取出前 5 段并组成网关地址，然后添加给 `br0` 网卡。至于 `pkill ndppd`，我不确定 ndppd 安装后是否会自动跟随系统启动，不过先杀一下肯定不会有问题的。

## 自动分配 IPv6 地址

有没有办法实现 IPv6 的 DHCP 呢？当然有，不过我还没尝试成功，之后再写吧。

[1]: https://github.com/RMerl/asuswrt-merlin
[2]: https://www.asus.com/Networking/RTAC66U/HelpDesk_Download/
[3]: http://entware.net/
[4]: https://github.com/RMerl/asuswrt-merlin/wiki/Entware
[5]: https://github.com/DanielAdolfsson/ndppd

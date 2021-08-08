> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.gigiwangs.com](https://www.gigiwangs.com/archives/2079)

> 1. 刷开发版 ROM, 获取 SSH(root) ssh root@192.168.31.1 2. 刷入 breed 1. 下载 breed2. Ssh 登录刷入 breed scp breed-mt7621-xi......

1. 刷开发版 ROM, 获取 SSH(root)
-------------------------

```
ssh root@192.168.31.1
```

2. 刷入 breed
-----------

*   1. [下载 breed](https://breed.hackpascal.net/breed-mt7621-xiaomi-r3g.bin)
*   2. Ssh 登录刷入 breed

```
scp breed-mt7621-xiaomi-r3g.bin root@192.168.31.1:/tmp
mtd -r write /tmp/breed-mt7621-xiaomi-r3g.bin  Bootloader
```

3. 登录 breed 刷 openwrt
---------------------

开机 reset 网线接入 进入 192.168.1.1  
刷入 x-wrt-8.0-b202005230224-ramips-mt7621-xiaomi_mir3g-squashfs-breed-factory.bin  
重启自动进入管理界面

4. 设置 opkg 源
------------

*   替换国内源

将 openwrt 默认 opkg distfeeds.conf 源替换为 mirrors.tuna.tsinghua.edu.cn/openwrt

编辑文件 / etc/opkg/distfeeds.conf

```
src/gz x-wrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/targets/ramips/mt7621/packages
src/gz x-wrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/packages/mipsel_24kc/base
src/gz x-wrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/packages/mipsel_24kc/luci
src/gz x-wrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/packages/mipsel_24kc/packages
src/gz x-wrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/packages/mipsel_24kc/routing
src/gz x-wrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/snapshots/packages/mipsel_24kc/telephony
```

*   设置 sourceforge 安装源 (可选)

> dns-forwarder 几个包, 也可以直接通过链接下载安装

```
wget http://openwrt-dist.sourceforge.net/packages/openwrt-dist.pub -O /tmp/openwrt-dist.pub
opkg-key add /tmp/openwrt-dist.pub
```

添加到 customfeeds.conf

```
src/gz openwrt_dist http://openwrt-dist.sourceforge.net/packages/base/mipsel_24kc
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/packages/luci
```

*   更新, 安装 CA 证书以支持 https

```
opkg --no-check-certificate update
 opkg install ca-certificates --no-check-certificate
```

### 使用 https 下载软件源

```
<code>sed -i 's/http:/https:/g' /etc/opkg/distfeeds.conf</code>
```

5. 安装包
------

> 如果没有安装 CA 包则不支持 https, 需添加 –no-check-certificate 参数

### shadowsocks

```
opkg  install luci-app-shadowsocks-libev
opkg  install shadowsocks-libev-config
opkg  install shadowsocks-libev-ss-local
opkg  install shadowsocks-libev-ss-redir
opkg  install shadowsocks-libev-ss-rules
opkg  install shadowsocks-libev-ss-server
opkg  install shadowsocks-libev-ss-tunnel
```

### dns-forwarder luci-app-dns-forwarder

安装 dns-forwarder luci-app-dns-forwarder

*   配置过 sourceforge 源则直接 opkg 安装

```
opkg install dns-forwarder luci-app-dns-forwarder
```

*   或者直接下载安装

[luci-app-dns-forwarder_1.6.2-1_all.ipk](http://openwrt-dist.sourceforge.net/packages/luci/luci-app-dns-forwarder_1.6.2-1_all.ipk)

[dns-forwarder_1.2.1-2_mipsel_24kc.ipk](http://openwrt-dist.sourceforge.net/archives/dns-forwarder/1.2.1-2/current/mipsel_24kc/dns-forwarder_1.2.1-2_mipsel_24kc.ipk)

```
<code>opkg install dns-forwarder_1.2.1-2_mipsel_24kc.ipk luci-app-dns-forwarder_1.6.2-1_all.ipk</code>
```

### dnsmasq-full

```
opkg install ip ipset libpthread iptables-mod-tproxy
opkg install zlib
opkg remove dnsmasq && opkg install dnsmasq-full
```

### v2ray-plugin (可选)

```
wget -c  https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.3.0/v2ray-plugin-linux-mips-v1.3.0.tar.gz
tar -xvf v2ray-plugin-linux-mips-v1.3.0.tar.gz
scp v2ray-plugin_linux_mipsle_sf root@192.168.15.1:/usr/bin

net.ipv4.tcp_fastopen = 3
```

6. 配置 dnsmasq 目录
----------------

*   1. 新建 / etc/dnsmasq.d 目录，然后使用如下命令，将该路径加入配置：

```
mkdir -p /etc/dnsmasq.d
uci add_list dhcp.@dnsmasq[0].confdir=/etc/dnsmasq.d
uci commit dhcp
```

*   2. 验证下是否成功：

```
uci get dhcp.@dnsmasq[0].confdir
```

*   3. 修改 /etc/dnsmasq.conf，在最后加入 conf-dir=/etc/dnsmasq.d

7. 设置 ipset
-----------

*   1. 下载 gfwlist2dnsmasq.sh 脚本

```
wget -c https://raw.githubusercontent.com/cokebar/gfwlist2dnsmasq/master/gfwlist2dnsmasq.sh --no-check-certificate
/usr/bin/gfwlist2dnsmasq.sh -s gfwlist -o /etc/dnsmasq.d/dnsmasq_gfwlist_ipset.conf
```

如果获取 gfwlist 出现问题, 替换 gfwlist2dnsmasq.sh 中 URL:BASE_URL=

*   **Official mirror URLs:**
*   Pagure: [https://pagure.io/gfwlist/raw/master/f/gfwlist.txt](https://pagure.io/gfwlist/raw/master/f/gfwlist.txt)
*   Bitbucket: [https://bitbucket.org/gfwlist/gfwlist/raw/HEAD/gfwlist.txt](https://bitbucket.org/gfwlist/gfwlist/raw/HEAD/gfwlist.txt)
*   Gitlab: [https://gitlab.com/gfwlist/gfwlist/raw/master/gfwlist.txt](https://gitlab.com/gfwlist/gfwlist/raw/master/gfwlist.txt)

*   2. ipset 设置

```
ipset -N gfwlist iphash
iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1100
iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1100
ip6tables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1100
ipset add gfwlist 8.8.8.8
/etc/init.d/dnsmasq restart
```

*   3. 加入 crontab

> 0 0 * * 0 cd /tmp && /usr/bin/gfwlist2dnsmasq.sh -s gfwlist -o /etc/dnsmasq.d/dnsmasq_gfwlist_ipset.conf > /dev/null

*   4. 添加开机启动

```
/etc/init.d/ipset-gfwlist
```

```
#!/bin/sh /etc/rc.common
START=99


start() {
    /usr/bin/gfwlist2dnsmasq.sh -s gfwlist -o /etc/dnsmasq.d/dnsmasq_gfwlist_ipset.conf > /dev/null 2>&1 &
    ipset -N gfwlist iphash > /dev/null 2>&1 &
    iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1100
    iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1100
    ipset add gfwlist 8.8.8.8 > /dev/null 2>&1 &
    /etc/init.d/dnsmasq restart > /dev/null 2>&1 &
}
stop() {
    return 0
}
```

```
软连接生效<br><code>ln -s /etc/init.d/ipset-gfwlist /etc/rc.d/S99ipset-gfwlist</code>
```

8. 设置 shadowsocks
-----------------

配置文件附录

9. 设置 dns-forwarder
-------------------

### 高版本 luci 无法加载页面 bug 解决

```
/usr/share/rpcd/acl.d

root@X-WRT:/usr/share/rpcd/acl.d# cat luci-app-dns-forwarder.json
{
    "luci-app-dns-forwarder": {
        "description": "Grant service list access to LuCI app dns-forwarder",
        "read": {
            "ubus": {
                "service": [ "list" ]
            },
            "uci": [ "dns-forwarder" ]
        },
        "write": {
            "file": {
                "/etc/dns-forwarder": [ "write" ]
            },
            "uci": [ "dns-forwarder" ]
        }
    }
}
```

`

### 配置文件 (可界面操作)

```
config dns-forwarder
    option enable '1'
    option listen_addr '127.0.0.1'
    option listen_port '5353'
    option dns_servers '8.8.8.8'
```

10. 配置网络接口
----------

WAN 口按需配置
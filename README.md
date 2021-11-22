# 服务端安装

1. [vultr](https://www.vultr.com) 买个虚拟机，server选Cloud Compute（便宜），然后location选择silicon valley或者LA（时延低），os选debian9 x64（自带BBR加速）
2. 使用 [一键安装脚本](https://github.com/233boy/v2ray/wiki/V2Ray/一键安装脚本)，命令：
```shell
bash <(curl -s -L https://git.io/v2ray.sh)
```
3. 协议建议选mKTP (6)


常见问题：未开启IPV4监听，详细描述点击 [这里](https://www.linodovultr.com/post/resolve-v2ray-after-install-can-not-connect.html)，增加IPV4监听，修改配置文件
```shell
vim /etc/v2ray/config.json
```
在"inbound"里，"protocol": "vmess"下面，增
加"listen":"12.34.56.78"，这个IP为实际IP，然后通过命令检查IPV4是否正常打开
检查命令
```shell
root@vultr:~# netstat -anp|grep v2ray
udp        0      0 xx.xx.xx.xx:xxxxx    0.0.0.0:*                           xxxx/v2ray
unix  3      [ ]         STREAM     CONNECTED     xxxxx    xxxx/v2ray
```
如果出了TCP6之外出现其他例如udp、tcp进程就表示可以用非IPV6了。

注意，每次使用v2ray config进行修改后，这个/etc/v2ray/config.json文件都会被覆盖，需要重新设置listen参数。

使用二维码进行手机设置，二维码生成方式：
```shell
v2ray qr
```
生成二维码URL后，使用二维码生成器生成二维码，地址：[https://cli.im](https://cli.im)

# 常用客户端

- win client：[v2ray win client](https://github.com/v2ray/v2ray-core/releases)
- android client：[v2ray android client](https://github.com/2dust/v2rayNG/releases)
- mac client：[v2ray mac client](https://github.com/Cenmrev/V2RayX/releases)
- linux client: [v2ray linux client](https://github.com/jiangxufeng/v2rayL/releases)
- ios client: 在appstore中搜索shadowrocket，quantumult，kitsunebi，需要使用美区appleId

# 补充

部分os（例如ubuntu）不自带netstat命令，如果需要安装netstat，则使用命令
```shell
apt-get install net-tools
```

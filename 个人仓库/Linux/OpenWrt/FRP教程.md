# Openwrt端
1. 下载OpenWrt Frpc
```shell
https://github.com/kuoruan/openwrt-frp/releases
```
2. 下载frpc luci(下载luci-app-frpc_1.2.1-1_all.ipk和luci-i18n-frpc-zh-cn_1.2.1-1_all.ipk)
```shell
https://github.com/kuoruan/luci-app-frpc/releases
```
3. 路由器后台或者使用sftp将下载的三个文件夹上传至/root目录
4. 安装
```shell
opkg install frpc_0.34.3-1_x86_64.ipk 
opkg install luci-app-frpc_1.2.1-1_all.ipk 
opkg install luci-i18n-frpc-zh-cn_1.2.1-1_all.ipk
```
参考网址：
- https://www.quarkbook.com/?p=1174  （frpc安装）
- https://naiyous.com/4482.html  （frp服务一键脚本）
- https://www.youtube.com/watch?v=D7e6TaWSneg  （frp服务及客户端使用）
# VPS端
1. 一键安装脚本
```shell
wget https://gitee.com/mvscode/frps-onekey/raw/master/install-frps.sh -O ./install-frps.sh chmod 700 ./install-frps.sh ./install-frps.sh install
```
2. 设置后台运行
```shell
nohup ./frps -c frps.ini >/dev/null 2>&1 &
```
3. 防火墙相关（==国内 [服务器](https://naiyous.com/4482.html#)可以直接设置防火墙规则==）
```shell
Debian或Ubuntu系统放行端口： ufw allow 端口号 iptables -A INPUT -p tcp --dport 端口号 -j ACCEPT CentOS系统放行端口： firewall-cmd --zone=public --add-port=端口号/tcp --permanent
```
4. FRP常用命令
```shell
开启FRP：frps start 
停止FRP：frps stop 
重启FRP：frps restart 
打开配置文件：frps config 
查看FRP版本：frps version 
检查FRP运行状态：frps status
```
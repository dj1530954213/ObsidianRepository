# 系统相关知识点
## 1、初始系统通过ssh远程登录root账号
``` shell
vim /etc/ssh/sshd_config#第一步进入配置文件
#第二步
PermitRootLogin yes
#第三步
PasswordAuthentication yes
#第四步
/etc/init.d/ssh restart
```


# 指令相关知识点
## 1、文件操作命令
1. 将A设备的文件拷贝至B设备
```linux
 #其中192.168.1.33是接收文件的设备
 scp frpc_0.57.0-r1_x86_64.ipk luci-app-frpc_1.2.1-1_all.ipk luci-i18n-frpc-zh-cn_1.2.1-1_all.ipk root@192.168.1.33:/root
```
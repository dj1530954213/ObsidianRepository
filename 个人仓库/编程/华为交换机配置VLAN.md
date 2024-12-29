### 一、需求
一、将24口交换机的端口分配为4个vlan分别问vlan100、vlan200、vlan300、vlan400对应着四个网段:192.168.1.*、172.19.0.*、172.19.6.*、192.168.7.*    其中vlan100是1~8口vlan200是9~16口、vlan300是17~20、vlan400是21~24。 
二、要求这四个网段能够互相通讯，
其中192.168.1.*的网关为192.168.1.1
172.19.0.*的网关为172.19.0.103
172.19.6.*的网关为172.19.6.129
192.168.7.*的网关为192.168.7.1




### 二、配置步骤
#### 1、划分Vlan
```shell
首先先检查一下当前的vlan划分情况
display vlan



第一步:创建vlan
vlan 100   //创建名称为100的vlan

description xf    //修改备注

interface range GigabitEthernet 0/0/1 to GigabitEthernet 0/0/8 //将1~8口划一个零时组进行统一的设置

port link-type access    //将端口配置为接入端口

port default vlan 100 //将端口分配至vlan 100

```

#### 2、创建网关
```shell
# 配置VLAN100的接口和网关 
interface Vlanif100 ip address 192.168.1.1 255.255.255.0 # 配置VLAN200的接口和网关 
interface Vlanif200 ip address 172.19.0.103 255.255.255.192
# 配置VLAN300的接口和网关 
interface Vlanif300 ip address 172.19.6.129 255.255.255.192
# 配置VLAN400的接口和网关 
interface Vlanif400 ip address 192.168.7.1 255.255.255.0
```

#### 3、查看vlan的网关和路由表
```shell
display ip interface brief  //查看各vlan网关地址
display ip routing-table    //查看路由表
```

#### 4、保存配置文件否则将会掉电后丢失配置
```shell
退出系统视图返回用户视图
#保存配置文件
save ****.cfg
#查看配置文件是否保存成功
dir

```

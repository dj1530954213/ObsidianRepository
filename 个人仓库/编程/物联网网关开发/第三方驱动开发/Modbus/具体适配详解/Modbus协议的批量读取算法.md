# 解决方案
## 一、调用驱动开发接口实现变量分发
- 按照0区、1区、3区、4区进行分配
- 创建4个变量字典用于存放分配出来的变量
## 二、设备连接时扫描所有变量
- Modbus协议的各种变量类型的数据大小
	```csharp
	short：2个字节  
		ushort：2个字节  
		int：4个字节  
		uint：4个字节  
		long：8个字节  
		ulong：8个字节  
		float：4个字节  
		double：8个字节
		string：具体字节数按照HSL中的定义读取时读取的个数为字节数，及读取1个字节(两个字符)

 ```
- 将变量分发至4个变量存储字典中,并按照HSL中的设计规则进行逐一读取
	```csharp
	/*对于功能码的差异分装在驱动中，用户操作的时候只填地址*/
	//0区数据读取
	bool coil100 = busTcpClient.ReadBool("100").Content;
	//1区数据读取
	bool coil100 = busTcpClient.ReadBool("x=2,100").Content;
	//3区数据读取
	float float100 = busTcpClient.ReadFloat("x=4,100").Content;
	//4区数据读取
	float float100 = busTcpClient.ReadFloat("100").Content;	
	  ```
  - 按照保持线圈、可读线圈、可读寄存器、保持寄存器的分类分别创建对应的连续读取记录字典和读取结果存放数组
  - 遍历地址数组判断最大连续读取区域并生成读取字符串
	  ```csharp
	  
	    ```
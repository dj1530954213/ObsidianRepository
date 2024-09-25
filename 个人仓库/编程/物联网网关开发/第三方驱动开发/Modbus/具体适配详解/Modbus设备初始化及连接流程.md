### 一、设备初始化流程
1. 设备初始化检查(DeviceInit)
	1. 检查驱动属性设置是否正常(CheckDevicePropertiesAsync)
		1. 连接参数属性是否设置(下发类特性标注  ProtocolDescribe)
		2. 协议类型是否设置(下发类特性标注  ProtocolType)
		3. 调用对应协议的初始化参数检查进行属性是否正确的分析GeneralInitCheck
	2. 根据当前设定的通讯协议类型创建对应的驱动实例
		1. 根据协议的类型创建实例CreateDevice
		2. 调用对应的创建方法比如：CreateDevice_ModbusTcp完成初始化参数赋值
		3. 将返回的实例转换为后续功能锁需要的ModbusDeviceConnectionBase(内部封了串口通讯与网口通讯的相关变化)、IReadWriteDevice、IModbus的私有字段
	3. 调用设备连接完成设备连接的相关准备工作
		1. 连接数据服务
		2. 逐个读取变量以生成可批量读取的点表AddressInit
		3. 根据批量读取点表完成批量读取报文的拼接AddressCombination
	4. 数据采集
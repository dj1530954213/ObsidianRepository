## 一、AI模块
1. 通道设定所需变量
```csharp
变量名            描述                 数据类型       
WU               工程量上限            WORD
WD               工程量下限            WORD
MU               模拟量上限            REAL
MD               模拟量下限            REAL
RU               目标上限              REAL
RD               目标下限              REAL
Enable           转换1投用/0停用       BOOL
Input_REAL       输入值_模拟量         REAL
Input_WORD       输入值_离散量         WORD
Input_Type       输入值类型(0为离散1为模拟)  BOOL
OutPut           输出值               REAL
```

![[Pasted image 20240928172400.png]]

块内逻辑
```CSHARP
IF Enable THEN
	IF Input_Type THEN(*当输入类型为模拟量时*)
		OutPut:=(Input_REAL - MD)/(MU - MD)*(RU-RD)*1.0+RD;
	ELSE
		OutPut:=(Input_WORD - WD)*1.0/(WU - WD)*(RU-RD)*1.0+RD;
	END_IF;
END_IF;
```

使用方法
![[Pasted image 20240928195957.png]]

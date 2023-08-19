**【野火】瑞萨RA MCU创意氛围赛+《手势识别控制终端》**

###  1、**硬件部分：**

设备型号：**野火RA6M5开发板**

![image-20230818005119681](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/image-20230818005119681.png)

**外围设备：**

​	**GY- AMG8833 IR 8x8 红外热像仪**

![image-20230818005611971](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/image-20230818005611971.png)




​	**1.44寸彩色TFT显示屏高清IPS LCD液晶屏模块128*128** 

![image-20230818010322608](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/image-20230818010322608.png)





**其他配件：**

​	面包板 x 1

​	杜邦线若干





**项目描述：**

​	本项目主要通过**AMG8833**模块 获取手部的温度，然后通过**BP神经网络**解析温度数据，来识别手部动作。当手部动作和预定控制指令激活动作相匹配时，向外部设备发送控制指令，当外部设备接收到对应指令执行对应的操作。



**项目环境描述：**

​	因为该设备是通过手部温度作为控制变量，所以项目运行的温度在28℃摄氏度下（设备静态是经过传感器测量得到的数据）。手部温度为，33℃左右，手部距离传感器大概在**5cm** 左右，并且处于 传感器芯片正前方。说明：环境温度会影响传感器的识别。



**设备引脚配置：**



 

![RenesasPinConfigView](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/RenesasPinConfigView.png)

**引脚连接：**

![PinConfig](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/PinConfig.png)



以及串口：

​	TX : P512

​	RX: P511



以上是项目的硬件部分

----

### 2、**软件部分：**

**项目完成使用到的软件有：**

​	e^2^ studio

​	vscode

​	字模软件 PCtoLCD2013

​	野火串口调试助手



**软件部分代码说明：**

> - **GY- AMG8833 IR 8x8 红外热像仪** 驱动部分代码说明：

​	**AMG8833**模块使用**I2C** 通讯协议：(使用 硬件I2C)

​	下面是模块是主要的各个功能驱动函数

​	根据数据手册说明：只要主机向从机发送**0x80**指令，从机设备 会直接 一次性按顺序发送完 温度栅格点 1-64 的温度数据 

​	其他指令：按照I2C 通讯协议读取

​	I2C 驱动 .C 文件部分函数

~~~c
// 设置传感器模式
void AMG88_SetSensorMode(AMG88_OperatingMode Mode)
{

    unsigned char buffer[2]={0x00,Mode};
    R_SCI_I2C_Write(&g_i2c6_ctrl, buffer, 2, false);
    return;


}

// 获取当前传感器模式

unsigned char AMG88_GetSensorMode(void)
{
    unsigned char OperatingModeBuffer=0;
    R_SCI_I2C_Write(&g_i2c6_ctrl, 0, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &OperatingModeBuffer, 1, false);
    return OperatingModeBuffer;
}



// 重启传感器
void AMG88_SensorReset(AMG88_ResetMode Mode)
{
    //
    unsigned char ResetBuffer[2]={0x01,(unsigned char)Mode};
    //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, 2, false);


    return;
}

// 设置帧率
void AMG88_SetFrameRate(AMG88_Frame Frame)
{

    unsigned char ResetBuffer[2]={0x02,(unsigned char)Frame};
    //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, 2, false);

    return;
}
// 获取传感器帧率
unsigned char AMG88_GetFrameRate(void)
{
    unsigned char OperatingModeBuffer=0;
    unsigned char Address[1]={0x02};
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &OperatingModeBuffer, 1, false);
    R_BSP_SoftwareDelay(2, 1000);
    return OperatingModeBuffer;
}
// 设置中断控制寄存器
void AMG88_SetICR(AMG88_ICR_REGISTER ICR)
{
    unsigned char ResetBuffer[2]={0x03,(unsigned char)ICR};
        //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, 2, false);

	return;
}
// 获取中断控制寄存器的数据
unsigned char AMG88_GetICR(void)
{
    unsigned char OperatingModeBuffer=0;
    unsigned char Address[1]={0x03};
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);
//    R_SCI_I2C_Write(&g_i2c6_ctrl, 0x03, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &OperatingModeBuffer, 1, false);
    return OperatingModeBuffer;
}
// 获取当前传感器状态
unsigned char AMG88_GetStatus(void)
{
    unsigned char OperatingModeBuffer=0;
    unsigned char Address[1]={0x04};
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);
//    R_SCI_I2C_Write(&g_i2c6_ctrl, 0x04, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &OperatingModeBuffer, 1, false);
    return OperatingModeBuffer;

}
// 清除传感器标志位
void AMG88_SetStatusClear(AMG_Status_FLAG ClearStatus)
{

    unsigned char ResetBuffer[2]={0x05,(unsigned char)ClearStatus};
            //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, 2, false);


    return;
}

// 
void AMG88_SetAverage(BOOL Flag)
{
    unsigned char ResetBuffer[2]={0x07,(Flag==TRUE)?(0xFF):(0)};
                //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, 2, false);

}
//
unsigned char AMG88_GetAverage(void)
{
    unsigned char OperatingModeBuffer=0;
//    R_SCI_I2C_Write(&g_i2c6_ctrl, 0x07, 1, true);
    unsigned char Address[1]={0x07};
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);

    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &OperatingModeBuffer, 1, false);
    return OperatingModeBuffer;
}

// 设置中断优先级
void AMG88_SetILR(unsigned char *ValueBuffer,unsigned char ArrayLenth)
{
    unsigned char ResetBuffer[7]={0x08,0x00,0x00,0x00,\
                                  0x00,0x00,0x00};
    if(ArrayLenth<=7 && ArrayLenth >= 1)
        return;
    for(unsigned char i= 1 ;i<7;i++)
    {

        if(i%2==0)
        {
            ResetBuffer[i]=(0x0F & ValueBuffer[i-1]);
        }else
        {
            ResetBuffer[i]=ValueBuffer[i-1];

        }

    }
    //unsigned char ResetBuffer=0x30;
    R_SCI_I2C_Write(&g_i2c6_ctrl, ResetBuffer, ArrayLenth+1, false);

	return;
}
unsigned char Tempeture_Flag[2];
// 获取传感器 热敏电阻 电阻值
unsigned short AMG88_GetThermistor(void)
{


    unsigned short buffer_flag=0;
    unsigned char Address[1]={0x0E};
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);
//    R_SCI_I2C_Write(&g_i2c6_ctrl, 0x0E, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, &Tempeture_Flag[0], 1, false);
    R_BSP_SoftwareDelay(2, 1000);
    Address[0]=0x0F;
    R_SCI_I2C_Write(&g_i2c6_ctrl, Address, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    R_SCI_I2C_Read(&g_i2c6_ctrl, &Tempeture_Flag[1], 1, false);
    R_BSP_SoftwareDelay(2, 1000);
    buffer_flag=Tempeture_Flag[1]<<8;
    buffer_flag|=Tempeture_Flag[0];
    return buffer_flag;
}
unsigned char Buffer[10];
unsigned char Revice[128];
// 获取传感器的温度
void AMG88_SensorData(void)
{
    /*
     * register address
     *
     * */
    Buffer[0]=0x80;
    //Send slave address
    R_SCI_I2C_Write(&g_i2c6_ctrl, Buffer, 1, true);
    R_BSP_SoftwareDelay(2, 1000);
    //Read Register data
    R_SCI_I2C_Read(&g_i2c6_ctrl, Revice, 128, false);
}
~~~

---


> - **1.44寸彩色TFT显示屏高清IPS LCD液晶屏模块128*128**  部分代码说明

​	**该LCD 液晶屏使用SPI 通讯协议：**(使用模拟SPI)

​	驱动芯片为**ST7735**

​	SPI驱动 .C 文件部分函数 

​	

```c

void SPI_init(void)
{

    SET_LED();
    SET_CS();
    SET_CDX();
    SET_RST();
    SET_CLK();
    SET_SDA();

    return;
}

void SPI_SendData(unsigned char Data) // CDX = 1
{

    unsigned char i;

    for (i = 0; i < 8; i++)
    {
        CLEAR_CLK();

        if ((Data & 0x80) != 0)
            SET_SDA();
        else
            CLEAR_SDA();

        Data <<= 1;

        SET_CLK();

    }

    return;
}

void SPI_WriteCommand(unsigned char Data) //CDX = 0
{

    CLEAR_CS();
    CLEAR_CDX();

    SPI_SendData (Data);

    SET_CS();

    return;
}
void SPI_WriteData(unsigned char Data) //CDX = 1
{

    CLEAR_CS();
    SET_CDX();

    SPI_SendData (Data);

    SET_CS();

    return;

}

void WriteDispData(unsigned char DataH, unsigned char DataL)
{

    SPI_SendData (DataH);
    SPI_SendData (DataL);


}
void LCD_Init(void)
{

    SET_RST();
    R_BSP_SoftwareDelay (100, BSP_DELAY_UNITS_MILLISECONDS);

    CLEAR_RST();
    R_BSP_SoftwareDelay (100, BSP_DELAY_UNITS_MILLISECONDS);

    SET_RST();
    R_BSP_SoftwareDelay (200, BSP_DELAY_UNITS_MILLISECONDS);

    SPI_WriteCommand (0x11); //Exit Sleep
    R_BSP_SoftwareDelay (120, BSP_DELAY_UNITS_MILLISECONDS);

    SPI_WriteCommand (0xB1);
    SPI_WriteData (0x05); //0a
    SPI_WriteData (0x3c); //14
    SPI_WriteData (0x3c);

    SPI_WriteCommand (0xB2);
    SPI_WriteData (0x05);
    SPI_WriteData (0x3c);
    SPI_WriteData (0x3c);

    SPI_WriteData (0xB3);
    SPI_WriteData (0x05);
    SPI_WriteData (0x3c);
    SPI_WriteData (0x3c);

    SPI_WriteData (0x05);
    SPI_WriteData (0x3c);
    SPI_WriteData (0x3c);

    SPI_WriteCommand (0xB4); // 前面的b1-b5 是设置帧速率
    SPI_WriteData (0x03);

    SPI_WriteCommand (0xC0); // Set VRH1[4:0] & VC[2:0] for VCI1 & GVDD      Power Control
    SPI_WriteData (0x28); 
    SPI_WriteData (0x08);
    SPI_WriteData (0x04); 

    SPI_WriteCommand (0xC1); // Set BT[2:0] for AVDD & VCL & VGH & VGL
    SPI_WriteData (0xC0);

    SPI_WriteCommand (0xC2); // Set VMH[6:0] & VML[6:0] for VOMH & VCOML
    SPI_WriteData (0x0D); 	 //54h
    SPI_WriteData (0x00);  	 //33h

    SPI_WriteCommand (0xC3);
    SPI_WriteData (0x8D);
    SPI_WriteData (0x2A);

    SPI_WriteCommand (0xC4);
    SPI_WriteData (0x8D);
    SPI_WriteData (0xEE);

    SPI_WriteCommand (0xC5);
    SPI_WriteData (0x1A);

    SPI_WriteCommand (0x36);    //MX,MY,RGB MODE
    SPI_WriteData (0x08);

    SPI_WriteCommand (0xe0);
    SPI_WriteData (0x04);    //2c
    SPI_WriteData (0x22);
    SPI_WriteData (0x07);
    SPI_WriteData (0x0A);
    SPI_WriteData (0x2E);
    SPI_WriteData (0x30);
    SPI_WriteData (0x25);
    SPI_WriteData (0x2A);
    SPI_WriteData (0x28);
    SPI_WriteData (0x26);
    SPI_WriteData (0x2E);
    SPI_WriteData (0x3A);
    SPI_WriteData (0x00);
    SPI_WriteData (0x01);
    SPI_WriteData (0x03);
    SPI_WriteData (0x03);

    SPI_WriteCommand (0xe1);
    SPI_WriteData (0x04);
    SPI_WriteData (0x16);
    SPI_WriteData (0x06);
    SPI_WriteData (0x06);
    SPI_WriteData (0x0D);
    SPI_WriteData (0x2D);
    SPI_WriteData (0x26);
    SPI_WriteData (0x23);
    SPI_WriteData (0x27);
    SPI_WriteData (0x27);
    SPI_WriteData (0x25);
    SPI_WriteData (0x2D);
    SPI_WriteData (0x3B);
    SPI_WriteData (0x00);
    SPI_WriteData (0x01);
    SPI_WriteData (0x04);
    SPI_WriteData (0x13);

    SPI_WriteCommand (0x3A);
    SPI_WriteData (0x05);

    SPI_WriteCommand (0x29); // Display On
    R_BSP_SoftwareDelay (20, BSP_DELAY_UNITS_MILLISECONDS);

}
void BlockWrite(unsigned short Xstart, unsigned short Xend, unsigned short Ystart, unsigned short Yend)
{
    SPI_WriteCommand (0x2A);
    SPI_WriteData (Xstart >> 8);
    SPI_WriteData (Xstart + 2);
//    SPI_WriteData(Xstart);
    SPI_WriteData (Xend >> 8);
    SPI_WriteData (Xend + 2);
//    SPI_WriteData(Xstart);

    SPI_WriteCommand (0x2B);
    SPI_WriteData (Ystart >> 8);
    SPI_WriteData (Ystart + 1);
    SPI_WriteData (Yend >> 8);
    SPI_WriteData (Yend + 1);

    SPI_WriteCommand (0x2c);
}
void DispColor(unsigned short color)
{
    unsigned short i, j;

    BlockWrite (0, COL - 1, 0, ROW - 1);

    for (i = 0; i < ROW; i++)
    {
        for (j = 0; j < COL; j++)
        {
            SPI_WriteData (color >> 8);
            SPI_WriteData (color);
//            DelayMs(1);
        }
    }

}
void ClearFullScreen(void)
{

    unsigned short i, j;
    BlockWrite (0, COL - 1, 0, ROW - 1);
    for (j = 0; j < COL; j++)
    {
        SPI_WriteData (i + 50);
        SPI_WriteData (j + 50);

    }

    return;
}
void DrawColor(unsigned short ColorNumber)
{

    SPI_WriteData (0xFF);
    SPI_WriteData (0xFF);
    return;
}

```

---

> - BP 神经网络：代码说明：

​	三层网络结构

​	第一层是输入层，第二层是隐藏层，第三层是输出层

![3层PB网络结构图](https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/3%E5%B1%82PB%E7%BD%91%E7%BB%9C%E7%BB%93%E6%9E%84%E5%9B%BE.png)

​	神经网络预测代码说明：

​	神经网络预测的原理是，将目标数据输入到神经网络中，经过神经网络中参数的迭代，使之得到符合要求的数据数据，然后保存神经网络中的参数（各个节点的权重参数）。使用该网络预测时，将训练好的参数，导入到神经网络中，该神经网络就预测和神经网络中相符合的数据。

​	该神经网络的相关信息如下：

​	三层 BP神经网络

​	输入层有 64 个元素  ， 隐藏层有 34个元素， 输出层有 10 个元素

​	训练次数 为 ：**10000次**，最终的错误率为：**0.00658**，学习率为： **0.1** ，动量因子 ： **0.1** 训练数据总共 160 组 （160组 中 ，分成 三份）

​	总共训练了 三个 手势 

<img src="https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/%E6%89%8B%E5%8A%BF1.jpg" alt="手势1" style="zoom:25%;" /><img src="https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/%E6%89%8B%E5%8A%BF2.jpg" alt="手势2" style="zoom:25%;" /><img src="https://gitee.com/kysfh/image/raw/master/Firebbs_RenesasMCU/%E6%89%8B%E5%8A%BF3.jpg" alt="手势3" style="zoom:25%;" />

> ​	**手势1 36组数据** 													**手势2 68 组数据** 											         	**手势3 54组数据** 

> ​                                **上图：为 编写文档时所拍，非传感器 测量时 图片 ，仅说明在采集测试数据时的手势动作**

​	训练数据示例：

~~~c
[[0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.5,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000],[0,1,0,0,0,0,0,0,0,0]],
// 手势 1  要求输出 结果 -----> [0,1,0,0,0,0,0,0,0,0]

[[0.0000,0.0000,0.5,0.5,0.0000,0.5,0.5,0.5,0.0000,0.0000,0.5,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000],[0,0,1,0,0,0,0,0,0,0]],
// 手势 2  要求输出 结果 -----> [0,0,1,0,0,0,0,0,0,0]

[[0.0000,0.0000,0.0000,0.5,0.5,0.5,0.5,0.5,0.0000,0.0000,0.0000,0.5,0.5,0.5,0.0000,0.5,0.0000,0.0000,0.5,0.0000,0.0000,0.5,0.0000,0.5,0.0000,0.5,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.5,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000,0.0000],[0,0,0,1,0,0,0,0,0,0]]
// 手势 3  要求输出 结果 -----> [0,0,0,1,0,0,0,0,0,0]

// 注： 以上数据仅为 测试数据中的手势数据的 一部分 ，不代表整体数据

// 预测输出数据 示例：
[-0.0023248156377385144, 0.035785164105157696, 0.05889932014156386, 0.9992514065884543, 0.0003713636538696458, -0.002541229896438062, -0.0033772818188316607, -0.0023972941452978813, 0.001043452650557289, -0.0026320033807735485]


~~~

​	输出数据说明：

​	该网络有10个数据输出 ，（如：）**[0,0,0,1,0,0,0,0,0,0]** （从左往右）依次是 0 - 9 手势 ，但本次训练 仅仅训练了3 个手势， 结果如上。

​	其他信息说明：

- 本次的隐藏层的数目依次经历了 12->24->128->34 的变化 ，具体的数目和输入输出的元素个数，没有实际的关联（网上虽然有建议） ，具体看情况而论，因为是三层网络，隐藏层的数量不可以太少，也不可以太多，太少，说简单的，输出的数据不在[0,1]的区间，太多，输出的都是0.9左右的数据
  - 输出的数据不在[0,1]的区间
    - 可以调整 学习率 或者 训练次数（增加），或者是动量因子（修改该参数时，学习率不变）
    - 调整隐藏层的节点数目（往大了调）
  - 输出的都是0.9左右的数据（过拟合）
    - 调整隐藏层的节点数目（往小了调）**（按实际情况调节）**
- 输出数据的设定，按照激活函数的取值选择
- 输入数据的选择，[0-1]之间 ，为了提供训练的成功率，在输入数据中做了一些处理
- 训练的前提是保证网络正常（代码没有写错）

​	优化训练的操作说明

> 1. 对数据进行了非0即0.5 的处理 ，对于超过 特定温度值的数据为0.5 ，不超过为 0（只要有相对应的特征即可）
> 2. 网络训练成功的标志，输出的数据在（本网络）[0,1]之间，并且输出的数据 对应符合 输入的数据（只要有符合的即可尝试在在设备上运行），建议训练完成的网络，在预测时，要同时多预测几个，防止是误差



~~~c
// 激活函数
double sigmoid(double x)
{
	return tanh(x);
}
    
// 前向传播
void Forward()
{
	unsigned char i=0,j=0;
	double Temp=0.0;
	double *InputValueTemp;
	InputValueTemp=InputValue;
	for( i=0 ;i< HIDDENSIZE ; i++)
	{	
		Temp=0;
		for(j=0 ; j < INPUTSIZE ; j++ )
		{

			Temp+=InputValue[j]*InputWeight[j*HIDDENSIZE+i];
		}
		HiddenValue[i]=sigmoid(Temp);
	}
	
	for( i=0 ;i < OUTPUTSIZE ; i++)
	{
		Temp=0;
		for( j = 0; j < HIDDENSIZE ;j++ )
		{
			Temp+=HiddenValue[j]*OutputWeight[j*OUTPUTSIZE+i];
		}
		OutputValue[i]=sigmoid(Temp);
		
		}

	}
	

}
~~~



左上角  手势1 白色

​			 手势2  浅绿色

​			 手势3 浅紫色
        
	> https://www.bilibili.com/video/BV1zu4y1Q7Qe/?spm_id_from=333.999.0.0&vd_source=f4e3357c3319ee1b23e4991a55af1df0



[点击跳转->Bilibili 演示视频](https://www.bilibili.com/video/BV1zu4y1Q7Qe/?spm_id_from=333.999.0.0&vd_source=f4e3357c3319ee1b23e4991a55af1df0)

本项目还有需要优化的地方，也有着许多不足。作者水平有限，希望广大网友批评指正。

以上就是该项目的所有内容


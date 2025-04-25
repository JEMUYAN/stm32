# CAN简介

## 协议（数据帧）

构成：帧起始；仲裁段；控制段；数据段；CRC段；ACK段；帧结束

![image-20250425162520922](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425162520922.png)

### 仲裁段

![image-20250425162916814](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425162916814.png)

- 禁止高七位为隐性电平
- 拓展格式中用SRR代替远程请求位（RTR），其为隐性位，在拓展ID结束后还有真正的RTR位
- IDE：标识符选择位，标准格式中其位于控制段，拓展格式中位于仲裁段，**但位置相同**

### 控制段

r~0~、r~1~为显性占位符

![image-20250425162943711](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425162943711.png)

### 数据段

- 长度：0~8字节（由DLC决定）
- MSB输出

### ACK

发送单元的ACK发送两个隐性位，而接收到正确消息的单元在**ACK槽**发送显性位通知发送单元正常接收结束

## 位时序

- 同步段（SS）
- 传播时间段（PTS）
- 相位缓冲段1/2（PBS1/2）
- SJW：再同步补偿宽度

每个段由若干Tq构成，**电平翻转最好发生在SS，采样点在PBS1结束处**

## CAN收发流程

![image-20250425164501826](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425164501826.png)

![image-20250425164541407](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425164541407.png)

接收到的报文数可以在 `CAN_RFxR` 中查询

## STM32CAN硬件框图

![image-20250425163600401](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425163600401.png)

### 过滤器配置

每个过滤器组由两个32位寄存器，`CAN_FxR1` 和 `CAN_FxR2` 组成

可配置为：屏蔽位模式和标识符列表屏蔽模式，列表模式下屏蔽寄存器也被当做标识符寄存器使用

关注寄存器映像：

- 一个32位过滤器：STDID[10:0]、EXTID[17:0]、IDE 和 RTR 位

- 两个16位过滤器：STDID[10:0]、IDE、RTR 和 EXTID[17:15]位

![image-20250425164144476](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425164144476.png)

### 时间特性

![image-20250425164654759](C:/Users/17721/AppData/Roaming/Typora/typora-user-images/image-20250425164654759.png)

### 寄存器

- 主控寄存器：INRQ位，初始化时该位要置1，初始化结束后该位再置0
- `CAN_TDLxR` ：配合上高字节寄存器正好储存8字节数据
- `CAN_RIxR` ：CAN接收FIFO邮箱标识符寄存器，**可以通过读取该寄存器查询接收报文的标识符**

# HAL库驱动

- 通讯状态结构体：是一个枚举类型
- 发送结构体：注意ID填在哪个成员下
- 接收数据函数：接收结构体指针参数是存放数据信息的

# project

第一个git的项目

目标：

用stm32f1的环回模式发送标准数据帧，开启中断，使用wtr的api，并用串口打印到vofa＋

内容包括数据和id




































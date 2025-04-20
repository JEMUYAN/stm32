# IIC简介

## 特点：

1. 数据线SDA和时钟线SCL都是双向线路，都通过一个电流源或上拉电阻连接到正的电压，所以当总线空闲时，这两条线路都是高电平
2. 数据传输速率：标准模式 100 kbit/s ；快速模式 400 bits/s ；高速模式 3.4 Mbit/s
3. 通信总线挂载的设备有数量限制
4. 支持多主机和多从机

## 时序

![image-20250416164125151](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416164125151.png)

- 起始&停止信号：开头和结尾
- 应答信号：

分清楚两个概念：主机和从机；接收器和发送器；

对于接收器和应答器：应答信号是发送器在时钟脉冲9前释放SDA，让接收器接管，通过应答信号判断接收是否成功

对于主机和从机：若接收器为主机，则在接收到最后一个字节后发送NACK信号，同质化发送器结束数据发送并且释放SDA，以便主机发送停止信号。

补充：主机是提供时序、给出起始/停止信号、发起并控制通信的设备

- 数据有效性：尽心数据传送时，时钟信号在高电平期间数据线上的数据必须保持稳定
- 空闲状态：SDA和SCL都为高电平

## 通信过程图

应答信号要跟前面数据代表的目的关联，要分清楚是从机/主机 or 发送器/接收器 给出的

发送器和接收器是相对的，比如读操作中发起请求时主机是发送器，接收数据时从机是发送器

### 写操作

![image-20250416165030668](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416165030668.png)

### 读操作

![image-20250416165113903](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416165113903.png)

## 24C02

2K bit 的串行 EEPROM 存储器，内部含有 256 个字节。

![image-20250416165428226](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416165428226.png)

WP：写保护引脚，接高电平只读，接地允许读和写

地址 = 可编程部分（Ax引脚） + 不可编程部分，最后一位用于设置数据传输方向，0是写操作

![image-20250416165554354](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416165554354.png)



### 写操作

![image-20250416170323224](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416170323224.png)

### 读操作

![image-20250416170334074](C:\Users\17721\AppData\Roaming\Typora\typora-user-images\image-20250416170334074.png)

# 软件模拟IIC






























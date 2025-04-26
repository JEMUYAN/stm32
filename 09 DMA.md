# DMA简介

DMA，全称为：Direct Memory Access，即直接存储器访问。

STM32F1中有2个DMA控制器，DMA1有7个通道，DMA2有5个通道，每个通道**专门用来管理来自一个或多个外设对存储器的访问请求。**还有一个仲裁器来协调各个DMA请求的优先级（主要是针对不同的通道）

增量模式和非增量模式：外设、寄存器一般非增量；存储器存储数组一般增量

## 框图

![屏幕截图 2025-04-26 131516](imagines/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-04-26%20131516.png)

仲裁：

每个通道都有自己的一套寄存器、FIFO（如果有）、数据计数器等。但是它们公用系统总线（AHB）、内存（SRAM/Flash）、外设接口，需要处理冲突

不会出现两个外设同时绑定同一 DMA 通道发出请求的情况。F1中每个DMA通道只会绑定一个唯一的外设请求源。**一个通道可以映射的请求可能有很多种，但是能绑定不意味着一定绑定、可以同时绑定。**

软件判定先进性，若没有判定成功再叠加硬件判定（通道编号）

## 寄存器

- DMA中断状态寄存器/中断标志清除寄存器
  - 前者只读，后者对应位写0可以清除前者的置位
  - 常用TCIFx位，即通道DMA传输完成
- 通道x传输数量寄存器：

DMA有单次传输最大限制，由该寄存器控制最大数据数量，数据的字长可以在通道配置寄存器中设置。**可以通过这个寄存器的值查询通道传输进度**；可以使用循环模式避免传输中断

# HAL库驱动

| 函数名                       | 功能                                   |
| ---------------------------- | -------------------------------------- |
| `HAL_DMA_Init()`             | 初始化 DMA（使用 `DMA_HandleTypeDef`） |
| `HAL_DMA_DeInit()`           | 反初始化 DMA                           |
| `HAL_DMA_Start()`            | 启动 DMA 普通传输                      |
| `HAL_DMA_Start_IT()`         | 启动带中断的 DMA 传输                  |
| `HAL_DMA_Abort()`            | 停止 DMA 传输                          |
| `HAL_DMA_IRQHandler()`       | DMA 中断处理函数                       |
| `HAL_DMA_PollForTransfer()`  | 阻塞等待 DMA 传输完成                  |
| `HAL_DMA_RegisterCallback()` | 注册用户自定义回调函数（中断）         |

很多外设会封装DMA，按照这个思路思考即可

```c
typedef struct
{
  DMA_Channel_TypeDef     *Instance;     // DMA通道寄存器地址
  DMA_InitTypeDef         Init;          // 初始化结构体
  HAL_LockTypeDef         Lock;          // 锁定机制
  __IO HAL_DMA_StateTypeDef State;       // 当前状态
  void                    *Parent;       // 指向上层驱动（如UART等）
  void (* XferCpltCallback)(struct __DMA_HandleTypeDef *hdma);     // 传输完成回调
  void (* XferErrorCallback)(struct __DMA_HandleTypeDef *hdma);    // 出错回调
} DMA_HandleTypeDef;

typedef struct
{
  uint32_t Direction;        // 数据方向：M2M, M2P, P2M
  uint32_t PeriphInc;        // 外设地址是否自增
  uint32_t MemInc;           // 存储器地址是否自增
  uint32_t PeriphDataAlignment; // 外设数据宽度
  uint32_t MemDataAlignment;    // 存储器数据宽度
  uint32_t Mode;             // 传输模式：正常/循环
  uint32_t Priority;         // 通道优先级
} DMA_InitTypeDef;
```

```c
void HAL_DMA_IRQHandler(DMA_HandleTypeDef *hdma);  // 中断处理主函数

// 用户可选重写
void HAL_DMA_XferCpltCallback(DMA_HandleTypeDef *hdma);    // 传输完成
void HAL_DMA_XferHalfCpltCallback(DMA_HandleTypeDef *hdma); // 半传输
void HAL_DMA_ErrorCallback(DMA_HandleTypeDef *hdma);       // 错误中断
```

| 宏指令                                               | 功能描述                        | 参数                                             | 示例使用                                                     |
| ---------------------------------------------------- | ------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `__HAL_DMA_GET_FLAG(__HANDLE__, __FLAG__)`           | 检查 DMA 标志是否设置           | `__HANDLE__`：DMA 句柄 `__FLAG__`：标志位        | `if (__HAL_DMA_GET_FLAG(&hdma, DMA_FLAG_TC)) { // 传输完成 }` |
| `__HAL_DMA_GET_IT_STATUS(__HANDLE__, __INTERRUPT__)` | 查询 DMA 中断是否触发           | `__HANDLE__`：DMA 句柄 `__INTERRUPT__`：中断源   | `if (__HAL_DMA_GET_IT_STATUS(&hdma, DMA_IT_TC)) { // 中断触发 }` |
| `__HAL_DMA_GET_IT(__HANDLE__, __INTERRUPT__)`        | 查询 DMA 中断状态               | `__HANDLE__`：DMA 句柄 `__INTERRUPT__`：中断标志 | `if (__HAL_DMA_GET_IT(&hdma, DMA_IT_TC)) { // 传输完成中断 }` |
| `__HAL_DMA_CLEAR_FLAG(__HANDLE__, __FLAG__)`         | 清除 DMA 状态标志               | `__HANDLE__`：DMA 句柄 `__FLAG__`：标志位        | `__HAL_DMA_CLEAR_FLAG(&hdma, DMA_FLAG_TC); // 清除传输完成标志` |
| `__HAL_DMA_CLEAR_IT(__HANDLE__, __INTERRUPT__)`      | 清除 DMA 中断标志               | `__HANDLE__`：DMA 句柄 `__INTERRUPT__`：中断标志 | `__HAL_DMA_CLEAR_IT(&hdma, DMA_IT_TC); // 清除传输完成中断`  |
| `HAL_DMA_GetState(__HANDLE__)`                       | 查询 DMA 当前状态               | `__HANDLE__`：DMA 句柄                           | `if (HAL_DMA_GetState(&hdma) == HAL_DMA_STATE_BUSY) { // DMA 正在忙 }` |
| `__HAL_DMA_GET_IT_SOURCE(__HANDLE__, __INTERRUPT__)` | 查询 DMA 中断是否使能并触发状态 | `__HANDLE__`：DMA 句柄 `__INTERRUPT__`：中断源   | `if (__HAL_DMA_GET_IT_SOURCE(&hdma, DMA_IT_TC)) { // 中断使能 }` |

| 标志或中断    | 描述                                               |
| ------------- | -------------------------------------------------- |
| `DMA_FLAG_TC` | 传输完成标志（Transfer Complete）                  |
| `DMA_FLAG_HT` | 半传输完成标志（Half Transfer Complete）           |
| `DMA_FLAG_TE` | 传输错误标志（Transfer Error）                     |
| `DMA_IT_TC`   | 传输完成中断（Transfer Complete Interrupt）        |
| `DMA_IT_HT`   | 半传输完成中断（Half Transfer Complete Interrupt） |
| `DMA_IT_TE`   | 传输错误中断（Transfer Error Interrupt）           |

注意：标志有区分编号（比如 `DMA1_FLAG_TC1` ）




















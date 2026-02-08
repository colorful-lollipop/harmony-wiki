# UART 接口详细文档

> Universal Asynchronous Receiver/Transmitter 串口通信接口规范

## 模块概述

**头文件**: `iot_uart.h`  
**功能**: 提供 UART 串口的配置、数据收发和流控管理能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 5 个公共函数

## 枚举类型

### IotUartIdxDataBit - 数据位

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_UART_DATA_BIT_5` | 5 | 5 位数据位 |
| `IOT_UART_DATA_BIT_6` | 6 | 6 位数据位 |
| `IOT_UART_DATA_BIT_7` | 7 | 7 位数据位 |
| `IOT_UART_DATA_BIT_8` | 8 | 8 位数据位 |

**证据**: `iot_uart.h:50-59`

### IotUartStopBit - 停止位

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_UART_STOP_BIT_1` | 1 | 1 位停止位 |
| `IOT_UART_STOP_BIT_2` | 2 | 2 位停止位 |

**证据**: `iot_uart.h:67-72`

### IotUartParity - 校验位

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_UART_PARITY_NONE` | 0 | 无校验 |
| `IOT_UART_PARITY_ODD` | 1 | 奇校验 |
| `IOT_UART_PARITY_EVEN` | 2 | 偶校验 |

**证据**: `iot_uart.h:80-87`

### IotUartBlockState - 阻塞状态

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_UART_BLOCK_STATE_NONE_BLOCK` | 0 | 非阻塞模式 |
| `IOT_UART_BLOCK_STATE_BLOCK` | 1 | 阻塞模式 |

**证据**: `iot_uart.h:95-100`

### IotFlowCtrl - 流控模式

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `IOT_FLOW_CTRL_NONE` | 0 | 无流控 |
| `IOT_FLOW_CTRL_RTS_CTS` | 1 | RTS/CTS 硬件流控 |
| `IOT_FLOW_CTRL_RTS_ONLY` | 2 | 仅 RTS |
| `IOT_FLOW_CTRL_CTS_ONLY` | 3 | 仅 CTS |

**证据**: `iot_uart.h:108-117`

## 数据结构

### IotUartAttribute - UART 属性

```c
typedef struct {
    unsigned int baudRate;           // 波特率 (如 9600, 115200)
    IotUartIdxDataBit dataBits;      // 数据位 (5-8)
    IotUartStopBit stopBits;         // 停止位 (1-2)
    IotUartParity parity;            // 校验位
    IotUartBlockState rxBlock;       // 接收阻塞设置
    IotUartBlockState txBlock;       // 发送阻塞设置
    unsigned char pad;              // 填充字节 (对齐)
} IotUartAttribute;
```

**证据**: `iot_uart.h:125-140`

## API 接口

### IoTUartInit - 初始化 UART

```c
unsigned int IoTUartInit(unsigned int id, const IotUartAttribute *param);
```

**功能**: 使用指定的属性配置初始化 UART 设备

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | UART 端口号 (通常 0, 1, 2...) |
| `param` | `const IotUartAttribute *` | 指向 UART 配置属性的指针 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
IotUartAttribute uart_attr = {
    .baudRate = 115200,
    .dataBits = IOT_UART_DATA_BIT_8,
    .stopBits = IOT_UART_STOP_BIT_1,
    .parity = IOT_UART_PARITY_NONE,
    .rxBlock = IOT_UART_BLOCK_STATE_BLOCK,
    .txBlock = IOT_UART_BLOCK_STATE_BLOCK,
    .pad = 0
};

unsigned int ret = IoTUartInit(0, &uart_attr);
```

**证据**: `iot_uart.h:143-155`

---

### IoTUartRead - 读取 UART 数据

```c
int IoTUartRead(unsigned int id, unsigned char *data, unsigned int dataLen);
```

**功能**: 从 UART 设备读取指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | UART 端口号 |
| `data` | `unsigned char *` | 指向接收数据缓冲区的指针 |
| `dataLen` | `unsigned int` | 期望读取的字节数 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| **正整数** | 实际读取到的字节数 |
| **-1** | 读取失败 |

**调用示例**:
```c
unsigned char rx_buf[128];
int read_len = IoTUartRead(0, rx_buf, sizeof(rx_buf) - 1);
if (read_len > 0) {
    rx_buf[read_len] = '\0';  // 添加字符串结束符
    printf("Received: %s\n", rx_buf);
}
```

**证据**: `iot_uart.h:158-169`

---

### IoTUartWrite - 写入 UART 数据

```c
int IoTUartWrite(unsigned int id, const unsigned char *data, unsigned int dataLen);
```

**功能**: 向 UART 设备写入指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | UART 端口号 |
| `data` | `const unsigned char *` | 指向待发送数据的指针 |
| `dataLen` | `unsigned int` | 待发送数据的长度 (字节数) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| **正整数** | 实际发送的字节数 |
| **-1** | 发送失败 |

**调用示例**:
```c
const char *msg = "Hello UART!\n";
int write_len = IoTUartWrite(0, (const unsigned char *)msg, strlen(msg));
```

**证据**: `iot_uart.h:172-183`

---

### IoTUartDeinit - 去初始化 UART

```c
unsigned int IoTUartDeinit(unsigned int id);
```

**功能**: 去初始化指定的 UART 设备，释放相关资源

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | UART 端口号 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 去初始化成功 |
| `IOT_FAILURE` | 去初始化失败 |

**证据**: `iot_uart.h:186-194`

---

### IoTUartSetFlowCtrl - 设置流控

```c
unsigned int IoTUartSetFlowCtrl(unsigned int id, IotFlowCtrl flowCtrl);
```

**功能**: 配置 UART 设备的硬件流控模式

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | UART 端口号 |
| `flowCtrl` | `IotFlowCtrl` | 流控模式枚举值 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**调用示例**:
```c
// 启用 RTS/CTS 硬件流控
IoTUartSetFlowCtrl(0, IOT_FLOW_CTRL_RTS_CTS);
```

**证据**: `iot_uart.h:197-208`

## 完整使用示例

```c
#include "iot_uart.h"
#include "iot_errno.h"
#include <string.h>

#define UART_ID 0

int uart_echo_example(void) {
    unsigned int ret;
    IotUartAttribute uart_attr = {
        .baudRate = 115200,
        .dataBits = IOT_UART_DATA_BIT_8,
        .stopBits = IOT_UART_STOP_BIT_1,
        .parity = IOT_UART_PARITY_NONE,
        .rxBlock = IOT_UART_BLOCK_STATE_BLOCK,
        .txBlock = IOT_UART_BLOCK_STATE_BLOCK,
        .pad = 0
    };

    // 1. 初始化 UART
    ret = IoTUartInit(UART_ID, &uart_attr);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 2. 发送数据
    const char *tx_data = "Hello, UART!\n";
    int sent = IoTUartWrite(UART_ID, (const unsigned char *)tx_data, strlen(tx_data));

    // 3. 接收数据 (回显)
    unsigned char rx_buf[128];
    int recv = IoTUartRead(UART_ID, rx_buf, sizeof(rx_buf) - 1);
    if (recv > 0) {
        rx_buf[recv] = '\0';
        IoTUartWrite(UART_ID, rx_buf, recv);  // 回显
    }

    // 4. 清理
    IoTUartDeinit(UART_ID);

    return IOT_SUCCESS;
}
```

## 常见波特率配置

| 波特率 | 应用场景 |
|--------|----------|
| 9600 | 低速设备、调试输出 |
| 19200 | 传统串口设备 |
| 38400 | 工业设备 |
| 57600 | 通用串口通信 |
| 115200 | 高速串口调试 |
| 230400 | 高速数据通信 |
| 460800 | 高速数据传输 |
| 921600 | 高速日志输出 |

## 注意事项

### 1. 阻塞 vs 非阻塞

- **阻塞模式**: `IoTUartRead` 会等待直到收到指定长度的数据或超时
- **非阻塞模式**: 立即返回，实际读取字节数可能小于请求长度

### 2. 数据完整性

- 发送数据时确保 `dataLen` 与实际数据长度一致
- 接收数据后建议检查返回值是否为预期长度

### 3. 硬件流控

仅在以下情况启用硬件流控：
- 高速传输 (> 115200 bps)
- 长时间连续数据传输
- 对端设备支持 RTS/CTS

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [GPIO 接口](API_GPIO.md)
- [I2C 接口](API_I2C.md)
- [架构说明](03_Architecture.md)

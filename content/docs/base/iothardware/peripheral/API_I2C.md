# I2C 接口详细文档

> Inter-Integrated Circuit 总线通信接口规范

## 模块概述

**头文件**: `iot_i2c.h`  
**功能**: 提供 I2C 总线的初始化、读写操作和波特率配置能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 5 个公共函数

## API 接口

### IoTI2cInit - 初始化 I2C

```c
unsigned int IoTI2cInit(unsigned int id, unsigned int baudrate);
```

**功能**: 使用指定的波特率初始化 I2C 设备

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | I2C 设备 ID (通常 0 或 1) |
| `baudrate` | `unsigned int` | I2C 波特率 (单位: Hz) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
// 初始化 I2C0，波特率 100kHz (标准模式)
unsigned int ret = IoTI2cInit(0, 100000);

// 初始化 I2C0，波特率 400kHz (快速模式)
ret = IoTI2cInit(0, 400000);
```

**证据**: `iot_i2c.h:45-56`

---

### IoTI2cDeinit - 去初始化 I2C

```c
unsigned int IoTI2cDeinit(unsigned int id);
```

**功能**: 去初始化指定的 I2C 设备，释放相关资源

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | I2C 设备 ID |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 去初始化成功 |
| `IOT_FAILURE` | 去初始化失败 |

**证据**: `iot_i2c.h:59-67`

---

### IoTI2cWrite - 写入 I2C 数据

```c
unsigned int IoTI2cWrite(unsigned int id, unsigned short deviceAddr,
                         const unsigned char *data, unsigned int dataLen);
```

**功能**: 向 I2C 从设备写入指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | I2C 设备 ID |
| `deviceAddr` | `unsigned short` | I2C 从设备地址 (7-bit 或 10-bit) |
| `data` | `const unsigned char *` | 指向待写入数据的指针 |
| `dataLen` | `unsigned int` | 待写入数据的长度 (字节数) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 写入成功 |
| `IOT_FAILURE` | 写入失败 |

**调用示例**:
```c
unsigned char write_data[] = {0x01, 0x02, 0x03};
unsigned int ret = IoTI2cWrite(0, 0x50, write_data, sizeof(write_data));
```

**证据**: `iot_i2c.h:70-83`

---

### IoTI2cRead - 读取 I2C 数据

```c
unsigned int IoTI2cRead(unsigned int id, unsigned short deviceAddr,
                        unsigned char *data, unsigned int dataLen);
```

**功能**: 从 I2C 从设备读取指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | I2C 设备 ID |
| `deviceAddr` | `unsigned short` | I2C 从设备地址 |
| `data` | `unsigned char *` | 指向接收数据缓冲区的指针 |
| `dataLen` | `unsigned int` | 待读取数据的长度 (字节数) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 读取成功 |
| `IOT_FAILURE` | 读取失败 |

**调用示例**:
```c
unsigned char read_buf[10];
unsigned int ret = IoTI2cRead(0, 0x50, read_buf, sizeof(read_buf));
if (ret == IOT_SUCCESS) {
    // read_buf 中已保存读取的数据
}
```

**证据**: `iot_i2c.h:86-99`

---

### IoTI2cSetBaudrate - 设置 I2C 波特率

```c
unsigned int IoTI2cSetBaudrate(unsigned int id, unsigned int baudrate);
```

**功能**: 动态设置 I2C 设备的通信波特率

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `unsigned int` | I2C 设备 ID |
| `baudrate` | `unsigned int` | 目标波特率 (单位: Hz) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 设置成功 |
| `IOT_FAILURE` | 设置失败 |

**证据**: `iot_i2c.h:101-111`

## 完整使用示例

```c
#include "iot_i2c.h"
#include "iot_errno.h"

#define I2C_BUS_0    0
#define SENSOR_ADDR  0x68  // 示例传感器地址

int i2c_sensor_example(void) {
    unsigned int ret;
    unsigned char write_data[2] = {0x00, 0x01};
    unsigned char read_data[2] = {0};

    // 1. 初始化 I2C 总线 (400kHz)
    ret = IoTI2cInit(I2C_BUS_0, 400000);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 2. 向传感器写入寄存器地址
    ret = IoTI2cWrite(I2C_BUS_0, SENSOR_ADDR, write_data, 1);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 3. 从传感器读取数据
    ret = IoTI2cRead(I2C_BUS_0, SENSOR_ADDR, read_data, 1);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 4. 清理
    IoTI2cDeinit(I2C_BUS_0);

    return IOT_SUCCESS;
}
```

## I2C 通信模式

### 标准模式 (Standard Mode)
- **波特率**: 100 kHz
- **特点**: 兼容性好，速度较慢
- **应用**: EEPROM、RTC 等低速设备

### 快速模式 (Fast Mode)
- **波特率**: 400 kHz
- **特点**: 速度提升 4 倍，兼容性仍然较好
- **应用**: 多数传感器

### 快速模式增强 (Fast Mode Plus)
- **波特率**: 1 MHz
- **特点**: 更高速度，距离较短
- **应用**: 高速传感器

### 高速模式 (High Speed Mode)
- **波特率**: 3.4 MHz
- **特点**: 最高速度，需要专用硬件支持
- **应用**: 大容量存储器

## 常见问题

### Q: I2C 通信失败如何排查？

1. **检查设备地址**: 确认从设备地址正确（注意 7-bit 与 8-bit 格式区分）
2. **检查连接**: SDA、SCL 线是否正确连接，是否有上拉电阻
3. **检查波特率**: 确认主从设备波特率匹配
4. **检查时序**: 高速模式下注意线路电容限制

### Q: 如何处理多从设备冲突？

**方案**: 使用 `IoTI2cWrite` 发送寄存器地址后，再用 `IoTI2cRead` 读取数据

```c
// 向从设备发送寄存器地址
IoTI2cWrite(id, device_addr, &reg_addr, 1);
// 从该寄存器读取数据
IoTI2cRead(id, device_addr, data, len);
```

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [GPIO 接口](API_GPIO.md)
- [UART 接口](API_UART.md)
- [架构说明](03_Architecture.md)

# 错误码定义文档

> Error Codes 错误码与返回值规范

## 模块概述

**头文件**: `iot_errno.h`  
**功能**: 定义 IoT Hardware 模块的返回值常量  
**依赖**: 无（基础定义文件）  
**宏定义数量**: 2 个

## 错误码定义

### IOT_SUCCESS - 操作成功

```c
#define IOT_SUCCESS    0
```

**值**: `0`

**说明**: 所有需要返回状态的函数，成功时返回此值

**使用示例**:
```c
unsigned int ret = IoTGpioInit(0);
if (ret == IOT_SUCCESS) {
    // 初始化成功
}
```

**证据**: `iot_errno.h:45`

---

### IOT_FAILURE - 操作失败

```c
#define IOT_FAILURE   (-1)
```

**值**: `-1`

**说明**: 操作失败时返回此值，具体失败原因需参考芯片文档

**使用示例**:
```c
unsigned int ret = IoTI2cInit(0, 100000);
if (ret == IOT_FAILURE) {
    // 初始化失败，处理错误
}
```

**证据**: `iot_errno.h:51`

---

## 返回值约定

### 标准返回值模式

| 场景 | 返回值类型 | 说明 |
|------|------------|------|
| **布尔操作** | `IOT_SUCCESS` / `IOT_FAILURE` | 成功/失败 |
| **数据读写** | **正整数** / **-1** | 实际字节数/失败 |
| **特殊操作** | **芯片特定值** | 需参考 HAL 文档 |

### 数据读写函数返回值

```c
// UART 读写返回实际传输字节数
int IoTUartRead(unsigned int id, unsigned char *data, unsigned int dataLen);
int IoTUartWrite(unsigned int id, const unsigned char *data, unsigned int dataLen);

// 示例
int len = IoTUartRead(0, buf, sizeof(buf));
if (len > 0) {
    // 成功读取 len 字节
} else if (len == -1) {
    // 读取失败
}
```

## 错误处理模式

### 1. 简单错误检查

```c
if (IoTGpioInit(pin) != IOT_SUCCESS) {
    return ERROR_GPIO_INIT_FAILED;
}
```

### 2. 链式错误检查

```c
unsigned int ret;

ret = IoTFlashInit();
if (ret != IOT_SUCCESS) goto cleanup_0;

ret = IoTFlashErase(addr, size);
if (ret != IOT_SUCCESS) goto cleanup_1;

ret = IoTFlashWrite(addr, size, data, 1);
if (ret != IOT_SUCCESS) goto cleanup_1;

IoTFlashDeinit();
return SUCCESS;

cleanup_1:
    IoTFlashDeinit();
cleanup_0:
    return ERROR;
```

### 3. 错误码转换

```c
// 将 HAL 错误码转换为标准错误码
unsigned int translate_hal_error(int hal_code) {
    if (hal_code >= 0) {
        return IOT_SUCCESS;  // HAL 成功
    }

    // HAL 特定错误码转换
    switch (hal_code) {
        case -1: return IOT_FAILURE;  // 通用错误
        case -2: return ERROR_BUSY;   // 设备忙
        case -3: return ERROR_TIMEOUT;// 超时
        default: return IOT_FAILURE;  // 未知错误
    }
}
```

## HAL 层扩展错误码

**注意**: HAL 层实现可能返回芯片特定的错误码，具体含义需参考 Hi3861 芯片手册。

### 常见 HAL 错误码

| HAL 返回值 | 含义 | 处理建议 |
|------------|------|----------|
| 正值 | 成功/字节数 | 正常使用 |
| -1 | 通用失败 | 检查参数和状态 |
| -2 | 设备忙 | 重试或等待 |
| -3 | 超时 | 检查连接和通信参数 |
| -4 | 无效参数 | 检查输入参数 |
| -5 | 权限错误 | 检查设备访问权限 |
| -0x100+ | 硬件错误 | 检查硬件连接 |

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [GPIO 接口](API_GPIO.md)
- [I2C 接口](API_I2C.md)
- [UART 接口](API_UART.md)

# Flash 接口详细文档

> Flash Memory 闪存存储接口规范

## 模块概述

**头文件**: `iot_flash.h`  
**功能**: 提供 Flash 存储器的初始化、读、写、擦除操作能力  
**依赖**: `iot_errno.h` (错误码)  
**函数数量**: 5 个公共函数

## API 接口

### IoTFlashInit - 初始化 Flash

```c
unsigned int IoTFlashInit(void);
```

**功能**: 初始化 Flash 存储设备

**参数**: 无

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 初始化成功 |
| `IOT_FAILURE` | 初始化失败 |

**调用示例**:
```c
unsigned int ret = IoTFlashInit();
if (ret != IOT_SUCCESS) {
    // 处理错误
}
```

**证据**: `iot_flash.h:89-96`

---

### IoTFlashDeinit - 去初始化 Flash

```c
unsigned int IoTFlashDeinit(void);
```

**功能**: 去初始化 Flash 存储设备，释放相关资源

**参数**: 无

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 去初始化成功 |
| `IOT_FAILURE` | 去初始化失败 |

**证据**: `iot_flash.h:99-106`

---

### IoTFlashRead - 读取 Flash 数据

```c
unsigned int IoTFlashRead(unsigned int flashOffset, unsigned int size,
                          unsigned char *ramData);
```

**功能**: 从 Flash 指定地址读取指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `flashOffset` | `unsigned int` | Flash 偏移地址 |
| `size` | `unsigned int` | 读取数据长度 (字节数) |
| `ramData` | `unsigned char *` | 指向接收数据缓冲区的指针 |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 读取成功 |
| `IOT_FAILURE` | 读取失败 |

**调用示例**:
```c
unsigned char read_buf[256];
unsigned int ret = IoTFlashRead(0x10000, sizeof(read_buf), read_buf);
```

**证据**: `iot_flash.h:45-57`

---

### IoTFlashWrite - 写入 Flash 数据

```c
unsigned int IoTFlashWrite(unsigned int flashOffset, unsigned int size,
                           const unsigned char *ramData, unsigned char doErase);
```

**功能**: 向 Flash 指定地址写入指定长度的数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `flashOffset` | `unsigned int` | Flash 目标地址 |
| `size` | `unsigned int` | 写入数据长度 (字节数) |
| `ramData` | `const unsigned char *` | 指向待写入数据的指针 |
| `doErase` | `unsigned char` | 是否自动擦除 (1=擦除, 0=不擦除) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 写入成功 |
| `IOT_FAILURE` | 写入失败 |

**调用示例**:
```c
unsigned char write_data[] = "Hello Flash!";
// 写入数据并自动擦除
unsigned int ret = IoTFlashWrite(0x10000, sizeof(write_data), write_data, 1);
```

**注意**:
- Flash 写入前必须先擦除（Flash 只能将 1 改写为 0）
- `doErase=1` 时会自动执行擦除操作
- `doErase=0` 时假设目标区域已擦除（直接写入）

**证据**: `iot_flash.h:60-74`

---

### IoTFlashErase - 擦除 Flash 数据

```c
unsigned int IoTFlashErase(unsigned int flashOffset, unsigned int size);
```

**功能**: 擦除 Flash 指定地址范围内数据

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `flashOffset` | `unsigned int` | Flash 起始地址 |
| `size` | `unsigned int` | 擦除数据长度 (字节数) |

**返回值**:
| 返回值 | 说明 |
|--------|------|
| `IOT_SUCCESS` | 擦除成功 |
| `IOT_FAILURE` | 擦除失败 |

**注意**:
- Flash 擦除是按块（Block）或扇区（Sector）进行的
- `size` 参数通常需要对齐到擦除块大小
- 擦除操作会耗时较长（毫秒级）

**调用示例**:
```c
// 擦除 4KB 区域
unsigned int ret = IoTFlashErase(0x10000, 4096);
```

**证据**: `iot_flash.h:77-86`

## 完整使用示例

```c
#include "iot_flash.h"
#include "iot_errno.h"

#define FLASH_ADDR  0x10000
#define FLASH_SIZE  256

int flash_data_store_example(void) {
    unsigned int ret;
    unsigned char write_data[FLASH_SIZE] = {0};
    unsigned char read_data[FLASH_SIZE] = {0};

    // 1. 初始化 Flash
    ret = IoTFlashInit();
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 2. 准备数据
    snprintf(write_data, FLASH_SIZE, "Data saved at %lu", clock());

    // 3. 写入数据（自动擦除）
    ret = IoTFlashWrite(FLASH_ADDR, FLASH_SIZE, write_data, 1);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    // 4. 读取数据验证
    ret = IoTFlashRead(FLASH_ADDR, FLASH_SIZE, read_data);
    if (ret != IOT_SUCCESS) {
        return ret;
    }

    printf("Read: %s\n", read_data);

    // 5. 清理
    IoTFlashDeinit();

    return IOT_SUCCESS;
}
```

## Flash 操作注意事项

### 1. 擦除与写入

Flash 存储器的特性：

| 操作 | 次数限制 | 说明 |
|------|----------|------|
| **擦除** | ~10,000 次/块 | 按块擦除，不可按字节 |
| **写入** | ~100,000 次/地址 | 可按字节/字写入 |

### 2. 对齐要求

```
┌─────────────────────────────────────────────────────────────┐
│ Flash 存储结构                                               │
├─────────────────────────────────────────────────────────────┤
│  Block 0 (4KB)                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Sector 0 (256B)  │  Sector 1 (256B)  │  ...        │   │
│  └─────────────────────────────────────────────────────┘   │
│  Block 1 (4KB)                                               │
│  ...                                                         │
└─────────────────────────────────────────────────────────────┘

写入地址和大小应对齐到 Sector 边界
擦除地址和大小应对齐到 Block 边界
```

### 3. 数据保持

| 特性 | 说明 |
|------|------|
| **数据保持时间** | > 10 年（室温） |
| **擦写循环** | 约 10,000-100,000 次 |

## 相关文档

- [API 参考总览](01_API_Reference.md)
- [API 参考总览](01_API_Reference.md)
- [安全评审](05_Security_Review.md)

# hievent API 参考

## 概述

hievent 模块提供两套 API：

1. **驱动层 API** (`hievent_driver.h`): 字符设备底层接口
2. **事件层 API** (`hiview_hievent.h`): 事件对象构造与管理

## 驱动层 API

### HieventInit

**函数原型**:
```c
int HieventInit(void);
```

**功能说明**:
初始化 hievent 驱动，注册字符设备节点。

**调用时机**:
- 内核模块加载时自动调用
- 初始化级别: `LOS_INIT_LEVEL_KMOD_EXTENDED`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 0 | 成功 |
| -ENOMEM | 内存分配失败 |

**代码位置**: `hievent_driver.c:377-387`

```c
int HieventInit(void)
{
    int ret = HieventDeviceInit();
    if (ret != 0) {
        return ret;
    }
    register_driver("/dev/hwlog_exception", &g_hieventFops,
                    DRIVER_MODE, &g_hieventDev);
    return 0;
}
```

### HieventWriteInternal

**函数原型**:
```c
int HieventWriteInternal(const char *buffer, size_t buflen);
```

**功能说明**:
向环形缓冲区写入事件数据。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| buffer | const char* | 写入数据缓冲区 |
| buflen | size_t | 数据长度 |

**参数校验**:
- `buflen < sizeof(int)`: 返回 `-EINVAL`
- `buflen > HIEVENT_LOG_BUFFER - sizeof(struct HieventEntry)`: 返回 `-EINVAL`
- `buffer` 为用户地址: 返回 `-EINVAL`
- `buffer[0] != CHECK_CODE`: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| >0 | 写入的负载长度 |
| -EINVAL | 参数无效 |
| -ENOMEM | 缓冲区满 |

**代码位置**: `hievent_driver.c:268-321`

## 事件层 API

### HiviewHieventCreate

**函数原型**:
```c
struct HiviewHievent *HiviewHieventCreate(unsigned int eventid);
```

**功能说明**:
创建新的 hievent 事件对象。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| eventid | unsigned int | 事件 ID |

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 非 NULL | 事件对象指针 |
| NULL | 内存分配失败 |

**代码位置**: `hiview_hievent.c:153-168`

### HiviewHieventPutIntegral

**函数原型**:
```c
int HiviewHieventPutIntegral(struct HiviewHievent *event,
                             const char *key, long value);
```

**功能说明**:
向事件添加整型键值对。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| event | struct HiviewHievent* | 事件对象 |
| key | const char* | 键名 |
| value | long | 整型值 |

**参数校验**:
- `event == NULL`: 返回 `-EINVAL`
- `key == NULL`: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 0 | 成功 |
| -EINVAL | 参数无效 |
| -ENOMEM | 内存分配失败 |

**代码位置**: `hiview_hievent.c:170-207`

### HiviewHieventPutString

**函数原型**:
```c
int HiviewHieventPutString(struct HiviewHievent *event,
                          const char *key, const char *value);
```

**功能说明**:
向事件添加字符串键值对。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| event | struct HiviewHievent* | 事件对象 |
| key | const char* | 键名 |
| value | const char* | 字符串值 |

**长度限制**:
- 最大字符串长度: `MAX_STR_LEN` = 10 * 1024 = 10240 字节

**参数校验**:
- `event == NULL`: 返回 `-EINVAL`
- `key == NULL`: 返回 `-EINVAL`
- `value == NULL`: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 0 | 成功 |
| -EINVAL | 参数无效 |
| -ENOMEM | 内存分配失败 |

**代码位置**: `hiview_hievent.c:209-249`

### HiviewHieventSetTime

**函数原型**:
```c
int HiviewHieventSetTime(struct HiviewHievent *event, long long seconds);
```

**功能说明**:
设置事件时间戳。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| event | struct HiviewHievent* | 事件对象 |
| seconds | long long | UTC 时间戳（秒） |

**参数校验**:
- `event == NULL`: 返回 `-EINVAL`
- `seconds == 0`: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 0 | 成功 |
| -EINVAL | 参数无效 |

**代码位置**: `hiview_hievent.c:251-259`

### HiviewHieventAddFilePath

**函数原型**:
```c
int HiviewHieventAddFilePath(struct HiviewHievent *event, const char *path);
```

**功能说明**:
向事件添加文件路径，用于附带日志文件。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| event | struct HiviewHievent* | 事件对象 |
| path | const char* | 文件路径 |

**路径限制**:
- 最大路径长度: `MAX_PATH_LEN` = 256 字节
- 最大路径数量: `MAX_PATH_NUMBER` = 10

**参数校验**:
- `event == NULL`: 返回 `-EINVAL`
- `path == NULL` 或为空: 返回 `-EINVAL`
- `path` 长度超过 256: 返回 `-EINVAL`
- 路径数量超过 10: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| 0 | 成功 |
| -EINVAL | 参数无效或路径超限 |
| -ENOMEM | 内存分配失败 |

**代码位置**: `hiview_hievent.c:294-301`

### HiviewHieventReport

**函数原型**:
```c
int HiviewHieventReport(struct HiviewHievent *obj);
```

**功能说明**:
上报事件到内核驱动。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| obj | struct HiviewHievent* | 事件对象 |

**内部流程**:
1. 调用 `HiviewHieventConvertString()` 序列化事件
2. 调用 `HiviewHieventWriteLogException()` 分包写入
3. 释放序列化缓冲区

**参数校验**:
- `obj == NULL`: 返回 `-EINVAL`

**返回值**:
| 返回值 | 含义 |
|-------|------|
| >=0 | 发送的数据包数量 |
| -EINVAL | 参数无效 |

**代码位置**: `hiview_hievent.c:501-521`

### HiviewHieventDestroy

**函数原型**:
```c
void HiviewHieventDestroy(struct HiviewHievent *event);
```

**功能说明**:
销毁事件对象，释放所有关联资源。

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| event | struct HiviewHievent* | 事件对象 |

**资源释放**:
- 遍历并释放 Payload 链表
- 释放所有文件路径字符串
- 释放事件对象本身

**代码位置**: `hiview_hievent.c:523-544`

### HiviewHieventFlush

**函数原型**:
```c
void HiviewHieventFlush(void);
```

**功能说明**:
刷新所有待处理事件到日志文件。

**内部实现**:
```c
void HiviewHieventFlush(void)
{
    struct HiviewHievent *hievent = HiviewHieventCreate(0x7BBE69BD);
    (void)HiviewHieventReport(hievent);
    HiviewHieventDestroy(hievent);
}
```

**代码位置**: `hiview_hievent.c:546-552`

## 设备节点使用

### 节点信息

| 属性 | 值 |
|-----|---|
| 设备路径 | `/dev/hwlog_exception` |
| 设备类型 | 字符设备 |
| 权限模式 | 0666 (所有用户可读写) |

### 标准接口

| 接口 | 支持状态 | 说明 |
|-----|---------|------|
| `open()` | ✅ | 总是返回 0 |
| `close()` | ✅ | 总是返回 0 |
| `read()` | ✅ | 阻塞读取事件 |
| `write()` | ✅ | 写入事件 |
| `ioctl()` | ⚠️ | LiteOS 暂不支持 |
| `mmap()` | ❌ | 未实现 |
| `poll()` | ✅ | POLLOUT 总是可写 |

### 使用示例

```c
#include <fcntl.h>
#include <unistd.h>
#include "hiview_hievent.h"

// 创建并上报事件
void ExampleReportEvent(void)
{
    int fd;
    struct HiviewHievent *event;

    // 方式 1: 使用事件层 API
    event = HiviewHieventCreate(0x1234);
    HiviewHieventPutIntegral(event, "error_code", -1);
    HiviewHieventPutString(event, "module", "example");
    HiviewHieventReport(event);
    HiviewHieventDestroy(event);

    // 方式 2: 直接操作设备节点
    fd = open("/dev/hwlog_exception", O_WRONLY);
    if (fd >= 0) {
        // 写入事件数据
        write(fd, data, len);
        close(fd);
    }
}
```

## 错误码汇总

| 错误码 | 定义位置 | 含义 |
|-------|---------|------|
| 0 | - | 成功 |
| -EINVAL | `errno.h` | 参数无效 |
| -ENOMEM | `errno.h` | 内存不足 |

---

*证据来源*:
- 函数签名与实现: `hievent_driver.c`, `hiview_hievent.c`
- 头文件定义: `hievent_driver.h`, `hiview_hievent.h`
- 宏定义: `hiview_hievent.c:48-58`

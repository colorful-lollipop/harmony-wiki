# 附录：配置项与宏定义

## 概述

本文档汇总 hievent 模块中的关键配置宏、编译开关和运行时参数，便于查阅和调试。

## 编译时配置

### Kconfig 配置

| 配置项 | 类型 | 默认值 | 依赖 | 说明 |
|-------|------|-------|------|------|
| `DRIVERS_HIEVENT` | bool | n | `DRIVERS` | 启用 hievent 驱动 |

**启用方式**:
```bash
# menuconfig
make menuconfig
# Device Drivers → [*] Enable hievent
```

**直接修改**:
```bash
echo "LOSCFG_DRIVERS_HIEVENT=y" >> .config
```

---

### GN 构建变量

| 变量名 | 来源 | 说明 |
|-------|------|------|
| `LOSCFG_DRIVERS_HIEVENT` | Kconfig | GN 构建开关 |
| `module_switch` | BUILD.gn | `defined(LOSCFG_DRIVERS_HIEVENT)` |
| `module_name` | BUILD.gn | `"drivers/liteos/hievent"` |

**参考**: `hievent/BUILD.gn:32-33`

---

## 运行时参数

### 环形缓冲区配置

| 宏定义 | 值 | 单位 | 说明 |
|-------|-----|------|------|
| `HIEVENT_LOG_BUFFER` | 1024 | 字节 | 环形缓冲区总大小 |
| `DRIVER_MODE` | 0666 | 八进制 | 设备节点权限 |

**证据**: `hievent_driver.c:56-57`

---

### 事件对象配置

| 宏定义 | 值 | 说明 |
|-------|-----|------|
| `MAX_PATH_NUMBER` | 10 | 最大文件路径数量 |
| `MAX_PATH_LEN` | 256 | 路径字符串最大长度 |
| `MAX_STR_LEN` | 10*1024 | 字符串值最大长度 |

**证据**: `hiview_hievent.c:35, 50-51`

---

### 整型值配置

| 宏定义 | 值 | 说明 |
|-------|-----|------|
| `INT_TYPE_MAX_LEN` | 21 | 整型值字符串表示最大长度 |

**证据**: `hiview_hievent.c:48`

---

### 缓冲区配置

| 宏定义 | 值 | 说明 |
|-------|-----|------|
| `EVENT_INFO_BUF_LEN` | 64*1024 | 事件信息缓冲区总长度 |
| `EVENT_INFO_PACK_BUF_LEN` | 2*1024 | 单个分包最大长度 |

**证据**: `hiview_hievent.c:54-55`

---

### 校验码

| 宏定义 | 值 | 说明 |
|-------|-----|------|
| `CHECK_CODE` | 0x7BCDABCD | 事件写入校验码 |
| `IDAP_LOGTYPE_CMD` | 1 | IDAP 日志类型 |

**证据**: `hievent_driver.h:37`, `hiview_hievent.c:472`

---

### 特殊事件 ID

| 事件 ID | 值 | 用途 |
|--------|-----|------|
| 刷新事件 | 0x7BBE69BD | `HiviewHieventFlush()` 使用 |

**证据**: `hiview_hievent.c:549`

---

## 头文件导出

### hievent_driver.h 导出

```c
// 宏
#define CHECK_CODE 0x7BCDABCD

// 类型
struct IdapHeader {
    char level;
    char category;
    char logType;
    char sn;
};

// 函数
int HieventInit(void);
int HieventWriteInternal(const char *buffer, size_t buflen);
```

### hiview_hievent.h 导出

```c
// 宏
#define MAX_PATH_NUMBER 10

// 类型
struct HiviewHievent { /* ... */ };
struct HiviewHieventPayload { /* ... */ };

// 函数
struct HiviewHievent *HiviewHieventCreate(unsigned int eventid);
int HiviewHieventPutIntegral(struct HiviewHievent *event,
                              const char *key, long value);
int HiviewHieventPutString(struct HiviewHievent *event,
                            const char *key, const char *value);
int HiviewHieventSetTime(struct HiviewHievent *event, long long seconds);
int HiviewHieventAddFilePath(struct HiviewHievent *event, const char *path);
int HiviewHieventReport(struct HiviewHievent *obj);
void HiviewHieventDestroy(struct HiviewHievent *event);
void HiviewHieventFlush(void);
```

---

## 设备节点

| 属性 | 值 |
|-----|---|
| 路径 | `/dev/hwlog_exception` |
| 类型 | 字符设备 |
| 权限 | 0666 |
| 主设备号 | 自动分配 |
| 从设备号 | 0 |

**注册位置**: `hievent_driver.c:384-385`

---

## 初始化级别

| 级别名称 | 宏定义 | 阶段 |
|---------|-------|------|
| 内核模块扩展 | `LOS_INIT_LEVEL_KMOD_EXTENDED` | 驱动初始化完成后 |

**使用方式**:
```c
LOS_MODULE_INIT(HieventInit, LOS_INIT_LEVEL_KMOD_EXTENDED);
```

**证据**: `hievent_driver.c:389`

---

## 错误码

| 错误码 | 含义 | 使用位置 |
|-------|------|---------|
| 0 | 成功 | - |
| -EINVAL | 参数无效 | 参数校验失败 |
| -ENOMEM | 内存不足 | 内存分配失败 |

---

*最后更新: 2024*
*基于代码版本: 当前 Git HEAD*

# 05 - API/接口差异

## 5.1 概述

相比上游 f2fs-tools v1.16.0，OpenHarmony 版本新增以下接口和能力：

| 类别 | 新增内容 | 说明 |
|-----|---------|------|
| 日志系统 | `SlogInit`, `SlogWrite`, `KlogWrite` | 增强日志能力 |
| DFX 诊断 | `DmdReport`, `DmdInsertError` | 错误统计和上报 |
| 安全函数 | `strncpy_s`, `vsnprintf_s` 等 | 边界检查函数 |

**说明**: 上游 f2fs-tools 主要提供命令行工具接口，OH 版本增加了编程接口用于系统服务集成。

---

## 5.2 日志系统接口

### 头文件
```c
#include "f2fs_log.h"
```

### 核心数据结构

```c
// 日志类型枚举
enum LogType {
    LOG_TYP_NONE = 0,
    LOG_TYP_FSCK,
    LOG_TYP_DUMP,
    LOG_TYP_DEFRAG,
    LOG_TYP_RESIZE,
    LOG_TYP_MKFS,
    LOG_TYP_MAX
};

// 日志信息结构
struct LogInfo {
    enum LogType type;      // 日志类型
    int slogFd;             // 文件日志文件描述符
    int klogFd;             // kmsg 文件描述符
    int klogLevel;          // 日志级别
    long slogOffset;        // 文件日志偏移
    int needTruncate;       // 是否需要截断
    char *slogFile;         // 日志文件路径
    char *slogFileBak;      // 备份日志文件路径
    char *logDir;           // 日志目录
};

extern struct LogInfo g_logI;
extern char *g_logTag[7];
```

### 日志级别

```c
#define KLOG_ERROR_LEVEL        3
#define KLOG_INFO_LEVEL         6
#define KLOG_DEFAULT_LEVEL      KLOG_INFO_LEVEL
```

### 接口函数

#### 1. SlogInit - 初始化日志系统

```c
int SlogInit(int funcType);
```

**参数**:
- `funcType`: 功能类型，取值如下：
  - `FSCK` (0)
  - `DUMP` (1)
  - `DEFRAG` (2)
  - `RESIZE` (3)

**返回值**:
- `0`: 成功
- `-1`: 失败

**功能**:
- 根据功能类型创建对应的日志文件（如 `fsck.log`）
- 初始化 kmsg 写入
- 创建日志目录（如果不存在）

**日志文件位置**:
- 主路径: `/log/f2fs-tools/`
- 备用路径: `/dev/f2fs-tools/`

---

#### 2. SlogWrite - 写入文件日志

```c
int SlogWrite(const char *fmt, ...);
```

**参数**:
- `fmt`: 格式化字符串
- `...`: 可变参数

**返回值**:
- 写入的字节数，或错误码

**功能**:
- 将格式化的日志写入到 `slogFile`
- 自动处理日志轮转（文件大小超过 256KB 时）

**使用示例**:
```c
SlogWrite("Checking segment %d, status: %s\n", segno, status);
```

---

#### 3. SlogExit - 关闭日志系统

```c
void SlogExit(void);
```

**功能**:
- 刷新并关闭日志文件
- 释放相关资源

---

#### 4. KlogWrite - 写入内核日志

```c
void KlogWrite(int level, const char* fmt, ...);
```

**参数**:
- `level`: 日志级别（3=ERROR, 6=INFO）
- `fmt`: 格式化字符串
- `...`: 可变参数

**功能**:
- 将日志写入 `/dev/kmsg`
- 可通过 `dmesg` 命令查看
- 支持日志级别过滤

**使用示例**:
```c
KlogWrite(KLOG_ERROR_LEVEL, "F2FS: Failed to read superblock\n");
```

---

### 宏定义

```c
// 错误日志宏
#define KLOGE(fmt, ...) \
    KlogWrite(KLOG_ERROR_LEVEL, "<3> %s: " fmt, \
            g_logTag[g_logI.type], ##__VA_ARGS__)

// 信息日志宏
#define KLOGI(fmt, ...) \
    KlogWrite(KLOG_INFO_LEVEL, "<6> %s: " fmt, \
            g_logTag[g_logI.type], ##__VA_ARGS__)

// 文件日志宏
#define SLOG(x...) SlogWrite(x)
```

**使用示例**:
```c
KLOGE("Failed to open device %s: %d\n", dev_path, errno);
KLOGI("Fsck completed in %ld ms\n", duration);
SLOG("Checkpoint version: %lu\n", ckpt_version);
```

---

## 5.3 DFX 诊断接口

### 头文件
```c
#include "f2fs_dmd.h"
```

### 核心数据结构

```c
// DMD 报告结构
struct DmdReport {
    uint64_t errBitmap[NR_ERR_BITMAP_UINT64];  // 错误位图（3 * 64bit）
    unsigned int propBitmap;                    // 属性位图
    unsigned long usedSpace;                    // 已用空间 (MB)
    unsigned long freeSpace;                    // 空闲空间 (MB)
    unsigned int features;                      // F2FS 特性标志
    unsigned short segsPerSec;                  // 每段扇区数
    unsigned short secsPerZone;                 // 每区段数
    unsigned long totalFsSectors;               // 总扇区数
    unsigned long ckptVersion;                  // Checkpoint 版本
    unsigned int ckptState;                     // Checkpoint 状态
    unsigned long costTime;                     // 执行耗时 (ms)
    char msg[FSCK_REPORT_MSG_SIZE];             // 详细消息 (1536 bytes)
};

// 属性标志
#define IS_UNISTORE_FL    0x00000001  // 是否 UniStore
#define FB_LOCKED_FL      0x00000002  // Bootloader 是否锁定
```

### 错误码定义

```c
// f2fs_dmd_errno.h
enum {
    PR_SUCCESS = 0,
    PR_ERROR,
    PR_FATAL_ERROR,
    PR_MISMATCH,
    PR_ISALNUM,
    PR_CHECK_PARTITION,
    PR_CHECK_SIT_TYPES,
    // ... 更多错误码
    PR_OTHER_CORRUPT,
    PR_FSCK_TIME_OVERCOST,  // fsck 执行超时
    NR_ERR_BITMAP_UINT64 = 3
};
```

### 接口函数

#### 1. DmdReport - 上报诊断信息

```c
int DmdReport(void);
```

**返回值**:
- `DMD_OK` (0): 成功
- `DMD_ERR` (-1): 失败

**功能**:
- 填充诊断报告结构
- 读取设备锁定状态
- 通过 ioctl 上报到 `/dev/storage`

**使用时机**:
- fsck 完成时
- 发现严重错误时
- 性能监控触发时

---

#### 2. DmdInsertError - 记录错误

```c
void DmdInsertError(int type, unsigned int err, const char *func, int line);
```

**参数**:
- `type`: 错误类型（如 `LOG_TYP_FSCK`）
- `err`: 错误码（如 `PR_ERROR`）
- `func`: 函数名
- `line`: 行号

**功能**:
- 设置错误位图
- 记录错误发生的函数和行号
- 统计相同错误的出现次数

---

#### 3. DmdCheckCostTime - 检查执行耗时

```c
void DmdCheckCostTime(const char *func, int line);
```

**功能**:
- 检查 fsck 执行时间是否超过阈值
- 根据文件系统大小确定阈值：
  - 512GB 以下: 1000ms
  - 512GB - 1TB: 3000ms
  - 1TB - 2TB: 5000ms

---

### 宏定义

```c
// 设置报告字段
#define DMD_SET_VALUE(field, value) ((g_dmdReport.field) = (value))

// 添加错误（自动记录函数和行号）
#define DMD_ADD_ERROR(type, err) \
    DmdInsertError(type, err, __func__, __LINE__)

// 添加带消息的错误
#define DMD_ADD_MSG_ERROR(type, err, fmt, ...) \
    DmdInsertMsgError(type, err, __func__, __LINE__, \
        "[ERRMSG(%s:%d)"fmt"]", __func__, __LINE__, ##__VA_ARGS__)

// 添加断言消息
#define DMD_ASSERT_MSG(func, line, fmt, ...) \
    DmdAssertMsg("[ASSERT(%s:%d)"fmt"]", func, line, ##__VA_ARGS__)

// 计算文件系统大小
#define COMPUTE_SIZE(sbi) \
    do { \
        uint64_t nodeSecs = round_up((sbi)->total_valid_node_count, ...); \
        uint64_t dataSecs = round_up((sbi)->total_valid_block_count - ...); \
        uint64_t freeBlks = ((sbi)->total_sections - dataSecs - nodeSecs) * ...; \
        uint64_t totalSize = g_dmdReport.totalFsSectors << ...; \
        g_dmdReport.freeSpace = freeBlks >> MB_BLK_SHIFT; \
        g_dmdReport.usedSpace = totalSize - g_dmdReport.freeSpace; \
    } while (0)

// 检查耗时并上报
#define DMD_CHECK_COST_TIME(sbi, costMs) \
    do { \
        g_dmdReport.costTime = (costMs); \
        if (g_dmdReport.usedSpace == 0) { \
            COMPUTE_SIZE(sbi); \
        } \
        DmdCheckCostTime(__func__, __LINE__); \
    } while (0)
```

### 使用示例

```c
#include "f2fs_dmd.h"

void CheckFileSystem(struct f2fs_sb_info *sbi) {
    int ret;
    struct timespec start, end;
    long cost_ms;
    
    // 记录开始时间
    clock_gettime(CLOCK_MONOTONIC, &start);
    
    // 设置文件系统基本信息
    DMD_SET_VALUE(totalFsSectors, sbi->total_sectors);
    DMD_SET_VALUE(segsPerSec, sbi->segs_per_sec);
    DMD_SET_VALUE(secsPerZone, sbi->secs_per_zone);
    DMD_SET_VALUE(ckptVersion, sbi->ckpt->checkpoint_ver);
    DMD_SET_VALUE(ckptState, sbi->ckpt->ckpt_flags);
    
    // 执行检查
    ret = do_fsck(sbi);
    if (ret != 0) {
        // 记录错误
        DMD_ADD_ERROR(LOG_TYP_FSCK, PR_ERROR);
    }
    
    // 发现特定错误
    if (invalid_node_found) {
        DMD_ADD_MSG_ERROR(LOG_TYP_FSCK, PR_FATAL_ERROR,
                         "Invalid node at nid=%u", nid);
    }
    
    // 计算耗时
    clock_gettime(CLOCK_MONOTONIC, &end);
    cost_ms = (end.tv_sec - start.tv_sec) * 1000 + 
              (end.tv_nsec - start.tv_nsec) / 1000000;
    
    // 检查是否超时
    DMD_CHECK_COST_TIME(sbi, cost_ms);
    
    // 上报诊断信息
    DmdReport();
}
```

---

## 5.4 安全函数接口

### 来源
来自 `bounds_checking_function` 组件，通过 `securec.h` 引入。

### 常用安全函数

| 函数 | 说明 | 替代的标准函数 |
|-----|------|--------------|
| `strncpy_s` | 安全的字符串拷贝 | `strncpy` |
| `strcpy_s` | 安全的字符串拷贝 | `strcpy` |
| `strcat_s` | 安全的字符串连接 | `strcat` |
| `memcpy_s` | 安全的内存拷贝 | `memcpy` |
| `memset_s` | 安全的内存设置 | `memset` |
| `vsnprintf_s` | 安全的格式化输出 | `vsnprintf` |
| `snprintf_s` | 安全的格式化输出 | `snprintf` |
| `sprintf_s` | 安全的格式化输出 | `sprintf` |

### 使用示例

```c
#include "securec.h"

// 字符串拷贝
char dest[256];
const char *src = "source string";
strncpy_s(dest, sizeof(dest), src, strlen(src));

// 格式化输出
char buffer[512];
int n = 123;
snprintf_s(buffer, sizeof(buffer), sizeof(buffer) - 1, "Number: %d", n);

// 内存操作
int arr[100];
memset_s(arr, sizeof(arr), 0, sizeof(arr));
```

---

## 5.5 接口使用限制

### 日志系统限制

| 项目 | 限制 |
|-----|------|
| 最大日志文件大小 | 256 KB |
| 日志缓冲区大小 | 512 bytes |
| 支持的日志类型 | 5 种 (FSCK, DUMP, DEFRAG, RESIZE, MKFS) |
| 日志级别 | 2 级 (ERROR=3, INFO=6) |

### DFX 诊断限制

| 项目 | 限制 |
|-----|------|
| 错误存储条目 | 64 条 |
| 错误位图大小 | 192 bits (3 * 64) |
| 消息缓冲区大小 | 1536 bytes |
| 函数名字长度 | 20 bytes |
| 超时阈值档位 | 3 档 |

---

## 5.6 与上游版本的行为差异

### 上游版本
- 所有输出到 stdout/stderr
- 无持久化日志
- 无错误统计和上报

### OH 版本
- 输出到 kmsg 和文件日志
- 支持持久化诊断信息
- 集成到系统 DFX 框架

### 迁移注意事项

1. **日志查看方式变化**:
   ```bash
   # 上游
   fsck.f2fs /dev/block/data 2>&1 | tee fsck.log
   
   # OH
   # 方式1: 查看 kmsg
   dmesg | grep F2FS
   
   # 方式2: 查看文件日志
   cat /log/f2fs-tools/fsck.log
   ```

2. **错误处理方式**:
   - 上游: 仅控制台输出
   - OH: 自动上报到系统 DFX 框架

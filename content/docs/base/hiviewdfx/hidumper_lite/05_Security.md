# 安全风险评审

## 概述

本文档对 `hidumper_lite` 项目进行安全风险评审，分析项目的攻击面、信任边界，并识别潜在的安全风险。每项风险均包含代码证据、可利用路径、影响范围和修复建议。

---

## 评审范围

### 评审目标

本次评审覆盖以下代码和功能：

| 评审对象 | 文件路径 | 评审内容 |
|----------|----------|----------|
| 命令行工具 | `lite/hidumper.c` | 参数解析、设备节点访问、IOCTL 调用 |
| 核心层 | `mini/hidumper_core.c` | AT 命令解析、适配器注册、参数校验 |
| 适配层 | `mini/hidumper_adapter.c` | 弱函数实现、初始化流程 |
| 构建配置 | `BUILD.gn`、`lite/BUILD.gn`、`mini/BUILD.gn` | 编译选项、依赖检查 |
| 模块配置 | `bundle.json` | 权限声明、依赖检查 |

### 未覆盖范围

- 内核态驱动代码（位于内核仓库）
- 平台特定实现（各芯片平台代码）
- 运行时动态加载机制
- 系统调用层面的安全检查

---

## 攻击面分析

### 攻击面清单

| 编号 | 攻击面 | 类型 | 暴露程度 | 证据位置 |
|------|--------|------|----------|----------|
| AS-01 | 命令行参数输入 | 用户输入 | 高 | `lite/hidumper.c:156-203` |
| AS-02 | AT 命令参数输入 | 用户输入 | 高 | `mini/hidumper_core.c:105-153` |
| AS-03 | 内存地址参数输入 | 用户输入 | 高 | `lite/hidumper.c:186-187` |
| AS-04 | 内存大小参数输入 | 用户输入 | 高 | `lite/hidumper.c:186-187` |
| AS-05 | 文件路径参数输入 | 用户输入 | 中 | `lite/hidumper.c:178-181` |
| AS-06 | /dev/hidumper 设备节点 | 系统接口 | 中 | `lite/hidumper.c:208` |
| AS-07 | IOCTL 命令传递 | 内核接口 | 中 | `lite/hidumper.c:85-96` |
| AS-08 | 适配器注册接口 | 系统接口 | 低 | `mini/hidumper_core.c:79-103` |

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     用户空间                              │  │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────┐ │  │
│  │  │  用户/Shell   │───▶│ hidumper     │───▶│/dev/     │ │  │
│  │  │  (不可信)     │    │ (可信)       │    │hidumper  │ │  │
│  │  └──────────────┘    └──────────────┘    │(半可信)  │ │  │
│  │                                               └────┬─────┘ │  │
│  └───────────────────────────────────────────────────┼───────┘  │
│                                                    │           │
│                                          ──────────┼─────────  │
│                                          信任边界   │           │
│                                          ──────────┼─────────  │
│                                                    ▼           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     内核空间                              │  │
│  │                      (可信)                               │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**信任边界说明**：

| 区域 | 信任级别 | 说明 |
|------|----------|------|
| 用户/Shell | 不可信 | 外部输入，需要严格校验 |
| hidumper 工具 | 可信 | 工具代码，需确保自身安全 |
| /dev/hidumper | 半可信 | 内核接口，需由内核保证安全 |
| 内核空间 | 可信 | 操作系统核心，可信执行环境 |

---

## 安全风险清单

### 风险一：内存地址参数缺乏边界检查

| 属性 | 值 |
|------|-----|
| 风险编号 | SEC-01 |
| 风险类型 | 输入验证不完整 |
| 严重程度 | 高 |
| 可利用性 | 中 |

**代码证据**：

```c
// lite/hidumper.c:186-187
param.start = strtoull(argv[TWO_OF_ARGC_PARAMETERS], NULL, BUF_SIZE_16);
param.size = strtoull(argv[THREE_OF_ARGC_PARAMETERS], NULL, BUF_SIZE_16);
// 无边界检查
```

**可利用路径**：

1. 攻击者构造恶意内存地址：`hidumper -m 0x00000000 0xFFFFFFFF`
2. 传递到内核态 `ioctl()` 调用
3. 内核尝试读取无效地址可能导致系统崩溃

**影响**：

| 影响类型 | 描述 |
|----------|------|
| 拒绝服务 | 非法内存访问导致系统崩溃 |
| 信息泄露 | 读取受保护内存区域 |

**修复建议**：

```c
// 添加地址范围验证
#define MIN_ADDR 0x20000000  // 示例：RAM 起始地址
#define MAX_ADDR 0x40000000  // 示例：RAM 结束地址

if (param.start < MIN_ADDR || param.start > MAX_ADDR) {
    printf("Invalid address: 0x%llx\n", param.start);
    return -1;
}
if (param.size > MAX_ADDR - param.start) {
    printf("Size exceeds valid range\n");
    return -1;
}
```

---

### 风险二：文件路径未验证路径遍历

| 属性 | 值 |
|------|-----|
| 风险编号 | SEC-02 |
| 风险类型 | 路径遍历 |
| 严重程度 | 中 |
| 可利用性 | 低 |

**代码证据**：

```c
// lite/hidumper.c:178-181
if (strncpy_s(param.filePath, sizeof(param.filePath),
    argv[TWO_OF_ARGC_PARAMETERS], sizeof(param.filePath) - 1) != EOK) {
    printf("param.filePath is not enough or strncpy_s failed\n");
    return -1;
}
// 未验证路径合法性
```

**可利用路径**：

1. 攻击者构造恶意路径：`hidumper -m 0x20000000 0x100 /../../../etc/passwd`
2. 文件写入到非预期位置
3. 可能覆盖系统文件或写入敏感位置

**影响**：

| 影响类型 | 描述 |
|----------|------|
| 任意文件写入 | 覆盖系统配置文件 |
| 权限提升 | 修改可执行文件或配置 |

**修复建议**：

```c
#include <stdlib.h>
#include <string.h>

// 路径验证函数
static int IsPathSafe(const char *path)
{
    // 检查路径遍历攻击
    if (strstr(path, "..") != NULL) {
        return 0;
    }
    
    // 检查绝对路径
    if (path[0] == '/') {
        // 允许的目录列表
        const char *allowed_dirs[] = {"/data/", "/tmp/", NULL};
        for (int i = 0; allowed_dirs[i] != NULL; i++) {
            if (strncmp(path, allowed_dirs[i], strlen(allowed_dirs[i])) == 0) {
                return 1;
            }
        }
        return 0;
    }
    
    return 1;
}
```

---

### 风险三：调试功能在生产环境泄露

| 属性 | 值 |
|------|-----|
| 风险编号 | SEC-03 |
| 风险类型 | 信息泄露 |
| 严重程度 | 低 |
| 可利用性 | 中 |

**代码证据**：

```c
// lite/hidumper.c:120-130
static void DumpMemData(int fd, struct MemDumpParam *param)
{
#ifdef OHOS_DEBUG
    int ret = ioctl(fd, HIDUMPER_MEM_DATA, param);
    // ... 调试功能
#else
    (void)fd;
    (void)param;
    printf("Unsupported!\n");
#endif
}
```

**问题分析**：

1. 调试功能依赖编译时宏 `OHOS_DEBUG`
2. 如果调试版本流入生产环境，可能泄露敏感内存数据
3. 崩溃注入功能可能被恶意利用

**影响**：

| 影响类型 | 描述 |
|----------|------|
| 内存数据泄露 | 调试版本可读取任意内存 |
| 系统崩溃 | 崩溃注入功能可导致拒绝服务 |

**修复建议**：

1. 确保调试版本不流入生产环境
2. 在发布版本中完全移除调试代码
3. 使用运行时权限控制调试功能访问

---

### 风险四：适配器注册后无注销机制

| 属性 | 值 |
|------|-----|
| 风险编号 | SEC-04 |
| 风险类型 | 资源管理 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**代码证据**：

```c
// mini/hidumper_core.c:35-36
static int g_isAdapterRegistered = 0;
static struct HiDumperAdapter g_hidumperAdapter;
```

**问题分析**：

1. 适配器注册后无法注销
2. 全局变量生命周期与应用相同
3. 无运行时适配器切换机制

**影响**：

| 影响类型 | 描述 |
|----------|------|
| 资源泄露 | 适配器无法动态卸载 |
| 功能僵化 | 无法实现热切换 |

**修复建议**（可选，取决于设计需求）：

```c
int HiDumperUnregisterAdapter(void)
{
    if (g_isAdapterRegistered == 0) {
        return -1;  // 未注册
    }
    
    memset(&g_hidumperAdapter, 0, sizeof(g_hidumperAdapter));
    g_isAdapterRegistered = 0;
    return 0;
}
```

---

### 风险五：AT 命令参数解析未检查数组越界

| 属性 | 值 |
|------|-----|
| 风险编号 | SEC-05 |
| 风险类型 | 缓冲区安全 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**代码证据**：

```c
// lite/hidumper.c:159-203
if (argc == ONE_OF_ARGC_PARAMETERS) {
    DumpALLInfo(fd);
} else if (argc == TWO_OF_ARGC_PARAMETERS && strcmp(argv[ONE_OF_ARGC_PARAMETERS], "-dc") == 0) {
    // ...
} else if (argc == THREE_OF_ARGC_PARAMETERS && strcmp(argv[ONE_OF_ARGC_PARAMETERS], "-m") == 0) {
    // ...
}
```

**问题分析**：

1. 代码通过 argc 判断参数数量
2. 但在 `ParameterMatching` 函数内部直接访问 `argv[index]`
3. 如果调用时 argc 与实际不匹配，可能越界访问

**影响**：

| 影响类型 | 描述 |
|----------|------|
| 未定义行为 | 数组越界访问 |
| 程序崩溃 | 访问无效内存 |

**修复建议**：

代码当前实现已通过 argc 检查避免越界访问，建议保持此模式。如需更严格保护，可添加运行时断言：

```c
static void SafeExecAction(int fd, unsigned int cmd)
{
    assert(fd >= 0 && "Invalid file descriptor");
    // ... 其余代码
}
```

---

## 安全最佳实践

### 已采用的安全措施

| 措施 | 说明 | 证据位置 |
|------|------|----------|
| 安全字符串函数 | 使用 `strncpy_s` 代替 `strcpy` | `lite/hidumper.c:178-181` |
| 安全内存复制 | 使用 `memcpy_s` 代替 `memcpy` | `mini/hidumper_core.c:96-97` |
| 参数校验 | 检查空指针和函数指针有效性 | `mini/hidumper_core.c:81-95` |
| 调试功能隔离 | 使用编译时宏隔离调试代码 | `lite/hidumper.c:120, 134, 147` |
| 设备节点只读打开 | 以 `O_RDONLY` 模式打开设备 | `lite/hidumper.c:208` |

### 建议采用的安全措施

| 措施 | 优先级 | 说明 |
|------|--------|------|
| 地址范围验证 | 高 | 添加内存地址合法性检查 |
| 路径遍历防护 | 中 | 验证文件路径安全性 |
| 运行时权限控制 | 中 | 控制调试功能访问权限 |
| 适配器热卸载 | 低 | 支持适配器动态注销 |

---

## 安全相关配置

### 编译时安全配置

| 配置项 | 安全版本 | 调试版本 | 说明 |
|--------|----------|----------|------|
| OHOS_DEBUG 宏 | 未定义 | 定义 | 控制调试功能 |
| -Wall 警告 | 启用 | 启用 | 发现潜在问题 |
| 链接安全库 | 是 | 是 | 使用 bounds_checking_function |

### 运行时安全建议

| 建议 | 说明 |
|------|------|
| 限制命令执行权限 | 仅授权用户可执行 hidumper |
| 监控异常调用 | 记录可疑的参数模式 |
| 定期安全审计 | 检查日志中的异常输入 |

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 |
| [03_API.md](03_API.md) | 接口文档 |
| [04_Build.md](04_Build.md) | 构建配置 |
| [06_Troubleshooting.md](06_Troubleshooting.md) | 问题排查指南 |

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本，完成 5 项风险识别 |

# 内部 API

## 目的

本文档描述 HiView Lite 的内部 API，包括模块接口、依赖方向、稳定性和可替换点。

## 适用范围

本文档适用于：
- 需要理解模块间接口的开发者
- 需要扩展功能的维护者
- 需要重构代码的架构师

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 了解模块职责
- [架构说明](03_Architecture.md) - 了解组件关系
- [对外 API](04_External_API.md) - 了解导出接口

---

## 模块接口

### hiview_service 内部接口

#### 静态函数（不对外导出）

| 函数 | 行数 | 说明 | 稳定性 |
|------|------|------|--------|
| `GetName` | hiview_service.c:53-57 | 获取服务名称 | 稳定 |
| `Initialize` | hiview_service.c:59-73 | 服务初始化 | 稳定 |
| `MessageHandle` | hiview_service.c:75-86 | 消息处理 | 稳定 |
| `GetTaskConfig` | hiview_service.c:88-93 | 获取任务配置 | 稳定 |
| `Output` | hiview_service.c:95-105 | 输出接口 | 稳定 |
| `InitHiviewComponent` | hiview_service.c:107-115 | 初始化组件 | 稳定 |

#### 内部数据结构

| 结构体 | 行数 | 说明 | 稳定性 |
|--------|------|------|--------|
| `g_hiviewService` | hiview_service.c:31-39 | 全局服务实例 | 稳定 |
| `g_hiviewInitFuncList[]` | hiview_service.c:41 | 初始化函数数组 | 可扩展 |
| `g_hiviewMsgHandleList[]` | hiview_service.c:42 | 消息处理函数数组 | 可扩展 |

---

### hiview_config 内部接口

#### 全局变量

| 变量 | 行数 | 说明 | 稳定性 |
|------|------|------|--------|
| `g_hiviewConfig` | hiview_config.c:19-27 | 全局配置 | 可扩展 |

#### 配置结构字段

| 字段 | 偏移 | 类型 | 说明 | 可修改性 |
|------|-------|------|------|----------|
| outputOption | 0 | uint8:4 | 输出模式 | 编译时配置 |
| hiviewInited | 0:4 | uint8:1 | 初始化标志 | 只读（运行时） |
| level | 0:5 | uint8:3 | 日志级别 | 可运行时修改 |
| logSwitch | 1 | uint8:1 | 日志开关 | 可运行时修改 |
| eventSwitch | 1:1 | uint8:1 | 事件开关 | 可运行时修改 |
| dumpSwitch | 1:2 | uint8:1 | Dump 开关 | 可运行时修改 |
| logOutputModule | 2 | uint64 | 输出模块 | 编译时配置 |
| writeFailureCount | 10 | uint16 | 写入失败计数 | 只读（统计用） |

---

### hiview_cache 内部接口

#### 静态函数（不对外导出）

| 函数 | 行数 | 说明 | 稳定性 |
|------|------|------|--------|
| `GetReadCursor` | hiview_cache.c:207-222 | 获取读游标 | 内部使用 |

#### 缓存结构字段

| 字段 | 类型 | 说明 | 可修改性 |
|------|------|------|----------|
| wCursor | uint16 | 写游标 | 内部修改 |
| usedSize | uint16 | 已用大小 | 内部修改 |
| size | uint16 | 缓存大小 | 初始化后只读 |
| type | HiviewCacheType | 缓存类型 | 初始化后只读 |
| buffer | uint8 * | 循环缓冲区指针 | 初始化后只读 |

---

### hiview_file 内部接口

#### 静态函数（不对外导出）

| 函数 | 行数 | 说明 | 稳定性 |
|------|------|------|--------|
| `GetDefineFileVersion` | hiview_file.c:25-38 | 获取文件版本 | 稳定 |
| `IsValidPath` | hiview_file.c:276-289 | 验证路径有效性 | 稳定 |

#### 文件结构字段

| 字段 | 类型 | 说明 | 可修改性 |
|------|------|------|----------|
| header | HiviewFileHeader | 文件头 | 内部修改 |
| path | const char * | 文件路径 | 初始化后只读 |
| outPath | char * | 输出路径 | 内部修改 |
| pFunc | FileProc | 文件监视器 | 内部修改 |
| mutex | HiviewMutexId_t | 互斥锁 | 内部修改 |
| fhandle | int32 | 文件句柄 | 内部修改 |
| configSize | uint32 | 配置大小 | 初始化后只读 |

---

### hiview_util 内部接口

#### 静态函数（不对外导出）

| 函数 | 行数 | 说明 | 稳定性 |
|------|------|------|--------|
| `HIVIEW_GetCurrentTimeDef` | hiview_util.c:67-75 | 默认获取时间函数 | 可替换（Hook） |
| `HIVIEW_UartPrintDef` | hiview_util.c:133-136 | 默认 UART 打印函数 | 可替换（Hook） |

#### 静态函数指针（默认实现）

| 函数指针 | 行数 | 说明 | 可替换性 |
|----------|------|------|----------|
| `hiview_open` | hiview_util.c:44 | 默认 open | ✅ 可 Hook |
| `hiview_close` | hiview_util.c:45 | 默认 close | ✅ 可 Hook |
| `hiview_read` | hiview_util.c:46 | 默认 read | ✅ 可 Hook |
| `hiview_write` | hiview_util.c:47 | 默认 write | ✅ 可 Hook |
| `hiview_lseek` | hiview_util.c:48 | 默认 lseek | ✅ 可 Hook |
| `hiview_fsync` | hiview_util.c:49 | 默认 fsync | ✅ 可 Hook |
| `hiview_unlink` | hiview_util.c:50 | 默认 unlink | ✅ 可 Hook |
| `hiview_rename` | hiview_util.c:51 | 默认 rename | ✅ 可 Hook |
| `hiview_get_time` | hiview_util.c:52 | 默认获取时间 | ✅ 可 Hook |
| `hiview_uart_print` | hiview_util.c:53 | 默认 UART 打印 | ✅ 可 Hook |

---

## 依赖方向

### 模块依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                  hiview_def (基础定义)             │
└─────────────────────┬───────────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │                           │
        ▼                           ▼
┌──────────────────┐      ┌──────────────────┐
│  hiview_config  │      │  hiview_util   │
│                │      └────────┬────────┘
└────────────────┘               │
                              │
         ┌─────────────────────┼──────────────────┐
         │                     │                  │
         ▼                     ▼                  ▼
┌──────────────────┐   ┌──────────────────┐  ┌──────────────────┐
│  hiview_cache  │   │  hiview_file   │  │hiview_service  │
└──────────────────┘   └────────┬─────────┘  └────────┬────────┘
                              │                     │
                              └──────────┬──────────┘
                                         │
                                         ▼
                                ┌──────────────────┐
                                │  外部模块        │
                                │(hilog/hievent)  │
                                └─────────────────┘
```

### 依赖矩阵

| 模块 | 依赖模块 | 依赖类型 | 说明 |
|------|----------|----------|------|
| hiview_def | 无 | - | 基础定义模块 |
| hiview_config | hiview_def | 公共定义 | 使用 ohos_types.h |
| hiview_util | hiview_def | 公共定义 | 使用 ohos_types.h |
| hiview_cache | hiview_util, hiview_def | 工具函数 | 使用内存管理、中断锁 |
| hiview_file | hiview_util, hiview_config, hiview_def | 工具函数、配置 | 使用文件系统、文件路径 |
| hiview_service | 所有模块 | - | 顶层服务，协调各模块 |

### 无循环依赖

✅ **HiView Lite 没有循环依赖**，依赖方向清晰：
- `hiview_def` → 无依赖
- `hiview_config` → `hiview_def`
- `hiview_util` → `hiview_def`
- `hiview_cache` → `hiview_util`, `hiview_def`
- `hiview_file` → `hiview_util`, `hiview_config`, `hiview_def`
- `hiview_service` → 所有模块

---

## 接口稳定性

### 稳定接口

这些接口是稳定的，不建议修改：

| 模块 | 接口 | 理由 |
|------|------|------|
| hiview_service | `GetName`, `Initialize`, `MessageHandle`, `GetTaskConfig` | SAMGR Lite 要求的标准接口 |
| hiview_service | `Output` | 对外提供的输出接口 |
| hiview_config | `HiviewConfigInit` | 初始化接口，CORE_INIT 调用 |
| hiview_cache | `InitHiviewStaticCache`, `InitHiviewCache` | 缓存初始化接口 |
| hiview_cache | `WriteToCache`, `ReadFromCache` | 缓存读写接口 |
| hiview_file | `InitHiviewFile`, `WriteToFile`, `ReadFromFile` | 文件操作接口 |
| hiview_file | `WriteFileHeader`, `ReadFileHeader` | 文件头管理接口 |

### 可扩展接口

这些接口是设计为可扩展的：

| 模块 | 接口 | 扩展方式 |
|------|------|----------|
| hiview_service | `HiviewRegisterInitFunc` | 注册新的初始化函数 |
| hiview_service | `HiviewRegisterMsgHandle` | 注册新的消息处理函数 |
| hiview_util | `HIVIEW_InitHook` | 覆盖默认的文件系统/时间函数 |
| hiview_file | `RegisterFileWatcher` | 注册文件监视器回调 |

### 不稳定接口

这些接口是内部使用的，可能随时修改：

| 模块 | 接口 | 说明 |
|------|------|------|
| hiview_cache | `GetReadCursor` | 内部辅助函数 |
| hiview_file | `GetDefineFileVersion` | 内部辅助函数 |
| hiview_file | `IsValidPath` | 内部辅助函数 |
| hiview_util | 静态函数指针（hiview_*） | 默认实现，通过 Hook 替换 |

---

## 可替换点

### 1. Hook 机制（hiview_util）

**位置**：`hiview_util.h:52-74`

**说明**：通过 `HIVIEW_InitHook` 可以替换所有的文件系统函数和系统函数。

**可替换的函数**：
- 文件系统：`open`, `close`, `read`, `write`, `lseek`, `fsync`, `unlink`, `rename`
- 时间函数：`hiview_get_time`
- 打印函数：`hiview_uart_print`

**使用场景**：
- 单元测试：替换文件系统函数为模拟实现
- 平台适配：替换为平台特定的实现
- 调试：记录所有文件操作

**证据来源**：
- hiview_util.h:52-74 - `HIVIEW_Hooks` 结构体
- hiview_util.c:148-174 - `HIVIEW_InitHook` 实现

### 2. 组件初始化函数注册（hiview_service）

**位置**：`hiview_service.h:59-60`

**说明**：外部模块可以注册初始化函数，在 `InitHiviewComponent` 中被调用。

**使用场景**：
- 新增 DFX 组件
- 自定义初始化逻辑

**证据来源**：
- hiview_service.h:59-60 - `HiviewInitFunc` 类型定义
- hiview_service.c:117-120 - `HiviewRegisterInitFunc` 实现

### 3. 消息处理函数注册（hiview_service）

**位置**：`hiview_service.h:60`

**说明**：外部模块可以注册消息处理函数，在 `MessageHandle` 中被调用。

**使用场景**：
- 自定义消息处理逻辑
- 响应新的消息类型

**证据来源**：
- hiview_service.h:60 - `HiviewMsgHandle` 类型定义
- hiview_service.c:122-125 - `HiviewRegisterMsgHandle` 实现

### 4. 文件监视器注册（hiview_file）

**位置**：`hiview_file.h:48`

**说明**：可以注册文件监视器回调，在文件满时被调用。

**使用场景**：
- 文件满时自动上传
- 文件满时自动清理
- 文件满时通知用户

**证据来源**：
- hiview_file.h:48 - `FileProc` 类型定义
- hiview_file.c:291-311 - `RegisterFileWatcher` 实现

---

## 接口调用约定

### 返回值约定

| 返回类型 | 成功值 | 失败值 |
|----------|---------|---------|
| boolean | TRUE | FALSE |
| int32 | >= 0 | < 0 |
| uint32 | 实际值 | 0 |
| void | - | - |

### 参数校验

所有函数都对 NULL 指针进行校验，示例：

```c
if (cache == NULL || data == NULL) {
    return -1;
}
```

**证据来源**：
- hiview_cache.c:60-62 - `WriteToCache` 参数校验
- hiview_file.c:42-44 - `InitHiviewFile` 参数校验
- hiview_util.c:56-58 - `HIVIEW_MemAlloc` 参数校验

---

## 关键结论

1. **依赖清晰** - 没有循环依赖，依赖方向清晰。
2. **接口分层** - 对外接口、内部接口、可替换接口明确区分。
3. **扩展性强** - 通过 Hook 机制和注册机制提供良好的扩展性。
4. **稳定接口** - 核心接口是稳定的，不建议随意修改。
5. **可替换点明确** - Hook 机制和注册机制提供了明确的可替换点。

---

*最后更新：2026-02-06*

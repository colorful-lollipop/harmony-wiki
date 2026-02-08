# 目录结构与模块职责

## 目的

本文档描述 HiView Lite 项目的目录结构和各模块的职责。

## 适用范围

本文档适用于：
- 需要了解代码组织的开发者
- 需要定位特定功能的维护者
- 需要理解模块边界的集成方

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目定位
- [架构说明](03_Architecture.md) - 了解组件关系
- [内部 API](05_Internal_API.md) - 了解模块接口

---

## 顶层目录结构

```
hiview_lite/
├── BUILD.gn                    # GN 构建文件
├── bundle.json                 # 组件元数据
├── LICENSE                    # Apache 2.0 许可证
├── README.md                  # 英文说明
├── README_zh.md               # 中文说明
├── hiview_cache.c              # 缓存实现
├── hiview_cache.h              # 缓存接口
├── hiview_config.c             # 配置实现
├── hiview_config.h             # 配置接口
├── hiview_def.h               # 公共定义
├── hiview_file.c               # 文件操作实现
├── hiview_file.h               # 文件操作接口
├── hiview_service.c            # 服务实现
├── hiview_service.h            # 服务接口
├── hiview_util.c               # 工具函数实现
└── hiview_util.h               # 工具函数接口
```

**说明**：所有源代码文件位于根目录，没有子目录结构。这是因为 HiView Lite 是一个轻量级模块，功能相对集中。

---

## 模块职责

### 1. 服务层（hiview_service）

**职责**：
- 向 SAMGR Lite 注册 HiView 服务
- 管理组件初始化函数列表
- 管理消息处理函数列表
- 处理内部消息

**主要功能**：

| 功能 | 函数 | 行数 | 说明 |
|------|--------|------|------|
| 服务初始化 | `Init` | hiview_service.c:45-51 | 注册服务到 SAMGR，初始化组件 |
| 服务启动 | `Initialize` | hiview_service.c:59-73 | 设置服务标识，标记已初始化 |
| 消息处理 | `MessageHandle` | hiview_service.c:75-86 | 根据 msgId 调用处理函数 |
| 获取任务配置 | `GetTaskConfig` | hiview_service.c:88-93 | 返回任务配置（栈大小、优先级） |
| 输出接口 | `Output` | hiview_service.c:95-105 | 接收消息并发送到服务 |
| 注册初始化函数 | `HiviewRegisterInitFunc` | hiview_service.c:117-120 | 注册组件初始化函数 |
| 注册消息处理函数 | `HiviewRegisterMsgHandle` | hiview_service.c:122-125 | 注册消息处理函数 |
| 发送消息 | `HiviewSendMessage` | hiview_service.c:127-139 | 发送内部消息 |
| 初始化组件 | `InitHiviewComponent` | hiview_service.c:107-115 | 调用所有注册的初始化函数 |

**关键数据结构**：

- `HiviewService` (hiview_service.h:54-57) - 服务结构体
- `g_hiviewInitFuncList[]` (hiview_service.c:41) - 组件初始化函数数组
- `g_hiviewMsgHandleList[]` (hiview_service.c:42) - 消息处理函数数组

**依赖**：
- SAMGR Lite（服务注册和消息传递）
- ohos_init.h（初始化宏）
- hiview_config.h（配置）
- hiview_util.h（工具函数）

---

### 2. 配置管理（hiview_config）

**职责**：
- 管理全局配置
- 定义文件路径宏
- 定义缓存大小
- 定义 RAM Dump 配置

**主要功能**：

| 功能 | 函数 | 行数 | 说明 |
|------|--------|------|------|
| 配置初始化 | `HiviewConfigInit` | hiview_config.c:29-34 | 初始化全局配置 |

**关键数据结构**：

- `g_hiviewConfig` (hiview_config.c:19-27) - 全局配置变量
- `HiviewConfig` (hiview_config.h:76-85) - 配置结构体

**配置项**：

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| outputOption | uint8:4 | OUTPUT_OPTION | 日志输出模式 |
| hiviewInited | uint8:1 | FALSE | 服务是否已初始化 |
| level | uint8:3 | OUTPUT_LEVEL | 日志输出级别 |
| logSwitch | uint8:1 | HILOG_LITE_SWITCH | 日志开关 |
| eventSwitch | uint8:1 | HIEVENT_LITE_SWITCH | 事件开关 |
| dumpSwitch | uint8:1 | DUMP_LITE_SWITCH | Dump 开关 |
| logOutputModule | uint64 | LOG_OUTPUT_MODULE | 日志输出模块 |
| writeFailureCount | uint16 | 0 | 写入失败计数 |

**依赖**：
- ohos_init.h（初始化宏）
- 无其他模块依赖

---

### 3. 缓存机制（hiview_cache）

**职责**：
- 提供循环缓冲区实现
- 支持静态内存和动态内存
- 提供线程安全的读写操作

**主要功能**：

| 功能 | 函数 | 行数 | 说明 |
|------|--------|------|------|
| 静态内存初始化 | `InitHiviewStaticCache` | hiview_cache.c:23-36 | 使用外部静态内存初始化 |
| 动态内存初始化 | `InitHiviewCache` | hiview_cache.c:38-56 | 使用动态内存初始化 |
| 写入缓存 | `WriteToCache` | hiview_cache.c:58-107 | 写入数据（带中断锁保护） |
| 读取缓存 | `ReadFromCache` | hiview_cache.c:109-147 | 读取数据 |
| 预读缓存 | `PrereadFromCache` | hiview_cache.c:149-182 | 预读数据（不改变读游标） |
| 丢弃数据 | `DiscardCacheData` | hiview_cache.c:184-191 | 丢弃所有缓存数据 |
| 销毁缓存 | `DestroyCache` | hiview_cache.c:193-205 | 销毁缓存并释放内存 |

**关键数据结构**：

- `HiviewCache` (hiview_cache.h:38-46) - 缓存结构体

**缓存类型**：

| 类型 | 值 | 说明 |
|------|-----|------|
| CORE_CACHE | 0 | 核心缓存 |
| LOG_CACHE | 1 | 日志缓存 |
| JS_LOG_CACHE | 2 | JS 日志缓存 |
| DUMP_CACHE | 3 | Dump 缓存 |
| FAULT_EVENT_CACHE | 4 | 故障事件缓存 |
| UE_EVENT_CACHE | 5 | UE 事件缓存 |
| STAT_EVENT_CACHE | 6 | 统计事件缓存 |

**依赖**：
- hiview_util.h（内存管理、中断锁）
- securec.h（安全字符串操作）
- ohos_types.h（基础类型）

---

### 4. 文件操作（hiview_file）

**职责**：
- 提供循环文件实现
- 管理文件头（版本、时间、游标）
- 提供文件处理（复制/重命名）
- 提供文件监视器

**主要功能**：

| 功能 | 函数 | 行数 | 说明 |
|------|--------|------|------|
| 初始化文件 | `InitHiviewFile` | hiview_file.c:40-96 | 打开文件，读取/创建文件头 |
| 写入文件头 | `WriteFileHeader` | hiview_file.c:98-119 | 写入文件头到文件 |
| 读取文件头 | `ReadFileHeader` | hiview_file.c:121-149 | 读取并验证文件头 |
| 写入文件 | `WriteToFile` | hiview_file.c:151-173 | 写入数据（检测文件满） |
| 读取文件 | `ReadFromFile` | hiview_file.c:175-197 | 读取数据（循环读取） |
| 获取已用大小 | `GetFileUsedSize` | hiview_file.c:199-205 | 获取文件已用大小 |
| 获取空闲大小 | `GetFileFreeSize` | hiview_file.c:207-214 | 获取文件空闲大小 |
| 关闭文件 | `CloseHiviewFile` | hiview_file.c:216-225 | 关闭文件并注销监视器 |
| 处理文件 | `ProcFile` | hiview_file.c:227-274 | 文件复制或重命名 |
| 注册监视器 | `RegisterFileWatcher` | hiview_file.c:291-311 | 注册文件满回调 |
| 注销监视器 | `UnRegisterFileWatcher` | hiview_file.c:313-340 | 注销文件满回调 |
| 验证路径 | `IsValidPath` | hiview_file.c:276-289 | 验证路径是否为预定义路径 |

**关键数据结构**：

- `HiviewFile` (hiview_file.h:86-95) - 文件结构体
- `HiviewFileHeader` (hiview_file.h:78-84) - 文件头
- `FileHeaderCommon` (hiview_file.h:70-76) - 公共文件头

**文件类型**：

| 类型 | 值 | 说明 |
|------|-----|------|
| HIVIEW_LOG_TEXT_FILE | 0 | 日志文本文件 |
| HIVIEW_LOG_BIN_FILE | 1 | 日志二进制文件 |
| HIVIEW_DUMP_FILE | 2 | Dump 文件 |
| HIVIEW_FAULT_EVENT_FILE | 3 | 故障事件文件 |
| HIVIEW_UE_EVENT_FILE | 4 | UE 事件文件 |
| HIVIEW_STAT_EVENT_FILE | 5 | 统计事件文件 |

**依赖**：
- hiview_util.h（文件系统封装、互斥锁）
- hiview_config.h（文件路径宏）
- hiview_def.h（文件头定义）
- hiview_log.h（日志，错误报告）
- hiview_event.h（事件，错误报告）

---

### 5. 工具函数（hiview_util）

**职责**：
- 封装内存管理
- 封装互斥锁
- 封装中断锁
- 封装文件系统
- 提供 Hook 机制
- 提供工具函数（时间、字节序转换）

**主要功能**：

| 功能 | 函数 | 行数 | 说明 |
|------|--------|------|------|
| 内存分配 | `HIVIEW_MemAlloc` | hiview_util.c:55-59 | 分配内存 |
| 内存释放 | `HIVIEW_MemFree` | hiview_util.c:61-65 | 释放内存 |
| 初始化互斥锁 | `HIVIEW_MutexInit` | hiview_util.c:89-92 | 创建互斥锁 |
| 加锁 | `HIVIEW_MutexLock` | hiview_util.c:94-100 | 加锁（等待） |
| 加锁（超时） | `HIVIEW_MutexLockOrWait` | hiview_util.c:102-108 | 加锁（带超时） |
| 解锁 | `HIVIEW_MutexUnlock` | hiview_util.c:110-116 | 解锁 |
| 关中断 | `HIVIEW_IntLock` | hiview_util.c:118-121 | 关中断 |
| 恢复中断 | `HIVIEW_IntRestore` | hiview_util.c:123-126 | 恢复中断 |
| 获取时间 | `HIVIEW_GetCurrentTime` | hiview_util.c:77-80 | 获取当前时间（毫秒） |
| 获取 RTC 时间 | `HIVIEW_RtcGetCurrentTime` | hiview_util.c:82-87 | 获取 RTC 时间 |
| 获取任务 ID | `HIVIEW_GetTaskId` | hiview_util.c:128-131 | 获取任务 ID |
| UART 打印 | `HIVIEW_UartPrint` | hiview_util.c:138-141 | UART 打印 |
| 延时 | `HIVIEW_Sleep` | hiview_util.c:143-146 | 延时 |
| 打开文件 | `HIVIEW_FileOpen` | hiview_util.c:176-183 | 打开文件 |
| 关闭文件 | `HIVIEW_FileClose` | hiview_util.c:185-191 | 关闭文件 |
| 读取文件 | `HIVIEW_FileRead` | hiview_util.c:193-199 | 读取文件 |
| 写入文件 | `HIVIEW_FileWrite` | hiview_util.c:201-207 | 写入文件 |
| 文件定位 | `HIVIEW_FileSeek` | hiview_util.c:209-215 | 文件定位 |
| 获取文件大小 | `HIVIEW_FileSize` | hiview_util.c:217-223 | 获取文件大小 |
| 同步文件 | `HIVIEW_FileSync` | hiview_util.c:225-231 | 同步文件 |
| 删除文件 | `HIVIEW_FileUnlink` | hiview_util.c:233-236 | 删除文件 |
| 复制文件 | `HIVIEW_FileCopy` | hiview_util.c:238-282 | 复制文件 |
| 移动文件 | `HIVIEW_FileMove` | hiview_util.c:284-294 | 移动文件 |
| 初始化 Hook | `HIVIEW_InitHook` | hiview_util.c:148-174 | 初始化 Hook 机制 |
| 32 位字节序转换 | `Change32Endian` | hiview_util.c:296-304 | 32 位字节序转换 |
| 16 位字节序转换 | `Change16Endian` | hiview_util.c:306-312 | 16 位字节序转换 |

**Hook 机制**：

| Hook 函数 | 默认值 | 说明 |
|----------|---------|------|
| open_fn | open | 文件打开函数 |
| close_fn | close | 文件关闭函数 |
| read_fn | read | 文件读取函数 |
| write_fn | write | 文件写入函数 |
| lseek_fn | lseek | 文件定位函数 |
| fsync_fn | fsync | 文件同步函数 |
| unlink_fn | unlink | 文件删除函数 |
| rename_fn | NULL | 文件重命名函数 |
| hiview_get_time_fn | HIVIEW_GetCurrentTimeDef | 获取时间函数 |
| hiview_uart_print_fn | HIVIEW_UartPrintDef | UART 打印函数 |

**依赖**：
- CMSIS-OS API（互斥锁、任务管理）
- LiteOS-M 中断 API（中断锁）
- POSIX API（文件系统、时间）

---

### 6. 公共定义（hiview_def）

**职责**：
- 定义公共宏
- 定义公共数据结构

**主要内容**：

| 定义 | 值 | 说明 |
|------|-----|------|
| HIVIEW_SERVICE | "hiview" | 服务名称 |
| LOG_INFO_HEAD | 0xEC | 日志信息头 |
| EVENT_INFO_HEAD | 0xEA | 事件信息头 |
| LOG_MODULE_NAME_LEN | 16 | 日志模块名长度 |
| LOG_CONTENT_MAX_LEN | 96 | 日志内容最大长度 |

**关键数据结构**：

- `HiLogModuleInfo` (hiview_def.h:42-46) - HiLog 模块信息
- `HiEventTag` (hiview_def.h:48-52) - HiEvent 标签

**依赖**：
- ohos_types.h（基础类型）

---

## 模块依赖关系

```
                    ┌─────────────┐
                    │ hiview_def │
                    │  (公共定义) │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ hiview_config│   │ hiview_util  │   │hiview_cache │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
                          ▼
                   ┌──────────────┐
                   │hiview_file  │
                   └──────┬───────┘
                          │
                          ▼
                   ┌──────────────┐
                   │hiview_service│
                   └─────────────┘
```

**说明**：
- `hiview_def` - 被所有模块依赖（基础定义）
- `hiview_util` - 被 `hiview_cache`、`hiview_file`、`hiview_service` 依赖
- `hiview_cache` - 独立模块，被其他模块使用
- `hiview_file` - 依赖 `hiview_util`、`hiview_config`、`hiview_def`
- `hiview_service` - 依赖所有模块，作为顶层服务

---

## 关键结论

1. **扁平化结构** - 所有源代码文件位于根目录，没有子目录结构。
2. **模块职责清晰** - 每个模块有明确的职责，耦合度低。
3. **依赖关系简单** - 没有循环依赖，依赖方向清晰。
4. **轻量级设计** - 每个模块功能精简，资源占用小。

---

*最后更新：2026-02-06*

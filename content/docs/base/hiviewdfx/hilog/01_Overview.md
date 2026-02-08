# HiLog 项目概览

> 生成时间: 2026-02-06
> 相关证据: `bundle.json`, `README_zh.md`

---

## 目的

本文档介绍 OpenHarmony HiLog 模块的整体定位、核心能力、运行环境，帮助新人快速理解项目全貌。

## 适用范围

适用于 OpenHarmony HiLog 模块（版本 3.1），包括：
- hilogd 日志常驻服务
- hilog 命令行工具
- libhilog 客户端库
- 多语言接口（JS、NDK、Rust、ETS）

---

## 项目定位

HiLog 是 OpenHarmony 日志系统，提供给系统框架、服务、应用打印日志，记录用户操作、系统运行状态等。

**子系统**: hiviewdfx
**系统类型**: standard
**许可证**: Apache License 2.0
**系统能力**: SystemCapability.HiviewDFX.HiLog

---

## 核心能力

### 1. 日志打印

**多级别支持**：
- `DEBUG` (3): 详细流程记录，用于调试分析
- `INFO` (4): 记录业务关键流程节点
- `WARN` (5): 较为严重的非预期情况
- `ERROR` (6): 程序或功能发生了错误
- `FATAL` (7): 重大致命异常，程序或功能即将崩溃

**多类型支持**：
- `LOG_APP` (0): 应用日志
- `LOG_CORE` (3): 核心服务、框架日志
- `LOG_INIT` (1): 启动阶段重要日志
- `LOG_KMSG` (4): 内核消息

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:42-58`

### 2. 隐私保护

通过格式化控制符实现敏感数据遮蔽：
- `%{public}s`: 始终显示明文
- `%{private}s`: 发布版本默认显示 `<private>`，debug 应用可关闭

**证据**:
- N-API 实现: `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:41-115`
- 隐私模式检查: `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:52-54`

### 3. 流量控制

防止日志打印流量过大导致性能恶化：
- **进程级别超限机制**: 每个进程每秒日志量不超过 50K，超过的日志不打印
- **Domain 级别超限机制**: 对超标 domain 进行流控
- 命令控制: `hilog -Q pidon/pidoff/domainon/domainoff`

**证据**: `README_zh.md:115-123`

### 4. 日志落盘

支持将缓冲区日志落盘到 `/data/log/hilog/`：
- **压缩算法**: zlib、zstd
- **文件轮转**: 支持最大 1000 个文件，索引 [0, 999] 回绕
- **文件大小范围**: 64K - 512M
- **命名格式**: `hilog.000.20170805-170154.gz`

**证据**:
- 落盘实现: `services/hilogd/log_persister.cpp`
- 配置常量: `frameworks/libhilog/include/hilog_common.h:38-41`

### 5. 日志查询与过滤

提供强大的过滤功能：
- **按类型过滤**: `-t app/core/init`
- **按级别过滤**: `-L D/I/W/E/F`
- **按 domain 过滤**: `-D 0x3200`
- **按 tag 过滤**: `-T MyTag`
- **按 PID 过滤**: `-P 1234`（需 root/shell 权限）
- **正则过滤**: `-e "error.*failed"`
- **排除过滤**: `-t ^core`（排除 core 类型）

**证据**: `README_zh.md:171-223`

---

## 运行环境

### 系统要求

**编译器**: Clang 8.0.0 及以上
**操作系统**: OpenHarmony、Linux、macOS、Windows（开发）

### 依赖组件

| 依赖 | 版本 | 用途 |
|------|-------|------|
| bounds_checking_function | - | 边界检查函数 |
| c_utils | - | C 工具库 |
| ffrt | - | Foundation Foundation Runtime |
| init | - | 初始化系统 |
| napi | - | Node-API 接口 |
| zlib | - | 压缩库 |
| runtime_core | - | 运行时核心（用于 ETS） |

**证据**: `bundle.json:23-32`

### 内存占用

| 类型 | 大小 |
|------|------|
| ROM | 648KB |
| RAM | 16336KB |

**证据**: `bundle.json:21-22`

---

## 关键概念

### 1. Log Domain（日志域）

用于标识子系统/模块的 16 位标识符：
- **应用 domain**: 0x0 - 0xFFFF
- **系统 domain**: 0xD000000 - 0xD0FFFFF

**证据**: `interfaces/native/innerkits/include/hilog/log_c.h:27-33`

### 2. Log Tag（日志标签）

字符串类型的模块标识符，用于日志分类和过滤：
- 通常为模块名或关键字
- 建议长度控制在合理范围

### 3. Ring Buffer（环行缓冲区）

固定大小的循环缓冲区：
- **类型分离**: APP、CORE、INIT、KMSG 独立 buffer
- **默认大小**: 256KB/类型
- **可配置**: 通过 `-G` 命令调整（64KB - 16MB）

**证据**:
- 缓冲区定义: `services/hilogd/include/log_buffer.h`
- 大小限制: `frameworks/libhilog/include/hilog_common.h:35-36`

### 4. Unix Domain Socket

进程间通信机制：
- **hilogInput**: SOCK_DGRAM，应用提交日志
- **hilogOutput**: SOCK_SEQPACKET，查询日志
- **hilogControl**: SOCK_SEQPACKET，控制命令

**证据**: `frameworks/libhilog/include/hilog_common.h:27-28`

---

## 日志格式

### 输出格式

```
日期 时间 进程号 线程号 日志级别 domainID/日志标签: 日志内容
```

### 示例

```
04-19 17:02:14.735  5394  5394 I A00032/testTag: this is a info level hilog
```

解析：
- **日期时间**: `04-19 17:02:14.735` - 月-日 时:分:秒.毫秒
- **进程号**: `5394` - PID
- **线程号**: `5394` - TID
- **日志级别**: `I` - INFO（D=DEBUG, I=INFO, W=WARN, E=ERROR, F=FATAL）
- **domainID**: `A00032` - `A` 表示应用日志，`00032` 为十六进制 domain ID
- **日志标签**: `testTag` - 自定义 tag
- **日志内容**: `this is a info level hilog` - 格式化后的日志消息

**证据**: `README_zh.md:73-78`

---

## 核心组件概览

### 1. hilogd 服务

**功能**: 用户态日志常驻服务
**责任**:
- 接收来自应用的日志
- 存储日志到环形缓冲区
- 处理日志查询请求
- 执行日志落盘任务
- 应用流量控制策略
- 读取内核日志

**关键文件**: `services/hilogd/`

### 2. hilog 工具

**功能**: 命令行日志查看工具
**责任**:
- 查询 hilogd 缓冲区日志
- 应用过滤条件
- 发送控制命令
- 显示统计信息

**关键文件**: `services/hilogtool/`

### 3. libhilog 库

**功能**: 对外 Native 日志库
**责任**:
- 提供 C/C++ 日志 API
- 通过 Unix Domain Socket 与 hilogd 通信
- 参数格式化和验证
- 错误处理

**关键文件**: `frameworks/libhilog/`

### 4. 语言接口

| 接口 | 语言 | 目标用户 |
|------|------|---------|
| hilog_napi | JavaScript | JS 应用开发者 |
| hilog_ndk | C/C++ | Native 应用开发者 |
| hilog_rust | Rust | Rust 应用开发者 |
| ani_hilog | ArkTS (ETS) | 声明式 UI/ArkUI 应用 |

---

## 特性亮点

### 1. 多语言支持

HiLog 提供多种语言的绑定：
- **JS**: 通过 N-API，12 个日志方法
- **C**: 5 个日志宏（DEBUG/INFO/WARN/ERROR/FATAL）
- **C++**: HiLog 类，标签化日志
- **Rust**: 通过 FFI 调用 Native API
- **ETS**: ANI 绑定

### 2. 高性能设计

- **Unix Domain Socket**: 比 Binder 更低延迟、更高吞吐量
- **SOCK_DGRAM**: 无连接开销，适合高频日志写入
- **环形缓冲区**: 无需动态内存分配，固定大小循环使用

### 3. 安全特性

- **UID 访问控制**: 限制非特权用户的 PID 过滤
- **隐私保护**: 默认隐藏敏感数据
- **流控保护**: 防止日志 DoS 攻击
- **SO_PASSCRED**: 内核提供的 PID/UID，防止伪造

### 4. 灵活配置

- **运行时调整**: Buffer 大小、日志级别、流控开关
- **参数控制**: 通过 param API 动态配置
- **多实例支持**: 支持多个落盘任务

---

## 相关跳转链接

- [目录结构详解](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [N-API 接口文档](04_NAPI_Interface.md)
- [安全分析](08_Security_Analysis.md)
- [常见问题](09_Troubleshooting.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 系统能力 | bundle.json:15-16 | SystemCapability.HiviewDFX.HiLog |
| 依赖列表 | bundle.json:23-32 | components.deps |
| 内存占用 | bundle.json:21-22 | rom, ram |
| 日志类型枚举 | interfaces/native/innerkits/include/hilog/log_c.h:43-58 | LogType |
| 日志级别枚举 | interfaces/native/innerkits/include/hilog/log_c.h:61-76 | LogLevel |
| Socket 名字 | frameworks/libhilog/include/hilog_common.h:27-28 | OUTPUT_SOCKET_NAME, CONTROL_SOCKET_NAME |
| 缓冲区大小限制 | frameworks/libhilog/include/hilog_common.h:35-36 | MIN_BUFFER_SIZE, MAX_BUFFER_SIZE |
| 落盘配置 | frameworks/libhilog/include/hilog_common.h:38-41 | MIN/MAX_LOG_FILE_SIZE |
| 编译器要求 | README_zh.md:59 | Clang 8.0.0 |

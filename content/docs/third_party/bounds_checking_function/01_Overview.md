# bounds_checking_function 概述

## 1. 原始库简介

### 1.1 基本信息

| 属性 | 说明 |
|------|------|
| **库名称** | libboundscheck |
| **来源项目** | openEuler |
| **版本** | v1.1.16 |
| **许可证** | Mulan Permissive Software License, Version 2 |
| **上游地址** | https://gitee.com/openeuler/libboundscheck |

### 1.2 功能描述

libboundscheck 是一个**安全 C 语言库**，遵循 **C11 Annex K (Bounds-checking interfaces)** 标准实现。它提供了一组带有边界检查的安全函数，用于替代传统的、不安全的 C 标准库函数（如 `memcpy`, `strcpy`, `sprintf` 等）。

### 1.3 核心功能

#### 内存操作类
- `memcpy_s`, `wmemcpy_s` - 安全内存复制
- `memmove_s`, `wmemmove_s` - 安全内存移动（处理重叠区域）
- `memset_s` - 安全内存初始化

#### 字符串操作类
- `strcpy_s`, `wcscpy_s` - 安全字符串复制
- `strncpy_s`, `wcsncpy_s` - 安全有限长度字符串复制
- `strcat_s`, `wcscat_s` - 安全字符串连接
- `strncat_s`, `wcsncat_s` - 安全有限长度字符串连接
- `strtok_s`, `wcstok_s` - 安全字符串分割

#### 格式化 I/O 类
- `sprintf_s`, `swprintf_s` - 安全格式化输出到字符串
- `snprintf_s`, `vsnprintf_s` - 安全有限长度格式化输出
- `vsprintf_s`, `vswprintf_s` - 安全可变参数格式化输出
- `sscanf_s`, `swscanf_s` - 安全格式化输入解析
- `scanf_s`, `wscanf_s` - 安全格式化输入
- `fscanf_s`, `fwscanf_s` - 安全文件格式化输入

#### 输入类
- `gets_s` - 安全字符串输入

### 1.4 安全特性

#### 边界检查
所有函数在执行操作前都会检查：
- **目标缓冲区大小** - 确保不会溢出
- **源数据大小** - 确保读取不会越界
- **重叠检测** - `memmove_s` 自动处理内存重叠

#### 错误处理
定义了专门的错误码：
- `EOK` (0) - 操作成功
- `EINVAL` (22) - 无效参数
- `EINVAL_AND_RESET` (150) - 无效参数（强制重置目标缓冲区）
- `ERANGE` (34) - 范围错误（目标缓冲区不足）
- `ERANGE_AND_RESET` (162) - 范围错误（强制重置目标缓冲区）
- `EOVERLAP_AND_RESET` (182) - 检测到缓冲区重叠

#### 性能优化
- **小数据优化** - 对小于 64 字节的数据使用结构体赋值优化
- **对齐优化** - 8 字节对齐的数据使用快速路径
- **内联支持** - 提供内联版本减少函数调用开销

---

## 2. OpenHarmony 中的作用与定位

### 2.1 基础设施地位

**bounds_checking_function** 是 OpenHarmony 的**核心基础安全库**，具有以下特点：

| 特性 | 说明 |
|------|------|
| **依赖范围** | 1941+ 个 BUILD.gn 文件依赖 |
| **系统覆盖** | mini / small / standard 全系统类型支持 |
| **启动阶段** | system / updater / ramdisk 全生命周期必需 |
| **组件层级** | chipsetsdk_sp / platformsdk / sasdk 三层 SDK 均暴露 |

### 2.2 在 OH 中的关键作用

#### 2.2.1 内存安全保障

OpenHarmony 作为面向 IoT 和移动设备的操作系统，对安全性有极高要求。bounds_checking_function 提供了：

1. **缓冲区溢出防护** - 替代不安全的 `memcpy`/`strcpy`，防止栈/堆溢出攻击
2. **格式化字符串防护** - 安全的 `sprintf_s` 防止格式化字符串漏洞
3. **整数溢出防护** - 内部检查防止大小参数溢出

#### 2.2.2 合规性要求

满足以下安全标准和认证要求：
- **CWE 合规** - 消除 CWE-120 (缓冲区溢出)、CWE-121 (栈溢出) 等常见问题
- **MISRA C** - 符合嵌入式 C 编码规范的安全要求
- **汽车/工控认证** - 满足 IEC 61508、ISO 26262 等标准对安全函数的要求

#### 2.2.3 系统稳定性

- **故障隔离** - 通过返回值而非异常/崩溃报告错误
- **确定性行为** - 所有函数都有明确的边界条件处理
- **零开销抽象** - 安全检查只在边界情况触发，正常路径保持高性能

### 2.3 典型使用场景

#### 场景 1：ArkCompiler 运行时
ArkCompiler 在编译和执行 JavaScript/ArkTS 代码时，需要在运行时进行大量的字符串和内存操作：
```cpp
// 处理 JS 字符串转换
char buffer[256];
strcpy_s(buffer, sizeof(buffer), jsString.c_str());
```

#### 场景 2：HDI 驱动接口
硬件驱动接口 (HDI) 层需要处理来自用户空间的输入数据：
```cpp
// 驱动参数拷贝
memcpy_s(driverBuffer, driverBufferSize, userData, userDataLen);
```

#### 场景 3：系统服务间通信
IPC (进程间通信) 中的数据序列化和反序列化：
```cpp
// Parcel 数据写入
sprintf_s(buf, sizeof(buf), "%s:%d", serviceName, handle);
```

#### 场景 4：开发工具链
HDC (设备连接器)、Profiler、HiPerf 等工具处理用户输入和设备输出：
```cpp
// 命令解析
sscanf_s(cmd, "%255s %255s", arg1, sizeof(arg1), arg2, sizeof(arg2));
```

### 2.4 与其他安全机制的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 安全体系                      │
├─────────────────────────────────────────────────────────────┤
│  应用层安全      │ ArkTS 类型系统、Ability 沙箱               │
├─────────────────────────────────────────────────────────────┤
│  运行时安全      │ ArkCompiler 字节码校验、内存管理           │
├─────────────────────────────────────────────────────────────┤
│  系统服务安全    │ IPC 权限校验、SA 访问控制                  │
├─────────────────────────────────────────────────────────────┤
│  原生代码安全    │ ┌──────────────────────────────────────┐ │
│                 │ │ bounds_checking_function             │ │
│                 │ │ • 边界检查内存/字符串操作              │ │
│                 │ │ • 格式化 I/O 安全                      │ │
│                 │ └──────────────────────────────────────┘ │
│                 │ ┌──────────────────────────────────────┐ │
│                 │ │ OpenSSL, MbedTLS                     │ │
│                 │ │ • 加密/解密安全                        │ │
│                 │ │ • 证书/密钥管理                        │ │
│                 │ └──────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  内核安全        │ Linux 内核安全模块、LiteOS 安全机制        │
├─────────────────────────────────────────────────────────────┤
│  硬件安全        │ TrustZone、安全存储                        │
└─────────────────────────────────────────────────────────────┘
```

### 2.5 架构位置

bounds_checking_function 位于 OpenHarmony 软件栈的**原生运行时层**：

```
┌────────────────────────────────────────────┐
│  应用框架层 (Application Framework)         │
│  - ArkUI, Ability Kit, App Kit             │
├────────────────────────────────────────────┤
│  系统服务层 (System Services)               │
│  - SAMgr, BMS, DMS, etc.                   │
├────────────────────────────────────────────┤
│  原生运行时层 (Native Runtime)              │
│  - ArkCompiler Runtime                     │
│  - ┌────────────────────────────────────┐ │
│  - │ bounds_checking_function           │ │
│  - │ • securec.h (安全函数声明)         │ │
│  - │ • libsec_shared.so (动态库)        │ │
│  - └────────────────────────────────────┘ │
│  - c_utils, hilog, ipc, etc.               │
├────────────────────────────────────────────┤
│  HAL / HDI 层                              │
│  - 硬件抽象接口                            │
├────────────────────────────────────────────┤
│  内核层 (Kernel)                           │
│  - Linux / LiteOS                          │
└────────────────────────────────────────────┘
```

### 2.6 维护策略

由于 bounds_checking_function 是**无 Patch** 的基础库，其维护策略为：

1. **上游同步** - 与 openEuler 的 libboundscheck 保持版本同步
2. **版本升级** - 当上游发布新版本时评估升级（当前 v1.1.16）
3. **安全响应** - 如发现 CVE，通过升级上游版本修复
4. **配置优化** - 通过 BUILD.gn 的编译选项进行 OH 特定优化（如 PAC 使能）

---

## 3. 文档导航

- [02_Patches.md](./02_Patches.md) - Patch 详细分析
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用
- [05_API_Differences.md](./05_API_Differences.md) - API 差异说明
- [06_Security.md](./06_Security.md) - 安全风险分析

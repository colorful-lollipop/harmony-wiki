# 概览

本文档介绍 OpenHarmony Preferences（首选项）模块的定位、能力边界和运行环境。

## 1 模块定位

### 1.1 基本定义

**首选项（Preferences）** 是 OpenHarmony 分布式数据管理子系统中的轻量级键值对（Key-Value）数据存储模块，为应用提供简单、快速的数据持久化能力。

**代码位置**：`//foundation/distributeddatamgr/preferences`

**模块定位图**：

```
OpenHarmony
├── distributeddatamgr（分布式数据管理）
│   ├── preferences（首选项）← 本模块
│   ├── distributeddatamgr
│   └── ...
├── ...
```

### 1.2 在系统中的位置

根据 `bundle.json:37-39`：

```json
{
  "name": "preferences",
  "subsystem": "distributeddatamgr"
}
```

**子系统**：distributeddatamgr（分布式数据管理）

**组件类型**：数据持久化组件

### 1.3 核心价值

| 特性 | 说明 |
|------|------|
| **轻量级** | 内存缓存 + 文件持久化，访问速度快 |
| **易用性** | 简洁的 API 设计，上手成本低 |
| **标准化** | 支持 JS/TS、C++、NDK 多语言接口 |
| **安全** | 基于系统权限框架的访问控制 |

---

## 2 核心能力

### 2.1 能力列表

根据 `bundle.json:40-43` 声明的 SysCap：

| 能力标识 | 说明 | 适用范围 |
|----------|------|----------|
| `SystemCapability.DistributedDataManager.Preferences.Core` | 核心能力 | 标准版设备 |
| `SystemCapability.DistributedDataManager.Preferences.Core.Lite` | 核心能力 | 轻量版设备 |

### 2.2 主要功能

根据 `README_zh.md` 和代码分析：

| 功能 | 描述 |
|------|------|
| **键值存储** | 支持 String、int、boolean、float、double、long 等类型 |
| **数据持久化** | 支持 XML 文件存储，可选 GSKV 存储 |
| **内存缓存** | 数据加载到内存，访问效率高 |
| **变化监听** | 支持注册观察者，监听数据变化 |
| **实例管理** | 支持从内存中移除未使用的实例 |

### 2.3 存储类型

根据 `preferences.h:32-35`：

```cpp
enum StorageType {
    XML = 0,   // XML 文件存储（默认）
    GSKV       // GSKV 存储（可选）
};
```

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| XML | 传统 XML 文件格式 | 通用场景 |
| GSKV | 高性能键值存储 | 大数据量场景（可选） |

---

## 3 数据约束

根据 `preferences.h:75-80`：

| 约束项 | 最大值 | 常量定义 |
|--------|--------|----------|
| Key 长度 | 1024 字符 | `MAX_KEY_LENGTH` |
| Value 长度 | 16MB | `MAX_VALUE_LENGTH` |

**建议**（来自 `README_zh.md:51`）：

> 存储的数据量应该是轻量级的，建议存储的数据不超过一万条，否则会在内存方面产生较大的开销。

### 3.1 Key 约束

| 约束 | 说明 |
|------|------|
| 类型 | String |
| 非空 | 要求非空 |
| 长度 | 不超过 1024 字符 |

### 3.2 Value 约束

| 约束 | 说明 |
|------|------|
| 类型 | String/int/bool/float/double/long |
| String 可空 | 可以为空 |
| String 长度 | 不超过 16 * 1024 * 1024 字符 |

---

## 4 运行环境

### 4.1 支持的系统类型

根据 `bundle.json:45-47`：

```json
"adapted_system_type": [
  "standard"
]
```

**支持**：标准版 OpenHarmony

### 4.2 资源消耗

根据 `bundle.json:48-49`：

| 资源 | 限额 |
|------|------|
| ROM | 512KB |
| RAM | 1024KB |

### 4.3 平台支持

根据 `BUILD.gn` 文件分析，支持以下平台：

| 平台 | 状态 | 说明 |
|------|------|------|
| OpenHarmony (OHOS) | ✅ 正式支持 | 主要目标平台 |
| Android | ✅ 支持 | 跨平台兼容 |
| iOS | ✅ 支持 | 跨平台兼容 |
| Windows | ✅ 支持 | 开发调试 |
| macOS | ✅ 支持 | 开发调试 |

---

## 5 目录结构

### 5.1 顶层目录

```
preferences/
├── .clang-format         # 代码格式配置
├── bundle.json           # 模块配置文件
├── preferences.gni       # GN 构建变量
├── LICENSE               # Apache 2.0 许可
├── README_zh.md          # 中文 README
├── figures/              # 文档图片
├── frameworks/           # 框架实现
├── interfaces/           # 对外接口
├── test/                 # 测试用例（不计入 Wiki）
└── wiki/                 # 文档目录
```

### 5.2 框架层结构（frameworks/）

```
frameworks/
├── js/                   # N-API JS 接口
│   └── napi/
│       ├── common/      # 公共 N-API 组件
│       ├── preferences/ # 首选项 JS API
│       ├── sendable_preferences/  # 可迁移首选项
│       ├── storage/     # 存储工具 JS API
│       └── system_storage/ # 系统存储 JS API
├── native/              # Native 实现
│   ├── include/         # 内部头文件
│   ├── platform/        # 平台相关（文件操作、锁等）
│   └── src/             # 核心实现
├── ndk/                 # NDK C 接口
├── cj/                  # CJ (C-JavaScript) FFI
├── ets/                 # ArkTS/Taihe 接口
└── common/              # 公共组件
```

### 5.3 接口层结构（interfaces/）

```
interfaces/
├── inner_api/           # 内部 Native API（C++）
│   ├── include/         # 头文件
│   └── BUILD.gn
└── ndk/                 # NDK C 接口
    ├── include/         # C 语言头文件
    └── BUILD.gn
```

---

## 6 关键概念

### 6.1 Key-Value 数据库

根据 `README_zh.md:13-15`：

> 一种以键值对存储数据的一种数据库。Key 是关键字，Value 是值。

### 6.2 非关系型数据库

根据 `README_zh.md:17-19`：

> 区别于关系数据库，不保证遵循 ACID（Atomic、Consistency、Isolation 及 Durability）特性，不采用关系模型来组织数据，数据之间无关系，扩展性好。

### 6.3 偏好数据

根据 `README_zh.md:21-23`：

> 用户经常访问和使用的数据。

### 6.4 Preferences 实例

根据 `README_zh.md:8`：

> 借助 getPreferences，可以将指定文件的内容加载到 Preferences 实例，每个文件最多有一个 Preferences 实例，系统会通过静态容器将该实例存储在内存中，直到主动从内存中移除该实例或者删除该文件。

**特性**：
- 单例模式：每个文件对应一个实例
- 内存缓存：实例常驻内存
- 手动管理：需要主动 flush() 持久化

---

## 7 与其他模块的关系

### 7.1 依赖关系

根据 `bundle.json:50-67`：

| 依赖组件 | 用途 |
|----------|------|
| ability_runtime | 应用上下文获取 |
| bundle_framework | Bundle 信息查询 |
| access_token | 权限管理 |
| napi | Node.js API 绑定 |
| hilog | 日志输出 |
| ipc | 进程间通信 |
| libxml2 | XML 解析 |
| ffrt | 任务调度 |

### 7.2 被依赖关系

本模块被以下模块使用：

- 应用层通过 JS/NDK API 调用
- 其他模块可能引用 inner API

---

## 8 快速开始

### 8.1 JS API 示例

```typescript
import preferences from '@ohos.data.preferences';

// 获取 Preferences 实例
let preferences = await preferences.getPreferences(context, 'myPreferences');

// 写入数据
await preferences.put('key1', 'value1');
await preferences.put('key2', 123);

// 读取数据
let value = await preferences.get('key1', 'default');

// 持久化
await preferences.flush();
```

### 8.2 NDK C API 示例

```c
#include "oh_preferences.h"

// 打开首选项
OH_PreferencesOption *option = OH_PreferencesOption_Create();
OH_PreferencesOption_SetName(option, "myPreferences");
int errCode = 0;
OH_Preferences *pref = OH_Preferences_Open(option, &errCode);

// 读取数据
int value = 0;
OH_Preferences_GetInt(pref, "key1", &value);

// 写入数据
OH_Preferences_SetInt(pref, "key2", 456);

// 关闭
OH_Preferences_Close(pref);
OH_PreferencesOption_Destroy(option);
```

---

## 9 相关文档

| 文档 | 说明 |
|------|------|
| [API 参考](./02_API_Reference.md) | 完整 API 清单（含 N-API、NDK、Inner API） |
| [架构设计](./01_Architecture.md) | 内部架构详解（含 Mermaid 图表） |
| [构建配置](./03_Build.md) | 构建流程说明（GN Targets、编译产物） |
| [安全评审](./04_Security_Review.md) | 安全风险分析（含 XXE、路径遍历等 8 个风险点） |
| [故障排查](./05_Troubleshooting.md) | 常见问题、错误码速查、调试方法 |

---

## 10 常见问题

### Q1: Preferences 和关系数据库有什么区别？

**答**：Preferences 是轻量级键值存储，不支持 SQL 查询，不保证 ACID，适合存储配置数据。大量数据或复杂查询场景建议使用 RDB（关系型数据库）。

### Q2: 数据存储在哪里？

**答**：默认存储在应用私有目录下的 XML 文件中。路径格式：`/data/app/el*/[bundleName]/pref/[preferencesName].xml`

### Q3: 如何监听数据变化？

**答**：使用 `on('change', listener)` 注册观察者，当数据变化时会触发回调。

### Q4: 存储有大小限制吗？

**答**：单个 Value 最大 16MB，单个 Key 最大 1024 字符。建议总数据量不超过 10000 条。

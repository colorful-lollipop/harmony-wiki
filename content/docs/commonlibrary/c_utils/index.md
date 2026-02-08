# c_utils 首页

## 简介

**c_utils** 是 OpenHarmony 的 C++ 公共基础类库，为标准系统提供常用的 C++ 开发工具类。

## 核心能力

| 类别 | 功能 | 关键组件 |
|------|------|----------|
| **文件系统** | 文件/目录操作、内存映射、匿名共享内存 | `file_ex`, `directory_ex`, `MappedFile`, `Ashmem` |
| **字符串** | 字符串处理、Unicode 转换 | `string_ex`, `unicode_ex` |
| **并发** | 线程池、读写锁、信号量 | `ThreadPool`, `RWLock`, `Semaphore` |
| **容器** | 线程安全容器、有序容器 | `SafeMap`, `SafeQueue`, `SortedVector` |
| **内存** | 引用计数、智能指针、RAII | `RefBase`, `sptr`, `wptr`, `unique_fd` |
| **序列化** | 数据序列化容器 | `Parcel` |
| **事件** | IO事件、定时器 | `IOEventReactor`, `Timer` |
| **模式** | 单例、观察者 | `Singleton`, `Observer` |

## 项目定位

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 标准系统                      │
├─────────────────────────────────────────────────────────────┤
│  应用层  │  框架层  │  服务层  │  内核层                      │
├──────────┼──────────┼──────────┼──────────────────────────────┤
│          │          │          │                              │
│   Apps   │  Ability │ Services │   Kernel                     │
│          │  Runtime │          │                              │
│          │          │          │                              │
├──────────┴──────────┴──────────┴──────────────────────────────┤
│                      公共基础库 (c_utils)                      │
│  ┌─────────┬─────────┬─────────┬─────────┬─────────┐         │
│  │ 文件系统 │  并发   │  内存   │  序列化 │  工具   │         │
│  └─────────┴─────────┴─────────┴─────────┴─────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## 运行环境

- **目标系统**: OpenHarmony 标准系统（Standard System）
- **支持平台**: 
  - OHOS/Linux（完整功能）
  - Windows/macOS（受限功能：Parcel, RefBase, String）
  - iOS（受限功能：Directory, Parcel, RefBase, RWLock, String）
  - Android（受限功能，无 hilog）

## 快速开始

### 依赖方式

在 `BUILD.gn` 中添加依赖：

```gn
ohos_shared_library("my_module") {
  external_deps = [
    "c_utils:utils",      # 动态库
    # 或
    "c_utils:utilsbase",  # 静态库
  ]
}
```

### 头文件包含

```cpp
#include "refbase.h"      // 引用计数
#include "parcel.h"       // 序列化
#include "safe_map.h"     // 线程安全Map
#include "file_ex.h"      // 文件操作
```

## 关键统计

| 指标 | 数值 |
|------|------|
| 对外头文件 | 32 个 |
| 源文件 | 24 个 cpp |
| 主要构建目标 | 3 个（utils, utilsbase, utils_rust）|
| 文档数量 | 22 篇中文 + 2 篇英文 |

## 相关资源

- [官方文档](../docs/zh-cn/)
- [源码](../base/)
- [构建配置](../base/BUILD.gn)

---

**下一页**: [项目概览](01_Overview.md)

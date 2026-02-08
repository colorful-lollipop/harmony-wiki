# 目录结构与模块职责

## 目的

本文档详细说明 startup_cangjie_wrapper 的目录结构和各模块的职责，帮助开发者快速定位代码。

## 适用范围

- 需要阅读源代码的开发者
- 准备修改或扩展项目的工程师

---

## 目录树

```
base/startup/startup_cangjie_wrapper/
├── .git/                        # Git 仓库
├── BUILD.gn                     # 根构建配置（22行）
├── LICENSE                      # Apache 2.0 License
├── OAT.xml                      # 开源审计追踪文件
├── README.md                    # 英文说明（56行）
├── README_zh.md                 # 中文说明（66行）
├── bundle.json                  # 组件元数据（43行）
├── figures/                     # 架构图目录
│   ├── startup_cangjie_wrapper_architecture_en.png
│   └── startup_cangjie_wrapper_architecture_zh.png
├── mock/                        # 模拟实现目录（Windows/Mac）
│   └── ohos.device_info.cj     # 模拟实现（416行）
├── ohos/                        # 仓颉启动恢复子系统接口实现
│   └── device_info/
│       ├── BUILD.gn             # 构建配置（37行）
│       └── device_info.cj       # 主实现文件（735行）
├── test/                        # 测试用例（已忽略）
│   └── device_info/
│       └── test/
└── wiki/                        # Wiki 文档
    ├── _work/                   # 工作目录
    │   ├── NOTES.md
    │   └── PLAN.md
    ├── SUMMARY.md
    ├── README.md
    ├── 00_Overview.md
    ├── 01_Project_Positioning.md
    ├── 02_Directory_Structure.md（当前文件）
    ├── 03_Architecture.md
    ├── 04_Cangjie_API.md
    ├── 05_Internal_API.md
    ├── 06_GN_Targets.md
    ├── 07_Build_Artifacts.md
    ├── 08_Security_Review.md
    ├── 09_Common_Issues.md
    └── appendix/
        ├── Callgraphs.md
        └── API_Reference.md
```

---

## 目录职责

### 根目录

| 文件 | 职责 | 代码证据 |
|------|------|---------|
| `BUILD.gn` | 定义根构建目标，复制 SDK 库 | `BUILD.gn:1-22` |
| `bundle.json` | 组件元数据、依赖、ROM/RAM 占用 | `bundle.json:1-43` |
| `LICENSE` | Apache 2.0 开源协议 | - |
| `README.md` | 英文项目说明 | `README.md:1-56` |
| `README_zh.md` | 中文项目说明 | `README_zh.md:1-66` |
| `OAT.xml` | 开源审计追踪文件 | - |

### figures/

| 文件 | 职责 |
|------|------|
| `startup_cangjie_wrapper_architecture_en.png` | 英文架构图 |
| `startup_cangjie_wrapper_architecture_zh.png` | 中文架构图 |

### mock/

**用途**: Windows/Mac 开发环境的模拟实现

| 文件 | 职责 | 代码证据 |
|------|------|---------|
| `ohos.device_info.cj` | 模拟实现，所有属性返回默认值 | `mock/ohos.device_info.cj:1-416` |

**特点**:
- 无 FFI 函数声明
- 所有公开属性返回默认值（String() 或 0）
- 不包含隐藏 API（productModelAlias、diskSN 等）
- 用于开发环境编译和测试

### ohos/

**用途**: 仓颉启动恢复子系统接口实现

#### ohos/device_info/

| 文件 | 职责 | 代码证据 |
|------|------|---------|
| `BUILD.gn` | 定义 ohos.device_info 共享库构建目标 | `ohos/device_info/BUILD.gn:1-37` |
| `device_info.cj` | 主实现文件，定义 DeviceInfo 类和 FFI 函数 | `ohos/device_info/device_info.cj:1-735` |

**关键代码结构**:
```cangjie
// Line 16-94: Package 声明和 FFI 函数
package ohos.device_info
import ohos.labels.{APILevel, Hide}
foreign {
    func FfiOHOSDeviceInfoDeviceType(): CString
    ...
}

// Line 96-709: DeviceInfo 类
public class DeviceInfo {
    // 32 个公开 static prop
    public static prop deviceType: String { get() { ... } }
    ...
    // 5 个 internal static prop（未实现）
    internal static prop productModelAlias: String { get() { return "" } }
    ...
}

// Line 711-734: PerformanceClassLevel 枚举
internal enum PerformanceClassLevel { ... }
```

### test/

**用途**: 测试用例

**注意**: 本文档不覆盖测试相关内容。

---

## 文件详细说明

### BUILD.gn（根目录）

**路径**: `/base/startup/startup_cangjie_wrapper/BUILD.gn`

**职责**: 定义 SDK 库复制目标

**关键内容**:
```gn
import("//build/templates/cangjie/cjc.gni")

startup_cangjie_wrapper_packages_ohos = [
    "//base/startup/startup_cangjie_wrapper/ohos/device_info:ohos.device_info"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_startup_cangjie_libs") {
  ohos_inputs = startup_cangjie_wrapper_packages_ohos
}
```

**证据**: `BUILD.gn:14-22`

---

### bundle.json

**路径**: `/base/startup/startup_cangjie_wrapper/bundle.json`

**职责**: 定义组件元数据

**关键信息**:
- 组件名称: startup_cangjie_wrapper
- 子系统: startup
- 版本: 6.1
- License: Apache License 2.0
- 适配系统: standard
- ROM: 150KB
- RAM: 116KB
- 依赖组件: cangjie_ark_interop, init
- 内部 kits: ohos.device_info

**证据**: `bundle.json:1-43`

---

### ohos/device_info/BUILD.gn

**路径**: `/base/startup/startup_cangjie_wrapper/ohos/device_info/BUILD.gn`

**职责**: 定义 ohos.device_info 共享库

**关键内容**:
```gn
ohos_cangjie_shared_library("ohos.device_info") {
  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.device_info.cj" ]
  } else {
    sources = [ "device_info.cj" ]
  }

  external_deps = [ "init:cj_device_info_ffi" ]
  cj_external_deps = [ "cangjie_ark_interop:ohos.labels" ]

  subsystem_name = "startup"
  part_name = "startup_cangjie_wrapper"
}
```

**证据**: `ohos/device_info/BUILD.gn:20-36`

---

### ohos/device_info/device_info.cj

**路径**: `/base/startup/startup_cangjie_wrapper/ohos/device_info/device_info.cj`

**职责**: 主实现文件

**代码结构**:

| 行号 | 内容 |
|------|------|
| 1-14 | Copyright 头部 |
| 16-17 | Package 声明和 import |
| 20-94 | FFI 函数声明（34 个） |
| 96-102 | DeviceInfo 类注解 |
| 103-709 | DeviceInfo 类实现 |
| 711-734 | PerformanceClassLevel 枚举 |

**证据**: `ohos/device_info/device_info.cj:1-735`

---

## 模块依赖图

```
┌─────────────────────────────────────┐
│      ohos.device_info             │
│   (ohos_cangjie_shared_library)   │
└──────────────┬────────────────────┘
               │ depends on
               ↓
┌─────────────────────────────────────┐
│    init:cj_device_info_ffi        │
│   (init 组件 SA 服务)             │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ cangjie_ark_interop:ohos.labels  │
│  (提供 APILevel、Hide 注解)       │
└─────────────────────────────────────┘
```

**证据**:
- `ohos/device_info/BUILD.gn:28-32`
- `bundle.json:24-26`

---

## 编译配置切换

### Windows/Mac 环境

使用模拟实现：

```gn
if (is_mingw || is_mac) {
  sources = [ "../../mock/ohos.device_info.cj" ]
}
```

**证据**: `ohos/device_info/BUILD.gn:22-24`

### Linux/OpenHarmony 环境

使用真实实现：

```gn
else {
  sources = [ "device_info.cj" ]
}
```

**证据**: `ohos/device_info/BUILD.gn:24-26`

---

## 关键结论

1. **结构简单**: 项目结构非常简单，仅包含一个主要模块 device_info
2. **职责清晰**: each 文件职责明确，边界清晰
3. **平台适配**: 通过编译配置切换实现不同平台的适配
4. **依赖明确**: 仅依赖 init 组件和 cangjie_ark_interop
5. **无子模块**: 无 src、include、framework 等复杂子目录
6. **单一实现**: 核心逻辑集中在 device_info.cj 文件中

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Project_Positioning.md](01_Project_Positioning.md) - 项目定位
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单
- [06_GN_Targets.md](06_GN_Targets.md) - GN 目标详解

---

## 参考资料

- [OpenHarmony GN 构建系统](https://docs.openharmony.cn/application-dev/quick-start/naming-rules)
- [仓颉语言包管理](https://developer.openharmony.cn/cn/doc/cangjie-package)

---

*最后更新: 2026-02-06*

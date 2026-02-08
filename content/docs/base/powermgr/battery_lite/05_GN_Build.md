# GN 构建文档

> **目的**: 说明 battery_lite 项目的 GN 构建配置、Targets 列表和编译产物。  
> **适用范围**: 需要理解构建系统或进行编译的开发者。  
> **关键结论**: 项目使用 GN 构建系统，通过条件编译支持 mini 和 small 两种系统类型，产出静态库或共享库。

---

## 根构建配置

### BUILD.gn

**位置**: `base/powermgr/battery_lite/BUILD.gn`  
**类型**: lite_component

```gn
import("//build/lite/config/component/lite_component.gni")

lite_component("batterymgr_lite") {
  features = [
    "frameworks:batterymgr",      # Native 框架库
    "services:batteryservice",     # 电池服务
  ]
}
```

**证据**: `BUILD.gn:14-21`。

### batterymgr.gni

**位置**: `base/powermgr/battery_lite/batterymgr.gni`  
**用途**: 定义构建参数和系统类型判断

```gn
batterymgr_path = "//base/powermgr/battery_lite"
batterymgr_frameworks_path = "${batterymgr_path}/frameworks/native"
batterymgr_interfaces_path = "${batterymgr_path}/interfaces"
batterymgr_innerkits_path = "${batterymgr_interfaces_path}/innerkits"
batterymgr_kits_path = "${batterymgr_interfaces_path}/kits"
batterymgr_services_path = "${batterymgr_path}/services"
batterymgr_utils_path = "//base/powermgr/powermgr_lite/utils"

declare_args() {
  battery_mini_system = false
  battery_small_system = false
}

if (ohos_kernel_type == "liteos_m") {
  battery_mini_system = true
  battery_library_type = "static_library"  # 静态库
  battery_system_type = "mini"
} else {
  battery_small_system = true
  battery_library_type = "shared_library"  # 共享库
  battery_system_type = "small"
}
```

**证据**: `batterymgr.gni:17-44`。

---

## Targets 列表

### 框架层 Targets

#### frameworks:batterymgr

**位置**: `frameworks/BUILD.gn`  
**类型**: lite_library  
**输出类型**: 根据 `battery_library_type` 决定

| 属性 | 值 |
|------|-----|
| 目标类型 | `battery_library_type` (静态库/共享库) |
| 包含目录 | `include`, `include/mini` 或 `include/small` |
| 公共配置 | `:batterymgr_public_config` |
| 依赖 | `samgr`, `hilog_lite`, `ipc` |

**条件编译** (mini vs small):

```gn
if (battery_mini_system) {
  deps = [ "//base/powermgr/battery_lite/frameworks/native/src/mini:battery_impl" ]
} else {
  deps = [
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//base/powermgr/battery_lite/frameworks/native/src/small:battery_impl",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
  ]
}
```

**证据**: `frameworks/BUILD.gn:20-35`。

#### frameworks/native/src/mini:battery_impl

**位置**: `frameworks/native/src/mini/BUILD.gn`  
**类型**: static_library

| 属性 | 值 |
|------|-----|
| 源文件 | `battery_framework.c` |
| 包含目录 | `kits`, `include`, `services/include`, `include/mini` |
| 依赖 | `posix`, `samgr` |

**证据**: `frameworks/native/src/mini/BUILD.gn:17-32`。

#### frameworks/native/src/small:battery_impl

**位置**: `frameworks/native/src/small/BUILD.gn`  
**类型**: static_library (同 mini 结构)

**证据**: 位置与 mini 类似，适配 small 系统差异。

### JS 框架 Targets

#### ace_battery_kits

**位置**: `frameworks/js/BUILD.gn`  
**类型**: lite_component

```gn
lite_component("ace_battery_kits") {
  features = [ "builtin:ace_kit_battery" ]
}
```

**证据**: `frameworks/js/BUILD.gn:18-20`。

#### builtin:ace_kit_battery

**位置**: `frameworks/js/builtin/BUILD.gn`  
**用途**: 构建 JS 电池模块

| 属性 | 值 |
|------|-----|
| 源文件 | `battery_module.cpp`, `battery_impl.c` |
| 包含目录 | `include`, `../../../native/include` |

**证据**: 需查看 `frameworks/js/builtin/BUILD.gn`。

### 服务层 Targets

#### services:batteryservice

**位置**: `services/BUILD.gn` (需确认存在)  
**类型**: service_component  
**用途**: 构建电池服务

**证据**: `BUILD.gn:19` 引用了 `"services:batteryservice"`。

---

## 编译产物

### 产物清单

| 产物名 | 类型 | 说明 |
|--------|------|------|
| `libbatterymgr.so` | 共享库 | Small 系统 Native 框架库 |
| `libbatterymgr.a` | 静态库 | Mini 系统 Native 框架库 |
| `libbatteryservice.so` | 共享库 | Small 系统服务 |
| `libbatteryservice.a` | 静态库 | Mini 系统服务 |

**证据**: `bundle.json:33-36` 声明了输出产物。

### 安装路径

| 产物 | 典型安装路径 |
|------|--------------|
| 共享库 | `/system/lib/` |
| 服务 | `/system/bin/` |
| 头文件 | `/include/` |

**说明**: 实际路径取决于系统配置和编译目标。

### 产物依赖关系

```
┌─────────────────────────────────────────────────────┐
│                  应用二进制                         │
├─────────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │         libbatterymgr.so/a                 │  │
│  │    (frameworks:batterymgr 产物)            │  │
│  └─────────────────────────────────────────────┘  │
│                    │                              │
│                    ▼                              │
│  ┌─────────────────────────────────────────────┐  │
│  │         libbatteryservice.so/a              │  │
│  │    (services:batteryservice 产物)          │  │
│  └─────────────────────────────────────────────┘  │
│                    │                              │
│                    ▼                              │
│  ┌─────────────────────────────────────────────┐  │
│  │              samgr_lite                     │  │
│  │         (系统服务管理框架)                   │  │
│  └─────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 依赖组件

### 系统组件依赖

| 组件 | 用途 | 来源 |
|------|------|------|
| `utils_lite` | 基础工具函数 | 系统组件 |
| `samgr_lite` | 服务能力管理 | 系统组件 |
| `ipc` | 进程间通信 | 系统组件 |
| `hilog_lite` | 日志输出 | 系统组件 |

**证据**: `bundle.json:38-42`。

### 第三方依赖

| 组件 | 用途 | 来源 |
|------|------|------|
| `bounds_checking_function` | 字符串安全函数 | 第三方 |

**证据**: `bundle.json:44-46`。

### 内部组件依赖

| 组件 | 用途 |
|------|------|
| `samgr` | 服务注册与发现 |
| `posix` | POSIX 接口 (仅 mini) |
| `hilog_shared` | 日志共享库 (仅 small) |
| `ipc_single` | IPC 单例模式 (仅 small) |

---

## 构建命令

### 全量构建

```bash
# 构建整个电池管理组件
hb set
hb build -f
```

### 单独构建

```bash
# 构建 batterymgr_lite 组件
hb build //base/powermgr/battery_lite:batterymgr_lite
```

### 清理构建

```bash
hb clean
```

---

## 配置开关

### 系统类型配置

```gn
# batterymgr.gni
declare_args() {
  battery_mini_system = false
  battery_small_system = false
}

if (ohos_kernel_type == "liteos_m") {
  battery_mini_system = true
  battery_library_type = "static_library"
  battery_system_type = "mini"
} else {
  battery_small_system = true
  battery_library_type = "shared_library"
  battery_system_type = "small"
}
```

### 功能配置

```gn
# config.gni (当前为空)
declare_args() {
  # enable_screensaver = false
}
```

---

## 产物映射表

| BUILD.gn Target | 系统类型 | 输出类型 | 输出名 |
|-----------------|----------|----------|--------|
| `frameworks:batterymgr` | mini | 静态库 | `libbatterymgr.a` |
| `frameworks:batterymgr` | small | 共享库 | `libbatterymgr.so` |
| `frameworks/native/src/mini:battery_impl` | mini | 静态库 | `libbattery_impl.a` |
| `ace_battery_kits` | all | 组件 | JS 电池模块 |
| `services:batteryservice` | mini/small | 库/服务 | `libbatteryservice.*` |

---

## 常见构建问题

### 问题 1: 找不到 samgr 依赖

**现象**: 编译报错找不到 `samgr_lite`

**解决**: 确保已完整拉取源码树：
```bash
repo sync --force-local
```

### 问题 2: 头文件包含错误

**现象**: 编译报错找不到 `battery_info.h`

**解决**: 检查 `include_dirs` 是否包含 `${batterymgr_kits_path}`

### 问题 3: 产物类型不符合预期

**现象**: 生成了共享库但期望静态库

**解决**: 检查 `battery_library_type` 是否正确设置

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [目录结构](01_Directory_Structure.md) | 代码组织结构 |
| [架构说明](02_Architecture.md) | 组件和数据流 |
| [内部 API](04_Inner_API.md) | 模块接口 |
| [安全评审](06_Security_Review.md) | 安全考量 |

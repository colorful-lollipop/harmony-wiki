# 目录结构

> **目的**: 说明 battery_lite 项目的代码组织结构、各模块职责和依赖关系。  
> **适用范围**: 所有需要理解代码布局的开发者。  
> **关键结论**: 项目采用分层架构（服务层→框架层→接口层），支持 mini 和 small 系统变体，通过条件编译实现差异化代码。

---

## 顶层目录结构

```
base/powermgr/battery_lite/
├── figures/                    # 架构图和文档图片
├── frameworks/                # 框架层
│   ├── js/                    # JS 运行时框架
│   │   ├── builtin/          # 内置 JS 模块
│   │   │   ├── include/      # JS 模块头文件
│   │   │   └── src/          # JS 模块源文件
│   │   └── BUILD.gn          # JS 框架构建配置
│   └── native/               # Native 框架
│       ├── include/           # Native 框架头文件
│       │   ├── mini/         # Mini 系统专用接口
│       │   └── small/        # Small 系统专用接口
│       ├── src/              # Native 框架实现
│       │   ├── mini/         # Mini 系统实现
│       │   └── small/        # Small 系统实现
│       └── BUILD.gn           # Native 框架构建配置
├── interfaces/               # 接口层
│   └── kits/                 # 外部接口
│       ├── battery_info.h    # 公共头文件
│       └── js/               # JS API 类型定义
│           └── @system.battery.d.ts  # TypeScript 定义
├── services/                 # 服务层
│   ├── include/              # 服务头文件
│   └── src/                 # 服务实现
│       ├── mini/            # Mini 系统适配
│       └── small/           # Small 系统适配
├── test/                    # 测试用例（不在本文档范围内）
├── BUILD.gn                 # 根构建入口
├── batterymgr.gni           # 构建配置参数
├── config.gni              # 功能开关配置
├── bundle.json             # 组件配置
└── README.md               # 项目说明文档
```

**证据**: `README_zh.md:27-39` 描述了项目目录结构。

---

## 各模块职责

### services/ - 服务层

**职责**: 提供电池管理核心服务实现，通过 SAMgr 注册为系统服务。

| 子目录/文件 | 职责 |
|-------------|------|
| `include/battery_device.h` | 定义电池设备特征 API (`BatteeryDeviceFeatureApi`)，包含 12 个操作接口 |
| `include/ibattery.h` | 定义 `IBattery` 接口和 `BatInfo` 数据结构 |
| `include/battery_manage_service.h` | 定义 `BatteryService` 服务结构 |
| `include/battery_manage_feature.h` | 定义 `BatteryFeatureApi` 特征实现 |
| `src/battery_device.c` | 电池设备服务实现，包含 LED 控制、关机、信息更新 |
| `src/battery_manage_feature.c` | 电池管理特征实现，作为服务代理层 |
| `src/battery_manage_service.c` | 电池管理服务初始化（可选实现） |
| `src/mini/battery_feature_impl.c` | Mini 系统的特征实现适配 |
| `src/small/battery_feature_impl.c` | Small 系统的特征实现适配 |

**关键证据**:
- `services/include/battery_device.h:46-61` 定义了 `BatteeryDeviceFeatureApi` 结构
- `services/include/ibattery.h:26-43` 定义了 `BatInfo` 数据结构

### frameworks/native/ - Native 框架层

**职责**: 提供 Native 应用使用的电池管理框架，封装 IPC 调用细节。

| 子目录/文件 | 职责 |
|-------------|------|
| `include/battery_framework.h` | 定义 `GetBatteryIUnknown()` 接口获取函数 |
| `include/battery_mgr.h` | 定义服务名常量 (`BATTERY_SERVICE`, `BATTERY_INNER`) |
| `include/batterymgr_intf_define.h` | 定义 `INHERIT_BATTERY_INTERFACE` 接口宏 |
| `include/mini/battery_interface.h` | Mini 系统专用电池接口定义 |
| `include/small/battery_interface.h` | Small 系统专用电池接口定义 |
| `src/mini/battery_framework.c` | Mini 系统框架实现，使用互斥锁保护接口获取 |
| `src/small/battery_framework.c` | Small 系统框架实现 |

**关键证据**:
- `frameworks/native/include/battery_framework.h:31-39` 定义了 `GetBatteryIUnknown()` 内联函数
- `frameworks/native/include/batterymgr_intf_define.h:26-33` 定义了接口宏

### frameworks/js/ - JS 框架层

**职责**: 提供 ACELite JS 运行时使用的电池管理模块。

| 子目录/文件 | 职责 |
|-------------|------|
| `builtin/include/battery_module.h` | 定义 `BatteryModule` 类，包含 7 个静态方法 |
| `builtin/include/battery_impl.h` | 定义 JS 到 Native 的桥接实现函数 |
| `builtin/src/battery_module.cpp` | JS 模块实现，包含参数解析和回调处理 |
| `builtin/src/battery_impl.c` | 桥接函数实现，调用 Native 接口 |

**关键证据**:
- `frameworks/js/builtin/include/battery_module.h:24-35` 定义了 `BatteryModule` 类
- `frameworks/js/builtin/src/battery_module.cpp:44-168` 实现了 7 个 JS API

### interfaces/ - 接口层

**职责**: 定义对外公共 API 和类型定义。

| 文件 | 职责 |
|------|------|
| `kits/battery_info.h` | 定义枚举类型和公共查询函数签名 |
| `kits/js/@system.battery.d.ts` | TypeScript 类型定义，供 JS 开发使用 |

**关键证据**:
- `interfaces/kits/battery_info.h:23-106` 定义了枚举类型和公共 API
- `interfaces/kits/js/@system.battery.d.ts:169-212` 定义了 JS 类和方法签名

---

## 条件编译结构

### Mini vs Small 系统变体

项目通过条件编译支持 mini 和 small 两种系统类型：

```c
// batterymgr.gni 配置逻辑
if (ohos_kernel_type == "liteos_m") {
  battery_mini_system = true
  battery_library_type = "static_library"  // 静态库
  battery_system_type = "mini"
} else {
  battery_small_system = true
  battery_library_type = "shared_library"  // 共享库
  battery_system_type = "small"
}
```

**证据**: `batterymgr.gni:31-44` 定义了系统类型判断逻辑。

### 系统类型对应的实现文件

| 系统类型 | 框架实现 | 特征实现 | 库类型 |
|----------|----------|----------|--------|
| mini | `src/mini/battery_framework.c` | `src/mini/battery_feature_impl.c` | 静态库 |
| small | `src/small/battery_framework.c` | `src/small/battery_feature_impl.c` | 共享库 |

---

## 头文件包含关系

```
对外公共头文件（应用使用）
└── interfaces/kits/battery_info.h
    ├── BatteryChargeState 枚举
    ├── BatteryHealthState 枚举
    ├── BatteryPluggedType 枚举
    └── 7 个公共函数声明

框架层头文件（框架使用）
├── frameworks/native/include/battery_framework.h
│   └── GetBatteryIUnknown()
├── frameworks/native/include/battery_mgr.h
│   └── 服务名常量
└── frameworks/native/include/batterymgr_intf_define.h
    └── INHERIT_BATTERY_INTERFACE 宏

服务层头文件（服务内部使用）
├── services/include/battery_device.h
│   ├── BatteeryDeviceFeatureApi 结构
│   └── 设备相关常量和类型
├── services/include/ibattery.h
│   ├── IBattery 接口
│   └── BatInfo 数据结构
└── services/include/battery_manage_feature.h
    └── BatteryFeatureApi 结构
```

---

## 依赖关系

### 外部依赖

| 依赖组件 | 用途 | 来源 |
|----------|------|------|
| `utils_lite` | 基础工具函数 | 系统组件 |
| `samgr_lite` | 服务能力管理 | 系统组件 |
| `ipc` | 进程间通信 | 系统组件 |
| `hilog_lite` | 日志输出 | 系统组件 |
| `bounds_checking_function` | 字符串安全函数 | 第三方 |

**证据**: `bundle.json:38-46` 列出了所有依赖组件。

### 内部依赖

```
interfaces/kits/battery_info.h
    ↓
frameworks/native/include/*
    ↓
services/include/*
    ↓
services/src/*
```

---

## 不含测试的说明

本文档描述的目录结构**不包含**以下测试相关内容：

- `test/` 目录下的所有文件
- `*_test.*` 命名模式的文件
- 测试用例和测试框架代码

测试代码不在本文档描述范围内，因为其不构成组件的业务功能和对外接口。

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [架构说明](02_Architecture.md) | 组件图和数据流 |
| [N-API 接口](03_N_API.md) | 完整 API 清单 |
| [GN 构建](05_GN_Build.md) | 构建 Targets 和产物 |

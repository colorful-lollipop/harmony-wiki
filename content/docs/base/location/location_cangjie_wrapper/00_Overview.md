# 项目概览

> location_cangjie_wrapper 项目定位、能力边界与关键概念

## 项目定位

### 定位声明

`location_cangjie_wrapper` 是 OpenHarmony 平台上位置服务的 **Cangjie（仓颉）API 封装层**。该项目为开发者提供基于 Cangjie 语言的位置服务能力，使应用能够获取设备的实时准确位置数据。

### 核心能力

| 能力 | 状态 | 说明 |
|------|------|------|
| 获取当前位置 | ✅ 支持 | 通过 GNSS 定位获取设备当前位置 |
| 检查位置开关 | ✅ 支持 | 判断设备位置服务是否开启 |
| 位置精度配置 | ✅ 支持 | 支持设置定位精度优先级 |
| 定位场景配置 | ✅ 支持 | 支持导航、轨迹追踪等场景 |
| 网络定位 | ❌ 暂不支持 | 蜂窝基站、WLAN、蓝牙定位 |
| 历史位置获取 | ❌ 暂不支持 | 获取最近缓存的位置信息 |
| 地理围栏 | ❌ 暂不支持 | 虚拟地理边界通知 |
| 地理编码 | ❌ 暂不支持 | 地址与坐标转换 |
| 位置监听 | ❌ 暂不支持 | 持续位置更新监听 |
| 国家码管理 | ❌ 暂不支持 | 位置状态管理 |

### 目标设备

- **标准设备（Standard）**：完整支持
- **轻量设备（Lite）**：不支持
- **小型设备（Mini）**：不支持

## 关键概念

### 坐标系统

系统采用 **WGS84（World Geodetic System 1984）** 作为地理坐标参考系统，使用经度（Longitude）和纬度（Latitude）描述地球上的位置。

| 坐标 | 范围 | 正值方向 |
|------|------|----------|
| 纬度（Latitude） | -90° ~ 90° | 北纬为正 |
| 经度（Longitude） | -180° ~ 180° | 东经为正 |

### GNSS 定位

GNSS（Global Navigation Satellite System）定位基于全球导航卫星系统，包括：

- **GPS**（美国全球定位系统）
- **GLONASS**（俄罗斯全球导航卫星系统）
- **BeiDou**（中国北斗卫星导航系统）
- **Galileo**（欧盟伽利略卫星导航系统）

定位过程中具体使用哪些卫星系统，取决于设备硬件能力。

### 位置精度等级

| 枚举值 | 说明 | 适用场景 |
|--------|------|----------|
| `Accuracy` | 优先保证定位精度 | 导航、地图 |
| `LowPower` | 优先保证低功耗 | 日常使用 |
| `FirstFix` | 优先保证首次定位速度 | 快速定位 |

### 定位场景

| 枚举值 | 说明 | 特点 |
|--------|------|------|
| `Navigation` | 导航场景 | 高精度、实时性 |
| `TrajectoryTracking` | 轨迹追踪 | 高精度 |
| `CarHailing` | 网约车 | 高精度、实时性 |
| `DailyLifeService` | 日常生活 | 低精度、低实时性 |
| `NoPower` | 节能模式 | 最低功耗 |

## 目录结构

```
base/location/location_cangjie_wrapper/
├── figures/                              # 架构图等资源文件
│   └── location_cangjie_wrapper_architecture.png
│
├── kit/                                  # Cangjie Kit 化代码
│   └── LocationKit/                     # LocationKit 模块
│       ├── BUILD.gn                      # Kit 构建配置
│       └── index.cj                      # 模块导出入口
│
├── ohos/                                 # 仓颉接口实现
│   └── geo_location_manager/             # geo_location_manager 模块
│       ├── BUILD.gn                      # 模块构建配置
│       ├── geo_location_manager.cj       # GeoLocationManager 核心类
│       ├── geo_location_manager_common.cj # 类型定义和公共逻辑
│       └── geo_location_manager_ffi.cj   # FFI 绑定实现
│
├── mock/                                 # Mock 实现（Windows/Mac 编译）
│   └── ohos.geo_location_manager.cj      # Mock 类实现
│
└── test/                                 # 测试用例（不纳入本文档范围）
```

### 代码职责划分

| 目录 | 职责 | 代码证据 |
|------|------|----------|
| `kit/LocationKit/` | 模块导出入口 | `index.cj:18` - package 声明 |
| `ohos/geo_location_manager/` | 核心实现 | 多文件协作实现 |
| `mock/` | 跨平台 Mock | BUILD.gn:20-28 条件编译 |

## 运行环境

### 系统要求

| 要求 | 版本/配置 |
|------|-----------|
| OpenHarmony | Standard 系统 |
| API Level | 22+ |
| Cangjie 运行时 | ArkCompiler Cangjie |
| 系统能力 | SystemCapability.Location.Location.Core |

### 依赖组件

| 组件名 | 版本 | 依赖说明 |
|--------|------|----------|
| `cangjie_ark_interop` | - | C 语言互操作、标签类、异常定义 |
| `hiviewdfx_cangjie_wrapper` | - | 日志接口（Hilog） |
| `location` | - | 位置服务组件（C++ FFI 实现） |

证据来源：`bundle.json:25-30`

### 资源占用

| 资源 | 占用量 | 说明 |
|------|--------|------|
| ROM | ~120KB | 编译产物总大小 |
| RAM | ~108KB | 运行时内存占用 |

证据来源：`bundle.json:22-23`

## 权限要求

### 应用权限

使用位置服务需要申请以下权限：

| 权限名 | 用途 | 敏感级别 |
|--------|------|----------|
| `ohos.permission.APPROXIMATELY_LOCATION` | 获取粗略位置 | normal |

### 使用限制

1. **用户确认**：用户必须开启设备「位置」开关
2. **权限申请**：应用需要向用户申请位置访问权限
3. **双重确认**：即使位置开关开启，仍需用户授权权限

### 错误码

| 错误码 | 含义 | 处理建议 |
|--------|------|----------|
| 201 | 权限验证失败 | 检查权限申请状态 |
| 801 | 设备能力不支持 | 检查 syscap 配置 |
| 3301000 | 位置服务不可用 | 检查位置开关 |
| 3301100 | 位置开关关闭 | 引导用户开启 |
| 3301200 | 获取位置失败 | 重试或检查定位条件 |

证据来源：`geo_location_manager.cj:38-44`

## 相关资源

### 内部链接

| 文档 | 说明 |
|------|------|
| [README](../README.md) | 项目原始说明 |
| [README_zh](../README_zh.md) | 中文项目说明 |
| [01_Architecture](01_Architecture.md) | 系统架构详解 |
| [02_API_Reference](02_API_Reference.md) | API 使用说明 |
| [03_Build](03_Build.md) | 构建配置说明 |
| [04_Security](04_Security.md) | 安全注意事项 |

### 外部链接

- [Location 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/Dev_Guide/source_zh_cn/location/cj-location-guidelines.md)
- [LocationKit API 参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_zh_cn/apis/LocationKit)
- [OpenHarmony 代码贡献指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/contribute/参与贡献.md)

### 相关仓库

| 仓库 | 说明 |
|------|------|
| [base_location](https://gitee.com/openharmony/base_location) | 位置服务原生组件 |
| [arkcompiler_cangjie_ark_interop](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop) | Cangjie-ArkTS 互操作 |
| [hiviewdfx_hiviewdfx_cangjie_wrapper](https://gitee.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) | Cangjie 日志封装 |

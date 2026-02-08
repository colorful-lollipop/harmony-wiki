# 模块职责

> **目的**: 了解项目的目录结构和模块职责划分
> **适用范围**: 新人快速定位代码、重构参考
> **最后更新**: 2025-02-06

---

## 目录结构

```
base/powermgr/powermgr_cangjie_wrapper
├── figures/                       # 架构图和说明图片
│   ├── power-management-subsystem-architecture.png
│   ├── powermgr_cangjie_wrapper_architecture.png
│   ├── powermgr_cangjie_wrapper_architecture_en.png
│   └── 电源管理子系统架构图.png
├── ohos/                         # Cangjie 源代码
│   └── battery_info/             # 电池信息模块
│       ├── battery_info.cj        # API 定义（BatteryInfo 类和枚举）
│       ├── native.cj             # FFI 外部函数声明
│       └── BUILD.gn              # 构建配置
├── mock/                         # Mock 实现（非生产环境）
│   └── ohos.battery_info.cj      # Windows/macOS Mock
├── test/                         # 测试用例（不作为业务证据）
│   └── battery_info/
├── wiki/                         # 工程文档（本文档目录）
├── BUILD.gn                      # 根构建配置
├── bundle.json                   # 组件描述
├── README.md                     # 英文说明
├── README_zh.md                  # 中文说明
├── LICENSE                       # Apache-2.0 许可证
└── OAT.xml                      # 测试配置

```

**证据**: 项目根目录结构

---

## 模块职责

### 1. ohos/battery_info - 电池信息 API 模块

**职责**: 提供电池信息查询的 Cangjie API

**子模块**:

#### 1.1 battery_info.cj - 主 API 定义

**职责**: 定义 `BatteryInfo` 类和相关枚举

**内容**:
- `BatteryInfo` 类：包含 10 个静态属性，提供电池信息查询
- `BatteryPluggedType` 枚举：充电器类型（4 个值）
- `BatteryChargeState` 枚举：充电状态（4 个值）
- `BatteryHealthState` 枚举：健康状态（6 个值）
- `BatteryCapacityLevel` 枚举：容量等级（7 个值）
- 枚举的 `parse()` 静态方法：将 Int32 转换为枚举值

**证据**: `ohos/battery_info/battery_info.cj:18-449`

#### 1.2 native.cj - FFI 声明

**职责**: 声明 FFI 外部函数，绑定 C 层接口

**内容**: 声明 10 个 FFI 函数（与 BatteryInfo 属性一一对应）

**证据**: `ohos/battery_info/native.cj:20-40`

#### 1.3 BUILD.gn - 模块构建配置

**职责**: 定义电池信息模块的构建规则

**内容**:
- Target 类型：`ohos_cangjie_shared_library`
- Target 名称：`ohos.battery_info`
- 条件编译：Windows/macOS 使用 mock 实现
- 依赖声明：Cangjie 库和 C FFI 库

**证据**: `ohos/battery_info/BUILD.gn:19-39`

### 2. mock - Mock 实现模块

**职责**: 为非生产环境（Windows/macOS）提供 Mock 实现

**内容**:
- 与生产代码相同的 API 结构
- 所有属性返回固定的默认值
- 不调用任何 FFI 函数

**证据**: `mock/ohos.battery_info.cj:17-406`

### 3. figures - 文档图片

**职责**: 存储架构图和说明图片

**内容**:
- 电源管理子系统架构图
- powermgr_cangjie_wrapper 架构图（中文/英文）

**证据**: `figures/` 目录

### 4. BUILD.gn - 根构建配置

**职责**: 定义组件级别的构建规则

**内容**:
- 导入 Cangjie 构建模板
- 定义包列表
- 创建 SDK copy target

**证据**: `BUILD.gn:14-21`

### 5. bundle.json - 组件描述

**职责**: 向 OpenHarmony 构建系统声明组件信息

**内容**:
- 组件元数据（名称、版本、许可证）
- 子系统归属
- 目标系统类型
- 资源占用（ROM/RAM）
- 依赖组件
- 子组件列表

**证据**: `bundle.json:1-43`

---

## 模块依赖关系

### 依赖方向

```
Cangjie 应用
    ↓
ohos.battery_info (powermgr_cangjie_wrapper)
    ↓
battery_manager:cj_battery_info_ffi (C FFI)
    ↓
battery_manager 服务 / 底层驱动
```

### 详细依赖

#### ohos/battery_info 模块的依赖

**Cangjie 依赖** (cj_external_deps):
- `cangjie_ark_interop:ohos.business_exception` - 业务异常类
- `cangjie_ark_interop:ohos.labels` - API Level 标注

**C 依赖** (external_deps):
- `battery_manager:cj_battery_info_ffi` - 电池管理 FFI 接口

**证据**: `ohos/battery_info/BUILD.gn:30-35`

#### powermgr_cangjie_wrapper 组件的依赖

**依赖组件** (bundle.json):
- `cangjie_ark_interop`
- `battery_manager`

**证据**: `bundle.json:24-26`

---

## 职责边界

### 不在当前模块的职责

- ❌ 驱动层实现（由 `battery_manager` 负责）
- ❌ 系统电源管理策略（由 `power_manager` 负责）
- ❌ 电量上报和状态更新（由底层服务负责）
- ❌ 测试用例（在 `test/` 目录，不作为业务证据）

### 当前模块的单一职责

- ✅ 封装 C FFI 接口为 Cangjie API
- ✅ 提供类型安全的枚举转换
- ✅ 统一错误处理机制

---

## 代码组织原则

1. **模块化**: 每个文件职责单一（API 定义、FFI 声明、构建配置分离）
2. **类型安全**: 使用枚举而非原始 Int32 值
3. **内存管理**: `CString` 需要手动释放（`LibC.free()`）
4. **条件编译**: Mock 和生产实现分离
5. **文档化**: 所有公共 API 都有注释和 API Level 标注

---

## 相关跳转

- [架构说明](02_Architecture.md) - 查看组件图和数据流
- [对外 API](03_Public_API.md) - API 详细清单
- [内部 API](04_Internal_API.md) - 模块接口和稳定性
- [GN Targets](05_GN_Targets.md) - 构建配置详解

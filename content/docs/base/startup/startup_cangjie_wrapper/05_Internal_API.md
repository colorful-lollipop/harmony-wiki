# 内部 API

## 目的

本文档说明 startup_cangjie_wrapper 的内部 API、模块接口、依赖关系和稳定性。

## 适用范围

- 需要深入理解内部实现的开发者
- 准备扩展或修改组件的工程师

---

## 概述

### 内部 API 分类

| 类型 | 数量 | 说明 |
|------|------|------|
| FFI 函数（foreign func） | 34 | 桥接到 init 组件 SA 服务 |
| 隐藏属性（internal static prop） | 5 | 未实现或不稳定的 API |
| 枚举（internal enum） | 1 | PerformanceClassLevel |

### API 可见性

| 可见性 | 类型 | 数量 |
|--------|------|------|
| public | DeviceInfo 类及其 32 个属性 | 1 类 32 属性 |
| internal | PerformanceClassLevel 枚举、5 个隐藏属性 | 1 枚举 5 属性 |
| foreign | 34 个 FFI 函数 | 34 函数 |

---

## FFI 函数

### 概述

**类型**: foreign func

**命名模式**: `FfiOHOSDeviceInfo*()`

**返回类型**: `CString` 或 `Int64`

**总数**: 34 个

**依赖**: `init:cj_device_info_ffi`

**证据**: `ohos/device_info/device_info.cj:22-94`

### FFI 函数清单

| 序号 | FFI 函数 | 返回类型 | 对应的公开 API | 代码行 |
|------|----------|---------|---------------|--------|
| 1 | FfiOHOSDeviceInfoDeviceType | CString | deviceType | 25 |
| 2 | FfiOHOSDeviceInfoOsFullName | CString | osFullName | 27 |
| 3 | FfiOHOSDeviceInfoProductModel | CString | productModel | 29 |
| 4 | FfiOHOSDeviceInfoBrand | CString | brand | 31 |
| 5 | FfiOHOSDeviceInfoUdid | CString | udid | 33 |
| 6 | FfiOHOSDeviceInfoBuildRootHash | CString | buildRootHash | 35 |
| 7 | FfiOHOSDeviceInfoBuildTime | CString | buildTime | 37 |
| 8 | FfiOHOSDeviceInfoBuildHost | CString | buildHost | 39 |
| 9 | FfiOHOSDeviceInfoBuildUser | CString | buildUser | 41 |
| 10 | FfiOHOSDeviceInfoBuildType | CString | buildType | 43 |
| 11 | FfiOHOSDeviceInfoVersionId | CString | versionId | 45 |
| 12 | FfiOHOSDeviceInfoFirstApiVersion | Int64 | firstApiVersion | 47 |
| 13 | FfiOHOSDeviceInfoSdkApiVersion | Int64 | sdkApiVersion | 49 |
| 14 | FfiOHOSDeviceInfoBuildVersion | Int64 | buildVersion | 51 |
| 15 | FfiOHOSDeviceInfoFeatureVersion | Int64 | featureVersion | 53 |
| 16 | FfiOHOSDeviceInfoSeniorVersion | Int64 | seniorVersion | 55 |
| 17 | FfiOHOSDeviceInfoMajorVersion | Int64 | majorVersion | 57 |
| 18 | FfiOHOSDeviceInfoDisplayVersion | CString | displayVersion | 59 |
| 19 | FfiOHOSDeviceInfoSerial | CString | serial | 61 |
| 20 | FfiOHOSDeviceInfoOsReleaseType | CString | osReleaseType | 63 |
| 21 | FfiOHOSDeviceInfoIncrementalVersion | CString | incrementalVersion | 65 |
| 22 | FfiOHOSDeviceInfoSecurityPatchTag | CString | securityPatchTag | 67 |
| 23 | FfiOHOSDeviceInfoAbiList | CString | abiList | 69 |
| 24 | FfiOHOSDeviceInfoBootloaderVersion | CString | bootloaderVersion | 71 |
| 25 | FfiOHOSDeviceInfoHardwareModel | CString | hardwareModel | 73 |
| 26 | FfiOHOSDeviceInfoSoftwareModel | CString | softwareModel | 75 |
| 27 | FfiOHOSDeviceInfoProductSeries | CString | productSeries | 77 |
| 28 | FfiOHOSDeviceInfoManufacture | CString | manufacture | 79 |
| 29 | FfiOHOSDeviceInfoMarketName | CString | marketName | 81 |
| 30 | FfiOHOSDeviceInfoDistributionOSName | CString | distributionOSName | 83 |
| 31 | FfiOHOSDeviceInfoDistributionOSVersion | CString | distributionOSVersion | 85 |
| 32 | FfiOHOSDeviceInfoDistributionOSApiVersion | Int64 | distributionOSApiVersion | 87 |
| 33 | FfiOHOSDeviceInfoDistributionOSReleaseType | CString | distributionOSReleaseType | 89 |
| 34 | FfiOHOSDeviceInfoDevOdid | CString | ODID | 91 |
| 35 | FfiOHOSDeviceInfoDistributionOSApiName | CString | distributionOSApiName | 93 |

### FFI 调用特点

#### 内存管理

| FFI 函数 | 返回类型 | 内存管理策略 | 备注 |
|---------|---------|-------------|------|
| FfiOHOSDeviceInfoUdid | CString | 动态分配，需 LibC.free() | 唯一需要手动释放的函数 |
| 其他 33 个 | CString/Int64 | 静态变量，无需 free | 直接使用 |

**证据**:
- `device_info.cj:549` - udid 的内存释放
- 其他属性无 LibC.free() 调用

#### 类型转换

| FFI 返回类型 | 仓颉类型 | 转换方式 | 证据 |
|------------|---------|---------|------|
| CString | String | cValue.toString() | 所有 String 属性 |
| Int64 | Int32 | Int32(ret) | 所有数值属性 |

**示例**:
```cangjie
// String 类型
let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
cValue.toString()

// Int32 类型
let ret = unsafe { FfiOHOSDeviceInfoMajorVersion() }
Int32(ret)
```

**证据**: `device_info.cj:115,368`

---

## 隐藏 API

### 概述

**类型**: internal static prop

**注解**: `@!Hide[isChecked: true]`

**数量**: 5 个

**说明**: 未实现或不稳定的 API，仅供内部使用

**证据**: `device_info.cj:198,672,682,692,703`

### 隐藏 API 清单

| API 名称 | 返回类型 | 实现状态 | 返回值 | 代码行 |
|---------|---------|---------|--------|--------|
| productModelAlias | String | 未实现 | "" (空字符串) | 198 |
| diskSN | String | 未实现 | "" (空字符串) | 672 |
| performanceClass | PerformanceClassLevel | 未实现 | ClassLevelHigh | 682 |
| chipType | String | 未实现 | "" (空字符串) | 692 |
| bootCount | Int32 | 未实现 | -1 | 703 |

### 隐藏 API 实现详情

#### productModelAlias

**位置**: `device_info.cj:198-203`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal static prop productModelAlias: String {
    get() {
        return ""
    }
}
```

**说明**: 产品型号别名，ArkTS 版本支持，仓颉版本暂未实现。

---

#### diskSN

**位置**: `device_info.cj:672-677`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal static prop diskSN: String {
    get() {
        return ""
    }
}
```

**说明**: 硬盘序列号，ArkTS 版本支持，仓颉版本暂未实现。

---

#### performanceClass

**位置**: `device_info.cj:682-687`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal static prop performanceClass: PerformanceClassLevel {
    get() {
        return PerformanceClassLevel.ClassLevelHigh
    }
}
```

**说明**: 设备能力等级，ArkTS 版本支持，仓颉版本暂未实现，默认返回 ClassLevelHigh。

---

#### chipType

**位置**: `device_info.cj:692-697`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal static prop chipType: String {
    get() {
        return ""
    }
}
```

**说明**: 设备 CPU 芯片类型，ArkTS 版本支持，仓颉版本暂未实现。

---

#### bootCount

**位置**: `device_info.cj:703-708`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal static prop bootCount: Int32 {
    get() {
        -1
    }
}
```

**说明**: 设备启动次数，ArkTS 版本支持，仓颉版本暂未实现，返回 -1 表示获取失败。

---

## 枚举类型

### PerformanceClassLevel

**可见性**: internal

**位置**: `device_info.cj:714-734`

**代码**:
```cangjie
@!Hide[isChecked: true]
internal enum PerformanceClassLevel {
    /**
     * Device Capability Level is high.
     */
    @!Hide[isChecked: true]
    ClassLevelHigh
    |
    /**
     * Device Capability Level is medium.
     */
    @!Hide[isChecked: true]
    ClassLevelMedium
    |
    /**
     * Device Capability Level is low.
     */
    @!Hide[isChecked: true]
    ClassLevelLow
    | ...
}
```

**说明**: 设备能力等级枚举，用于 `performanceClass` 属性（未实现）。

---

## 模块依赖关系

### 依赖方向图

```
ohos.device_info (startup_cangjie_wrapper)
  │
  ├─ external_deps
  │   └─ init:cj_device_info_ffi
  │       └─ FFI 函数（34 个）
  │
  └─ cj_external_deps
      └─ cangjie_ark_interop:ohos.labels
          ├─ APILevel 注解
          └─ Hide 注解
```

**证据**:
- `ohos/device_info/BUILD.gn:28-32`
- `ohos/device_info/device_info.cj:18-94`

### 依赖分析

#### init:cj_device_info_ffi

**类型**: external_deps

**用途**: 提供设备信息 SA 服务的 FFI 接口

**依赖关系**:
- `ohos.device_info` 依赖 `init:cj_device_info_ffi`
- `init:cj_device_info_ffi` 提供 34 个 FFI 函数
- FFI 函数通过 SA 服务机制调用底层实现

**证据**: `ohos/device_info/BUILD.gn:28`

#### cangjie_ark_interop:ohos.labels

**类型**: cj_external_deps

**用途**: 提供注解类

**导入的符号**:
- `APILevel` - API 级别注解
- `Hide` - 隐藏 API 注解

**使用场景**:
- `@APILevel[since: "22", syscap: "SystemCapability.Startup.SystemInfo"]`
- `@Hide[isChecked: true]`

**证据**:
- `ohos/device_info/BUILD.gn:30-32`
- `ohos/device_info/device_info.cj:20,99,198`

---

## 接口稳定性

### 稳定接口

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| public static prop | 稳定 | 32 个公开 API，API Level 22 |
| FFI 函数 | 稳定 | 依赖 init 组件，由 init 组件保证 |
| APILevel 注解 | 稳定 | API 级别管理机制 |

### 不稳定接口

| 类型 | 稳定性 | 说明 |
|------|--------|------|
| internal static prop | 不稳定 | 5 个隐藏 API，未实现 |
| internal enum | 不稳定 | PerformanceClassLevel，未实际使用 |
| Hide 注解 | 不稳定 | 隐藏机制可能变化 |

### 稳定性证据

**public API 稳定性**:
- 使用 `@APILevel` 注解明确标注 API Level
- API Level 22 表示稳定版本
- 无 `@Deprecated` 注解

**证据**: `device_info.cj:99,108,123...`

**internal API 不稳定性**:
- 使用 `@Hide` 注解标记
- 实现为空或返回默认值
- 注释说明"暂不支持"

**证据**: `device_info.cj:198,672,682,692,703`

---

## 可替换点

### 无可替换点

**说明**: 本项目为纯封装层，无内部逻辑，无可替换点。

**原因**:
- 所有公开 API 直接调用 FFI 函数
- 无业务逻辑、算法、配置等可替换组件
- 唯一的适配点是 mock 实现，通过编译配置切换

**证据**:
- 所有 getter 实现均非常简单（1-5 行）
- 无复杂逻辑或条件分支
- mock 实现通过 BUILD.gn 的 `if (is_mingw || is_mac)` 切换

---

## 错误传播

### 错误类型

| 错误类型 | 处理位置 | 传播方式 |
|---------|---------|---------|
| 权限拒绝 | 系统框架层 | 抛出异常（BusinessException） |
| FFI 调用失败 | FFI 层 | 未处理，可能导致崩溃 |
| 内存分配失败 | FFI 层 | 未处理，可能导致崩溃 |

### 异常处理策略

**应用层**: 无异常处理逻辑

**系统框架层**: 负责权限检查和异常抛出

**证据**: `device_info.cj` - 所有 getter 无 try-catch 或异常处理

---

## 关键结论

1. **纯封装层**: 内部 API 主要是 FFI 桥接，无业务逻辑
2. **依赖清晰**: 仅依赖 init 组件和 cangjie_ark_interop
3. **稳定接口**: 32 个 public API 标记为稳定（API Level 22）
4. **不稳定接口**: 5 个 internal API 标记为 @Hide，未实现
5. **无异常处理**: 封装层无异常处理，依赖系统框架层
6. **无可替换点**: 纯封装设计，无可替换组件
7. **单向依赖**: 仅依赖外部组件，无环形依赖

---

## 相关跳转链接

- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - 公开 API 清单
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链

---

## 参考资料

- [OpenHarmony FFI 规范](https://developer.openharmony.cn/cn/docs/documentation/guide)
- [仓颉语言注解](https://developer.openharmony.cn/cn/doc/cangjie-annotation)
- [cangjie_ark_interop 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)

---

*最后更新: 2026-02-06*

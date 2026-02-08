# 项目定位与边界

## 目的

本文档定义 startup_cangjie_wrapper 的项目定位、边界和核心能力，明确其职责范围。

## 适用范围

- 需要理解项目边界的开发者和维护者
- 准备集成或扩展设备信息能力的工程师

---

## 项目定位

startup_cangjie_wrapper 是 OpenHarmony **启动恢复子系统（startup）** 中的一个**仓颉语言封装组件**，负责为仓颉应用提供设备信息查询服务。

### 定位层级

```
OpenHarmony 系统
└── Startup（启动恢复子系统）
    └── startup_cangjie_wrapper（仓颉封装组件）
        └── device_info（设备信息模块）
```

### 核心价值

1. **语言桥接**: 将 C/C++ 实现的设备信息服务封装为仓颉 API
2. **统一接口**: 为仓颉开发者提供统一的设备信息查询接口
3. **权限隔离**: 在封装层实现权限检查和控制
4. **类型安全**: 利用仓颉的强类型系统提供类型安全的 API

---

## 项目边界

### 职责范围（在边界内）

| 职责 | 说明 |
|------|------|
| **设备信息查询** | 提供设备类型、品牌、型号、版本等信息 |
| **仓颉 API 暴露** | 定义仓颉语言级别的静态属性 API |
| **FFI 桥接** | 通过 FFI 调用 init 组件的设备信息服务 |
| **权限注解** | 使用 `@APILevel` 注解标注 API 级别和权限要求 |
| **API 级别管理** | 统一管理 API Level（22）和 SysCap |

### 不在职责范围内（越界）

| 职责 | 负责方 |
|------|--------|
| **设备信息存储** | init 组件（底层 SA 服务） |
| **设备信息采集** | init 组件（系统底层） |
| **权限实际校验** | OpenHarmony 系统框架层 |
| **其他仓颉 API 封装** | 其他子系统对应的仓颉封装组件 |
| **测试用例** | test/ 目录（不在本文档覆盖范围） |

---

## 核心能力

### 1. 设备信息查询

提供 32 个公开 API，涵盖以下维度：

| 维度 | API 数量 | 示例 |
|------|---------|------|
| 设备类型 | 6 | deviceType, brand, manufacture, productModel... |
| 系统版本 | 7 | majorVersion, seniorVersion, featureVersion... |
| 构建信息 | 7 | buildTime, buildUser, buildHost... |
| 设备标识 | 3 | udid, serial, ODID |
| Distribution OS | 5 | distributionOSName, distributionOSVersion... |
| 其他 | 4 | abiList, securityPatchTag, bootloaderVersion... |

**证据**: `ohos/device_info/device_info.cj:103-667`

### 2. 权限控制

通过注解和系统框架实现权限控制：

| 权限 | 使用场景 | API |
|------|---------|-----|
| `ohos.permission.sec.ACCESS_UDID` | 获取 UDID | `udid` (line 545) |
| `ohos.permission.sec.ACCESS_UDID` | 获取序列号 | `serial` (line 243) |

**限制**: 此权限只能由系统应用和企业定制应用申请。

**证据**: `README_zh.md:49`, `ohos/device_info/device_info.cj:240,542`

### 3. FFI 桥接

提供 34 个 FFI 函数，桥接到 init 组件的 SA 服务：

```
foreign func FfiOHOSDeviceInfo*() -> CString/Int64
```

**证据**: `ohos/device_info/device_info.cj:22-94`

### 4. 内存管理

自动处理 FFI 返回的 CString：

- **大部分属性**: FFI 返回静态 CString，无需手动释放
- **特例**: `udid` 属性需要手动调用 `LibC.free()`

**证据**:
```cangjie
// ohos/device_info/device_info.cj:547-550
let cValue = unsafe { FfiOHOSDeviceInfoUdid() }
let value = cValue.toString()
unsafe { LibC.free(cValue) }
return value
```

---

## 依赖关系

### 外部依赖

| 依赖组件 | 依赖项 | 用途 |
|---------|--------|------|
| **init 组件** | `cj_device_info_ffi` | 提供设备信息 SA 服务 |
| **cangjie_ark_interop** | `ohos.labels` | 提供 APILevel、Hide 等注解 |

**证据**:
- `bundle.json:24-26`
- `ohos/device_info/BUILD.gn:28-32`

### 被依赖关系

| 组件 | 依赖方式 |
|------|---------|
| **仓颉应用** | 导入 `ohos.device_info` 包 |
| **OpenHarmony SDK** | 作为 SDK 的一部分分发给开发者 |

---

## 与其他组件的对比

### vs ArkTS 版本的 device_info

| 维度 | ArkTS | 仓颉 | 说明 |
|------|-------|------|------|
| 实现语言 | TypeScript/ArkTS | Cangjie | 不同语言栈 |
| API 形式 | 对象方法 + 属性 | 静态属性 | 仓颉全部为静态属性 |
| 异步支持 | 部分异步 API | 全部同步 | 仓颉版本无异步 |
| 认证型号别名 | ✓ | ✗ | `productModelAlias` 未实现 |
| 硬盘序列号 | ✓ | ✗ | `diskSN` 未实现 |
| 设备能力等级 | ✓ | ✗ | `performanceClass` 未实现 |
| API Level | 12+ | 22 | 仓颉版本 API Level 更高 |

**证据**: `README_zh.md:51-55`, `ohos/device_info/device_info.cj:198,672,682,692,703`

### vs 其他仓颉封装组件

| 组件 | 子系统 | 职责 |
|------|--------|------|
| **startup_cangjie_wrapper** | startup | 设备信息查询 |
| **其他仓颉封装** | 不同子系统 | 各子系统对应 API |

---

## 关键概念

### SysCap（系统能力）

- **定义**: `SystemCapability.Startup.SystemInfo`
- **含义**: 启动系统信息能力
- **作用**: 用于声明组件依赖的系统能力

**证据**: 所有 API 均标注 `syscap: "SystemCapability.Startup.SystemInfo"`

### API Level

- **当前版本**: 22
- **含义**: API 稳定性级别
- **作用**: 用于 API 兼容性管理

**证据**: 所有公开 API 均标注 `@APILevel since: "22"`

### @Hide 注解

- **用途**: 标记未实现或内部 API
- **参数**: `isChecked: true`
- **示例**: `@!Hide[isChecked: true]`

**证据**: `ohos/device_info/device_info.cj:198,672,682,692,703`

---

## 关键结论

1. **职责明确**: 仅负责设备信息查询的仓颉封装，不涉及设备信息采集和存储
2. **依赖清晰**: 依赖 init 组件和 cangjie_ark_interop
3. **边界清晰**: 设备信息采集在 init 组件，权限校验在系统框架层
4. **类型安全**: 所有 API 均为强类型，返回 String 或 Int32
5. **同步设计**: 无异步 API，简化使用
6. **Beta 特性**: 当前版本为 beta，API 可能变化

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单
- [05_Internal_API.md](05_Internal_API.md) - 内部 API

---

## 参考资料

- [OpenHarmony 设备信息 API（ArkTS）](https://docs.openharmony.cn/application-dev/reference/apis/js-apis-device-info.md)
- [OpenHarmony 系统能力定义](https://docs.openharmony.cn/application-dev/compatibility/syscap-overview)
- [OpenHarmony 权限开发指南](https://docs.openharmony.cn/application-dev/security/permission-list)

---

*最后更新: 2026-02-06*

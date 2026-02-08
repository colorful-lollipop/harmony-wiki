# 仓颉 API 清单

## 目的

本文档列出 startup_cangjie_wrapper 提供的所有仓颉（Cangjie）API，包括 API 清单、调用链和参数说明。

## 适用范围

- 使用仓颉语言开发 OpenHarmony 应用的开发者
- 需要了解 API 细节的工程师

---

## API 概览

### API 统计

| 类型 | 数量 |
|------|------|
| 公开 API（public） | 32 |
| 隐藏 API（internal） | 5 |
| 总计 | 37 |

### API 分类

| 分类 | 数量 | 示例 |
|------|------|------|
| 设备类型信息 | 6 | deviceType, brand, manufacture |
| 系统版本信息 | 7 | majorVersion, sdkApiVersion, osFullName |
| 构建信息 | 7 | buildTime, buildUser, buildRootHash |
| 设备标识 | 3 | udid, serial, ODID |
| Distribution OS | 5 | distributionOSName, distributionOSVersion |
| 其他 | 4 | abiList, securityPatchTag, bootloaderVersion |

---

## API 清单表

### 设备类型信息（6 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| deviceType | String | 无 | FfiOHOSDeviceInfoDeviceType() | device_info.cj:112 |
| brand | String | 无 | FfiOHOSDeviceInfoBrand() | device_info.cj:142 |
| manufacture | String | 无 | FfiOHOSDeviceInfoManufacture() | device_info.cj:127 |
| marketName | String | 无 | FfiOHOSDeviceInfoMarketName() | device_info.cj:157 |
| productSeries | String | 无 | FfiOHOSDeviceInfoProductSeries() | device_info.cj:172 |
| productModel | String | 无 | FfiOHOSDeviceInfoProductModel() | device_info.cj:187 |

#### 详细说明

##### deviceType

**类型**: `String`

**权限**: 无

**FFI 函数**: `FfiOHOSDeviceInfoDeviceType()`

**说明**: 获取设备类型，返回值可能是 phone、tablet、tv、wearable、car、smartVision 等。

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**调用链**:
```
DeviceInfo.deviceType
  → device_info.cj:115 { FfiOHOSDeviceInfoDeviceType() }
  → init SA 服务
  → 返回: "phone"
```

**代码**:
```cangjie
public static prop deviceType: String {
    get() {
        let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
        cValue.toString()
    }
}
```

**证据**: `device_info.cj:112-118`

---

##### brand

**类型**: `String`

**权限**: 无

**FFI 函数**: `FfiOHOSDeviceInfoBrand()`

**说明**: 获取设备品牌。

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**调用链**:
```
DeviceInfo.brand
  → device_info.cj:145 { FfiOHOSDeviceInfoBrand() }
  → init SA 服务
  → 返回: "HUAWEI"
```

**证据**: `device_info.cj:142-148`

---

### 系统版本信息（7 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| osFullName | String | 无 | FfiOHOSDeviceInfoOsFullName() | device_info.cj:350 |
| majorVersion | Int32 | 无 | FfiOHOSDeviceInfoMajorVersion() | device_info.cj:366 |
| seniorVersion | Int32 | 无 | FfiOHOSDeviceInfoSeniorVersion() | device_info.cj:382 |
| featureVersion | Int32 | 无 | FfiOHOSDeviceInfoFeatureVersion() | device_info.cj:397 |
| buildVersion | Int32 | 无 | FfiOHOSDeviceInfoBuildVersion() | device_info.cj:412 |
| sdkApiVersion | Int32 | 无 | FfiOHOSDeviceInfoSdkApiVersion() | device_info.cj:426 |
| displayVersion | String | 无 | FfiOHOSDeviceInfoDisplayVersion() | device_info.cj:303 |

#### 详细说明

##### majorVersion

**类型**: `Int32`

**权限**: 无

**FFI 函数**: `FfiOHOSDeviceInfoMajorVersion()` (返回 Int64，转换为 Int32)

**说明**: 获取主版本号（M版本号），范围 1-99。

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**调用链**:
```
DeviceInfo.majorVersion
  → device_info.cj:368 { FfiOHOSDeviceInfoMajorVersion() }
  → init SA 服务 (返回 Int64)
  → Int32(ret) 转换
  → 返回: 5
```

**代码**:
```cangjie
public static prop majorVersion: Int32 {
    get() {
        let ret = unsafe { FfiOHOSDeviceInfoMajorVersion() }
        Int32(ret)
    }
}
```

**证据**: `device_info.cj:366-371`

---

### 构建信息（7 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| buildTime | String | 无 | FfiOHOSDeviceInfoBuildTime() | device_info.cj:514 |
| buildUser | String | 无 | FfiOHOSDeviceInfoBuildUser() | device_info.cj:484 |
| buildHost | String | 无 | FfiOHOSDeviceInfoBuildHost() | device_info.cj:499 |
| buildType | String | 无 | FfiOHOSDeviceInfoBuildType() | device_info.cj:469 |
| buildRootHash | String | 无 | FfiOHOSDeviceInfoBuildRootHash() | device_info.cj:529 |
| incrementalVersion | String | 无 | FfiOHOSDeviceInfoIncrementalVersion() | device_info.cj:318 |
| versionId | String | 无 | FfiOHOSDeviceInfoVersionId() | device_info.cj:454 |

---

### 设备标识（3 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| udid | String | ACCESS_UDID | FfiOHOSDeviceInfoUdid() | device_info.cj:545 |
| serial | String | ACCESS_UDID | FfiOHOSDeviceInfoSerial() | device_info.cj:243 |
| ODID | String | 无 | FfiOHOSDeviceInfoDevOdid() | device_info.cj:661 |

#### 详细说明

##### udid

**类型**: `String`

**权限**: `ohos.permission.sec.ACCESS_UDID`（仅系统应用和企业定制应用）

**FFI 函数**: `FfiOHOSDeviceInfoUdid()`

**说明**: 获取设备 UDID（唯一设备标识符）。

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**特殊处理**: 需要手动释放 FFI 返回的动态分配内存。

**调用链**:
```
DeviceInfo.udid
  → device_info.cj:547 { FfiOHOSDeviceInfoUdid() }
  → init SA 服务 (返回动态分配的 CString)
  → cValue.toString() 转换
  → unsafe { LibC.free(cValue) } 释放内存
  → 返回: "dff3cdfd-7beb-1e7d-fdf7-1dbfddd7d30c"
```

**代码**:
```cangjie
public static prop udid: String {
    get() {
        let cValue = unsafe { FfiOHOSDeviceInfoUdid() }
        let value = cValue.toString()
        unsafe { LibC.free(cValue) }
        return value
    }
}
```

**证据**: `device_info.cj:540-552`

---

### Distribution OS 信息（5 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| distributionOSName | String | 无 | FfiOHOSDeviceInfoDistributionOSName() | device_info.cj:564 |
| distributionOSVersion | String | 无 | FfiOHOSDeviceInfoDistributionOSVersion() | device_info.cj:582 |
| distributionOSApiVersion | Int32 | 无 | FfiOHOSDeviceInfoDistributionOSApiVersion() | device_info.cj:600 |
| distributionOSApiName | String | 无 | FfiOHOSDeviceInfoDistributionOSApiName() | device_info.cj:616 |
| distributionOSReleaseType | String | 无 | FfiOHOSDeviceInfoDistributionOSReleaseType() | device_info.cj:634 |

---

### 其他信息（4 个）

| API 名称 | 返回类型 | 权限要求 | FFI 函数 | 代码位置 |
|---------|---------|---------|---------|---------|
| abiList | String | 无 | FfiOHOSDeviceInfoAbiList() | device_info.cj:273 |
| securityPatchTag | String | 无 | FfiOHOSDeviceInfoSecurityPatchTag() | device_info.cj:288 |
| bootloaderVersion | String | 无 | FfiOHOSDeviceInfoBootloaderVersion() | device_info.cj:258 |
| osReleaseType | String | 无 | FfiOHOSDeviceInfoOsReleaseType() | device_info.cj:335 |

---

### 隐藏 API（internal，5 个）

| API 名称 | 返回类型 | 注解 | 说明 | 代码位置 |
|---------|---------|------|------|---------|
| productModelAlias | String | @Hide | 未实现，返回空字符串 | device_info.cj:199 |
| diskSN | String | @Hide | 未实现，返回空字符串 | device_info.cj:673 |
| performanceClass | PerformanceClassLevel | @Hide | 未实现，返回 ClassLevelHigh | device_info.cj:683 |
| chipType | String | @Hide | 未实现，返回空字符串 | device_info.cj:693 |
| bootCount | Int32 | @Hide | 未实现，返回 -1 | device_info.cj:704 |

**说明**: 这些 API 标记为 `@Hide[isChecked: true]`，表示未实现或不稳定，仅供内部使用。

**证据**: `device_info.cj:198,672,682,692,703`

---

## 完整 API 映射表

| 序号 | API 名称 | 返回类型 | 权限 | FFI 函数 | 代码行 | 同步/异步 |
|------|---------|---------|------|---------|--------|-----------|
| 1 | deviceType | String | 无 | FfiOHOSDeviceInfoDeviceType | 112 | 同步 |
| 2 | manufacture | String | 无 | FfiOHOSDeviceInfoManufacture | 127 | 同步 |
| 3 | brand | String | 无 | FfiOHOSDeviceInfoBrand | 142 | 同步 |
| 4 | marketName | String | 无 | FfiOHOSDeviceInfoMarketName | 157 | 同步 |
| 5 | productSeries | String | 无 | FfiOHOSDeviceInfoProductSeries | 172 | 同步 |
| 6 | productModel | String | 无 | FfiOHOSDeviceInfoProductModel | 187 | 同步 |
| 7 | softwareModel | String | 无 | FfiOHOSDeviceInfoSoftwareModel | 212 | 同步 |
| 8 | hardwareModel | String | 无 | FfiOHOSDeviceInfoHardwareModel | 227 | 同步 |
| 9 | serial | String | ACCESS_UDID | FfiOHOSDeviceInfoSerial | 243 | 同步 |
| 10 | bootloaderVersion | String | 无 | FfiOHOSDeviceInfoBootloaderVersion | 258 | 同步 |
| 11 | abiList | String | 无 | FfiOHOSDeviceInfoAbiList | 273 | 同步 |
| 12 | securityPatchTag | String | 无 | FfiOHOSDeviceInfoSecurityPatchTag | 288 | 同步 |
| 13 | displayVersion | String | 无 | FfiOHOSDeviceInfoDisplayVersion | 303 | 同步 |
| 14 | incrementalVersion | String | 无 | FfiOHOSDeviceInfoIncrementalVersion | 318 | 同步 |
| 15 | osReleaseType | String | 无 | FfiOHOSDeviceInfoOsReleaseType | 335 | 同步 |
| 16 | osFullName | String | 无 | FfiOHOSDeviceInfoOsFullName | 350 | 同步 |
| 17 | majorVersion | Int32 | 无 | FfiOHOSDeviceInfoMajorVersion | 366 | 同步 |
| 18 | seniorVersion | Int32 | 无 | FfiOHOSDeviceInfoSeniorVersion | 382 | 同步 |
| 19 | featureVersion | Int32 | 无 | FfiOHOSDeviceInfoFeatureVersion | 397 | 同步 |
| 20 | buildVersion | Int32 | 无 | FfiOHOSDeviceInfoBuildVersion | 412 | 同步 |
| 21 | sdkApiVersion | Int32 | 无 | FfiOHOSDeviceInfoSdkApiVersion | 426 | 同步 |
| 22 | firstApiVersion | Int32 | 无 | FfiOHOSDeviceInfoFirstApiVersion | 440 | 同步 |
| 23 | versionId | String | 无 | FfiOHOSDeviceInfoVersionId | 454 | 同步 |
| 24 | buildType | String | 无 | FfiOHOSDeviceInfoBuildType | 469 | 同步 |
| 25 | buildUser | String | 无 | FfiOHOSDeviceInfoBuildUser | 484 | 同步 |
| 26 | buildHost | String | 无 | FfiOHOSDeviceInfoBuildHost | 499 | 同步 |
| 27 | buildTime | String | 无 | FfiOHOSDeviceInfoBuildTime | 514 | 同步 |
| 28 | buildRootHash | String | 无 | FfiOHOSDeviceInfoBuildRootHash | 529 | 同步 |
| 29 | udid | String | ACCESS_UDID | FfiOHOSDeviceInfoUdid | 545 | 同步 |
| 30 | distributionOSName | String | 无 | FfiOHOSDeviceInfoDistributionOSName | 564 | 同步 |
| 31 | distributionOSVersion | String | 无 | FfiOHOSDeviceInfoDistributionOSVersion | 582 | 同步 |
| 32 | distributionOSApiVersion | Int32 | 无 | FfiOHOSDeviceInfoDistributionOSApiVersion | 600 | 同步 |
| 33 | distributionOSApiName | String | 无 | FfiOHOSDeviceInfoDistributionOSApiName | 616 | 同步 |
| 34 | distributionOSReleaseType | String | 无 | FfiOHOSDeviceInfoDistributionOSReleaseType | 634 | 同步 |
| 35 | ODID | String | 无 | FfiOHOSDeviceInfoDevOdid | 661 | 同步 |
| 36 | productModelAlias | String | @Hide | 无 FFI | 199 | 同步 |
| 37 | diskSN | String | @Hide | 无 FFI | 673 | 同步 |
| 38 | performanceClass | PerformanceClassLevel | @Hide | 无 FFI | 683 | 同步 |
| 39 | chipType | String | @Hide | 无 FFI | 693 | 同步 |
| 40 | bootCount | Int32 | @Hide | 无 FFI | 704 | 同步 |

---

## 参数校验与错误码

### 参数校验

**特点**: 所有 API 均为属性（property），无参数传递，无需参数校验。

### 异常处理

**机制**: 权限检查失败时，由系统框架层抛出异常。

**权限相关异常**:
- `ohos.permission.sec.ACCESS_UDID` 权限未声明或未授予时，调用 `udid` 或 `serial` 会抛出异常。

**异常类型**: 由 `cangjie_ark_interop` 提供的 `BusinessException` 类（详见 cangjie_ark_interop 文档）。

---

## 关键结论

1. **全同步**: 所有 37 个 API 均为同步属性访问
2. **无参数**: 所有 API 为属性，无需参数
3. **强类型**: 返回 String 或 Int32，类型安全
4. **权限控制**: udid 和 serial 需要 ACCESS_UDID 权限
5. **内存管理**: 仅 udid 需要手动释放内存
6. **隐藏 API**: 5 个 API 标记为 @Hide，未实现
7. **API Level**: 所有公开 API 标注为 API Level 22
8. **SysCap**: 所有公开 API 标注为 SystemCapability.Startup.SystemInfo

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [appendix/API_Reference.md](appendix/API_Reference.md) - API 完整参考
- [仓颉设备信息 API 官方文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-device_info.md)

---

## 参考资料

- [OpenHarmony 权限列表](https://docs.openharmony.cn/application-dev/security/permission-list)
- [仓颉语言属性语法](https://developer.openharmony.cn/cn/doc/cangjie-property)

---

*最后更新: 2026-02-06*

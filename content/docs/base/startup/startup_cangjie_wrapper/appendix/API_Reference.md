# API 完整参考

## 目的

本文档提供 startup_cangjie_wrapper 所有 API 的完整参考，包括签名、说明、参数和返回值。

## 适用范围

- 需要详细 API 信息的开发者
- 进行 API 集成的工程师

---

## API 快速索引

### 设备类型信息
- [deviceType](#devicetype)
- [brand](#brand)
- [manufacture](#manufacture)
- [marketName](#marketname)
- [productSeries](#productseries)
- [productModel](#productmodel)
- [softwareModel](#softwaremodel)
- [hardwareModel](#hardwaremodel)

### 系统版本信息
- [osFullName](#osfullname)
- [majorVersion](#majorversion)
- [seniorVersion](#seniorversion)
- [featureVersion](#featureversion)
- [buildVersion](#buildversion)
- [sdkApiVersion](#sdkapiversion)
- [firstApiVersion](#firstapiversion)
- [displayVersion](#displayversion)
- [osReleaseType](#osreleasetype)

### 构建信息
- [buildTime](#buildtime)
- [buildUser](#builduser)
- [buildHost](#buildhost)
- [buildType](#buildtype)
- [buildRootHash](#buildroothash)
- [incrementalVersion](#incrementalversion)
- [versionId](#versionid)

### 设备标识
- [udid](#udid)
- [serial](#serial)
- [ODID](#odid)

### Distribution OS 信息
- [distributionOSName](#distributionosname)
- [distributionOSVersion](#distributionosversion)
- [distributionOSApiVersion](#distributionosapiversion)
- [distributionOSApiName](#distributionosapiname)
- [distributionOSReleaseType](#distributionosreleasetype)

### 其他信息
- [abiList](#abilist)
- [securityPatchTag](#securitypatchtag)
- [bootloaderVersion](#bootloaderversion)

---

## 设备类型信息

### deviceType

**签名**:
```cangjie
public static prop deviceType: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取设备类型，返回值可能是 phone、tablet、tv、wearable、car、smartVision 等。

**返回值示例**: `"phone"`, `"tablet"`, `"tv"`

**代码位置**: `device_info.cj:112`

**FFI 函数**: `FfiOHOSDeviceInfoDeviceType()`

**同步/异步**: 同步

**示例**:
```cangjie
let deviceType = DeviceInfo.deviceType
println("Device Type: ${deviceType}")  // 输出: Device Type: phone
```

---

### brand

**签名**:
```cangjie
public static prop brand: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取设备品牌。

**返回值示例**: `"HUAWEI"`, `"XIAOMI"`

**代码位置**: `device_info.cj:142`

**FFI 函数**: `FfiOHOSDeviceInfoBrand()`

**同步/异步**: 同步

**示例**:
```cangjie
let brand = DeviceInfo.brand
println("Brand: ${brand}")  // 输出: Brand: HUAWEI
```

---

### manufacture

**签名**:
```cangjie
public static prop manufacture: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取设备制造商。

**返回值示例**: `"Huawei Device Co., Ltd."`

**代码位置**: `device_info.cj:127`

**FFI 函数**: `FfiOHOSDeviceInfoManufacture()`

**同步/异步**: 同步

**示例**:
```cangjie
let manufacture = DeviceInfo.manufacture
println("Manufacture: ${manufacture}")
```

---

### marketName

**签名**:
```cangjie
public static prop marketName: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取外部产品系列。

**代码位置**: `device_info.cj:157`

**FFI 函数**: `FfiOHOSDeviceInfoMarketName()`

**同步/异步**: 同步

---

### productSeries

**签名**:
```cangjie
public static prop productSeries: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取产品系列。

**代码位置**: `device_info.cj:172`

**FFI 函数**: `FfiOHOSDeviceInfoProductSeries()`

**同步/异步**: 同步

---

### productModel

**签名**:
```cangjie
public static prop productModel: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取产品型号。

**代码位置**: `device_info.cj:187`

**FFI 函数**: `FfiOHOSDeviceInfoProductModel()`

**同步/异步**: 同步

---

### softwareModel

**签名**:
```cangjie
public static prop softwareModel: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取软件型号。

**代码位置**: `device_info.cj:212`

**FFI 函数**: `FfiOHOSDeviceInfoSoftwareModel()`

**同步/异步**: 同步

---

### hardwareModel

**签名**:
```cangjie
public static prop hardwareModel: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取硬件型号。

**代码位置**: `device_info.cj:227`

**FFI 函数**: `FfiOHOSDeviceInfoHardwareModel()`

**同步/异步**: 同步

---

## 系统版本信息

### osFullName

**签名**:
```cangjie
public static prop osFullName: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取 OS 版本完整字符串。

**返回值示例**: `"OpenHarmony 5.0.0"`

**代码位置**: `device_info.cj:350`

**FFI 函数**: `FfiOHOSDeviceInfoOsFullName()`

**同步/异步**: 同步

**示例**:
```cangjie
let osFullName = DeviceInfo.osFullName
println("OS: ${osFullName}")  // 输出: OS: OpenHarmony 5.0.0
```

---

### majorVersion

**签名**:
```cangjie
public static prop majorVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取主版本号（M版本号），范围 1-99，随整体架构更新而增加。

**返回值示例**: `5`

**代码位置**: `device_info.cj:366`

**FFI 函数**: `FfiOHOSDeviceInfoMajorVersion()` (返回 Int64，转换为 Int32)

**同步/异步**: 同步

**示例**:
```cangjie
let majorVersion = DeviceInfo.majorVersion
println("Major Version: ${majorVersion}")  // 输出: Major Version: 5
```

---

### seniorVersion

**签名**:
```cangjie
public static prop seniorVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取次版本号（S版本号），范围 0-99，随部分架构或主要功能更新而增加。

**代码位置**: `device_info.cj:382`

**FFI 函数**: `FfiOHOSDeviceInfoSeniorVersion()`

**同步/异步**: 同步

---

### featureVersion

**签名**:
```cangjie
public static prop featureVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取特性版本号（F版本号），范围 0-99，随计划的新功能增加而增加。

**代码位置**: `device_info.cj:397`

**FFI 函数**: `FfiOHOSDeviceInfoFeatureVersion()`

**同步/异步**: 同步

---

### buildVersion

**签名**:
```cangjie
public static prop buildVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取构建版本号（B版本号），范围 0-999，随每次新开发构建而增加。

**代码位置**: `device_info.cj:412`

**FFI 函数**: `FfiOHOSDeviceInfoBuildVersion()`

**同步/异步**: 同步

---

### sdkApiVersion

**签名**:
```cangjie
public static prop sdkApiVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取 SDK API 版本号。

**代码位置**: `device_info.cj:426`

**FFI 函数**: `FfiOHOSDeviceInfoSdkApiVersion()`

**同步/异步**: 同步

---

### firstApiVersion

**签名**:
```cangjie
public static prop firstApiVersion: Int32
```

**返回类型**: `Int32`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取首个 API 版本号。

**代码位置**: `device_info.cj:440`

**FFI 函数**: `FfiOHOSDeviceInfoFirstApiVersion()`

**同步/异步**: 同步

---

### displayVersion

**签名**:
```cangjie
public static prop displayVersion: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取显示版本号。

**返回值示例**: `"5.0.0"`

**代码位置**: `device_info.cj:303`

**FFI 函数**: `FfiOHOSDeviceInfoDisplayVersion()`

**同步/异步**: 同步

---

### osReleaseType

**签名**:
```cangjie
public static prop osReleaseType: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取 OS 发布类型，类别可以是 Release、Beta 或 Canary，具体发布类型可能是 Release、Beta1 等。

**返回值示例**: `"Release"`, `"Beta"`, `"Canary"`

**代码位置**: `device_info.cj:335`

**FFI 函数**: `FfiOHOSDeviceInfoOsReleaseType()`

**同步/异步**: 同步

---

## 设备标识

### udid

**签名**:
```cangjie
public static prop udid: String
```

**返回类型**: `String`

**权限**: `ohos.permission.sec.ACCESS_UDID`（仅系统应用和企业定制应用）

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取设备 UDID（唯一设备标识符）。

**返回值示例**: `"dff3cdfd-7beb-1e7d-fdf7-1dbfddd7d30c"`

**代码位置**: `device_info.cj:545`

**FFI 函数**: `FfiOHOSDeviceInfoUdid()`

**同步/异步**: 同步

**内存管理**: 需要手动调用 `LibC.free()`

**示例**:
```cangjie
let udid = DeviceInfo.udid
println("UDID: ${udid}")
```

**注意**: 需要声明 `ohos.permission.sec.ACCESS_UDID` 权限。

---

### serial

**签名**:
```cangjie
public static prop serial: String
```

**返回类型**: `String`

**权限**: `ohos.permission.sec.ACCESS_UDID`（仅系统应用和企业定制应用）

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取设备序列号。

**代码位置**: `device_info.cj:243`

**FFI 函数**: `FfiOHOSDeviceInfoSerial()`

**同步/异步**: 同步

---

### ODID

**签名**:
```cangjie
public static prop ODID: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取开放设备标识符（ODID），这是一个开发者级的非永久设备标识符。同一个开发者的应用在同一设备上运行时，具有相同的 ODID。

**返回值示例**: `"dff3cdfd-7beb-1e7d-fdf7-1dbfddd7d30c"`

**ODID 重新生成场景**:
- 恢复出厂设置
- 卸载并重装同一开发者的所有应用

**ODID 生成规则**:
- 同一开发者的应用在同一设备上：相同 ODID
- 不同开发者的应用在同一设备上：各自有不同的 ODID
- 同一开发者的应用在不同设备上：各自有不同的 ODID
- 不同开发者的应用在不同设备上：各自有不同的 ODID

**代码位置**: `device_info.cj:661`

**FFI 函数**: `FfiOHOSDeviceInfoDevOdid()`

**同步/异步**: 同步

---

## 其他信息

### abiList

**签名**:
```cangjie
public static prop abiList: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取应用二进制接口（ABI）列表。

**返回值示例**: `"arm64-v8a"`

**代码位置**: `device_info.cj:273`

**FFI 函数**: `FfiOHOSDeviceInfoAbiList()`

**同步/异步**: 同步

---

### securityPatchTag

**签名**:
```cangjie
public static prop securityPatchTag: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取安全补丁级别。

**返回值示例**: `"2024-12-01"`

**代码位置**: `device_info.cj:288`

**FFI 函数**: `FfiOHOSDeviceInfoSecurityPatchTag()`

**同步/异步**: 同步

---

### bootloaderVersion

**签名**:
```cangjie
public static prop bootloaderVersion: String
```

**返回类型**: `String`

**权限**: 无

**API Level**: 22

**SysCap**: SystemCapability.Startup.SystemInfo

**说明**: 获取 Bootloader 版本号。

**代码位置**: `device_info.cj:258`

**FFI 函数**: `FfiOHOSDeviceInfoBootloaderVersion()`

**同步/异步**: 同步

---

## 隐藏 API

以下 API 标记为 `@Hide[isChecked: true]`，未实现或不稳定：

### productModelAlias
- **状态**: 未实现，返回空字符串
- **代码位置**: `device_info.cj:198`

### diskSN
- **状态**: 未实现，返回空字符串
- **代码位置**: `device_info.cj:672`

### performanceClass
- **状态**: 未实现，返回 `ClassLevelHigh`
- **代码位置**: `device_info.cj:682`

### chipType
- **状态**: 未实现，返回空字符串
- **代码位置**: `device_info.cj:692`

### bootCount
- **状态**: 未实现，返回 -1
- **代码位置**: `device_info.cj:704`

---

## 常见用法示例

### 基本使用

```cangjie
package main

import ohos.device_info

func main() {
    // 获取设备类型
    let deviceType = DeviceInfo.deviceType
    println("Device Type: ${deviceType}")

    // 获取 OS 版本
    let osFullName = DeviceInfo.osFullName
    println("OS: ${osFullName}")

    // 获取主版本号
    let majorVersion = DeviceInfo.majorVersion
    println("Major Version: ${majorVersion}")
}
```

### 获取 UDID（需要权限）

```cangjie
package main

import ohos.device_info

func main() {
    // 需要声明 ohos.permission.sec.ACCESS_UDID 权限
    let udid = DeviceInfo.udid
    println("UDID: ${udid}")
}
```

---

## 相关跳转链接

- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单表
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链
- [仓颉设备信息 API 官方文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-device_info.md)

---

*最后更新: 2026-02-06*

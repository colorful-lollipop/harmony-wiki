# 配置开关

> bundlemanager_cangjie_wrapper 关键宏定义与 Feature Flags

## 注解配置

### @APILevel 注解参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `since` | String | 是 | API 引入的 API Level 版本 |
| `syscap` | String | 是 | 系统能力标识 |
| `throwexception` | Bool | 否 | 是否支持抛出异常（默认 false） |
| `workerthread` | Bool | 否 | 是否支持 Worker 线程执行（默认 false） |

**使用示例** (`bundle_manager.cj:63-80`):

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.BundleManager.BundleFramework.Core"
]
public class BundleManager {
    @!APILevel[
        since: "22",
        syscap: "SystemCapability.BundleManager.BundleFramework.Core",
        workerthread: true
    ]
    public static func getBundleInfoForSelf(bundleFlags: Int32): BundleInfo {
        // ...
    }
}
```

---

## @Hide 注解参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `isChecked` | Bool | 是 | 是否已检查确认可隐藏 |

**使用示例** (`element_name.cj:97-103`):

```cj
@!Hide[isChecked: true]
internal var uri: Option<String> = Option.None

@!Hide[isChecked: true]
internal var shortName: Option<String> = Option.None
```

---

## 条件编译

### 平台判断

| 条件 | 说明 | 影响模块 |
|------|------|----------|
| `is_mingw` | Windows MinGW 环境 | 所有子模块 |
| `is_mac` | macOS 环境 | 所有子模块 |

**使用示例** (`ohos/bundle/BUILD.gn:21-25`):

```gn
if (is_mingw || is_mac){
    sources = ["../../mock/ohos.bundle.cj"]
} else {
    sources = ["bundle.cj"]
}
```

---

## BundleFlag 枚举值

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `GET_BUNDLE_INFO_WITH_APPLICATION` | 0x00000000 | 获取应用信息 |
| `GET_BUNDLE_INFO_WITH_HAP_MODULE` | 0x00000001 | 获取 HAP 模块信息 |
| `GET_BUNDLE_INFO_WITH_ABILITY` | 0x00000002 | 获取 Ability 信息 |
| `GET_BUNDLE_INFO_WITH_PERMISSION` | 0x00000004 | 获取权限信息 |
| `GET_BUNDLE_INFO_WITH_METADATA` | 0x00000008 | 获取元数据 |
| `GET_BUNDLE_INFO_ALL` | 0xFFFFFFFF | 获取全部信息 |

**使用示例** (`bundle_manager.cj:84-86`):

```cj
let query = BMQuery(bundleName, GET_BUNDLE_INFO, bundleFlags, userId: userId)
```

---

## 错误码常量

### Bundle 错误 (17700001 - 17700060)

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 17700001 | ERROR_BUNDLE_NOT_EXIST | Bundle 不存在 |
| 17700002 | ERROR_MODULE_NOT_EXIST | 模块不存在 |
| 17700003 | ERROR_ABILITY_NOT_EXIST | Ability 不存在 |
| 17700004 | ERROR_INVALID_USER_ID | 无效用户 ID |
| 17700024 | ERROR_PROFILE_NOT_EXIST | Profile 不存在 |
| 17700029 | ERROR_ABILITY_IS_DISABLED | Ability 已禁用 |
| 17700055 | ERROR_INVALID_LINK | 无效链接 |
| 17700056 | ERROR_SCHEME_NOT_IN_QUERYSCHEMES | scheme 不在查询列表 |

### 服务异常 (17700101+)

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 17700101 | ERROR_BUNDLE_SERVICE_EXCEPTION | Bundle 服务异常 |

**使用示例** (`error_code.cj:20-74`):

```cj
const ERROR_BUNDLE_NOT_EXIST: Int32 = 17700001
const ERROR_MODULE_NOT_EXIST: Int32 = 17700002
const ERROR_ABILITY_NOT_EXIST: Int32 = 17700003
```

---

## 构建配置常量

### 子系统/部件标识

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `subsystem_name` | `"bundlemanager"` | 子系统名称 |
| `part_name` | `"bundlemanager_cangjie_wrapper"` | 部件名称 |

**使用示例** (`ohos/bundle/BUILD.gn:27-28`):

```gn
subsystem_name = "bundlemanager"
part_name = "bundlemanager_cangjie_wrapper"
```

---

## 系统能力要求

### API Level 要求

| 组件 | 最低 API Level | 说明 |
|------|---------------|------|
| BundleManager | 22 | 全部 API |
| ElementName | 22 | 全部属性 |
| Metadata | 22 | 全部属性 |
| Skill | 22 | 全部属性 |

### System Capability

| Capability | 用途 |
|-------------|------|
| `SystemCapability.BundleManager.BundleFramework.Core` | Bundle 框架核心能力 |

**使用示例**:

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.BundleManager.BundleFramework.Core"
]
```

---

## 常量定义

### 缓存相关

| 常量 | 值 | 说明 |
|------|-----|------|
| `BASE_USER_RANGE` | 200000 | 用户 ID 基础范围 |

**使用示例** (`bundle_manager.cj:84`):

```cj
let userId = uid / BASE_USER_RANGE
```

### 成功码

| 常量 | 值 | 说明 |
|------|-----|------|
| `SUCCESS_CODE` | 0 | 操作成功 |

**使用示例** (`bundle_manager.cj:157`):

```cj
if (code != SUCCESS_CODE) {
    throw BusinessException(code, getErrorMsg(code))
}
```

---

## 相关文档

- [02_API_Reference.md](../../02_API_Reference.md) - API 参考
- [04_Build.md](../../04_Build.md) - 构建配置
- [SUMMARY.md](../../SUMMARY.md) - 文档导航

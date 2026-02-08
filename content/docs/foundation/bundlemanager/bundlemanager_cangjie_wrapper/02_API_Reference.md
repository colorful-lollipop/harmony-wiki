# API 参考

> bundlemanager_cangjie_wrapper 对外 N-API 接口完整清单

## API 清单

### BundleManager

**命名空间**: `ohos.bundle.bundle_manager`
**类**: `BundleManager`
**文件**: `ohos/bundle/bundle_manager/bundle_manager.cj`

| JS API | C/C++ 入口 | 参数 | 返回值 | 同步/异步 | 异常 |
|--------|-----------|------|--------|-----------|------|
| `getBundleInfoForSelf(bundleFlags)` | `BundleManager.getBundleInfoForSelf` | Int32 | BundleInfo | 同步（workerthread） | - |
| `getProfileByAbility(moduleName, abilityName, metadataName?)` | `BundleManager.getProfileByAbility` | String, String, String? | Array<String> | 异步（workerthread） | BusinessException |
| `canOpenLink(link)` | `BundleManager.canOpenLink` | String | Bool | 同步 | BusinessException |

**参数校验**:
- `bundleFlags`: Int32 类型，受 `BundleFlag` 枚举约束
- `moduleName`: 非空字符串，长度限制由底层 C 接口控制
- `abilityName`: 非空字符串，长度限制由底层 C 接口控制
- `link`: 非空字符串，需符合 URL 格式规范

**错误码**:
| 错误码 | 说明 | 触发条件 |
|--------|------|----------|
| 17700002 | 模块不存在 | moduleName 不匹配 |
| 17700003 | Ability 不存在 | abilityName 不匹配 |
| 17700024 | Profile 不存在 | HAP 中无对应 profile |
| 17700029 | Ability 已禁用 | Ability 被禁用 |
| 17700055 | 无效链接 | link 格式不正确 |
| 17700056 | scheme 不在查询列表 | link scheme 未注册 |

**证据来源**: `bundle_manager.cj:67-163`, `error_code.cj:20-74`

---

### ElementName

**命名空间**: `ohos.element_name`
**类**: `ElementName`
**文件**: `ohos/element_name/element_name.cj`

| JS API | C/C++ 入口 | 参数 | 返回值 | 同步/异步 | 异常 |
|--------|-----------|------|--------|-----------|------|
| `new ElementName(bundleName, abilityName, deviceId?, moduleName?)` | `ElementName.init` | String, String, String?, String? | ElementName | 同步 | - |

**属性**:
| 属性 | 类型 | 可读 | 可写 | 说明 |
|------|------|------|------|------|
| `deviceId` | String | ✅ | ✅ | 设备 ID |
| `bundleName` | String | ✅ | ✅ | Bundle 名称 |
| `moduleName` | String | ✅ | ✅ | 模块名称 |
| `abilityName` | String | ✅ | ✅ | Ability 名称 |

**证据来源**: `element_name.cj:57-160`

---

### Metadata

**命名空间**: `ohos.metadata`
**类**: `Metadata`
**文件**: `ohos/metadata/metadata.cj`

**注意**: Metadata 类为内部使用，通过其他类的 `metadata` 属性访问。

| 属性 | 类型 | 可读 | 可写 | 说明 |
|------|------|------|------|------|
| `name` | String | ✅ | ❌ | 元数据名称 |
| `value` | String | ✅ | ❌ | 元数据值 |
| `resource` | String | ✅ | ❌ | 元数据资源 |

**证据来源**: `metadata.cj:30-118`

---

### Skill

**命名空间**: `ohos.skill`
**类**: `Skill`
**文件**: `ohos/skill/skill.cj`

**注意**: Skill 类为内部使用，通过 AbilityInfo 的 `skills` 属性访问。

| 属性 | 类型 | 可读 | 可写 | 说明 |
|------|------|------|------|------|
| `actions` | Array<String> | ✅ | ❌ | Skill 动作集合 |
| `entities` | Array<String> | ✅ | ❌ | Skill 实体集合 |
| `uris` | Array<SkillUri> | ✅ | ❌ | Skill URI 集合 |
| `domainVerify` | Bool | ✅ | ❌ | 域验证标志 |

### SkillUri

**命名空间**: `ohos.skill`
**类**: `SkillUri`
**文件**: `ohos/skill/skill.cj`

| 属性 | 类型 | 可读 | 可写 | 说明 |
|------|------|------|------|------|
| `scheme` | String | ✅ | ❌ | URI 协议 |
| `host` | String | ✅ | ❌ | 主机 |
| `port` | Int32 | ✅ | ❌ | 端口 |
| `path` | String | ✅ | ❌ | 路径 |
| `pathStartWith` | String | ✅ | ❌ | 路径前缀 |
| `pathRegex` | String | ✅ | ❌ | 路径正则 |
| `uriType` | String | ✅ | ❌ | URI 类型 |
| `utd` | String | ✅ | ❌ | UTD |
| `maxFilesSupported` | Int32 | ✅ | ❌ | 最大支持文件数 |
| `linkFeature` | String | ✅ | ❌ | 链接特性 |

**证据来源**: `skill.cj:30-173`

---

## 辅助类

### BundleInfo

**文件**: `ohos/bundle/bundle_manager/bundle_info.cj`

| 属性 | 类型 | 说明 |
|------|------|------|
| `uid` | Int32 | 用户 ID |
| `name` | String | Bundle 名称 |
| `vendor` | String | 供应商 |
| `versionCode` | UInt32 | 版本号 |
| `versionName` | String | 版本名 |
| `appInfo` | ApplicationInfo | 应用信息 |
| `hapModulesInfo` | Array<HapModuleInfo> | HAP 模块信息 |
| `signatureInfo` | SignatureInfo | 签名信息 |

### ApplicationInfo

**文件**: `ohos/bundle/bundle_manager/application_info.cj`

| 属性 | 类型 | 说明 |
|------|------|------|
| `name` | String | 应用名 |
| `bundleName` | String | Bundle 名称 |
| `description` | String | 描述 |
| `label` | String | 标签 |
| `icon` | String | 图标 |
| `uid` | Int32 | 用户 ID |
| `permissions` | Array<String> | 权限列表 |

**证据来源**: `bundle_info.cj:29-176`, `application_info.cj:1-100`

---

## 枚举类型

### BundleFlag

**文件**: `ohos/bundle/bundle_manager/bundle_flag.cj`

| 枚举值 | 说明 |
|--------|------|
| `GET_BUNDLE_INFO_WITH_APPLICATION` | 获取应用信息 |
| `GET_BUNDLE_INFO_WITH_HAP_MODULE` | 获取 HAP 模块信息 |
| `GET_BUNDLE_INFO_WITH_ABILITY` | 获取 Ability 信息 |
| `GET_BUNDLE_INFO_WITH_PERMISSION` | 获取权限信息 |
| `GET_BUNDLE_INFO_WITH_METADATA` | 获取元数据 |
| `GET_BUNDLE_INFO_ALL` | 获取全部信息 |

### LaunchType

**文件**: `ohos/bundle/bundle_manager/cj_bundle_enum.cj`

| 枚举值 | 说明 |
|--------|------|
| `SINGLETON` | 单例 |
| `STANDARD` | 标准 |
| `SPECIFIED` | 指定 |

**证据来源**: `bundle_flag.cj`, `cj_bundle_enum.cj`

---

## 调用链示例

### getBundleInfoForSelf 调用链

```
JS/Cangjie 调用
    ↓
BundleManager.getBundleInfoForSelf()
    ↓
FfiOHOSGetCallingUid() [FFI]
    ↓
bundle_framework C 接口
    ↓
IPC → BundleManagerService
    ↓
返回 RetBundleInfoV2
    ↓
BundleInfo(ret) [C 结构体转 Cangjie 类]
    ↓
返回 BundleInfo 对象
```

**证据来源**: `bundle_manager.cj:81-98`

### canOpenLink 调用链

```
JS/Cangjie 调用
    ↓
BundleManager.canOpenLink(link)
    ↓
LibC.mallocCString(link) [C 字符串转换]
    ↓
FfiBundleManagerCanOpenLink(cLink, out code) [FFI]
    ↓
错误码检查
    ↓
BusinessException(code) [异常抛出]
    ↓
返回 Bool 结果
```

**证据来源**: `bundle_manager.cj:151-162`

---

## 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [SUMMARY.md](SUMMARY.md) - 文档导航

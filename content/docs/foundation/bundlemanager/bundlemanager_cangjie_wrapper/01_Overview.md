# 项目概览

> bundlemanager_cangjie_wrapper 项目定位、核心能力与运行环境

## 项目定位

### 模块定位

`bundlemanager_cangjie_wrapper` 是 **OpenHarmony Bundle Management 子系统的 Cangjie 语言封装层**。

```
┌─────────────────────────────────────────────────────────────────────┐
│                     OpenHarmony 系统架构                              │
├─────────────────────────────────────────────────────────────────────┤
│  应用层                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Cangjie 应用                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↓                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  bundlemanager_cangjie_wrapper (本仓库)                      │   │
│  │  - Cangjie 类型安全的 API 封装                               │   │
│  │  - FFI 调用底层 C 接口                                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↓                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  bundle_framework (依赖组件)                                 │   │
│  │  - Bundle 管理 C 接口实现                                    │   │
│  │  - IPC/System Ability 通信                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↓                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Bundle Manager Service (系统服务)                           │   │
│  │  - Bundle 信息存储与查询                                      │   │
│  │  - 权限管理                                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

**证据来源**: `README.md:1-8`, `bundle.json:1-15`

### 能力边界

| 能力 | 状态 | 说明 |
|------|------|------|
| 查询当前应用 BundleInfo | ✅ 已支持 | `getBundleInfoForSelf` |
| 获取 Ability Profile | ✅ 已支持 | `getProfileByAbility` |
| 检查链接可打开性 | ✅ 已支持 | `canOpenLink` |
| 查询其他应用信息 | ❌ 未支持 | 暂无实现 |
| UID 转 BundleName | ❌ 未支持 | 暂无实现 |

**证据来源**: `README.md:57-72`

### 运行环境

| 属性 | 值 |
|------|-----|
| 适配系统类型 | standard（标准设备） |
| API Level | 22+ |
| 系统能力 | SystemCapability.BundleManager.BundleFramework.Core |
| ROM 占用 | 400KB |
| RAM 占用 | 468KB |

**证据来源**: `bundle.json:15-19`

## 核心能力

### 1. Bundle 信息查询

通过 `BundleManager` 类提供当前应用的 Bundle 信息查询能力：

```cj
// 获取当前应用的 BundleInfo
let bundleInfo = BundleManager.getBundleInfoForSelf(bundleFlags)
```

### 2. Profile 配置获取

获取应用配置文件的 JSON 字符串：

```cj
// 根据 Ability 获取 profile
let profiles = BundleManager.getProfileByAbility(moduleName, abilityName)
```

### 3. 链接可打开性检查

检查指定链接是否可以由当前应用打开：

```cj
// 检查链接是否可以打开
let canOpen = BundleManager.canOpenLink(link)
```

### 4. 数据类定义

提供标准化的数据类用于承载 Bundle 相关信息：

| 类 | 用途 |
|------|------|
| `ElementName` | 标识 Ability 的元素名称 |
| `Metadata` | 元数据定义 |
| `Skill` | Skill 配置信息 |
| `BundleInfo` | Bundle 完整信息 |
| `AbilityInfo` | Ability 配置信息 |

**证据来源**: `bundle_manager.cj:67-163`

## 关键概念

### Cangjie FFI (Foreign Function Interface)

Cangjie 通过 `foreign` 关键字声明外部 C 函数调用：

```cj
foreign {
    func FfiOHOSGetCallingUid(): Int32
    func FfiOHOSGetBundleInfoForSelfV2(bundleFlags: Int32): RetBundleInfoV2
}
```

### API Level 注解

使用 `@!APILevel` 注解标记 API 版本和系统能力：

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.BundleManager.BundleFramework.Core",
    workerthread: true
]
public static func getBundleInfoForSelf(bundleFlags: Int32): BundleInfo
```

### C 结构体映射

使用 `@C` 注解定义 C 兼容结构体：

```cj
@C
struct RetBundleInfoV2 {
    var name: CString,
    var vendor: CString,
    // ...
}
```

**证据来源**: `bundle_manager.cj:20-46`, `cj_bundle_ffi.cj:26-46`

## 模块职责

### bundle_manager（核心API）

| 职责 | 说明 |
|------|------|
| Bundle 信息查询 | getBundleInfoForSelf, getBundleInfo |
| Profile 获取 | getProfileByAbility, getProfileByExtensionAbility |
| 链接检查 | canOpenLink |
| ABC 验证 | verifyAbc |

### element_name（数据类）

| 职责 | 说明 |
|------|------|
| ElementName 定义 | deviceId, bundleName, moduleName, abilityName |
| FFI 绑定 | 与 ability_runtime C 接口交互 |

### metadata（数据类）

| 职责 | 说明 |
|------|------|
| Metadata 定义 | name, value, resource |
| 数组处理 | CArrMetadata 转换 |

### skill（数据类）

| 职责 | 说明 |
|------|------|
| Skill 定义 | actions, entities, uris, domainVerify |
| SkillUri 定义 | scheme, host, port, path 等 |

**证据来源**: 各模块 `.cj` 文件

## 依赖组件

| 组件 | 用途 | 依赖类型 |
|------|------|----------|
| `bundle_framework` | Bundle 管理 C 接口 | Native (external_deps) |
| `ability_runtime` | ElementName C 定义 | Native (external_deps) |
| `cangjie_ark_interop` | APILevel 注解、BusinessException | Cangjie (cj_external_deps) |
| `hiviewdfx_cangjie_wrapper` | HiLog 日志 | Cangjie (cj_external_deps) |
| `global_cangjie_wrapper` | 应用资源访问 | Cangjie (cj_external_deps) |

**证据来源**: `bundle.json:21-27`, `ohos/bundle/bundle_manager/BUILD.gn:46-56`

---

## 相关文档

- [02_API_Reference.md](02_API_Reference.md) - 详细 API 参考
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [SUMMARY.md](SUMMARY.md) - 文档导航

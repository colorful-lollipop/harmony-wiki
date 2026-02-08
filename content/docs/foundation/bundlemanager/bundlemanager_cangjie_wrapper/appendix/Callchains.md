# 关键调用链

> bundlemanager_cangjie_wrapper 入口到核心逻辑的完整调用链

## 调用链概览

| 入口 API | 核心逻辑 | 调用深度 |
|----------|----------|----------|
| `getBundleInfoForSelf()` | 查询 BundleInfo | 5 层 |
| `getProfileByAbility()` | 获取 Profile JSON | 6 层 |
| `canOpenLink()` | 链接可打开性检查 | 5 层 |

---

## 调用链 1: getBundleInfoForSelf

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 1: JS/Cangjie 调用入口                                                │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:81                       │
│ Code: BundleManager.getBundleInfoForSelf(bundleFlags: Int32): BundleInfo  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 2: UID 获取与缓存检查                                                 │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:82-96                   │
│ Code:                                                                     │
│   let uid = unsafe { FfiOHOSGetCallingUid() }                             │
│   let query = BMQuery(bundleName, GET_BUNDLE_INFO, bundleFlags, ...)      │
│   if (cache.contains(query)) { return cache[query] }                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 3: FFI 调用                                                          │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:91                      │
│ Code: let ret = unsafe { FfiOHOSGetBundleInfoForSelfV2(bundleFlags) }    │
│ Symbol: FfiOHOSGetCallingUid, FfiOHOSGetBundleInfoForSelfV2               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 4: Native C 接口                                                     │
│ Dep: bundle_framework:cj_bundle_manager_ffi                                │
│ Symbol: OH_NativeBundle_GetBundleInfoForSelfV2                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 5: IPC 调用                                                          │
│ Component: BundleMgrHost → BundleManagerService                            │
│ Method: GetBundleInfoForSelf                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 6: BundleManagerService 处理                                          │
│ Component: BundleManagerService                                             │
│ Operation: 查询 BundleInfo 存储                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ 返回路径                                                                   │
│ RetBundleInfoV2 (C Structure) → BundleInfo(ret) → 返回                   │
│ File: ohos/bundle/bundle_manager/bundle_info.cj:158                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

**证据来源**:
- `bundle_manager.cj:81-98` - 主流程
- `bundle_manager.cj:27-29` - FFI 声明
- `cj_bundle_ffi.cj:401-434` - RetBundleInfoV2 定义

---

## 调用链 2: getProfileByAbility

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 1: JS/Cangjie 调用入口                                                │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:118                     │
│ Code: BundleManager.getProfileByAbility(moduleName, abilityName, ...)     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 2: C 字符串转换                                                      │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:119-125                 │
│ Code:                                                                     │
│   cModuleName = LibC.mallocCString(moduleName).asResource()                │
│   cAbilityName = LibC.mallocCString(abilityName).asResource()              │
│   cMetadataName = LibC.mallocCString(metadataName).asResource()            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 3: FFI 调用                                                          │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:126                     │
│ Code: let cValue: RetDataCArrString = FfiGetProfileByAbility(...)         │
│ Symbol: FfiGetProfileByAbility                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 4: Native C 接口                                                     │
│ Dep: bundle_framework:cj_bundle_manager_ffi                                │
│ Symbol: OH_NativeBundle_GetProfileByAbility                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 5: IPC 调用                                                          │
│ Component: BundleMgrHost → BundleManagerService                            │
│ Method: GetAbilityProfile                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 6: 错误码检查                                                        │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:128-129                 │
│ Code:                                                                     │
│   if (cValue.code != 0) {                                                 │
│       throw BusinessException(cValue.code, getErrorMsg(cValue.code))       │
│   }                                                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 7: 结果处理                                                          │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:131-132                 │
│ Code:                                                                     │
│   res = readArrStr(cValue.data)                                           │
│   cValue.data.free()                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

**错误码映射**:
| 错误码 | 常量名 | 触发条件 |
|--------|--------|----------|
| 17700002 | ERROR_MODULE_NOT_EXIST | moduleName 不存在 |
| 17700003 | ERROR_ABILITY_NOT_EXIST | abilityName 不存在 |
| 17700024 | ERROR_PROFILE_NOT_EXIST | HAP 中无 profile |
| 17700029 | ERROR_ABILITY_IS_DISABLED | Ability 已禁用 |

**证据来源**:
- `bundle_manager.cj:118-136` - 主流程
- `error_code.cj:20-74` - 错误码定义

---

## 调用链 3: canOpenLink

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 1: JS/Cangjie 调用入口                                                │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:151                     │
│ Code: BundleManager.canOpenLink(link: String): Bool                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 2: C 字符串转换                                                      │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:152-153                 │
│ Code:                                                                     │
│   let cLink = LibC.mallocCString(link)                                    │
│   var code = 0i32                                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 3: FFI 调用                                                          │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:155                     │
│ Code: let ret = FfiBundleManagerCanOpenLink(cLink, inout code)           │
│ Symbol: FfiBundleManagerCanOpenLink                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 4: C 字符串释放                                                      │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:156                     │
│ Code: LibC.free(cLink)                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 5: Native C 接口                                                     │
│ Dep: bundle_framework:cj_bundle_manager_ffi                                │
│ Symbol: OH_NativeBundle_CanOpenLink                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 6: IPC 调用                                                          │
│ Component: BundleMgrHost → BundleManagerService                            │
│ Method: CanOpenLink                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 7: 错误码检查                                                        │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:157-159                 │
│ Code:                                                                     │
│   if (code != SUCCESS_CODE) {                                              │
│       throw BusinessException(code, getErrorMsg(code))                    │
│   }                                                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ 返回结果                                                                   │
│ File: ohos/bundle/bundle_manager/bundle_manager.cj:160                     │
│ Code: return ret                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

**证据来源**:
- `bundle_manager.cj:151-162` - 完整流程
- `bundle_manager.cj:37` - FFI 声明

---

## ElementName 创建调用链

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 1: JS/Cangjie 调用入口                                                │
│ File: ohos/element_name/element_name.cj:119                                │
│ Code: ElementName(bundleName, abilityName, deviceId?, moduleName?)         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 2: 属性赋值                                                          │
│ File: ohos/element_name/element_name.cj:120-124                            │
│ Code:                                                                     │
│   this.bundleName = bundleName                                             │
│   this.abilityName = abilityName                                           │
│   this.deviceId = deviceId                                                 │
│   this.moduleName = moduleName                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 3: Handle 创建（延迟）                                               │
│ File: ohos/element_name/element_name.cj:146-158                           │
│ Code: createElementNameHandle()                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 4: FFI 调用                                                          │
│ File: ohos/element_name/element_name.cj:155                               │
│ Code: FFICJElementNameCreateWithContent(...)                              │
│ Symbol: FFICJElementNameCreateWithContent                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ Layer 5: Native C 接口                                                     │
│ Dep: ability_runtime:cj_ability_ffi                                         │
│ Symbol: OH_Ability_ElementNameCreateWithContent                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

**证据来源**:
- `element_name.cj:119-159` - ElementName 实现
- `element_name.cj:34-39` - FFI 声明

---

## 相关文档

- [02_API_Reference.md](../../02_API_Reference.md) - API 参考
- [03_Architecture.md](../../03_Architecture.md) - 架构说明
- [SUMMARY.md](../../SUMMARY.md) - 文档导航

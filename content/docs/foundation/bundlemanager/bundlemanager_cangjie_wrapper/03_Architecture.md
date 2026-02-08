# 架构说明

> bundlemanager_cangjie_wrapper 组件架构、数据流与线程模型

## 系统架构

### 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          应用层 (Cangjie Application)                         │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  BundleManager (ohos.bundle.bundle_manager)                         │   │
│  │  - getBundleInfoForSelf(bundleFlags)                                │   │
│  │  - getProfileByAbility(moduleName, abilityName, metadataName?)     │   │
│  │  - canOpenLink(link)                                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ ElementName │  │  Metadata   │  │   Skill     │  │ BundleInfo  │       │
│  │             │  │             │  │             │  │             │       │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                       Cangjie FFI Layer (本仓库)                            │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  foreign { func FfiOHOSGetBundleInfoForSelfV2(...) }               │   │
│  │  foreign { func FfiBundleManagerCanOpenLink(...) }                  │   │
│  │  foreign { func FfiGetProfileByAbility(...) }                      │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  @C struct RetBundleInfoV2 { ... }                                 │   │
│  │  @C struct RetAbilityInfoV2 { ... }                                │   │
│  │  C 结构体映射与内存管理                                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Native Layer (bundle_framework)                          │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  BundleMgrHost (IPC Skeleton)                                       │   │
│  │  - GetBundleInfoForSelf                                            │   │
│  │  - GetAbilityInfo                                                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  BundleDataMgr                                                     │   │
│  │  - BundleInfo storage                                              │   │
│  │  - HAP module info                                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                       Bundle Manager Service (System Ability)               │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  BundleManagerService                                               │   │
│  │  - Bundle registry                                                 │   │
│  │  - Permission management                                           │   │
│  │  - Installation/Uninstallation                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

**证据来源**: `README.md:8-30`, `bundle_manager.cj:26-46`

### 模块依赖关系

```
ohos.bundle.bundle_manager
├── 内部依赖 (cj_deps)
│   ├── ohos.element_name
│   ├── ohos.metadata
│   └── ohos.skill
│
├── Cangjie 外部依赖 (cj_external_deps)
│   ├── ohos.business_exception (cangjie_ark_interop)
│   ├── ohos.labels (cangjie_ark_interop)
│   ├── ohos.ffi (cangjie_ark_interop)
│   ├── ohos.hilog (hiviewdfx_cangjie_wrapper)
│   └── ohos.resource (global_cangjie_wrapper)
│
└── Native 外部依赖 (external_deps)
    └── bundle_framework:cj_bundle_manager_ffi
```

**证据来源**: `ohos/bundle/bundle_manager/BUILD.gn:40-56`

---

## 数据流

### getBundleInfoForSelf 数据流

```
1. 用户调用 BundleManager.getBundleInfoForSelf(bundleFlags)
         ↓
2. FFI 层: FfiOHOSGetCallingUid() 获取调用者 UID
         ↓
3. FFI 层: FfiOHOSGetBundleInfoForSelfV2(bundleFlags)
         ↓
4. Native 层: bundle_framework C 接口
         ↓
5. IPC: BundleMgrHost → BundleManagerService
         ↓
6. BundleManagerService: 查询 BundleInfo
         ↓
7. 返回 RetBundleInfoV2 (C 结构体)
         ↓
8. Cangjie 层: BundleInfo(ret) 构造
         ↓
9. 返回 BundleInfo 对象
```

**证据来源**: `bundle_manager.cj:81-98`

### getProfileByAbility 数据流

```
1. 用户调用 BundleManager.getProfileByAbility(moduleName, abilityName, metadataName?)
         ↓
2. LibC.mallocCString() 转换为 C 字符串
         ↓
3. FFI 层: FfiGetProfileByAbility(cModuleName, cAbilityName, cMetadataName)
         ↓
4. Native 层: bundle_framework C 接口
         ↓
5. IPC: BundleMgrHost → BundleManagerService
         ↓
6. BundleManagerService: 查找 Ability Profile
         ↓
7. 返回 RetDataCArrString (C 结构体)
         ↓
8. 错误码检查: if (cValue.code != 0)
         ↓
9. 抛出 BusinessException(cValue.code, getErrorMsg(cValue.code))
         ↓
10. 返回 Array<String> (JSON profile 数组)
```

**证据来源**: `bundle_manager.cj:118-136`

### canOpenLink 数据流

```
1. 用户调用 BundleManager.canOpenLink(link)
         ↓
2. LibC.mallocCString(link) 转换为 C 字符串
         ↓
3. FFI 层: FfiBundleManagerCanOpenLink(cLink, out code)
         ↓
4. Native 层: bundle_framework C 接口
         ↓
5. BundleManagerService: 检查链接是否可打开
         ↓
6. LibC.free(cLink) 释放 C 字符串
         ↓
7. 错误码检查: if (code != SUCCESS_CODE)
         ↓
8. 抛出 BusinessException(code, getErrorMsg(code))
         ↓
9. 返回 Bool 结果
```

**证据来源**: `bundle_manager.cj:151-162`

---

## 线程模型

### Cangjie M:N 线程模型

| 特性 | 说明 |
|------|------|
| 线程类型 | 用户态轻量级线程（协程） |
| 调度模型 | M:N 模型 |
| 内存开销 | 每个线程约 8KB |
| 切换开销 | 纳秒级，不进入内核态 |
| 抢占支持 | 支持抢占式调度 |

### API 线程标注

| API | workerthread | 说明 |
|-----|-------------|------|
| `getBundleInfoForSelf` | ✅ true | 支持在 worker 线程执行 |
| `getProfileByAbility` | ✅ true | 支持在 worker 线程执行 |
| `canOpenLink` | ❌ false | 仅主线程执行 |

**证据来源**: `bundle_manager.cj:63-162`

### 缓存同步机制

```cj
// BundleInfo 缓存
var cache = HashMap<BMQuery, BundleInfo>()
let MTX = Mutex()

func checkToCache(uid: Int32, callingUid: Int32, query: BMQuery, bundleInfo: BundleInfo): Unit {
    if (uid != callingUid) {
        return  // 跨用户调用，不缓存
    }
    synchronized(MTX) {
        cache[query] = bundleInfo
    }
}
```

**证据来源**: `bundle_manager.cj:48-58`

---

## 错误传播机制

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cangjie 异常处理层                           │
│                                                                 │
│  try {                                                          │
│      FFI 调用                                                   │
│  } catch (e: BusinessException) {                              │
│      throw BusinessException(code, getErrorMsg(code))           │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Native 层错误码                               │
│                                                                 │
│  - 17700001: ERROR_BUNDLE_NOT_EXIST                            │
│  - 17700002: ERROR_MODULE_NOT_EXIST                             │
│  - 17700003: ERROR_ABILITY_NOT_EXIST                            │
│  - ...                                                         │
│  - 17700101: ERROR_BUNDLE_SERVICE_EXCEPTION                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    IPC 层错误码                                  │
│                                                                 │
│  - ERR_OK (0): 成功                                            │
│  - 其他:详见 IPC 错误码规范                                      │
└─────────────────────────────────────────────────────────────────┘
```

**证据来源**: `bundle_manager.cj:118-136`, `error_code.cj:20-74`

---

## 内存管理

### C 结构体内存释放

```cj
// 每个 FFI 返回的 C 结构体都有对应的 free() 方法
@C
struct RetBundleInfoV2 {
    func free(): Unit {
        unsafe {
            LibC.free(this.name)
            LibC.free(this.vendor)
            // ... 释放所有 CString 字段
            this.appInfo.free()
        }
    }
}

// 使用模式
let ret = unsafe { FfiOHOSGetBundleInfoForSelfV2(bundleFlags) }
defer { unsafe { ret.free() } }  // 确保释放
let info = BundleInfo(ret)
```

**证据来源**: `cj_bundle_ffi.cj:401-434`

### 数组内存释放

```cj
// C 数组释放辅助函数
func freeCArrRetHapModuleInfoV2(cArr: CArrRetHapModuleInfoV2) {
    if (cArr.head.isNotNull() && cArr.size != 0) {
        unsafe {
            for (i in 0..cArr.size) {
                cArr.head.read(i).free()
            }
            LibC.free<RetHapModuleInfoV2>(cArr.head)
        }
    }
}
```

**证据来源**: `cj_bundle_ffi.cj:447-456`

---

## 相关文档

- [02_API_Reference.md](02_API_Reference.md) - API 参考
- [04_Build.md](04_Build.md) - 构建配置
- [SUMMARY.md](SUMMARY.md) - 文档导航

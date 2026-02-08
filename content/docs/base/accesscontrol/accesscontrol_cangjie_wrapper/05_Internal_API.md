# 内部 API

## 目的

本文档描述 `accesscontrol_cangjie_wrapper` 的内部模块接口、依赖关系、稳定性说明，帮助开发者理解内部实现细节和扩展点。

## 适用范围

本文档适用于：
- 需要深入理解内部实现的维护者
- 需要进行二次开发的开发者
- 需要评估稳定性的架构师

## 关键结论

1. **内部模块**: 2 个内部模块（ability_access_ctrl、permission_request_result）
2. **依赖方向**: ability_access_ctrl → permission_request_result，无循环依赖
3. **稳定性**: 公开 API 稳定（@APILevel 注解），内部实现可能变更
4. **FFI 层**: 通过 FFI 调用 access_token C 接口，核心逻辑在底层
5. **扩展点**: 声明但未使用的 FFI 函数可用于扩展功能

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 查看模块组织
- [架构说明](03_Architecture.md) - 理解依赖关系
- [对外 API](04_Public_API.md) - 查看公开 API
- [工作笔记](wiki/_work/NOTES.md) - 查看代码证据索引

---

## 内部模块清单

### 模块列表

| 模块 | 路径 | 职责 | 依赖 |
|------|------|------|------|
| **ability_access_ctrl** | `ohos/ability_access_ctrl/` | 权限管理主 API | permission_request_result (内部), ability_cangjie_wrapper, cangjie_ark_interop, hiviewdfx_cangjie_wrapper, access_token (FFI) |
| **permission_request_result** | `ohos/security/permission_request_result/` | 权限请求数据结构 | cangjie_ark_interop, hiviewdfx_cangjie_wrapper |

证据：`ohos/ability_access_ctrl/BUILD.gn:29-39`, `ohos/security/permission_request_result/BUILD.gn:26-31`

---

## 内部模块详细说明

### 1. ability_access_ctrl 模块

**包名**: `ohos.ability_access_ctrl`

**职责**: 提供权限管理主 API

**公开类**:
- `GrantStatus` (enum): 权限授权状态枚举
- `AbilityAccessCtrl`: 工厂类
- `AtManager`: 主类，提供权限管理方法

**内部类**:
- 无

**私有方法**:
- 无（所有方法都是公开的）

**代码文件**:
- `cj_ability_access_ctrl.cj`: 主 API 实现
- `cj_ability_access_ctrl_error.cj`: 错误码定义

---

#### 内部类型和常量

**权限类型**:
```cangjie
public type Permissions = String
```

证据：`cj_ability_access_ctrl.cj:38`

**StageContext 类型**:
```cangjie
type StageContext = CPointer<Unit>
```

证据：`cj_ability_access_ctrl.cj:40`

**常量**:
```cangjie
const SECURITY_DOMAIN_ACCESSTOKEN: UInt32 = 0xD005A01
let ACCESS_LOG = HilogChannel(0, SECURITY_DOMAIN_ACCESSTOKEN, "CJ-AbilityAccessCtrl")
```

证据：`cj_ability_access_ctrl.cj:69-70`

---

#### FFI 接口声明

**foreign 块**:
```cangjie
foreign {
    func FfiOHOSAbilityAccessCtrlCheckAccessTokenSync(tokenID: UInt32, cPermissionName: CString): Int32

    func FfiOHOSAbilityAccessCtrlGrantUserGrantedPermission(tokenID: UInt32, cPermissionName: CString,
        permissionFlags: UInt32): Int32

    func FfiOHOSAbilityAccessCtrlRevokeUserGrantedPermission(tokenID: UInt32, cPermissionName: CString,
        permissionFlags: UInt32): Int32

    func FfiOHOSAbilityAccessCtrlOn(cType: CString, cTokenIDList: CArrUI32, cPermissionList: CArrString, funcId: Int64): Int32

    func FfiOHOSAbilityAccessCtrlOff(cType: CString, cTokenIDList: CArrUI32, cPermissionList: CArrString, funcId: Int64): Int32

    func FfiOHOSAbilityAccessCtrlRequestPermissionsFromUser(context: StageContext, cPermissionList: CArrString,
        id: Int64): Unit

    func FfiOHOSAbilityAccessCtrlRequestPermissionsFromUserByStdFunc(context: StageContext, cPermissionList: CArrString,
        callbackPtr: CPointer<Unit>): Unit

    func FfiOHOSAbilityAccessCtrlRequestPermissionOnSetting(context: StageContext, cPermissionList: CArrString,
        id: Int64): Unit

    func FfiOHOSAbilityAccessCtrlRequestGlobalSwitch(context: StageContext, switchType: Int32, id: Int64): Unit

    func memcpy_s(dest: CPointer<UInt32>, destMax: UIntNative, src: CPointer<UInt32>, count: UIntNative): Int32
}
```

证据：`cj_ability_access_ctrl.cj:42-67`

**使用状态**:

| FFI 函数 | 使用状态 | 说明 |
|---------|---------|------|
| FfiOHOSAbilityAccessCtrlCheckAccessTokenSync | ✅ 已使用 | checkAccessToken 使用 |
| FfiOHOSAbilityAccessCtrlGrantUserGrantedPermission | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlRevokeUserGrantedPermission | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlOn | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlOff | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlRequestPermissionsFromUser | ✅ 已使用 | requestPermissionsFromUser 使用 |
| FfiOHOSAbilityAccessCtrlRequestPermissionsFromUserByStdFunc | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlRequestPermissionOnSetting | ❌ 未使用 | 声明但未调用 |
| FfiOHOSAbilityAccessCtrlRequestGlobalSwitch | ❌ 未使用 | 声明但未调用 |
| memcpy_s | ❌ 未使用 | 声明但未调用 |

---

#### 内部方法

**GrantStatus.toGrantStatus()**:
```cangjie
static func toGrantStatus(code: Int32): GrantStatus {
    if (code == -1) {
        return PermissionDenied
    } else {
        return PermissionGranted
    }
}
```

证据：`cj_ability_access_ctrl.cj:100-106`

**getErrorInfo()** (来自 cj_ability_access_ctrl_error.cj):
```cangjie
func getErrorInfo(code: Int32): String {
    if (let Some(v) <- getUniversalErrorMsg(code)) {
        return v
    } else if (ERROR_CODE_MAP.contains(code)) {
        return ERROR_CODE_MAP[code]
    } else {
        return "Unknown error"
    }
}
```

证据：`cj_ability_access_ctrl_error.cj:44-52`

---

### 2. permission_request_result 模块

**包名**: `ohos.security.permission_request_result`

**职责**: 提供权限请求数据结构

**公开类**:
- `PermissionRequestResult`: 权限请求结果类

**内部类**:
- `CPermissionRequestResult`: C 结构体定义（用于 FFI）
- `RetDataCPermissionRequestResult`: 返回数据结构（用于 FFI）

**代码文件**:
- `permission_request_result.cj`: 数据结构实现

---

#### PermissionRequestResult 类

**属性**:

| 属性 | 类型 | 可见性 | 说明 |
|------|------|--------|------|
| permissions | Array<String> | public | 请求的权限列表 |
| authResults | Array<Int32> | public | 授权结果 |
| dialogShownResults | Array<Bool> | public | 是否显示对话框 |
| errorReasons | Array<Int32> | internal (@Hide) | 错误原因（内部） |

证据：`permission_request_result.cj:42-76`

**方法**:

| 方法 | 可见性 | 说明 | 证据 |
|------|--------|------|------|
| init() | protected | 构造函数 | `permission_request_result.cj:78-84` |
| fromCPermissionRequestResult() | protected static | 从 C 结构体转换 | `permission_request_result.cj:89-127` |

---

#### C 结构体

**CPermissionRequestResult**:
```cangjie
@C
protected struct CPermissionRequestResult {
    CPermissionRequestResult(
        let permissions: CArrString,
        let authResults: CArrI32,
        let dialogShownResults: CArrBool
    ) {}
}
```

证据：`permission_request_result.cj:130-137`

**RetDataCPermissionRequestResult**:
```cangjie
@C
protected struct RetDataCPermissionRequestResult {
    RetDataCPermissionRequestResult(
        protected let code: Int32,
        protected let data: CPermissionRequestResult
    ) {}
}
```

证据：`permission_request_result.cj:139-145`

---

## 模块依赖关系

### 内部依赖图

```
ohos.ability_access_ctrl
└── import ohos.security.permission_request_result.*
```

证据：`cj_ability_access_ctrl.cj:25-26`

### 外部依赖

| 模块 | 依赖类型 | 目标 | 用途 |
|------|---------|------|------|
| ability_access_ctrl | import | ohos.app.ability.ui_ability | 使用 UIAbilityContext |
| ability_access_ctrl | import | ohos.business_exception | 使用 BusinessException, AsyncCallback |
| ability_access_ctrl | import | ohos.ffi | 使用 FFI 类型 (CArrString, CArrUI32, Callback1Param) |
| ability_access_ctrl | import | std.deriving.Derive | 使用 Derive 宏 |
| ability_access_ctrl | import | ohos.hilog.HilogChannel | 使用日志 |
| ability_access_ctrl | import | ohos.labels.APILevel | 使用 API 级别注解 |
| ability_access_ctrl | cj_deps | permission_request_result | 编译时依赖 |
| ability_access_ctrl | cj_external_deps | ability_cangjie_wrapper:ohos.app.ability.ui_ability | 运行时依赖 |
| ability_access_ctrl | cj_external_deps | cangjie_ark_interop:ohos.business_exception | 运行时依赖 |
| ability_access_ctrl | cj_external_deps | cangjie_ark_interop:ohos.ffi | 运行时依赖 |
| ability_access_ctrl | cj_external_deps | cangjie_ark_interop:ohos.labels | 运行时依赖 |
| ability_access_ctrl | cj_external_deps | hiviewdfx_cangjie_wrapper:ohos.hilog | 运行时依赖 |
| ability_access_ctrl | external_deps | access_token:cj_ability_access_ctrl_ffi | FFI 链接 |
| permission_request_result | import | ohos.business_exception.BusinessException | 异常处理 |
| permission_request_result | import | ohos.ffi | FFI 类型 |
| permission_request_result | import | ohos.labels.{APILevel, Hide} | 注解 |
| permission_request_result | cj_external_deps | cangjie_ark_interop:ohos.business_exception | 运行时依赖 |
| permission_request_result | cj_external_deps | cangjie_ark_interop:ohos.ffi | 运行时依赖 |
| permission_request_result | cj_external_deps | cangjie_ark_interop:ohos.labels | 运行时依赖 |
| permission_request_result | cj_external_deps | hiviewdfx_cangjie_wrapper:ohos.hilog | 运行时依赖 |

证据：`ohos/ability_access_ctrl/BUILD.gn:29-39`, `ohos/security/permission_request_result/BUILD.gn:26-31`, `cj_ability_access_ctrl.cj:20-28`, `permission_request_result.cj:20-22`

---

## 稳定性说明

### 稳定接口（Stable）

| 接口 | 稳定性级别 | 说明 |
|------|-----------|------|
| **AbilityAccessCtrl.createAtManager()** | Stable | 工厂方法，API Level 22 |
| **AtManager.checkAccessToken()** | Stable | 权限检查 API，API Level 22 |
| **AtManager.requestPermissionsFromUser()** | Stable | 权限请求 API，API Level 22 |
| **GrantStatus** | Stable | 权限状态枚举，API Level 22 |
| **PermissionRequestResult** | Stable | 权限结果类，API Level 22 |

证据：所有公开 API 都有 `@APILevel[since: "22"]` 注解

### 内部实现（Internal，可能变更）

| 组件 | 稳定性 | 说明 |
|------|--------|------|
| **FFI 接口** | Internal | C 接口可能随 access_token 子系统变更 |
| **C 结构体** | Internal | CPermissionRequestResult, RetDataCPermissionRequestResult |
| **错误码映射** | Internal | ERROR_CODE_MAP 可能扩展 |
| **内部方法** | Internal | fromCPermissionRequestResult() 等内部方法 |
| **未使用的 FFI 函数** | Internal | 可能被移除或启用 |

### 不推荐使用

| 接口 | 原因 | 建议 |
|------|------|------|
| **PermissionRequestResult.errorReasons** | 内部字段，@Hide 注解 | 不依赖此字段 |

---

## 扩展点

### 可扩展的 FFI 函数

以下 FFI 函数已声明但未使用，可用于扩展功能：

| FFI 函数 | 潜在用途 | 实现难度 |
|---------|---------|---------|
| **FfiOHOSAbilityAccessCtrlGrantUserGrantedPermission** | 授予用户权限 | 低 |
| **FfiOHOSAbilityAccessCtrlRevokeUserGrantedPermission** | 撤销用户权限 | 低 |
| **FfiOHOSAbilityAccessCtrlOn** | 注册权限变更监听 | 中 |
| **FfiOHOSAbilityAccessCtrlOff** | 注销权限变更监听 | 中 |
| **FfiOHOSAbilityAccessCtrlRequestPermissionsFromUserByStdFunc** | 使用标准函数的权限请求 | 中 |
| **FfiOHOSAbilityAccessCtrlRequestPermissionOnSetting** | 在设置页面请求权限 | 中 |
| **FfiOHOSAbilityAccessCtrlRequestGlobalSwitch** | 请求全局开关 | 中 |

证据：`cj_ability_access_ctrl.cj:45-64`

### 扩展建议

**示例：添加权限授予接口**

```cangjie
public func grantUserGrantedPermission(tokenID: UInt32, permissionName: Permissions): Unit {
    unsafe {
        let cPermissionName = LibC.mallocCString(permissionName)
        let ret = FfiOHOSAbilityAccessCtrlGrantUserGrantedPermission(tokenID, cPermissionName, 0u32)
        LibC.free(cPermissionName)
        if (ret != 0) {
            throw BusinessException(ret, getErrorInfo(ret))
        }
    }
}
```

---

## 内存管理

### C 内存管理策略

**unsafe 块使用**:

| 位置 | 操作 | 说明 |
|------|------|------|
| checkAccessToken | mallocCString / free | 单个字符串 |
| requestPermissionsFromUser | toArrayCString / freeArrCString | 字符串数组 |
| fromCPermissionRequestResult | C 数组转 Cangjie 数组 + 逐个 free | 批量释放 |

**潜在问题**:
- 如果 FFI 调用抛出异常，free 可能不执行
- 需要确保所有分配的内存都被释放

证据：`cj_ability_access_ctrl.cj:163-168`, `cj_ability_access_ctrl.cj:195-216`, `permission_request_result.cj:89-127`

---

## 线程安全

### 当前线程模型

| API | 线程安全 | 说明 |
|-----|---------|------|
| **checkAccessToken** | 线程安全 | 无共享状态，纯函数 |
| **requestPermissionsFromUser** | 线程安全 | 无共享状态，异步回调独立 |

### 原因

- **无全局变量**: 所有状态都通过参数传递
- **无静态变量**: 没有静态可变状态
- **FFI 层线程安全**: access_token 子系统保证线程安全

---

## 性能考虑

### 时间复杂度

| 操作 | 复杂度 | 说明 |
|------|--------|------|
| **checkAccessToken** | O(1) | 单次 FFI 调用 |
| **requestPermissionsFromUser** | O(n) | n = permissionList.size |

### 内存分配

| 操作 | 内存分配 | 说明 |
|------|---------|------|
| **checkAccessToken** | 1 个 CString | 临时字符串 |
| **requestPermissionsFromUser** | n 个 CString + 数组结构 | n = permissionList.size |

---

## 下一步

1. 查看 [对外 API](04_Public_API.md) 了解公开 API 使用
2. 参考 [GN Targets](06_GN_Targets.md) 理解构建配置
3. 阅读 [安全评审](08_Security_Review.md) 了解安全注意事项

# 目录结构与模块职责

## 目的

本文档描述 `accesscontrol_cangjie_wrapper` 项目的目录结构、模块职责划分，帮助读者快速定位代码和理解代码组织方式。

## 适用范围

本文档适用于：
- 需要修改代码的开发者
- 需要定位特定功能的维护者
- 理解代码组织的新手开发者

## 关键结论

1. **项目根目录**：包含构建配置、组件配置、架构图
2. **ohos 目录**：存放所有 Cangjie 源代码，分为两个模块
3. **ability_access_ctrl 模块**：提供权限管理主 API（AbilityAccessCtrl、AtManager）
4. **permission_request_result 模块**：提供权限请求数据结构（PermissionRequestResult）
5. **mock 目录**：提供 Windows/macOS 平台的 Mock 实现
6. **test 目录**：包含测试代码（本文档不涵盖）

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目全貌
- [架构说明](03_Architecture.md) - 理解模块依赖关系
- [工作笔记](wiki/_work/NOTES.md) - 查看代码证据索引

---

## 项目目录树（不含测试）

```
base/accesscontrol/accesscontrol_cangjie_wrapper/
├── BUILD.gn                                          # 根构建配置
├── bundle.json                                       # 组件配置（依赖、资源、子系统）
├── LICENSE                                            # Apache 2.0 License
├── README.md                                          # 项目说明（英文）
├── README_zh.md                                       # 项目说明（中文）
├── OAT.xml                                            # 开源审计模板
│
├── figures/                                           # 架构图资源
│   └── accesscontrol_cangjie_wrapper_architecture_en.png
│
├── ohos/                                              # Cangjie 接口代码
│   ├── ability_access_ctrl/                           # 权限管理模块
│   │   ├── BUILD.gn                                  # 模块构建配置
│   │   ├── cj_ability_access_ctrl.cj                 # 主 API 实现
│   │   └── cj_ability_access_ctrl_error.cj           # 错误码定义
│   │
│   └── security/
│       └── permission_request_result/                 # 权限请求结果模块
│           ├── BUILD.gn                              # 模块构建配置
│           └── permission_request_result.cj           # 结果数据结构
│
├── mock/                                              # Mock 实现（Windows/macOS）
│   ├── ohos.ability_access_ctrl.cj                    # ability_access_ctrl Mock
│   └── ohos.security.permission_request_result.cj     # permission_request_result Mock
│
└── test/                                              # 测试代码（本文档不涵盖）
    └── ability_access_ctrl/
        └── test/
            ├── oh-package.json5
            └── entry/
                └── oh-package.json5
```

证据：`bash` 目录扫描

---

## 目录说明

### 根目录文件

| 文件/目录 | 说明 | 证据 |
|----------|------|------|
| **BUILD.gn** | 根构建配置，定义了两个子组件的包和 SDK 复制任务 | `BUILD.gn:16-23` |
| **bundle.json** | 组件配置，定义依赖、子系统、资源占用等 | `bundle.json:12-48` |
| **LICENSE** | Apache 2.0 开源协议 | - |
| **README.md** | 英文项目说明文档 | - |
| **README_zh.md** | 中文项目说明文档 | - |
| **OAT.xml** | 开源审计模板 | - |
| **figures/** | 架构图资源目录 | - |

---

## ohos 目录详解

### 1. ability_access_ctrl 模块

**路径**：`ohos/ability_access_ctrl/`

**职责**：提供权限管理主 API

**文件列表**：

| 文件 | 行数（约） | 职责 |
|------|-----------|------|
| **cj_ability_access_ctrl.cj** | 220 | 主 API 实现（GrantStatus、AbilityAccessCtrl、AtManager） |
| **cj_ability_access_ctrl_error.cj** | 53 | 错误码定义（ERROR_CODE_MAP） |
| **BUILD.gn** | 44 | 模块构建配置 |

**核心类**：

- **GrantStatus** (enum): 权限授权状态枚举
  - `PermissionDenied`: 权限被拒绝
  - `PermissionGranted`: 权限已授予
  - 证据：`cj_ability_access_ctrl.cj:80-107`

- **AbilityAccessCtrl**: 工厂类
  - `createAtManager()`: 创建 AtManager 实例
  - 证据：`cj_ability_access_ctrl.cj:116-130`

- **AtManager**: 主类，提供权限管理方法
  - `checkAccessToken()`: 检查权限状态（同步）
  - `requestPermissionsFromUser()`: 请求权限授权（异步）
  - 证据：`cj_ability_access_ctrl.cj:139-219`

**FFI 接口声明**：

```cangjie
foreign {
    func FfiOHOSAbilityAccessCtrlCheckAccessTokenSync(tokenID: UInt32, cPermissionName: CString): Int32
    func FfiOHOSAbilityAccessCtrlRequestPermissionsFromUser(context: StageContext, cPermissionList: CArrString, id: Int64): Unit
    // ... 其他声明但未使用的 FFI 函数
}
```

证据：`cj_ability_access_ctrl.cj:42-67`

---

### 2. permission_request_result 模块

**路径**：`ohos/security/permission_request_result/`

**职责**：提供权限请求数据结构

**文件列表**：

| 文件 | 行数（约） | 职责 |
|------|-----------|------|
| **permission_request_result.cj** | 145 | 权限请求结果数据结构 |
| **BUILD.gn** | 36 | 模块构建配置 |

**核心类**：

- **PermissionRequestResult**: 权限请求结果类
  - `permissions`: 请求的权限列表（Array<String>）
  - `authResults`: 授权结果（Array<Int32>）
  - `dialogShownResults`: 是否显示对话框（Array<Bool>）
  - `errorReasons`: 错误原因（内部字段，Array<Int32>）
  - 证据：`permission_request_result.cj:34-128`

- **CPermissionRequestResult**: C 结构体定义（用于 FFI）
  - 证据：`permission_request_result.cj:130-137`

- **RetDataCPermissionRequestResult**: 返回数据结构（用于 FFI）
  - 证据：`permission_request_result.cj:139-145`

**关键方法**：

- `fromCPermissionRequestResult()`: 从 C 结构体转换为 Cangjie 对象
  - 处理 C 数组到 Cangjie 数组的转换
  - 负责内存释放
  - 证据：`permission_request_result.cj:89-127`

---

## mock 目录

**路径**：`mock/`

**职责**：提供 Windows/macOS 平台的 Mock 实现

**文件列表**：

| 文件 | 说明 |
|------|------|
| **ohos.ability_access_ctrl.cj** | ability_access_ctrl 模块的 Mock 实现 |
| **ohos.security.permission_request_result.cj** | permission_request_result 模块的 Mock 实现 |

**使用条件**：

```gn
if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.ability_access_ctrl.cj" ]
} else {
    sources = [
      "cj_ability_access_ctrl.cj",
      "cj_ability_access_ctrl_error.cj",
    ]
}
```

证据：`ohos/ability_access_ctrl/BUILD.gn:20-27`

---

## 模块依赖关系

### 依赖图

```
ohos.ability_access_ctrl
├── import ohos.security.permission_request_result (内部依赖)
├── import ohos.app.ability.ui_ability (外部依赖)
├── import ohos.business_exception (外部依赖)
├── import ohos.ffi (外部依赖)
├── import ohos.hilog (外部依赖)
└── external_deps: access_token:cj_ability_access_ctrl_ffi

ohos.security.permission_request_result
├── import ohos.business_exception (外部依赖)
├── import ohos.ffi (外部依赖)
└── import ohos.hilog (外部依赖)
```

### 依赖方向

| 模块 | 依赖类型 | 目标模块 | 用途 |
|------|---------|---------|------|
| ability_access_ctrl | 内部 import | permission_request_result | 使用 PermissionRequestResult |
| ability_access_ctrl | cj_deps | permission_request_result | 编译时依赖 |
| ability_access_ctrl | cj_external_deps | ability_cangjie_wrapper | 使用 UIAbilityContext |
| ability_access_ctrl | cj_external_deps | cangjie_ark_interop | 使用 FFI 和异常类 |
| ability_access_ctrl | cj_external_deps | hiviewdfx_cangjie_wrapper | 使用 HiLog |
| ability_access_ctrl | external_deps | access_token | 调用 C FFI 接口 |

证据：`ohos/ability_access_ctrl/BUILD.gn:29-39`, `ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:20-26`

---

## 模块职责边界

### ability_access_ctrl 模块

**职责**：
- ✅ 提供权限管理 API（checkAccessToken、requestPermissionsFromUser）
- ✅ 参数验证（TokenID、Context）
- ✅ FFI 调用封装
- ✅ 错误处理和异常抛出
- ✅ 日志记录

**不负责**：
- ❌ 权限存储（由 access_token 负责）
- ❌ 权限策略定义（由 access_token 负责）
- ❌ UI 对话框渲染（由 ability_cangjie_wrapper 负责）
- ❌ 权限请求结果解析的底层细节（由 permission_request_result 负责）

### permission_request_result 模块

**职责**：
- ✅ 定义权限请求数据结构
- ✅ C 结构体到 Cangjie 对象的转换
- ✅ C 内存管理（释放 C 数组）
- ✅ 提供权限结果的公共属性

**不负责**：
- ❌ 权限请求逻辑（由 ability_access_ctrl 负责）
- ❌ 权限检查逻辑（由 access_token 负责）
- ❌ UI 交互（由 ability_cangjie_wrapper 负责）

---

## 代码组织规范

### 命名规范

| 类型 | 命名规则 | 示例 | 证据 |
|------|---------|------|------|
| **包名** | 小写 + 下划线 | ohos.ability_access_ctrl | `cj_ability_access_ctrl.cj:18` |
| **类名** | 大驼峰 | AbilityAccessCtrl, AtManager | `cj_ability_access_ctrl.cj:116, 139` |
| **方法名** | 小驼峰 | checkAccessToken, requestPermissionsFromUser | `cj_ability_access_ctrl.cj:158, 188` |
| **常量名** | 全大写 + 下划线 | INVALID_PARA, SECURITY_DOMAIN_ACCESSTOKEN | `cj_ability_access_ctrl.cj:30, 69` |
| **类型别名** | 大驼峰 | Permissions, StageContext | `cj_ability_access_ctrl.cj:38, 40` |
| **FFI 函数名** | 前缀 Ffi + 大驼峰 | FfiOHOSAbilityAccessCtrlCheckAccessTokenSync | `cj_ability_access_ctrl.cj:43` |

### 文件组织

- **主 API 文件**: `cj_*.cj` 格式，包含主要类和方法
- **错误文件**: `*_error.cj` 格式，包含错误码定义
- **构建文件**: `BUILD.gn` 放在模块根目录

### 注释规范

- 文件头部包含版权声明和 License 信息
- 公开 API 包含 Javadoc 风格注释
- 包含 `@APILevel` 注解标注 API 级别和系统能力

示例：
```cangjie
/**
 * Checks whether a specified application has been granted the given permission.
 *
 * @param { UInt32 } tokenID - Token ID of the application.
 * @param { Permissions } permissionName - Name of the permission to be verified.
 * @returns { GrantStatus } Returns permission verify result.
 * @throws { BusinessException } 12100001 - Invalid parameter.
 */
@!APILevel[ since: "22", syscap: "SystemCapability.Security.AccessToken" ]
public func checkAccessToken(tokenID: UInt32, permissionName: Permissions): GrantStatus
```

证据：`cj_ability_access_ctrl.cj:142-169`

---

## 测试目录（简要说明）

虽然本文档不涵盖测试内容，但了解测试结构有助于理解项目：

```
test/ability_access_ctrl/test/
├── oh-package.json5         # 测试依赖配置
└── entry/
    └── oh-package.json5     # 入口依赖配置
```

测试代码位于 `test/` 目录，遵循 OpenHarmony 标准测试组织方式。

---

## 快速定位指南

| 功能 | 文件位置 | 关键类/方法 |
|------|---------|------------|
| **权限检查 API** | `ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:158` | AtManager.checkAccessToken() |
| **权限请求 API** | `ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:188` | AtManager.requestPermissionsFromUser() |
| **错误码定义** | `ohos/ability_access_ctrl/cj_ability_access_ctrl_error.cj:27-42` | ERROR_CODE_MAP |
| **权限结果类** | `ohos/security/permission_request_result/permission_request_result.cj:34` | PermissionRequestResult |
| **FFI 声明** | `ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:42-67` | foreign 块 |
| **模块构建** | `ohos/ability_access_ctrl/BUILD.gn` | ohos_cangjie_shared_library |

---

## 下一步

1. 阅读 [架构说明](03_Architecture.md) 理解模块依赖和数据流
2. 查看 [对外 API](04_Public_API.md) 了解 API 详细说明
3. 参考 [内部 API](05_Internal_API.md) 深入内部实现

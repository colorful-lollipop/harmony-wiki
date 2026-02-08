# 定位与边界

## 目的

本文档明确 `accesscontrol_cangjie_wrapper` 项目的定位、边界和核心能力，帮助读者理解项目在系统中的位置和适用场景。

## 适用范围

本文档适用于：
- 评估是否使用本项目的架构师和技术决策者
- 需要理解项目适用范围的集成开发者
- 确认项目限制的维护者

## 关键结论

1. **定位**: Cangjie 语言的权限管理 API 封装层，不是权限管理核心实现
2. **核心能力**: 权限检查和请求两个 API，基于 access_token FFI 接口
3. **运行环境**: 仅支持标准系统（standard），不支持小型系统
4. **关键限制**: Beta 特性，功能不完整，API 可能变更
5. **扩展性**: 有限，主要通过 FFI 调用 access_token，不实现自定义权限逻辑

## 相关跳转

- [项目概览](00_Overview.md) - 了解项目全貌
- [对外 API](04_Public_API.md) - 查看 API 清单
- [安全评审](08_Security_Review.md) - 了解安全边界

---

## 项目定位

### 在 OpenHarmony 系统中的位置

```
OpenHarmony
├── 应用层
│   └── Cangjie 应用
├── 子系统层
│   ├── accesscontrol (权限控制子系统)
│   │   ├── access_token (C 实现) ← 权限管理核心
│   │   └── accesscontrol_cangjie_wrapper (本项目) ← Cangjie API 封装
│   ├── ability
│   │   └── ability_cangjie_wrapper
│   ├── hiviewdfx
│   │   └── hiviewdfx_cangjie_wrapper
│   └── arkcompiler
│       └── cangjie_ark_interop
└── 基础设施层
    └── ...
```

证据：`bundle.json:14`, `README.md:32`

### 定位说明

**本项目是**：
- ❌ 权限管理核心实现（由 access_token 提供）
- ✅ Cangjie 语言的 API 封装层
- ✅ Cangjie 与 access_token 之间的桥梁
- ✅ 面向 Cangjie 开发者的权限管理接口

**本项目不是**：
- ❌ 新的权限管理系统
- ❌ 权限策略引擎
- ❌ 安全审计工具
- ❌ 权限管理数据库

---

## 核心能力

### 1. 权限检查（checkAccessToken）

**功能**：验证应用是否拥有指定权限

**能力范围**：
- ✅ 验证 TokenID 的有效性（非零检查）
- ✅ 检查权限授权状态（Granted/Denied）
- ✅ 支持跨平台（当前应用权限查询）
- ✅ 同步调用，立即返回结果

**能力限制**：
- ❌ 不返回权限授予时间
- ❌ 不返回权限授予方信息
- ❌ 不支持批量权限检查

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:158`

### 2. 权限请求（requestPermissionsFromUser）

**功能**：向用户请求权限授权

**能力范围**：
- ✅ 弹出权限请求对话框（通过 UIAbilityContext）
- ✅ 支持批量权限请求
- ✅ 异步回调返回结果
- ✅ 返回详细的授权信息（权限列表、结果、对话框显示状态）

**能力限制**：
- ❌ 不支持权限请求优先级
- ❌ 不支持自定义请求文案
- ❌ 不支持权限说明链接

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:188`

### 未实现功能

以下功能在 README 中声明但**未实现**（`README.md:55-60`）：

- ❌ 检查应用权限状态
- ❌ 显示全局开关设置对话框
- ❌ 显示二次授权权限设置对话框

---

## 运行环境

### 支持的系统类型

| 系统类型 | 支持状态 | 证据 |
|---------|---------|------|
| **标准系统** (Standard) | ✅ 支持 | `bundle.json:17-19` |
| **小型系统** (Small) | ❌ 不支持 | 未声明支持 |

### 支持的编译平台

| 平台 | 支持状态 | 实现方式 | 证据 |
|------|---------|----------|------|
| **Linux** | ✅ 支持 | 真实实现（access_token FFI） | `ohos/ability_access_ctrl/BUILD.gn:20-27` |
| **Windows** | ✅ Mock 支持 | Mock 实现 | `ohos/ability_access_ctrl/BUILD.gn:20-22` |
| **macOS** | ✅ Mock 支持 | Mock 实现 | `ohos/ability_access_ctrl/BUILD.gn:20-22` |

### 资源占用

| 资源类型 | 占用量 | 证据 |
|---------|--------|------|
| **ROM** | 162 KB | `bundle.json:22` |
| **RAM** | 180 KB | `bundle.json:23` |

---

## 关键概念

### TokenID

**定义**：应用的 32 位唯一标识符

**作用**：
- 在 access_token 子系统中唯一标识应用
- 用于权限验证和授权查询
- 包含应用身份 APPID、用户 ID、应用克隆索引、APL 等级等信息

**获取方式**：
- 由系统自动分配
- 通过 access_token 相关 API 查询
- 跨设备场景下 TokenID 可能不同

**安全说明**：
- TokenID = 0 被视为无效参数（`cj_ability_access_ctrl.cj:159-161`）
- 恶意应用可能尝试伪造 TokenID 进行权限查询

证据：`README.md:5`, `cj_ability_access_ctrl.cj:42`

### GrantStatus

**定义**：权限授权状态枚举

**枚举值**：
- `PermissionDenied` (-1): 权限被拒绝
- `PermissionGranted` (非 -1): 权限已授予

**转换逻辑**：
```cangjie
static func toGrantStatus(code: Int32): GrantStatus {
    if (code == -1) {
        return PermissionDenied
    } else {
        return PermissionGranted
    }
}
```

证据：`cj_ability_access_ctrl.cj:80-107`

### Permissions

**定义**：权限名称类型（String 别名）

**格式**：标准权限格式，例如 `ohos.permission.READ_CALENDAR`

**限制**：
- 文档声明权限名称不应超过 256 字符
- 代码中**未实现**长度验证（见安全风险）

证据：`cj_ability_access_ctrl.cj:38`, `README.md:150`

### UIAbilityContext

**定义**：Stage 模型的 UIAbility 上下文

**作用**：
- 提供 Stage 模型的上下文信息
- 用于弹出权限请求对话框
- 必须属于应用自身（不能使用其他应用的上下文）

**验证**：
```cangjie
let stageContext = getStageContext(context)
if (stageContext.isNull()) {
    throw BusinessException(COMMON_INNER_ERROR, getErrorInfo(COMMON_INNER_ERROR))
}
```

证据：`cj_ability_access_ctrl.cj:190-193`

---

## 边界与限制

### 功能边界

| 功能 | 状态 | 说明 |
|------|------|------|
| **权限检查** | ✅ 已实现 | 仅支持单个权限检查 |
| **权限请求** | ✅ 已实现 | 支持批量，异步回调 |
| **权限授予** | ❌ 未实现 | 声明了 FFI 函数但未调用 |
| **权限撤销** | ❌ 未实现 | 声明了 FFI 函数但未调用 |
| **权限状态查询** | ❌ 未实现 | README 中声明未实现 |
| **全局开关** | ❌ 未实现 | README 中声明未实现 |

证据：`cj_ability_access_ctrl.cj:45-49`, `README.md:55-60`

### 数据边界

| 数据类型 | 限制 | 实现 |
|---------|------|------|
| **TokenID** | 不能为 0 | 已实现（`cj_ability_access_ctrl.cj:159`） |
| **PermissionName** | 不应超过 256 字符 | 未实现 |
| **PermissionList** | 不能为空或 null | 未明确验证 |
| **Context** | 必须有效且属于应用自身 | 已实现（`cj_ability_access_ctrl.cj:191`） |

### 调用边界

| 调用方式 | 限制 |
|---------|------|
| **同步调用** | checkAccessToken 仅支持同步 |
| **异步调用** | requestPermissionsFromUser 仅支持异步回调 |
| **跨设备** | 不支持跨设备权限查询（沙箱/远程设备 TokenID 限制） |
| **跨应用** | 不能使用其他应用的 Context |

---

## 依赖边界

### 外部依赖（必需）

| 组件 | 依赖类型 | 用途 | 失败影响 |
|------|---------|------|----------|
| **access_token** | external_deps | 提供 C FFI 接口 | 功能不可用 |
| **ability_cangjie_wrapper** | cj_external_deps | 提供 UIAbilityContext | 权限请求对话框无法弹出 |
| **hiviewdfx_cangjie_wrapper** | cj_external_deps | 提供 HiLog 日志 | 仅影响调试日志 |
| **cangjie_ark_interop** | cj_external_deps | 提供 FFI 和异常类 | 编译失败 |

证据：`ohos/ability_access_ctrl/BUILD.gn:31-39`

### 内部依赖

| 模块 | 依赖关系 | 说明 |
|------|---------|------|
| ability_access_ctrl | → permission_request_result | 权限请求数据结构 |

---

## 扩展性说明

### 可扩展点

1. **新增 API**：可以添加新的 FFI 调用封装
   - FFI 函数已声明：GrantUserGrantedPermission、RevokeUserGrantedPermission、RequestPermissionOnSetting 等
   - 证据：`cj_ability_access_ctrl.cj:45-64`

2. **权限验证逻辑**：可以在 FFI 调用前后添加额外的验证
   - 例如：权限名称格式验证、白名单检查等

3. **错误处理**：可以扩展错误码映射和错误消息

### 不可扩展点

1. **权限策略**：权限策略由 access_token 子系统定义，本封装层无法修改
2. **TokenID 分配**：TokenID 由系统分配，本封装层无法自定义
3. **权限存储**：权限信息由 access_token 子系统管理，本封装层不存储

---

## 性能特征

| 操作 | 性能特征 | 说明 |
|------|---------|------|
| **checkAccessToken** | 同步，快速 | 单次 FFI 调用 |
| **requestPermissionsFromUser** | 异步，较慢 | 涉及 UI 交互，等待用户响应 |
| **权限列表转换** | O(n) | C 数组转 Cangjie 数组，线性复杂度 |

---

## 与其他组件的关系

### 与 access_token 的关系

- **access_token**：提供 C 层权限管理实现
- **本项目**：提供 Cangjie API 封装
- **关系**：封装层 vs 被封装层，通过 FFI 交互

证据：`README.md:29`

### 与 ability_cangjie_wrapper 的关系

- **ability_cangjie_wrapper**：提供 UIAbilityContext
- **本项目**：使用 UIAbilityContext 弹出权限请求对话框
- **关系**：服务消费者 vs 服务提供者

证据：`README.md:32`

### 与 cangjie_ark_interop 的关系

- **cangjie_ark_interop**：提供 FFI 框架、异常类、API 级别注解
- **本项目**：使用 FFI 框架调用 C 接口
- **关系**：框架使用者 vs 框架提供者

证据：`README.md:31`

---

## 下一步

1. 查看 [对外 API](04_Public_API.md) 了解 API 详细说明
2. 阅读 [安全评审](08_Security_Review.md) 了解安全限制
3. 参考 [故障排查](09_Troubleshooting.md) 解决常见问题

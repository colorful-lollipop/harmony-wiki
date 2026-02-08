# 项目概览

## 目的

本文档提供 `accesscontrol_cangjie_wrapper` 项目的快速概览，帮助读者快速理解项目全貌。

## 适用范围

本文档适用于：
- 第一次接触本项目的开发者
- 需要了解项目整体架构的维护者
- 评估是否使用本项目的技术决策者

## 关键结论

1. **项目定位**: 基于 OpenHarmony access_token 能力的 Cangjie API 封装库（Beta 特性）
2. **核心功能**: 提供 `checkAccessToken` 和 `requestPermissionsFromUser` 两个权限管理 API
3. **技术栈**: Cangjie 编程语言 + FFI（Foreign Function Interface）
4. **依赖组件**: access_token、hiviewdfx_cangjie_wrapper、cangjie_ark_interop、ability_cangjie_wrapper
5. **代码规模**: 约 400 行 Cangjie 代码（不含测试）
6. **输出产物**: 两个共享库（`.so`）

## 相关跳转

- [定位与边界](01_Positioning_Boundaries.md) - 了解项目边界和限制
- [目录结构](02_Directory_Structure.md) - 熟悉代码组织
- [对外 API](04_Public_API.md) - 学习 API 使用
- [工作笔记](wiki/_work/NOTES.md) - 查看代码证据索引

---

## 项目简介

`accesscontrol_cangjie_wrapper` 是 OpenHarmony 系统的一个子系统组件，位于 `base/accesscontrol/accesscontrol_cangjie_wrapper`。该项目将 OpenHarmony 底层 `access_token` 子系统的能力封装为 Cangjie 编程语言的 API，为开发者提供应用权限管理能力。

### 基本信息

| 属性 | 值 |
|------|-----|
| **包名** | @ohos/accesscontrol_cangjie_wrapper |
| **子系统** | accesscontrol |
| **部件** | accesscontrol_cangjie_wrapper |
| **版本** | 6.1 (Beta) |
| **适用系统** | standard（标准系统） |
| **License** | Apache 2.0 |
| **代码行数** | 约 400 行（不含测试） |
| **ROM 占用** | 162 KB |
| **RAM 占用** | 180 KB |

证据：`bundle.json:12-23`

---

## 核心功能

### 1. checkAccessToken - 权限检查

检查指定应用是否被授予某个权限。

**功能特点**：
- 同步调用，立即返回权限状态
- 支持 TokenID 和权限名称验证
- 返回枚举值（PermissionGranted / PermissionDenied）

**使用场景**：
- 应用启动前检查关键权限
- 动态权限验证
- 跨平台权限状态查询（当前应用）

代码位置：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:158`

### 2. requestPermissionsFromUser - 权限请求

向用户请求权限授权。

**功能特点**：
- 异步回调机制
- 支持批量权限请求
- 返回详细的授权结果（权限列表、授权结果、对话框显示状态）

**使用场景**：
- 首次运行应用请求权限
- 用户主动触发的权限请求
- 需要用户授权的敏感操作

代码位置：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:188`

---

## 技术栈

### 编程语言

- **Cangjie**: 仓颉编程语言（OpenHarmony 自研语言）
- **C**: FFI 互操作层（通过 FFI 调用 access_token C 接口）

### 构建系统

- **GN**: Google Ninja 构建系统
- **CJC**: Cangjie 编译器
- **模板**: `//build/templates/cangjie/cjc.gni`

证据：`BUILD.gn:14`

---

## 依赖组件

| 组件 | 用途 | 证据 |
|------|------|------|
| **access_token** | 提供基础访问控制功能（C FFI 接口） | `bundle.json:25-30` |
| **hiviewdfx_cangjie_wrapper** | HiLog 日志打印 | `bundle.json:25-30` |
| **cangjie_ark_interop** | APILevel 定义、BusinessException 异常类、FFI 支持 | `bundle.json:25-30` |
| **ability_cangjie_wrapper** | 提供 UIAbilityContext 用于弹出权限请求对话框 | `bundle.json:25-30` |

---

## 架构概览

### 系统层次

```
┌─────────────────────────────────────┐
│   应用层（Cangjie 应用）              │
└─────────────┬───────────────────────┘
              │
              ↓
┌─────────────────────────────────────┐
│  accesscontrol_cangjie_wrapper      │
│  - AbilityAccessCtrl               │
│  - AtManager                       │
│  - PermissionRequestResult         │
└────┬──────────────────┬────────────┘
     │                  │
     ↓                  ↓
┌─────────────────────────────────────┐
│  ability_cangjie_wrapper            │  ← UIAbilityContext
└─────────────────────────────────────┘
              │
              ↓
┌─────────────────────────────────────┐
│  access_token (C 层)                │  ← FFI 接口
└─────────────────────────────────────┘
```

### 数据流

1. **权限检查**：应用 → AtManager.checkAccessToken → FFI → access_token → 返回 GrantStatus
2. **权限请求**：应用 → AtManager.requestPermissionsFromUser → UIAbilityContext → 弹出对话框 → 异步回调返回结果

---

## 代码规模

| 模块 | 文件数 | 代码行数（约） |
|------|--------|---------------|
| ability_access_ctrl | 2 (.cj) | 220 行 |
| permission_request_result | 1 (.cj) | 145 行 |
| BUILD.gn | 3 | 103 行 |
| 配置文件 | 1 (bundle.json) | 50 行 |
| **总计** | **7** | **约 600 行** |

---

## 当前限制（Beta 特性）

以下功能尚未实现（`README.md:55-60`）：

- ❌ 检查应用权限状态
- ❌ 显示全局开关设置对话框
- ❌ 显示二次授权权限设置对话框

---

## 快速开始

### 1. 导入模块

```cangjie
import ohos.ability_access_ctrl.*
```

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:18`

### 2. 创建管理器

```cangjie
let atManager = AbilityAccessCtrl.createAtManager()
```

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:127`

### 3. 检查权限

```cangjie
let tokenID = 123456u32
let permission = "ohos.permission.READ_CALENDAR"
let status = atManager.checkAccessToken(tokenID, permission)
if (status == GrantStatus.PermissionGranted) {
    // 权限已授予
}
```

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:158`

### 4. 请求权限

```cangjie
atManager.requestPermissionsFromUser(
    context,
    ["ohos.permission.READ_CALENDAR", "ohos.permission.CAMERA"],
    { error, result =>
        if (error == None) {
            // 处理权限请求结果
            for (i in 0..result.authResults.size) {
                if (result.authResults[i] == 0) {
                    // 权限已授予
                }
            }
        }
    }
)
```

证据：`ohos/ability_access_ctrl/cj_ability_access_ctrl.cj:188`

---

## 项目状态

| 维度 | 状态 |
|------|------|
| **开发阶段** | Beta |
| **API 稳定性** | 可能变更 |
| **文档完整性** | 完整 |
| **测试覆盖** | 存在测试代码 |
| **生产就绪** | 否（Beta 特性） |

---

## 下一步

1. 阅读 [定位与边界](01_Positioning_Boundaries.md) 了解项目适用场景和限制
2. 查看 [目录结构](02_Directory_Structure.md) 熟悉代码组织
3. 学习 [对外 API](04_Public_API.md) 掌握 API 使用方法
4. 参考 [安全评审](08_Security_Review.md) 了解安全注意事项

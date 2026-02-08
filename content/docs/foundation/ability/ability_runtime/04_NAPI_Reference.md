# N-API 接口参考

## 概述

ability_runtime 通过 N-API（Node.js API）为 JavaScript/TypeScript 应用提供调用 Ability 运行时功能的接口。所有 N-API 模块都通过 `napi_module_register()` 函数注册。

## N-API 模块清单

### 按功能分类

#### 1. 基础 Ability 模块

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| ability | `ability_module.cpp` | Ability 基础操作 |
| ability_context | `ability_context_module.cpp` | Ability 上下文管理 |
| ability_constant | `ability_constant_module.cpp` | Ability 常量定义 |
| abilityDataUriUtils | `ability_data_uri_utils_module.cpp` | URI 工具 |

**代码证据**：`frameworks/js/napi/ability/` 目录下各模块的 `*_module.cpp` 文件

#### 2. 应用管理模块

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| application | `application_module.cpp` | 应用基础管理 |
| app_manager | `app_manager_module.cpp` | 应用管理器 |
| application_context | `application_context_module.cpp` | 应用上下文 |
| ability_stage | `ability_stage_module.cpp` | AbilityStage |
| ability_lifecycle_callback | `ability_lifecycle_callback_module.cpp` | 生命周期回调 |

**代码证据**：`frameworks/js/napi/app/` 目录下各模块

#### 3. FA 模型模块（API 8）

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| featureAbility | `feature_ability_module.cpp` | Feature Ability |
| particleAbility | `particleAbility_module.cpp` | Particle Ability |

**代码证据**：`frameworks/js/napi/featureAbility/` 和 `frameworks/js/napi/particleAbility/`

#### 4. Extension 模块

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| service_extension_ability | `service_extension_ability_module.cpp` | Service Extension |
| ui_extension_ability | `ui_extension_ability_module.cpp` | UI Extension |
| auto_fill_extension_ability | `auto_fill_extension_ability_module.cpp` | 自动填充 Extension |
| photo_editor_extension_ability | `photo_editor_extension_ability_module.cpp` | 图片编辑 Extension |
| share_extension_ability | `share_extension_ability_module.cpp` | 分享 Extension |
| embedded_ui_extension_ability | `embedded_ui_extension_ability_module.cpp` | 嵌入式 UI Extension |
| ui_service_extension_ability | `ui_service_extension_ability_module.cpp` | UI 服务 Extension |
| action_extension_ability | `action_extension_ability_module.cpp` | 动作 Extension |

**代码证据**：`frameworks/js/napi/` 目录下各 `*_extension_ability/` 目录

#### 5. 系统能力管理

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| ability_manager | `ability_manager_module.cpp` | Ability 管理服务 |
| mission_manager | `mission_manager_module.cpp` | 任务管理 |
| js_mission_manager | `distributed_mission_manager.cpp` | 分布式任务管理 |

#### 6. 跨组件通信

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| caller | `caller_module.cpp` | 调用者 API |
| callee | `callee_module.cpp` | 被调用者 API |
| wantagent | `want_agent_module.cpp` | Want 代理 |
| uri_permission | `native_module.cpp` | URI 权限管理 |

#### 7. 应用配置

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| configuration_constant | `configuration_constant_module.cpp` | 配置常量 |
| application_context_constant | `application_context_constant_module.cpp` | 应用配置常量 |
| wantConstant | `native_module.cpp` | Want 常量 |

#### 8. 错误码与诊断

| 模块名 | 头文件 | 主要功能 |
|-------|--------|---------|
| errorcode | `ability_errorcode_module.cpp` | 错误码定义 |

## 核心 API 详解

### abilityManager

**模块注册**：`frameworks/js/napi/ability_manager/ability_manager_module.cpp`

**主要方法**：

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `startAbility()` | want, options, callback | void | 启动 Ability |
| `startAbilityWithAccount()` | want, accountId, callback | void | 指定账户启动 |
| `terminateSelf()` | callback | void | 终止当前 Ability |
| `connectAbility()` | want, connection, callback | number | 连接 Service |
| `disconnectAbility()` | connection, callback | void | 断开连接 |
| `getAbilityRunningInfo()` | - | AbilityRunningInfo | 获取运行信息 |
| `getMissionInfo()` | missionId, callback | MissionInfo | 获取任务信息 |
| `getMissionListInfo()` | callback | MissionListInfo | 获取任务列表 |

**参数校验**：
- `want`：必须包含 `bundleName` 或 `action`
- `accountId`：必须在有效范围内
- `connection`：必须实现 IAbilityConnection 接口

**错误码**：
- `0`：成功
- `16000001`：指定的 Ability 不存在
- `16000002`：参数错误
- `16000003`： IPC 错误
- `16000004`：目标应用已冻结

### applicationContext

**模块注册**：`frameworks/js/napi/app/application_context/application_context_module.cpp`

**主要方法**：

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getApplicationContext()` | - | ApplicationContext | 获取应用上下文 |
| `getProcessInfo()` | - | ProcessInfo | 获取进程信息 |

### featureAbility

**模块注册**：`frameworks/js/napi/featureAbility/feature_ability_module.cpp`

**主要方法**（FA 模型）：

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `startAbility()` | want, requestCode | void | 启动 Ability |
| `startAbilityForResult()` | want, requestCode | void | 启动并获取结果 |
| `terminateSelf()` | - | void | 终止当前 Ability |
| `connectAbility()` | want, connection | number | 连接 Service |
| `disconnectAbility()` | connection | void | 断开连接 |

## N-API 注册模式

### 标准注册结构

```cpp
// frameworks/js/napi/ability_manager/ability_manager_module.cpp:35
napi_module_register(&_module);

// 模块定义
napi_module _module = {
    nm_version: 1,
    nm_flags: 0,
    nm_filename: nullptr,
    nm_register_func: Init,
    nm_modname: "abilityManager",
    nm_priv: ((void*)0),
    reserved: { 0 }
};
```

### Init 函数结构

```cpp
static napi_value Init(napi_env env, napi_value exports) {
    // 定义属性
    napi_property_descriptor desc[] = {
        {"startAbility", nullptr, StartAbility, nullptr, nullptr, nullptr, napi_default, nullptr},
        {"terminateSelf", nullptr, TerminateSelf, nullptr, nullptr, nullptr, napi_default, nullptr},
        // ... 更多方法
    };
    
    // 注册到 exports
    napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
    
    return exports;
}
```

### 导出符号位置

所有 N-API 模块注册文件位于：`frameworks/js/napi/*/native_module.cpp` 或 `*_module.cpp`

## 异步调用模式

### Callback 模式

```typescript
abilityManager.startAbility(want, (err) => {
    if (err) {
        console.error(`启动失败: ${err.code} - ${err.message}`);
    } else {
        console.log('启动成功');
    }
});
```

### Promise 模式

```typescript
try {
    await abilityManager.startAbility(want);
    console.log('启动成功');
} catch (err) {
    console.error(`启动失败: ${err.code} - ${err.message}`);
}
```

### 线程模型

- **JS 主线程**：接收回调和返回 Promise
- **Native 线程**：执行实际业务逻辑
- **IPC 线程**：处理跨进程请求

## 参数校验机制

### want 参数校验

```cpp
// 伪代码示例
napi_value StartAbility(napi_env env, napi_callback_info info) {
    // 1. 解析参数
    Want *want = ParseWant(env, args);
    
    // 2. 校验必填字段
    if (want->bundleName.empty() && want->action.empty()) {
        // 设置错误码 ERR_INVALID_VALUE
        return ThrowError(env, 401);
    }
    
    // 3. 校验路径安全
    if (!IsPathSafe(want->uri)) {
        // 设置错误码 ERR_INVALID_VALUE
        return ThrowError(env, 401);
    }
    
    // 4. 调用底层服务
    int32_t result = AbilityManagerClient::GetInstance()->StartAbility(*want);
    
    return result;
}
```

### Bundle/Ability 名称校验

- 长度限制：最大 256 字符
- 字符集：`[a-zA-Z0-9.-_]`
- 格式：`bundleName` 必须包含 `.`，`abilityName` 可选

## 常见错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | ERR_OK | 成功 |
| 401 | ERR_INVALID_VALUE | 参数错误 |
| 16000001 | - | 指定的能力不存在 |
| 16000002 | - | 指定的参数不合法 |
| 16000003 | - | IPC 错误 |
| 16000004 | - | 目标应用已冻结 |
| 16000005 | - | 权限校验失败 |
| 16000006 | - | 操作被禁止 |
| 16000007 | - | 任务栈已满 |

## 相关文档

- [Inner API](05_Inner_API.md)
- [安全风险评审](08_Security_Review.md)
- [常见问题](09_Troubleshooting.md)

# 安全风险评审

## 概述

本文档对 ability_runtime 进行基于代码证据的安全风险分析，涵盖攻击面识别、信任边界分析和可利用点梳理。

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────┐      ┌─────────────────────────┐              │
│  │     系统服务层           │      │      应用进程层          │              │
│  │  (高信任)              │      │  (中等信任)             │              │
│  │  • AbilityManagerSvc  │      │  • 应用代码             │              │
│  │  • AppManagerSvc      │      │  • HAP 包              │              │
│  │  • BundleManager      │      │  • Native 模块          │              │
│  │  • IPC/RPC 框架        │      │  • JS 运行时            │              │
│  └───────────┬─────────────┘      └───────────┬─────────────┘              │
│              │                                │                               │
│              │         IPC/Binder 通信        │                               │
│              │       (跨信任边界)             │                               │
│              └────────────────────────────────┘                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 信任等级

| 区域 | 信任等级 | 说明 |
|------|---------|------|
| 系统服务层 | 高 | 内置系统组件，权限最高 |
| IPC 框架 | 高 | 负责跨进程通信控制 |
| 应用进程 | 中 | 第三方代码运行环境 |
| N-API 接口 | 中 | 应用调用系统能力的入口 |

## 攻击面识别

### 1. N-API 接口（高攻击面）

**描述**：JS/TS 应用通过 N-API 调用系统能力，是外部输入的主要入口。

**代码证据**：
```
frameworks/js/napi/*/native_module.cpp (89 个模块)
每个模块通过 napi_module_register() 注册
```

**风险点**：
- 参数解析可能存在边界检查不足
- 回调函数可能引入 UAF
- Promise/Callback 竞态条件

### 2. IPC 接口（中攻击面）

**描述**：AbilityManagerService 的 IPC 接口接收跨进程请求。

**代码证据**：
```
services/abilitymgr/src/ability_manager_stub.cpp
services/abilitymgr/src/ability_manager_proxy.cpp
```

**风险点**：
- Parcel 数据解析可能越界
- IPC 调用权限校验可能绕过

### 3. Want 参数解析（中攻击面）

**描述**：启动 Ability 时传入的 Want 参数需要解析和验证。

**代码证据**：
```
services/abilitymgr/src/utils/ability_permission_util.cpp
services/abilitymgr/src/implicit_start_processor.cpp
```

**风险点**：
- URI 路径遍历
- Bundle/Ability 名称注入
- Intent 参数注入

### 4. 文件系统访问（低攻击面）

**描述**：应用可能通过 Ability 访问文件系统。

**相关组件**：
- DataAbility
- UriPermissionManager

### 5. 动态加载（中攻击面）

**描述**：应用可能动态加载代码或资源。

**相关组件**：
- QuickFix 热修复
- 动态模块加载

## 权限校验机制

### PermissionVerification 类

**头文件**：`services/common/include/permission_verification.h`

**核心方法**：

```cpp
class PermissionVerification {
public:
    // 权限校验
    bool VerifyCallingPermission(const std::string &permissionName, 
                                const uint32_t specifyTokenId = 0);
    bool VerifyPermissionByTokenId(const int &tokenId, 
                                   const std::string &permissionName) const;
    
    // 调用身份验证
    bool IsSACall() const;           // 系统能力调用
    bool IsShellCall() const;        // Shell 调用
    bool IsSystemAppCall() const;    // 系统应用调用
    
    // 扩展权限校验
    int CheckCallServiceAbilityPermission(const VerificationInfo &verificationInfo,
                                         uint32_t specifyTokenId = 0);
    int CheckCallDataAbilityPermission(const VerificationInfo &verificationInfo,
                                       bool isShell);
};
```

### 权限校验调用点

**代码证据**：`services/abilitymgr/src/ability_manager_service.cpp` 中大量权限校验调用

示例：
```cpp
// 第 11671 行
return CheckCallServiceAbilityPermission(abilityRequest);

// 第 156 行
if (!AAFwk::PermissionVerification::GetInstance()->JudgeCallerIsAllowedToUseSystemAPI()) {
    // 权限不足，拒绝操作
}
```

## 可利用点分析

### 可利用点 1：Want 参数注入

**风险等级**：中等

**描述**：启动 Ability 时传入的 Want 参数可能包含恶意数据。

**触发场景**：
```cpp
// 未验证的 Want 参数可能导致：
// 1. 路径遍历攻击
// 2. Bundle 名称注入
// 3. Action 注入
Want want;
want.SetParam("malicious_key", malicious_value);
AbilityManagerClient::GetInstance()->StartAbility(want);
```

**代码证据**：
```cpp
// services/abilitymgr/src/implicit_start_processor.cpp
// 存在参数解析逻辑，但需验证是否完整
```

**影响**：
- 提权攻击
- 绕过权限检查
- 信息泄露

**修复建议**：
- 对 Want 参数进行白名单校验
- 限制可传递的参数 key
- 对敏感参数进行签名验证

---

### 可利用点 2：回调函数 UAF

**风险等级**：高

**描述**：N-API 回调函数在异步操作中可能存在 Use-After-Free。

**触发场景**：
```typescript
// JS 层回调在 Native 处理过程中可能被释放
let connection = {
    onConnect: () => { /* ... */ },
    onDisconnect: () => { /* ... */ }
};
abilityManager.connectAbility(want, connection, (err) => {
    // 回调中访问已释放的对象
});
```

**代码证据**：
```cpp
// frameworks/js/napi/inner/napi_ability_common/js_napi_common.cpp
// NAPIAbilityConnection 管理回调生命周期
static std::map<ConnectionKey, sptr<NAPIAbilityConnection>, key_compare> connects_;
```

**影响**：
- 远程代码执行
- 进程崩溃

**修复建议**：
- 使用引用计数管理回调对象生命周期
- 在异步操作完成前禁止回调对象释放
- 添加对象有效性检查

---

### 可利用点 3：IPC 参数越界

**风险等级**：中等

**描述**：IPC 参数解析可能存在缓冲区溢出。

**触发场景**：
```cpp
// 客户端发送恶意构造的 MessageParcel
// 服务端解析时越界访问
MessageParcel data;
// 恶意数据可能导致读取未授权内存
data.ReadString();  // 可能越界
data.ReadInt32();   // 可能解析错误
```

**代码证据**：
```cpp
// services/abilitymgr/src/ability_manager_stub.cpp
// IPC 参数解析逻辑
case START_ABILITY: {
    auto want = Want::Unmarshaling(data);  // Want 反序列化
    // ...
}
```

**影响**：
- 内存信息泄露
- 远程代码执行

**修复建议**：
- 在 MessageParcel 读取时进行边界检查
- 使用安全的反序列化库
- 添加参数完整性校验

---

### 可利用点 4：权限校验绕过

**风险等级**：高

**描述**：通过特定调用路径可能绕过权限检查。

**触发场景**：
```cpp
// 直接调用内部接口可能跳过权限校验
// 某些 IPC 接口权限检查不完整
```

**代码证据**：
```cpp
// services/abilitymgr/src/utils/ability_permission_util.cpp
// 权限检查分散在多个函数中
if (!AAFwk::PermissionVerification::GetInstance()->IsSystemAppCall()) {
    // 权限检查
}
```

**影响**：
- 未授权操作
- 提权攻击

**修复建议**：
- 统一权限校验入口
- 所有 IPC 接口强制权限校验
- 使用装饰器模式统一处理

---

### 可利用点 5：路径遍历

**风险等级**：中等

**描述**：通过 URI 或文件路径访问受保护资源。

**触发场景**：
```cpp
// 恶意 URI 可能访问系统文件
Want want;
want.SetUri("../../../etc/passwd");
DataAbilityHelper::AcquireDataAbilityHelper(..., want, ...);
```

**代码证据**：
```cpp
// services/abilitymgr/src/uri_extension/uri_handler.cpp
// URI 处理逻辑需验证路径安全性
```

**影响**：
- 敏感文件泄露
- 配置篡改

**修复建议**：
- URI 规范化时拒绝 `..` 路径
- 白名单限制可访问的 URI 前缀
- 使用沙箱隔离

---

### 可利用点 6：热修复补丁加载

**风险等级**：高

**描述**：QuickFix 热修复机制可能加载恶意补丁。

**触发场景**：
```cpp
// 应用从非可信源加载热修复补丁
QuickFixManagerClient::GetInstance()->ApplyQuickFix(patchPath);
```

**代码证据**：
```cpp
// services/quickfixmgr/src/quick_fix_manager_service.cpp
// QuickFix 加载逻辑需验证签名
```

**影响**：
- 代码执行
- 恶意代码注入

**修复建议**：
- 强制要求补丁签名校验
- 限制补丁来源
- 沙箱执行修复代码

---

### 可利用点 7：Service 连接劫持

**风险等级**：中等

**描述**：恶意的 ServiceAbility 可能劫持正常的服务连接。

**触发场景**：
```cpp
// 伪装成目标 Service 接收连接
Intent intent;
intent.SetAction("ohos.ams.service.system.TARGET_SERVICE");
ConnectAbility(intent, maliciousConnection);
```

**代码证据**：
```cpp
// services/abilitymgr/src/ability_connect_manager.cpp
// 连接建立逻辑
```

**影响**：
- 中间人攻击
- 数据窃取

**修复建议**：
- 验证 Service 的 Bundle 签名
- 使用安全的 Intent 匹配规则
- 添加连接目标验证

---

### 可利用点 8：Token 伪造

**风险等级**：高

**描述**：通过伪造 AccessToken 绕过权限检查。

**触发场景**：
```cpp
// 伪造高权限 Token
uint32_t fakeTokenId = 0x12345678;
PermissionVerification::GetInstance()->VerifyPermissionByTokenId(fakeTokenId, permission);
```

**代码证据**：
```cpp
// services/abilitymgr/src/utils/update_caller_info_util.cpp
// Token 验证逻辑
IPCSkeleton::GetCallingTokenID()
```

**影响**：
- 完全权限绕过
- 系统控制

**修复建议**：
- 使用内核安全机制获取真实 Token
- Token 与进程绑定
- 定期更新 Token 验证机制

## 安全最佳实践

### 开发者建议

1. **输入验证**：所有外部输入必须验证
2. **最小权限**：只请求必要权限
3. **安全编码**：避免使用不安全的 API
4. **错误处理**：不要泄露敏感信息

### 架构建议

1. **纵深防御**：多层安全检查
2. **零信任**：默认不信任任何输入
3. **沙箱隔离**：限制组件权限范围
4. **审计日志**：记录安全相关操作

## 相关文档

- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
- [Inner API](05_Inner_API.md)

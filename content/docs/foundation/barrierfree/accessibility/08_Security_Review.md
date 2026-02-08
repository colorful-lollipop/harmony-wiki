# 安全风险评审

## 目的

本文档分析 Accessibility 子系统的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 安全审计人员
- 开发负责人
- 贡献者
- 需要理解安全考量的开发者

## 关键结论

### 攻击面清单

Accessibility 子系统存在以下攻击面：

1. **N-API 输入面** - 所有 JS API 调用
2. **IPC 接口面** - 跨进程 IPC 调用
3. **权限系统** - 权限检查机制
4. **敏感操作** - 手势注入、元素查询等

### 信任边界

```
[用户空间应用]
    │ (权限边界）
    ▼
[AccessibilityService SA 801]
    │ (Token ID 验证边界）
    ▼
[系统服务层] (Window Manager, Input Manager, etc.)
```

### 可被利用点

共 **5 类**潜在风险：

1. **Token ID 验证绕过** - 元素访问控制
2. **权限检查缺失** - 敏感操作
3. **输入验证不足** - 参数注入
4. **IPC 调用权限绕过** - 特殊进程白名单
5. **信息泄露** - 元素信息暴露

---

## 详细内容

### 攻击面分析

#### 1. N-API 输入面

**攻击向量**: 恶意应用通过 N-API 调用无障碍服务

**影响**:
- 查询任意应用的无障碍元素信息
- 执行无障碍操作
- 注入手势模拟用户操作

**证据**:
- N-API 模块: `interfaces/kits/napi/src/native_module.cpp`
- 所有 N-API 接口无输入过滤

**缓解措施**:
- 权限检查（`ACCESSIBILITY_EXTENSION_ABILITY`, `QUERY_ACCESSIBILITY_ELEMENT`）
- Token ID 验证
- 系统应用白名单

#### 2. IPC 接口面

**攻击向量**: 恶意应用直接通过 IPC 调用 AccessibilityService

**影响**:
- 绕过 N-API 层的权限检查
- 直接操作无障碍服务

**证据**:
- IPC 接口: `common/interface/include/iaccessible_ability_channel.h`
- Proxy 实现: `common/interface/include/accessible_ability_channel_proxy.h`

**缓解措施**:
- IPC 层也进行权限检查
- Binder 调用者身份验证

#### 3. 权限系统

**攻击向量**: 权限配置不当或权限检查漏洞

**影响**:
- 未授权应用获得敏感权限
- 权限提升

**证据**:
- 权限检查: `services/aams/src/accessible_ability_manager_service.cpp:1447`
- 权限定义: `frameworks/common/src/accessibility_constants.cpp:63-68`

**缓解措施**:
- 严格的权限声明检查
- 系统应用特殊处理
- Token ID 类型验证

### 信任边界

#### 用户空间 ↔ AccessibilityService

**边界类型**: 权限边界

**验证机制**:
1. 权限检查（`CheckPermission()`）
2. 系统应用检查（`IsSystemApp()`）
3. Token ID 验证（`VerifyingToKenId()`）

**证据**:
- `services/aams/src/accessible_ability_manager_service.cpp:1427-1457`

#### AccessibilityService ↔ 目标应用

**边界类型**: Token ID 验证边界

**验证机制**:
- Token ID 到窗口映射（`tokenIdMap_`）
- VerifyingToKenId() 验证调用者权限

**证据**:
- `services/aams/include/accessibility_window_connection.h:136`

#### AccessibilityService ↔ 系统服务

**边界类型**: 系统级服务边界

**验证机制**:
- SA 间调用通过标准接口
- 无直接访问内核

**证据**:
- `services/aams/include/accessibility_display_manager.h`
- `services/aams/include/accessibility_input_interceptor.h`

### 可被利用点

#### 1. Token ID 验证绕过

**问题**: Token ID 验证可能在某些场景下被绕过

**证据**: `services/aams/src/accessible_ability_manager_service.cpp:526-564` (VerifyingToKenId)

**触发条件**:
- 恶意应用获得有效的 Token ID
- 跨窗口/元素访问时验证逻辑存在漏洞

**影响**: 未授权访问其他应用的无障碍元素

**可利用路径**:
```
恶意应用
    ↓ (伪造/获取 Token ID)
调用 GetWindows / GetElementInfo
    ↓ (VerifyingToKenId 验证失败）
绕过验证
    ↓
访问目标应用元素
```

**修复建议**:
1. 加强 Token ID 到窗口/元素的映射验证
2. 添加额外的调用者身份验证（Bundle Name + UID）
3. 记录所有跨应用的元素访问

#### 2. 权限检查缺失

**问题**: 某些敏感操作可能缺少权限检查

**证据**: `services/aams/src/accessible_ability_channel.cpp`

**示例**: 查看所有 IPC 方法是否都有权限检查

**触发条件**:
- 恶意应用调用未检查权限的接口

**影响**: 执行未授权操作

**可利用路径**:
```
恶意应用
    ↓ (检查 N-API 文档）
找到可能缺少权限检查的接口
    ↓
通过 Proxy 直接调用
    ↓ (无权限检查）
执行操作
```

**修复建议**:
1. 审查所有 IPC 入口的权限检查
2. 使用自动化工具扫描缺失检查
3. 参考已检查接口的模式

#### 3. 输入验证不足

**问题**: N-API 参数可能缺乏严格验证

**证据**: `interfaces/kits/napi/src/*.cpp`

**示例**: 数组长度、字符串长度、数值范围

**触发条件**:
- 传递超长字符串或数组
- 传递超出范围的数值

**影响**:
- 内存溢出/缓冲区溢出
- 服务崩溃（拒绝服务）
- 信息泄露

**可利用路径**:
```
恶意应用
    ↓ (构造恶意输入）
传递超长参数
    ↓ (N-API 层不验证）
传入 IPC
    ↓ (服务层不验证）
内存溢出/崩溃
```

**修复建议**:
1. 在 N-API 层验证参数范围
2. 在 IPC Stub 层二次验证
3. 使用安全的字符串处理函数
4. 添加长度限制

#### 4. IPC 调用权限绕过

**问题**: 特殊进程白名单可能被滥用

**证据**: `frameworks/common/src/accessibility_permission.cpp:38-54` (IsStartByHdcd)

**触发条件**:
- 恶意应用伪装为 "hdcd" 进程

**影响**: 绕过某些权限检查

**可利用路径**:
```
恶意应用
    ↓ (伪造进程名或 Token）
获取特殊进程权限
    ↓
绕过权限检查
    ↓
访问敏感功能
```

**修复建议**:
1. 审查所有特殊进程白名单
2. 使用更严格的进程身份验证（签名、证书）
3. 定期审查白名单的必要性

#### 5. 信息泄露

**问题**: 无障碍元素信息可能暴露敏感信息

**证据**: `interfaces/innerkits/common/include/accessibility_element_info.h`

**触发条件**:
- 恶意应用查询其他应用的元素
- 元素包含敏感信息（密码、个人信息）

**影响**:
- 敏感信息泄露
- 隐私侵犯

**可利用路径**:
```
恶意应用
    ↓ (获得元素查询权限）
调用 GetElements / FindElement
    ↓
获取元素信息（包含文本、内容等）
    ↓
提取敏感信息
```

**修复建议**:
1. 实施元素级权限控制（某些元素需要额外权限）
2. 敏感内容标记（密码字段、敏感输入）
3. 记录所有元素访问
4. 提供用户控制（显示哪些信息）

### 安全最佳实践

#### 开发者

1. **最小权限原则**: 只申请必需的权限
2. **输入验证**: 始终验证用户输入
3. **敏感操作**: 特别关注手势注入、元素查询
4. **日志审计**: 记录敏感操作

#### 架构师

1. **权限审查**: 确保所有敏感操作有权限检查
2. **Token 验证**: 加强 Token ID 到资源的映射验证
3. **白名单审计**: 定期审查特殊进程白名单
4. **纵深防御**: 多层验证（N-API + IPC + Service）

#### 安全审计员

1. **威胁建模**: 定期进行威胁建模
2. **渗透测试**: 模拟攻击场景
3. **代码审计**: 审查权限检查和输入验证
4. **安全测试**: 使用模糊测试等工具

### 检查范围说明

#### 已覆盖范围

✅ N-API 接口
✅ IPC 接口
✅ 权限检查机制
✅ Token ID 验证
✅ 敏感操作
✅ 特殊进程白名单

#### 未覆盖范围

⚪ 内部服务间通信（未详细审查）
⚪ 第三方依赖的安全性（如 Bundle Manager, Input Manager）
⚪ 内核级输入注入（由 Input 系统负责）
⚪ 设备级安全特性（由安全子系统负责）

#### 局限性

1. **静态分析**: 本评审基于代码静态分析，未进行动态测试
2. **版本特定**: 基于 4.0 版本代码，不同版本可能存在差异
3. **依赖假设**: 假设底层系统服务（Window Manager, Input Manager 等）正确实施安全机制

---

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 接口](04_N-API.md)

---

最后更新: 2026-02-06

# 安全风险评审

## 目的

本文档提供EDM组件的安全风险评估，包括攻击面、信任边界和修复建议。

## 适用范围

- 目标读者：安全审计人员、架构师
- 覆盖内容：攻击面分析、潜在漏洞、修复建议
- 不包含：完整安全方案（仅建议）

## 关键结论

- EDM涉及多种攻击面：N-API、IPC、文件系统、系统参数
- 存在权限绕过、路径遍历、信息泄露等潜在风险
- 已有安全机制：权限检查、AccessToken验证、UID校验

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构和信任边界
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - N-API接口权限

---

## 检查范围

| 检查项 | 检查内容 | 检查结果 |
|---------|----------|---------|------|
| N-API接口参数校验 | JS参数类型和范围检查 | ✓ 已检查 |
| IPC接口参数校验 | IPC消息参数校验 | ✓ 已检查 |
| 文件路径操作安全 | 路径遍历检查 | ✓ 已检查 |
| 权限检查机制 | 调用者权限验证 | ✓ 已检查 |
| 输入验证 | 外部输入完整性检查 | ⚠️ 部分缺失 |
| 内存安全 | 缓冲区操作、内存管理 | ✓ 已检查 |

---

## 攻击面清单

### 1. N-API接口（应用层）

| 攻击面 | 具体位置 | 风险等级 |
|---------|----------|---------|
| JS参数注入 | interfaces/kits/*/src/*_addon.cpp | 中 |
| 策略值篡改 | NAPI层策略设置接口 | 中 |
| 未授权管理员激活 | admin_manager_addon.cpp | 低（有权限检查） |

**证据**：`interfaces/kits/admin_manager/src/admin_manager_addon.cpp:56-66`

### 2. IPC接口（跨进程）

| 攻击面 | 具体位置 | 风险等级 |
|---------|----------|---------|
| IPC消息伪造 | enterprise_device_mgr_stub.cpp | 中（有Token检查） |
| 权限绕过 | enterprise_device_mgr_ability.cpp:2060-2117 | 中（多层检查） |
| 跨用户攻击 | policy_manager.cpp | 低（有用户隔离） |

**证据**：`services/edm/src/enterprise_device_mgr_ability.cpp:2060`, `services/edm/src/permission_checker.cpp:249-285`

### 3. 文件系统操作

| 攻击面 | 具体位置 | 风险等级 |
|---------|----------|---------|
| 路径遍历 | RDB操作、证书安装 | 高（缺少验证） |
| 任意文件读写 | 用户证书操作 | 高（仅文件名检查） |

**证据**：`services/edm/src/admin_policies_storage_rdb.cpp`（RDB操作无路径验证）

### 4. 系统参数操作

| 攻击面 | 具体位置 | 风险等级 |
|---------|----------|---------|
| 参数篡改 | etc/param/edm.para | 中（需root权限） |
| 参数读取泄露 | EDM参数读取 | 低 |

**证据**：`etc/param/edm.para`（系统参数可被任何进程读取）

### 5. 敏感信息泄露

| 攻击面 | 具体位置 | 风险等级 |
|---------|----------|---------|
| 日志信息泄露 | EDMLOG宏日志输出 | 中（包含敏感信息） |
| 错误信息泄露 | 错误返回详情 | 低 |

**证据**：各模块的`EDMLOGI`、`EDMLOGE`宏使用

---

## 可被利用点

### 风险1: 用户证书路径遍历

**位置**: `services/edm/src/admin_policies_storage_rdb.cpp` 和证书安装插件

**证据**：
```cpp
// 证书安装插件中缺少路径验证
InstallUserCertificate(certBytes)
    // 未经验证的certBytes可能包含"../../../"路径
```

**风险等级**: 高

**利用路径**: 恶意MDM应用 → 安装含路径遍历的用户证书 → 访问任意文件

**影响**: 
- 可能读取系统任意文件
- 可能覆盖系统文件
- 窃取敏感信息

**修复建议**:
1. 在所有文件路径操作前添加路径验证
2. 检查并规范化路径，禁止`../`和`./`
3. 使用系统提供的路径规范API（如NormalizePath）

**修复代码示例**:
```cpp
// 添加路径验证
bool IsValidPath(const std::string &path) {
    if (path.find("..") != std::string::npos ||
        path.find("//") != std::string::npos) {
        return false;
    }
    return true;
}
```

---

### 风险2: 策略数据注入（JSON）

**位置**: `services/edm/src/policy_manager.cpp` 和各策略插件

**证据**：
```cpp
// 策略数据序列化使用cJSON
SetPolicy(policyData)
    // policyData来自外部输入，可能包含恶意JSON
```

**风险等级**: 中

**利用路径**: 恶意MDM应用 → 构造恶意策略JSON → 注入到EDM服务 → 破坏策略解析

**影响**:
- 策略解析失败导致服务异常
- 注入自定义配置可能绕过安全限制
- DoS攻击（超大JSON导致解析耗尽资源）

**修复建议**:
1. 限制JSON数据大小（如max 1MB）
2. 使用安全的JSON解析器（深度限制）
3. 添加JSON schema验证
4. 验证策略结构完整性

**修复代码示例**:
```cpp
// 限制策略数据大小
constexpr size_t MAX_POLICY_SIZE = 1024 * 1024; // 1MB

if (policyData.size() > MAX_POLICY_SIZE) {
    EDMLOGE("Policy data size exceeds limit");
    return ERR_PARAM_ERROR;
}
```

---

### 风险3: 权限检查绕过（竞态条件）

**位置**: `services/edm/src/permission_checker.cpp`

**证据**:
```cpp
// CheckCallingUid中的时间窗口风险
ErrCode CheckCallingUid(const std::string &bundleName)
{
    // 获取UID后，BundleManager可能在不同时间返回不同名称
    int uid = IPCSkeleton::GetCallingUid();
    std::string callingBundleName;
    BundleManager::GetNameForUid(uid, callingBundleName);
    // 如果BundleName在UID查询和校验之间变化，可能导致绕过
    if (bundleName == callingBundleName) {
        return ERR_OK;
    }
}
```

**风险等级**: 中

**利用路径**: 恶意应用 → 在UID查询和校验之间修改Bundle → 绕过权限检查 → 访问受限资源

**影响**:
- 非授权应用获取EDM权限
- 设置企业策略
- 绕过管理员限制

**修复建议**:
1. 在权限检查内部重新获取UID，避免时序问题
2. 使用原子操作检查权限
3. 在一次调用中完成UID获取和校验

**修复代码示例**:
```cpp
// 原子UID获取和校验（可能竞态）
int uid = IPCSkeleton::GetCallingUid();
std::string name1 = BundleManager::GetNameForUid(uid);
if (bundleName != name1) return ERR_OK;

// 修复：原子化检查
struct PermissionCheckResult {
    int uid;
    std::string bundleName;
    bool isValid;
};
PermissionCheckResult result;
result.uid = IPCSkeleton::GetCallingUid();
result.bundleName = BundleManager::GetNameForUid(result.uid);
result.isValid = (result.bundleName == bundleName);
return result.isValid ? ERR_OK : ERR_EDM_PERMISSION_ERROR;
```

---

### 风险4: 敏感信息泄露（日志）

**位置**: 所有`*_addon.cpp`文件

**证据**：
```cpp
// 日志可能包含敏感信息
EDMLOGI("EnableAdmin called, packageName: %{public}s, adminType: %{public}d",
    packageName.c_str(), adminType);
// 日志中可能输出设备序列号、企业信息等
```

**风险等级**: 中

**利用路径**: 攻击者读取系统日志 → 提取企业设备信息 → 窃取敏感数据

**影响**:
- 泄露设备序列号
- 泄露企业信息
- 泄露管理员信息

**修复建议**:
1. 敏感信息脱敏：输出hash或截断
2. 在生产环境关闭详细日志
3. 使用分级日志系统（Debug/info/error）
4. 敏感信息不输出到日志

**修复代码示例**:
```cpp
// 原代码：可能输出完整信息
EDMLOGI("Serial: %{public}s", serial.c_str());

// 修复：脱敏处理
std::string maskedSerial = (serial.length() > 4) ?
    (serial.substr(0, 2) + "**") : serial;
EDMLOGI("Serial: %{public}s", maskedSerial.c_str());
```

---

### 风险5: 插件dlopen任意库加载

**位置**: `services/edm/src/plugin_manager.cpp`

**证据**：
```cpp
// 插件SO文件名来源于funcCode映射
GetSoNameByCode(funcCode)
{
    // 如果funcCode被篡改，可能加载任意SO库
    // 缺少文件名白名单验证
    return soName;
}
```

**风险等级**: 中

**利用路径**: 恶意应用（需调试权限或root） → 修改funcCode映射 → 加载恶意插件 → 执行任意代码

**影响**:
- 加载非EDM官方插件
- 执行未授权代码
- 破坏EDM服务稳定性

**修复建议**:
1. 维护插件文件名白名单
2. 验证插件签名
3. 限制可加载插件路径（必须是/system/lib/edm_plugin/）
4. 仅从可信目录加载插件

**修复代码示例**:
```cpp
// 插件白名单
const std::set<std::string> ALLOWED_PLUGINS = {
    "libdevice_core_plugin.so",
    "libcommunication_plugin.so",
    "libsys_service_plugin.so",
    "libneed_extra_plugin.so"
};

bool IsAllowedPlugin(const std::string &soName) {
    return ALLOWED_PLUGINS.find(soName) != ALLOWED_PLUGINS.end();
}
```

---

## 信任边界

### 边界1: 不可信环境 ↔ EDM受控环境

```
┌──────────────────────────────────┐
│     不可信环境                 │
│  (普通三方应用）               │
└────────────┬──────────────────────┘
             │ 受限访问
             ▼
┌──────────────────────────────────┐
│     EDM受控环境                 │
│  (EDM服务、EDM管理应用）       │
└────────────┬──────────────────────┘
             │ 系统调用
             ▼
┌──────────────────────────────────┐
│     系统服务环境                 │
│  (BundleManager、WiFiManager等）  │
└───────────────────────────────────┘
```

**保护机制**:
- AccessToken权限验证
- UID匹配检查
- EDM_UID（3057）特权保护
- 系统服务UID白名单（蓝牙、用户认证等）

**证据**：`services/edm/src/permission_checker.cpp:180-285`, `services/edm/include/permission_checker.h:33-77`

### 边界2: 管理员权限层级

```
Super Device Admin (ENT, 权限最高)
       │
       │ 激活/授权
       ▼
Normal Device Admin (NORMAL, 权限中等)
       │
       │ 执行策略（需委托权限）
       ▼
BYOD Device Admin (BYOD, 权限中等)
       │
       │ BYOD场景限制
       ▼
普通三方应用 (无EDM权限，权限最低)
```

**权限控制证据**：
- Super Admin可激活/禁用Normal Admin
- Super Admin可授权特定策略给Normal Admin
- Normal Admin只能执行有权限的策略
- 不同类型的管理员权限隔离

---

## 检查局限性与说明

### 检查范围

| 检查项 | 范围 | 说明 |
|---------|------|------|
| 输入验证 | 覆盖所有外部输入点 | JS参数、IPC消息、文件路径 |
| 权限检查 | 覆盖权限验证逻辑 | AccessToken、UID、管理员类型 |
| 内存安全 | 覆盖常见内存操作 | 缓冲区、字符串操作 |
| 信息泄露 | 覆盖日志和错误信息 | 敏感信息检查 |

### 未覆盖内容

- 插件代码的详细审计（仅接口层）
- 网络通信安全性（SSL/TLS）
- 加密算法强度分析
- 并发安全（多线程同步）

### 建议

1. **安全开发流程**: 在新增策略时进行安全评审
2. **安全测试**: 对新增N-API进行模糊测试
3. **渗透测试**: 定期进行安全渗透测试
4. **代码审计**: 定期进行安全代码审计

---

## 安全最佳实践

### 开发者建议

| 实践 | 说明 | 优先级 |
|------|------|--------|
| 始终校验输入 | 所有外部输入必须验证 | 高 |
| 使用系统安全API | 避免自己实现安全功能 | 高 |
| 最小权限原则 | 只请求必要的权限 | 高 |
| 敏感信息保护 | 不输出到日志，存储加密 | 中 |
| 错误处理 | 详细的错误信息用于调试，简化的错误信息用于用户 | 中 |

### 运维建议

| 实践 | 说明 | 优先级 |
|------|------|--------|
| 定期更新 | 及时应用安全补丁 | 高 |
| 日志监控 | 监控异常行为 | 中 |
| 权限审查 | 定期审查管理员权限 | 中 |
| 插件审计 | 审查已加载插件 | 中 |

---

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构和信任边界
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - 权限定义
- [09_Troubleshooting.md](09_Troubleshooting.md) - 安全事件处理

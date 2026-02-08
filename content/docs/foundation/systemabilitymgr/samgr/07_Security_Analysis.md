# 安全风险分析

## 概述

本报告基于代码证据对 Samgr 组件进行安全分析，识别潜在的攻击面、信任边界和可利用点。

**分析范围**:
- 接口层: `interfaces/innerkits/`
- 服务实现: `services/samgr/native/`
- 客户端框架: `frameworks/native/`
- 配置文件解析: `services/common/`

**排除范围**:
- 测试代码 (`test/`, `unittest/`)
- 示例代码 (`examples/`)

## 攻击面清单

### 1. IPC 接口攻击面

| 接口 | 风险等级 | 说明 |
|------|----------|------|
| `AddSystemAbility` | 高 | 服务注册，可能被恶意注册假服务 |
| `GetSystemAbility` | 中 | 服务查询，可能泄露服务信息 |
| `LoadSystemAbility` | 高 | 触发进程启动，可能被滥用 |
| `SubscribeSystemAbility` | 低 | 状态订阅，可能导致 DoS |
| `RemoveSystemAbility` | 高 | 服务注销，可能导致服务不可用 |

### 2. 文件系统攻击面

| 路径 | 风险等级 | 说明 |
|------|----------|------|
| `/system/profile/*.json` | 中 | SA 配置文件解析 |
| `/system/etc/param/samgr.para` | 低 | 系统参数 |

### 3. 网络攻击面

| 接口 | 风险等级 | 说明 |
|------|----------|------|
| `GetSystemAbility(deviceId)` | 高 | 分布式服务发现 |
| `DBinder` 服务 | 高 | 跨设备 RPC |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐         ┌─────────────┐                      │
│   │  普通应用    │         │  系统服务    │                      │
│   │  (不可信)   │         │  (可信)     │                      │
│   └──────┬──────┘         └──────┬──────┘                      │
│          │                       │                              │
│          │ 信任边界 1            │                              │
│          ▼                       ▼                              │
│   ┌─────────────────────────────────────┐                      │
│   │         Samgr IPC 接口              │                      │
│   │   - GetSystemAbility                │                      │
│   │   - SubscribeSystemAbility          │                      │
│   └─────────────────────────────────────┘                      │
│          │                                                      │
│          │ 信任边界 2                                           │
│          ▼                                                      │
│   ┌─────────────────────────────────────┐                      │
│   │      Samgr 内部实现                 │                      │
│   │   - abilityMap_ (受锁保护)          │                      │
│   │   - SELinux 权限检查                │                      │
│   └─────────────────────────────────────┘                      │
│          │                                                      │
│          │ 信任边界 3                                           │
│          ▼                                                      │
│   ┌─────────────────────────────────────┐                      │
│   │      本地能力管理器 (LSAMgr)        │                      │
│   │   - 进程启动/停止                   │                      │
│   └─────────────────────────────────────┘                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 可被利用点分析

### 利用点 1: 服务注册伪造

**风险等级**: 🔴 高

**证据**:
```cpp
// system_ability_manager.cpp:1042
int32_t SystemAbilityManager::AddSystemAbility(int32_t systemAbilityId,
    const sptr<IRemoteObject>& ability, const SAExtraProp& extraProp) {
    if (!CheckInputSysAbilityId(systemAbilityId) || ability == nullptr) {
        return ERR_INVALID_VALUE;
    }
    // 重复注册会覆盖
    abilityMap_[systemAbilityId] = std::move(saInfo);
}
```

**利用路径**:
1. 攻击者获取 `AddSystemAbility` 权限
2. 注册伪造服务覆盖真实服务
3. 其他应用连接到伪造服务
4. 中间人攻击或信息窃取

**影响**:
- 服务劫持
- 信息泄露
- 权限提升

**修复建议**:
1. 加强 `AddSystemAbility` 权限检查，限制仅允许目标进程注册
2. 添加注册白名单机制
3. 对重复注册进行更严格的审计

---

### 利用点 2: 路径遍历攻击

**风险等级**: 🟡 中

**证据**:
```cpp
// services/common/src/parse_util.cpp:230
bool ParseUtil::LoadSaProfiles(const std::string& profilePath) {
    string realPath = GetRealPath(profilePath);
    if (CheckPathExist(realPath)) {
        return LoadSaProfilesFromJson(realPath);
    }
}
```

**分析**:
- 使用了 `realpath()` 进行路径规范化，防护有效
- 但 `CheckPathExist()` 存在 TOCTOU 窗口

**潜在利用**:
```cpp
// CheckPathExist 和实际打开之间有时间窗口
bool ParseUtil::CheckPathExist(const string& profilePath) {
    std::ifstream profileStream(profilePath.c_str());
    return profileStream.good();  // 检查后
}
// ... 实际使用 profilePath 打开文件
```

**影响**:
- 配置文件替换
- 恶意 SA 配置注入

**修复建议**:
1. 使用文件描述符传递，避免 TOCTOU
2. 在打开文件后立即验证文件属性

---

### 利用点 3: 整数溢出 (SA ID)

**风险等级**: 🟢 低 (已防护)

**证据**:
```cpp
// if_system_ability_manager.h:382
bool CheckInputSysAbilityId(int32_t sysAbilityId) const {
    if (sysAbilityId >= FIRST_SYS_ABILITY_ID && 
        sysAbilityId <= LAST_SYS_ABILITY_ID) {
        return true;
    }
    return false;
}
```

**分析**:
- `LAST_SYS_ABILITY_ID = 0x00ffffff` (16777215)
- 在 int32_t 范围内，无溢出风险
- 已实施边界检查

**状态**: ✅ 安全

---

### 利用点 4: 资源耗尽 (DoS)

**风险等级**: 🟡 中

**证据**:
```cpp
// system_ability_manager.cpp:1054
auto saSize = abilityMap_.size();
if (saSize >= MAX_SERVICES) {
    return ERR_INVALID_VALUE;
}
```

```cpp
// system_ability_manager.cpp:907
int32_t SystemAbilityManager::SubscribeSystemAbility(...) {
    // 缺少对 listenerMap_ 大小的限制
}
```

**分析**:
- `abilityMap_` 有 `MAX_SERVICES` 限制
- 但 `listenerMap_` 没有明确的大小限制
- 恶意应用可能通过大量订阅导致内存耗尽

**利用路径**:
1. 创建大量进程
2. 每个进程订阅大量 SA
3. 内存耗尽导致 OOM

**修复建议**:
```cpp
// 建议添加订阅数量限制
static constexpr int32_t MAX_SUBSCRIBE_PER_PROCESS = 256;
int32_t GetSubscribeCount(int32_t callingPid) {
    // 统计该进程的订阅数
}
```

---

### 利用点 5: 动态加载滥用

**风险等级**: 🔴 高

**证据**:
```cpp
// system_ability_manager_stub.cpp:567
case static_cast<uint32_t>(SamgrInterfaceCode::LOAD_SYSTEM_ABILITY_TRANSACTION): {
    if (!CheckPermission(ACCESS_PERMISSION)) {
        return ERR_PERMISSION_DENIED;
    }
    return LoadSystemAbility(systemAbilityId, callback);
}
```

**分析**:
- `LoadSystemAbility` 可触发进程启动
- 频繁调用可能导致资源耗尽
- 无速率限制

**利用路径**:
1. 频繁调用 `LoadSystemAbility`
2. 系统不断启动新进程
3. CPU/内存资源耗尽

**修复建议**:
```cpp
// 添加速率限制
class RateLimiter {
    bool CheckRateLimit(int32_t saId, int32_t callingPid) {
        // 实现令牌桶或滑动窗口
    }
};
```

---

### 利用点 6: 权限检查绕过

**风险等级**: 🟡 中

**证据**:
```cpp
// system_ability_manager_util.cpp:108
bool CheckDistributedPermission() {
    auto callingUid = IPCSkeleton::GetCallingUid();
    if (callingUid != UID_ROOT && callingUid != UID_SYSTEM) {
        return false;
    }
    return true;
}
```

**分析**:
- 仅检查 UID，未检查 capabilities
- 如果普通应用获取 UID 1000，可能绕过检查

**修复建议**:
```cpp
// 建议同时检查 capabilities
bool CheckDistributedPermission() {
    auto callingUid = IPCSkeleton::GetCallingUid();
    if (callingUid != UID_ROOT && callingUid != UID_SYSTEM) {
        return false;
    }
    // 额外检查 CAP_SYS_ADMIN 或类似 capability
    return CheckCapability(CAP_SYS_ADMIN);
}
```

---

### 利用点 7: JSON 解析风险

**风险等级**: 🟡 中

**证据**:
```cpp
// parse_util.cpp 多处使用 nlohmann::json
nlohmann::json profileJson = nlohmann::json::parse(profileStream);
```

**潜在风险**:
- 嵌套层级过深导致栈溢出
- 超大 JSON 文件导致内存耗尽
- 虽然设置了 `MAX_JSON_OBJECT_SIZE`，但某些字段未限制

**修复建议**:
```cpp
// 设置 JSON 解析限制
auto profileJson = nlohmann::json::parse(profileStream, nullptr, false, 
    true,  // allow_exceptions
    50,    // max_depth
    1024*1024  // max_bytes
);
```

---

### 利用点 8: 竞态条件

**风险等级**: 🟢 低

**证据**:
```cpp
// system_ability_manager.cpp:1055-1060
{
    unique_lock<shared_mutex> writeLock(abilityMapLock_);
    if (abilityMap_.count(systemAbilityId) > 0) {
        HILOGI("SA already exists, update it: %{public}d", systemAbilityId);
    }
    abilityMap_[systemAbilityId] = std::move(saInfo);
}
```

**分析**:
- 使用 RAII 锁管理，无明显的 UAF 或双释放
- 锁粒度合理
- 但 `CheckSystemAbility` 和 `AddSystemAbility` 之间无原子性保证

**状态**: ✅ 基本安全，但建议加强注释

---

### 利用点 9: 日志注入

**风险等级**: 🟢 低

**证据**:
```cpp
// 多处日志直接打印用户输入
HILOGI("insert %{public}d. size : %{public}zu", systemAbilityId, abilityMap_.size());
```

**分析**:
- 使用 `%{public}` 标记，日志可输出
- 但所有输入都经过校验，无格式字符串漏洞

**状态**: ✅ 安全

---

### 利用点 10: 静态初始化顺序

**风险等级**: 🟢 低

**证据**:
```cpp
// system_ability_manager_util.cpp:37
void* SamgrUtil::penglaiFunc_ = InitPenglaiFunc();
```

**分析**:
- 静态全局变量初始化顺序不确定
- 但 `InitPenglaiFunc()` 内部有 null 检查

**状态**: ✅ 风险可控

## 安全加固建议

### 立即修复 (高优先级)

1. **添加订阅数量限制**
   ```cpp
   // system_ability_manager.cpp
   static constexpr int32_t MAX_LISTENERS_PER_PROCESS = 256;
   ```

2. **添加加载速率限制**
   ```cpp
   // system_ability_manager_stub.cpp
   RateLimiter loadLimiter;
   if (!loadLimiter.CheckLimit(callingPid)) {
       return ERR_RATE_LIMITED;
   }
   ```

3. **加强注册权限验证**
   ```cpp
   // 验证调用者是否是配置文件中的 process
   if (!CheckCallerIsSaOwner(saId)) {
       return ERR_PERMISSION_DENIED;
   }
   ```

### 中期改进 (中优先级)

1. 统一使用文件描述符传递避免 TOCTOU
2. 添加 JSON 解析深度限制
3. 增强分布式权限检查 (capabilities)

### 长期规划 (低优先级)

1. 代码审计工具集成
2. 模糊测试覆盖率提升
3. 安全测试自动化

## 检查局限性说明

1. **未分析**:
   - 配置文件的完整性校验机制
   - SELinux 策略的完整覆盖
   - DBinder 的分布式安全

2. **假设**:
   - 假设 IPC 框架本身安全
   - 假设文件系统权限正确配置
   - 假设 init 进程可信

3. **建议进一步分析**:
   - 完整的 SELinux 策略审计
   - 分布式场景下的中间人攻击
   - 硬件抽象层的安全边界

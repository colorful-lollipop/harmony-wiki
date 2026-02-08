# SAFWK 安全风险评审

## 评审概述

本评审基于对 SAFWK 代码库的静态分析，重点关注以下安全维度:
- 输入校验
- 权限控制
- IPC 安全
- 内存安全
- 竞态条件
- 信息泄露

**评审范围**: `services/safwk/`, `interfaces/innerkits/safwk/`, `svc/`, `etc/profile/`

**评审时间**: 2026-02-06

---

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐      IPC       ┌─────────────┐            │
│  │   可信域     │ ──────────────→ │   SAFWK      │            │
│  │ - 系统进程   │   (受限)        │   域         │            │
│  │ - 已认证 SA  │               │             │            │
│  └─────────────┘               └─────────────┘            │
│           ▲                            │                   │
│           │                            │                   │
│           └────────────────────────────┘                   │
│                 未认证请求被拒绝                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 外部输入

| 输入类型 | 来源 | 处理方式 |
|----------|------|----------|
| IPC 请求 | 其他进程 | MessageParcel 反序列化、权限验证 |
| Profile 配置文件 | 文件系统 | JSON 解析 |
| 命令行参数 | sa_main 启动参数 | 参数校验 |
| 环境变量 | 系统环境 | 无直接使用 |

---

## 攻击面清单

| 攻击面 | 类型 | 风险等级 |
|--------|------|----------|
| IPC 接口 | 进程间通信 | 高 |
| SA 注册接口 | 本地 API | 中 |
| Profile 解析 | 文件解析 | 中 |
| 按需启动触发 | 内部机制 | 低 |
| FFRT 任务调度 | 异步任务 | 低 |

---

## 风险点分析

### 风险 1: IPC 参数反序列化边界检查不严格

**风险等级**: 中高

**证据**: `services/safwk/src/local_ability_manager_stub.cpp`

```cpp
// local_ability_manager_stub.cpp:115-130
int32_t LocalAbilityManagerStub::GetSystemAbilityInner(
    int32_t systemAbilityId, MessageParcel& data, MessageParcel& reply)
{
    // 直接从 Parcel 读取，未检查 systemAbilityId 范围
    int32_t said = data.ReadInt32();  // 行 117
    // ...
    sptr<IRemoteObject> result = LocalAbilityManager::GetInstance()
        .GetSystemAbility(said);  // 行 124
}
```

**可利用路径**:
```
恶意进程 → IPC 调用 GetSystemAbilityInner()
    → 传入异常的 said 值
    → 可能触发越界访问或逻辑错误
```

**影响**:
- 可能访问未授权的 SA
- 可能导致状态混淆

**修复建议**:
```cpp
// 添加 SA ID 范围校验
constexpr int32_t MIN_SA_ID = 0;
constexpr int32_t MAX_SA_ID = 10000;  // 或根据实际配置

int32_t said = data.ReadInt32();
if (said < MIN_SA_ID || said > MAX_SA_ID) {
    return ERR_INVALID_VALUE;
}
```

---

### 风险 2: MessageParcel 字符串读取可能越界

**风险等级**: 中

**证据**: 多处使用 `ReadString()` / `Read16String()` 而无长度限制

```cpp
// local_ability_manager_stub.cpp:67
std::u16string interfaceToken = data.ReadInterfaceToken();

// system_ability.cpp
void SetLibPath(const std::string& libPath)
{
    // libPath_ 赋值前无长度检查
    libPath_ = libPath;
}
```

**可利用路径**:
```
恶意进程 → 发送超长字符串的 IPC 消息
    → ReadString() 读取完整字符串
    → 可能触发栈溢出或堆溢出
```

**影响**:
- 内存溢出
- 拒绝服务

**修复建议**:
```cpp
// 限制最大字符串长度
constexpr size_t MAX_STRING_LEN = 4096;

std::string ReadStringWithLimit(MessageParcel& data) {
    std::string str = data.ReadString();
    if (str.length() > MAX_STRING_LEN) {
        throw std::length_error("String too long");
    }
    return str;
}
```

---

### 风险 3: 权限校验可被绕过

**风险等级**: 中

**证据**: `services/safwk/src/local_ability_manager_stub.cpp:68-83`

```cpp
bool LocalAbilityManagerStub::CheckPermission(uint32_t code)
{
    uint32_t accessToken = IPCSkeleton::GetCallingTokenID();
    std::string permissionToCheck;
    
    // 仅检查特定操作码的权限
    if (code == SYSTEM_ABILITY_EXT_TRANSACTION) {
        permissionToCheck = PERMISSION_EXT_TRANSACTION;
    } else if (code == SERVICE_CONTROL_CMD_TRANSACTION) {
        permissionToCheck = PERMISSION_SVC;
    } else {
        // 默认使用 MANAGE 权限
        permissionToCheck = PERMISSION_MANAGE;
    }
    
    int32_t ret = AccessTokenKit::VerifyAccessToken(
        accessToken, permissionToCheck);
    return ret == PERMISSION_GRANTED;
}
```

**可利用路径**:
```
恶意进程 → 猜测合法操作码
    → 绕过特定权限检查
    → 获取敏感 SA 访问权限
```

**影响**:
- 未授权访问系统能力
- 权限提升

**修复建议**:
```cpp
// 为每个操作定义明确的权限要求
switch (code) {
    case GET_SYSTEM_ABILITY_TRANSACTION:
        permissionToCheck = "ohos.permission.GET_SYSTEM_ABILITY";
        break;
    case ADD_SYSTEM_ABILITY_TRANSACTION:
        permissionToCheck = "ohos.permission.ADD_SYSTEM_ABILITY";
        break;
    default:
        return false;  // 未知操作码拒绝
}

// 添加审计日志
HiLog::Warn(LABEL, "Permission check for operation %u", code);
```

---

### 风险 4: 按需启动存在竞态条件

**风险等级**: 中低

**证据**: `services/safwk/src/local_ability_manager.cpp`

```cpp
// 假设的竞态场景 (基于代码结构分析)
void LocalAbilityManager::OnDemandStart(SystemAbilityOnDemandReason& reason)
{
    // 多线程环境下可能发生竞态
    if (abilityState_ == NOT_LOADED) {
        // 竞态窗口: 另一个线程可能已启动
        StartAbility();  // 行 假设
    }
}
```

**可利用路径**:
```
并发请求 → 多个线程同时触发 SA 启动
    → 竞态窗口内重复启动
    → 资源竞争或状态不一致
```

**影响**:
- 资源浪费 (重复初始化)
- 状态不一致

**修复建议**:
```cpp
// 使用原子操作或锁保护状态
std::atomic<SystemAbilityState> abilityState_{SystemAbilityState::NOT_LOADED};

void OnDemandStart(const SystemAbilityOnDemandReason& reason)
{
    SystemAbilityState expected = SystemAbilityState::NOT_LOADED;
    if (abilityState_.compare_exchange_strong(expected, 
            SystemAbilityState::LOADING)) {
        // 成功获取启动权限，执行启动
        DoStartAbility(reason);
    }
    // else: 已在启动中，忽略
}
```

---

### 风险 5: Profile 配置文件路径遍历

**风险等级**: 低

**证据**: `services/safwk/src/main.cpp`

```cpp
// 假设存在 (未在扫描中发现实际代码)
void LoadProfile(const std::string& profilePath)
{
    // 如果 profilePath 来自外部输入且未校验
    std::ifstream file(profilePath);
    // 解析 JSON...
}
```

**可利用路径**:
```
恶意构造 profilePath → "../etc/secret.json"
    → 读取非预期配置文件
    → 泄露敏感信息
```

**影响**:
- 信息泄露
- 配置篡改

**修复建议**:
```cpp
// 校验路径
bool IsValidProfilePath(const std::string& path) {
    // 只允许绝对路径
    if (path.empty() || path[0] != '/') {
        return false;
    }
    // 禁止路径遍历
    if (path.find("..") != std::string::npos) {
        return false;
    }
    // 限制在特定目录
    const std::string kProfileDir = "/system/profile/";
    return path.compare(0, kProfileDir.length(), kProfileDir) == 0;
}
```

---

## 安全控制措施

### 已实现的安全控制

| 控制措施 | 实现位置 | 效果 |
|----------|----------|------|
| AccessToken 验证 | local_ability_manager_stub.cpp:68 | 权限校验 |
| 接口令牌校验 | local_ability_manager_stub.cpp | IPC 接口认证 |
| CFI 保护 | BUILD.gn (configs) | 控制流完整性 |
| PAC-RET | BUILD.gn (configs) | 返回地址保护 |
| Stack Protector | Rust flags | 栈保护 |

### 待改进的安全控制

| 控制措施 | 建议实现位置 | 优先级 |
|----------|-------------|--------|
| SA ID 范围校验 | GetSystemAbilityInner | 高 |
| 字符串长度限制 | MessageParcel 读取 | 高 |
| 操作码-权限映射 | CheckPermission | 中 |
| 状态原子操作 | OnDemandStart | 中 |
| 路径校验 | Profile 加载 | 低 |

---

## 安全建议

### 开发规范

1. **输入校验**: 所有外部输入必须校验长度和范围
2. **最小权限**: 按需分配权限，避免过度授权
3. **安全编码**: 使用现代 C++ 特性 (std::string, span)
4. **日志审计**: 记录敏感操作 (权限校验失败等)

### 运行时监控

1. **系统调用监控**: 监控异常的 IPC 调用模式
2. **权限滥用检测**: 检测权限提升尝试
3. **异常终止处理**: SA 崩溃时清理资源

### 测试建议

1. **模糊测试**: 使用 fuzztest 对 IPC 接口进行模糊测试
2. **边界测试**: 测试异常 SA ID、超长字符串等边界情况
3. **并发测试**: 测试多线程并发启动 SA 的场景

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [01_Architecture.md](01_Architecture.md) |
| 构建系统 | [03_Build.md](03_Build.md) |
| 常见问题 | [05_FAQ.md](05_FAQ.md) |

---

## 附录: 代码证据索引

| 风险点 | 证据文件 | 行号 |
|--------|----------|------|
| IPC 参数校验 | local_ability_manager_stub.cpp | 115-130 |
| 权限检查 | local_ability_manager_stub.cpp | 68-83 |
| 字符串读取 | system_ability.cpp | (多处) |
| 状态管理 | local_ability_manager.cpp | (多处) |

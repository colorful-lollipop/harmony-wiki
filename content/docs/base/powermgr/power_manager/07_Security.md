# 安全风险评审

本文档对 Power Manager 模块进行安全风险分析，包括攻击面、信任边界和已知风险点。

## 1. 攻击面分析

### 1.1 外部输入点

| 输入源 | 类型 | 位置 | 说明 |
|--------|------|------|------|
| **N-API** | JS 参数 | `frameworks/napi/power/*.cpp` | 应用层 API 调用 |
| **IPC** | 序列化数据 | `services/zidl/src/*.cpp` | 跨进程通信 |
| **Shell** | 命令参数 | `utils/shell/*.cpp` | 调试命令 |
| **配置** | JSON/XML | `services/native/profile/*.json` | 配置文件 |
| **系统参数** | sysparam | `utils/param/*.cpp` | 系统属性 |

### 1.2 敏感操作

| 操作 | 权限要求 | 影响范围 |
|------|----------|----------|
| `shutdown/reboot` | `ohos.permission.SHUTDOWN` | 整机关机/重启 |
| `setPowerMode` | `ohos.permission.POWER_MANAGER` | 电源模式变更 |
| `wakeup` | 无 | 设备唤醒 |
| `createRunningLock` | 自动检查 UID | 系统保持唤醒 |
| `Hibernate` | 系统权限 | 深度休眠 |

---

## 2. 信任边界

### 2.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    应用层 (User Space)                     │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐             │  │
│  │  │可信应用   │  │普通应用  │  │不可信应用│             │  │
│  │  │(系统签名) │  │(三方签名) │  │(无签名)  │             │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘             │  │
│  └───────┼──────────────┼──────────────┼─────────────────────┘  │
│          │              │              │                        │
│          ▼              ▼              ▼                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    N-API 层                             │  │
│  │              参数校验 → 权限检查 → IPC 转发              │  │
│  └────────────────────────┬────────────────────────────────┘  │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    IPC 层 (Binder)                      │  │
│  │              UID 验证 → 调用链追踪 → 审计日志             │  │
│  └────────────────────────┬────────────────────────────────┘  │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   服务层 (System)                       │  │
│  │              业务逻辑 → 状态机 → HDI 调用               │  │
│  └─────────────────────────────────────────────────────────┘  │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   HDI 层 (Kernel)                      │  │
│  │              驱动调用 → 硬件操作                         │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 信任边界说明

| 边界 | 信任级别 | 说明 |
|------|----------|------|
| **应用 → N-API** | 中 | 需参数校验 |
| **N-API → IPC** | 高 | 需 UID 验证 |
| **IPC → Service** | 高 | 需权限检查 |
| **Service → HDI** | 最高 | 系统级操作 |

---

## 3. 安全风险清单

### 3.1 输入验证风险

#### 风险点 1: 关机/重启原因字符串未校验

**风险等级**: 中

**证据** (`frameworks/napi/power/power_napi.cpp`):
```cpp
// 参数直接传递，未做长度/内容校验
napi_value PowerNapi::Shutdown(napi_env env, napi_callback_info info)
{
    std::string reason;
    // ... reason 从 JS 参数获取
    proxy_->ShutdownDevice(reason);  // 直接传递
}
```

**触发条件**:
1. 应用调用 `power.shutdown(reason)`
2. `reason` 字符串过长或包含特殊字符

**影响**:
- 可能导致日志溢出
- 配置文件写入异常
- 系统行为异常

**修复建议**:
```cpp
// 添加参数校验
constexpr size_t MAX_REASON_LENGTH = 128;
if (reason.length() > MAX_REASON_LENGTH) {
    return NapiErrors::ThrowError(env, ERR_PARAM_INVALID);
}
// 过滤特殊字符
```

---

#### 风险点 2: RunningLock 名称未校验

**风险等级**: 中

**证据** (`frameworks/napi/runninglock/runninglock_napi.cpp`):
```cpp
napi_value RunningLockNapi::Create(...)
{
    std::string name;
    napi_get_value_string_utf8(env, argv[0], ..., &name);
    // 名称直接使用，未校验
    auto lock = RunningLockMgr::CreateRunningLock(name, type);
}
```

**触发条件**:
1. 应用调用 `runningLock.create(name, type)`
2. `name` 包含路径遍历字符 (`../`)
3. `name` 过长

**影响**:
- 日志注入
- 可能的路径遍历
- 调试信息混乱

**修复建议**:
```cpp
// 白名单字符校验
constexpr const char* VALID_NAME_CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-";
if (name.find_first_not_of(VALID_NAME_CHARS) != std::string::npos) {
    return NapiErrors::ThrowError(env, ERR_PARAM_INVALID);
}
```

---

### 3.2 权限检查风险

#### 风险点 3: 部分 API 缺少权限检查

**风险等级**: 高

**证据** (`frameworks/native/power_mgr_client.cpp`):
```cpp
// wakeup API 无权限检查
bool PowerMgrClient::WakeupDevice(const std::string& deviceId,
    const std::string& wakeupReason)
{
    // 无权限校验，任何应用可调用
    return proxy_->WakeupDevice(deviceId, wakeupReason);
}
```

**触发条件**:
1. 恶意应用调用 `power.wakeup()`
2. 导致设备频繁唤醒
3. 消耗电量

**影响**:
- 拒绝服务攻击
- 电池耗尽
- 用户体验下降

**修复建议**:
```cpp
// 添加权限检查
#include "permission.h"
if (!PermissionCheck("ohos.permission.WAKEUP")) {
    return false;
}
```

---

#### 风险点 4: RunningLock 创建缺少 UID 验证

**风险等级**: 高

**证据** (`services/native/src/runninglock/running_lock_mgr.cpp`):
```cpp
sptr<RunningLock> RunningLockMgr::CreateRunningLock(
    const std::string& name, RunningLockType type)
{
    // 未验证调用方 UID
    auto lock = new RunningLockEntity(name, type);
    lock->SetCallerInfo(GetCallingUid(), GetCallingPid());
    // ...
}
```

**触发条件**:
1. 应用创建 RunningLock
2. 恶意应用伪造 UID
3. 占用系统资源

**影响**:
- 资源耗尽
- 权限提升
- 拒绝服务

**修复建议**:
```cpp
// 验证 UID
if (!IsSystemApp(GetCallingUid())) {
    // 非系统应用限制创建数量
    if (GetCurrentLockCount() >= MAX_USER_LOCKS) {
        return nullptr;
    }
}
```

---

### 3.3 IPC 通信风险

#### 风险点 5: IPC 数据反序列化边界检查

**风险等级**: 中

**证据** (`services/zidl/src/power_mgr_async_reply_stub.cpp`):
```cpp
int PowerMgrStubAsync::OnRemoteRequest(...)
{
    // 从 data 读取，未校验长度
    std::string reason = data.ReadString();
    int32_t timeout = data.ReadInt32();
    // ...
}
```

**触发条件**:
1. 构造恶意 IPC 消息
2. 数据长度异常
3. 导致解析错误

**影响**:
- 解析崩溃
- 内存越界

**修复建议**:
```cpp
// 校验数据可读性
if (!data.HasReadabilityTracker()) {
    return ERR_INVALID_DATA;
}
```

---

### 3.4 资源管理风险

#### 风险点 6: RunningLock 数量无上限

**风险等级**: 中

**证据** (`services/native/src/runninglock/running_lock_mgr.cpp`):
```cpp
// 无最大锁数量限制
std::map<std::string, sptr<RunningLockEntity>> runningLockMap_;
```

**触发条件**:
1. 恶意应用大量创建 RunningLock
2. 内存占用持续增长
3. 系统性能下降

**影响**:
- 内存耗尽
- OOM Kill

**修复建议**:
```cpp
// 添加数量限制
constexpr size_t MAX_RUNNING_LOCKS = 256;
if (runningLockMap_.size() >= MAX_RUNNING_LOCKS) {
    POWER_HILOGE("Too many running locks");
    return nullptr;
}
```

---

### 3.5 配置安全风险

#### 风险点 7: 配置文件路径遍历

**风险等级**: 中

**证据** (`services/native/src/wakeup/wakeup_source_parser.cpp`):
```cpp
// 配置文件路径可能存在注入
std::string configPath = CONFIG_DIR + configFile;
```

**触发条件**:
1. 配置文件路径可控
2. 攻击者修改配置

**影响**:
- 加载恶意配置
- 执行任意代码

**修复建议**:
```cpp
// 使用白名单配置路径
constexpr const char* ALLOWED_CONFIG_PATHS[] = {
    "/system/etc/power_config/",
    "/vendor/etc/power_config/"
};
if (!IsPathAllowed(configPath)) {
    return false;
}
```

---

## 4. 安全加固建议

### 4.1 输入验证

| 检查项 | 当前状态 | 加固建议 |
|--------|----------|----------|
| API 参数长度 | 部分检查 | 统一长度限制 |
| 特殊字符过滤 | 部分实现 | 全量实现 |
| 路径遍历防护 | 未实现 | 实现路径规范化 |
| UID 验证 | 部分实现 | 全面实现 |

### 4.2 权限管理

| 检查项 | 当前状态 | 加固建议 |
|--------|----------|----------|
| 关机/重启权限 | 已实现 | 保持 |
| 电源模式权限 | 已实现 | 保持 |
| 唤醒权限 | 未实现 | 添加 |
| RunningLock UID | 部分实现 | 全面实现 |

### 4.3 资源限制

| 限制项 | 当前状态 | 加固建议 |
|--------|----------|----------|
| RunningLock 数量 | 无限制 | 添加上限 |
| 回调注册数量 | 无限制 | 添加上限 |
| IPC 消息大小 | 无限制 | 添加上限 |
| 频率限制 | 无实现 | 实现频率控制 |

### 4.4 日志审计

| 检查项 | 当前状态 | 加固建议 |
|--------|----------|----------|
| API 调用日志 | 部分实现 | 补充缺失 |
| 权限检查日志 | 部分实现 | 补充缺失 |
| 异常日志 | 已实现 | 保持 |
| 安全事件审计 | 未实现 | 实现 HiSysEvent |

---

## 5. 检查范围与局限性

### 5.1 已检查范围

- [x] N-API 层 (`frameworks/napi/`)
- [x] Native Client (`frameworks/native/`)
- [x] Service 层 (`services/native/src/`)
- [x] IPC 层 (`services/zidl/`)
- [x] 工具层 (`utils/`)

### 5.2 未检查范围

- [ ] 测试代码 (`test/`)
- [ ] 外部依赖库
- [ ] 内核驱动 (`drivers_interface_power`)
- [ ] 系统配置 (`etc/init/`)

### 5.3 局限性说明

1. **静态分析**: 本评审基于静态代码分析，未进行动态测试
2. **代码覆盖**: 部分边界情况未完全覆盖
3. **运行时行为**: 某些安全问题可能在特定场景下触发

---

## 6. 安全相关文件

| 文件 | 说明 |
|------|------|
| `utils/permission/permission.cpp` | 权限检查实现 |
| `services/native/include/power_mgr_service.h` | 服务安全配置 |
| `powermanager.yaml` | HiSysEvent 安全事件 |
| `bundle.json` | 权限声明 |

---

## 7. 安全测试建议

1. **模糊测试**: 对 N-API 和 IPC 接口进行 fuzz 测试
2. **渗透测试**: 模拟恶意应用调用
3. **资源耗尽测试**: 大量创建 RunningLock
4. **权限测试**: 非授权调用测试

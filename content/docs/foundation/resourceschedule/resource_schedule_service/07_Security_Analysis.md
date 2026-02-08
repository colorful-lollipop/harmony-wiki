# 安全风险分析

## 目的

本文档基于代码证据分析 Resource Schedule Service 的安全风险，包括攻击面、信任边界和可被利用点。

## 适用范围

- 安全工程师
- 代码审计人员
- 系统架构师

---

## 攻击面清单

### 1. N-API 接口攻击面

| 接口 | 风险类型 | 说明 |
|------|----------|------|
| `setProcessPriority(pid, priority)` | 权限提升 | 修改任意进程优先级 |
| `setPowerSaveMode(pid, mode)` | 拒绝服务 | 强制降频影响性能 |
| `getLevel()` | 信息泄露 | 获取系统负载信息 |

### 2. IPC 接口攻击面

| 接口 | SA ID | 风险类型 | 说明 |
|------|-------|----------|------|
| `ReportData()` | 1901 | 权限绕过 | 伪造事件影响调度 |
| `KillProcess()` | 1901 | 拒绝服务 | 终止进程 |
| `SendRequestSync()` | 1918 | 代码执行 | 执行调度操作 |

### 3. 配置文件攻击面

| 文件 | 风险类型 | 说明 |
|------|----------|------|
| `cgroup_action_config.json` | 配置篡改 | 修改 cgroup 策略 |
| `res_sched_plugin_switch.xml` | 功能绕过 | 禁用安全插件 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                                │
└─────────────────────────────────────────────────────────────────┘

【不可信区域】
  ┌─────────────────────────────────────────────────────────────┐
  │  三方应用                                                    │
  │  ├── 受限 N-API (backgroundProcessManager 部分接口)         │
  │  └── 不可调用 IPC 直接                                        │
  └──────────────────┬──────────────────────────────────────────┘
                     │ 安全边界
                     ▼
【半可信区域】
  ┌─────────────────────────────────────────────────────────────┐
  │  系统应用 / 系统服务                                          │
  │  ├── N-API (systemload, backgroundProcessManager)            │
  │  ├── IPC (受限 ResType 上报)                                 │
  │  └── 需权限检查                                               │
  └──────────────────┬──────────────────────────────────────────┘
                     │ 安全边界
                     ▼
【可信区域】
  ┌─────────────────────────────────────────────────────────────┐
  │  ressched 服务 (SA 1901)                                     │
  │  ├── 权限检查中心                                            │
  │  ├── 事件分发器                                              │
  │  └── 插件管理器                                              │
  └──────────────────┬──────────────────────────────────────────┘
                     │ IPC
                     ▼
  ┌─────────────────────────────────────────────────────────────┐
│  ressched_executor 服务 (SA 1918)                            │
  │  └── 内核操作接口                                            │
  └─────────────────────────────────────────────────────────────┘
```

---

## 可利用点分析

### 可被利用点 #1: ReportData 权限绕过

**证据位置**: `ressched/services/resschedservice/src/res_sched_service.cpp:221-290`

**问题描述**:
虽然服务对 ReportData 做了权限检查，但存在多种绕过路径：

```cpp
// 代码片段
bool ResSchedService::IsThirdPartType(const uint32_t type) {
    // 检查资源类型是否允许第三方应用上报
    if (allowAllAppReportRes_.find(type) == allowAllAppReportRes_.end()) {
        return false;
    }
    
    // 检查是否为前台应用
    if (allowFgAppReportRes_.find(type) != allowFgAppReportRes_.end() &&
        !ResSchedMgr::GetInstance().IsForegroundApp(IPCSkeleton::GetCallingPid())) {
        return false;
    }
    
    // 检查 Token 类型
    AccessToken::AccessTokenID tokenId = OHOS::IPCSkeleton::GetCallingTokenID();
    auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
    if (tokenType != Security::AccessToken::ATokenTypeEnum::TOKEN_HAP) {
        return false;
    }
    return true;
}
```

**利用路径**:
1. 恶意应用伪造前台状态
2. 利用 `THIRDPART_RES` 列表中的资源类型（如 `RES_TYPE_CLICK_RECOGNIZE`）
3. 发送大量伪造事件干扰调度决策

**影响**: 中等 - 可能导致错误调度决策，影响系统性能

**修复建议**:
```cpp
// 建议: 增加签名验证或频率限制
if (!VerifyAppSignature(callingUid)) {
    return false;
}
```

---

### 可被利用点 #2: Payload 大小检查与解析

**证据位置**: `ressched/services/resschedservice/src/res_sched_service.cpp:783-796`

**问题描述**:
```cpp
// 代码片段
int32_t ResSchedService::CheckReportDataParcel(const uint32_t& type, 
    const int64_t& value, const std::string& payload, int32_t uid) {
    // ...
    if (payload.size() > PAYLOAD_MAX_SIZE) {  // 4096
        RESSCHED_LOGE("too long payload.size:%{public}u", (uint32_t)payload.size());
        return ERR_RES_SCHED_PARCEL_ERROR;
    }
    return ERR_OK;
}
```

后续解析:
```cpp
nlohmann::json ResSchedService::StringToJsonObj(const std::string& str) {
    nlohmann::json jsonTmp = nlohmann::json::parse(str, nullptr, false);
    if (jsonTmp.is_discarded()) {
        return jsonObj;  // 返回空对象
    }
    return jsonTmp;
}
```

**利用路径**:
1. 构造 4096 字节的畸形 JSON
2. 解析失败但服务继续处理
3. 可能导致插件处理空对象时的未定义行为

**影响**: 低 - 仅导致处理失败

**修复建议**:
```cpp
// 建议: 解析失败时返回错误
if (jsonTmp.is_discarded()) {
    RESSCHED_LOGE("JSON parse failed");
    return ERR_RES_SCHED_PARCEL_ERROR;  // 返回错误而非空对象
}
```

---

### 可被利用点 #3: KillProcess 权限控制

**证据位置**: `ressched/services/resschedservice/src/res_sched_service.cpp:392-413`

**问题描述**:
```cpp
ErrCode ResSchedService::KillProcess(const std::string& payload, int32_t& resultValue) {
    uint32_t accessToken = IPCSkeleton::GetCallingTokenID();
    int32_t uid = IPCSkeleton::GetCallingUid();
    Security::AccessToken::ATokenTypeEnum tokenTypeFlag =
        Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(accessToken);
    
    // 仅允许特定 UID 调用
    if ((uid != MEMMGR_UID && uid != SAMGR_UID && uid != HIVIEW_UID && uid != GRAPHIC_UID)
        || tokenTypeFlag != Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE) {
        RESSCHED_LOGE("no permission, kill process fail");
        resultValue = RES_SCHED_KILL_PROCESS_FAIL;
        return ERR_OK;
    }
    // ...
}
```

**利用路径**:
1. 攻击者需要获取特定系统服务的权限
2. 一旦 compromised，可以终止任意进程

**影响**: 高 - 可导致拒绝服务

**修复建议**:
```cpp
// 建议: 增加操作审计日志
void AuditLog(const std::string& operation, int32_t uid, int32_t pid) {
    HiSysEventWrite(..., "KILL_PROCESS", "uid", uid, "target_pid", pid);
}
```

---

### 可被利用点 #4: 插件动态加载

**证据位置**: `ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp`

**问题描述**:
```cpp
// 代码片段
void PluginMgr::LoadPlugin(const std::string& pluginName) {
    void* handle = dlopen(pluginPath.c_str(), RTLD_NOW);
    // ...
    CreatePluginFunc createPlugin = 
        (CreatePluginFunc)dlsym(handle, "CreatePlugin");
    // ...
}
```

**风险**: 如果插件路径可被篡改，可能导致恶意代码加载

**利用路径**:
1. 攻击者替换配置文件中的插件路径
2. 或利用符号链接劫持

**影响**: 高 - 可能导致代码执行

**修复建议**:
```cpp
// 建议: 插件路径白名单 + 签名验证
bool VerifyPluginSignature(const std::string& path) {
    // 验证插件签名
    return VerifySignature(path, EXPECTED_SIGNATURE);
}
```

---

### 可被利用点 #5: 限流机制绕过

**证据位置**: `ressched/services/resschedservice/src/res_sched_service.cpp:830-875`

**问题描述**:
```cpp
bool ResSchedService::IsLimitRequest(int32_t uid) {
    std::lock_guard<std::mutex> lock(mutex_);
    int64_t nowTime = ResCommonUtil::GetNowMillTime(true);
    CheckAndUpdateLimitData(nowTime);
    
    if (allRequestCount_.load() >= ALL_UID_REQUEST_LIMIT_COUNT) {  // 650
        return true;
    }
    // ...
}
```

**利用路径**:
1. 多个 UID 协同攻击，每个 UID 保持 250 req/s 以下
2. 总计可能超过系统处理能力

**影响**: 中 - 可能导致服务过载

**修复建议**:
```cpp
// 建议: 增加更细粒度的限流
bool IsLimitRequest(int32_t uid, uint32_t resType) {
    // 按资源类型限流
    auto& typeCounter = typeRequestCount_[resType];
    if (typeCounter > TYPE_REQUEST_LIMIT) {
        return true;
    }
    // ...
}
```

---

## 安全检查清单

| 检查项 | 状态 | 证据位置 |
|--------|------|----------|
| UID 检查 | ✅ | `res_sched_service.cpp:260-289` |
| Token 类型检查 | ✅ | `res_sched_service.cpp:273-277` |
| Payload 大小限制 | ✅ | `res_sched_service.cpp:791-794` |
| 权限缓存 | ✅ | `res_sched_service.cpp:279-289` |
| 请求限流 | ✅ | `res_sched_service.cpp:830-875` |
| 插件签名验证 | ❌ | 未实现 |
| 操作审计日志 | ⚠️ | 部分实现 |
| SELinux 策略 | ✅ | `resource_schedule_service.cfg` |

---

## 修复建议汇总

### 高优先级

1. **增加插件签名验证**
   - 在加载插件前验证签名
   - 维护受信任插件白名单

2. **增强 KillProcess 审计**
   - 记录所有杀进程操作
   - 包括调用者 UID、目标 PID、时间戳

### 中优先级

3. **改进限流机制**
   - 按资源类型限流
   - 检测异常模式

4. **增加 JSON 解析错误处理**
   - 解析失败时返回明确错误
   - 记录解析失败的 payload 样本

### 低优先级

5. **增加伪造事件检测**
   - 检测异常的事件序列
   - 标记可疑应用

---

## 代码证据

### 权限检查实现

```cpp
// 文件: ressched/services/resschedservice/src/res_sched_service.cpp:260-290

bool ResSchedService::IsHasPermission(const uint32_t type, int32_t uid) {
    // 检查 UID 白名单
    const auto& item = allowSomeSAReportRes_.find(type);
    if (item != allowSomeSAReportRes_.end()) {
        if (item->second.find(uid) == item->second.end()) {
            return false;
        }
    }
    
    // 检查 Token 类型
    AccessToken::AccessTokenID tokenId = OHOS::IPCSkeleton::GetCallingTokenID();
    auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
    if (tokenType != Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE) {
        return false;
    }
    
    // 检查权限
    int32_t hasPermission = AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenId, NEEDED_PERMISSION);
    return hasPermission == 0;
}
```

---

## 相关链接

- [架构设计](01_Architecture.md) - 信任边界图
- [内部 API](04_Inner_API.md) - 接口权限说明
- [问题排查](08_Troubleshooting.md) - 调试方法

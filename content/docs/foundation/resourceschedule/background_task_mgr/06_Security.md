# 安全风险评审

## 概述

本文档基于代码证据对后台任务管理模块进行安全风险分析，识别攻击面、信任边界和可被利用点。

**评审范围**: 
- `services/` - 服务实现
- `frameworks/` - 框架层
- `interfaces/` - 接口层

**排除范围**:
- `test/` - 测试代码
- 第三方依赖库内部实现

---

## 攻击面清单

### 1. 外部输入攻击面

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| JS API参数 | N-API接口 | 高 |
| IPC调用 | IBackgroundTaskMgr接口 | 高 |
| 配置文件 | JSON配置文件 | 中 |
| Dump命令 | Dump接口 | 低 |

### 2. 资源操作攻击面

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 短时任务配额 | 申请/取消延时挂起 | 中 |
| 长时任务状态 | 启动/停止/挂起任务 | 高 |
| 能效资源 | CPU/GPS等资源申请 | 高 |
| 通知管理 | 通知栏操作 | 中 |

### 3. 数据存储攻击面

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 关系型数据库 | 任务持久化 | 中 |
| 内存数据结构 | 任务记录映射 | 中 |
| 配置缓存 | 签名/配额配置 | 中 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不可信区域                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   JS App    │  │   C++ App   │  │    C App    │             │
│  │  (N-API)    │  │  (IPC Call) │  │   (NDK)     │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘             │
└─────────┼────────────────┼────────────────┼────────────────────┘
          │                │                │
          └────────────────┴────────────────┘
                           │
                    【信任边界1】
                    参数校验/反序列化
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    IPC边界 (Binder)                              │
│              Parcel数据序列化/反序列化                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    【信任边界2】
                    权限检查/身份验证
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      可信区域                                    │
│              BackgroundTaskMgrService                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  安全检查层                                              │   │
│  │  - CheckCallingToken()                                   │   │
│  │  - CheckHapCalling()                                     │   │
│  │  - VerifyAccessToken()                                   │   │
│  │  - IsSystemApp()                                         │   │
│  └─────────────────────────┬───────────────────────────────┘   │
│                            │                                     │
│  ┌─────────────────────────▼───────────────────────────────┐   │
│  │  业务逻辑层                                              │   │
│  │  - BgTransientTaskMgr                                    │   │
│  │  - BgContinuousTaskMgr                                   │   │
│  │  - BgEfficiencyResourcesMgr                              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 可被利用点分析

### 可被利用点1: 短时任务配额耗尽攻击

**证据**: `services/transient_task/src/bg_transient_task_mgr.cpp`

**代码位置**:
```cpp
// bg_transient_task_mgr.cpp:152-175
ErrCode BgTransientTaskMgr::IsCallingInfoLegal(
    int32_t uid, int32_t pid, std::string &name, const sptr<IExpiredCallback>& callback) {
    // 验证UID/PID
    if (!VerifyCallingInfo(uid, pid)) {
        return ERR_BGTASK_INVALID_PID_OR_UID;
    }
    // ...
}

// decision_maker.cpp:45-89
DecisionMaker::Decision DecisionMaker::Decide(
    int32_t uid, const std::string& bundleName, bool isCached) {
    // 检查当日配额
    if (IsExceedMaxBgTaskDurationPerDay(uid)) {
        return Decision::DENY;
    }
    // ...
}
```

**攻击路径**:
1. 恶意应用重复申请短时任务
2. 每次申请通过后立即取消
3. 但配额仍被计算（因为配额基于申请时长）
4. 导致正常应用无法申请短时任务

**影响**: 拒绝服务（DoS）- 耗尽系统短时任务配额

**触发条件**:
- 应用具有基本后台权限
- 配额计算逻辑不区分正常/恶意申请

**修复建议**:
```cpp
// 建议：增加配额消耗速率限制
if (recentRequestCount[uid] > THRESHOLD_PER_MINUTE) {
    BGTASK_LOGW("Quota request rate limit exceeded for uid %d", uid);
    return ERR_BGTASK_RATE_LIMITED;
}
```

---

### 可被利用点2: 能效资源CPU级别滥用

**证据**: `services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp`

**代码位置**:
```cpp
// bg_efficiency_resources_mgr.cpp:292-315
ErrCode BgEfficiencyResourcesMgr::CheckIfCanApplyCpuLevel(
    const sptr<EfficiencyResourceInfo> &resourceInfo, 
    const std::string &bundleName, 
    pid_t uid) {
    int32_t applyCpuLevel = resourceInfo->GetCpuLevel();
    if (applyCpuLevel < static_cast<int32_t>(EfficiencyResourcesCpuLevel::DEFAULT) ||
        applyCpuLevel > static_cast<int32_t>(EfficiencyResourcesCpuLevel::LARGE_CPU)) {
        return ERR_BGTASK_EFFICIENCY_RESOURCES_CPU_LEVEL_INVALID;
    }
    // ...
}

// bg_task_config_file_info.cpp:33-48
bool BgTaskConfigFileInfo::CheckCpuLevel(
    const std::string &bundleName, 
    int32_t cpuLevel) {
    auto iter = allowApplyCpuBundleInfoMap_.find(bundleName);
    if (iter == allowApplyCpuBundleInfoMap_.end()) {
        return false;
    }
    return cpuLevel <= iter->second.maxCpuLevel_;
}
```

**攻击路径**:
1. 应用在配置文件中配置了CPU级别权限
2. 应用反复申请LARGE_CPU级别资源
3. 申请后不主动释放（依赖超时）
4. 可能导致CPU资源被持续占用

**影响**: 资源耗尽 - 高CPU占用影响系统性能

**触发条件**:
- 应用在白名单中
- 使用持久化资源申请（isPersist=true）

**修复建议**:
```cpp
// 建议：增加CPU级别申请的持续时间限制
if (cpuLevel == LARGE_CPU && timeOut > MAX_LARGE_CPU_TIMEOUT) {
    return ERR_BGTASK_CPU_TIMEOUT_TOO_LONG;
}
```

---

### 可被利用点3: 长时任务通知栏欺骗

**证据**: `services/continuous_task/src/bg_continuous_task_mgr.cpp`

**代码位置**:
```cpp
// bg_continuous_task_mgr.cpp:846-920
ErrCode BgContinuousTaskMgr::CheckNotificationText(
    std::string &notificationText,
    const std::shared_ptr<ContinuousTaskRecord> continuousTaskRecord) {
    // 检查通知文本是否包含应用名
    std::string appName = GetAppName(continuousTaskRecord);
    if (notificationText.find(appName) == std::string::npos) {
        BGTASK_LOGW("Notification text does not contain app name");
        return ERR_BGTASK_NOTIFICATION_VERIFY_FAILED;
    }
    return ERR_OK;
}

// 但如果应用名本身包含特殊字符或欺骗性内容
// 如 "System"、"官方" 等
```

**攻击路径**:
1. 应用注册时使用欺骗性名称（如"System Update"）
2. 申请长时任务
3. 通知栏显示应用名称
4. 用户可能被误导认为通知来自系统

**影响**: 社会工程学攻击 - 用户混淆

**触发条件**:
- 应用Bundle名称具有欺骗性
- 通过应用商店审核

**修复建议**:
```cpp
// 建议：增加应用名黑名单检查
const std::set<string> FORBIDDEN_NAMES = {"System", "官方", "Official"};
if (FORBIDDEN_NAMES.count(appName)) {
    return ERR_BGTASK_DECEPTIVE_APP_NAME;
}
```

---

### 可被利用点4: Dump接口信息泄露

**证据**: `services/core/src/background_task_mgr_service.cpp`

**代码位置**:
```cpp
// background_task_mgr_service.cpp:136-178
int32_t BackgroundTaskMgrService::Dump(int32_t fd, 
    const std::vector<std::u16string> &args) {
    if (!AllowDump()) {
        return ERR_BGTASK_PERMISSION_DENIED;
    }
    // ...
}

// background_task_mgr_service.cpp:168-178
bool BackgroundTaskMgrService::AllowDump() {
    // 检查调用者是否有DUMP权限
    Security::AccessToken::AccessTokenID callingToken = IPCSkeleton::GetFirstTokenID();
    // ...
}
```

**问题**: 虽然检查了DUMP权限，但Dump输出可能包含敏感信息

**Dump内容示例**:
```
// 包含的信息：
- 所有运行中的应用UID
- 长时任务详情（Bundle名、Ability名）
- 短时任务配额使用情况
- 能效资源申请记录
```

**攻击路径**:
1. 恶意系统应用获取DUMP权限
2. 通过Dump收集其他应用的运行状态
3. 分析用户行为模式

**影响**: 信息泄露 - 用户隐私暴露

**修复建议**:
```cpp
// 建议：对敏感信息进行脱敏
std::string AnonymizeUid(int32_t uid) {
    return "UID_" + std::to_string(uid % 1000);  // 部分隐藏
}
```

---

### 可被利用点5: 配置文件的JSON解析漏洞

**证据**: `services/common/src/bgtask_config.cpp`

**代码位置**:
```cpp
// bgtask_config.cpp:184-231
bool BgtaskConfig::ParseConfig(const std::string& configData, int32_t sourceType) {
    nlohmann::json jsonObj;
    try {
        jsonObj = nlohmann::json::parse(configData);
    } catch (nlohmann::json::parse_error& e) {
        BGTASK_LOGE("Parse json failed");
        return false;
    }
    // 解析配置...
    if (!jsonObj.is_null() && !jsonObj.empty()) {
        if (jsonObj.contains("transientTaskExemptedQuatoList")) {
            // 解析豁免列表
        }
    }
}
```

**攻击路径**:
1. 配置文件通过未加密通道传输
2. 中间人攻击篡改配置
3. 添加恶意应用到豁免列表
4. 恶意应用获得额外配额或权限

**影响**: 权限提升 - 绕过配额限制

**触发条件**:
- 配置文件来源不可信
- 缺乏签名验证

**修复建议**:
```cpp
// 建议：配置文件签名验证
if (!VerifyConfigSignature(configData, signature)) {
    BGTASK_LOGE("Config signature verification failed");
    return false;
}
```

---

## 安全检查清单

### 已实施的安全措施 ✅

| 检查项 | 实施位置 | 状态 |
|--------|----------|------|
| UID/PID验证 | `bg_transient_task_mgr.cpp:506-509` | ✅ |
| 权限检查 | `background_task_mgr_service.cpp:146-166` | ✅ |
| 系统应用验证 | `bundle_manager_helper.cpp:80-83` | ✅ |
| 原子服务拦截 | `background_task_mgr_service.cpp:801-805` | ✅ |
| Bundle签名验证 | `bgtask_config.cpp:316-320` | ✅ |
| 参数类型检查 | `efficiency_resources_operation.cpp:101-149` | ✅ |
| 配额上限检查 | `decision_maker.cpp:77-89` | ✅ |

### 建议增强的安全措施 ⚠️

| 检查项 | 建议 | 优先级 |
|--------|------|--------|
| 速率限制 | 短时任务申请频率限制 | 高 |
| CPU时间限制 | LARGE_CPU最大持续时间 | 中 |
| 应用名审核 | Bundle名黑名单 | 中 |
| Dump脱敏 | UID/PID部分隐藏 | 低 |
| 配置签名 | 配置文件完整性验证 | 高 |

---

## 隐私合规检查

### 数据收集清单

| 数据类型 | 收集位置 | 用途 | 建议 |
|----------|----------|------|------|
| UID | 多处 | 身份识别 | 日志脱敏 |
| Bundle名 | 多处 | 应用识别 | 保留 |
| 任务类型 | 多处 | 功能实现 | 保留 |
| 配额使用 | decision_maker | 资源管理 | 聚合统计 |

### 数据保留策略

| 数据 | 保留时间 | 位置 |
|------|----------|------|
| 任务记录 | 应用存活期间 | 内存 |
| 配额记录 | 当日有效 | 内存/数据库 |
| 配置缓存 | 直到更新 | 内存 |

---

## 修复建议汇总

### 高优先级

1. **配置文件签名验证**
   - 位置: `services/common/src/bgtask_config.cpp`
   - 实现: 添加RSA/ECC签名验证

2. **短时任务速率限制**
   - 位置: `services/transient_task/src/decision_maker.cpp`
   - 实现: 添加每分钟申请次数限制

### 中优先级

3. **CPU资源时间限制**
   - 位置: `services/efficiency_resources/src/bg_efficiency_resources_mgr.cpp`
   - 实现: LARGE_CPU级别最大30分钟限制

4. **应用名黑名单**
   - 位置: `services/continuous_task/src/bg_continuous_task_mgr.cpp`
   - 实现: 启动时检查Bundle名

### 低优先级

5. **Dump信息脱敏**
   - 位置: `services/core/src/background_task_mgr_service.cpp`
   - 实现: UID哈希化处理

---

## 相关文档

- [架构说明](02_Architecture.md) - 信任边界详细说明
- [N-API接口](03_NAPI_Reference.md) - 输入参数校验
- [内部API](04_Inner_API.md) - 权限检查接口

# 安全风险评审

> 本文档对 Form Fwk 进行安全风险分析，基于代码证据识别潜在攻击面与风险点

## 1. 攻击面清单

| 攻击面 | 类型 | 入口点 | 说明 |
|--------|------|--------|------|
| **N-API 接口** | JS API | `frameworks/js/napi/*` | JS 层对外接口，接收外部输入 |
| **IPC 接口** | IPC | `interfaces/inner_api/*` | 跨进程通信接口 |
| **SA 接口** | System Ability | `services/src/form_mgr/form_mgr_service.cpp` | 系统服务注册点 |
| **配置文件解析** | XML | `services/config/form_xml_parser.cpp` | XML 配置文件解析 |
| **卡片数据传递** | 数据 | `FormProviderData` | 卡片更新数据传递 |

## 2. 信任边界与数据流

```
┌─────────────────────────────────────────────────────────────────────┐
│                         信任边界（应用进程）                          │
│  ┌──────────────┐    IPC    ┌──────────────────────────────┐    │
│  │   应用 JS    │ ────────▶ │   N-API 层 (formProvider...)  │    │
│  └──────────────┘           └──────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                                      │ IPC 调用
                                      ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       信任边界（系统服务进程）                        │
│  ┌──────────────────────────────┐                                  │
│  │  FormMgrService (SA 403)    │                                  │
│  │  - 权限检查                 │                                  │
│  │  - 表单管理                 │                                  │
│  │  - 事件调度                 │                                  │
│  └──────────────────────────────┘                                  │
└─────────────────────────────────────────────────────────────────────┘
```

## 3. 已发现的可利用点

### 3.1 权限校验不完整风险

**证据** (`services/src/form_mgr/form_mgr_service.cpp:987-1013`):
```cpp
int FormMgrService::CheckFormPermission(...)
{
    // 综合权限检查
    // 存在多处权限检查入口
}
```

**风险描述**:
- 部分 IPC 接口可能存在权限检查遗漏
- 建议全面审计所有 IPC 接口的权限覆盖情况

**触发路径**:
```
恶意应用 → IPC 调用 → 未授权的 Form 操作
```

**影响**:
- 未经授权的卡片访问
- 卡片数据泄露

**修复建议**:
- 对所有 IPC 接口增加权限检查
- 统一权限检查入口 `CheckFormPermission()`

---

### 3.2 跨用户访问风险

**证据** (`services/src/data_center/form_data_mgr.cpp:2349-2371`):
```cpp
int FormDataMgr::CheckInvalidForm(...)
{
    // 检查 formId 和跨用户操作
}
```

**风险描述**:
- 跨用户访问控制不严格可能导致卡片数据泄露
- 需要验证调用者与卡片所属用户的一致性

**触发路径**:
```
用户 A 的恶意应用 → 构造跨用户 formId → 访问用户 B 的卡片
```

**影响**:
- 跨用户卡片数据访问
- 隐私泄露

**修复建议**:
- 加强跨用户 formId 验证
- 增加用户隔离检查

---

### 3.3 FormId 注入风险

**证据** (`services/src/form_mgr/form_mgr_adapter.cpp:151-154`):
```cpp
if (formId <= 0) {
    // formId 有效性检查
}
```

**风险描述**:
- 部分接口对 formId 边界检查不严格
- 负数或超大 formId 可能导致异常行为

**触发路径**:
```
恶意应用 → 传入异常 formId → 服务异常或崩溃
```

**影响**:
- 服务拒绝
- 潜在的拒绝服务攻击

**修复建议**:
- 统一 formId 校验逻辑
- 增加边界值测试

---

### 3.4 临时卡片权限风险

**证据** (`services/src/form_refresh/check_mgr/self_form_checker.cpp:26-41`):
```cpp
// SelfFormChecker - 检查表单是否属于调用者
```

**风险描述**:
- 临时卡片（CastTempForm）权限验证可能不严格
- 可能被恶意应用利用获取敏感卡片数据

**触发路径**:
```
恶意应用 → CastTempForm → 获取非授权临时卡片
```

**影响**:
- 卡片数据泄露
- 权限提升

**修复建议**:
- 加强临时卡片所有权验证
- 增加临时卡片使用限制

---

### 3.5 XML 配置注入风险

**证据** (`services/config/form_xml_parser.cpp:40-65`):
```cpp
// XML 根节点验证
if (xmlStrcmp(root->name, BAD_CAST "root")) {
    // 验证失败
}
```

**风险描述**:
- XML 解析器可能受到 XXE (XML External Entity) 攻击
- 恶意构造的 XML 可能导致信息泄露

**触发路径**:
```
攻击者 → 恶意配置文件 → XML 解析漏洞
```

**影响**:
- 本地文件读取
- 服务拒绝

**修复建议**:
- 禁用 XML 外部实体
- 增加 XML 解析安全加固

---

## 4. 安全机制分析

### 4.1 已有的安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| **权限验证** | `form_util.cpp:245-256` | `VerifyCallingPermission()` |
| **Token 验证** | `form_util.cpp:227-243` | `IsSACall()` + AccessTokenKit |
| **UID 验证** | `form_data_mgr.cpp` | 多处 `callerUid` 比对 |
| **Bundle 验证** | `form_info_mgr.cpp:461-474` | `IsCaller()` |
| **跨账户权限** | `form_mgr_service.cpp:1759-1780` | `CheckAcrossLocalAccountsPermission()` |
| **多层级 Checker** | `check_mgr/` | 7 种检查器 |

### 4.2 各类 Checker 职责

| 检查器 | 职责 | 防护目标 |
|--------|------|----------|
| `SelfFormChecker` | 验证卡片所有权 | 防止越权访问 |
| `CallingBundleChecker` | 验证调用者包名 | 防止包名伪造 |
| `CallingUserChecker` | 验证调用者 UID | 防止 UID 伪造 |
| `SystemAppChecker` | 验证系统应用 | 防止普通应用冒充 |
| `ActiveUserChecker` | 验证活跃用户 | 防止跨用户访问 |
| `UntrustAppChecker` | 验证应用信任状态 | 防止非信任应用 |

---

## 5. 安全建议

### 5.1 高优先级

1. **全面权限审计**
   - 对所有 IPC 接口进行权限覆盖检查
   - 统一权限检查入口

2. **跨用户隔离**
   - 增强跨用户 formId 验证
   - 增加用户上下文验证

### 5.2 中优先级

3. **输入验证标准化**
   - 统一 formId 校验逻辑
   - 增加边界值测试用例

4. **XML 解析加固**
   - 禁用外部实体
   - 增加 XML Schema 验证

### 5.3 低优先级

5. **临时卡片限制**
   - 增加临时卡片使用场景限制
   - 缩短临时卡片有效期

6. **日志与监控**
   - 增加安全事件日志
   - 监控异常访问模式

---

## 6. 详细代码证据

### 6.1 权限检查实现证据

**VerifyCallingPermission 实现** (`services/src/common/util/form_util.cpp:245-256`):
```cpp
bool FormUtil::VerifyCallingPermission(const std::string &permissionName) {
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, permissionName);
    if (ret == Security::AccessToken::PermissionState::PERMISSION_DENIED) {
        return false;
    }
    return true;
}
```

**系统应用检查** (`services/src/form_mgr/form_mgr_service.cpp:1734-1742`):
```cpp
bool FormMgrService::CheckCallerIsSystemApp() const {
    auto callerTokenID = IPCSkeleton::GetCallingFullTokenID();
    if (!FormUtil::IsSACall() && 
        !Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(callerTokenID)) {
        HILOG_ERROR("The caller not system-app,can't use system-api");
        return false;
    }
    return true;
}
```

### 6.2 Bundle所有权验证证据

**IsCaller实现** (`services/src/data_center/form_info/form_info_mgr.cpp:455-474`):
```cpp
bool FormInfoMgr::IsCaller(const std::string& bundleName) {
    // Get bundle info from BMS
    bool ret = IN_PROCESS_CALL(
        bms->GetBundleInfo(bundleName, GET_BUNDLE_DEFAULT, bundleInfo, 
                          FormUtil::GetCurrentAccountId()));
    
    // Compare token IDs
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    if (bundleInfo.applicationInfo.accessTokenId == callerToken) {
        return true;
    }
    return false;
}
```

### 6.3 跨账户权限检查证据

**CheckAcrossLocalAccountsPermission** (`services/src/form_mgr/form_mgr_service.cpp:1759-1775`):
```cpp
bool FormMgrService::CheckAcrossLocalAccountsPermission() const {
    int callingUid = IPCSkeleton::GetCallingUid();
    int32_t userId = FormUtil::GetCallerUserId(callingUid);
    int32_t currentActiveUserId = FormUtil::GetCurrentAccountId();
    
    if (userId != currentActiveUserId) {
        // Caller is from different user - check special permission
        bool isCallingPermAccount = FormUtil::VerifyCallingPermission(
            AppExecFwk::Constants::PERMISSION_INTERACT_ACROSS_LOCAL_ACCOUNTS);
        if (!isCallingPermAccount) {
            return false;
        }
    }
    return true;
}
```

### 6.4 权限变更监听证据

**PermissionCustomizeListener** (`services/src/data_center/form_data_proxy_record.cpp:40-179`):
```cpp
class PermissionCustomizeListener : public Security::AccessToken::PermStateChangeCallbackCustomize {
    virtual void PermStateChangeCallback(Security::AccessToken::PermStateChangeInfo& result) {
        // Handle permission changes and refresh forms accordingly
    }
};

void FormDataProxyRecord::RegisterPermissionListener(
    const std::vector<FormDataProxy> &formDataProxies) {
    Security::AccessToken::PermStateChangeScope scopeInfo;
    scopeInfo.tokenIDs = {tokenId_};
    callbackPtr_ = std::make_shared<PermissionCustomizeListener>(scopeInfo, weak_from_this());
    Security::AccessToken::AccessTokenKit::RegisterPermStateChangeCallback(callbackPtr_);
}
```

---

## 7. 检查范围与局限性

### 7.1 已检查范围

- ✅ `services/src/form_mgr/` - 核心服务
- ✅ `services/src/data_center/` - 数据管理层
- ✅ `services/src/form_refresh/check_mgr/` - 刷新检查器
- ✅ `interfaces/inner_api/` - Inner API 接口
- ✅ `frameworks/js/napi/` - N-API 层
- ✅ `services/src/common/util/` - 工具函数（权限检查）
- ✅ `services/config/` - 配置解析

### 7.2 未检查范围

- ⚠️ 第三方依赖的内部实现
- ⚠️ 运行时动态加载场景
- ⚠️ 分布式场景下的跨设备安全
- ⚠️ Form Render Service内部实现细节

### 7.3 局限性说明

1. 本分析基于静态代码扫描，未进行运行时动态分析
2. 部分安全机制的具体实现依赖其他子系统（如 AccessTokenKit）
3. 分布式卡片场景的安全分析需要进一步研究
4. 部分证据行号可能随代码版本变化，请以实际代码为准

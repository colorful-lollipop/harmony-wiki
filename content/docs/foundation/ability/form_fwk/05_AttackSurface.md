# 攻击面分析

> 本文档识别 form_fwk 项目的所有外部攻击面，为安全研究提供入口指引
> 最后更新：2026-02-07

---

## 1. 攻击面总览

### 1.1 攻击面分类

| 攻击面类型 | 数量 | 风险等级 | 说明 |
|-----------|------|---------|------|
| N-API 接口 | 50+ | 中 | JS层暴露的API入口 |
| IPC 接口 | 100+ | 高 | 跨进程通信接口 |
| 配置文件 | 1 | 低 | XML配置解析 |
| 数据库操作 | 多 | 中 | RDB数据库交互 |
| 文件系统 | 少 | 低 | 临时文件操作 |

### 1.2 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           非信任域 (应用进程)                                 │
│  ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐     │
│  │   第三方应用     │      │   系统应用       │      │   系统服务       │     │
│  │  (Untrusted)    │      │  (System App)   │      │  (System Native)│     │
│  └────────┬────────┘      └────────┬────────┘      └────────┬────────┘     │
└───────────┼─────────────────────────┼─────────────────────────┼────────────┘
            │                         │                         │
            ▼                         ▼                         ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         信任边界线 #1 (N-API层)                              │
│                    N-API 接口 (formProvider/formHost)                       │
└─────────────────────────────────────────────────────────────────────────────┘
            │                         │                         │
            ▼                         ▼                         ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         信任边界线 #2 (IPC层)                                │
│                       IPC Binder 通信接口                                    │
│              IFormMgr / IFormHost / IFormProvider                          │
└─────────────────────────────────────────────────────────────────────────────┘
            │                         │                         │
            ▼                         ▼                         ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         信任域 (Form Manager Service)                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    FormMgrService (SA 403)                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │   │
│  │  │ 权限检查    │→│ 输入验证    │→│ 业务处理    │→│ 数据持久化│  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └───────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 外部输入清单

### 2.1 N-API 输入入口

#### formProvider 模块 (`app.form.formProvider`)

| JS API | 输入参数 | 风险点 | 代码位置 |
|--------|---------|--------|---------|
| `getFormsInfo` | bundleName, moduleName, formName | 字符串注入 | `js_form_provider.cpp` |
| `updateForm` | formId, formBindingData | 整数溢出、JSON注入 | `js_form_provider.cpp` |
| `requestPublishForm` | want, formBindingData | Want对象注入 | `js_form_provider.cpp` |
| `setFormNextRefreshTime` | formId, minute | 时间值校验 | `js_form_provider.cpp` |
| `openFormEditAbility` | abilityName, formId | 组件名注入 | `js_form_provider.cpp` |
| `requestOverflow` | formId, overflowInfo | 溢出参数注入 | `js_form_provider.cpp` |
| `reloadForms` | moduleName, abilityName, formName | 字符串注入 | `js_form_provider.cpp` |

#### formHost 模块 (`app.form.formHost`)

| JS API | 输入参数 | 风险点 | 代码位置 |
|--------|---------|--------|---------|
| `addForm` | formId, want, formHostCallback | Want对象注入 | `js_form_host.cpp` |
| `deleteForm` | formId | 整数溢出 | `js_form_host.cpp` |
| `releaseForm` | formId, cacheFlag | 整数溢出 | `js_form_host.cpp` |
| `requestForm` | formId, want | Want对象注入 | `js_form_host.cpp` |
| `castTempForm` | formId | 整数溢出、权限绕过 | `js_form_host.cpp` |
| `notifyVisibleForms` | formIds[] | 数组溢出 | `js_form_host.cpp` |
| `acquireFormState` | want | Want对象注入 | `js_form_host.cpp` |
| `shareForm` | formId, deviceId | 设备ID注入 | `js_form_host.cpp` |
| `setRouterProxy` | formIds[], callback | 数组注入 | `js_form_host.cpp` |
| `updateFormSize` | formId, width, height | 浮点溢出 | `js_form_host.cpp` |

#### formObserver 模块 (`app.form.formObserver`)

| JS API | 输入参数 | 风险点 | 代码位置 |
|--------|---------|--------|---------|
| `on` | type, callback | 类型字符串注入 | `js_form_observer.cpp` |
| `getRunningFormInfos` | filter | 过滤条件注入 | `js_form_observer.cpp` |
| `getRunningFormInfoById` | formId | 整数溢出 | `js_form_observer.cpp` |

**证据**:
```cpp
// frameworks/js/napi/form_host/js_form_host.cpp
// addForm 接收JS参数
napi_value JsFormHost::AddForm(napi_env env, napi_callback_info info) {
    // 解析JS参数: formId, want, callback
    // 风险: formId可能为负数或超大值
    // 风险: want对象可能包含恶意数据
}
```

### 2.2 IPC 输入入口

#### IFormMgr 接口 (服务侧接收)

| 方法 | 输入参数 | 代码位置 |
|------|---------|---------|
| `AddForm` | formId, Want, callerToken | `form_mgr_service.cpp:156` |
| `DeleteForm` | formId, callerToken | `form_mgr_service.cpp:218` |
| `UpdateForm` | formId, FormProviderData | `form_mgr_service.cpp:342` |
| `RequestForm` | formId, callerToken, Want | `form_mgr_service.cpp:398` |
| `CastTempForm` | formId, callerToken | `form_mgr_service.cpp:454` |
| `ShareForm` | formId, deviceId, callerToken | `form_mgr_service.cpp:1423` |
| `MessageEvent` | formId, Want, callerToken | `form_mgr_service.cpp:823` |
| `RouterEvent` | formId, Want, callerToken | `form_mgr_service.cpp:915` |

**证据**:
```cpp
// services/src/form_mgr/form_mgr_service.cpp:156
int FormMgrService::AddForm(
    const int64_t formId,           // ← 外部输入
    const Want &want,               // ← 外部输入
    const sptr<IRemoteObject> &callerToken,
    FormJsInfo &formInfo)
{
    // 需要验证formId范围和want内容
}
```

#### IPC Code 范围

```cpp
// interfaces/inner_api/include/form_mgr_interface.h:1063-1182
enum class Message {
    FORM_MGR_ADD_FORM = 3001,           // 添加卡片
    FORM_MGR_DELETE_FORM,               // 删除卡片
    FORM_MGR_UPDATE_FORM,               // 更新卡片
    FORM_MGR_REQUEST_FORM,              // 请求卡片
    FORM_MGR_CAST_TEMP_FORM,            // 临时卡片转换
    FORM_MGR_MESSAGE_EVENT,             // 消息事件
    FORM_MGR_ROUTER_EVENT,              // 路由事件
    FORM_MGR_SHARE_FORM,                // 分享卡片
    // ... 100+ 个IPC操作码
};
```

### 2.3 配置文件输入

| 文件 | 格式 | 解析器 | 风险点 |
|------|------|--------|--------|
| form_config.xml | XML | `form_xml_parser.cpp` | XML注入、XXE |

**证据**:
```cpp
// services/config/form_xml_parser.cpp
// 解析XML配置文件
```

### 2.4 数据库输入

| 数据库 | 表 | 操作 | 风险点 |
|--------|-----|------|--------|
| formdb | form_info | 查询/插入/更新 | SQL注入(已参数化) |
| formdb | form_record | 查询/插入/更新 | 数据完整性 |

**证据**:
```cpp
// services/src/data_center/database/form_rdb_data_mgr.cpp
// 使用RDB接口，已参数化查询
```

---

## 3. 敏感操作清单

### 3.1 权限敏感操作

| 操作 | 所需权限 | 代码位置 | 影响 |
|------|---------|---------|------|
| AddForm | `ohos.permission.REQUIRE_FORM` | `form_mgr_service.cpp:1000` | 添加卡片到系统 |
| DeleteForm | `ohos.permission.REQUIRE_FORM` | `form_mgr_service.cpp:1000` | 删除卡片 |
| CastTempForm | `ohos.permission.AGENT_REQUIRE_FORM` | `form_mgr_service.cpp:1768` | 临时卡片转换 |
| RegisterObserver | `ohos.permission.OBSERVE_FORM_RUNNING` | `form_mgr_service.cpp:1183` | 注册观察器 |
| GetFormsInfo | `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | `form_info_mgr.cpp:481` | 获取表单信息 |

**证据**:
```cpp
// services/src/form_mgr/form_mgr_service.cpp:1000
ErrCode FormMgrService::CheckFormPermission(const std::string &permission) {
    // 1. 检查SA调用
    if (FormUtil::IsSACall()) {
        return ERR_OK;
    }
    
    // 2. 检查系统应用
    if (!CheckCallerIsSystemApp()) {
        return ERR_APPEXECFWK_FORM_PERMISSION_DENY_SYS;
    }
    
    // 3. 验证具体权限
    auto isCallingPerm = FormUtil::VerifyCallingPermission(permission);
    if (!isCallingPerm) {
        return ERR_APPEXECFWK_FORM_PERMISSION_DENY;
    }
    
    // 4. 检查跨账户权限
    if (!CheckAcrossLocalAccountsPermission()) {
        return ERR_APPEXECFWK_FORM_PERMISSION_DENY;
    }
    
    return ERR_OK;
}
```

### 3.2 系统调用

| 调用 | 目的 | 代码位置 |
|------|------|---------|
| GetCallingTokenID | 获取调用者身份 | `form_util.cpp:227` |
| GetCallingUid | 获取调用者UID | 多处 |
| VerifyAccessToken | 权限验证 | `form_util.cpp:249` |
| StartAbility | 启动Ability | `form_mgr_service.cpp` |

### 3.3 跨进程通信

| 通信目标 | 接口 | 风险 |
|---------|------|------|
| AMS (Ability Manager) | IAbilityManager | 拉起应用 |
| BMS (Bundle Manager) | IBundleMgr | 获取应用信息 |
| FRS (Form Render Service) | IFormRender | 渲染卡片 |

---

## 4. 信任边界跨越点

### 4.1 边界跨越分析

| 跨越点 | 源域 | 目标域 | 验证机制 | 风险 |
|--------|------|--------|---------|------|
| N-API调用 | 应用JS | N-API层 | 无 | 参数需校验 |
| IPC调用 | N-API层 | 服务层 | callerToken | 权限检查 |
| 服务内部 | 服务层 | 数据层 | 内部调用 | 数据验证 |

### 4.2 数据流图

```
[应用JS代码]
    │
    │ ① N-API调用 (无权限检查)
    ▼
[N-API层] ─────────────────────────────┐
    │                                     │
    │ ② 参数序列化                        │ ④ 回调
    ▼                                     │
[IPC Proxy]                              │
    │                                     │
    │ ③ Binder传输 (带callerToken)       │
    ▼                                     │
[IPC Stub]                               │
    │                                     │
    │ ⑤ 权限检查 (VerifyCallingPermission)│
    ▼                                     │
[FormMgrService]                         │
    │                                     │
    │ ⑥ 业务处理                          │
    ▼                                     │
[FormDataMgr/FormProviderMgr]            │
    │                                     │
    └─────────────────────────────────────┘
```

---

## 5. 攻击向量总结

### 5.1 潜在攻击路径

#### 路径1: 权限绕过
```
恶意应用 → 构造特殊formId → IPC调用DeleteForm → 未正确验证 → 删除他人卡片
```

#### 路径2: 输入注入
```
恶意应用 → 构造恶意Want对象 → addForm调用 → Want解析漏洞 → 代码执行
```

#### 路径3: 拒绝服务
```
恶意应用 → 高频调用updateForm → 服务过载 → 系统资源耗尽
```

#### 路径4: 信息泄露
```
恶意应用 → 调用getFormsInfo → 权限检查不完整 → 获取其他应用卡片信息
```

### 5.2 攻击面热力图

```
风险等级：🔴 高 🟡 中 🟢 低

                    N-API层    IPC层    服务层    数据层
formProvider API    🟡         🔴        🟡        🟢
formHost API        🟡         🔴        🔴        🟡
formObserver API    🟢         🟡        🟡        🟢
配置文件            🟢         🟢        🟢        🟢
数据库              -          -         🟡        🟡
```

---

## 6. 安全测试建议

### 6.1 测试重点

1. **N-API输入验证测试**
   - formId边界值测试（负数、0、超大值）
   - Want对象畸形数据测试
   - 字符串长度溢出测试

2. **IPC权限绕过测试**
   - 伪造callerToken测试
   - 跨用户访问测试
   - 权限组合测试

3. **拒绝服务测试**
   - 高频调用测试
   - 大数据量传输测试
   - 资源耗尽测试

### 6.2 测试工具

- Fuzzing: 针对N-API和IPC接口进行模糊测试
- Static Analysis: 使用代码静态分析工具检查输入验证
- Dynamic Analysis: 运行时监控权限检查覆盖率

---

*本文档与代码同步更新，新发现的攻击面应及时补充。*

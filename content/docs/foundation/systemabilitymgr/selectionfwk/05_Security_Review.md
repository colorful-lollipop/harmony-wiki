# 安全风险评审

本文档对划词服务子系统进行全面的安全风险分析，识别潜在攻击面、信任边界、数据流，并基于代码证据提出安全风险点及修复建议。

## 1. 威胁模型概述

### 1.1 系统定位与信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界 (Trust Boundary)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────┐                    ┌─────────────────────────────┐ │
│  │   User Space        │                    │     Kernel Space            │ │
│  │                     │                    │                             │ │
│  │  ┌───────────────┐ │                    │  ┌───────────────────────┐ │ │
│  │  │ Selection      │ │                    │  │  Binder Driver         │ │ │
│  │  │ Service (SA)   │ │◄── IPC (Binder) ──►│  │                        │ │ │
│  │  │ ID: 8500      │ │                    │  └───────────────────────┘ │ │
│  │  └───────┬───────┘ │                    │                             │ │
│  │          │         │                    │  ┌───────────────────────┐ │ │
│  │          ▼         │                    │  │  Input Driver         │ │ │
│  │  ┌───────────────┐ │                    │  │  (Mouse/Keyboard)     │ │ │
│  │  │ N-API Modules │ │                    │  └───────────────────────┘ │ │
│  │  └───────┬───────┘ │                    │                             │ │
│  │          │         │                    └─────────────────────────────┘ │
│  │          ▼         │                                                   │
│  │  ┌───────────────┐ │                                                   │
│  │  │   JS Apps     │ │                                                   │
│  │  └───────────────┘ │                                                   │
│  │                     │                                                   │
│  └─────────────────────┘                                                   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                         外部系统交互边界                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面清单

| 攻击面类型 | 入口点 | 暴露接口 | 信任级别 |
|-----------|--------|---------|---------|
| **IPC 调用** | Binder | ISelectionService.idl | 高 (系统进程) |
| **N-API** | JS 接口 | selectionInput.* | 中 (应用进程) |
| **输入事件** | InputManager | 鼠标/键盘事件 | 高 (系统级) |
| **剪贴板** | Pasteboard | GetPasteboardData | 中 (跨应用) |
| **配置文件** | Init | selection_service.cfg | 高 (系统配置) |
| **数据库** | RDB | SelectionConfigDatabase | 中 (用户数据) |

## 2. 数据流分析

### 2.1 划词数据流

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              数据流图                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [User Selection]                                                           │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    Input System                                      │    │
│  │  - Mouse Double Click                                               │    │
│  │  - Pointer Drag                                                    │    │
│  │  - Selection Update                                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │               SelectionInputMonitor                                  │    │
│  │  1. 事件捕获                                                        │    │
│  │  2. 状态机判断                                                      │    │
│  │  3. 区域坐标计算                                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │               Pasteboard Service                                     │    │
│  │  1. 获取剪贴板数据                                                   │    │
│  │  2. 数据类型校验 (必须是纯文本)                                       │    │
│  │  3. 长度限制 (≤6000 bytes)                                          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │               SelectionService (SA)                                   │    │
│  │  1. 应用校验 (SelectionAppValidator)                                │    │
│  │  2. 数据封装 (SelectionInfoData)                                    │    │
│  │  3. IPC 回调 (ISelectionListener)                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │               N-API Binding Layer                                    │    │
│  │  1. 参数序列化                                                      │    │
│  │  2. 回调触发 (JSCallbackObject)                                     │    │
│  │  3. Promise resolve                                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       │                                                                     │
│       ▼                                                                     │
│  [JS Application]                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 3. 安全风险点分析

### 3.1 风险清单

#### 风险 1: 输入事件伪造 (MEDIUM)

**证据来源**: `service/src/selection_input_monitor.cpp`

```cpp
class SelectionInputMonitor : public MMI::InputObserver {
public:
    virtual void OnInputEvent(std::shared_ptr<MMI::KeyEvent> event) override;
    virtual void OnInputEvent(std::shared_ptr<MMI::PointerEvent> event) override;
    // 未验证输入事件的来源进程 (pid/uid)
};
```

**风险描述**:
- `SelectionInputMonitor` 接收所有 `InputManager` 事件
- 未校验事件来源的 `pid`/`uid` 是否合法
- 恶意应用可能伪造鼠标/键盘事件触发非预期的划词操作

**触发条件**:
1. 恶意应用获取 `INPUT_MANAGER` 权限
2. 向 `SelectionInputMonitor` 注入伪造事件
3. 触发选中非预期文本内容

**影响范围**:
- 用户隐私泄露 (获取非预期的选中内容)
- 面板误弹出 (用户体验)

**修复建议**:
```cpp
// 在 OnInputEvent 中添加来源验证
void SelectionInputMonitor::OnInputEvent(std::shared_ptr<MMI::PointerEvent> event) {
    // 添加: 验证 event->GetSourcePid() 是否为合法输入源
    pid_t sourcePid = event->GetSourcePid();
    if (!IsValidInputSource(sourcePid)) {
        SELECTION_HILOGW("Ignore event from invalid source: %{public}d", sourcePid);
        return;
    }
    
    // 原有逻辑
    // ...
}
```

#### 风险 2: 剪贴板数据未完全消毒 (HIGH)

**证据来源**: `service/src/selection_service.cpp` + `common/selection_data_inner.h`

```cpp
// selection_data_inner.h:84
if (!out.WriteString(data.bundleName)) {
    return false;
}
```

**风险描述**:
- `SelectionInfoData` 通过 `Parcelable` 序列化跨进程传递
- `bundleName` 字段直接写入 Parcel，未验证内容安全性
- 恶意服务端可能注入恶意 `bundleName` 触发客户端问题

**触发条件**:
1. SelectionService 被攻陷或存在恶意修改
2. 构造恶意 `bundleName` 包含特殊字符 (`, ; | &` 等)
3. JS 端接收后未校验直接使用

**影响范围**:
- 客户端应用崩溃
- 潜在的代码注入风险

**修复建议**:
```cpp
// 在 Marshalling 前添加 bundleName 校验
bool SelectionInfoData::Marshalling(Parcel& out) const {
    // 添加: 白名单校验 bundleName 格式
    if (!IsValidBundleName(data.bundleName)) {
        SELECTION_HILOGE("Invalid bundleName: %{public}s", data.bundleName.c_str());
        return false;
    }
    
    // 原有序列化逻辑
    // ...
}
```

#### 风险 3: IPC 参数校验不完整 (MEDIUM)

**证据来源**: `interfaces/idl/ISelectionService.idl`

```idl
interface OHOS.SelectionFwk.ISelectionService {
    void IsCurrentSelectionApp([in] int pid, [out] boolean resultValue);
    void GetSelectionContent([out] String selectionContent);
    void SetPanelShowingStatus([in] boolean status);
};
```

**风险描述**:
- `pid` 参数为 `int` 类型，未限定范围
- IPC 调用端可能传入负数或超大 `pid` 值
- 服务端未进行边界检查

**触发条件**:
1. 恶意应用调用 `IsCurrentSelectionApp(-1)`
2. 传入超大 `pid` 值 (如 `INT_MAX`)
3. 服务端 `GetCallingPid()` 缓存可能出错

**影响范围**:
- 服务端逻辑错误
- 状态不一致

**修复建议**:
```cpp
// 在 SelectionService::IsCurrentSelectionApp 中添加校验
ErrCode SelectionService::IsCurrentSelectionApp(int pid, bool& resultValue)
{
    // 添加: PID 范围校验
    if (pid < 0 || pid > SELECTION_MAX_PID) {
        SELECTION_HILOGE("Invalid pid: %{public}d", pid);
        return SelectionServiceError::INVALID_DATA;
    }
    
    // 原有逻辑
}
```

#### 风险 4: 回调注册缺乏身份验证 (MEDIUM)

**证据来源**: `service/src/selection_service.cpp:183-199`

```cpp
ErrCode SelectionService::RegisterListener(const sptr<ISelectionListener>& listener)
{
    if (!SelectionAppValidator::GetInstance().Validate()) {
        return SelectionServiceError::UNAUTHENTICATED_ERROR;
    }

    pid_.store(IPCSkeleton::GetCallingPid());
    // ... 注册逻辑
}
```

**风险描述**:
- `RegisterListener` 仅通过 `SelectionAppValidator` 校验
- `SelectionAppValidator` 校验逻辑可能依赖包名白名单
- 未校验调用者的 `uid`/`tokenId` 是否匹配包名

**触发条件**:
1. 恶意应用伪造包名通过白名单
2. 注册恶意 `ISelectionListener` 回调
3. 接收其他应用的 `SelectionInfoData`

**影响范围**:
- 跨应用数据泄露
- 隐私信息非授权访问

**修复建议**:
```cpp
ErrCode SelectionService::RegisterListener(const sptr<ISelectionListener>& listener)
{
    if (!SelectionAppValidator::GetInstance().Validate()) {
        return SelectionServiceError::UNAUTHENTICATED_ERROR;
    }

    // 添加: UID 校验
    int32_t callingUid = IPCSkeleton::GetCallingUid();
    int32_t callingPid = IPCSkeleton::GetCallingPid();
    
    // 验证 uid 与包名的一致性
    if (!VerifyAppIdentity(callingPid, callingUid)) {
        SELECTION_HILOGE("App identity verification failed");
        return SelectionServiceError::UNAUTHENTICATED_ERROR;
    }

    pid_.store(callingPid);
    // ... 原有逻辑
}
```

#### 风险 5: 面板创建缺乏内容过滤 (LOW)

**证据来源**: `js_selection_panel.cpp` + `SelectionPanel` 模块

```cpp
// 开发者可以通过 createPanel 创建自定义面板
napi_value JsSelectionAbility::JsCreatePanel()
{
    // 未限制面板的显示内容来源
}
```

**风险描述**:
- `SelectionPanel` 支持自定义 UI 内容
- 恶意划词应用可能利用面板显示钓鱼内容
- 面板可以移动到任意屏幕位置

**触发条件**:
1. 恶意划词应用创建面板
2. 面板显示伪造的登录界面
3. 用户误操作导致信息泄露

**影响范围**:
- 钓鱼攻击
- 用户欺诈

**修复建议**:
```typescript
// 在 PanelInfo 中添加内容安全标记
interface PanelInfo {
    type: PanelType;
    contentSource?: 'system' | 'application';  // 新增: 内容来源限制
    allowedDomains?: string[];                  // 新增: 允许的域名
}
```

#### 风险 6: 配置持久化注入 (LOW)

**证据来源**: `service/src/db_selection_config_repository.cpp`

```cpp
// 配置写入数据库
int32_t DBSelectionConfigRepository::SaveSelectionConfig(const SelectionConfig& config)
{
    // 配置文件可能包含敏感信息
}
```

**风险描述**:
- 用户配置存储在 `SelectionConfigDatabase`
- 配置项包含黑名单/白名单应用列表
- 未加密存储可能被其他应用读取

**触发条件**:
1. 设备 root 或越狱
2. 直接访问数据库文件
3. 读取用户划词偏好设置

**影响范围**:
- 用户隐私泄露
- 划词行为被分析

**修复建议**:
```cpp
// 对敏感配置字段进行加密
int32_t DBSelectionConfigRepository::SaveSelectionConfig(const SelectionConfig& config)
{
    // 敏感字段加密存储
    std::string encrypted = EncryptConfig(config.sensitiveFields);
    // ...
}
```

### 3.2 风险汇总表

| ID | 风险名称 | 严重程度 | 攻击面 | 利用难度 | 状态 |
|----|---------|---------|--------|---------|------|
| SEC-001 | 输入事件伪造 | MEDIUM | InputManager | 中 | 待修复 |
| SEC-002 | 剪贴板数据未消毒 | HIGH | Parcelable | 低 | 待修复 |
| SEC-003 | IPC 参数校验不完整 | MEDIUM | ISelectionService | 低 | 待修复 |
| SEC-004 | 回调注册缺乏认证 | MEDIUM | ISelectionListener | 中 | 待修复 |
| SEC-005 | 面板内容无过滤 | LOW | SelectionPanel | 低 | 待修复 |
| SEC-006 | 配置持久化未加密 | LOW | RDB | 低 | 建议改进 |

## 4. 权限与访问控制

### 4.1 系统能力声明

**证据来源**: `bundle.json:12-14`

```json
"syscap": [
  "SystemCapability.SelectionInput.Selection"
]
```

### 4.2 权限要求

| 能力名称 | 用途 | 保护级别 |
|---------|------|---------|
| `ohos.permission.GET_BUNDLE_INFO` | 获取应用包名 | normal |
| `ohos.permission.USE_DATA` | 访问用户数据 | normal |
| `ohos.permission.INPUT_MONITORING` | 监控输入事件 | system_basic |

### 4.3 应用校验机制

**证据来源**: `service/src/selection_app_validator.cpp`

```cpp
class SelectionAppValidator {
public:
    bool Validate();
    bool IsSelectionApp(const std::string& bundleName);
};
```

## 5. 安全加固措施

### 5.1 已实现的安全措施

| 措施 | 实现位置 | 效果 |
|-----|---------|------|
| CFI 保护 | BUILD.gn `sanitize.cfi` | 控制流完整性 |
| Bounds Check | `bounds_checking_function` | 数组越界防护 |
| ASan/UBSan | BUILD.gn `sanitize` | 内存错误检测 |
| PACRet | BUILD.gn `branch_protector_ret` | 返回地址保护 |

**证据来源**: `service/BUILD.gn:26-35`

```gn
sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
    integer_overflow = true
    ubsan = true
}
```

### 5.2 代码签名

**证据来源**: `bundle.json` 配置

- 所有 `.so` 库经过系统签名
- SELinux 策略限制进程权限

### 5.3 建议新增措施

| 措施 | 优先级 | 说明 |
|-----|-------|------|
| IPC 参数校验强化 | 高 | 补充 pid/uid 范围检查 |
| 数据消毒 | 高 | bundleName 格式验证 |
| 回调解密 | 中 | 回调数据签名验证 |
| 配置加密 | 低 | 敏感配置加密存储 |

## 6. 安全测试建议

### 6.1 模糊测试

**证据来源**: `test/fuzztest/`

```
test/
├── fuzztest/
│   └── selection_service_fuzztest/
└── unittest/
    └── selection_manager_ut/
```

**建议测试用例**:
1. 伪造 IPC 参数 (超大 pid、负数、null)
2. 构造畸形 SelectionInfoData
3. 注入恶意 bundleName
4. 并发回调注册/注销

### 6.2 渗透测试

| 测试项 | 方法 | 目标 |
|-------|------|------|
| 输入注入 | 模拟 MMI 事件 | 事件来源验证 |
| IPC 越权 | 跨 uid 调用 | 权限校验 |
| 数据篡改 | 拦截修改 Parcel | 数据完整性 |
| 配置注入 | 替换数据库 | 配置安全 |

## 7. 安全开发指南

### 7.1 新增 API 安全要求

1. **参数校验**: 所有外部输入必须经过范围、格式校验
2. **身份验证**: IPC 调用需验证调用者身份
3. **最小权限**: 默认拒绝，必要时显式授权
4. **日志脱敏**: 禁止在日志中输出敏感信息

### 7.2 代码审查清单

- [ ] IPC 接口参数是否完整校验?
- [ ] 数据序列化是否进行消毒处理?
- [ ] 回调注册是否验证调用者身份?
- [ ] 敏感配置是否加密存储?
- [ ] 是否使用安全编译选项?

---

**相关链接**:

- [返回 SUMMARY](./SUMMARY.md)
- [架构说明](./02_Architecture.md)
- [N-API 参考](./03_NAPI_Reference.md)
- [构建系统](./04_Build_System.md)

## 附录 A: 检查范围声明

### A.1 本次评审覆盖范围

| 组件 | 覆盖程度 | 说明 |
|-----|---------|------|
| SelectionService (SA) | 完全覆盖 | 核心服务逻辑 |
| N-API Modules | 完全覆盖 | JS 接口绑定 |
| IPC Interfaces | 完全覆盖 | IDL 定义 |
| InputMonitor | 部分覆盖 | 事件处理逻辑 |
| ConfigRepository | 部分覆盖 | 存储逻辑 |

### A.2 本次评审未覆盖范围

| 组件 | 未覆盖原因 |
|-----|-----------|
| MMI 底层驱动 | 超出子系统范围 |
| Pasteboard Service | 依赖其他子系统 |
| WindowManager | 依赖其他子系统 |
| 系统签名机制 | 超出子系统范围 |

### A.3 局限性说明

1. **静态分析限制**: 部分逻辑需要运行时动态分析验证
2. **并发场景**: 并发安全性未进行压力测试
3. **性能影响**: 安全加固可能带来性能开销，需评估

---

**文档版本**: 1.0  
**生成时间**: 2025-02-06  
**评审范围**: selectionfwk 子系统核心代码

# 攻击面分析 (Attack Surface Analysis)

**目的**: 为安全研究员提供划词服务子系统的快速攻击面速查  
**适用范围**: 安全审计、渗透测试、威胁建模  
**更新日期**: 2026-02-07

---

## 1. 攻击面总览

### 1.1 系统边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界 (Trust Boundary)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────┐                    ┌─────────────────────────────┐ │
│  │   应用层 (App)       │                    │     系统服务层              │ │
│  │                     │                    │                             │ │
│  │  ┌───────────────┐  │     N-API        │  ┌───────────────────────┐  │ │
│  │  │ JS划词应用     │◄─┼─────────────────►│  │ N-API Binding         │  │ │
│  │  │ (Extension)    │  │   (JS↔C++)       │  │ (frameworks/js/napi/) │  │ │
│  │  └───────────────┘  │                    │  └───────────┬───────────┘  │ │
│  │                     │                    │              │              │ │
│  └─────────────────────┘                    │              ▼              │ │
│                                             │  ┌───────────────────────┐  │ │
│  ┌─────────────────────┐                    │  │ IPC Client            │  │ │
│  │ 外部输入源           │                    │  │ (selection_client)    │  │ │
│  │                     │                    │  └───────────┬───────────┘  │ │
│  │ • 多模输入(MMI)     │                    │              │              │ │
│  │ • 剪贴板服务        │◄─────────────────►│              │              │ │
│  │ • 系统参数          │     IPC Binder     │              ▼              │ │
│  │ • 配置文件          │                    │  ┌───────────────────────┐  │ │
│  │                     │                    │  │ SelectionService      │  │ │
│  └─────────────────────┘                    │  │ (SA ID: 8500)         │  │ │
│                                             │  └───────────────────────┘  │ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面分类

| 攻击面类型 | 数量 | 风险等级 | 关键入口 |
|-----------|------|---------|---------|
| **N-API接口** | 4个模块 | 中 | `frameworks/js/napi/*` |
| **IPC接口** | 2个IDL | 高 | `ISelectionService.idl` |
| **输入事件** | 3类事件 | 高 | `SelectionInputMonitor` |
| **外部数据** | 4种来源 | 中 | 剪贴板、配置、参数 |

---

## 2. N-API 攻击面

### 2.1 模块清单

| 模块名 | JS导入路径 | C++实现 | 关键方法数 |
|--------|-----------|---------|-----------|
| selectionManager | `@selectionInput.selectionManager` | `js_selection_engine_setting.cpp` | 6+ |
| SelectionPanel | `@selectionInput.SelectionPanel` | `js_selection_panel.cpp` | 8+ |
| SelectionExtensionAbility | `@selectionInput.SelectionExtensionAbility` | `selection_extension_ability_module.cpp` | 2+ |
| SelectionExtensionContext | `@selectionInput.SelectionExtensionContext` | `selection_extension_context_module.cpp` | 1+ |

**证据**: `frameworks/js/napi/selection_ability/selection_engine_module.cpp:39`
```cpp
static napi_module _module = {
    .nm_modname = "selectionInput.selectionManager",
    // ...
};
```

### 2.2 关键API入口点

#### selectionManager 模块

**文件**: `frameworks/js/napi/selection_ability/js_selection_engine_setting.cpp`

| 方法 | 参数类型 | 验证点 | 风险 |
|------|---------|--------|------|
| `on(type, callback)` | string, function | EventChecker验证type | 回调注入 |
| `off(type, callback?)` | string, function? | 类型验证 | 拒绝服务 |
| `getSelectionContent()` | - | 频率限制(50次/500ms) | 资源耗尽 |
| `createPanel(ctx, info)` | Context, PanelInfo | 坐标/尺寸验证 | 参数绕过 |
| `destroyPanel(panel)` | Panel | 实例验证 | UAF风险 |

**证据**: `js_selection_engine_setting.cpp:46-47`
```cpp
static constexpr int MAX_CALLS_PER_WINDOW = 50;
static constexpr int WINDOW_MS = 500;
```

#### SelectionPanel 模块

**文件**: `frameworks/js/napi/selection_ability/js_panel.cpp`

| 方法 | 参数类型 | 验证点 | 风险 |
|------|---------|--------|------|
| `show()` | - | 面板状态检查 | 状态混淆 |
| `hide()` | - | 面板状态检查 | 状态混淆 |
| `moveTo(x, y)` | number, number | x>=0, y>=0 | 坐标注入 |
| `startMoving()` | - | 权限检查 | 未授权操作 |

**证据**: `js_panel.cpp:263-264`
```cpp
if (x < 0 || y < 0) {
    return ThrowError(env, SFErrorCode::EXCEPTION_PARAMCHECK, "x and y must be non-negative");
}
```

---

## 3. IPC 攻击面

### 3.1 服务接口

**文件**: `interfaces/idl/ISelectionService.idl:18-24`

```idl
interface OHOS.SelectionFwk.ISelectionService {
    void RegisterListener([in] ISelectionListener listener);
    void UnregisterListener([in] ISelectionListener listener);
    void IsCurrentSelectionApp([in] int pid, [out] boolean resultValue);
    void GetSelectionContent([out] String selectionContent);
    void SetPanelShowingStatus([in] boolean status);
}
```

### 3.2 IPC接口风险分析

| 方法 | 输入参数 | 风险点 | 证据位置 |
|------|---------|--------|---------|
| `RegisterListener` | `ISelectionListener` | 回调对象注入 | `selection_service.cpp:183-199` |
| `IsCurrentSelectionApp` | `int pid` | PID范围未校验 | `ISelectionService.idl:21` |
| `GetSelectionContent` | - | 数据泄露风险 | - |
| `SetPanelShowingStatus` | `boolean` | 状态篡改 | - |

**证据**: `service/src/selection_service.cpp:241-249`
```cpp
ErrCode SelectionService::RegisterListener(const sptr<ISelectionListener>& listener)
{
    if (!SelectionAppValidator::GetInstance().Validate()) {
        return SelectionServiceError::UNAUTHENTICATED_ERROR;
    }
    pid_.store(IPCSkeleton::GetCallingPid());
    // ...
}
```

---

## 4. 输入事件攻击面

### 4.1 事件监听架构

**文件**: `service/include/selection_input_monitor.h`

```cpp
class SelectionInputMonitor : public IInputEventConsumer {
public:
    virtual void OnInputEvent(std::shared_ptr<KeyEvent> keyEvent) const;
    virtual void OnInputEvent(std::shared_ptr<PointerEvent> pointerEvent) const;
    virtual void OnInputEvent(std::shared_ptr<AxisEvent> axisEvent) const;
};
```

### 4.2 输入事件处理状态机

**文件**: `service/include/selection_input_monitor.h:38-55`

```cpp
enum class SelectInputState : uint32_t {
    SELECT_INPUT_INITIAL = 0,
    SELECT_INPUT_WORD_BEGIN,
    SELECT_INPUT_WAIT_LEFT_MOVE,
    SELECT_INPUT_WAIT_DOUBLE_CLICK,
    SELECT_INPUT_WAIT_TRIPLE_CLICK,
    SELECT_INPUT_WORD_END,
    SELECT_INPUT_DONE,
};
```

### 4.3 关键常量

**文件**: `service/include/selection_input_monitor.h:32-36`

| 常量 | 值 | 安全意义 |
|------|-----|---------|
| `DOUBLE_CLICK_TIME` | 550ms | 双击超时窗口 |
| `TRIPLE_CLICK_TIME` | 300ms | 三击超时窗口 |
| `MAX_PASTERBOARD_TEXT_LENGTH` | 2000 | 剪贴板文本上限 |
| `MAX_POSITION_CHANGE_OFFSET` | 10 | 位置容差 |

### 4.4 输入事件风险

| 风险 | 描述 | 利用路径 |
|------|------|---------|
| 事件伪造 | 未验证事件来源PID | 恶意应用注入MMI事件 |
| 状态机混淆 | 状态转换缺乏保护 | 异常事件序列导致状态混乱 |
| 时序攻击 | 依赖时间戳判断 | 时间戳伪造 |

---

## 5. 外部数据攻击面

### 5.1 剪贴板数据

**数据来源**: Pasteboard Service  
**处理位置**: `service/src/selection_input_monitor.cpp`

```cpp
class SelectionPasteboardDisposableObserver : public PasteboardDisposableObserver {
    void OnTextReceived(const std::string &text, int32_t errCode) override;
};
```

**风险**:
- 文本长度限制: MAX_PASTERBOARD_TEXT_LENGTH (2000字节)
- 内容类型: 纯文本(需校验)
- 编码: UTF-8(需验证)

### 5.2 系统参数

**配置项**: `sa_profile/8500.json:13-35`

| 参数名 | 类型 | 用途 | 风险 |
|--------|------|------|------|
| `sys.selection.switch` | string | 服务开关 | 未授权修改 |
| `sys.selection.trigger` | string | 触发方式 | 配置注入 |
| `sys.selection.app` | string | 当前应用 | 应用劫持 |
| `sys.selection.timeout` | string | 超时配置 | DoS |

**证据**: `service/include/selection_service.h:41-44`
```cpp
constexpr const char *SYS_SELECTION_SWITCH = "sys.selection.switch";
constexpr const char *SYS_SELECTION_TRIGGER = "sys.selection.trigger";
constexpr const char *SYS_SELECTION_APP = "sys.selection.app";
constexpr const char *SYS_SELECTION_TIMEOUT = "sys.selection.timeout";
```

### 5.3 配置文件

**位置**: `etc/para/selection_para`

**风险**: 配置文件篡改导致服务异常

### 5.4 数据库

**类型**: Relational Store (RDB)  
**用途**: 用户配置持久化  
**风险**: SQL注入、数据泄露

**证据**: `service/include/selection_config_database.h`
```cpp
class SelectionConfigDatabase {
    int32_t SaveSelectionConfig(const SelectionConfig& config);
    int32_t GetSelectionConfig(SelectionConfig& config);
};
```

---

## 6. 信任边界跨越点

### 6.1 边界点清单

| 边界 | 跨越方向 | 验证机制 | 风险等级 |
|------|---------|---------|---------|
| JS → C++ | 应用→框架 | 类型检查 | 中 |
| C++ → IPC | 框架→服务 | BundleName验证 | 中 |
| IPC → SA | 客户端→服务端 | AppValidator | 高 |
| MMI → SA | 输入→服务 | 无来源验证 | 高 |
| Pasteboard → SA | 外部→服务 | 长度限制 | 中 |

### 6.2 关键验证点

#### 应用身份验证

**文件**: `service/src/selection_app_validator.cpp:34-49`

```cpp
bool SelectionAppValidator::Validate() const
{
    auto currentBundleName = GetCurrentBundleName();
    auto bundleNameFromSys = GetBundleNameFromSys();
    // 比较当前应用包名与系统配置
    return currentBundleName.value() == bundleNameFromSys.value();
}
```

**风险**: 仅验证BundleName，未验证UID/PID一致性

#### 系统应用校验

**文件**: `common/selectionfwk_js_utils.cpp:44`

```cpp
{ EXCEPTION_NOT_SYSTEM_APP, "Permission denied. Called by non-system application."},
```

---

## 7. 敏感操作清单

### 7.1 系统调用

| 操作 | 位置 | 权限要求 | 风险 |
|------|------|---------|------|
| `InputManager::RegisterInputEventMonitor` | `selection_service.cpp` | INPUT_MONITORING | 全局输入监听 |
| `AbilityManager::ConnectAbility` | `selection_service.cpp` | 系统权限 | 启动Extension |
| `Window::Create` | `selection_panel.cpp` | WINDOW_MANAGER | 创建悬浮窗 |
| `RDB::Insert/Update` | `selection_config_database.cpp` | 存储权限 | 数据持久化 |

### 7.2 特权接口调用

**证据**: `service/src/selection_service.cpp:292-350` (OnStart方法)

```cpp
void SelectionService::OnStart() {
    // 注册系统服务
    SystemAbility::MakeAndRegisterAbility(...);
    // 初始化输入监控
    InputMonitorInit();
    // 注册系统事件监听
    SubscribeSysEventReceiver();
}
```

---

## 8. 攻击向量汇总

### 8.1 已识别攻击向量

| 编号 | 攻击向量 | 入口 | 影响 | 利用难度 |
|------|---------|------|------|---------|
| AV-001 | 恶意N-API回调注册 | `RegisterListener` | 数据泄露 | 中 |
| AV-002 | IPC参数注入 | `IsCurrentSelectionApp` | 权限绕过 | 低 |
| AV-003 | 输入事件伪造 | `SelectionInputMonitor` | 非授权触发 | 高 |
| AV-004 | 面板坐标注入 | `moveTo` | UI劫持 | 低 |
| AV-005 | 配置篡改 | `sys.selection.app` | 应用劫持 | 中 |
| AV-006 | 频率限制绕过 | `getSelectionContent` | DoS | 中 |

### 8.2 攻击路径示例

**路径1: 数据窃取**
```
恶意应用注册回调 → 监听selectionCompleted事件 
→ 获取其他应用划词内容 → 泄露敏感信息
```

**路径2: 权限提升**
```
伪造PID参数 → IsCurrentSelectionApp返回true 
→ 获取划词内容权限 → 非授权访问
```

**路径3: UI劫持**
```
创建面板 → moveTo覆盖系统UI 
→ 显示钓鱼内容 → 用户欺诈
```

---

## 9. 安全检查清单

### 9.1 审计检查项

- [ ] N-API参数是否全部经过类型/范围校验
- [ ] IPC调用是否验证调用者身份
- [ ] 输入事件是否验证来源PID
- [ ] 回调注册是否绑定到正确应用
- [ ] 外部数据是否经过消毒处理
- [ ] 敏感配置是否加密存储
- [ ] 日志是否脱敏处理

### 9.2 渗透测试建议

| 测试类型 | 目标 | 方法 |
|---------|------|------|
| 模糊测试 | N-API参数 | 构造畸形参数调用 |
| IPC注入 | ISelectionService | 跨UID调用测试 |
| 事件注入 | SelectionInputMonitor | 模拟MMI事件 |
| 配置篡改 | 系统参数 | 修改para文件测试 |

---

## 10. 相关文档

| 文档 | 内容 |
|------|------|
| [安全风险评估](05_Security_Review.md) | 深度风险分析与修复建议 |
| [架构说明](02_Architecture.md) | 系统架构与信任边界 |
| [N-API参考](03_NAPI_Reference.md) | 完整API接口文档 |

---

**文档版本**: 1.0  
**最后更新**: 2026-02-07  
**证据来源**: 代码静态分析 (selectionfwk 1.0)

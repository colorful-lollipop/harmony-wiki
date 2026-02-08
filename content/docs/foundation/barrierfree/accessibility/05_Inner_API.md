# 内部 API (Inner Kits)

## 目的

本文档说明 Accessibility 子系统的内部 C/C++ API，包括模块接口、依赖方向和稳定性标注。

## 适用范围

- 内部模块开发者
- 框架层开发者
- 服务层开发者
- 贡献者

## 关键结论

### Inner Kits 清单

共 **5 个** Inner Kits：

1. **accessibility_common** - 通用数据类型和常量
2. **accessibility_interface** - IPC 接口（Proxy/Stub/Parcel）
3. **accessibleability** (AAkit) - 无障碍辅助能力 Kit
4. **accessibilityconfig** (ACkit) - 无障碍功能设定 Kit
5. **accessibilityclient** (ASACkit) - 无障碍能力客户端 Kit

### 依赖方向

```
accessibility_common (最底层）
    ↓
accessibility_interface (依赖 common）
    ↓
accessibleability / accessibilityconfig / accessibilityclient (依赖 interface + common）
    ↓
accessibleabilityms / 所有 Kits (依赖以上所有）
```

**无循环依赖**：依赖方向单向，从底层到高层。

### 稳定性标注

| Kit | 稳定性 | 说明 |
|-----|---------|------|
| accessibility_common | **稳定** | 基础数据类型，很少变化 |
| accessibility_interface | **稳定** | IPC 接口，版本化管理 |
| accessibleability | **较稳定** | 扩展能力接口，谨慎升级 |
| accessibilityconfig | **较稳定** | 配置接口，扩展时不破坏 |
| accessibilityclient | **较稳定** | 客户端接口，保持兼容 |

---

## 详细内容

### 1. accessibility_common

#### 模块信息

| 属性 | 值 |
|------|-----|
| 产物名 | libaccessibility_common.so |
| 目标路径 | `//interfaces/innerkits/common:accessibility_common` |
| BUILD.gn | `interfaces/innerkits/common/BUILD.gn` |

**证据**: `interfaces/innerkits/common/BUILD.gn`

#### 主要头文件

| 头文件 | 说明 |
|---------|------|
| `accessibility_def.h` | 基础类型定义（枚举、宏） |
| `accessibility_constants.h` | 常量定义 |
| `accessibility_element_info.h` | 无障碍元素信息结构 |
| `accessibility_event_info.h` | 无障碍事件信息结构 |
| `accessibility_window_info.h` | 窗口信息结构 |

**证据**: `interfaces/innerkits/common/include/`

#### 关键数据结构

##### AccessibilityElementInfo

**主要字段**:
- `elementId` - 元素 ID
- `bundleName` - 包名
- `componentType` - 组件类型
- `contents` - 内容文本
- `description` - 描述
- `checkable` - 可选中
- `clickable` - 可点击
- `focusable` - 可聚焦
- ... 共 50+ 属性

**证据**: `interfaces/innerkits/common/include/accessibility_element_info.h`

##### AccessibilityEventInfo

**主要字段**:
- `eventType` - 事件类型
- `windowId` - 窗口 ID
- `elementId` - 元素 ID
- `componentType` - 组件类型
- `timestamp` - 时间戳
- ... 共 20+ 属性

**证据**: `interfaces/innerkits/common/include/accessibility_event_info.h`

### 2. accessibility_interface

#### 模块信息

| 属性 | 值 |
|------|-----|
| 产物名 | libaccessibility_interface.so |
| 目标路径 | `//common/interface:accessibility_interface` |
| BUILD.gn | `common/interface/BUILD.gn` |

**证据**: `common/interface/BUILD.gn`

#### 主要接口类

##### IAccessibleAbilityClient

**定义位置**: `common/interface/include/iaccessible_ability_client.h`

**职责**: 无障碍能力客户端接口，由 Proxy 调用，Stub 实现。

**主要方法**:
```cpp
virtual int32_t EnableAbility(const std::string &name, bool enable) = 0;
virtual int32_t RegisterInteractionConnection(
    const sptr<IAccessibleAbilityConnection> &connection,
    const sptr<IRemoteObject> &callback) = 0;
virtual int32_t GetEnabledAbility(
    std::vector<AccessibilityAbilityInfo> &abilityInfos) = 0;
// ... 更多方法
```

**证据**: `common/interface/include/iaccessible_ability_client.h`

##### IAccessibleAbilityChannel

**定义位置**: `common/interface/include/iaccessible_ability_channel.h`

**职责**: 无障碍通道接口，用于扩展应用与服务的通信。

**主要方法**:
```cpp
virtual int32_t SearchElementInfoByAccessibilityId(
    const int32_t accessibilityId, AccessibilityElementInfo &elementInfo) = 0;
virtual int32_t ExecuteAction(const AccessibilityActionInfo &actionInfo) = 0;
virtual int32_t GetWindows(std::vector<AccessibilityWindowInfo> &windowInfos) = 0;
// ... 更多方法
```

**证据**: `common/interface/include/iaccessible_ability_channel.h`

##### IAccessibilityElementOperator

**定义位置**: `common/interface/include/iaccessibility_element_operator.h`

**职责**: 元素操作接口，用于操作目标应用的元素。

**主要方法**:
```cpp
virtual int32_t GetFocusElementInfo(AccessibilityElementInfo &elementInfo) = 0;
virtual int32_t SearchElementInfoByContent(
    const std::string &text, std::vector<AccessibilityElementInfo> &elementInfos) = 0;
// ... 更多方法
```

**证据**: `common/interface/include/iaccessibility_element_operator.h`

#### Proxy/Stub 类

| 类类型 | 位置 | 说明 |
|---------|------|------|
| AccessibleAbilityManagerStateObserverProxy | `common/interface/include/accessible_ability_manager_state_observer_proxy.h` | 状态观察者 Proxy |
| AccessibleAbilityChannelProxy | `common/interface/include/accessible_ability_channel_proxy.h` | 通道 Proxy |
| AccessibilityElementOperatorProxy | `common/interface/include/accessibility_element_operator_proxy.h` | 元素操作 Proxy |
| AccessibleAbilityManagerStateObserverStub | `common/interface/include/accessible_ability_manager_state_observer_stub.h` | 状态观察者 Stub |
| AccessibleAbilityChannelStub | `common/interface/include/accessible_ability_channel_stub.h` | 通道 Stub |
| AccessibilityElementOperatorStub | `common/interface/include/accessibility_element_operator_stub.h` | 元素操作 Stub |

**证据**: `common/interface/include/`

### 3. accessibleability (AAkit)

#### 模块信息

| 属性 | 值 |
|------|-----|
| 产物名 | libaccessibleability.so |
| 目标路径 | `//interfaces/innerkits/aafwk:accessibleability` |
| BUILD.gn | `interfaces/innerkits/aafwk/BUILD.gn` |

**证据**: `interfaces/innerkits/aafwk/BUILD.gn`

#### 主要类

##### AccessibleAbilityClient

**定义位置**: `frameworks/aafwk/include/accessible_ability_client_impl.h`

**职责**: 无障碍能力客户端实现，用于无障碍扩展应用。

**主要方法**:
```cpp
int32_t RegisterAbility(const sptr<AccessibleAbilityListener> &listener);
int32_t GetFocusElementInfo(AccessibilityElementInfo &elementInfo);
int32_t SearchElementInfoByContent(
    const std::string &text, std::vector<AccessibilityElementInfo> &elementInfos);
int32_t ExecuteAction(const AccessibilityActionInfo &actionInfo);
```

**证据**: `frameworks/aafwk/include/accessible_ability_client_impl.h`

##### AccessibleAbilityListener

**定义位置**: `interfaces/innerkits/aafwk/include/accessible_ability_listener.h`

**职责**: 无障碍能力监听器接口，回调接口。

**主要方法**:
```cpp
virtual void OnAccessibilityEvent(const AccessibilityEventInfo &eventInfo) = 0;
virtual void OnAbilityStateChange(const bool state) = 0;
```

**证据**: `interfaces/innerkits/aafwk/include/accessible_ability_listener.h`

### 4. accessibilityconfig (ACkit)

#### 模块信息

| 属性 | 值 |
|------|-----|
| 产物名 | libaccessibilityconfig.so |
| 目标路径 | `//interfaces/innerkits/acfwk:accessibilityconfig` |
| BUILD.gn | `interfaces/innerkits/acfwk/BUILD.gn` |

**证据**: `interfaces/innerkits/acfwk/BUILD.gn`

#### 主要类

##### AccessibilityConfig

**定义位置**: `frameworks/acfwk/include/accessibility_config_impl.h`

**职责**: 无障碍配置实现，用于读取和设置无障碍配置。

**主要方法**:
```cpp
bool GetHighContrastTextState() const;
void SetHighContrastTextState(bool state);
bool GetInvertColorState() const;
void SetInvertColorState(bool state);
// ... 共 20+ 配置项
```

**证据**: `frameworks/acfwk/include/accessibility_config_impl.h`

### 5. accessibilityclient (ASACkit)

#### 模块信息

| 属性 | 值 |
|------|-----|
| 产物名 | libaccessibilityclient.so |
| 目标路径 | `//interfaces/innerkits/asacfwk:accessibilityclient` |
| BUILD.gn | `interfaces/innerkits/asacfwk/BUILD.gn` |

**证据**: `interfaces/innerkits/asacfwk/BUILD.gn`

#### 主要类

##### AccessibilitySystemAbilityClient

**定义位置**: `frameworks/asacfwk/include/accessibility_system_ability_client_impl.h`

**职责**: 系统能力客户端实现，用于普通应用使用无障碍能力。

**主要方法**:
```cpp
int32_t RegisterStateCallback(const sptr<IAccessibilityStateCallback> &callback);
int32_t GetEnabledAbilityList(
    std::vector<AccessibilityAbilityInfo> &abilityInfos);
bool IsEnabled() const;
// ... 更多方法
```

**证据**: `frameworks/asacfwk/include/accessibility_system_ability_client_impl.h`

##### AccessibilityElementOperator

**定义位置**: `interfaces/innerkits/asacfwk/include/accessibility_element_operator.h`

**职责**: 元素操作接口，用于查询和操作无障碍元素。

**主要方法**:
```cpp
int32_t GetFocusElementInfo(AccessibilityElementInfo &elementInfo);
int32_t SearchElementInfosByText(
    const std::string &text, std::vector<AccessibilityElementInfo> &elementInfos);
int32_t PerformAction(const AccessibilityActionInfo &actionInfo);
// ... 更多方法
```

**证据**: `interfaces/innerkits/asacfwk/include/accessibility_element_operator.h`

### 模块依赖关系

```
┌─────────────────────────────────────────┐
│         Kits (对外接口）             │
│  (N-API / ANI / CJ)                  │
└───────────────┬──────────────────────┘
                │
┌───────────────▼──────────────────────┐
│   Inner Kits (C/C++ API）          │
│                                     │
│  ┌────────────┐ ┌───────────┐ ┌───┐│
│  │ASACkit    │ │  ACkit   │ │AAkit││
│  │(client)   │ │(config)   │ │     ││
│  └──────┬─────┘ └─────┬─────┘ └─┬─┘
│         │               │            │    │
└─────────┴───────────────┴────────────┘
                │
┌───────────────▼───────────────────────┐
│   Interface (IPC）                  │
│   accessibility_interface              │
└───────────────┬──────────────────────┘
                │
┌───────────────▼───────────────────────┐
│   Common (基础类型）                 │
│   accessibility_common               │
└───────────────────────────────────────┘
```

### 接口稳定性说明

#### 稳定接口

这些接口预期长期保持兼容：

1. **accessibility_common 中的数据结构**
   - `AccessibilityElementInfo`
   - `AccessibilityEventInfo`
   - `AccessibilityWindowInfo`

**原因**: 基础数据结构，变更会影响所有上层模块。

**证据**: `interfaces/innerkits/common/include/`

2. **IPC 接口**
   - `IAccessibilityElementOperator`
   - `IAccessibilityAbilityChannel`

**原因**: 跨进程接口，版本化管理。

**证据**: `common/interface/include/`

#### 较稳定接口

这些接口预期谨慎升级，保持向后兼容：

1. **AAkit 接口**
   - `AccessibleAbilityClient`
   - `AccessibleAbilityListener`

2. **ASACkit 接口**
   - `AccessibilitySystemAbilityClient`
   - `AccessibilityElementOperator`

3. **ACkit 接口**
   - `AccessibilityConfig`

**原因**: 对外 API，扩展时不破坏兼容性。

**证据**: `interfaces/innerkits/*/include/`

### 可替换点

| 模块 | 可替换点 | 说明 |
|-------|---------|------|
| ASACkit | 元素查询实现 | 可替换查询算法 |
| ASACkit | 事件分发 | 可优化事件路由 |
| AAMS | 窗口管理 | 可更换窗口管理策略 |
| AAMS | 手势识别 | 可替换手势算法 |

---

## 相关链接

- [项目概览](00_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [GN 构建系统](06_GN_Targets.md)

---

最后更新: 2026-02-06

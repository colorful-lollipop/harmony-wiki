# 对外 N-API 接口

## 目的

本文档提供 Accessibility 子系统对外 N-API 接口的完整清单和详细说明。

## 适用范围

- 使用无障碍 API 的应用开发者
- 需要 API 参考的开发者
- 接口维护人员

## 关键结论

### N-API 模块清单

共 **7 个** N-API 模块：

1. `accessibility` - 主模块（26 个 API）
2. `accessibility.config` - 配置模块（6 个 API + 20 个配置属性）
3. `accessibility.GesturePath` - 手势路径类
4. `accessibility.GesturePoint` - 手势点类
5. `application.AccessibilityExtensionAbility` - 扩展能力
6. `application.AccessibilityExtensionContext` - 扩展上下文（17 个 API）
7. `AccessibilityElement` - 元素类（18 个方法 + 50+ 属性）

### 权限要求

| API 模块 | 需要的权限 |
|-----------|-------------|
| accessibility | 无（大部分操作）
| accessibility.config | `WRITE_ACCESSIBILITY_CONFIG` (写入操作） |
| AccessibilityExtensionContext | `ACCESSIBILITY_EXTENSION_ABILITY` 或 `QUERY_ACCESSIBILITY_ELEMENT` |

### 同步/异步模式

大部分 API 支持 **同步和异步**两种模式：
- 同步：以 `Sync` 后缀命名（如 `isOpenAccessibilitySync`）
- 异步：返回 Promise 或使用 Callback

---

## 详细内容

### 1. accessibility 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/src/native_module.cpp:333` |
| 模块名称 | "accessibility" |
| 导出 API 数 | 26 |
| 导出常量 | AccessibilityEventType (60+), AccessibilityAction (18) |

**证据**: `interfaces/kits/napi/src/native_module.cpp:333`

#### API 清单

| JS API 名称 | C++ 实现函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|-------------|---------------|------------|--------|------|--------|
| isOpenAccessibility | `IsOpenAccessibility` | 异步 | 无 | void | `Promise<boolean>` |
| isOpenAccessibilitySync | `IsOpenAccessibilitySync` | 同步 | 无 | void | `boolean` |
| isOpenTouchGuide | `IsOpenTouchExploration` | 异步 | 无 | void | `Promise<boolean>` |
| isOpenTouchGuideSync | `IsOpenTouchExplorationSync` | 同步 | 无 | void | `boolean` |
| isScreenReaderOpenSync | `IsScreenReaderOpenSync` | 同步 | 无 | void | `boolean` |
| getTouchModeSync | `GetTouchModeSync` | 同步 | 无 | void | `number` |
| getAbilityLists | `GetAbilityList` | 异步 | 无 | void | `Promise<AbilityInfo[]>` |
| getAccessibilityExtensionList | `GetAccessibilityExtensionList` | 异步 | 无 | void | `Promise<AbilityInfo[]>` |
| getAccessibilityExtensionListSync | `GetAccessibilityExtensionListSync` | 同步 | 无 | void | `AbilityInfo[]` |
| on | `SubscribeState` | 异步 | 无 | event, callback | `boolean` |
| off | `UnsubscribeState` | 异步 | 无 | event, callback | `boolean` |
| sendEvent | `SendEvent` | 异步 | 无 | eventInfo | `Promise<boolean>` |
| sendAccessibilityEvent | `SendAccessibilityEvent` | 异步 | 无 | eventInfo | `Promise<boolean>` |
| getCaptionsManager | `GetCaptionsManager` | 同步 | 无 | void | `CaptionsManager` |
| onAudioMonoStateChange | `SubscribeStateAudioMonoState` | 异步 | 无 | callback | `boolean` |
| offAudioMonoStateChange | `UnsubscribeStateAudioMonoState` | 异步 | 无 | callback | `boolean` |
| isAudioMonoEnabled | `GetAudioMonoState` | 异步 | 无 | void | `Promise<boolean>` |
| isAudioMonoEnabledSync | `GetAudioMonoStateSync` | 同步 | 无 | void | `boolean` |
| onAnimationReduceStateChange | `SubscribeStateAnimationReduce` | 异步 | 无 | callback | `boolean` |
| offAnimationReduceStateChange | `UnsubscribeStateAnimationReduce` | 异步 | 无 | callback | `boolean` |
| isAnimationReduceEnabled | `GetAnimationOffState` | 异步 | 无 | void | `Promise<boolean>` |
| isAnimationReduceEnabledSync | `GetAnimationOffStateSync` | 同步 | 无 | void | `boolean` |
| onFlashReminderStateChange | `SubscribeStateFlashReminder` | 异步 | 无 | callback | `boolean` |
| offFlashReminderStateChange | `UnsubscribeStateFlashReminder` | 异步 | 无 | callback | `boolean` |
| isFlashReminderEnabled | `GetFlashReminderSwitch` | 异步 | 无 | void | `Promise<boolean>` |
| isFlashReminderEnabledSync | `GetFlashReminderSwitchSync` | 同步 | 无 | void | `boolean` |

**证据**: `interfaces/kits/napi/src/native_module.cpp:259-287`

#### 常量定义

##### AccessibilityEventType (部分)

| 常量名称 | 值 | 说明 |
|-----------|-----|------|
| TYPE_ACCESSIBILITY_FOCUS | 1 | 无障碍焦点 |
| TYPE_FOCUS | 2 | 焦点 |
| TYPE_CLICK | 3 | 点击 |
| TYPE_LONG_CLICK | 4 | 长按 |
| TYPE_TEXT_UPDATE | 5 | 文本更新 |
| TYPE_WINDOW_ADD | 6 | 窗口添加 |
| TYPE_WINDOW_REMOVE | 7 | 窗口移除 |
| ... | ... | 共 60+ 种 |

**证据**: `interfaces/kits/napi/src/native_module.cpp:64-217`

##### AccessibilityAction

| 常量名称 | 值 | 说明 |
|-----------|-----|------|
| ACCESSIBILITY_FOCUS | 0x01 | 无障碍焦点 |
| CLEAR_ACCESSIBILITY_FOCUS | 0x02 | 清除无障碍焦点 |
| FOCUS | 0x03 | 焦点 |
| CLICK | 0x04 | 点击 |
| LONG_CLICK | 0x05 | 长按 |
| SELECT | 0x06 | 选择 |
| SCROLL_FORWARD | 0x10 | 向前滚动 |
| SCROLL_BACKWARD | 0x11 | 向后滚动 |
| ... | ... | 共 18 种 |

**证据**: `interfaces/kits/napi/src/native_module.cpp:219-252`

#### CaptionsManager 类

| 属性 | getter | setter | 说明 |
|------|--------|--------|------|
| enabled | GetCaptionStateEnabled | SetCaptionStateEnabled | 字幕启用状态 |
| style | GetCaptionStyle | SetCaptionStyle | 字幕样式 |

**证据**: `interfaces/kits/napi/src/napi_accessibility_system_ability_client.cpp:958-981`

#### CaptionsStyle 类

| 属性 | getter | setter | 类型 | 说明 |
|------|--------|--------|------|------|
| fontFamily | GetCaptionsFontFamily | SetCaptionsFontFamily | string | 字体族 |
| fontScale | GetCaptionsFontScale | SetCaptionsFontScale | number | 字体缩放 |
| fontColor | GetCaptionFrontColor | SetCaptionFrontColor | number | 字体颜色 |
| fontEdgeType | GetCaptionFontEdgeType | SetCaptionFontEdgeType | number | 字体边缘类型 |
| backgroundColor | GetCaptionBackgroundColor | SetCaptionBackgroundColor | number | 背景颜色 |
| windowColor | GetCaptionWindowColor | SetCaptionWindowColor | number | 窗口颜色 |

**证据**: `interfaces/kits/napi/src/napi_accessibility_system_ability_client.cpp:1559-1591`

### 2. accessibility.config 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/accessibility_config/src/native_module.cpp:698` |
| 模块名称 | "accessibility.config" |
| 导出 API 数 | 6 |
| 配置属性数 | 20 |

**证据**: `interfaces/kits/napi/accessibility_config/src/native_module.cpp:698`

#### API 清单

| JS API 名称 | C++ 实现函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|-------------|---------------|------------|--------|------|--------|
| on | `SubscribeState` | 异步 | `WRITE_ACCESSIBILITY_CONFIG` | type, callback | `boolean` |
| off | `UnsubscribeState` | 异步 | `WRITE_ACCESSIBILITY_CONFIG` | type, callback | `boolean` |
| enableAbility | `EnableAbility` | 异步 | `WRITE_ACCESSIBILITY_CONFIG` | name, enable | `Promise<boolean>` |
| enableAbilityWithCallback | `EnableAbilityWithCallback` | 异步（Callback） | `WRITE_ACCESSIBILITY_CONFIG` | name, enable, callback | `boolean` |
| disableAbility | `DisableAbility` | 异步 | `WRITE_ACCESSIBILITY_CONFIG` | name | `Promise<boolean>` |
| setMagnificationState | `SetMagnificationState` | 异步 | `WRITE_ACCESSIBILITY_CONFIG` | state | `Promise<boolean>` |

**证据**: `interfaces/kits/napi/accessibility_config/src/native_module.cpp:635-642`

#### 配置属性

| 属性名称 | 配置类型 | 权限 | 说明 |
|----------|---------|--------|------|
| highContrastText | CONFIG_HIGH_CONTRAST_TEXT | `WRITE_ACCESSIBILITY_CONFIG` | 高对比度文本 |
| invertColor | CONFIG_INVERT_COLOR | `WRITE_ACCESSIBILITY_CONFIG` | 反色 |
| daltonizationState | CONFIG_DALTONIZATION_STATE | `WRITE_ACCESSIBILITY_CONFIG` | 色盲状态 |
| daltonizationColorFilter | CONFIG_DALTONIZATION_COLOR_FILTER | `WRITE_ACCESSIBILITY_CONFIG` | 色盲颜色滤镜 |
| contentTimeout | CONFIG_CONTENT_TIMEOUT | `WRITE_ACCESSIBILITY_CONFIG` | 内容超时 |
| animationOff | CONFIG_ANIMATION_OFF | `WRITE_ACCESSIBILITY_CONFIG` | 动画关闭 |
| brightnessDiscount | CONFIG_BRIGHTNESS_DISCOUNT | `WRITE_ACCESSIBILITY_CONFIG` | 亮度折扣 |
| screenMagnifier | CONFIG_SCREEN_MAGNIFICATION | `WRITE_ACCESSIBILITY_CONFIG` | 屏幕放大镜 |
| audioMono | CONFIG_AUDIO_MONO | `WRITE_ACCESSIBILITY_CONFIG` | 音频单声道 |
| audioBalance | CONFIG_AUDIO_BALANCE | `WRITE_ACCESSIBILITY_CONFIG` | 音频平衡 |
| mouseKey | CONFIG_MOUSE_KEY | `WRITE_ACCESSIBILITY_CONFIG` | 鼠标键 |
| mouseAutoClick | CONFIG_MOUSE_AUTOCLICK | `WRITE_ACCESSIBILITY_CONFIG` | 鼠标自动点击 |
| shortkey | CONFIG_SHORT_KEY | `WRITE_ACCESSIBILITY_CONFIG` | 短键 |
| shortkeyTarget | CONFIG_SHORT_KEY_TARGET | `WRITE_ACCESSIBILITY_CONFIG` | 短键目标 |
| shortkeyMultiTargets | CONFIG_SHORT_KEY_MULTI_TARGET | `WRITE_ACCESSIBILITY_CONFIG` | 短键多目标 |
| captions | CONFIG_CAPTION_STATE | `WRITE_ACCESSIBILITY_CONFIG` | 字幕 |
| captionsStyle | CONFIG_CAPTION_STYLE | `WRITE_ACCESSIBILITY_CONFIG` | 字幕样式 |
| clickResponseTime | CONIFG_CLICK_RESPONSE_TIME | `WRITE_ACCESSIBILITY_CONFIG` | 点击响应时间 |
| ignoreRepeatClick | CONFIG_IGNORE_REPEAT_CLICK_STATE | `WRITE_ACCESSIBILITY_CONFIG` | 忽略重复点击 |
| repeatClickInterval | CONFIG_IGNORE_REPEAT_CLICK_TIME | `WRITE_ACCESSIBILITY_CONFIG` | 重复点击间隔 |

**证据**: `interfaces/kits/napi/accessibility_config/src/native_module.cpp:643-662`

### 3. accessibility.GesturePath 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/accessibility_gesture_path/src/native_module.cpp:60` |
| 模块名称 | "accessibility.GesturePath" |
| 导出类 | GesturePath |

**证据**: `interfaces/kits/napi/accessibility_gesture_path/src/native_module.cpp:60`

### 4. accessibility.GesturePoint 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/accessibility_gesture_point/src/native_module.cpp:60` |
| 模块名称 | "accessibility.GesturePoint" |
| 导出类 | GesturePoint |

**证据**: `interfaces/kits/napi/accessibility_gesture_point/src/native_module.cpp:60`

### 5. application.AccessibilityExtensionAbility 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/accessibility_extension/accessibility_extension_module.cpp:23` |
| 模块名称 | "application.AccessibilityExtensionAbility" |
| 导出类 | AccessibilityExtensionAbility |

**证据**: `interfaces/kits/napi/accessibility_extension/accessibility_extension_module.cpp:23`

### 6. application.AccessibilityExtensionContext 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| 注册点 | `interfaces/kits/napi/accessibility_extension_context/accessibility_extension_context_module.cpp:23` |
| 模块名称 | "application.AccessibilityExtensionContext" |
| 导出 API 数 | 17 |

**证据**: `interfaces/kits/napi/accessibility_extension_context/accessibility_extension_context_module.cpp:23`

#### API 清单

| JS API 名称 | C++ 实现函数 | 同步/异步 | 权限 | 参数 | 返回值 |
|-------------|---------------|------------|--------|------|--------|
| setTargetBundleName | `SetTargetBundleName` | 同步 | `ACCESSIBILITY_EXTENSION_ABILITY` | bundleName | `boolean` |
| getFocusElement | `GetFocusElement` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | elementId | `Promise<AccessibilityElement>` |
| getAccessibilityFocusedElement | `GetFocusElementSys` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `Promise<AccessibilityElement>` |
| getWindowRootElement | `GetWindowRootElement` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | windowId | `Promise<AccessibilityElement>` |
| getRootInActiveWindow | `GetWindowRootElementSys` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `Promise<AccessibilityElement>` |
| getWindows | `GetWindows` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `Promise<WindowInfo[]>` |
| getAccessibilityWindowsSync | `GetAccessibilityWindowsSync` | 同步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `WindowInfo[]` |
| injectGesture | `InjectGesture` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | gesturePath | `Promise<boolean>` |
| injectGestureSync | `InjectGestureSync` | 同步 | `ACCESSIBILITY_EXTENSION_ABILITY` | gesturePath | `boolean` |
| startAbility | `StartAbility` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | abilityInfo | `Promise<boolean>` |
| enableScreenCurtain | `EnableScreenCurtain` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | enable | `Promise<boolean>` |
| getElements | `GetElements` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | queryConditions | `Promise<AccessibilityElement[]>` |
| getDefaultFocusedElementIds | `GetDefaultFocusedElementIds` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | windowId | `Promise<number[]>` |
| holdRunningLockSync | `HoldRunningLock` | 同步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `boolean` |
| unholdRunningLockSync | `UnholdRunningLock` | 同步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `boolean` |
| on | `RegisterCallback` | 异步 | 无 | event, callback | `boolean` |
| off | `UnRegisterCallback` | 异步 | 无 | event, callback | `boolean` |
| notifyDisconnect | `NotifyDisconnect` | 异步 | `ACCESSIBILITY_EXTENSION_ABILITY` | void | `Promise<boolean>` |

**证据**: `interfaces/kits/napi/accessibility_extension_context/napi_accessibility_extension_context.cpp:1146-1176`

### 7. AccessibilityElement 类

#### 类信息

| 属性 | 值 |
|------|-----|
| 定义点 | `interfaces/kits/napi/accessibility_extension_module_loader/src/napi_accessibility_element.cpp:298-312` |
| 导出方法数 | 18 |
| 导出属性数 | 50+ |

**证据**: `interfaces/kits/napi/accessibility_extension_module_loader/src/napi_accessibility_element.cpp:298-312`

#### 方法清单

| JS 方法名称 | C++ 实现函数 | 参数 | 返回值 |
|-------------|---------------|------|--------|
| attributeNames | `AttributeNames` | void | `string[]` |
| attributeValue | `AttributeValue` | attributeName | `any` |
| actionNames | `ActionNames` | void | `string[]` |
| enableScreenCurtain | `EnableScreenCurtain` | enable | `boolean` |
| performAction | `PerformAction` | actionName, params | `Promise<boolean>` |
| getCursorPosition | `GetCursorPosition` | void | `{column: number, row: number}` |
| findElement | `FindElement` | condition | `Promise<AccessibilityElement>` |
| findElementById | `FindElementById` | elementId | `Promise<AccessibilityElement>` |
| findElementByContent | `FindElementByContent` | content | `Promise<AccessibilityElement[]>` |
| findElementByFocusDirection | `FindElementByFocusDirection` | direction | `Promise<AccessibilityElement>` |
| findElementsByAccessibilityHintText | `FindElementsByAccessibilityHintText` | text | `Promise<AccessibilityElement[]>` |
| getParent | `GetParent` | void | `Promise<AccessibilityElement>` |
| getChildren | `GetChildren` | void | `Promise<AccessibilityElement[]>` |
| getRoot | `GetRootElement` | void | `Promise<AccessibilityElement>` |
| executeAction | `ExecuteAction` | actionName, params | `Promise<boolean>` |
| findElementsByCondition | `FindElementsByCondition` | condition | `Promise<AccessibilityElement[]>` |

**证据**: `interfaces/kits/napi/accessibility_extension_module_loader/src/napi_accessibility_element.cpp:207-223`

#### 属性清单（部分）

| 属性名称 | 值类型 | 说明 |
|----------|---------|------|
| ACCESSIBILITY_FOCUSED | boolean | 无障碍焦点状态 |
| BUNDLE_NAME | string | 包名 |
| CHECKABLE | boolean | 可选中 |
| CHECKED | boolean | 已选中 |
| CLICKABLE | boolean | 可点击 |
| COMPONENT_ID | number | 组件 ID |
| COMPONENT_TYPE | string | 组件类型 |
| CONTENTS | string | 内容 |
| CURRENT_INDEX | number | 当前索引 |
| DESCRIPTION | string | 描述 |
| EDITABLE | boolean | 可编辑 |
| END_INDEX | number | 结束索引 |
| ERROR | string | 错误信息 |
| FOCUSABLE | boolean | 可聚焦 |
| HINT_TEXT | string | 提示文本 |
| INPUT_TYPE | number | 输入类型 |
| IS_ACTIVE | boolean | 是否激活 |
| IS_ENABLE | boolean | 是否启用 |
| IS_FOCUSED | boolean | 是否聚焦 |
| IS_PASSWORD | boolean | 是否密码 |
| IS_VISIBLE | boolean | 是否可见 |
| ITEM_COUNT | number | 项目数量 |
| LAYER | number | 图层 |
| LONG_CLICKABLE | boolean | 可长按 |
| PAGE_ID | number | 页面 ID |
| PLURAL_LINE_SUPPORTED | boolean | 支持多行 |
| RECT | object | 位置区域 |
| RESOURCE_NAME | string | 资源名称 |
| SCREEN_RECT | object | 屏幕区域 |
| SCROLLABLE | boolean | 可滚动 |
| SELECTED | boolean | 已选择 |
| START_INDEX | number | 开始索引 |
| TEXT | string | 文本 |
| TEXT_LENGTH_LIMIT | number | 文本长度限制 |
| TEXT_MOVE_UNIT | number | 文本移动单位 |
| TYPE | string | 类型 |
| VALUE_MAX | number | 最大值 |
| VALUE_MIN | number | 最小值 |
| VALUE_NOW | number | 当前值 |
| WINDOW_ID | number | 窗口 ID |
| PARENT_ID | number | 父元素 ID |
| CHILDREN_IDS | number[] | 子元素 ID 列表 |

**证据**: `interfaces/kits/napi/accessibility_extension_module_loader/src/napi_accessibility_element.cpp:224-293`

### 参数校验机制

所有 N-API 接口都包含参数校验：

| 校验类型 | 检查点 | 证据 |
|---------|--------|------|
| 类型检查 | N-API 类型自动校验 | N-API 框架自动处理 |
| 空值检查 | napi_is_null / napi_is_undefined | 各实现函数开头 |
| 范围检查 | 数值范围、数组长度 | `napi_accessibility_system_ability_client.cpp` |
| 权限检查 | `CheckPermission()` | `services/aams/src/accessible_ability_manager_service.cpp:1447` |

### 错误码

所有异步操作返回标准的 N-API 错误码：

| 错误类型 | 值 | 说明 |
|---------|-----|------|
| napi_invalid_arg | NAPI_INVALID_ARG | 无效参数 |
| napi_object_expected | NAPI_OBJECT_EXPECTED | 需要对象 |
| napi_string_expected | NAPI_STRING_EXPECTED | 需要字符串 |
| napi_pending_exception | NAPI_PENDING_EXCEPTION | 待处理异常 |

---

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](03_Architecture.md)
- [附录：调用链图](appendix/Callgraphs.md)

---

最后更新: 2026-02-06

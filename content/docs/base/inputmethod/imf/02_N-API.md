# N-API 接口

## 概述

输入法框架提供完整的 N-API 接口，支持 JS/ArkTS 调用。所有 N-API 模块通过 `napi_module_register` 注册。

## 模块列表

| 模块 | 路径 | 注册文件 | 模块名 |
|------|------|----------|--------|
| inputMethod | `frameworks/js/napi/inputmethodclient` | `input_method_module.cpp` | `inputMethod` |
| InputMethodController | `frameworks/js/napi/inputmethodclient` | `input_method_module.cpp` | `InputMethodController` |
| inputMethodList | `frameworks/js/napi/inputmethodlist` | `inputmethodlist.cpp` | `inputMethodList` |
| panel | `frameworks/js/napi/inputmethodpanel` | `input_method_panel_module.cpp` | `panel` |
| inputMethodEngine | `frameworks/js/napi/inputmethodability` | `input_method_engine_module.cpp` | `inputMethodEngine` |
| inputMethodExtensionAbility | `frameworks/js/napi/inputmethod_extension_ability` | `inputmethod_extension_ability_module.cpp` | `inputMethodExtensionAbility` |
| inputMethodExtensionContext | `frameworks/js/napi/inputmethod_extension_context` | `inputmethod_extension_context_module.cpp` | `inputMethodExtensionContext` |
| keyboardPanelManager | `frameworks/js/napi/keyboardpanelmanager` | `keyboard_panel_manager_module.cpp` | `keyboardPanelManager` |

## inputMethod 模块

**注册位置**: `frameworks/js/napi/inputmethodclient/input_method_module.cpp:47`

### 方法列表

| JS 方法 | C++ 实现 | 描述 |
|---------|----------|------|
| `switchInputMethod` | `JsInputMethod::SwitchInputMethod` | 切换输入法 |
| `getCurrentInputMethod` | `JsInputMethod::GetCurrentInputMethod` | 获取当前输入法 |
| `getCurrentInputMethodSubtype` | `JsInputMethod::GetCurrentInputMethodSubtype` | 获取当前输入法子类型 |
| `getDefaultInputMethod` | `JsInputMethod::GetDefaultInputMethod` | 获取默认输入法 |
| `getSystemInputMethodConfigAbility` | `JsInputMethod::GetSystemInputMethodConfigAbility` | 获取输入法配置能力 |
| `switchCurrentInputMethodSubtype` | `JsInputMethod::SwitchCurrentInputMethodSubtype` | 切换当前输入法子类型 |
| `switchCurrentInputMethodAndSubtype` | `JsInputMethod::SwitchCurrentInputMethodAndSubtype` | 切换输入法和子类型 |
| `setSimpleKeyboardEnabled` | `JsInputMethod::SetSimpleKeyboardEnabled` | 设置简单键盘启用 |
| `onAttachmentDidFail` | `JsInputMethod::OnAttachmentDidFail` | 订阅附件失败事件 |
| `offAttachmentDidFail` | `JsInputMethod::OffAttachmentDidFail` | 取消订阅附件失败事件 |

### 静态属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `AttachFailureReason` | Property | 获取附件失败原因 |

### 使用示例

```javascript
import inputMethod from '@ohos.inputMethod';

// 切换输入法
inputMethod.switchInputMethod({
  name: 'com.example.ime',
  id: 'sogou'
});

// 获取当前输入法
let currentMethod = inputMethod.getCurrentInputMethod();
console.log('当前输入法:', currentMethod.name);
```

## InputMethodController 模块

**注册位置**: `frameworks/js/napi/inputmethodclient/input_method_module.cpp`

### 静态方法

| JS 方法 | 描述 |
|---------|------|
| `getInputMethodController` | 获取输入法控制器 |
| `getController` | 获取控制器实例 |

### 静态属性

| 属性 | 描述 |
|------|------|
| `KeyboardStatus` | 键盘状态枚举 |
| `EnterKeyType` | 回车键类型枚举 |
| `TextInputType` | 文本输入类型枚举 |
| `Direction` | 方向枚举 |
| `ExtendAction` | 扩展操作枚举 |
| `EnabledState` | 启用状态枚举 |
| `RequestKeyboardReason` | 请求键盘原因枚举 |
| `CapitalizeMode` | 大写模式枚举 |

### 实例方法

| JS 方法 | 描述 |
|---------|------|
| `attach` | 附加编辑器 |
| `attachWithUIContext` | 使用 UIContext 附加 |
| `detach` | 分离编辑器 |
| `showTextInput` | 显示文本输入 |
| `hideTextInput` | 隐藏文本输入 |
| `setCallingWindow` | 设置调用窗口 |
| `updateCursor` | 更新光标 |
| `changeSelection` | 更改选择 |
| `updateAttribute` | 更新属性 |
| `stopInput` | 停止输入 |
| `stopInputSession` | 停止输入会话 |
| `hideSoftKeyboard` | 隐藏软键盘 |
| `showSoftKeyboard` | 显示软键盘 |
| `on` | 订阅事件 |
| `off` | 取消订阅事件 |
| `sendMessage` | 发送消息 |
| `recvMessage` | 接收消息 |
| `discardTypingText` | 放弃输入文本 |

### 事件类型

| 事件名 | 描述 |
|--------|------|
| `insertText` | 插入文本 |
| `deleteLeft` | 删除左侧字符 |
| `deleteRight` | 删除右侧字符 |
| `sendKeyboardStatus` | 发送键盘状态 |
| `sendFunctionKey` | 发送功能键 |
| `moveCursor` | 移动光标 |
| `handleExtendAction` | 处理扩展操作 |
| `getLeftTextOfCursor` | 获取光标左侧文本 |
| `getRightTextOfCursor` | 获取光标右侧文本 |
| `getTextIndexAtCursor` | 获取光标处文本索引 |

### 使用示例

```javascript
import { InputMethodController } from '@ohos.inputMethod';

let controller = InputMethodController.getController();

// 附加编辑器
controller.attach({
  supportDefaultIme: true
});

// 订阅文本变化事件
controller.on('insertText', (text) => {
  console.log('插入文本:', text);
});

// 发送消息
controller.sendMessage({
  msgId: 'custom_msg',
  msgParam: 'Hello'
});
```

## inputMethodList 模块

**注册位置**: `frameworks/js/napi/inputmethodlist/inputmethodlist.cpp:44`

### 描述

提供输入法列表的查询和管理功能。

## panel 模块

**注册位置**: `frameworks/js/napi/inputmethodpanel/input_method_panel_module.cpp:44`

### 描述

提供输入法面板的控制和管理功能。

## inputMethodEngine 模块

**注册位置**: `frameworks/js/napi/inputmethodability/input_method_engine_module.cpp:51`

### 描述

提供输入法引擎的接口，供输入法应用调用。

## keyboardPanelManager 模块

**注册位置**: `frameworks/js/napi/keyboardpanelmanager/keyboard_panel_manager_module.cpp:43`

### 描述

提供键盘面板的管理功能。

## 错误码定义

| 错误码 | 描述 |
|--------|------|
| `ERROR_OK` | 成功 |
| `ERROR_GENERAL` | 通用错误 |
| `ERROR_PARAMETER` | 参数错误 |
| `ERROR_NOT_ATTACHED` | 未附加 |
| `ERROR_PERMISSION` | 权限错误 |
| `ERROR_SERVICE_NOT_FOUND` | 服务未找到 |

## 相关文档

- [架构说明](./01_Architecture.md)
- [Inner API 接口](./03_Inner_API.md)

# Inner API 接口

## 概述

Inner API 是输入法框架内部使用的 C++ 接口，主要供系统应用和输入法调用。

## 接口清单

### inputmethod_controller

**路径**: `interfaces/inner_api/inputmethod_controller`

**稳定性**: 稳定接口

#### 头文件

| 头文件 | 描述 |
|--------|------|
| [`input_method_controller.h`](./inner_api/inputmethod_controller/include/input_method_controller.h) | 输入法控制器主接口 |
| [`ime_event_listener.h`](./inner_api/inputmethod_controller/include/ime_event_listener.h) | IME 事件监听器 |
| [`ime_event_monitor_manager.h`](./inner_api/inputmethod_controller/include/ime_event_monitor_manager.h) | 事件监控管理器 |
| [`ime_system_channel.h`](./inner_api/inputmethod_controller/include/ime_system_channel.h) | 系统通道 |
| [`visibility.h`](./inner_api/inputmethod_controller/include/visibility.h) | 可见性定义 |

#### 主要类

##### InputMethodController

```cpp
namespace OHOS {
namespace MiscServices {

class InputMethodController : public RefBase {
public:
    // 获取单例
    static sptr<InputMethodController> GetInstance();

    // 附加编辑器
    int32_t Attach(sptr<OnTextChangedListener> listener,
                    bool isShowKeyboard,
                    const TextConfig &textConfig,
                    ClientType type = ClientType::INNER_KIT);

    // 显示软键盘
    int32_t ShowTextInput(ClientType type = ClientType::INNER_KIT);

    // 隐藏软键盘
    int32_t HideTextInput();

    // 关闭
    int32_t Close();

    // 切换输入法
    int32_t SwitchInputMethod(SwitchTrigger trigger,
                              const std::string &name,
                              const std::string &subName = "");

    // 列出输入法
    int32_t ListInputMethod(std::vector<Property> &props);

    // 获取当前输入法
    std::shared_ptr<Property> GetCurrentInputMethod();

    // 停止输入会话
    int32_t StopInputSession();

    // 发送私人命令
    int32_t SendPrivateCommand(
        const std::unordered_map<std::string, PrivateDataValue> &privateCommand,
        bool validateDefaultIme = true);
};

} // namespace MiscServices
} // namespace OHOS
```

##### OnTextChangedListener

```cpp
class OnTextChangedListener : public virtual RefBase {
public:
    virtual void InsertText(const std::u16string &text) = 0;
    virtual void DeleteForward(int32_t length) = 0;
    virtual void DeleteBackward(int32_t length) = 0;
    virtual void SendKeyEventFromInputMethod(const KeyEvent &event) = 0;
    virtual void SendKeyboardStatus(const KeyboardStatus &keyboardStatus) = 0;
    virtual void SendFunctionKey(const FunctionKey &functionKey) = 0;
    virtual void SetKeyboardStatus(bool status) = 0;
    virtual void MoveCursor(const Direction direction) = 0;
    // ... 更多方法
};
```

### inputmethod_ability

**路径**: `interfaces/inner_api/inputmethod_ability`

**稳定性**: 稳定接口

#### 头文件

| 头文件 | 描述 |
|--------|------|
| [`input_method_ability_interface.h`](./inner_api/inputmethod_ability/include/input_method_ability_interface.h) | 输入法能力接口 |
| [`input_method_engine_listener.h`](./inner_api/inputmethod_ability/include/input_method_engine_listener.h) | 输入法引擎监听器 |
| [`input_method_types.h`](./inner_api/inputmethod_ability/include/input_method_types.h) | 输入法类型定义 |
| [`keyboard_listener.h`](./inner_api/inputmethod_ability/include/keyboard_listener.h) | 键盘监听器 |
| [`text_input_client_listener.h`](./inner_api/inputmethod_ability/include/text_input_client_listener.h) | 文本输入客户端监听器 |

#### 主要类

##### InputMethodAbility

```cpp
namespace OHOS {
namespace MiscServices {

class InputMethodAbility {
public:
    static InputMethodAbility &GetInstance();

    // 初始化
    int32_t Init();

    // 启动输入法
    int32_t StartInput(const InputMethodInfo &info, bool isSync = true);

    // 停止输入法
    int32_t StopInput();

    // 显示键盘
    int32_t ShowSoftKeyboard();

    // 隐藏键盘
    int32_t HideSoftKeyboard();

    // 设置键盘监听器
    void SetKeyboardListener(const sptr<KeyboardListener> &listener);

    // 设置引擎监听器
    void SetEngineListener(const sptr<InputMethodEngineListener> &listener);
};

} // namespace MiscServices
} // namespace OHOS
```

### imf_hook

**路径**: `interfaces/inner_api/imf_hook`

**稳定性**: 内部接口（可能变化）

#### 头文件

| 头文件 | 描述 |
|--------|------|
| [`imf_hook.h`](./inner_api/imf_hook/include/imf_hook.h) | IMF Hook 接口 |

## 依赖关系

```
应用层
   │
   ▼
┌─────────────────────┐
│  InputMethodController  │
│      (inner_api)        │
└──────────┬────────────┘
           │
           ▼
┌─────────────────────┐
│  InputMethodAbility  │
│      (inner_api)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       IMSA          │
│    (services)        │
└─────────────────────┘
```

## 稳定性标注

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `inputmethod_controller` | 稳定 | 系统 API，供系统应用使用 |
| `inputmethod_ability` | 稳定 | 系统 API，供输入法应用使用 |
| `imf_hook` | 不稳定 | 内部 Hook 接口，慎用 |

## 相关文档

- [架构说明](./01_Architecture.md)
- [N-API 接口](./02_N-API.md)
- [构建与编译](./04_Build.md)

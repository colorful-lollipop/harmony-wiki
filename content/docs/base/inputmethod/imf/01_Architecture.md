# 架构说明

## 子系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                     EditText / TextField                    │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     输入法框架 (IMF)                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   JS/N-API 层    │  │  Inner API 层   │  │   NDK C API     │  │
│  │  inputMethod     │  │  InputMethod    │  │  native_*        │  │
│  │  InputMethodCtrl │  │  Controller     │  │                 │  │
│  │  inputMethodList │  │  InputMethod    │  │                 │  │
│  └────────┬────────┘  │  Ability        │  │                 │  │
│           │            └────────┬────────┘  └─────────────────┘  │
│           ▼                     ▼                                 │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    Native Framework                          │  │
│  │  ┌──────────────────────┐  ┌──────────────────────────┐   │  │
│  │  │  inputmethod_client   │  │    inputmethod_ability    │   │  │
│  │  │  (应用客户端)          │  │    (输入法客户端)          │   │  │
│  │  └──────────┬───────────┘  └─────────────┬────────────┘   │  │
│  └─────────────┼─────────────────────────────┼─────────────────┘  │
│               │                             │                      │
│               ▼                             ▼                      │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    输入法服务 (IMSA)                          │  │
│  │         InputMethodSystemAbility (SA 3008)                   │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │  │
│  │  │ 状态管理     │ │ 配置管理     │ │ 输入通道管理         │   │  │
│  │  │ ImeStateMgr │ │ ImeCfgMgr   │ │ InputControlChannel │   │  │
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘   │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                     输入法应用 (IME)                          │  │
│  │              第三方输入法实现 (InputMethod Extension)           │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 模块职责

### frameworks/native/inputmethod_controller (应用客户端)

**路径**: `frameworks/native/inputmethod_controller`

**职责**:
- 与输入法服务建立连接
- 接收用户输入事件
- 管理编辑器状态
- 处理光标位置信息

**关键类**:
- `InputMethodController` - 主控制器
- `OnTextChangedListener` - 文本变化监听
- `InputDataChannel` - 输入数据通道

### frameworks/native/inputmethod_ability (输入法客户端)

**路径**: `frameworks/native/inputmethod_ability`

**职责**:
- 与输入法服务交互
- 监听输入法状态变化
- 管理输入法面板
- 转发输入事件

**关键类**:
- `InputMethodAbility` - 能力入口
- `InputMethodPanel` - 面板管理
- `KeyboardListener` - 键盘监听

### services (输入法服务)

**路径**: `services`

**职责**:
- 管理所有输入法生命周期
- 处理输入法切换
- 维护用户输入状态
- 权限校验

**关键类**:
- `InputMethodSystemAbility` - 系统能力入口
- `ImeStateManager` - 状态管理
- `ImeCfgManager` - 配置管理
- `ClientGroup` - 客户端组管理

### frameworks/js/napi (JS/N-API 层)

**路径**: `frameworks/js/napi`

**职责**:
- 暴露 JS/ArkTS API
- 封装 Native 接口
- 处理 JS 与 Native 交互

**关键模块**:
- `inputmethod` - 输入法管理
- `InputMethodController` - 控制器
- `inputMethodList` - 列表管理
- `panel` - 面板控制
- `inputMethodEngine` - 引擎接口
- `keyboardPanelManager` - 面板管理器

## 数据流

### 文本输入流程

```
用户按键 -> 输入法应用 -> InputMethodAbility -> IMSA ->
InputControlChannel -> InputMethodController -> EditText
```

### 键盘显示流程

```
EditText 焦点获取 -> InputMethodController.ShowSoftKeyboard() ->
IMSA.ShowCurrentInput() -> InputMethodAbility.CreatePanel() ->
输入法面板显示
```

## 线程模型

| 组件 | 线程 | 说明 |
|------|------|------|
| JS/N-API | JS 线程 | 处理 JS 调用 |
| InputMethodController | 主线程 | UI 交互 |
| InputMethodAbility | 独立线程 | 输入法事件处理 |
| IMSA | Sa 线程 | SystemAbility 线程 |
| 面板渲染 | UI 线程 | 使用 ArkUI 框架 |

## 依赖关系

```
┌─────────────┐
│  JS/N-API   │  ◄─── 依赖 ───┐
└─────────────┘               │
                              ▼
┌─────────────┐         ┌─────────────┐
│  Inner API  │  ◄─── 依赖 ───┤  NDK C API │
└─────────────┘               └─────────────┘
      │
      ▼
┌─────────────┐         ┌─────────────┐
│   Native    │  ───▶  │    IMSA     │
│  Framework  │  IPC   │  (SA 3008)  │
└─────────────┘         └─────────────┘
```

## 相关文档

- [N-API 接口](./02_N-API.md)
- [Inner API 接口](./03_Inner_API.md)
- [安全评审](./05_Security.md)

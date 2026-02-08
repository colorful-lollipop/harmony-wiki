# 01_Overview - 项目定位与核心能力

## 1.1 项目背景

### 1.1.1 定位

`multimodalinput_input` 是 OpenHarmony 多模态输入子系统的核心仓库，提供：

- **输入事件采集**: 触摸屏、鼠标、键盘、触摸板、手柄等设备
- **事件处理与分发**: 事件归一化、拦截、过滤、分发
- **设备管理**: 输入设备的发现、配置、状态管理
- **跨设备协作**: 意图识别、拖拽、协同输入

### 1.1.2 适用范围

| 场景 | 支持情况 |
|------|---------|
| 标准系统 (Standard) | ✅ 支持 |
| 轻量系统 (Lite) | ❌ 不支持 |
| 小型系统 (Mini) | ❌ 不支持 |

### 1.1.3 依赖关系

```
上层依赖:
  ├── @ohos.multimodalInput (JS API)
  ├── @kit.InputKit (ArkTS API)
  └── 窗口管理系统

平级依赖:
  ├── window_manager - 窗口信息
  ├── display_manager - 显示管理
  ├── app_manager - 应用管理
  └── security - 访问控制

下层依赖:
  ├── libinput - 事件采集
  ├── drivers - 设备驱动
  └── HDF - 硬件抽象层
```

## 1.2 核心能力

### 1.2.1 输入事件处理

| 能力 | 描述 | 优先级 |
|------|------|--------|
| 事件归一化 | libinput 事件 → MMI 统一事件格式 | P0 |
| 事件拦截 | 预拦截事件流 | P1 |
| 事件过滤 | 条件过滤事件 | P1 |
| 事件分发 | 分发到目标应用窗口 | P0 |
| 事件订阅 | 按键/触摸订阅回调 | P1 |

### 1.2.2 设备管理能力

| 能力 | 描述 | 代码位置 |
|------|------|---------|
| 设备发现 | 自动发现可用输入设备 | `service/device_manager/` |
| 设备枚举 | 列出所有输入设备 | `inputDevice.getDeviceList()` |
| 设备配置 | 配置设备参数 | `inputDevice.setFunctionKeyEnabled()` |
| 设备状态 | 监控设备连接状态 | `inputDevice.on('change')` |

### 1.2.3 特殊输入能力

| 能力 | 支持设备 | 备注 |
|------|---------|------|
| 红外发射 | 红外设备 | `infraredEmitter` 模块 |
| 指关节手势 | 触摸屏 | `knuckle` 模块 |
| 表冠控制 | 旋转输入 | `crown_transform_processor` |
| 游戏手柄 | 游戏手柄 | `joystick_event` 模块 |

## 1.3 运行环境

### 1.3.1 硬件要求

| 设备类型 | 最低配置 | 推荐配置 |
|---------|---------|---------|
| CPU | ARMv8-A | ARMv8-A |
| 内存 | 512MB | 1GB |
| 存储 | 512KB (ROM) | 1MB (ROM) |

### 1.3.2 软件依赖

| 依赖项 | 版本要求 | 说明 |
|--------|---------|------|
| OpenHarmony | 4.0+ | 标准系统 |
| libinput | 1.0+ | 输入事件采集 |
| HDF | 1.0+ | 硬件驱动框架 |
| SAMGR | 1.0+ | 系统能力管理 |

### 1.3.3 系统能力声明

```json
// bundle.json 中的 SystemCapability 声明
{
  "syscap": [
    "SystemCapability.MultimodalInput.Input.Core",
    "SystemCapability.MultimodalInput.Input.InfraredEmitter",
    "SystemCapability.MultimodalInput.Input.Cooperator",
    "SystemCapability.MultimodalInput.Input.Pointer",
    "SystemCapability.MultimodalInput.Input.ShortKey",
    "SystemCapability.MultimodalInput.Input.InputMonitor",
    "SystemCapability.MultimodalInput.Input.InputSimulator",
    "SystemCapability.MultimodalInput.Input.InputDevice",
    "SystemCapability.MultimodalInput.Input.InputConsumer"
  ]
}
```

## 1.4 关键概念

### 1.4.1 输入事件类型

```
InputEvent (基类)
├── KeyEvent (按键事件)
│   ├── 按键按下/释放
│   └── 组合键
├── PointerEvent (指针事件)
│   ├── MouseEvent (鼠标事件)
│   ├── TouchEvent (触摸事件)
│   └── StylusEvent (手写笔事件)
└── SwitchEvent (开关事件)
```

### 1.4.2 事件流概念

| 概念 | 描述 |
|------|------|
| 原始事件 | libinput 采集的原始设备事件 |
| 归一化事件 | MMI 内部统一格式事件 |
| 派发事件 | 分发到应用的事件 |
| 注入事件 | 模拟输入事件 |

### 1.4.3 客户端类型

| 类型 | 标识 | 说明 |
|------|------|------|
| HAP 应用 | TOKEN_HAP | 普通应用 |
| 系统服务 | TOKEN_NATIVE | 系统原生服务 |
| Shell 命令 | TOKEN_SHELL | Shell 进程 |

## 1.5 目录职责

### 1.5.1 核心目录

| 目录 | 职责 | 关键文件 |
|------|------|---------|
| `interfaces/native/` | 对外 Native API | `input_manager.h` |
| `frameworks/napi/` | N-API 绑定 | `js_register_module.cpp` |
| `service/` | 服务层实现 | `mmi_service.cpp` |
| `sa_profile/` | SA 配置 | `3101.json` |

### 1.5.2 工具目录

| 目录 | 职责 |
|------|------|
| `tools/inject_event/` | 事件注入工具 |
| `util/common/` | 公共工具函数 |
| `util/napi/` | N-API 工具函数 |

## 1.6 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|---------|
| 3.1 | 2024-02 | 当前版本 |
| 3.0 | 2023-XX | 重构 N-API 架构 |
| 2.0 | 2022-XX | 增加跨设备协作 |

## 1.7 相关资源

- **源码仓库**: https://gitee.com/openharmony/multimodalinput_input
- **Issue 反馈**: https://gitee.com/openharmony/multimodalinput_input/issues
- **API 参考**: https://opendeep.wiki/openharmony/docs/api-reference

# 目录结构

## 顶层结构

```
ide_previewer/
├── cli/              # 命令处理模块
├── gn/               # 构建配置
├── jsapp/            # 渲染引擎调用
├── mock/             # 交互层模拟
├── util/             # 工具模块
├── test/             # 测试目录 (不纳入文档)
├── BUILD.gn          # 根构建文件
├── bundle.json       # 组件配置
└── README.md         # 项目说明
```

## 模块详解

### CLI 模块 (cli/)

**职责**: 命令解析与 IDE 通信

| 文件 | 类型 | 职责 |
|------|------|------|
| `CommandLine.h/cpp` | 基类+实现 | 40+ 命令类定义与执行 |
| `CommandLineInterface.h/cpp` | 单例 | 命名管道通信、消息处理 |
| `CommandLineFactory.h/cpp` | 工厂 | 命令对象创建 |

**关键类**:
- `CommandLineInterface`: 单例，命令循环入口
- `CommandLine`: 命令基类
- `TouchPressCommand`, `TouchMoveCommand`, `TouchReleaseCommand`: 触摸事件
- `KeyPressCommand`: 键盘事件
- `ResolutionCommand`, `OrientationCommand`: 显示参数
- `LoadDocumentCommand`: 文档加载

### JSAPP 模块 (jsapp/)

**职责**: 渲染引擎调用与生命周期管理

| 子目录 | 类型 | 目标 |
|--------|------|------|
| `JsApp.h/cpp` | 抽象基类 | 生命周期接口 |
| `rich/JsAppImpl.h/cpp` | Rich 实现 | ArkUI Ace Engine |
| `lite/JsAppImpl.h/cpp` | Lite 实现 | ACELite Engine |
| `rich/external/` | 内部 Kit | 对外接口 |

**关键类**:
- `JsApp`: 抽象基类 (Start/Restart/Interrupt/Stop)
- `JsAppImpl` (rich/lite): 具体实现
- `StageContext`: HSP 模块管理
- `EventRunner`: 事件循环
- `EventHandler`: 事件处理

### MOCK 模块 (mock/)

**职责**: 设备交互层模拟

| 子目录 | 目标 |
|--------|------|
| `VirtualScreen.h/cpp` | 虚拟屏幕管理 |
| `VirtualMessage.h/cpp` | 消息处理 |
| `MouseInput.h/cpp` | 鼠标/触摸 |
| `KeyInput.h/cpp` | 键盘输入 |
| `MouseWheel.h/cpp` | 滚轮 |
| `LanguageManager.h/cpp` | 语言管理 |
| `SystemCapability.h/cpp` | 能力检查 |
| `rich/` | Rich 实现 |
| `lite/` | Lite 实现 |

**Lite 特有**:
- `BatteryModuleImpl`: 电池模拟
- `BrightnessModuleImpl`: 亮度模拟
- `SensorModuleImpl`: 传感器模拟
- `VibratorModuleImpl`: 振动模拟
- `GeoLocation`: 位置模拟
- `AblityKit`: Ability 生命周期

### UTIL 模块 (util/)

**职责**: 跨平台工具与基础设施

| 文件 | 职责 |
|------|------|
| `CommandParser.h/cpp` | 命令行参数解析 |
| `WebSocketServer.h/cpp` | WebSocket 服务 |
| `LocalSocket.h/cpp` | 命名管道 |
| `FileSystem.h/cpp` | 文件系统 |
| `JsonReader.h/cpp` | JSON 解析 |
| `SharedData.h/cpp` | 共享数据 |
| `CppTimer.h/cpp` | 定时器 |
| `CrashHandler.h/cpp` | 崩溃处理 |
| `TraceTool.h/cpp` | 追踪工具 |
| `KeyboardHelper.h/cpp` | 键盘辅助 |
| `ClipboardHelper.h/cpp` | 剪贴板 |
| `ModelManager.h/cpp` | 设备模型 |

**平台特定**:
- `windows/`: Windows API 封装
- `unix/`: macOS Unix 接口
- `linux/`: Linux 特有实现

### GN 模块 (gn/)

**职责**: 构建配置

| 文件 | 职责 |
|------|------|
| `config.gni` | 平台配置 |

---

## 模块依赖关系

```
RichPreviewer/ThinPreviewer (入口)
    │
    ├── cli/ (命令处理)
    │   └── util/ (基础设施)
    │       └── mock/ (可选)
    │
    ├── jsapp/ (渲染调用)
    │   ├── mock/ (交互模拟)
    │   │   └── util/ (工具)
    │   └── rich/external/ (内部 Kit)
    │
    └── mock/ (交互模拟)
        └── util/ (工具)
```

---

## 相关文档

- 概览: [00_Overview.md](./00_Overview.md)
- 架构: [02_Architecture.md](./02_Architecture.md)
- 内部 API: [04_Inner_API.md](./04_Inner_API.md)

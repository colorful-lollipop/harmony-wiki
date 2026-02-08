# 项目概览

## 项目定位

**ArkXtest** 是 OpenHarmony 的自动化测试框架，为应用和系统开发者提供统一的测试能力支撑。框架采用模块化设计，支持多语言绑定（ArkTS-Dynamic、ArkTS-Static、Cangjie），能够满足单元测试、UI 自动化测试、性能测试等多种测试场景。

ArkXtest 不是一个被测应用，而是一个测试工具框架。它本身不包含具体的业务测试用例，而是提供编写、运行测试用例的能力，以及与系统交互的高权限接口。

## 核心能力

ArkXtest 由三个核心测试框架组成：

### 1. JsUnit (Hypium)

单元测试框架，提供以下能力：

| 能力 | 说明 |
|------|------|
| **基础流程** | `describe`、`it`、`beforeAll`、`beforeEach`、`afterEach`、`afterAll` |
| **断言库** | 25+ 断言方法（`assertEqual`、`assertContain`、`assertTrue` 等） |
| **Mock 能力** | 函数级 Mock，支持 `when()`/`afterReturn()`/`afterThrow()` |
| **数据驱动** | `dataProvider` 复用测试脚本，使用不同输入数据驱动执行 |
| **异步测试** | Promise-based 异步测试支持 |
| **专项能力** | 筛选、跳过、超时、随机执行、压力测试 |

**证据**: `README_zh.md:24-33` 描述了 JsUnit 的 7 项核心功能特性

### 2. UiTest

UI 自动化测试框架，通过简洁易用的 API 提供查找和操作界面控件的能力：

| 能力 | 说明 |
|------|------|
| **控件定位** | `By`/`On` 选择器，支持 ID、文本、类型、状态等多种匹配条件 |
| **控件操作** | `click`、`inputText`、`scrollSearch`、`drag` 等 |
| **窗口管理** | `UiWindow` 支持窗口获取、焦点、调整大小、关闭等 |
| **输入注入** | 触摸、键盘、鼠标、手写笔事件注入 |
| **事件监听** | `UIEventObserver` 监听组件事件、窗口变化等 |
| **录制回放** | UI 操作录制与回放功能 |

**证据**: `README_zh.md:8` 描述 UiTest 提供查找和操作界面控件能力

### 3. PerfTest

白盒性能测试框架，支持采集指定代码段的性能数据：

| 指标类型 | 说明 |
|----------|------|
| **DURATION** | 代码段执行时间 |
| **CPU_LOAD** | 系统 CPU 负载 |
| **CPU_USAGE** | 进程 CPU 使用率 |
| **MEMORY_RSS** | 进程常驻集大小 |
| **MEMORY_PSS** | 进程比例集大小 |
| **APP_START_RESPONSE_TIME** | 应用启动响应延迟 |
| **APP_START_COMPLETE_TIME** | 应用启动完成延迟 |
| **PAGE_SWITCH_COMPLETE_TIME** | 页面切换完成延迟 |
| **LIST_SWIPE_FPS** | 列表滑动帧率 |

**证据**: `perftest/collection/include/data_collection.h:29-40` 定义了 `PerfMetric` 枚举

## 运行环境

### 系统要求

- **系统类型**: 标准系统（standard）
- **API 版本**: API version 8+
- **SDK 版本**: OpenHarmony SDK

### 硬件要求

| 资源 | 占用 |
|------|------|
| ROM | ~500KB |
| RAM | ~100KB |

**证据**: `bundle.json:25-26` 定义了资源占用

### 权限要求

| 组件 | 需要的系统能力 |
|------|---------------|
| UiTest | `SystemCapability.Test.UiTest` |
| PerfTest | `SystemCapability.Test.PerfTest` |

**证据**: `bundle.json:17-20` 定义了系统能力依赖

## 关键概念

### 客户端-服务器模型

UiTest 和 PerfTest 采用客户端-服务器架构：

```
┌─────────────────┐          IPC           ┌─────────────────┐
│  测试应用        │ ───────────────────> │  服务端守护进程   │
│  (ArkTS/JS)     │                      │  (uitest/perftest)│
└─────────────────┘                      └─────────────────┘
        │                                        │
        │ N-API/ANI                             │ 访问系统
        ↓                                        ↓
┌─────────────────┐                      ┌─────────────────┐
│ 客户端库         │                      │  Accessibility  │
│ libuitest.z.so  │                      │ WindowManager   │
│ libperftest.z.so│                      │ InputSystem     │
└─────────────────┘                      └─────────────────┘
```

**证据**: `uitest/BUILD.gn:178-246` 定义了 `uitest_server` 可执行文件和客户端库

### TestServer SA

TestServer 是一个系统能力服务（SA ID: 5502），为 UiTest 和 PerfTest 提供高权限系统能力调用：

| 功能 | 说明 |
|------|------|
| 窗口管理 | 窗口模式切换、终止、最小化 |
| 剪贴板 | 设置剪贴板内容 |
| 系统事件 | 发布系统公共事件 |
| CPU 频率 | CPU 频率锁定 |
| 性能采集 | 进程 CPU/内存采集 |
| SmartPerf | 进程控制 |

**证据**: `testserver/sa_profile/5502.json:5` 定义了 SA ID 5502

### 多语言支持

ArkXtest 支持三种前端语言：

| 语言 | 绑定方式 | 产物 |
|------|----------|------|
| ArkTS-Dynamic | N-API | `*.z.so` + `*.abc` |
| ArkTS-Static | ANI | `*.ani.so` + `*.abc` |
| Cangjie | FFI | `*.z.so` |

**证据**: `uitest/BUILD.gn:431-439` 定义了 `uitestkit` 组装目标

## 目录结构

```
arkxtest/
├── jsunit/              # 单元测试框架
│   ├── src/           # ArkTS-Dynamic 实现
│   ├── src_static/    # ArkTS-Static 实现
│   ├── package.json   # NPM 包配置
│   └── index.js       # 入口
│
├── uitest/             # UI 自动化测试框架
│   ├── core/          # C++ 核心逻辑
│   ├── server/        # 服务端守护进程
│   ├── connection/    # IPC 通信层
│   ├── input/         # 输入注入
│   ├── record/        # 录制回放
│   ├── addon/        # 扩展（截图等）
│   ├── ets/ani/      # ArkTS-Static 绑定
│   ├── napi/         # ArkTS-Dynamic 绑定
│   ├── cj/           # Cangjie FFI 绑定
│   └── test/         # C++ 单元测试
│
├── perftest/          # 性能测试框架
│   ├── core/         # 核心逻辑
│   ├── collection/   # 数据采集
│   ├── connection/  # IPC 通信
│   ├── napi/        # N-API 绑定
│   └── ani/         # ANI 绑定
│
├── testserver/        # 测试服务器 SA
│   ├── src/         # 服务实现
│   ├── init/        # 初始化配置
│   └── sa_profile/  # SA 配置
│
├── hamock/           # Mock 框架
└── coverage_signal_handler/  # 覆盖率信号处理
```

**证据**: `README_zh.md:14-19` 描述了主要目录结构

## 约束与限制

1. **API 版本限制**: 首批接口从 API version 8 开始支持，新增接口会标注起始版本
2. **系统类型**: 仅支持标准系统（standard）
3. **Mock 限制**: 仅支持 Mock 自定义对象，不支持 Mock 系统 API 对象
4. **测试框架**: 本框架是测试工具，不是被测对象

**证据**: `README_zh.md:20-21` 描述了约束限制

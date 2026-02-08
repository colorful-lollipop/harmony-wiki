# 目录结构与模块职责

## 目的

本文档详细描述项目的目录结构、各模块的职责和文件组织方式。

## 适用范围

- 了解项目代码组织的开发者
- 需要定位特定功能的维护者
- 学习 OpenHarmony 应用结构的开发者

## 关键结论

1. **目录结构简洁**: 项目采用扁平化组织，所有代码集中在 `app/` 目录
2. **模块独立**: demolink、iothardware、samgr 三个主要模块职责清晰
3. **纯 C 代码**: 无 JS/TS 文件，全部为 C 语言实现
4. **测试代码包含**: SAMGR 示例中引用了 ACTS 测试代码

## 顶层目录结构

```
applications/sample/wifi-iot/         # 项目根目录
├── README.md                         # 项目说明文档（英文）
├── README_zh.md                      # 项目说明文档（中文）
├── LICENSE                           # Apache 2.0 许可证
├── bundle.json                       # 组件描述与依赖配置
├── app/                              # 应用源代码根目录
│   ├── BUILD.gn                      # 主构建脚本
│   ├── bundle.json                   # 应用级配置
│   ├── startup/                      # 启动模块（占位）
│   │   └── BUILD.gn
│   ├── demolink/                     # Demo SDK 集成示例
│   │   ├── BUILD.gn
│   │   ├── demosdk.c                 # SDK 实现入口
│   │   ├── demosdk_adapter.c         # SDK 适配层
│   │   ├── demosdk_adapter.h         # SDK 适配层头文件
│   │   ├── demosdk.h                 # SDK 头文件
│   │   └── helloworld.c             # 启动入口
│   ├── iothardware/                  # IoT 硬件操作示例
│   │   ├── BUILD.gn
│   │   └── led_example.c            # LED 控制示例
│   └── samgr/                        # SAMGR_Lite 服务框架示例
│       ├── BUILD.gn
│       ├── example.h                  # 公共定义
│       ├── service_example.c          # 服务示例（188 行）
│       ├── feature_example.c          # 特性示例（320 行）
│       ├── broadcast_example.c       # 广播示例（241 行）
│       ├── bootstrap_example.c        # 启动顺序示例（241 行）
│       ├── maintenance_example.c     # 维护接口示例（179 行）
│       ├── service_recovery_example.c # 服务恢复示例（119 行）
│       ├── specified_task_example.c   # 指定任务示例（227 行）
│       └── task_example.c            # 任务示例（123 行）
└── wiki/                             # 本 Wiki 文档目录
    ├── README.md
    ├── SUMMARY.md
    ├── 01_Project_Overview.md
    ├── 02_Directory_Structure.md
    ├── ...
    └── _work/                        # 工作笔记和计划
        ├── NOTES.md
        └── PLAN.md
```

## 模块职责清单

| 目录 | 职责 | 主要文件 | 入口 | 代码行数 |
|------|------|---------|------|---------|
| `app/startup/` | 启动模块（占位，无实际代码） | BUILD.gn | 无 | 0 |
| `app/demolink/` | Demo SDK 集成示例，展示如何创建和运行自定义 SDK | demosdk.c, helloworld.c | DemoSdkEntry | ~128 行 |
| `app/iothardware/` | IoT 硬件操作示例（GPIO LED 控制） | led_example.c | LedExampleEntry | ~82 行 |
| `app/samgr/` | SAMGR_Lite 服务框架示例，展示服务注册、特性调用、广播等 | *.c | Init 函数 | ~1765 行 |

## 模块详细说明

### 1. startup 模块

**职责**:
- 应用启动入口点
- 目前为空实现（占位模块）

**文件说明**:
- `BUILD.gn`: 定义 `startup` source_set（空）

证据：
- `app/startup/BUILD.gn:14-18` 定义了空的 source_set

### 2. demolink 模块

**职责**:
- 展示如何集成自定义 SDK
- 演示任务创建与管理
- 展示线程优先级与栈大小配置

**文件说明**:

| 文件 | 说明 | 关键函数/宏 |
|------|------|------------|
| `demosdk.c` | SDK 核心实现 | DemoSdkEntry, DemoSdkBiz |
| `demosdk_adapter.c` | SDK 适配层（封装线程操作） | DemoSdkCreateTask, DemoSdkSleepMs |
| `demosdk_adapter.h` | 适配层头文件 | TaskPara 结构定义 |
| `demosdk.h` | SDK 头文件 | DemoSdkEntry 声明 |
| `helloworld.c` | 启动入口 | DemoSdkMain |

证据：
- `app/demolink/demosdk.c:33-48` DemoSdkEntry 函数实现
- `app/demolink/helloworld.c:19-24` DemoSdkMain 函数
- `app/demolink/helloworld.c:24` 使用 `SYS_RUN` 宏注册启动函数

### 3. iothardware 模块

**职责**:
- 展示 GPIO 硬件操作
- LED 闪烁控制
- 线程任务创建

**文件说明**:

| 文件 | 说明 | 关键函数/宏 |
|------|------|------------|
| `led_example.c` | LED 控制示例 | LedExampleEntry, LedTask |

证据：
- `app/iothardware/led_example.c:61-79` LedExampleEntry 函数
- `app/iothardware/led_example.c:35-59` LedTask 任务函数
- `app/iothardware/led_example.c:81` 使用 `SYS_RUN` 宏注册

**GPIO 操作流程**:
1. 初始化 GPIO: `IoTGpioInit(LED_TEST_GPIO)`
2. 设置方向: `IoTGpioSetDir(LED_TEST_GPIO, IOT_GPIO_DIR_OUT)`
3. 设置输出值: `IoTGpioSetOutputVal(LED_TEST_GPIO, 1/0)`

### 4. samgr 模块

**职责**:
- 展示 SAMGR_Lite 服务框架的完整用法
- 服务（Service）注册与发现
- 特性（Feature）注册与调用
- 广播（Broadcast）发布-订阅机制
- 服务启动顺序控制
- 服务恢复机制
- 任务配置管理

**文件说明**:

| 文件 | 代码行数 | 说明 | 关键内容 |
|------|---------|------|---------|
| `example.h` | 37 行 | 公共定义 | 服务/特性名称宏定义 |
| `service_example.c` | 188 行 | 服务示例 | 服务注册、默认特性 API |
| `feature_example.c` | 320 行 | 特性示例 | 特性注册、同步/异步调用 |
| `broadcast_example.c` | 241 行 | 广播示例 | 发布-订阅、主题管理 |
| `bootstrap_example.c` | 241 行 | 启动顺序 | SYSEX_SERVICE_INIT 分层启动 |
| `maintenance_example.c` | 179 行 | 维护接口 | SAMGR_PrintServices 打印服务信息 |
| `service_recovery_example.c` | 119 行 | 服务恢复 | 服务异常恢复机制 |
| `specified_task_example.c` | 227 行 | 指定任务 | 自定义任务配置 |
| `task_example.c` | 123 行 | 任务示例 | 基础任务配置 |
| `samgr_maintenance.c` | - | 外部引用 | 来自 test/xts/acts |

证据：
- `app/samgr/BUILD.gn:15-25` 列出所有源文件
- `app/samgr/example.h:18-36` 定义服务/特性名称宏

**SAMGR 核心概念示例映射**:

| 概念 | 文件位置 | 实现说明 |
|------|---------|---------|
| Service 注册 | service_example.c:92 | `SAMGR_GetInstance()->RegisterService()` |
| Feature 注册 | feature_example.c:188 | `SAMGR_GetInstance()->RegisterFeature()` |
| Default Feature API | service_example.c:93 | `RegisterDefaultFeatureApi()` |
| Feature API | feature_example.c:189 | `RegisterFeatureApi()` |
| Broadcast | broadcast_example.c:88 | 注册广播服务，使用发布-订阅 |

## 模块依赖关系

### 内部依赖

```
┌─────────────────────────────────────────────────────────────┐
│                    app/ (应用层)                           │
│                                                             │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐ │
│  │startup   │     │  demolink    │     │ iothardware  │ │
│  │(空占位)  │     │ (Demo SDK)   │     │  (GPIO)      │ │
│  └──────────┘     └──────────────┘     └──────────────┘ │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │               samgr (服务框架示例)                    │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │  │
│  │  │Service  │ │ Feature │ │Broadcast│ │Bootstrap│  │  │
│  │  │Example  │ │ Example │ │ Example │ │ Example │  │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 外部依赖

根据 `bundle.json:23-30` 和各 `BUILD.gn` 的 `deps` 字段：

| 模块 | 依赖组件 | 证据位置 |
|------|---------|---------|
| demolink | utils_lite | app/demolink/BUILD.gn:21 |
| iothardware | utils_lite, liteos_m, peripheral | app/iothardware/BUILD.gn:17-20 |
| samgr | utils_lite, liteos_m, samgr_lite | app/samgr/BUILD.gn:27-34 |

详细依赖：

#### demolink 模块
- `//commonlibrary/utils_lite/include` (utils_lite)

#### iothardware 模块
- `//commonlibrary/utils_lite/include` (utils_lite)
- `//kernel/liteos_m/kal/cmsis` (liteos_m CMSIS 接口)
- `//base/iothardware/peripheral/interfaces/inner_api` (peripheral)

#### samgr 模块
- `//commonlibrary/utils_lite/include` (utils_lite)
- `//kernel/liteos_m/components/cmsis` (liteos_m CMSIS)
- `//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr` (samgr_lite)
- `//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast` (broadcast)
- `//foundation/systemabilitymgr/samgr_lite/samgr/adapter` (samgr_adapter)
- `//foundation/systemabilitymgr/samgr_lite/samgr/source` (samgr source)
- `//test/xts/acts/distributed_schedule_lite/samgr_hal/utils` (ACTS 测试工具)

## 代码组织规范

### 文件命名规范

- 源文件：`*_example.c` - 示例代码
- 头文件：`*.h` - 公共定义
- 构建文件：`BUILD.gn` - GN 构建脚本

### 初始化宏使用

项目使用 OpenHarmony 启动宏注册自动初始化函数：

| 宏名 | 用途 | 使用位置 |
|------|------|---------|
| `SYS_RUN()` | 在系统启动时运行函数 | demolink/helloworld.c:24, iothardware/led_example.c:81 |
| `SYSEX_SERVICE_INIT()` | 系统扩展服务初始化 | samgr/service_example.c:98, samgr/broadcast_example.c:93 |
| `SYSEX_FEATURE_INIT()` | 系统扩展特性初始化 | samgr/feature_example.c:194 |
| `LAYER_INITCALL_DEF()` | 分层初始化调用 | samgr/service_example.c:187, samgr/feature_example.c:320 |

### 启动顺序

根据初始化宏和 SAMGR 文档，启动顺序一般为：

1. `SYS_RUN` - 基础应用初始化
2. `SYSEX_SERVICE_INIT` - 服务注册
3. `SYSEX_FEATURE_INIT` - 特性注册
4. `LAYER_INITCALL_DEF` - 测试用例执行

## 无测试目录

根据项目约定，项目不包含测试目录。所有测试/示例代码均包含在主源文件中（如 `*_example.c`）。

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [对外 API 文档](04_N-API_External.md)
- [内部 API](05_Inner_API.md)

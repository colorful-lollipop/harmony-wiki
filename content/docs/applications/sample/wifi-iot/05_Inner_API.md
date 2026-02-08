# 内部 API

## 目的

本文档描述项目内部模块间的接口定义、依赖方向和稳定性。

## 适用范围

- 需要了解模块间交互的开发者
- 进行重构或模块修改的工程师
- 维护内部接口兼容性的开发者

## 关键结论

1. **模块边界清晰**: demolink、iothardware、samgr 三个模块相对独立
2. **SAMGR_Lite 依赖**: samgr 模块依赖 SAMGR_Lite 框架
3. **无循环依赖**: 模块间依赖为单向，无循环依赖
4. **内部 API 不稳定**: 所有内部 API 可能在未来版本中变化（这是示例代码）

## 模块接口定义

### 1. demolink 模块内部接口

#### 1.1 头文件导出

| 头文件 | 说明 | 导出的符号 |
|-------|------|-----------|
| `demosdk.h` | SDK 公共接口 | `DemoSdkEntry()` |
| `demosdk_adapter.h` | 适配层接口 | `TaskPara`, `DemoSdkCreateTask()`, `DemoSdkSleepMs()` |

证据：
- `app/demolink/demosdk.h`: 公共头文件
- `app/demolink/demosdk_adapter.h`: 适配层头文件

#### 1.2 内部数据结构

```c
// 任务参数结构
struct TaskPara {
    const char *name;   // 任务名称
    void *func;         // 任务函数指针
    uint32 prio;        // 任务优先级
    uint32 size;        // 任务栈大小
};
```

证据：
- `app/demolink/demosdk_adapter.h` 中定义（具体实现未直接展示，但通过 demosdk.c:36-40 使用推断）

#### 1.3 内部函数

| 函数名 | 位置 | 说明 | 稳定性 |
|-------|------|------|--------|
| DemoSdkBiz | demosdk.c:25-31 | SDK 业务逻辑任务 | 内部 |
| DemoSdkEntry | demosdk.c:33-48 | SDK 入口函数 | 内部 |
| DemoSdkCreateTask | demosdk_adapter.c | 创建任务（封装 osThreadNew） | 内部 |
| DemoSdkSleepMs | demosdk_adapter.c | 睡眠函数（封装 LOS_Msleep） | 内部 |

### 2. iothardware 模块内部接口

#### 2.1 头文件导出

该模块无独立的公共头文件，所有函数均为内部函数，通过 `SYS_RUN` 宏自动调用。

#### 2.2 内部数据结构

```c
// LED 状态枚举
enum LedState {
    LED_ON = 0,     // 开灯
    LED_OFF,        // 关灯
    LED_SPARK,      // 闪烁
};
```

证据：
- `app/iothardware/led_example.c:27-31` 定义

#### 2.3 内部函数

| 函数名 | 位置 | 说明 | 稳定性 |
|-------|------|------|--------|
| LedTask | led_example.c:35-59 | LED 控制任务 | 内部 |
| LedExampleEntry | led_example.c:61-79 | LED 示例入口（自动调用） | 内部 |

### 3. samgr 模块内部接口

#### 3.1 头文件导出

| 头文件 | 说明 | 导出的符号 |
|-------|------|-----------|
| `example.h` | 公共定义 | 服务/特性名称宏 |

证据：
- `app/samgr/example.h`: 公共头文件

#### 3.2 内部数据结构

```c
// 默认特性 API
typedef struct DefaultFeatureApi {
    INHERIT_IUNKNOWN;
    void (*SyncCall)(IUnknown *iUnknown);
} DefaultFeatureApi;

// 示例服务
typedef struct ExampleService {
    INHERIT_SERVICE;
    INHERIT_IUNKNOWNENTRY(DefaultFeatureApi);
    Identity identity;
} ExampleService;

// Demo API
typedef struct DemoApi {
    INHERIT_IUNKNOWN;
    BOOL (*AsyncCall)(IUnknown *iUnknown, const char *buff);
    BOOL (*AsyncTimeCall)(IUnknown *iUnknown);
    BOOL (*SyncCall)(IUnknown *iUnknown, struct Payload *payload);
    BOOL (*AsyncCallBack)(IUnknown *iUnknown, const char *buff, Handler handler);
} DemoApi;

// Demo 特性
typedef struct DemoFeature {
    INHERIT_FEATURE;
    INHERIT_IUNKNOWNENTRY(DemoApi);
    Identity identity;
} DemoFeature;
```

证据：
- `app/samgr/service_example.c:28-37` 定义
- `app/samgr/feature_example.c:42-54` 定义

#### 3.3 内部函数

| 函数名 | 位置 | 说明 | 稳定性 |
|-------|------|------|--------|
| GetName | service_example.c:39-43 | 获取服务名称 | 内部 |
| Initialize | service_example.c:47-54 | 服务初始化 | 内部 |
| MessageHandle | service_example.c:56-61 | 消息处理 | 内部 |
| GetTaskConfig | service_example.c:63-69 | 获取任务配置 | 内部 |
| SyncCall | service_example.c:73-78 | 同步调用 | 内部 |
| Init | service_example.c:90-96 | 注册初始化 | 内部 |
| CASE_GetIUnknown | service_example.c:102-125 | 获取 IUnknown | 内部 |
| CASE_SyncCall | service_example.c:127-134 | 同步调用测试 | 内部 |
| CASE_ReleaseIUnknown | service_example.c:136-151 | 释放 IUnknown | 内部 |
| CASE_RegisterInvalidService | service_example.c:153-177 | 注册无效服务测试 | 内部 |
| RunTestCase | service_example.c:179-185 | 运行测试用例 | 内部 |
| FEATURE_GetName | feature_example.c:79-83 | 获取特性名称 | 内部 |
| FEATURE_OnInitialize | feature_example.c:85-92 | 特性初始化 | 内部 |
| FEATURE_OnStop | feature_example.c:94-101 | 特性停止 | 内部 |
| FEATURE_OnMessage | feature_example.c:105-128 | 特性消息处理 | 内部 |
| AsyncCall | feature_example.c:142-157 | 异步调用 | 内部 |
| AsyncTimeCall | feature_example.c:159-166 | 定时异步调用 | 内部 |
| SyncCall | feature_example.c:130-140 | 同步调用 | 内部 |
| AsyncCallBack | feature_example.c:168-184 | 异步回调 | 内部 |
| Init | feature_example.c:186-192 | 特性注册初始化 | 内部 |
| CASE_GetIUnknown | feature_example.c:198-221 | 获取 IUnknown | 内部 |
| CASE_SyncCall | feature_example.c:223-244 | 同步调用测试 | 内部 |
| CASE_AsyncCall | feature_example.c:247-264 | 异步调用测试 | 内部 |
| CASE_AsyncTimeCall | feature_example.c:266-272 | 定时异步调用测试 | 内部 |
| AsyncHandler | feature_example.c:274-280 | 异步处理器 | 内部 |
| CASE_AsyncCallBack | feature_example.c:282-291 | 异步回调测试 | 内部 |
| CASE_ReleaseIUnknown | feature_example.c:293-308 | 释放 IUnknown | 内部 |
| RunTestCase | feature_example.c:310-318 | 运行测试用例 | 内部 |

## 模块依赖方向

### 依赖关系图

```
┌───────────────────────────────────────────────────────────────┐
│                    外部组件（依赖）                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │utils_lite│  │liteos_m  │  │peripheral│  │samgr_lite││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘│
└───────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌───────────────────────────────────────────────────────────────┐
│                  内部模块（依赖方）                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │ demolink │  │iothardware│   │  samgr   │  │ startup  ││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘│
└───────────────────────────────────────────────────────────────┘
```

### 详细依赖表

| 源模块 | 目标模块 | 依赖类型 | 证据位置 |
|-------|---------|---------|---------|
| demolink | utils_lite | include_dirs | app/demolink/BUILD.gn:21 |
| iothardware | utils_lite | include_dirs | app/iothardware/BUILD.gn:18 |
| iothardware | liteos_m | include_dirs | app/iothardware/BUILD.gn:19 |
| iothardware | peripheral | include_dirs | app/iothardware/BUILD.gn:20 |
| samgr | utils_lite | include_dirs | app/samgr/BUILD.gn:28 |
| samgr | liteos_m | include_dirs | app/samgr/BUILD.gn:29 |
| samgr | samgr_lite | include_dirs | app/samgr/BUILD.gn:30-34 |

### 依赖验证（无循环依赖）

检查所有模块的 BUILD.gn，未发现模块间相互依赖：

- `app/demolink/BUILD.gn`: 无 deps
- `app/iothardware/BUILD.gn`: 无 deps
- `app/samgr/BUILD.gn`: 无 deps（只有 include_dirs）
- `app/startup/BUILD.gn`: 空

**结论**: 模块间无循环依赖。

证据：
- `app/demolink/BUILD.gn`: 完整内容无 deps 字段
- `app/iothardware/BUILD.gn`: 完整内容无 deps 字段
- `app/samgr/BUILD.gn`: 完整内容无 deps 字段
- `app/startup/BUILD.gn`: 完整内容为空

## 接口稳定性

### 稳定性标注规则

| 级别 | 说明 | 建议使用 | 代码位置证据 |
|------|------|---------|-------------|
| **Stable** | 公共 API，承诺向后兼容 | 生产代码 | 无（这是示例代码） |
| **Unstable** | 内部 API，可能变化 | 仅示例/学习 | 所有内部函数 |

### SAMGR_Lite 接口稳定性

SAMGR_Lite 框架本身提供的接口相对稳定，但本项目中的示例实现为内部 API。

**SAMGR_Lite 框架接口（外部）**:
- `SAMGR_GetInstance()` - 稳定
- `RegisterService()` - 稳定
- `RegisterFeature()` - 稳定
- `GetFeatureApi()` - 稳定
- `SendRequest()` - 稳定

**项目示例接口（内部）**:
- 所有 `CASE_*` 函数 - 不稳定（测试代码）
- 示例服务/特性实现 - 不稳定（学习参考）

证据：
- `app/samgr/service_example.c:187` `LAYER_INITCALL_DEF(RunTestCase, test, "test")` 表明是测试代码

### Demo SDK 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `DemoSdkEntry()` | 不稳定 | 示例代码，仅供参考 |
| `DemoSdkCreateTask()` | 不稳定 | 内部封装，可能变化 |
| `DemoSdkSleepMs()` | 不稳定 | 内部封装，可能变化 |

证据：
- `app/demolink/demosdk.c`: 命名为 "demo"，表明是示例代码

### IoT GPIO 接口稳定性

HAL 层接口由系统提供，相对稳定。本项目中的示例实现为内部 API。

**HAL 层接口（外部）**:
- `IoTGpioInit()` - 稳定
- `IoTGpioSetDir()` - 稳定
- `IoTGpioSetOutputVal()` - 稳定

**项目示例接口（内部）**:
- `LedTask()` - 不稳定
- `LedExampleEntry()` - 不稳定

证据：
- `app/iothardware/led_example.c`: 文件名包含 "example"，表明是示例代码

## 可替换点

### 1. Demo SDK 适配层

`demosdk_adapter.c/h` 提供了线程创建和睡眠的封装，可替换为其他实现。

**替换点**:
- `DemoSdkCreateTask()` - 可替换为其他任务创建 API
- `DemoSdkSleepMs()` - 可替换为其他睡眠 API

证据：
- `app/demolink/demosdk_adapter.c`: 适配层文件

### 2. SAMGR 示例实现

所有 SAMGR 示例（service_example.c, feature_example.c 等）都可替换或扩展。

**替换点**:
- 服务定义（ExampleService）
- 特性定义（DemoFeature）
- API 定义（DemoApi, DefaultFeatureApi）

证据：
- `app/samgr/`: 所有文件都是示例代码

### 3. GPIO 示例实现

LED 示例可替换为其他硬件控制示例。

**替换点**:
- GPIO 引脚号（LED_TEST_GPIO = 9）
- 控制逻辑（闪烁模式）

证据：
- `app/iothardware/led_example.c:25` `#define LED_TEST_GPIO 9`

## 内部 API 使用建议

### 开发新功能

1. **参考现有模式**: 参考 demolink 或 samgr 模块
2. **不依赖内部 API**: 内部 API 可能变化
3. **使用 SAMGR_Lite 接口**: SAMGR_Lite 框架接口相对稳定

### 修改现有模块

1. **保持接口兼容**: 如果修改接口，考虑向后兼容
2. **更新相关调用**: 检查所有调用点
3. **更新文档**: 同步更新相关文档

### 测试

1. **运行现有测试**: `LAYER_INITCALL_DEF` 定义的测试用例
2. **添加新测试**: 按现有模式添加

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [目录结构与模块职责](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [对外 API 文档](04_N-API_External.md)
- [GN 构建系统](06_GN_Build.md)

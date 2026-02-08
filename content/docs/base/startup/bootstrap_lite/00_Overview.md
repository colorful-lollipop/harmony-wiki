# Bootstrap_Lite - 项目概览

## 项目定位

### 组件定位
**Bootstrap_Lite** 是 OpenHarmony 轻量系统的**启动引导组件**，负责在系统启动时按照预定义顺序初始化各系统服务和功能模块。

**证据**: `path:README.md`

> bootstrap启动引导组件，提供了各服务和功能的启动入口标识。在SAMGR启动时，会调用bootstrap标识的入口函数，并启动系统服务。

### 核心职责
1. **系统初始化入口** - 提供 `OHOS_SystemInit()` 作为系统级初始化入口
2. **服务注册** - 向 SAMGR 注册 Bootstrap 服务
3. **启动顺序管理** - 通过链接器脚本段机制组织初始化函数调用顺序
4. **阶段化启动** - 支持 BSP → Device → Core → Service → Feature → Run 的阶段化启动

### 适用场景
- **目标设备**: 轻量系统设备（参考内存≥128KB）
- **典型硬件**: Hi3861V100 开发板
- **适用系统**: Mini System, Small System

## 核心能力

### 1. 链接器脚本段组织
利用 GCC/Clang 的链接器脚本段特性，将不同阶段的初始化函数组织在独立的段中：

**证据**: `path:services/source/bootstrap_service.h:44-66`

```c
#define APP_BEGIN(name, step)                                 \
    ({  extern InitCall __zinitcall_app_##name##_start;       \
        InitCall *initCall = &__zinitcall_app_##name##_start; \
        (initCall);                                           \
    })
```

### 2. SAMGR 服务集成
作为 SAMGR 的系统服务运行，处理系统启动相关消息：

**证据**: `path:services/source/bootstrap_service.c:30-40`

```c
static void Init(void)
{
    static Bootstrap bootstrap;
    bootstrap.GetName = GetName;
    bootstrap.Initialize = Initialize;
    bootstrap.MessageHandle = MessageHandle;
    bootstrap.GetTaskConfig = GetTaskConfig;
    bootstrap.flag = FALSE;
    SAMGR_GetInstance()->RegisterService((Service *)&bootstrap);
}
SYS_SERVICE_INIT(Init);
```

### 3. 多阶段初始化
支持多优先级、多阶段的初始化调用：

| 阶段 | 宏定义 | 说明 |
|------|--------|------|
| MODULE_INIT | bsp/device/core/run | 板级、设备、核心、运行时初始化 |
| SYS_INIT | service/feature | 系统服务、系统特性初始化 |
| INIT_APP_CALL | service/feature | 应用级服务/特性初始化 |

## 运行环境

### 硬件要求
- **最小内存**: 128KB RAM
- **参考设备**: Hi3861V100 (WiFi IoT 芯片)

### 软件依赖

**内部依赖**:
| 依赖组件 | 版本 | 说明 |
|----------|------|------|
| samgr_lite | - | Service Manager Lite |
| libbegetutil | - | 初始化工具库 |

**第三方依赖**:
| 依赖组件 | 版本 | 说明 |
|----------|------|------|
| bounds_checking_function | - | 边界检查函数（仅 liteos_a/linux） |

### 系统集成位置

```
系统启动流程:
BIOS/UEFI
    ↓
OHOS_SystemInit() [入口]
    ↓
MODULE_INIT(bsp) → MODULE_INIT(device) → MODULE_INIT(core)
    ↓
SYS_INIT(service) → SYS_INIT(feature)
    ↓
MODULE_INIT(run)
    ↓
SAMGR_Bootstrap()
    ↓
LiteParamService()
    ↓
应用启动
```

## 关键概念

### InitCall
初始化函数指针类型，用于链接器脚本段中的初始化函数数组遍历。

**证据**: `path:services/source/bootstrap_service.h:26-33`

```c
#define APP_CALL(name, step)                                      \
    do {                                                          \
        InitCall *initcall = (InitCall *)(APP_BEGIN(name, step)); \
        InitCall *initend = (InitCall *)(APP_END(name, step));    \
        for (; initcall < initend; initcall++) {                  \
            (*initcall)();                                        \
        }                                                         \
    } while (0)
```

### Bootstrap Service
向 SAMGR 注册的系统服务，负责协调系统级启动完成后的服务初始化。

### Service Manager (SAMGR)
OpenHarmony 轻量系统的服务管理器，负责管理系统服务的注册、发现和通信。

## 版本历史

| 版本 | 发布日期 | 主要变更 |
|------|----------|----------|
| 4.0.2 | - | 当前版本 |
| 4.0.1 | - | 初始化版本 |

## 相关文档

- **[架构设计](01_Architecture.md)** - 详细架构说明
- **[初始化机制](03_Initialization.md)** - 启动流程详解
- **[构建配置](02_Build.md)** - GN 构建说明
- **[安全分析](04_Security.md)** - 安全风险评估

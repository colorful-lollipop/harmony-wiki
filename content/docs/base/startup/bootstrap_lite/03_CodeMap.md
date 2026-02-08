# Bootstrap_Lite - 代码地图

## 目录结构总览

```
base/startup/bootstrap_lite/
├── figures/                              # 架构图资源
│   └── bootstrap_lite系统架构.png
├── services/                             # 服务实现目录
│   ├── BUILD.gn                          # 组件级构建配置
│   └── source/                           # 源代码目录
│       ├── BUILD.gn                      # 源文件构建配置
│       ├── bootstrap_service.h           # 应用级初始化宏定义
│       ├── bootstrap_service.c           # Bootstrap 服务实现
│       ├── core_main.h                   # 系统级初始化宏定义
│       └── system_init.c                 # 系统初始化入口
├── bundle.json                           # 组件配置文件
├── LICENSE                               # Apache 2.0 许可证
├── OAT.xml                               # OpenHarmony 软件资产追踪
├── README.md                             # 英文项目说明
├── README_zh.md                          # 中文项目说明
└── wiki/                                 # Wiki 文档目录
    ├── README.md
    ├── SUMMARY.md
    ├── 00_Overview.md
    ├── 01_Architecture.md
    ├── 02_Build.md
    ├── 03_Initialization.md
    ├── 03_CodeMap.md                     # 本文档
    ├── 04_Security.md
    └── _work/                            # 工作区
        ├── ASSESSMENT.md
        ├── NOTES.md
        └── PLAN.md
```

---

## 核心文件导航

### 1. 系统初始化入口

**文件**: `services/source/system_init.c`

**职责**: 提供 `OHOS_SystemInit()` 系统级初始化入口函数

**关键符号**:
| 符号 | 类型 | 行号 | 说明 |
|------|------|------|------|
| `OHOS_SystemInit()` | 函数 | 19 | 系统初始化入口 |
| `MODULE_INIT()` | 宏调用 | 21-23, 26 | 模块初始化阶段 |
| `SYS_INIT()` | 宏调用 | 24-25 | 系统初始化阶段 |
| `SAMGR_Bootstrap()` | 函数调用 | 27 | 启动 SAMGR |
| `LiteParamService()` | 函数调用 | 28 | 启动参数服务 |

**代码路径**:
```
system_init.c:19 OHOS_SystemInit()
├── MODULE_INIT(bsp)      // core_main.h:74-77
├── MODULE_INIT(device)   // core_main.h:74-77
├── MODULE_INIT(core)     // core_main.h:74-77
├── SYS_INIT(service)     // core_main.h:69-72
├── SYS_INIT(feature)     // core_main.h:69-72
├── MODULE_INIT(run)      // core_main.h:74-77
├── SAMGR_Bootstrap()     // 外部: samgr_lite
└── LiteParamService()    // 外部: libbegetutil
```

---

### 2. Bootstrap 服务实现

**文件**: `services/source/bootstrap_service.c`

**职责**: 实现向 SAMGR 注册的 Bootstrap 系统服务

**关键符号**:
| 符号 | 类型 | 行号 | 说明 |
|------|------|------|------|
| `Bootstrap` | 结构体 | 20-24 | 服务实例结构 |
| `Init()` | 函数 | 30-40 | 服务初始化 |
| `GetName()` | 函数 | 42-46 | 返回服务名 |
| `Initialize()` | 函数 | 48-53 | 服务初始化回调 |
| `MessageHandle()` | 函数 | 55-80 | 消息处理主函数 |
| `GetTaskConfig()` | 函数 | 82-89 | 返回任务配置 |
| `BOOTSTRAP_SERVICE` | 宏 | 45 | 服务名称常量 |

**消息处理映射**:
```
MessageHandle() bootstrap_service.c:55
├── BOOT_SYS_COMPLETED    // 58-66: 系统启动完成
│   └── INIT_APP_CALL()   // 调用应用级初始化
├── BOOT_APP_COMPLETED    // 68-70: 应用启动完成
└── BOOT_REG_SERVICE      // 72-74: 服务注册
```

**任务配置** (`bootstrap_service.c:87`):
```c
TaskConfig config = {LEVEL_HIGH, PRI_NORMAL, 0x800, 20, SHARED_TASK};
// 优先级级别: LEVEL_HIGH
// 任务优先级: PRI_NORMAL
// 栈大小: 0x800 (2KB)
// 消息队列深度: 20
// 任务类型: SHARED_TASK
```

---

### 3. 应用级初始化宏定义

**文件**: `services/source/bootstrap_service.h`

**职责**: 定义应用级（APP）初始化宏，用于注册和调用应用服务/特性

**关键符号**:
| 符号 | 类型 | 行号 | 说明 |
|------|------|------|------|
| `APP_NAME()` | 宏 | 23 | 生成链接器段名称 |
| `MODULE_NAME()` | 宏 | 24 | 生成模块段名称 |
| `APP_CALL()` | 宏 | 26-33 | 遍历调用 APP 初始化函数 |
| `MODULE_CALL()` | 宏 | 35-42 | 遍历调用模块初始化函数 |
| `APP_BEGIN()` | 宏 | 45-49 | GCC 方式段起始符号 |
| `APP_END()` | 宏 | 51-55 | GCC 方式段结束符号 |
| `MODULE_BEGIN()` | 宏 | 57-66 | 模块段起始/结束 |
| `INIT_APP_CALL()` | 宏 | 68-71 | 简化调用宏 |
| `INIT_TEST_CALL()` | 宏 | 73-76 | 测试初始化调用 |

**支持的编译器**:
- GCC / Clang (`__GNUC__` 或 `__clang__`): 行 44-76
- IAR ARM (`__ICCARM__`): 行 78-116
- 其他: 编译错误 (行 118)

**链接器段名称格式**:
```c
// APP 级
".zinitcall.app." #name #step ".init"  // 如: .zinitcall.app.service0.init

// 模块级
".zinitcall." #name #step ".init"      // 如: .zinitcall.bsp0.init
```

---

### 4. 系统级初始化宏定义

**文件**: `services/source/core_main.h`

**职责**: 定义系统级（SYS）和模块级（MODULE）初始化宏

**关键符号**:
| 符号 | 类型 | 行号 | 说明 |
|------|------|------|------|
| `SYS_NAME()` | 宏 | 23 | 生成系统段名称 |
| `MODULE_NAME()` | 宏 | 24 | 生成模块段名称 |
| `SYS_CALL()` | 宏 | 26-33 | 遍历调用 SYS 初始化函数 |
| `MODULE_CALL()` | 宏 | 35-42 | 遍历调用 MODULE 初始化函数 |
| `SYS_BEGIN()` | 宏 | 46-50 | GCC 系统段起始 |
| `SYS_END()` | 宏 | 52-56 | GCC 系统段结束 |
| `MODULE_BEGIN()` | 宏 | 58-67 | GCC 模块段起止 |
| `SYS_INIT()` | 宏 | 69-72 | 简化系统调用 |
| `MODULE_INIT()` | 宏 | 74-77 | 简化模块调用 |

**IAR 扩展** (行 79-137):
- 支持 5 个步骤 (0-4) 的初始化
- 使用 `#pragma section` 指令

---

### 5. 源文件构建配置

**文件**: `services/source/BUILD.gn`

**职责**: 定义 `libbootstrap.a` 静态库的构建规则

**关键配置**:
| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sources` | `bootstrap_service.c`, `system_init.c` | 源文件 |
| `include_dirs` | 4 个路径 | 头文件搜索路径 |
| `deps` | `libbegetutil` | 依赖库 |
| `cflags` | `-Wall` | 编译警告 |

**条件编译**:
```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  // 轻量内核：无额外包含
} else if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  // 标准内核：添加 bounds_checking_function
  include_dirs += [ "//third_party/bounds_checking_function/include" ]
}
```

---

### 6. 组件级构建配置

**文件**: `services/BUILD.gn`

**职责**: 定义组件级构建目标和 NDK 导出

**构建目标**:
| 目标 | 类型 | 说明 |
|------|------|------|
| `bootstrap` | lite_component | 组件入口 |
| `bootstrap_lite_ndk` | ndk_lib | NDK 库导出 |
| `bootstrap_notice_file` | notice_file | 版权声明 |

**NDK 头文件** (`head_files`):
- `//commonlibrary/utils_lite/include/ohos_init.h`
- `//commonlibrary/utils_lite/include/ohos_errno.h`
- `//commonlibrary/utils_lite/include/ohos_types.h`

---

## 符号索引表

### 按功能分类

#### 初始化入口
| 符号 | 文件 | 行号 | 类型 |
|------|------|------|------|
| `OHOS_SystemInit()` | system_init.c | 19 | 函数 |
| `Init()` | bootstrap_service.c | 30 | 函数 (static) |
| `SYS_SERVICE_INIT()` | 外部 | - | 宏 (来自 ohos_init.h) |

#### 初始化宏（系统级）
| 符号 | 文件 | 行号 | 类型 |
|------|------|------|------|
| `SYS_INIT()` | core_main.h | 69 | 宏 |
| `MODULE_INIT()` | core_main.h | 74 | 宏 |
| `SYS_CALL()` | core_main.h | 26 | 宏 |
| `MODULE_CALL()` | core_main.h | 35 | 宏 |

#### 初始化宏（应用级）
| 符号 | 文件 | 行号 | 类型 |
|------|------|------|------|
| `INIT_APP_CALL()` | bootstrap_service.h | 68 | 宏 |
| `INIT_TEST_CALL()` | bootstrap_service.h | 73 | 宏 |
| `APP_CALL()` | bootstrap_service.h | 26 | 宏 |
| `MODULE_CALL()` | bootstrap_service.h | 35 | 宏 |

#### 服务接口
| 符号 | 文件 | 行号 | 类型 |
|------|------|------|------|
| `BOOTSTRAP_SERVICE` | bootstrap_service.c | 45 | 常量 (宏) |
| `MessageHandle()` | bootstrap_service.c | 55 | 函数 (static) |
| `GetTaskConfig()` | bootstrap_service.c | 82 | 函数 (static) |
| `Bootstrap` | bootstrap_service.c | 20 | 结构体 |

#### 消息 ID
| 符号 | 值/来源 | 说明 |
|------|---------|------|
| `BOOT_SYS_COMPLETED` | 外部 (samgr_lite) | 系统启动完成 |
| `BOOT_APP_COMPLETED` | 外部 (samgr_lite) | 应用启动完成 |
| `BOOT_REG_SERVICE` | 外部 (samgr_lite) | 注册服务 |

---

## 外部依赖接口

### SAMGR (Service Manager Lite)

**头文件**: `samgr_lite.h`

**使用的接口**:
| 接口 | 用途 | 位置 |
|------|------|------|
| `SAMGR_GetInstance()` | 获取 SAMGR 单例 | bootstrap_service.c:38 |
| `RegisterService()` | 注册 Bootstrap 服务 | bootstrap_service.c:38 |
| `SAMGR_SendResponseByIdentity()` | 发送响应 | bootstrap_service.c:65,69,73 |
| `SAMGR_Bootstrap()` | 启动 SAMGR | system_init.c:27 |

### Init 工具库 (libbegetutil)

**使用的接口**:
| 接口 | 用途 | 位置 |
|------|------|------|
| `LiteParamService()` | 启动参数服务 | system_init.c:28 |

---

## 代码阅读路径建议

### 路径 A：理解启动流程（10 分钟）

1. `system_init.c:19` - 查看 `OHOS_SystemInit()` 函数
2. `core_main.h:69-77` - 理解 `SYS_INIT()` / `MODULE_INIT()` 宏
3. `bootstrap_service.c:30-40` - 查看服务注册
4. `bootstrap_service.c:55-80` - 查看消息处理

### 路径 B：理解链接器段机制（15 分钟）

1. `core_main.h:46-67` - GCC 段符号定义 (`SYS_BEGIN`, `SYS_END`)
2. `core_main.h:26-42` - 遍历调用宏 (`SYS_CALL`, `MODULE_CALL`)
3. `bootstrap_service.h:45-66` - APP 级段定义
4. `system_init.c:19-29` - 实际调用链

### 路径 C：安全审计（20 分钟）

1. `bootstrap_service.c:55-80` - 消息处理（输入点）
2. `bootstrap_service.c:87` - 任务配置（资源限制）
3. `bootstrap_service.h:23-24` - 段名称定义（命名冲突）
4. `services/source/BUILD.gn:24-28` - 条件编译（边界检查）

---

## 相关文档链接

- **[项目概览](../00_Overview.md)** - 项目定位和核心能力
- **[架构设计](../01_Architecture.md)** - 组件关系和数据流
- **[初始化机制](../03_Initialization.md)** - 启动流程深入分析
- **[构建配置](../02_Build.md)** - GN 构建系统详解
- **[安全分析](../04_Security.md)** - 安全风险评估

---

*最后更新: 2024-02*
*版本: 4.0.2*

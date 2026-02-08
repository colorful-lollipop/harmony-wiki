# 编译产物参考

## 目的

本文档详细说明 ace_engine_lite 的编译产物、输出路径、运行时加载关系和产物用途。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 所有编译产物（.so/.a 文件）

---

## 产物类型

### 动态库（.so 文件）

#### 产物列表

| 库文件 | 目标平台 | 输出目标 | 主要用途 |
|---------|----------|----------|------|
| `libace_lite.so` | liteos_a, linux | ace_lite | 主框架库 |
| `libace_common.so` | liteos_a, linux | ace_common | 公共工具库 |
| `libace_native_engine.so` | liteos_a, linux | ace_native_engine | JS 引擎适配 |
| `libace_module_manager.so` | liteos_a, linux | ace_module_manager | 模块管理器 |

#### 输出路径

```bash
# 默认输出目录
out/<board_name>/libs/
```

#### 运行时加载

**动态库加载方式**：
- **LiteOS-A**: 静态链接到应用（.a 文件）
- **LiteOS-M**: 静态链接到应用（.a 文件）
- **Linux**: 动态加载（dlopen 或 LD_PRELOAD）

**证据**：无显式 dlopen 调用（由系统加载器处理）

---

### 静态库（.a 文件）

#### 产物列表

| 库文件 | 目标平台 | 输出目标 | 主要用途 |
|---------|----------|----------|------|
| `libace_lite.a` | liteos_m | ace_lite | 主框架库 |
| `libace_common.a` | liteos_m | ace_common | 公共工具库 |
| `libace_native_engine.a` | liteos_m | ace_native_engine | JS 引擎适配 |
| `libace_module_manager.a` | liteos_m | ace_module_manager | 模块管理器 |

#### 输出路径

```bash
# 默认输出目录
out/<board_name>/libs/
```

#### 运行时使用

**静态库使用方式**：
- **LiteOS-M**: 直接链接到应用镜像
- **Simulator**: 静态链接到可执行文件

**证据**：`frameworks/BUILD.gn:51-55`（target_type 配置）

---

### 产物组织结构

### LiteOS-A 平台

```
out/<board_name>/
├── libs/
│   ├── libace_lite.so
│   ├── libace_common.so
│   ├── libace_native_engine.so
│   └── libace_module_manager.so
├── bin/
│   └── 应用可执行文件（链接上述 .so）
└── ...
```

### LiteOS-M 平台

```
out/<board_name>/
├── libs/
│   ├── libace_lite.a
│   ├── libace_common.a
│   ├── libace_native_engine.a
│   └── libace_module_manager.a
└── 应用可执行文件（静态链接上述 .a）
```

### Simulator 平台

```
out/<board_name>/
├── libs/
│   └── libace_lite.a
└── 模拟器可执行文件（静态链接 libace_lite.a）
```

**证据**：各平台 BUILD.gn 配置

---

## 产物用途说明

### libace_lite.so/libace_lite.a

**作用**：主框架库，包含所有核心功能

**主要组件**：
- UI 组件系统
- 路由和页面管理
- 样式管理
- JS 模块系统
- 上下文管理
- 动画支持

**大小估算**：
- ROM: ~521KB
- RAM: ~82KB

**证据**：`bundle.json:22-23`（rom: "521KB", ram: "~82KB"）

---

### libace_common.so/libace_common.a

**作用**：公共工具库

**主要功能**：
- 日志系统（HILOG）
- 内存管理（堆、缓存）
- 基础工具类（字符串、数字、时间等）

**依赖**：
- `bounds_checking_function`（安全 C 函数）

**证据**：`frameworks/common/BUILD.gn:48-54`

---

### libace_native_engine.so/libace_native_engine.a

**作用**：JerryScript 引擎的 C++ 封装层

**主要功能**：
- JSI（JavaScript Interface）抽象
- 异步任务管理
- 消息队列
- JS 值操作和类型转换

**依赖**：
- `jerryscript`（JerryScript 引擎）

**证据**：`frameworks/native_engine/BUILD.gn:63-72`

---

### libace_module_manager.so/libace_module_manager.a

**作用**：JS 模块加载和管理

**主要功能**：
- 模块按需加载（require）
- 内置模块管理
- 产品模块扩展支持
- 私有模块支持
- 模块生命周期管理

**依赖**：
- `ace_common_lite`
- `ace_native_engine_lite`
- `ace_utils_kits`

**证据**：`frameworks/module_manager/BUILD.gn:50-54`

---

## 运行时加载机制

### 应用启动流程

```
┌─────────────────────────────────────────────────┐
│  1. 系统启动                                        │
│     init 进程                                        │
│    ↓                                                   │
│  2. 加载 ACE 框架                                  │
│     dlopen("libace_lite.so") 或静态链接                │
│    ↓                                                   │
│  3. 初始化 JS 引擎                                  │
│     JerryScript::jerry_init()                        │
│    ↓                                                   │
│  4. 加载应用 JS 代码                                  │
│     requireNative("system.app")                        │
│     ModuleManager::RequireModule()                      │
│    ↓                                                   │
│  5. 解析和执行 app.js                              │
│     jerry_parse() + jerry_run()                        │
│    ↓                                                   │
│  6. 创建 UI 组件树                                  │
│     Component::RenderComponent()                       │
│    ↓                                                   │
│  7. 渲染到屏幕                                      │
│     UI Lite 渲染                                     │
└─────────────────────────────────────────────────────────┘
```

**证据**：`frameworks/src/core/context/js_app_environment.cpp:72-97`

### 模块加载流程

```
JS: const app = requireNative("system.app");
  ↓
ModuleManager::RequireModule("system.app")
  ↓
ParseModuleName("system.app")  // 解析为 category="system", name="app"
  ↓
GetModuleObject("app")  // 在 OHOS_MODULES 数组中查找
  ↓
InitAppModule(exports)  // 调用初始化函数
  ↓
JSI::SetModuleAPI(exports, "getInfo", AppModule::GetInfo)
  ↓
return exports  // 返回 exports 对象给 JS
```

**证据**：`frameworks/module_manager/module_manager.cpp:30-66`

---

## 安装路径

### 系统路径

| 平台 | 库文件安装路径 |
|------|----------------|-------------------|
| LiteOS-A/Linux | `/usr/lib/` 或 `/system/lib/` |
| Simulator | 与可执行文件同目录 |

### 应用资源路径

**JS Bundle**：`/storage/data/apps/<bundle_name>/assets/js/default/`

**样式文件**：`/storage/data/apps/<bundle_name>/resources/base/style/` 或 `style.css`

**配置文件**：`/storage/data/apps/<bundle_name>/manifest.json`

**证据**：`frameworks/src/core/context/js_app_context.cpp:77-113`

---

## 产物大小分析

### 代码统计

| 类别 | 文件数量 | 行数（估算） |
|------|----------|-------------|
| 核心 src/ | ~102 | ~30,000 |
| 模块 modules/ | ~22 | ~8,000 |
| 组件 components/ | ~41 | ~15,000 |
| 样式 stylemgr/ | ~9 | ~5,000 |
| 工具 base/ | ~15 | ~4,000 |
| 头文件 include/ | ~50 | ~10,000 |
| **总计** | **~239** | **~72,000** |

### ROM 占用分解

| 组件 | ROM 大小（估算） |
|------|----------------|-------------|
| 核心框架（ace_lite） | ~300KB |
| JS 引擎（JerryScript） | ~150KB |
| 图形框架（ui_lite） | ~50KB |
| 其他依赖（i18n, resmgr 等） | ~21KB |
| **总计** | **~521KB** |

**证据**：`bundle.json:22`（rom: "521KB"）

---

## 构建优化

### 编译器优化

```gn
# liteos_a 平台使用 ICCARM 编译器
if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
    cflags = ["--diag_suppress", "Pe111,Pa137,Pe177,Pa205,Pe226,Pe366,Pe367"]
}
```

**证据**：`frameworks/BUILD.gn:31-36`

### 代码优化

**启用优化**：
- `-O2` / `-O3`（优化级别）
- `-flto`（链接时优化，如果支持）
- `-Os`（代码大小优化）

---

## 运行时检查清单

### 关键检查点

| 检查项 | 位置 | 说明 |
|---------|------|------|
| JerryScript 初始化 | `js_app_environment.cpp:84` | 确保引擎正常启动 |
| 模块加载 | `module_manager.cpp:30-66` | 确保所有必需模块可用 |
| Ability 启动 | `ace_ability.cpp:30-62` | 确保应用正确加载 |
| 内存分配 | `ace_log.cpp` | 监控内存使用 |
| 错误处理 | `dfx_assist.cpp` | 捕获和记录运行时错误 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
- [06_GN_Targets.md](06_GN_Targets.md) - GN 构建目标

# Hilog Lite 编译产物

本文档描述 hilog_lite 组件的编译产物、安装路径和运行时加载关系。

## 产物清单

### 框架层产物

| Target | 类型 | 产物名称 | 适用系统 |
|--------|------|----------|----------|
| `frameworks/mini:hilog_lite_static` | 静态库 | `libhilog_lite_static.a` | LiteOS-M |
| `frameworks/mini:hilog_lite` | 静态库/配置 | `libhilog_lite_static.a` 或配置 | LiteOS-M |
| `frameworks/featured:hilog_static` | 静态库 | `libhilog_static.a` | LiteOS-A |
| `frameworks/featured:hilog_shared` | 动态库 | `libhilog_shared.so` | LiteOS-A |

### 服务层产物

| Target | 类型 | 产物名称 | 描述 |
|--------|------|----------|------|
| `services/hilogcat:hilogcat` | 可执行 | `hilogcat` | 日志查看命令行工具 |
| `services/apphilogcat:apphilogcat` | 可执行 | `apphilogcat` | 应用日志落盘服务 |

### 命令层产物

| Target | 类型 | 产物名称 |
|--------|------|----------|
| `command:hilog_command_static` | 静态库 | `libhilog_command_static.a` |
| `command:hilog_command_shared` | 动态库 | `libhilog_command_shared.so` |

### JS 层产物

| Target | 类型 | 产物名称 |
|--------|------|----------|
| `frameworks/js:ace_hilog_kits` | 组件 | JS 日志绑定 |

---

## 产物输出路径

### 默认输出路径

```
out/{product}/libs/                    # 库文件
out/{product}/bin/                    # 可执行文件
```

### NDK 输出路径

```
out/{product}/ndk/                    # NDK 产物
├── hilog_lite/                       # 轻量系统 NDK
│   ├── include/                      # 头文件
│   │   ├── hiview_log.h
│   │   └── log.h
│   └── libhilog_lite_static.a        # 静态库
└── hilog/                            # 小型系统 NDK
    ├── include/                      # 头文件
    │   ├── hilog_cp.h
    │   ├── hilog_trace.h
    │   ├── hiview_log.h
    │   └── log.h
    └── libhilog_shared.so           # 动态库
```

---

## 头文件输出

### 轻量系统对外接口 (kits/hilog_lite)

```
interfaces/native/kits/hilog_lite/
├── log.h              # 日志宏包装
└── hiview_log.h       # 核心 API 定义
```

### 小型系统对外接口 (kits/hilog)

```
interfaces/native/kits/hilog/
└── log.h              # 日志 API
```

### 小型系统内部接口 (innerkits/hilog)

```
interfaces/native/innerkits/hilog/
├── hilog_cp.h         # C++ HiLog 类封装
├── hilog_trace.h      # 追踪 API
├── hiview_log.h       # 核心 API
└── log.h              # 日志 API
```

---

## 运行时加载关系

### 静态库使用方式

```
┌─────────────────────────────────────────────────────────┐
│                   应用程序 (App)                         │
└─────────────────────┬───────────────────────────────────┘
                      │ #include <hilog/log.h>
                      │ #include "log.h"
                      ▼
┌─────────────────────────────────────────────────────────┐
│              链接 libhilog_static.a                      │
│                   (编译时链接)                           │
└─────────────────────────────────────────────────────────┘
```

**CMake/MAKE 添加方式**:

```makefile
# 静态链接
LDFLAGS += -L$(OUT)/libs -lhilog_static
```

### 动态库使用方式

```
┌─────────────────────────────────────────────────────────┐
│                   应用程序 (App)                         │
└─────────────────────┬───────────────────────────────────┘
                      │ #include <hilog/log.h>
                      ▼
┌─────────────────────────────────────────────────────────┐
│              链接 libhilog_shared.so                     │
│                   (运行时加载)                           │
└─────────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│           System Loader (dlopen/dlsym)                   │
│                   动态加载 .so                           │
└─────────────────────────────────────────────────────────┘
```

**CMake 添加方式**:

```cmake
# 动态链接
target_link_libraries(app PUBLIC hilog_shared)
```

### NDK 使用方式

```c
// 包含头文件
#include <hilog/log.h>

// 链接库 (在 Android.bp 或 CMakeLists.txt 中)
// static: libhilog_lite_static.a
// shared: libhilog_shared.so
```

---

## 库依赖关系

### libhilog_static.a 依赖

```
libhilog_static.a
├── libsec_static.a (bounds_checking_function)
└── hiview_lite (utils, samgr)
```

### libhilog_shared.so 依赖

```
libhilog_shared.so
├── libsec_shared.so (bounds_checking_function)
├── libc.so
└── libhilog_shared.so
```

> 证据来源: frameworks/featured/BUILD.gn:51-52, 62-63

### hilogcat 依赖

```
hilogcat (executable)
├── libhilog_command_*.a (静态) / .so (动态)
├── libhilog_shared.so
└── libsec_shared.so
```

> 证据来源: services/hilogcat/BUILD.gn:19-23

### apphilogcat 依赖

```
apphilogcat (executable)
├── libhilog_command_*.a (静态) / .so (动态)
├── libhilog_shared.so
└── libsec_shared.so
```

> 证据来源: services/apphilogcat/BUILD.gn:54-59

---

## 安装路径

### 系统库安装

| 产物 | 安装路径 | 描述 |
|------|----------|------|
| `libhilog_shared.so` | `/system/lib/` | 系统动态库 |
| `libhilog_static.a` | `/system/lib/` | 系统静态库 (可选) |
| `hilogcat` | `/system/bin/` | 系统工具 |
| `apphilogcat` | `/system/bin/` | 系统DK 安装

| 产物 | 安装服务 |

### N路径 | 描述 |
|------|----------|------|
| 头文件 | `/ndk/system/include/hilog/` | NDK 头文件 |
| `libhilog_shared.so` | `/ndk/system/lib/` | NDK 动态库 |
| `libhilog_lite_static.a` | `/ndk/system/lib/` | NDK 静态库 |

### 日志文件安装

| 产物 | 默认路径 | 描述 |
|------|----------|------|
| 应用日志 | `/storage/data/log/` | 日志目录 |

> 证据来源: services/apphilogcat/BUILD.gn:24 (`hilog_lite_apphilogcat_log_dir = "/storage/data/log"`)

---

## 资源占用估算

### ROM 占用

| 组件 | 估算大小 |
|------|----------|
| libhilog_static.a | ~50KB |
| libhilog_shared.so | ~30KB |
| hilogcat | ~20KB |
| apphilogcat | ~25KB |
| **总计** | **~125KB** |

> 实际大小取决于编译优化选项和功能配置

### RAM 占用

| 组件 | 估算大小 |
|------|----------|
| Ring Buffer | ~8KB (默认配置) |
| 静态缓存 | ~1KB |
| 运行时堆栈 | ~2KB |
| **总计** | **~11KB** |

> 实际大小取决于 `hilog_lite_file_size` 配置

---

## 常见问题

### Q1: 静态库和动态库如何选择？

| 场景 | 推荐 | 原因 |
|------|------|------|
| 系统库 | 动态库 | 减少重复代码，便于更新 |
| _bootloader | 静态库 | 启动时依赖最少 |
| 资源受限设备 | 静态库 | 减少运行时依赖 |

### Q2: 如何减小库大小？

| 方法 | 配置项 |
|------|--------|
| 禁用隐私功能 | `hilog_lite_disable_privacy_feature = true` |
| 禁用 JS 功能 | `hilog_lite_disable_js_feature = true` |
| 禁用静态库 | `hilog_lite_disable_hilog_static = true` |
| 启用链接时优化 | LTO (Link-Time Optimization) |

### Q3: 日志文件路径如何修改？

```gn
# 在产品配置中修改
hilog_lite_apphilogcat_log_dir = "/custom/path/log"
```

---

## 相关文档

- [概览](01_Overview.md)
- [GN Targets](05_GN_Targets.md)
- [配置项](appendix/Config_Flags.md)

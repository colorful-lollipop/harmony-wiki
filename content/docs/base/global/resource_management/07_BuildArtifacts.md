# 编译产物文档

## 目的

本文档说明 OpenHarmony 资源管理组件的编译产物，包括输出文件、安装路径和运行时加载关系。

## 适用范围

本文档覆盖所有非测试的编译产物，包括共享库、静态库、ABC 文件和可执行文件。

## 关键结论

| 产物类型 | 数量 | 主要产物 | 安装路径 |
|----------|------|----------|----------|
| 共享库 (.so) | 7 个 | libglobal_resmgr.so, libresourcemanager.so | /system/lib64/ |
| Native 库 (.z) | 1 个 | resmgr_ani.z | /system/lib64/module/ |
| ABC 文件 | 3 个 | resourceManager.abc | /system/framework/ |
| 平台适配库 | 3 个 | win_resmgr, mac_resmgr, linux_resmgr | IDE 预览环境 |

## 产物清单

### 核心共享库

| Target | 输出文件 | 安装路径 | 大小估算 |
|--------|----------|----------|----------|
| `global_resmgr` | `libglobal_resmgr.so` | `/system/lib64/` | ~400KB |
| `librawfile` | `librawfile.so` | `/system/lib64/` | ~50KB |
| `ohresmgr` | `libohresmgr.so` | `/system/lib64/` | ~30KB |

**证据**: `frameworks/resmgr/BUILD.gn`, bundle.json

### JS NAPI 共享库

| Target | 输出文件 | 安装路径 | 大小估算 |
|--------|----------|----------|----------|
| `resourcemanager` | `libresourcemanager.so` | `/system/lib64/` | ~100KB |
| `sendableresourcemanager` | `libsendableresourcemanager.so` | `/system/lib64/` | ~50KB |
| `resmgr_napi_core` | `libresmgr_napi_core.so` | `/system/lib64/` | ~80KB |

**证据**: `interfaces/js/kits/BUILD.gn`, `interfaces/js/innerkits/core/BUILD.gn`

### ETS/ANI 产物

| Target | 输出文件 | 安装路径 | 大小估算 |
|--------|----------|----------|----------|
| `resmgr_ani` | `resmgr_ani.z` | `/system/lib64/module/` | ~60KB |
| `resourceManager` | `resourceManager.abc` | `/system/framework/` | ~20KB |
| `resource` | `resource.abc` | `/system/framework/` | ~10KB |
| `rawFileDescriptor` | `rawFileDescriptor.abc` | `/system/framework/` | ~5KB |

**证据**: `interfaces/ets/ani/resourceManager/BUILD.gn`

### Cangjie FFI 产物

| Target | 输出文件 | 安装路径 | 大小估算 |
|--------|----------|----------|----------|
| `cj_resource_manager_ffi` | `libcj_resource_manager_ffi.so` | `/system/lib64/` | ~40KB |

**证据**: `interfaces/cj/BUILD.gn`

### 平台适配产物

| Target | 输出文件 | 平台 | 用途 |
|--------|----------|------|------|
| `global_resmgr_win` | `libglobal_resmgr.dll` | Windows | IDE 预览 |
| `global_resmgr_mac` | `libglobal_resmgr.dylib` | macOS | IDE 预览 |
| `global_resmgr_linux` | `libglobal_resmgr.so` | Linux | IDE 预览 |

**证据**: `resmgr.gni`, `frameworks/resmgr/BUILD.gn`

## 安装路径

### 系统分区

| 路径 | 用途 | 权限 |
|------|------|------|
| `/system/lib64/` | 系统共享库 | 只读 |
| `/system/framework/` | ETS ABC 文件 | 只读 |
| `/system/lib64/module/` | Native 模块 | 只读 |

### 应用分区

| 路径 | 用途 | 权限 |
|------|------|------|
| `/data/app/` | 应用安装目录 | 应用私有 |
| `/data/storage/` | 应用存储目录 | 应用私有 |

### 系统资源

| 路径 | 说明 | 权限 |
|------|------|------|
| `/system/app/ohos.global.systemres/` | 系统资源包 (非沙箱) | 只读 |
| `/data/storage/el1/bundle/ohos.global.systemres/` | 系统资源包 (沙箱) | 只读 |

**证据**: `frameworks/resmgr/src/system_resource_manager.cpp:25-41`

## 运行时加载关系

### JS 应用加载流程

```
应用启动
    ↓
加载 libresourcemanager.so
    ↓
加载 libglobal_resmgr.so (依赖)
    ↓
加载 libhilog.so (依赖)
    ↓
加载 libhisysevent.so (依赖)
    ↓
NAPI 模块注册
    ↓
JS 应用调用 @ohos.resmgr
```

**证据**: `interfaces/js/kits/BUILD.gn` deps, `bundle.json`

### ArkTS 应用加载流程

```
应用启动
    ↓
加载 resmgr_ani.z (Native 模块)
    ↓
加载 libglobal_resmgr.so (依赖)
    ↓
加载 resourceManager.abc (ETS 模块)
    ↓
ANI 模块注册
    ↓
ETS 应用调用 @ohos.resourceManager
```

**证据**: `interfaces/ets/ani/resourceManager/BUILD.gn`

### Native 应用加载流程

```
应用启动
    ↓
加载 libohresmgr.so
    ↓
加载 libglobal_resmgr.so (依赖)
    ↓
Native 应用调用 Native API
```

**证据**: `frameworks/resmgr/BUILD.gn` deps

### 依赖链分析

#### libresourcemanager.so

```
libresourcemanager.so
    ↓
libresmgr_napi_core.so
    ↓
libglobal_resmgr.so
    ↓
libhilog.so
libhisysevent.so
libhitrace.so
libz.so
libcjson.so
libicui18n.so (可选)
libicuuc.so (可选)
```

#### resmgr_ani.z

```
resmgr_ani.z
    ↓
libglobal_resmgr.so
    ↓
libhilog.so
libhisysevent.so
libz.so
libcjson.so
libicui18n.so (可选)
```

**证据**: BUILD.gn deps 配置

## 库加载顺序

### 加载顺序

1. **核心库** (libglobal_resmgr.so) - 最先加载
2. **绑定层库** (libresmgr_napi_core.so, resmgr_ani.z)
3. **API 库** (libresourcemanager.so, libohresmgr.so)
4. **系统依赖库** (hilog, hisysevent, zlib, etc.)

### 符号解析

- **核心库** 不依赖其他组件的库
- **绑定层库** 依赖核心库和 N-API 框架
- **API 库** 依赖绑定层库
- **系统依赖库** 独立加载

**证据**: BUILD.gn deps 顺序

## 动态加载与静态链接

### 动态加载

**产物类型**: 所有 `.so` 和 `.z` 文件

**加载时机**: 运行时按需加载

**优点**:
- 内存占用小（多个应用共享）
- 更新方便（无需重新编译应用）

**缺点**:
- 启动时间略长（需要加载多个库）

**证据**: BUILD.gn target 类型

### 静态链接

**产物类型**: 无（所有库都是动态链接）

**说明**: 资源管理组件不提供静态库，所有依赖都是动态链接

**证据**: BUILD.gn target 类型 (ohos_shared_library)

## 版本管理

### 库版本

| 库 | 版本 | 命名空间 |
|-----|------|----------|
| libglobal_resmgr.so | 4.0 | OHOS::Global::Resource |
| libresourcemanager.so | 4.0 | NAPI 模块 |
| libohresmgr.so | 4.0 | Native API |

**证据**: `bundle.json:version: "4.0"`

### ABI 稳定性

| 接口 | 稳定性 | 兼容性 |
|------|--------|----------|
| JS N-API | 稳定 | 向后兼容 |
| Native API | 稳定 | 向后兼容 |
| Inner API | 相对稳定 | 可能变更 |
| 内部实现 | 不稳定 | 可能变更 |

**证据**: 头文件位置和文档说明

## 性能影响

### 内存占用

| 库 | 静态内存 | 动态内存 (估算) |
|-----|----------|-----------------|
| libglobal_resmgr.so | ~300KB | ~2MB (资源缓存) |
| libresourcemanager.so | ~50KB | ~100KB |
| resmgr_ani.z | ~40KB | ~80KB |

**证据**: bundle.json ROM/RAM 估算

### 启动时间

| 阶段 | 时间 (估算) |
|------|------------|
| 加载 libglobal_resmgr.so | ~5ms |
| 加载 libresmgr_napi_core.so | ~2ms |
| 加载 libresourcemanager.so | ~2ms |
| NAPI 模块注册 | ~1ms |
| **总计** | **~10ms** |

**证据**: 库大小和依赖数量估算

## 调试支持

### 符号表

**产物**: `.so` 文件包含调试符号

**用途**: GDB/LLDB 调试

**说明**: Release 构建会去除符号表

**证据**: GN 配置

### 日志

**库**: libhilog.so

**使用**: 所有库使用 HiLog 记录日志

**日志标签**: ResourceMgr

**证据**: `dfx/hisysevent_adapter/`, `hilog_wrapper.h`

### 系统事件

**库**: libhisysevent.so

**事件类型**: 资源加载、资源查找失败等

**配置文件**: `hisysevent.yaml`

**证据**: `hisysevent.yaml`

## 常见问题

### 库加载失败

**症状**: 应用崩溃或无法找到资源

**可能原因**:
1. 依赖库未安装
2. 库路径错误
3. 库版本不匹配

**定位方法**:
1. 检查 `/system/lib64/` 目录
2. 检查日志中的加载错误
3. 使用 `ldd` 命令检查依赖

### ABC 文件未找到

**症状**: ETS 应用无法加载资源管理模块

**可能原因**:
1. ABC 文件未正确安装
2. Native 模块未加载

**定位方法**:
1. 检查 `/system/framework/` 目录
2. 检查 `/system/lib64/module/` 目录
3. 检查 ANI 加载日志

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [GN Targets](06_GNTargets.md) - 构建系统和依赖关系
- [常见问题](09_FAQ.md) - 问题排查指南

---

**生成时间**: 2026-02-06
**证据来源**: bundle.json, BUILD.gn, frameworks/resmgr/src/system_resource_manager.cpp

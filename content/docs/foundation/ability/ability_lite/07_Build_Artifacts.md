# 编译产物

## 目的

本文档描述 ability_lite 的编译输出产物，包括库文件、可执行文件和安装路径。

## 适用范围

- 需要了解输出产物的开发者
- 系统集成人员

## 产物清单

### 库文件

| 产物名称 | 类型 | 输出路径 | 说明 |
|----------|------|----------|------|
| `libability.so` | shared_library | `system/lib/` | AbilityKit 库 (标准系统) |
| `libability.a` | static_library | 链接到最终镜像 | AbilityKit 库 (LiteOS-M) |
| `libabilitymanager.so` | shared_library | `system/lib/` | AbilityManager 客户端 (标准) |
| `libabilitymanager.a` | static_library | 链接到最终镜像 | AbilityManager 客户端 (LiteOS-M) |
| `libabilityms.so` | shared_library | `system/lib/` | AMS 服务库 (标准) |
| `libabilityms.a` | static_library | 链接到最终镜像 | AMS 服务库 (LiteOS-M) |
| `libwant.a` | static_library | 链接到其他库 | Want 库 |
| `libaafwk.so` | shared_library | `system/lib/module/` | N-API JS 绑定 |

### 可执行文件

| 产物名称 | 类型 | 输出路径 | 说明 |
|----------|------|----------|------|
| `aa` | executable | `dev_tools/bin/` | AMS 命令行工具 |

## 产物详细说明

### libability.so / libability.a

**来源**: `frameworks/ability_lite:ability`

**包含功能**:
- Ability 基类实现
- AbilitySlice 管理 (条件编译)
- Ability 生命周期调度
- 主线程管理

**依赖**:
```
libabilitymanager.so
libbundle.so
libipc_single.so
libhilog_shared.so
libkv_store.so
```

**安装路径**:
- 标准系统: `/system/lib/libability.so`
- LiteOS-M: 静态链接到应用或系统镜像

### libabilitymanager.so / libabilitymanager.a

**来源**: `frameworks/abilitymgr_lite:abilitymanager`

**包含功能**:
- AMS 客户端通信
- StartAbility/StopAbility API
- IPC 封装

**依赖**:
```
libbundle.so
libipc_single.so
libhilog_shared.so
```

**安装路径**:
- 标准系统: `/system/lib/libabilitymanager.so`
- LiteOS-M: 静态链接

### libabilityms.so / libabilityms.a

**来源**: `services/abilitymgr_lite:abilityms`

**包含功能**:
- AMS 服务实现
- Ability 生命周期管理
- 应用进程管理
- Ability 栈管理
- 任务调度

**依赖**:
```
libsamgr.so
libbundle.so
libipc_single.so
libhilog_shared.so
```

**安装路径**:
- 标准系统: `/system/lib/libabilityms.so`
- LiteOS-M: 静态链接到系统镜像

**运行时**: 作为系统服务运行在 foundation 进程

### libaafwk.so

**来源**: `interfaces/kits/js/napi:aafwk`

**包含功能**:
- N-API 模块实现
- JS startAbility/stopAbility 绑定

**依赖**:
```
libabilitymanager.so
libace_napi.so
```

**安装路径**: `/system/lib/module/libaafwk.so`

**运行时加载**: JS 引擎通过 `require('@ohos.aafwk')` 加载

### aa 工具

**来源**: `services/abilitymgr_lite/tools:aa`

**包含功能**:
- 启动 Ability: `aa start -p <bundle> -n <ability>`
- 停止 Ability: `aa stop -p <bundle> -n <ability>`
- Dump 信息: `aa dump`

**依赖**:
```
libabilitymanager.so
libsamgr.so
libbundle.so
libipc_single.so
libhilog_shared.so
libkv_store.so
libcjson_shared.so
```

**输出路径**: `$root_out_dir/dev_tools/aa`

**安装路径**: `/dev_tools/bin/aa`

## 运行时加载关系

### 标准系统 (Linux-based)

```
Foundation Process:
├─ foundation
│   ├─ libabilityms.so (AMS 服务)
│   ├─ libsamgr.so (服务管理)
│   └─ ...

Application Process:
├─ app_process
│   ├─ libability.so (AbilityKit)
│   ├─ libabilitymanager.so (AMS 客户端)
│   └─ ...

JS Runtime:
└─ libaafwk.so (N-API 模块)
    └─ 加载时依赖 libabilitymanager.so
```

### LiteOS-M 系统

```
System Image:
├─ ability_lite 代码静态链接到系统镜像
│   ├─ abilityms (AMS 服务)
│   ├─ ability (AbilityKit)
│   └─ abilitymanager (AMS 客户端)
│
└─ 应用代码静态链接
    └─ 与系统共享 AbilityKit 代码
```

## 产物大小估算

| 产物 | 估算大小 (Release) |
|------|-------------------|
| libability.so | 100-200 KB |
| libabilitymanager.so | 50-100 KB |
| libabilityms.so | 200-400 KB |
| libaafwk.so | 20-50 KB |
| aa | 100-200 KB |

**总 ROM 占用**: 约 500-1000 KB (标准系统，含依赖)

**RAM 占用** (运行时):
- AMS 服务: 100-200 KB
- 每个应用进程: 50-100 KB (AbilityKit)

## 安装路径映射

### 标准系统

| 产物 | 源码路径 | 安装路径 |
|------|----------|----------|
| libability.so | `frameworks/ability_lite` | `/system/lib/` |
| libabilitymanager.so | `frameworks/abilitymgr_lite` | `/system/lib/` |
| libabilityms.so | `services/abilitymgr_lite` | `/system/lib/` |
| libaafwk.so | `interfaces/kits/js/napi` | `/system/lib/module/` |
| aa | `services/abilitymgr_lite/tools` | `/dev_tools/bin/` |

### SDK/NDK 输出

| 产物 | 输出路径 |
|------|----------|
| ability_notes (NDK) | `ndk/ability/` |
| 头文件 | `ndk/ability/include/` |
| 库文件 | `ndk/ability/lib/` |

## 相关链接

- [GN 构建目标](06_GN_Targets.md)
- [架构说明](02_Architecture.md)
- [目录结构](01_Directory_Structure.md)

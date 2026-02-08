# 编译产物文档

## 目的

本文档详细说明 EventHandler 部件的编译产物，包括输出文件、安装路径和运行时加载关系。

## 适用范围

- 构建系统：GN + Ninja
- 目标平台：OpenHarmonyOS（Linux）
- 安装目录：系统分区和模块目录

## 产物概览

### 主要共享库

| 产物 | 类型 | 安装路径 | 代码位置 |
|------|------|---------|---------|
| `libeventhandler.so` | ohos_shared_library | `system_base_dir`, `updater_base_dir` | frameworks/eventhandler/BUILD.gn:29 |
| `libeventhandler_native.so` | ohos_shared_library | 默认模块路径 | frameworks/native/BUILD.gn:17 |
| `libemitter.so` | ohos_shared_library | `module/events/` | frameworks/napi/BUILD.gn:60 |
| `libemitter_interops.so` | ohos_shared_library | 默认模块路径 | frameworks/napi/BUILD.gn:67 |
| `libeventEmitter.so` | ohos_shared_library | 默认模块路径 | frameworks/emitter/BUILD.gn:19 |
| `libcj_emitter_ffi.so` | ohos_shared_library | 默认模块路径 | frameworks/cj/BUILD.gn:17 |

### ArkUI 字节码

| 产物 | 类型 | 安装路径 | 代码位置 |
|------|------|---------|---------|
| `event_emitter_abc.abc` | generate_static_abc | `/system/framework/` | frameworks/emitter/BUILD.gn:64 |

### 配置文件

| 产物 | 类型 | 安装路径 | 代码位置 |
|------|------|---------|---------|
| `event_emitter_abc.abc` | ohos_prebuilt_etc | `framework/` | frameworks/emitter/BUILD.gn:71 |

## 详细产物说明

### libeventhandler.so

**目标**: `//base/notification/eventhandler/frameworks/eventhandler:libeventhandler`

**代码证据**：`frameworks/eventhandler/BUILD.gn:29`

**安装路径**：
- `system_base_dir` - 系统基础目录（如 `/system/`）
- `updater_base_dir` - 更新基础目录（如 `/system/`）

**依赖**：
- 外部：`c_utils:utils`, `hilog:libhilog`, `hitrace:libhitracechain`, `init:libbegetutil`
- 条件：`ipc:ipc_single`, `resource_schedule_service:ressched_client`, `hichecker:libhichecker`, `ffrt:libffrt`

**运行时加载**：
- 被 `libeventhandler_native.so` 动态加载
- 被 `libemitter.so` 动态加载
- 被 `libemitter_interops.so` 动态加载
- 被 `libcj_emitter_ffi.so` 动态加载

**代码证据**：
- `eventhandler_native.so` 依赖：`native/BUILD.gn:34`
- `emitter_interops.so` 依赖：`napi/BUILD.gn:92`
- `eventEmitter.so` 依赖：`emitter/BUILD.gn:47`
- `cj_emitter_ffi.so` 依赖：`cj/BUILD.gn:40`

### libeventhandler_native.so

**目标**: `//base/notification/eventhandler/frameworks/native:eventhandler_native`

**代码证据**：`frameworks/native/BUILD.gn:17`

**安装路径**：默认模块路径（如 `/usr/lib/` 或 `/system/lib/`）

**依赖**：
- 内部：`libeventhandler.so`
- 外部：`c_utils:utils`, `hilog:libhilog`

**对外接口**：
- C 函数导出：`GetEventRunnerNativeObjForThread()`, `CreateEventRunnerNativeObj()`, `EventRunnerRun()`, `EventRunnerStop()`, `EventRunnerAddFileDescriptorListener()`, `EventRunnerRemoveFileDescriptorListener()`

**代码证据**：
- 接口定义：`interfaces/kits/native/native_interface_eventhandler.h:37`
- 函数实现：`frameworks/native/src/native_interface_eventhandler.cpp`

### libemitter.so

**目标**: `//base/notification/eventhandler/frameworks/napi:emitter`

**代码证据**：`frameworks/napi/BUILD.gn:40`

**安装路径**：`module/events/`（相对于模块安装目录）

**依赖**：
- 内部：`emitter_interops.so`
- 外部：`c_utils:utils`, `napi:ace_napi`

**功能**：
- N-API 模块注册：`events.emitter`
- 导出函数：`Init()` → `EmitterInit()`

**代码证据**：
- 模块注册：`napi/src/init.cpp:22-26`

### libemitter_interops.so

**目标**: `//base/notification/eventhandler/frameworks/napi:emitter_interops`

**代码证据**：`frameworks/napi/BUILD.gn:67`

**安装路径**：默认模块路径

**依赖**：
- 内部：`libeventhandler.so`
- 外部：`c_utils:utils`, `hilog:libhilog`, `napi:ace_napi`

**功能**：
- N-API 实现：`JS_On()`, `JS_Once()`, `JS_Off()`, `JS_Emit()`, `JS_GetListenerCount()`
- 增强功能注册：`EmitterEnhancedApiRegister`

**代码证据**：
- N-API 导出：`napi/src/events_emitter.cpp:406-411`
- 增强注册：`napi/src/interops.cpp:20-45`

### libeventEmitter.so

**目标**: `//base/notification/eventhandler/frameworks/emitter:eventEmitter`

**代码证据**：`frameworks/emitter/BUILD.gn:19`

**安装路径**：默认模块路径

**依赖**：
- 内部：`libeventhandler.so`
- 内部：`emitter_interops.so`
- 外部：`c_utils:utils`, `hilog:libhilog`, `napi:ace_napi`, `runtime_core:ani`, `runtime_core:ani_helpers`

**功能**：
- ANI Emitter 实现（Ark Native Interface）
- N-API Emitter 实现
- 序列化支持（N-API 和 ANI）
- 异步回调管理器（N-API 和 ANI）

**源文件**（共 8 个）：

| 源文件 | 模块 | 说明 |
|-------|------|------|
| `ani_emitter.cpp` | ANI | ANI Emitter 实现 |
| `ani_serialize.cpp` | ANI | ANI 序列化 |
| `ani_async_callback_manager.cpp` | base | ANI 回调管理器 |
| `ani_deserialize.cpp` | base | ANI 反序列化 |
| `async_callback_manager.cpp` | base | N-API 回调管理器 |
| `napi_async_callback_manager.cpp` | base | N-API 回调管理器 |
| `napi_emitter.cpp` | napi | N-API Emitter 实现 |
| `napi_serialize.cpp` | napi | N-API 序列化 |

**代码证据**：`emitter/BUILD.gn:36-45`

### event_emitter_abc.abc

**目标**: `//base/notification/eventhandler/frameworks/emitter:event_emitter_abc`

**代码证据**：`frameworks/emitter/BUILD.gn:64`

**安装路径**：`/system/framework/event_emitter_abc.abc`

**类型**：`generate_static_abc`（静态字节码生成）

**源文件**：
- `@ohos.events.emitter.ets`

**代码证据**：`emitter/BUILD.gn:66`

**安装目标**：
- 通过 `event_emitter_abc_etc` 安装到 `/system/framework/` 目录

**代码证据**：`emitter/BUILD.gn:68`

### libcj_emitter_ffi.so

**目标**: `//base/notification/eventhandler/frameworks/cj:cj_emitter_ffi`

**代码证据**：`frameworks/cj/BUILD.gn:17`

**安装路径**：默认模块路径

**依赖**：
- 内部：`libeventhandler.so`
- 外部：`napi:cj_bind_ffi`

**功能**：
- CJ FFI 接口导出：`CJ_OnWithId()`, `CJ_OnWithStringId()`, `CJ_EmitWithId()`, 等等

**编译条件**：
- 正常编译：`!ohos_indep_compiler_enable && !build_ohos_sdk`
- SDK 构建：`ohos_indep_compiler_enable || build_ohos_sdk`

**代码证据**：`cj/BUILD.gn:35-49`

**源文件**（SDK 构建 Mock）：
- `emitter_mock.cpp`

**代码证据**：`cj/BUILD.gn:48`

## 运行时加载关系

### 库加载顺序

```
应用启动
  ↓
libemitter.so (N-API 模块)
  ↓ 加载
libeventhandler.so (核心库)
  ↓ 加载
libemitter_interops.so (互操作层)
  ↓ 加载
[可选] libcj_emitter_ffi.so (CJ FFI)
  ↓ 加载
[可选] libeventhandler_native.so (Native API)
```

### 动态链接依赖

| 库 | 依赖库 | 加载时机 |
|-----|-------|---------|
| `libemitter.so` | `libemitter_interops.so`, `libeventhandler.so` | 运行时 |
| `libemitter_interops.so` | `libeventhandler.so` | 运行时 |
| `libeventEmitter.so` | `libeventhandler.so`, `libemitter_interops.so` | 运行时 |
| `libcj_emitter_ffi.so` | `libeventhandler.so` | 运行时 |
| `libeventhandler_native.so` | `libeventhandler.so` | 运行时 |

**代码证据**：
- 各 BUILD.gn 的 `deps` 字段

### 模块加载器

**N-API 模块**：`events.emitter`
- 加载：`libemitter.so`
- 注册点：`napi_module_register(&_module)`
- 初始化函数：`Init()`

**代码证据**：`frameworks/napi/src/init.cpp:22-42`

### 版本控制

**符号版本脚本**：`libemitter.map`

**代码证据**：`frameworks/napi/libemitter.map`

**作用**：
- 控制导出符号
- 版本化 ABI
- 向后兼容

## 安装目录结构

### 系统分区

```
/system/
├── lib/
│   └── libeventhandler.so        (system_base_dir)
├── framework/
│   └── event_emitter_abc.abc     (ArkUI 字节码)
└── ...

/usr/lib/ 或 /system/lib/
├── libemitter.so                   (module/events/)
├── libemitter_interops.so
├── libeventEmitter.so
└── libcj_emitter_ffi.so
```

### 模块目录

```
/modules/
└── events/
    └── libemitter.so              (relative_install_dir = "module/events")
```

**代码证据**：
- `relative_install_dir`: `module/events` - `frameworks/napi/BUILD.gn:62`

## 构建验证

### 产物验证

#### 检查符号

```bash
# 查看 libeventhandler.so 导出符号
readelf -sW libeventhandler.so | grep "EventHandler\|EventRunner\|InnerEvent"
```

#### 检查依赖

```bash
# 查看动态库依赖
readelf -d libeventhandler.so | grep NEEDED
```

#### 检查安装

```bash
# 验证文件是否安装到正确路径
ls -l /system/lib/libeventhandler.so
ls -l /system/framework/event_emitter_abc.abc
```

### 运行时调试

#### 启用详细日志

```bash
# 设置 HiLog 日志级别
hdc shell param set debug.hilog.log.on true
hdc shell param set debug.hilog.tag on AppExecFwk/EH:V
```

#### 查看 EventRunner 状态

```bash
# 查看 EventRunner Dump 信息
hdc shell "hilog -T AppExecFwk/EH"
```

## 构建大小

### 预估大小

| 产物 | 预估大小 |
|------|---------|
| `libeventhandler.so` | ~500 KB（来自 bundle.json） |
| `libeventhandler_native.so` | ~50 KB |
| `libemitter.so` | ~100 KB |
| `libemitter_interops.so` | ~150 KB |
| `libeventEmitter.so` | ~200 KB |
| `event_emitter_abc.abc` | ~50 KB |

**代码证据**：
- `bundle.json:24-25`（ROM/RAM 估算）

## 清理与重建

### 清理产物

```bash
# 清理构建产物
rm -rf out/
rm -rf build/

# 清理安装的库（需要 root 权限）
hdc shell rm /system/lib/libeventhandler.so
```

### 增量构建

```bash
# 仅构建变更的目标
gn gen out/default
ninja -C out/default libeventhandler
```

## 相关跳转

- [GN 构建目标](06_GN_Targets.md) - 详细的 targets 定义
- [常见问题](09_Troubleshooting.md) - 构建和调试问题
- [项目概览](01_Overview.md) - 编译特性说明

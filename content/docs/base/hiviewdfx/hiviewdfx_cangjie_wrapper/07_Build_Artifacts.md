# 编译产物

## 概述

本文档描述 hiviewdfx_cangjie_wrapper 的编译产物，包括输出文件、安装路径和运行时加载关系。

---

## 产物清单

### 按模块分类

| 模块 | 产物类型 | 产物名 | 说明 |
|------|---------|--------|------|
| kit.PerformanceAnalysisKit | .so | libmodule.z.so | Kit 共享库 |
| ohos.hilog | .so | libmodule.z.so | HiLog 共享库 |
| ohos.hi_trace_meter | .so | libmodule.z.so | HiTraceMeter 共享库 |
| ohos.hiviewdfx.hi_app_event | .so | libmodule.z.so | HiAppEvent 共享库 |
| ohos.hiviewdfx | .so | libmodule.z.so | DFX 聚合库 |

**说明**: 所有 Cangjie 共享库均输出为 `libmodule.z.so`

---

## 输出路径

### 标准输出路径

```
out/{product}/libs/
├── libmodule.z.so                    # kit.PerformanceAnalysisKit
├── libmodule.z.so                    # ohos.hilog
├── libmodule.z.so                    # ohos.hi_trace_meter
└── libmodule.z.so                    # ohos.hiviewdfx.hi_app_event
```

### SDK 输出路径

通过 `copy_sdk_hiviewdfx_cangjie_libs` 任务复制到 SDK 目录：

```
sdk/{api_version}/
├── native/
│   └── libs/
│       └── libmodule.z.so            # Kit 产物
└── cangjie/
    └── api/
        └── kit/PerformanceAnalysisKit/  # Cangjie API 源码
```

**数据来源**: `BUILD.gn:26-29`

---

## 安装路径

### 系统安装路径

```
/system/lib/module/
└── libmodule.z.so                    # 所有模块共享库

/system/lib/module/ohos/
└── libmodule.z.so                    # 各模块独立共享库
```

### SDK 安装路径

```
{SDK_PATH}/native/libs/
└── libmodule.z.so                    # SDK 产物

{SDK_PATH}/cangjie/api/
└── kit/PerformanceAnalysisKit/       # Cangjie API
    ├── index.cj
    ├── hilog/
    ├── hi_trace_meter/
    └── hiviewdfx/
        └── hi_app_event/
```

---

## 运行时加载关系

### 依赖关系图

```
Cangjie Application
    │
    ├── load --> libmodule.z.so (kit.PerformanceAnalysisKit)
    │
    └── load --> libmodule.z.so (ohos.hilog)
    │
    └── load --> libmodule.z.so (ohos.hiviewdfx.hi_app_event)
    │
    └── load --> libmodule.z.so (ohos.hi_trace_meter)
                │
                ├── load --> libhilog.so (Native)
                ├── load --> libcj_hiappevent_ffi.so
                └── load --> libcj_hitracemeter_ffi.so
```

### 动态依赖

使用 `ldd` 命令可查看共享库的动态依赖：

```bash
ldd libmodule.z.so
```

典型依赖（以 ohos.hilog 为例）：

```
libc.so.6                    # C 标准库
libhilog.so                  # HiLog Native 库
libmodule.z.so               # Cangjie 运行时
libCJRuntime.so              # Cangjie 运行时库
```

---

## 产物大小

### ROM 占用

| 模块 | 大小 |
|------|------|
| hiviewdfx_cangjie_wrapper (总计) | 400 KB |

**数据来源**: `bundle.json:20`

### RAM 占用

| 模块 | 大小 |
|------|------|
| hiviewdfx_cangjie_wrapper (总计) | 420 KB |

**数据来源**: `bundle.json:21`

---

## 版本兼容性

### API Level 要求

| 产物 | 最低 API Level | 说明 |
|------|---------------|------|
| 所有 Cangjie 模块 | 22 | API Level 22+ |

**数据来源**: 各 `.cj` 文件中的 `@!APILevel[since: "22"]` 注解

### 系统能力要求

| 产物 | 所需 syscap |
|------|------------|
| kit.PerformanceAnalysisKit | SystemCapability.HiviewDFX.* |
| ohos.hilog | SystemCapability.HiviewDFX.HiLog |
| ohos.hi_trace_meter | SystemCapability.HiviewDFX.HiTrace |
| ohos.hiviewdfx.hi_app_event | SystemCapability.HiviewDFX.HiAppEvent |

---

## 产物验证

### 构建验证

构建完成后，使用以下命令验证产物：

```bash
# 1. 检查产物是否存在
ls -la out/{product}/libs/libmodule.z.so

# 2. 检查动态依赖
ldd out/{product}/libs/libmodule.z.so

# 3. 检查符号表
nm -D out/{product}/libs/libmodule.z.so | grep -E "(HiLog|HiAppEvent|HiTrace)"
```

### 运行验证

```cj
// Cangjie 应用验证
import kit.PerformanceAnalysisKit

// HiLog 验证
import ohos.hilog.{Hilog, LogLevel}
Hilog.info(0xD002800, "Test", "HiLog loaded successfully", [])

// HiTraceMeter 验证
import ohos.hi_trace_meter.HiTraceMeter
HiTraceMeter.startTrace("test", 1)
HiTraceMeter.finishTrace("test", 1)

// HiAppEvent 验证
import ohos.hiviewdfx.hi_app_event.{HiAppEvent, AppEventInfo, EventType}
HiAppEvent.configure( ConfigOption() )
```

---

## 常见问题

### 问题1: 产物找不到

**现象**: 运行时报 `dlopen failed: library "libmodule.z.so" not found`

**解决**:
```bash
# 1. 检查产物是否构建
ls out/{product}/libs/

# 2. 检查是否正确安装
ls /system/lib/module/

# 3. 检查库搜索路径
export LD_LIBRARY_PATH=/system/lib/module:$LD_LIBRARY_PATH
```

### 问题2: 依赖缺失

**现象**: 运行时报 `undefined symbol` 或 `dependency not found`

**解决**:
```bash
# 1. 检查完整依赖链
ldd out/{product}/libs/libmodule.z.so

# 2. 确认 Native 库存在
ls /system/lib/libhilog.so
ls /system/lib/libcj_hiappevent_ffi.so

# 3. 检查系统能力
dumpsys window | grep mSyscap
```

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [06_GN_Build.md](06_GN_Build.md) | 构建系统 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题 |
| [08_Security_Review.md](08_Security_Review.md) | 安全评审 |

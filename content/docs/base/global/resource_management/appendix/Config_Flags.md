# 配置选项

## 目的

本文档汇总 OpenHarmony 资源管理组件的关键配置选项、宏定义和 feature flags。

## 适用范围

本文档覆盖构建配置、运行时配置和平台适配配置。

## 关键结论

| 类别 | 选项数量 | 主要用途 |
|------|----------|----------|
| GN 构建选项 | 1 | ICU 支持 |
| 平台检测变量 | 3 | 平台适配 |
| 宏定义 | N/A | 编译时配置 |

## GN 构建选项

### resource_management_support_icu

**类型**: `bool`

**默认值**: `true`

**说明**: 是否启用 ICU (International Components for Unicode) 国际化库支持

**影响**:
- `true`: 使用 ICU 进行语言匹配和处理
- `false`: 使用轻量级语言匹配实现

**使用方法**:

```bash
# 在 args.gn 中设置
resource_management_support_icu = false
```

或在命令行中设置：

```bash
gn gen out/default --args="resource_management_support_icu=false"
```

**影响范围**:
- `frameworks/resmgr/BUILD.gn` - ICU 依赖
- `frameworks/resmgr/src/locale_matcher.cpp` - 语言匹配实现
- ROM/RAM 占用

**证据**: `resmgr.gni:27`

---

## 平台检测变量

### is_mingw

**类型**: `bool`

**条件**: `"${current_os}_${current_cpu}" == "mingw_x86_64"`

**说明**: 是否为 Windows MinGW 平台

**影响**:
- `true`: 构建 `global_resmgr_win`
- `false`: 不构建 Windows 特定目标

**使用场景**: IDE 预览环境

**证据**: `resmgr.gni:15`

---

### is_linux

**类型**: `bool`

**条件**: `"${current_os}_${current_cpu}" == "linux_x64"`

**说明**: 是否为 Linux 平台

**影响**:
- `true`: 构建 `global_resmgr_linux`
- `false`: 不构建 Linux 特定目标

**使用场景**: IDE 预览环境

**证据**: `resmgr.gni:18`

---

### is_mac

**类型**: `bool`

**条件**:
```gn
"${current_os}_${current_cpu}" == "mac_x64" ||
"${current_os}_${host_cpu}" == "mac_arm64"
```

**说明**: 是否为 macOS 平台

**影响**:
- `true`: 构建 `global_resmgr_mac`
- `false`: 不构建 macOS 特定目标

**使用场景**: IDE 预览环境

**证据**: `resmgr.gni:21-22`

---

## 运行时配置

### 资源配置

**类型**: `ResConfig` 类

**说明**: 运行时资源配置，包括语言、方向、设备类型等

**可配置项**:

| 配置项 | 类型 | 说明 |
|--------|------|------|
| Locale | `std::string` | 语言和区域 (如 "zh-CN") |
| Direction | `enum` | 方向 (VERTICAL/HORIZONTAL) |
| DeviceType | `enum` | 设备类型 (PHONE/TABLET/TV/etc.) |
| ScreenDensity | `enum` | 屏幕密度 (SDPI/MDPI/HDPI/XHDPI/etc.) |
| ColorMode | `enum` | 颜色模式 (DARK/LIGHT) |
| MCC | `int` | 移动国家代码 |
| MNC | `int` | 移动网络代码 |

**使用方法**:

```cpp
ResConfig config;
config.SetLocale("zh-CN");
config.SetDirection(DIRECTION_VERTICAL);
config.SetDeviceType(DEVICE_TYPE_PHONE);
config.SetScreenDensity(SCREEN_DENSITY_XHDPI);
config.SetColorMode(COLOR_MODE_LIGHT);

resMgr->UpdateConfiguration(config);
```

**证据**: `interfaces/inner_api/include/res_config.h`

---

### 系统资源路径

**类型**: `std::string` 常量

**说明**: 系统资源包路径配置

**路径列表**:

| 路径类型 | 路径 | 用途 |
|----------|------|------|
| 沙箱系统资源 | `/data/storage/el1/bundle/ohos.global.systemres...` | SystemAbility 使用 |
| 非沙箱系统资源 | `/system/app/ohos.global.systemres/SystemResources.hap` | appspawn 使用 |

**证据**: `frameworks/resmgr/src/system_resource_manager.cpp:25-41`

---

## 编译选项

### 日志级别

**类型**: 编译宏

**说明**: 控制日志输出级别

**可配置值**:
- `DEBUG`: 调试日志
- `INFO`: 信息日志
- `WARNING`: 警告日志
- `ERROR`: 错误日志

**使用方法**:

在代码中设置：

```cpp
HiLog::SetLogLevel(HiLog::LOG_DEBUG);
```

**证据**: `frameworks/resmgr/include/hilog_wrapper.h`

---

### 调试符号

**类型**: 构建类型

**说明**: 是否包含调试符号

**可配置值**:
- `debug`: 包含调试符号
- `release`: 优化构建，去除调试符号

**使用方法**:

```bash
# Debug 构建
gn gen out/debug --args="is_debug=true"

# Release 构建
gn gen out/release --args="is_debug=false"
```

**证据**: BUILD.gn 配置

---

## 系统事件配置

### hisysevent.yaml

**路径**: `/hisysevent.yaml`

**说明**: 系统事件上报配置

**内容示例**:

```yaml
- name: RESOURCE_LOAD
  level: MINOR
- name: RESOURCE_LOAD_FAIL
  level: CRITICAL
- name: RESOURCE_NOT_FOUND
  level: MINOR
```

**事件类型**:
- `RESOURCE_LOAD`: 资源加载成功
- `RESOURCE_LOAD_FAIL`: 资源加载失败
- `RESOURCE_NOT_FOUND`: 资源未找到

**证据**: `hisysevent.yaml`

---

## 常见配置场景

### 场景 1: 禁用 ICU 支持以减少 ROM 占用

**目标**: 减少 ROM 占用，接受语言匹配功能受限

**配置**:

```bash
# args.gn
resource_management_support_icu = false
```

**影响**:
- ROM: 减少约 2-3MB (ICU 库)
- RAM: 减少约 1-2MB
- 功能: 语言匹配可能不准确

**证据**: `resmgr.gni:27`

---

### 场景 2: 启用详细日志用于调试

**目标**: 调试资源加载问题

**配置**:

```cpp
// 在应用启动时
HiLog::SetLogLevel(HiLog::LOG_DEBUG);
```

或修改 HiLog 包装器：

```cpp
// frameworks/resmgr/include/hilog_wrapper.h
#define RESMGR_TAG "ResourceMgr"
#define HILOG_DEBUG(...) HiLog::Debug(RESMGR_TAG, __VA_ARGS__)
```

**证据**: `frameworks/resmgr/include/hilog_wrapper.h`

---

### 场景 3: 适配特定平台

**目标**: 在特定平台上构建资源管理组件

**配置**:

```bash
# Windows 平台
gn gen out/windows --args="current_os=\"mingw\" current_cpu=\"x86_64\""

# Linux 平台
gn gen out/linux --args="current_os=\"linux\" current_cpu=\"x64\""

# macOS 平台
gn gen out/mac --args="current_os=\"mac\" current_cpu=\"x86_64\""
```

**证据**: `resmgr.gni:15-23`

---

## 配置验证

### 检查 ICU 支持

```bash
# 检查构建日志
grep "icu" build.log

# 或检查二进制文件
nm /system/lib64/libglobal_resmgr.so | grep icu
```

**证据**: `frameworks/resmgr/BUILD.gn` deps

---

### 检查平台适配

```bash
# 检查当前平台
echo ${current_os}_${current_cpu}

# 检查构建产物
ls -l out/default/lib*/libglobal_resmgr.so
```

**证据**: `resmgr.gni:15-23`

---

## 相关文档

- [GN Targets](06_GNTargets.md) - 构建系统和依赖关系
- [编译产物](07_BuildArtifacts.md) - 编译产物和部署
- [常见问题](09_FAQ.md) - 问题排查指南

---

**生成时间**: 2026-02-06
**证据来源**: resmgr.gni, hisysevent.yaml, BUILD.gn

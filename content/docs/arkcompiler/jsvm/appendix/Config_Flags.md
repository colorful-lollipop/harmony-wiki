# 配置 Flags

> 关键宏/feature flags

## 目的与适用范围

### 目的
本文档描述 JSVM 的关键编译选项和特性开关。

### 适用范围
- 需要定制构建的开发者
- 需要了解特性的开发者
- 需要进行性能调优的开发者

---

## 编译参数（declare_args）

### enable_inspector

**类型**: boolean

**默认值**: true

**用途**: 启用/禁用 Inspector 调试功能。

**定义位置**: `jsvm.gni:32`

**影响**:
- 启用: 编译 Inspector 模块，提供调试支持
- 禁用: 减少 ROM/RAM 占用，但无法调试

**证据位置**: `jsvm.gni:32`

### enable_debug

**类型**: boolean

**默认值**: false

**用途**: 启用/禁用调试功能。

**定义位置**: `jsvm.gni:31`

**影响**:
- 启用: 增加调试日志和检查
- 禁用: 性能更好

**证据位置**: `jsvm.gni:31`

### jsvm_shared_libuv

**类型**: boolean

**默认值**: true

**用途**: 是否共享 libuv 库。

**定义位置**: `jsvm.gni:30`

**影响**:
- true: 共享 libuv，减少 ROM 占用
- false: 静态链接 libuv，增加 ROM 占用但更独立

**证据位置**: `jsvm.gni:30`

### use_platform_ohos

**类型**: boolean

**默认值**: true

**用途**: 是否使用 OpenHarmony 平台实现。

**定义位置**: `jsvm.gni:33`

**影响**:
- true: 使用 OpenHarmony 平台实现
- false: 需要提供其他平台实现

**证据位置**: `jsvm.gni:33`

### support_hwasan

**类型**: boolean

**默认值**: true

**用途**: 是否支持 Hardware-Assisted Address Sanitizer。

**定义位置**: `jsvm.gni:34`

**影响**:
- true: 支持 HWASAN，可以检测内存错误
- false: 不支持 HWASAN

**证据位置**: `jsvm.gni:34`

---

## 运行时配置

### JIT 配置

**配置文件**: `jit_enable_list_appid.conf`

**用途**: 配置启用 JIT 的应用 ID 列表。

**位置**: `/system/etc/jsvm/jit_enable_list_appid.conf`

**格式**: 每行一个应用 ID

**证据位置**: `BUILD.gn:18-23` - jit_enable_list_appid target

---

## 宏定义

### JSVM_VERSION

**定义**: `#define JSVM_VERSION 8`

**用途**: 控制使用的 JSVM-API 版本。

**位置**: `interface/kits/jsvm.h:61`

**影响**:
- 控制 API 的行为
- 影响向后兼容性

**证据位置**: `interface/kits/jsvm.h:51-63`

### JSVM_EXTERN

**定义**: `#define JSVM_EXTERN __attribute__((visibility("default")))`（非 Windows）

**用途**: 定义导出符号的可见性。

**位置**: `interface/kits/jsvm.h:67-80`

**影响**:
- Windows: `__declspec(dllexport)`
- 其他: `__attribute__((visibility("default")))`

**证据位置**: `interface/kits/jsvm.h:67-80`

---

## 相关链接

- [GN Targets 与编译产物](./07_GN_Targets.md) - 构建系统
- [目录结构与模块职责](./02_Directory_Structure.md) - jsvm.gni 配置

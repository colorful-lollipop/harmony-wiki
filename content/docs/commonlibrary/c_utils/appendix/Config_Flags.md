# 配置与宏

## 目的

本文档汇总 c_utils 的所有编译配置选项和宏定义。

---

## Feature Flags

### 声明位置

`base/BUILD.gn:16-26`

```gn
declare_args() {
  c_utils_feature_coverage = false
  c_utils_debug_refbase = false
  c_utils_track_all = false
  c_utils_print_track_at_once = false
  c_utils_debug_log_enabled = false
  c_utils_feature_intsan = true
  c_utils_parcel_object_check = true
  c_utils_feature_enable_pgo = false
  c_utils_feature_pgo_path = ""
}
```

### 详细说明

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `c_utils_feature_coverage` | bool | false | 启用代码覆盖率统计（gcov）|
| `c_utils_debug_refbase` | bool | false | 启用 RefBase 调试跟踪 |
| `c_utils_track_all` | bool | false | 跟踪所有 RefBase 对象生命周期 |
| `c_utils_print_track_at_once` | bool | false | 实时打印跟踪信息 |
| `c_utils_debug_log_enabled` | bool | false | 启用 DEBUG_UTILS 日志 |
| `c_utils_feature_intsan` | bool | true | 启用整数溢出检测（IntegerSanitizer）|
| `c_utils_parcel_object_check` | bool | true | 启用 Parcel 对象检查 |
| `c_utils_feature_enable_pgo` | bool | false | 启用 PGO 优化 |
| `c_utils_feature_pgo_path` | string | "" | PGO profile 数据路径 |

### 使用示例

```bash
# 启用 RefBase 调试
./build.sh --product-name rk3568 --build-target c_utils \
    --gn-args "c_utils_debug_refbase=true"

# 启用覆盖率统计
./build.sh --product-name rk3568 --build-target c_utils \
    --gn-args "c_utils_feature_coverage=true"

# 多个参数
./build.sh --product-name rk3568 --build-target c_utils \
    --gn-args "c_utils_debug_refbase=true c_utils_track_all=true"
```

---

## 宏定义

### 平台宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `OHOS_PLATFORM` | BUILD.gn:44 | OpenHarmony 平台 |
| `IOS_PLATFORM` | BUILD.gn:32 | iOS 平台 |
| `WINDOWS_PLATFORM` | BUILD.gn:35 | Windows 平台 |
| `MAC_PLATFORM` | BUILD.gn:38 | macOS 平台 |
| `EMULATOR_PLATFORM` | BUILD.gn:41 | 模拟器平台 |
| `ANDROID_PLATFORM` | 推断 | Android 平台 |

### 调试宏

| 宏 | 定义位置 | 说明 | 依赖 |
|----|----------|------|------|
| `DEBUG_REFBASE` | debug_refbase config | 启用 RefBase 调试 | c_utils_debug_refbase |
| `TRACK_ALL` | track_all config | 跟踪所有对象 | c_utils_track_all |
| `PRINT_TRACK_AT_ONCE` | print_track_at_once config | 实时打印 | c_utils_print_track_at_once |
| `DEBUG_UTILS` | debug_log_enabled config | 启用调试日志 | c_utils_debug_log_enabled |
| `PARCEL_OBJECT_CHECK` | parcel_object_check config | Parcel对象检查 | c_utils_parcel_object_check |

### 功能宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `CONFIG_HILOG` | BUILD.gn | 启用 hilog 日志（非Android/iOS）|
| `UTILS_CXX_RUST` | BUILD.gn:315 | 启用 Rust FFI 支持 |

---

## Config 定义

### utils_config

```gn
config("utils_config") {
  include_dirs = [ "include" ]
  defines = []
  # 根据平台添加相应宏
}
```

**效果**: 所有依赖 c_utils 的模块自动获得 `base/include` 包含路径。

### utils_coverage_config

```gn
config("utils_coverage_config") {
  visibility = [ ":*" ]
  if (c_utils_feature_coverage) {
    cflags = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}
```

**效果**: 启用 gcov 代码覆盖率统计。

### 调试 Config

```gn
config("debug_refbase") {
  visibility = [ ":*" ]
  defines = [ "DEBUG_REFBASE" ]
}

config("track_all") {
  visibility = [ ":*" ]
  defines = [ "TRACK_ALL" ]
}

config("print_track_at_once") {
  visibility = [ ":*" ]
  defines = [ "PRINT_TRACK_AT_ONCE" ]
}

config("debug_log_enabled") {
  visibility = [ ":*" ]
  defines = [ "DEBUG_UTILS" ]
}

config("parcel_object_check") {
  visibility = [ ":*" ]
  defines = [ "PARCEL_OBJECT_CHECK" ]
}
```

---

## 安全编译选项

### 整数溢出检测

```gn
if (c_utils_feature_intsan) {
  sanitize = {
    integer_overflow = true
  }
  branch_protector_ret = "pac_ret"
}
```

**效果**:
- 检测整数溢出
- ARM64 启用 PAC-RET（返回地址保护）

### PGO 优化

```gn
if (c_utils_feature_enable_pgo) {
  cflags = [
    "-fprofile-use=${c_utils_feature_pgo_path}/libutils.profdata",
    "-Wno-error=backend-plugin",
    "-Wno-profile-instr-out-of-date",
    "-Wno-profile-instr-unprofiled",
  ]
}
```

**效果**: 使用 Profile Guided Optimization 提升性能。

### 链接器选项

```gn
ldflags = [ "-Wl,-Bsymbolic" ]

if (c_utils_feature_enable_pgo && target_cpu == "arm64") {
  ldflags += [ "-Wl,--aarch64-inline-plt" ]
}
```

**效果**:
- `-Bsymbolic`: 优先绑定本地符号，加速启动
- `--aarch64-inline-plt`: 内联 PLT 条目，减少跳转

---

## 常量定义

### 文件操作限制

```cpp
// base/src/file_ex.cpp:24
const int MAX_FILE_LENGTH = 32 * 1024 * 1024;  // 32MB
```

### Parcel 限制

```cpp
// base/src/parcel.cpp:44-45
static const size_t DEFAULT_CPACITY = 204800;      // 200KB 默认容量
static const size_t CAPACITY_THRESHOLD = 4096;     // 4KB 扩容阈值
```

### RefBase 常量

```cpp
// base/include/refbase.h:47
#define INITIAL_PRIMARY_VALUE (1 << 28)  // 初始强引用值
```

### Binder 类型标识

```cpp
// base/src/parcel.cpp:46-47
static const int BINDER_TYPE_HANDLE = 0x73682a85;  // 远程对象句柄
static const int BINDER_TYPE_FD = 0x66642a85;      // 文件描述符
```

---

## 相关跳转

- [GN Targets](../06_GN_Targets.md) - 构建配置
- [安全风险](../08_Security_Review.md) - 安全编译选项

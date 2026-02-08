# OH 构建适配

## 构建系统概述

MindSpore Lite 的 OH 构建是从上游 CMake 迁移到 GN 构建系统的完整适配。

### 构建架构

```
上游构建 (CMake)
       ↓
   GN 迁移层
       ↓
OH 构建系统 (GN)
```

### 迁移策略

| 上游配置 | OH 适配 | 位置 |
|----------|---------|------|
| CMake | GN (BUILD.gn) | `mindspore-src/source/mindspore-lite/BUILD.gn` |
| GLOG | hilog | `src/common/hi_app_event/` |
| Android NDK | OH NDK | `external_deps` 配置 |
| 平台检测 | OH 架构检测 | 条件编译 |

---

## BUILD.gn 结构

### 根级别构建入口

**文件**: `/Volumes/lexar/code/d/work/oh/third_party/mindspore/BUILD.gn`

```gn
import("//build/ohos.gni")

# 主构建目标
ohos_group("mindspore-all") {
  deps = [ "mindspore-src/source/mindspore-lite/:mindspore" ]
}

# MindIR 格式库
group("mindir") {
  deps = [ "mindspore-src/source/mindspore-lite/mindir:mindir_lib" ]
}

# NDK 库
ohos_group("mindspore-ndk") {
  deps = [ "mindspore-src/source/mindspore-lite/:mindspore_ndk" ]
}

# 核心库
ohos_group("mindspore-lib") {
  deps = [ "mindspore-src/source/mindspore-lite/:mindspore_lib" ]
}

# 测试套件
group("mindspore-test") {
  testonly = true
  deps = [
    "test:mindspore_fuzz_test",
    "test:mindspore_system_test",
    "test:mindspore_unit_test",
  ]
}
```

---

## 核心库构建配置

**文件**: `mindspore-src/source/mindspore-lite/BUILD.gn`

### 主构建目标

```gn
ohos_group("mindspore") {
  deps = [
    ":mindspore_lib",
    ":mindspore_ndk",
    ":mindspore_train_lib",
    "mindir:mindir_lib",
    "src/litert/js_api:mindsporelite_napi",
    "src/litert/taihe/ability_delegator:mindspore_taihe"
  ]
}
```

### 推理库 (mindspore_lib)

```gn
ohos_shared_library("mindspore_lib") {
  branch_protector_ret = "pac_ret"  # PAC-RET 分支保护
  output_name = "libmindspore-lite"
  output_extension = "so"
  
  deps = [
    "src/litert/kernel/cpu/nnacl_c/:nnacl_obj",
    "../mindspore/mindspore/core/mindrt/:mindrt_obj",
    "src/litert/kernel/cpu/:cpu_kernel_obj",
    "src/common/:lite_common_mid_obj",
  ]
  
  sources = all_sources  # 400+ 源文件
  
  defines = [
    "ENABLE_MINDRT",
    "MS_COMPILE_OHOS",
    "PRIMITIVE_WRITEABLE",
    "RUNTIME_PASS_CLIP",
    "ENABLE_MULTI_LAYOUT",
    "VERSION_STR=\"2.7.0\"",
    "ENABLE_HI_APP_EVENT",
  ]
  
  configs = [
    ":mindspore_api",
    ":disable_android",
    ":opencl_option",
    ":secure_option",
    ":link_option_lto",
  ]
  
  external_deps = [
    "hilog:libhilog",
    "bounds_checking_function:libsec_shared",
    "flatbuffers:libflatbuffers_static",
    "qos_manager:qos",
    "ipc:ipc_single",
    "qos_manager:concurrent_task_client"
  ]
  
  innerapi_tags = [ "platformsdk" ]
  part_name = "mindspore"
  subsystem_name = "thirdparty"
}
```

### NDK 库 (mindspore_ndk)

```gn
ohos_shared_library("mindspore_ndk") {
  branch_protector_ret = "pac_ret"
  output_name = "libmindspore_lite_ndk"
  output_extension = "so"
  
  deps = [
    ":mindspore_lib",
    ":mindspore_train_lib",
  ]
  
  sources = c_api_sources  # C API 源文件
  
  defines = [
    "MS_COMPILE_OHOS",
    "PRIMITIVE_WRITEABLE",
    "RUNTIME_PASS_CLIP",
    "ENABLE_MULTI_LAYOUT",
    "VERSION_STR=\"2.7.0\"",
    "ENABLE_HI_APP_EVENT"
  ]
  
  external_deps = [
    "flatbuffers:libflatbuffers_static",
    "hilog:libhilog"
  ]
  
  innerapi_tags = [ "ndk", "llndk" ]
  part_name = "mindspore"
  subsystem_name = "thirdparty"
}
```

---

## OH 特定配置

### Android 标志禁用

```gn
config("disable_android") {
  cflags = [
    "-UANDROID",
    "-U__ANDROID__",
    "-U__ANDROID_API__",
  ]
  cflags_cc = [
    "-UANDROID",
    "-U__ANDROID__",
    "-U__ANDROID_API__",
  ]
  ldflags = [
    "-Wl,--no-as-needed",
  ]
}
```

**用途**: 确保 MindSpore 在 OH 环境下不使用 Android 特定代码路径

---

### 安全加固配置

```gn
config("secure_option") {
  cflags = [
    "-fstack-protector-all",
    "-D_FORTIFY_SOURCE=2",
  ]
}

config("link_option_lto") {
  ldflags = ["-Wl,--lto-O0"]
}
```

**用途**:
- Stack Protector: 防止栈溢出攻击
- FORTIFY_SOURCE: 运行时缓冲区溢出检测
- ThinLTO: 链接时优化，减少二进制体积

---

### RTTI 启用

```gn
remove_configs = [ "//build/config/compiler:no_rtti" ]
```

**用途**: MindSpore 需要运行时类型信息 (RTTI) 进行多态操作

---

## 架构特定配置

### ARM32

```gn
if (target_cpu == "arm") {
  defines += [
    "ENABLE_ARM",
    "ENABLE_ARM32",
    "ENABLE_NEON",
  ]
}
```

### ARM64

```gn
if (target_cpu == "arm64") {
  defines += [
    "ENABLE_ARM",
    "ENABLE_ARM64",
    "ENABLE_NEON",
    "ENABLE_FP16",
    "USE_OPENCL_WRAPPER",
    "MS_OPENCL_PROFILE=false",
    "CL_TARGET_OPENCL_VERSION=200",
    "CL_HPP_TARGET_OPENCL_VERSION=120",
    "CL_HPP_MINIMUM_OPENCL_VERSION=120",
  ]
}
```

---

## 外部依赖

### OH 系统依赖

```gn
external_deps = [
  "hilog:libhilog",                    # OH 日志
  "bounds_checking_function:libsec_shared",  # 安全函数
  "flatbuffers:libflatbuffers_static", # 模型序列化
  "qos_manager:qos",                   # QoS 管理
  "qos_manager:concurrent_task_client", # 并发任务
  "ipc:ipc_single",                    # 进程间通信
  "c_utils:utils",                     # C 工具库
  "hdf_core:libhdi",                   # HDI 框架
  "drivers_interface_nnrt:nnrt_idl_headers",  # NNRT 驱动
  "hiappevent:hiappevent_innerapi",    # 应用事件
  "napi:ace_napi",                     # N-API
  "ability_runtime:abilitykit_native", # 能力框架
  "resource_management:global_resmgr", # 资源管理
]
```

---

## NNRT 条件支持

```gn
SUPPORT_NNRT = false
# currently, only arm/arm64 real machine support nnrt
if ((target_cpu == "arm" || target_cpu == "arm64") && !is_emulator) {
  SUPPORT_NNRT = true
}

if (SUPPORT_NNRT) {
  if (mindspore_feature_nnrt_metagraph) {
    defines += [ "SUPPORT_NNRT_METAGRAPH" ]
    sources += [
      "src/litert/delegate/nnrt/hiai_foundation_wrapper.cc",
      "src/litert/cache_session.cc",
    ]
    print("enabled feature: mindspore_feature_nnrt_metagraph")
  }
  
  sources += [
    "src/litert/delegate/nnrt/checker/primitive_check.cc",
    "src/litert/delegate/nnrt/nnrt_delegate.cc",
    "src/litert/delegate/nnrt/nnrt_model_kernel.cc",
    "src/litert/delegate/nnrt/nnrt_allocator.cc",
    "src/litert/delegate/nnrt/extension_options_parser.cc",
    "src/litert/delegate/nnrt/nnrt_utils.cc",
    "src/litert/delegate/nnrt/nnrt_wrapper.cc",
  ]
  
  deps += [ "mindir:mindir_lib" ]
  defines += [ "SUPPORT_NNRT" ]
}
```

---

## 构建目标清单

| 目标名称 | 类型 | 输出文件 | 描述 |
|----------|------|----------|------|
| `mindspore` | ohos_group | - | 主构建组 |
| `mindspore_lib` | ohos_shared_library | libmindspore-lite.so | 核心推理库 |
| `mindspore_ndk` | ohos_shared_library | libmindspore_lite_ndk.so | NDK API 库 |
| `mindspore_train_lib` | ohos_shared_library | libmindspore-lite-train.so | 训练库 |
| `mindir_lib` | ohos_shared_library | libmindir.so | MindIR 格式库 |
| `mindsporelite_napi` | ohos_shared_library | mindspore_lite_napi.so | N-API 模块 |
| `mindspore_ani_taihe_native` | taihe_shared_library | - | ANI 绑定 |
| `benchmark_bin` | ohos_executable | benchmark | 基准测试工具 |

---

## 编译选项对照表

| 上游 CMake 选项 | OH BUILD.gn 定义 | 默认值 | 说明 |
|-----------------|------------------|--------|------|
| MSLITE_ENABLE_NPU | - | OFF | NPU 支持 (OH 使用 NNRT) |
| MSLITE_ENABLE_TRAIN | SUPPORT_TRAIN | ON | 训练支持 |
| MSLITE_ENABLE_CONVERTER | - | OFF | 模型转换工具 |
| MSLITE_ENABLE_TOOLS | - | OFF | 嵌入式工具 |
| MSLITE_ENABLE_TESTCASES | - | OFF | 测试用例 |
| MSLITE_ENABLE_RUNTIME_GLOG | - | OFF | OH 使用 hilog |
| MSLITE_ENABLE_STRING_KERNEL | - | ON | 字符串内核 |
| MSLITE_ENABLE_CONTROLFLOW | - | ON | 控制流 |
| MSLITE_ENABLE_AUTO_PARALLEL | - | ON | 自动并行 |
| MSLITE_ENABLE_WEIGHT_DECODE | - | ON | 权重解码 |
| MSLITE_ENABLE_CUSTOM_KERNEL | - | ON | 自定义内核 |
| MSLITE_ENABLE_MINDRT | ENABLE_MINDRT | ON | MindRT 运行时 |
| MSLITE_ENABLE_DELEGATE | - | ON | 委托支持 |
| MSLITE_ENABLE_INT8 | - | ON | INT8 量化 |
| MSLITE_ENABLE_FP16 | ENABLE_FP16 | OFF (ARM64 ON) | 半精度 |
| MSLITE_ENABLE_NNRT | SUPPORT_NNRT | OFF (ARM real ON) | NNRT 加速 |

---

## 构建命令

### 完整构建

```bash
./build.sh --product-name <product> --build-target mindspore-all
```

### NDK 构建

```bash
./build.sh --product-name <product> --build-target mindspore-ndk
```

### 测试构建

```bash
./build.sh --product-name <product> --build-target mindspore_test_target
```

### 启用 NNRT

```bash
# 在产品配置中启用
build.py --product-name <product> \
  --build-target mindspore-all \
  --gn-args mindspore_feature_nnrt_metagraph=true
```

---

## 构建产物

### 库文件

```
out/<product>/libs/
├── libmindspore-lite.so          # 核心推理库 (~8MB)
├── libmindspore_lite_ndk.so      # NDK 库 (~2MB)
├── libmindspore-lite-train.so     # 训练库 (~4MB)
├── libmindir.so                   # MindIR 格式库
└── mindspore_lite_napi.so        # N-API 模块
```

### 头文件

```
include/
├── c_api/                         # C API 头文件
│   ├── context_c.h
│   ├── data_type_c.h
│   ├── format_c.h
│   ├── model_c.h
│   ├── status_c.h
│   ├── tensor_c.h
│   └── types_c.h
└── api/                           # C++ API 头文件
```

---

## 故障排查

### 常见构建错误

#### 1. RTTI 相关错误

**错误**: `error: cannot use dynamic_cast with -fno-rtti`

**解决**: 确保移除 `//build/config/compiler:no_rtti` 配置

#### 2. Android 标志冲突

**错误**: `error: undefined reference to 'android_LOG'`

**解决**: 确保使用 `disable_android` 配置禁用 Android 标志

#### 3. NNRT 依赖缺失

**错误**: `error: cannot find -lnnrt`

**解决**: 仅在 ARM/ARM64 真机上启用 NNRT 支持

---

## 性能优化

### 构建优化

| 优化项 | 配置 | 效果 |
|--------|------|------|
| ThinLTO | `-flto=thin` | 二进制体积减少 30% |
| Os 优化 | `-Os` | 平衡大小和性能 |
| PAC-RET | `branch_protector_ret = "pac_ret"` | 安全加固 |

### 运行时优化

| 优化项 | 定义 | 适用场景 |
|--------|------|----------|
| FP16 | `ENABLE_FP16` | ARM64 浮点运算 |
| INT8 | `MSLITE_ENABLE_INT8` | 量化推理 |
| NEON | `ENABLE_NEON` | ARM SIMD 加速 |

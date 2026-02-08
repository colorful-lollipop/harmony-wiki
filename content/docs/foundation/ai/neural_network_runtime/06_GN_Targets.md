# 06. GN 构建目标

## 目的

本文档详细介绍 Neural Network Runtime 的 GN 构建系统，包括所有 targets、依赖关系和关键配置。

## 适用范围

- 系统构建工程师
- 需要修改构建配置的开发者

## BUILD.gn 文件列表

| 路径 | 说明 |
|------|------|
| `BUILD.gn` | 根构建文件 |
| `config/BUILD.gn` | 编译配置定义 |
| `frameworks/native/neural_network_core/BUILD.gn` | Core 库构建 |
| `frameworks/native/neural_network_runtime/BUILD.gn` | Runtime 库构建 |
| `example/drivers/nnrt/v1_0/BUILD.gn` | v1.0 示例入口 |
| `example/drivers/nnrt/v1_0/hdi_cpu_service/BUILD.gn` | v1.0 HDI 服务 |
| `example/drivers/nnrt/v2_0/BUILD.gn` | v2.0 示例入口 |
| `example/drivers/nnrt/v2_0/hdi_cpu_service/BUILD.gn` | v2.0 HDI 服务 |

## 根构建文件

**文件**: `BUILD.gn`

```gn
import("//build/ohos.gni")

group("nnrt_target") {
  deps = [
    "frameworks/native/neural_network_core:libneural_network_core",
    "frameworks/native/neural_network_runtime:libneural_network_runtime",
  ]
}

group("nncore_target") {
  deps = [ "frameworks/native/neural_network_core:libneural_network_core" ]
}

group("nnrt_test_target") {
  testonly = true
  deps = [ "test/unittest:unittest" ]
}

group("nnrt_fuzztest") {
  testonly = true
  deps = [ "test/fuzztest:fuzztest" ]
}
```

**证据**: `BUILD.gn:16-35`

### 根 Targets

| Target | 类型 | 说明 | 依赖 |
|--------|------|------|------|
| `nnrt_target` | group | 完整 NNRT 构建 | libneural_network_core, libneural_network_runtime |
| `nncore_target` | group | 仅 Core 库 | libneural_network_core |
| `nnrt_test_target` | group | 单元测试 | unittest |
| `nnrt_fuzztest` | group | Fuzz 测试 | fuzztest |

## Core 库构建

**文件**: `frameworks/native/neural_network_core/BUILD.gn`

### Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `nnrt_config` | config | - | 编译配置 |
| `nnrt_public_config` | config | - | 公共头文件配置 |
| `libneural_network_core` | ohos_shared_library | libneural_network_core.so | Core 共享库 |

### libneural_network_core 详情

```gn
ohos_shared_library("libneural_network_core") {
  sources = [
    "backend_manager.cpp",
    "backend_registrar.cpp",
    "neural_network_core.cpp",
    "nnrt_client.cpp",
    "tensor_desc.cpp",
    "utils.cpp",
    "validation.cpp",
  ]

  branch_protector_ret = "pac_ret"
  install_images = [ "system", "updater" ]

  configs = [ ":nnrt_config" ]
  public_configs = [
    "//foundation/ai/neural_network_runtime/config:coverage_flags",
    ":nnrt_public_config",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "openssl:libcrypto_shared",
  ]

  metadata = {
    subsystem_name = "ai"
    innerapi_tags = [ "ndk" ]
    part_name = "neural_network_runtime"
  }
}
```

**证据**: `frameworks/native/neural_network_core/BUILD.gn`

### 关键配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| branch_protector_ret | pac_ret | 分支保护 |
| install_images | [system, updater] | 安装镜像 |
| innerapi_tags | [ndk] | NDK 公开 API |
| subsystem_name | ai | 子系统 |
| part_name | neural_network_runtime | 部件名 |

## Runtime 库构建

**文件**: `frameworks/native/neural_network_runtime/BUILD.gn`

### Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `nnrt_config` | config | - | 编译配置和头文件路径 |
| `libneural_network_runtime` | ohos_shared_library | libneural_network_runtime.so | Runtime 共享库 |

### libneural_network_runtime 详情

```gn
ohos_shared_library("libneural_network_runtime") {
  sources = [
    # Core 源文件 (28 个)
    "hdi_device_v1_0.cpp",
    "hdi_device_v2_0.cpp",
    "hdi_device_v2_1.cpp",
    "hdi_prepared_model_v1_0.cpp",
    "hdi_prepared_model_v2_0.cpp",
    "hdi_prepared_model_v2_1.cpp",
    "inner_model.cpp",
    "lite_graph_to_hdi_model_v1_0.cpp",
    "lite_graph_to_hdi_model_v2_0.cpp",
    "lite_graph_to_hdi_model_v2_1.cpp",
    "memory_manager.cpp",
    "neural_network_runtime.cpp",
    "neural_network_runtime_compat.cpp",
    "nn_tensor.cpp",
    "nnbackend.cpp",
    "nncompiled_cache.cpp",
    "nncompiler.cpp",
    "nnexecutor.cpp",
    "nntensor.cpp",
    "ops_builder.cpp",
    "ops_registry.cpp",
    "quant_param.cpp",
    "register_hdi_device_v1_0.cpp",
    "register_hdi_device_v2_0.cpp",
    "register_hdi_device_v2_1.cpp",
    "transform.cpp",
    
    # 算子源文件 (110 个)
    "ops/abs_builder.cpp",
    "ops/add_builder.cpp",
    "ops/avgpool_builder.cpp",
    "ops/batch_to_space_nd_builder.cpp",
    "ops/batchnorm_builder.cpp",
    "ops/bias_add_builder.cpp",
    "ops/cast_builder.cpp",
    "ops/ceil_builder.cpp",
    "ops/clip_builder.cpp",
    "ops/concat_builder.cpp",
    "ops/constant_of_shape_builder.cpp",
    "ops/conv2d_builder.cpp",
    "ops/conv2d_transpose_builder.cpp",
    "ops/cos_builder.cpp",
    "ops/crop_builder.cpp",
    "ops/depth_to_space_builder.cpp",
    "ops/depthwise_conv2d_native_builder.cpp",
    "ops/detection_post_process_builder.cpp",
    "ops/div_builder.cpp",
    "ops/eltwise_builder.cpp",
    "ops/equal_builder.cpp",
    "ops/erf_builder.cpp",
    "ops/exp_builder.cpp",
    "ops/expandims_builder.cpp",
    "ops/fill_builder.cpp",
    "ops/flatten_builder.cpp",
    "ops/floor_builder.cpp",
    "ops/fullconnection_builder.cpp",
    "ops/gather_builder.cpp",
    "ops/gather_nd_builder.cpp",
    "ops/gelu_builder.cpp",
    "ops/greater_builder.cpp",
    "ops/greater_equal_builder.cpp",
    "ops/hard_sigmoid_builder.cpp",
    "ops/hswish_builder.cpp",
    "ops/instance_norm_builder.cpp",
    "ops/l2_normalize_builder.cpp",
    "ops/layernorm_builder.cpp",
    "ops/leaky_relu_builder.cpp",
    "ops/less_builder.cpp",
    "ops/lessequal_builder.cpp",
    "ops/log_builder.cpp",
    "ops/log_softmax_builder.cpp",
    "ops/logical_and_builder.cpp",
    "ops/logical_not_builder.cpp",
    "ops/logical_or_builder.cpp",
    "ops/lrn_builder.cpp",
    "ops/lstm_builder.cpp",
    "ops/matmul_builder.cpp",
    "ops/maximum_builder.cpp",
    "ops/maxpool_builder.cpp",
    "ops/minimum_builder.cpp",
    "ops/mod_builder.cpp",
    "ops/mul_builder.cpp",
    "ops/neg_builder.cpp",
    "ops/notequal_builder.cpp",
    "ops/onehot_builder.cpp",
    "ops/ops_validation.cpp",
    "ops/pad_builder.cpp",
    "ops/pooling_builder.cpp",
    "ops/pow_builder.cpp",
    "ops/prelu_builder.cpp",
    "ops/quant_dtype_cast_builder.cpp",
    "ops/range_builder.cpp",
    "ops/rank_builder.cpp",
    "ops/reciprocal_builder.cpp",
    "ops/reduceL2_builder.cpp",
    "ops/reduceall_builder.cpp",
    "ops/reducemax_builder.cpp",
    "ops/reducemean_builder.cpp",
    "ops/reducemin_builder.cpp",
    "ops/reduceprod_builder.cpp",
    "ops/reducesum_builder.cpp",
    "ops/relu_builder.cpp",
    "ops/relu6_builder.cpp",
    "ops/reshape_builder.cpp",
    "ops/resize_bilinear_builder.cpp",
    "ops/round_builder.cpp",
    "ops/rsqrt_builder.cpp",
    "ops/scale_builder.cpp",
    "ops/scatter_nd_builder.cpp",
    "ops/select_builder.cpp",
    "ops/shape_builder.cpp",
    "ops/sigmoid_builder.cpp",
    "ops/sin_builder.cpp",
    "ops/slice_builder.cpp",
    "ops/softmax_builder.cpp",
    "ops/space_to_batch_nd_builder.cpp",
    "ops/space_to_depth_builder.cpp",
    "ops/sparse_to_dense_builder.cpp",
    "ops/split_builder.cpp",
    "ops/sqrt_builder.cpp",
    "ops/square_builder.cpp",
    "ops/squared_difference_builder.cpp",
    "ops/squeeze_builder.cpp",
    "ops/stack_builder.cpp",
    "ops/strided_slice_builder.cpp",
    "ops/sub_builder.cpp",
    "ops/swish_builder.cpp",
    "ops/tanh_builder.cpp",
    "ops/tile_builder.cpp",
    "ops/top_k_builder.cpp",
    "ops/transpose_builder.cpp",
    "ops/unsqueeze_builder.cpp",
    "ops/unstack_builder.cpp",
    "ops/where_builder.cpp",
  ]

  branch_protector_ret = "pac_ret"
  install_images = [ "system", "updater" ]

  public_configs = [
    "//foundation/ai/neural_network_runtime/config:coverage_flags",
    ":nnrt_config",
  ]

  deps = [
    "../neural_network_core:libneural_network_core",
  ]

  external_deps = [
    "c_utils:utils",
    "drivers_interface_nnrt:libnnrt_proxy_1.0",
    "drivers_interface_nnrt:libnnrt_proxy_2.0",
    "drivers_interface_nnrt:libnnrt_proxy_2.1",
    "hdf_core:libhdf_utils",
    "hilog:libhilog",
    "hitrace:libhitracechain",
    "init:libbegetutil",
    "ipc:ipc_core",
    "json:nlohmann_json_static",
    "mindspore:mindir_lib",
    "eventhandler:libeventhandler",
  ]

  metadata = {
    subsystem_name = "ai"
    innerapi_tags = [ "ndk" ]
    part_name = "neural_network_runtime"
  }
}
```

**证据**: `frameworks/native/neural_network_runtime/BUILD.gn`

### 依赖关系

```
libneural_network_runtime.so
    ├── deps: libneural_network_core.so
    └── external_deps:
        ├── c_utils:utils
        ├── drivers_interface_nnrt:libnnrt_proxy_1.0/2.0/2.1
        ├── hdf_core:libhdf_utils
        ├── hilog:libhilog
        ├── hitrace:libhitracechain
        ├── init:libbegetutil
        ├── ipc:ipc_core
        ├── json:nlohmann_json_static
        ├── mindspore:mindir_lib
        └── eventhandler:libeventhandler
```

## 配置定义

**文件**: `config/BUILD.gn`

```gn
declare_args() {
  neural_network_runtime_coverage = false
}

config("coverage_flags") {
  if (neural_network_runtime_coverage) {
    cflags = [ "--coverage" ]
    cflags_cc = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}
```

**证据**: `config/BUILD.gn`

### 配置参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `neural_network_runtime_coverage` | bool | false | 启用代码覆盖率检测 |

## 示例驱动构建

### v2.0 HDI 服务

**文件**: `example/drivers/nnrt/v2_0/hdi_cpu_service/BUILD.gn`

```gn
ohos_shared_library("libnnrt_device_service_2.0") {
  sources = [
    "src/nnrt_device_service.cpp",
    "src/node_functions.cpp",
    "src/node_registry.cpp",
    "src/prepared_model_service.cpp",
    "src/shared_buffer_parser.cpp",
    "src/validation.cpp",
  ]

  include_dirs = [
    "include",
    "../mindspore",
    "//third_party/flatbuffers/include",
  ]

  deps = [ ":mindspore_demo" ]

  external_deps = [
    "c_utils:utils",
    "drivers_interface_nnrt:libnnrt_stub_2.0",
    "hdf_core:libhdf_utils",
    "hilog:libhilog",
    "ipc:ipc_core",
  ]
}

ohos_shared_library("libnnrt_driver") {
  sources = [ "src/nnrt_device_driver.cpp" ]
  deps = [ ":libnnrt_device_service_2.0" ]
  external_deps = [
    "c_utils:utils",
    "drivers_interface_nnrt:libnnrt_stub_2.0",
    "hdf_core:libhdf_host",
    "hdf_core:libhdf_ipc_adapter",
    "hdf_core:libhdf_utils",
    "hdf_core:libhdi",
    "hilog:libhilog",
    "ipc:ipc_core",
  ]
}

group("hdf_nnrt_service") {
  deps = [
    ":libnnrt_device_service_2.0",
    ":libnnrt_driver",
  ]
}
```

**证据**: `example/drivers/nnrt/v2_0/hdi_cpu_service/BUILD.gn`

### 示例驱动 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `mindspore_demo` | ohos_prebuilt_shared_library | libmindspore-lite.so | 预构建 MindSpore 库 |
| `libnnrt_device_service_2.0` | ohos_shared_library | libnnrt_device_service_2.0.so | 设备服务实现 |
| `libnnrt_driver` | ohos_shared_library | libnnrt_driver.so | 驱动入口 |
| `hdf_nnrt_service` | group | - | 服务组合 |

## 构建命令

### 完整构建

```bash
./build.sh --product-name rk3568 --ccache --build-target neural_network_runtime --jobs 4
```

### 仅构建 Core 库

```bash
./build.sh --product-name rk3568 --ccache --build-target nncore_target --jobs 4
```

### 构建测试

```bash
./build.sh --product-name rk3568 --ccache --build-target nnrt_test_target --jobs 4
```

## Target 映射关系

```
nnrt_target (group)
    ├── libneural_network_core (ohos_shared_library)
    │   └── libneural_network_core.so
    └── libneural_network_runtime (ohos_shared_library)
        └── libneural_network_runtime.so
            └── deps: libneural_network_core.so

example/drivers/nnrt/v2_0/nnrt_entry (group)
    └── hdf_nnrt_service (group)
        ├── mindspore_demo (prebuilt)
        │   └── libmindspore-lite.so
        ├── libnnrt_device_service_2.0 (ohos_shared_library)
        │   └── libnnrt_device_service_2.0.so
        └── libnnrt_driver (ohos_shared_library)
            └── libnnrt_driver.so
```

## 关键配置项

### 安全编译选项

| 选项 | 值 | 说明 |
|------|-----|------|
| -fstack-protector-all | 启用 | 栈保护 |
| -fexceptions | 启用 | C++ 异常支持 |
| branch_protector_ret | pac_ret | 分支保护 (PAC-RET) |

### 安装路径

| 产物 | 安装镜像 | 路径 |
|------|----------|------|
| libneural_network_core.so | system, updater | /system/lib/ |
| libneural_network_runtime.so | system, updater | /system/lib/ |
| libnnrt_device_service_2.0.so | chipset_base_dir | /vendor/lib/ |
| libnnrt_driver.so | chipset_base_dir | /vendor/lib/ |

## 相关跳转

- [编译产物](07_Build_Artifacts.md)
- [目录结构](03_Directory_Structure.md)
- [架构说明](02_Architecture.md)

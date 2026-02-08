# GN 构建目标

## 构建系统概述

UDMF 项目使用 GN（Generate Ninja）作为构建系统，配置文件主要位于项目根目录和各子目录。GN 构建文件使用 `.gn` 和 `.gni` 扩展名，定义构建目标（target）、依赖关系和编译选项。

根据源码证据（`BUILD.gn` 根构建文件和 `udmf.gni` 配置头文件），UDMF 包含约 25 个 BUILD.gn 文件，分布在 `interfaces/`、`framework/`、`adapter/` 和 `conf/` 等目录。构建产物包括共享库（.so）、静态库（.a）、字节码（.abc）等多种格式。

## 根构建配置

### BUILD.gn 入口文件

根目录 `BUILD.gn:16-53` 定义了顶层构建目标：

```gn
import("//build/ohos.gni")

# 主构建产物组
group("udmf_packages") {
  if (is_standard_system) {
    deps = [
      # 接口层
      "interfaces/components:udmfcomponents",
      "interfaces/innerkits:pixelmap_wrapper",
      "interfaces/innerkits:udmf_client",
      "interfaces/innerkits:utd_client",
      "interfaces/innerkits/aipcore:aip_core_mgr_static",
      
      # N-API 接口
      "interfaces/jskits:intelligence_napi",
      "interfaces/jskits:udmf_data_napi",
      "interfaces/jskits:unifieddatachannel_napi",
      "interfaces/jskits:uniformtypedescriptor_napi",
      
      # NDK 接口
      "interfaces/ndk:libudmf",
    ]
  }
}

# 单元测试组
group("unittest") {
  testonly = true
  deps = [
    "framework/common/test/unittest:unittest",
    "framework/innerkitsimpl/test/unittest:unittest",
    "framework/jskitsimpl/test:stage_unittest",
    "framework/jskitsimpl/unittest:unittest",
    "framework/ndkimpl/test/unittest:unittest",
  ]
}

# 模糊测试组
group("fuzztest") {
  testonly = true
  deps = [
    "framework/common/test/fuzztest:fuzztest",
    "framework/innerkitsimpl/test/fuzztest:fuzztest",
    "framework/ndkimpl/test/fuzztest:fuzztest",
  ]
}
```

### udmf.gni 配置头文件

`udmf.gni` 定义构建路径变量：

```gn
# 第三方依赖路径
third_party_path = "//third_party"

# Ability 框架路径
if (is_arkui_x) {
  aafwk_path = "//foundation/appframework/ability/ability_runtime"
} else {
  aafwk_path = "//foundation/ability/ability_runtime"
}

# UDMF 路径变量
udmf_root_path = "//foundation/distributeddatamgr/udmf"
udmf_framework_path = "${udmf_root_path}/framework"
udmf_interfaces_path = "${udmf_root_path}/interfaces"
udmf_service_path = "${udmf_root_path}/service"

# 依赖路径
kv_store_path = "//foundation/distributeddatamgr/kv_store"
access_kit_path = "//base/security/access_token"
```

## InnerKit 构建目标

### interfaces/innerkits/BUILD.gn

```gn
# UDMF 客户端库
ohos_shared_library("udmf_client") {
  # 可见性配置
  cflags_cc = [ "-fvisibility=hidden" ]
  
  # 源码文件
  sources = [
    "${udmf_framework_path}/innerkitsimpl/client/udmf_client.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/unified_data.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/unified_record.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/unified_data_helper.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/type_descriptor.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/unified_data_properties.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/unified_html_record_process.cpp",
  ]
  
  # 公共类型文件
  sources += [
    "${udmf_framework_path}/innerkitsimpl/data/plain_text.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/text.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/link.cpp",
    "${udmf_framework_framework}/innerkitsimpl/data/file.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/folder.cpp",
  ]
  
  # 媒体类型文件
  sources += [
    "${udmf_framework_path}/innerkitsimpl/data/image.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/video.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/audio.cpp",
  ]
  
  # 系统定义类型
  sources += [
    "${udmf_framework_path}/innerkitsimpl/data/system_defined_record.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/system_defined_appitem.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/system_defined_form.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/system_defined_pixelmap.cpp",
  ]
  
  # 应用定义类型
  sources += [
    "${udmf_framework_path}/innerkitsimpl/data/application_defined_record.cpp",
  ]
  
  # 依赖
  deps = [
    "${udmf_interfaces_path}/innerkits/convert:ndk_data_conversion",
    "${udmf_interfaces_path}/innerkits/common:udmf_innerkits_common",
    "${kv_store_path}:distributed_kv_store",
    "//foundation/ability/ability_runtime:ability_runtime_base",
    "//base/security/access_token:access_token",
    "//base/hiviewdfx/hilog:hilog",
  ]
  
  # 公共依赖
  public_deps = [
    "${udmf_interfaces_path}/innerkits/client:udmf_client_target",
    "${udmf_interfaces_path}/innerkits/client:utd_client_target",
  ]
}

# UTD 客户端库
ohos_shared_library("utd_client") {
  visibility = [ ":*" ]
  sources = [
    "${udmf_framework_path}/innerkitsimpl/client/utd_client.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/type_descriptor.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/preset_type_descriptors.cpp",
    "${udmf_framework_path}/innerkitsimpl/data/flexible_type.cpp",
    "${udmf_framework_path}/common/utd_graph.cpp",
    "${udmf_framework_path}/common/custom_utd_store.cpp",
    "${udmf_framework_path}/common/custom_utd_json_parser.cpp",
    "${udmf_framework_path}/common/utd_cfgs_checker.cpp",
    "${udmf_framework_path}/innerkitsimpl/client/getter_system.cpp",
  ]
  
  deps = [
    "${kv_store_path}:distributed_kv_store",
    "//base/security/access_token:access_token",
    "//base/hiviewdfx/hilog:hilog",
    "//foundation/systemabilityparameter/samgr:samgr_client",
  ]
  
  # 公共头文件目录
  include_dirs = [ "${udmf_interfaces_path}/innerkits/data" ]
}

# 像素图包装器
ohos_shared_library("pixelmap_wrapper") {
  sources = [
    "${udmf_framework_path}/innerkitsimpl/dynamic/pixelmap_loader.cpp",
    "${udmf_interfaces_path}/innerkits/dynamic/pixelmap_wrapper.cpp",
  ]
  
  deps = [
    "//foundation/graphic/graphic_image:image_native",
  ]
}
```

## NDK 构建目标

### interfaces/ndk/BUILD.gn

```gn
# NDK 库（libudmf.so）
ohos_shared_library("libudmf") {
  # 符号可见性
  defines = [ "API_EXPORT=__attribute__((visibility (\"default\")))" ]
  
  # 安装路径
  relative_install_dir = "ndk"
  output_name = "udmf"
  output_extension = "so"
  
  # 源码
  sources = [
    "${udmf_framework_path}/ndkimpl/data/udmf.cpp",
    "${udmf_framework_path}/ndkimpl/data/uds.cpp",
    "${udmf_framework_path}/ndkimpl/data/utd.cpp",
    "${udmf_framework_path}/ndkimpl/data/data_provider_impl.cpp",
  ]
  
  # 依赖
  deps = [
    "${udmf_interfaces_path}/innerkits:udmf_client",
    "${udmf_interfaces_path}/innerkits:utd_client",
    "${kv_store_path}:distributed_kv_store",
    "//foundation/graphic/graphic_image:image_native",
  ]
  
  # 外部链接
  external_deps = [
    "hilog:hilog",
    "ipc:ipc_core",
    "safwk:safwk",
  ]
}
```

## N-API 构建目标

### interfaces/jskits/BUILD.gn

```gn
# 统一数据通道 N-API
ohos_shared_library("unifieddatachannel_napi") {
  output_name = "unifieddatachannel_napi"
  relative_install_dir = "module/data"
  
  sources = [
    "${udmf_framework_path}/jskitsimpl/module/unified_data_channel_napi_module.cpp",
    "${udmf_framework_path}/jskitsimpl/data/unified_data_channel_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/data/unified_data_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/data/get_data_params_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/data/data_load_params_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/data/unified_data_properties_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/data/summary_napi.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/jskits/common:jskits_napi_queue",
    "${udmf_interfaces_path}/jskits/common:jskits_napi_data_utils",
    "${udmf_interfaces_path}/innerkits:udmf_client",
    "${udmf_interfaces_path}/innerkits:utd_client",
  ]
}

# 统一类型描述符 N-API
ohos_shared_library("uniformtypedescriptor_napi") {
  output_name = "uniformtypedescriptor_napi"
  relative_install_dir = "module/data"
  
  sources = [
    "${udmf_framework_path}/jskitsimpl/module/uniform_type_descriptor_napi_module.cpp",
    "${udmf_framework_path}/jskitsimpl/data/uniform_type_descriptor_napi.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/jskits/common:jskits_napi_data_utils",
    "${udmf_interfaces_path}/innerkits:utd_client",
  ]
}

# 数据 N-API
ohos_shared_library("udmf_data_napi") {
  sources = [
    "${udmf_framework_path}/jskitsimpl/data/record_type_napi.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/innerkits:udmf_client",
  ]
}

# 智能 N-API
ohos_shared_library("intelligence_napi") {
  output_name = "intelligence_napi"
  relative_install_dir = "module/data"
  
  sources = [
    "${udmf_framework_path}/jskitsimpl/intelligence/native_module_intelligence.cpp",
    "${udmf_framework_path}/jskitsimpl/intelligence/text_embedding_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/intelligence/image_embedding_napi.cpp",
    "${udmf_framework_path}/jskitsimpl/intelligence/aip_napi_utils.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/innerkits/aipcore:aip_core_mgr_static",
  ]
}
```

## Taihe 构建目标

### interfaces/taihe/BUILD.gn

```gn
# Taihe 原生库
taihe_shared_library("udmf_taihe_native") {
  idl_sources = [
    "ohos.data.intelligence.taihe",
    "ohos.data.unifiedDataChannel.taihe",
    "ohos.data.uniformDataStruct.taihe",
    "ohos.data.uniformTypeDescriptor.taihe",
  ]
  
  sources = [
    "include/*.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/innerkits:udmf_client",
    "${udmf_interfaces_path}/innerkits:utd_client",
  ]
}

# Taihe ABC 字节码
generate_static_abc("udmf_abc") {
  sources = [ "ets/**/*.ets" ]
  output_dir = "$target_gen_dir/udmf_abc"
}

# Taihe 构建组
group("udmf_taihe") {
  deps = [
    ":udmf_taihe_native",
    ":udmf_abc",
  ]
}
```

## Cangjie 构建目标

### interfaces/cj/BUILD.gn

```gn
# Cangjie FFI - UnifiedDataChannel
ohos_shared_library("cj_unified_data_channel_ffi") {
  sources = [
    "include/unified_data_ffi.cpp",
    "include/unified_data_impl.cpp",
    "include/unified_record_ffi.cpp",
    "include/unified_record_impl.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/innerkits:udmf_client",
  ]
}

# Cangjie FFI - UniformTypeDescriptor
ohos_shared_library("cj_uniform_type_descriptor_ffi") {
  sources = [
    "include/type_descriptor_ffi.cpp",
    "include/type_descriptor_impl.cpp",
  ]
  
  deps = [
    "${udmf_interfaces_path}/innerkits:utd_client",
  ]
}
```

## UI 组件构建目标

### interfaces/components/BUILD.gn

```gn
# JS 组件字节码
es2abc_gen_abc("gen_udmfcomponents_abc") {
  sources = [ "udmfcomponents.js" ]
}

# 组件库
ohos_shared_library("udmfcomponents") {
  # 嵌入 ABC 数据
  deps = [ ":gen_udmfcomponents_abc" ]
  
  # 链接 ABC
  libs = [ "udmfcomponents_abc.o" ]
}
```

## AI 核心构建目标

### interfaces/innerkits/aipcore/BUILD.gn

```gn
# AI 核心静态库
ohos_static_library("aip_core_mgr_static") {
  sources = [
    "${udmf_framework_path}/innerkitsimpl/intelligence/aip_core_manager.cpp",
  ]
  
  deps = [
    "//foundation/ai/engine/services/client:ai_engine_client",
  ]
}
```

## 配置构建目标

### conf/BUILD.gn

```gn
# UTD 类型配置
copy("utd_conf") {
  sources = [ "uniform_data_types.json" ]
  outputs = [ "$root_out_dir/system/etc/utd/conf/uniform_data_types.json" ]
}
```

## 安全编译选项

### 通用安全配置

所有共享库目标启用以下安全编译选项：

```gn
# CFI（控制流完整性）
sanitize = {
  cfi = true
  cfi_cross_dso = true
}

# 边界 sanitizer
boundary_sanitize = true

# 未定义行为 sanitizer
ubsan = true

# PAC（指针认证）
branch_protector_ret = "pac_ret"

# 符号可见性
cflags_cc = [ "-fvisibility=hidden" ]
```

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口
- [31_Build_Artifacts.md](./31_Build_Artifacts.md)：编译产物清单
- [40_Security_Analysis.md](./40_Security_Analysis.md)：安全分析

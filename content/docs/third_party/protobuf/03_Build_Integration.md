# OH 构建适配

## 3.1 BUILD.gn 结构概览

### 导入与配置

```gn
import("//build/ohos.gni")

THIRDPARTY_PROTOBUF_SUBSYS_NAME = "thirdparty"
THIRDPARTY_PROTOBUF_PART_NAME = "protobuf"

config("protobuf_config") {
  include_dirs = [
    "src",
    "third_party/abseil-cpp",
    "third_party/utf8_range",
  ]
  cflags = [ "-Wno-gcc-compat" ]
}
```

### 目标定义

| 目标名称 | 类型 | 关键特性 |
|----------|------|----------|
| `protobuf_lite` | ohos_shared_library | 轻量级共享库 |
| `protobuf_lite_static` | ohos_static_library | 轻量级静态库 |
| `protobuf` | ohos_shared_library | 完整版共享库 |
| `protobuf_static` | ohos_static_library | 完整版静态库 |
| `protoc_lib` | ohos_shared_library | 编译器库（共享） |
| `protoc_static_lib` | ohos_static_library | 编译器库（静态） |
| `protoc` | ohos_executable | 编译器可执行文件 |

## 3.2 关键编译选项

### 3.2.1 宏定义 (defines)

```gn
# 通用配置
"-D HAVE_PTHREAD"           # POSIX 线程支持

# OH 特有配置
"HAVE_HILOG"                # OpenHarmony 日志系统（仅 target，非 MinGW）
"_FILE_OFFSET_BITS_SET_LSEEK"  # MinGW 文件偏移兼容

# 警告抑制
"-Wno-sign-compare"         # 符号比较警告
"-Wno-deprecated-declarations"  # 废弃声明警告
"-Wno-gcc-compat"           # GCC 兼容性警告
"-Wno-macro-redefined"      # MinGW lseek 宏重定义
"-Wno-unused-function"     # 未使用函数警告
"-Wno-unused-private-field" # 未使用私有字段警告
```

### 3.2.2 包含路径 (include_dirs)

```gn
include_dirs = [
  "src/google/protobuf/**/*.h",   # protobuf 头文件
  "src/google/protobuf/**/*.inc", # 内联实现
  "src",                          # 公共头文件目录
  "third_party/utf8_range",      # UTF-8 验证库
  "//third_party/protobuf",       # protoc 编译依赖
]
```

### 3.2.3 外部依赖 (external_deps)

```gn
# 标准配置
external_deps = [ "abseil-cpp:absl_base_static" ]

# 条件依赖（is_arkui_x）
deps = [ "//third_party/abseil-cpp:absl_base_static" ]
```

## 3.3 平台适配

### 3.3.1 目标平台 vs 主机平台

```gn
# 目标平台（非 MinGW）
if (!is_mingw) {
  external_deps = [ "abseil-cpp:absl_base_static" ]
  defines = [ "HAVE_HILOG" ]  # 仅目标平台启用 HILOG
}

# MinGW 平台
else {
  defines = [ "_FILE_OFFSET_BITS_SET_LSEEK" ]
}
```

### 3.3.2 ArkUI-X 特殊处理

```gn
# ArkUI-X 平台使用内部依赖
if (is_arkui_x) {
  deps = [ "//third_party/abseil-cpp:absl_base_static" ]
} else {
  external_deps = [ "abseil-cpp:absl_base_static" ]
}
```

### 3.3.3 macOS/iOS 框架链接

```gn
# 仅 macOS/iOS
if (is_mac || is_ios) {
  frameworks = [ "Foundation.framework" ]
}
```

### 3.3.4 安全加固

```gn
# protobuf_lite 启用 PAC-RET
ohos_shared_library("protobuf_lite") {
  branch_protector_ret = "pac_ret"
  # ...
}
```

## 3.4 与上游构建系统差异

### 3.4.1 构建系统对比

| 维度 | 上游 (CMake/Bazel) | OpenHarmony (BUILD.gn) |
|------|-------------------|------------------------|
| **构建工具** | CMake / Bazel | GN (OHOS 构建系统) |
| **目标类型** | STATIC/SHARED LIBRARY | ohos_static_library / ohos_shared_library |
| **配置方式** | CMake 选项 | GN config + 条件编译 |
| **安装方式** | cmake --install | install_enable = true/false |
| **子系统** | N/A | subsystem_name + part_name |

### 3.4.2 源文件覆盖

上游 CMakeLists.txt 包含完整的 protobuf 功能，而 OH BUILD.gn 按需选择：

| 组件 | CMake (上游) | BUILD.gn (OH) | 差异 |
|------|-------------|----------------|------|
| **lite 运行时** | 可选 | 完整支持 | 一致 |
| **full 运行时** | 完整 | 完整 | 一致 |
| **编译器 (protoc)** | 完整 | 完整 | 一致 |
| **语言生成器** | 全部 | 全部 | 一致 |
| **模糊测试** | Bazel 规则 | Patch 修复 | 1 个 Patch |

### 3.4.3 功能裁剪

OH 版本**未裁剪**任何上游功能，所有特性均保持可用。

## 3.5 特殊处理

### 3.5.1 源文件组织

```gn
# protobuf_lite 源文件示例
sources = [
  "src/google/protobuf/any_lite.cc",
  "src/google/protobuf/arena.cc",
  # ... lite 专有文件
  "third_party/utf8_range/utf8_range.c",
  "third_party/utf8_range/utf8_validity.cc",
]
```

### 3.5.2 内层 API 标签

```gn
innerapi_tags = [
  "platformsdk_indirect",  # 间接平台 SDK
  "chipsetsdk_sp",          # 芯片 SDK 特权
]
```

### 3.5.3 安装配置

| 目标 | install_enable | 说明 |
|------|----------------|------|
| protobuf_lite | true | 安装到系统库 |
| protobuf | true | 安装到系统库 |
| protoc_lib | false | 编译器库，不单独安装 |
| protoc | false | 可执行文件，不单独安装 |

## 3.6 构建配置示例

### 3.6.1 最小构建配置

```gn
# 最小 protobuf_lite_static 配置
ohos_static_library("my_protobuf") {
  sources = [
    "src/google/protobuf/message_lite.cc",
    "src/google/protobuf/arena.cc",
    "src/google/protobuf/stubs/common.cc",
  ]
  include_dirs = ["src"]
  external_deps = [ "abseil-cpp:absl_base_static" ]
}
```

### 3.6.2 自定义 HILOG 集成

```gn
# 如果需要自定义日志行为
config("my_protobuf_config") {
  defines = [ "HAVE_PTHREAD" ]
  # 不定义 HAVE_HILOG 以使用标准日志
}
```

## 3.7 构建验证

### 构建命令

```bash
# 构建 protobuf_lite 静态库
hb build -T //third_party/protobuf:protobuf_lite_static

# 构建完整版本
hb build -T //third_party/protobuf:protobuf_static

# 构建 protoc
hb build -T //third_party/protobuf:protoc
```

### 验证清单

- [ ] `protobuf_lite_static` 构建成功
- [ ] `protobuf_static` 构建成功
- [ ] `protoc` 可执行文件运行正常
- [ ] 链接测试（静态/动态）通过
- [ ] HILOG 日志输出正常（目标平台）

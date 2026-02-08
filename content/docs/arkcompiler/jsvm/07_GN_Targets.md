# GN Targets 与编译产物

> targets 列表、类型、依赖、产物、开关

## 目的与适用范围

### 目的
本文档描述 JSVM 的 GN 构建系统，包括所有 targets、依赖关系、编译产物等。

### 适用范围
- 需要理解构建系统的开发者
- 需要进行构建优化的开发者
- 需要进行跨平台移植的开发者

---

## GN Targets 清单

### 1. jit_enable_list_appid

**类型**: ohos_prebuilt_etc

**用途**: 复制 JIT 启用列表配置文件。

**源文件**: `./jit_enable_list_appid.conf`

**输出文件**: `jit_enable_list_appid.conf`

**安装路径**: `/system/etc/jsvm/`

**依赖**: 无

**证据位置**: `BUILD.gn:18-23`

### 2. copy_v8

**类型**: action

**用途**: 从预编译目录复制 V8 库和头文件。

**脚本**: `copy_v8.sh`

**输出文件**:
- `$target_gen_dir/libv8_shared.so`
- `$target_gen_dir/v8-include`

**依赖**: 无

**参数**:
- `--target_gen_dir`: 目标生成目录
- V8 源路径: `//vendor/${dependency_tag}/binary/artifacts/js_engine_url/`
- `target_cpu`: 目标 CPU 架构

**证据位置**: `BUILD.gn:28-43`

### 3. copy_llhttp

**类型**: action

**用途**: 复制 llhttp 源码。

**脚本**: `copy_llhttp.sh`

**输出文件**:
- `$target_gen_dir/llhttp/src/api.c`
- `$target_gen_dir/llhttp/src/http.c`
- `$target_gen_dir/llhttp/src/llhttp.c`
- `$target_gen_dir/llhttp/include`

**依赖**: 无

**参数**:
- `--target_gen_dir`: 目标生成目录

**证据位置**: `BUILD.gn:45-60`

### 4. libv8_config

**类型**: config

**用途**: 提供 V8 头文件路径。

**包含目录**: `$target_gen_dir/v8-include`

**依赖**: 无

**证据位置**: `BUILD.gn:62-64`

### 5. libv8

**类型**: ohos_prebuilt_shared_library

**用途**: V8 预编译共享库。

**源文件**: `$target_gen_dir/libv8_shared.so`（由 copy_v8 生成）

**输出文件**: `libv8_shared.so`

**依赖**:
- `:copy_v8`

**公共配置**: `:libv8_config`

**子系统**: arkcompiler

**部件**: jsvm

**安装启用**: true

**安装镜像**: [system]

**证据位置**: `BUILD.gn:66-77`

### 6. llhttp_config

**类型**: config

**用途**: 提供 llhttp 头文件路径。

**包含目录**: `$target_gen_dir/llhttp/include`

**依赖**: 无

**证据位置**: `BUILD.gn:79-81`

### 7. llhttp

**类型**: ohos_static_library

**用途**: llhttp 静态库。

**源文件**:
- `$target_gen_dir/llhttp/src/api.c`
- `$target_gen_dir/llhttp/src/http.c`
- `$target_gen_dir/llhttp/src/llhttp.c`

**依赖**:
- `:copy_llhttp`

**公共配置**: `:llhttp_config`

**子系统**: arkcompiler

**部件**: jsvm

**证据位置**: `BUILD.gn:83-95`

### 8. public_jsvm_config

**类型**: config

**用途**: 提供 JSVM 公共头文件路径。

**包含目录**: `interface/kits`

**依赖**: 无

**证据位置**: `BUILD.gn:97-99`

### 9. build_libjsvm

**类型**: action

**用途**: 构建 libjsvm.so。

**脚本**: `build_jsvm.sh`

**输出文件**: `$target_gen_dir/libjsvm.so`（或 `asan/libjsvm.so`）

**外部依赖**:
- bounds_checking_function:libsec_static
- hilog:libhilog
- hisysevent:libhisysevent
- hitrace:hitrace_meter
- icu:shared_icui18n
- icu:shared_icuuc
- init:libbegetutil
- libuv:uv
- openssl:libcrypto_shared
- openssl:libssl_shared
- resource_schedule_service:ressched_client
- zlib:libz

**依赖**:
- `:libv8`
- `:llhttp`

**参数**:
- `--target_gen_dir`: 目标生成目录
- `--target_out_dir`: 目标输出目录
- `--target_cpu`: 目标 CPU 架构
- `--target_platform`: 目标平台
- `--prefix`: 工具链前缀
- `--sysroot`: sysroot 路径
- `--is_asan`: 是否启用 Address Sanitizer
- `--use_hwasan`: 是否启用 Hardware-Assisted Address Sanitizer
- `--cmake_path`: CMake 路径
- `--dependency_tag`: 依赖标签
- `--jsvm_path`: JSVM 路径

**证据位置**: `BUILD.gn:121-181`

### 10. libjsvm

**类型**: ohos_prebuilt_shared_library

**用途**: JSVM 主共享库（对外产物）。

**源文件**: `$target_gen_dir/libjsvm.so`（由 build_libjsvm 生成）

**输出文件**: `libjsvm.so`

**依赖**:
- `:jit_enable_list_appid`
- `:build_libjsvm`

**公共配置**: `:public_jsvm_config`

**子系统**: arkcompiler

**部件**: jsvm

**安装启用**: true

**安装镜像**: [system]

**内部 API 标签**: [ndk]

**条件编译**: ASAN/HWASAN 支持

**证据位置**: `BUILD.gn:101-119`

### 11. jsvm_packages

**类型**: group

**用途**: 顶层打包目标。

**依赖**:
- `:libjsvm`

**证据位置**: `BUILD.gn:183-185`

---

## 依赖关系图

```
jsvm_packages (group)
    └── libjsvm (ohos_prebuilt_shared_library)
            ├── jit_enable_list_appid (ohos_prebuilt_etc)
            └── build_libjsvm (action)
                    ├── libv8 (ohos_prebuilt_shared_library)
                    │       └── copy_v8 (action)
                    └── llhttp (ohos_static_library)
                            └── copy_llhttp (action)
```

**证据位置**: `BUILD.gn:18-185`

---

## 编译产物

### 主产物

| 产物 | 类型 | 位置 | 用途 |
|------|------|------|------|
| libjsvm.so | 共享库 | /system/lib/ | JSVM 主库 |
| libv8_shared.so | 共享库 | /system/lib/ | V8 引擎库 |
| jit_enable_list_appid.conf | 配置文件 | /system/etc/jsvm/ | JIT 配置 |

**证据位置**:
- `BUILD.gn:101-119` - libjsvm
- `BUILD.gn:66-77` - libv8
- `BUILD.gn:18-23` - jit_enable_list_appid

### 辅助产物

| 产物 | 类型 | 位置 | 用途 |
|------|------|------|------|
| llhttp | 静态库 | 构建目录 | HTTP 解析 |

**证据位置**: `BUILD.gn:83-95` - llhttp

---

## 编译配置

### jsvm.gni

**路径**: `jsvm.gni`

**用途**: 定义源文件列表和编译参数。

**源文件列表**:
```gni
jsvm_sources = [
  "src/js_native_api_v8.cpp",
  "src/jsvm_env.cpp",
  "src/jsvm_reference.cpp",
]

jsvm_inspector_sources = [
  "src/inspector/inspector_socket.cpp",
  "src/inspector/inspector_socket_server.cpp",
  "src/inspector/inspector_utils.cpp",
  "src/inspector/js_native_api_v8_inspector.cpp",
]
```

**编译参数**:
```gni
declare_args() {
  jsvm_shared_libuv = true
  enable_debug = false
  enable_inspector = true
  use_platform_ohos = true
  support_hwasan = true
}
```

**证据位置**: `jsvm.gni:14-36`

---

## 特性开关

### enable_inspector

**类型**: boolean

**默认值**: true

**用途**: 启用/禁用 Inspector 功能。

**定义位置**: `jsvm.gni:32`

### enable_debug

**类型**: boolean

**默认值**: false

**用途**: 启用/禁用调试功能。

**定义位置**: `jsvm.gni:31`

### jsvm_shared_libuv

**类型**: boolean

**默认值**: true

**用途**: 是否共享 libuv。

**定义位置**: `jsvm.gni:30`

### use_platform_ohos

**类型**: boolean

**默认值**: true

**用途**: 是否使用 OpenHarmony 平台实现。

**定义位置**: `jsvm.gni:33`

### support_hwasan

**类型**: boolean

**默认值**: true

**用途**: 是否支持 Hardware-Assisted Address Sanitizer。

**定义位置**: `jsvm.gni:34`

---

## 构建流程

### 1. 复制 V8 和 llhttp

```
copy_v8 → 复制 libv8_shared.so 和 v8-include
copy_llhttp → 复制 llhttp 源码
```

**证据位置**: `BUILD.gn:28-60`

### 2. 构建辅助库

```
llhttp → 编译 llhttp.c, http.c, api.c
```

**证据位置**: `BUILD.gn:83-95`

### 3. 构建 libjsvm.so

```
build_jsvm.sh → 编译所有源文件，链接依赖库
```

**证据位置**: `BUILD.gn:121-181`

### 4. 安装产物

```
libjsvm.so → 安装到 /system/lib/
libv8_shared.so → 安装到 /system/lib/
jit_enable_list_appid.conf → 安装到 /system/etc/jsvm/
```

**证据位置**: `BUILD.gn:18-77,101-119`

---

## 运行时加载关系

### 动态链接依赖

libjsvm.so 运行时依赖以下共享库：

- libv8_shared.so
- libhilog.so
- libhisysevent.so
- libhitrace_meter.so
- libicui18n.so
- libicuuc.so
- libbegetutil.so
- libuv.so（如果 jsvm_shared_libuv = true）
- libcrypto.so
- libssl.so
- libz.so

**证据位置**: `BUILD.gn:122-135` - build_libjsvm external_deps

### 加载顺序

1. 系统加载 libjsvm.so
2. libjsvm.so 加载 libv8_shared.so
3. libjsvm.so 加载其他依赖库

---

## 相关链接

- [目录结构与模块职责](./02_Directory_Structure.md) - 构建配置文件
- [附录：配置 Flags](./appendix/Config_Flags.md) - 编译选项详解

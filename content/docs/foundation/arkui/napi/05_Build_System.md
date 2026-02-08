# 构建系统

## GN 构建概览

**证据**：`BUILD.gn`, `napi.gni`

N-API 组件使用 GN（Generate Ninja）构建系统，共有 18 个 `.gn` 文件和 2 个 `.gni` 文件。

## 主要 Targets

### 根目录 BUILD.gn

| Target | 类型 | 输出 | 依赖 | 证据位置 |
|--------|------|------|------|----------|
| `ace_napi_config` | config | - | - | `BUILD.gn:22` |
| `data_protector_config` | config | - | - | `BUILD.gn:100` |
| `module_manager_config` | config | - | - | `BUILD.gn:105` |
| `pac_data_protector_feature` | ohos_source_set | - | `:ace_napi_config`, `:data_protector_config` | `BUILD.gn:109` |
| `ace_napi_static` | ohos_source_set | - | `:ace_napi_config`, 外部依赖 | `BUILD.gn:120` |
| `ace_napi` | ohos_static_library / ohos_shared_library | `libace_napi.so` | `:ace_napi_static` | `BUILD.gn` |
| `cj_bind_native` | ohos_shared_library | `libcj_bind_native.so` | `:ffi_bind_native_config`, `hilog`, `c_utils` | `BUILD.gn` |
| `cj_bind_ffi` | ohos_shared_library | `libcj_bind_ffi.so` | `:cj_bind_ffi_source`, `:cj_bind_native` | `BUILD.gn` |
| `cj_ffi_libraries` | group | - | `:cj_bind_ffi` | `BUILD.gn` |
| `napi_packages` | group | - | `:ace_napi`, `ark_interop:ark_interop` | `BUILD.gn` |

### interfaces/inner_api/cjffi/ark_interop/BUILD.gn

| Target | 类型 | 输出 | 依赖 | 证据位置 |
|--------|------|------|------|----------|
| `ark_interop_config` | config | - | - | `ark_interop/BUILD.gn` |
| `cj_envsetup` | ohos_source_set | - | `:ark_interop_config` | `ark_interop/BUILD.gn` |
| `ark_interop` | ohos_shared_library | `libark_interop.so` | `:cj_envsetup`, `:ace_napi` | `ark_interop/BUILD.gn` |

## napi.gni 配置变量

**证据**：`napi.gni`

| 变量 | 用途 | 默认值 |
|------|------|--------|
| `napi_path` | N-API 基础路径 | `//foundation//arkui/napi` |
| `ets_runtime_path` | ETS 运行时路径 | `//arkcompiler/ets_runtime` |
| `enabled_data_protector` | 启用数据保护 | `false`（arm64 OHOS 自动启用） |
| `napi_sources` | 源文件列表 | 44+ 核心文件 | `napi.gni:19-55` |
| `napi_enable_container_scope` | 启用容器作用域 | `false` | `napi.gni:58` |
| `napi_enable_memleak_debug` | 启用内存泄漏调试 | `true` | `napi.gni:59` |
| `napi_feature_enable_pgo` | 启用 PGO 构建 | `false` | `napi.gni:62` |
| `napi_feature_pgo_path` | PGO profile 路径 | `""` | `napi.gni:65` |
| `module_output_path` | 输出目录 | `napi/napi` | `napi.gni:75` |

## 核心源文件

**证据**：`napi.gni:19-55`

### 分类列表

| 分类 | 文件 |
|------|------|
| **回调/作用域管理** | `callback_scope_manager/native_callback_scope_manager.cpp` |
| **模块管理** | `module_manager/module_checker_delegate.cpp`, `module_manager/module_load_checker.cpp`, `module_manager/native_module_manager.cpp` |
| **NativeEngine (Ark)** | `native_engine/impl/ark/ark_*.cpp` (7 个文件) |
| **NativeEngine (核心)** | `native_engine/native_*.cpp` (10 个文件) |
| **引用管理** | `reference_manager/native_reference_manager.cpp` |
| **工具类** | `utils/data_protector.cpp`, `utils/log.cpp`, `utils/platform/*.cpp` |

## 平台特定编译定义

**证据**：`BUILD.gn:22-98`

| 平台 | 编译定义 |
|------|----------|
| OHOS 标准 | `OHOS_PLATFORM`, `OHOS_STANDARD_PLATFORM`, `ENABLE_HITRACE`, `ENABLE_EVENT_HANDLER`, `ENABLE_FFRT` |
| OHOS 模拟器 | `SIMULATOR` |
| Windows (MinGW) | `WINDOWS_PLATFORM`, `PREVIEW` |
| macOS | `MAC_PLATFORM`, `PREVIEW` |
| Linux | `LINUX_PLATFORM`, `PREVIEW` |
| iOS (ArkUI-X) | `IOS_PLATFORM` |
| Android (ArkUI-X) | `ANDROID_PLATFORM` |
| Watch/Wearable | `DISABLE_SHORT_IDLE_CHECK` |

## CPU 架构特定定义

| CPU | 编译定义 |
|-----|----------|
| x86_64/amd64 | `NAPI_TARGET_AMD64`, `NAPI_TARGET_64` |
| x86 | `NAPI_TARGET_X86`, `NAPI_TARGET_32` |
| arm64 | `NAPI_TARGET_ARM64`, `NAPI_TARGET_64`, `_ARM64_` |
| arm | `NAPI_TARGET_ARM32`, `NAPI_TARGET_32` |

## 产物清单

| 产物 | 路径 | 描述 |
|------|------|------|
| `libace_napi.so` / `libace_napi.a` | `out/` | 主 N-API 库 |
| `libark_interop.so` | `out/` | Ark 互操作库 |
| `libcj_bind_native.so` | `out/` | CJ 原生绑定 |
| `libcj_bind_ffi.so` | `out/` | CJ FFI 绑定 |

## 构建依赖链

```
napi_packages (group)
├── ace_napi (shared_library/static_library)
│   └── ace_napi_static (source_set)
│       ├── pac_data_protector_feature [可选]
│       │   └── utils/data_protector.cpp
│       ├── cj_envsetup [OHOS only]
│       └── 外部依赖: libark_jsruntime, libuv, icu 等
│
├── ark_interop (shared_library)
│   ├── cj_envsetup (source_set)
│   └── ace_napi
│
cj_ffi_libraries (group)
└── cj_bind_ffi (shared_library)
    ├── cj_bind_ffi_source (source_set)
    └── cj_bind_native (shared_library)
```

## 构建命令

```bash
# 构建主 N-API 库
./build.sh --product-name xxx --target-name ace_napi

# 构建完整 N-API 包
./build.sh --product-name xxx --target-name napi_packages

# 运行测试
./build.sh --product-name xxx --target-name napi_packages_test
```

## 示例模块构建

**证据**：`sample/native_module_calc/BUILD.gn`

```gn
ohos_shared_library("calc") {
  sources = [
    "napi_calc.cpp",
  ]
  deps = [
    "//foundation/arkui/napi:ace_napi",
  ]
  relative_install_dir = "module"
  subsystem_name = "arkui"
  part_name = "napi"
}
```

## 配置开关

| 开关 | 用途 | 默认值 | 证据位置 |
|------|------|--------|----------|
| `napi_enable_container_scope` | 启用容器作用域 | false | `napi.gni:58` |
| `napi_enable_memleak_debug` | 启用内存泄漏检测 | true | `napi.gni:59` |
| `napi_feature_enable_pgo` | 启用 PGO 优化 | false | `napi.gni:62` |
| `enabled_data_protector` | 启用数据保护 | auto | `napi.gni:70` |

---

**相关文档**：
- [N-API 接口参考](./04_NAPI_Reference.md)
- [故障排查](./07_Troubleshooting.md)

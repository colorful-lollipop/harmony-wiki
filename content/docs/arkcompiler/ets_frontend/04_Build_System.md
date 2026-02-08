# GN 构建系统

## 概述

ets_frontend 使用 GN (Generate Ninja) 作为构建系统，输出 Ninja build files 进行编译。

## 根构建配置

### BUILD.gn (arkcompiler/ets_frontend/BUILD.gn)

**主入口**: `group("ets_frontend_build")`

```gn
group("ets_frontend_build") {
  deps = [
    "./es2panda:es2panda_build",
    "./merge_abc:merge_proto_abc_build",
  ]
}
```

**证据**: `BUILD.gn:21-26`

### 编译配置 (ark_config.gni)

| 配置项 | 描述 |
|--------|------|
| `ark_standalone_build` | 独立构建模式 |
| `is_arkui_x` | ArkUI-X 模式 |
| `is_linux/is_mingw/is_mac` | 目标平台 |
| `is_fastverify` | 快速验证模式 |
| `enable_hilog` | 启用日志 |
| `enable_relayout_profile` | 重排分析 |

### 平台定义 (BUILD.gn:29-95)

根据不同平台设置宏定义：

| 平台 | 宏定义 |
|------|--------|
| Linux | `PANDA_TARGET_UNIX`, `PANDA_TARGET_LINUX` |
| Windows | `PANDA_TARGET_WINDOWS` |
| macOS | `PANDA_TARGET_UNIX`, `PANDA_TARGET_MACOS` |
| OpenHarmony | `PANDA_TARGET_OHOS`, `PANDA_TARGET_UNIX` |

## 关键 Targets

### 主 Targets 列表

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| `ets_frontend_build` | group | - | 根 BUILD.gn |
| `es2panda` | executable | es2abc | es2panda/BUILD.gn |
| `es2panda_build` | group | - | es2panda/BUILD.gn |
| `merge_proto_abc_build` | executable | merge_abc | merge_abc/BUILD.gn |
| `libes2panda_public` | shared_library | libes2panda.so | ets2panda/BUILD.gn |
| `libes2panda_public_headers` | headers | - | ets2panda/BUILD.gn |
| `ets2panda` | executable | ets2panda | ets2panda/aot/BUILD.gn |
| `build_arkguard_etc` | - | arkguard | arkguard/BUILD.gn |
| `ohos_ets_build_system` | - | 构建系统 | ets2panda/driver/build_system/ |
| `ohos_ets_dependency_analyzer` | - | 依赖分析 | ets2panda/driver/dependency_analyzer/ |

**证据**: `bundle.json:36-61`

### 编译器配置

```gn
config("ark_config") {
  visibility = ["./*", "//arkcompiler/ets_frontend/*"]

  defines = ["PANDA_TARGET_MOBILE_WITH_MANAGED_LIBS=1"]

  # C++ 标准
  cflags_cc = [
    "-std=c++17",
    "-pedantic",
    "-Wall",
    "-Wextra",
    "-Werror",
    "-fno-rtti",
  ]

  # 平台特定优化
  if (is_fastverify) {
    cflags_cc += ["-O3", "-ggdb3", "-gdwarf-4"]
  } else if (is_debug) {
    cflags_cc += ["-Og", "-ggdb3", "-gdwarf-4"]
  }
}
```

**证据**: `BUILD.gn:29-167`

### 依赖配置 (bundle.json)

```json
{
  "deps": {
    "components": [
      "json",
      "runtime_core",
      "zlib",
      "bounds_checking_function",
      "protobuf",
      "icu",
      "abseil-cpp",
      "hilog",
      "typescript"
    ],
    "third_party": [
      "typescript"
    ]
  }
}
```

## 构建命令

### 标准构建

```bash
./build.sh --product-name rk3568 --build-target ets_frontend_build
```

### 独立构建

```bash
# 设置独立构建
ark_standalone_build=true gn gen out/clang_x64
ninja -C out/clang_x64
```

### 单模块构建

```bash
ninja -C out/rk3568/clang_x64 arkcompiler/ets_frontend:es2panda
```

## 构建产物

### 目录结构

```
out/
└── <product>/
    └── <toolchain>/
        └── arkcompiler/
            └── ets_frontend/
                ├── es2panda          # 编译器可执行文件
                ├── libes2panda.so    # 公共库
                ├── merge_abc         # 合并工具
                └── arkguard/         # 保护工具
```

### 产物说明

| 产物 | 类型 | 用途 |
|------|------|------|
| `es2panda` | Executable | JS/TS 编译器命令行工具 |
| `ets2panda` | Executable | ETS 编译器命令行工具 |
| `libes2panda.so` | Shared Library | 公共库，供其他模块链接 |
| `merge_abc` | Executable | ABC 文件合并工具 |
| `arkguard` | Library/Executable | 代码保护工具 |

## 相关文档

- [编译产物与运行时](06_Build_Outputs.md)
- [命令行接口](02_CLI_Reference.md)

# OpenHarmony Build 项目概览

## 项目定位

OpenHarmony **build** 仓库是 OpenHarmony 操作系统的**编译构建子系统**，提供基于 GN (Generate Ninja) 和 Ninja 的完整编译构建框架。

## 核心能力

### 1. 多语言构建支持

| 语言 | 支持程度 | 主要模板 |
|------|---------|---------|
| C/C++ | 完整支持 | `ohos_shared_library`, `ohos_executable`, `ohos_static_library` |
| Rust | 完整支持 | `ohos_rust_executable`, `ohos_rust_shared_library`, `ohos_rust_static_library` |
| ArkTS/TypeScript | 完整支持 | `ohos_abc` (编译为 Ark Bytecode) |
| 仓颉 (Cangjie) | 完整支持 | `ohos_cangjie_shared_library`, `ohos_cangjie_static_library` |
| Java | 支持 | Android 应用构建 |

### 2. 多架构支持

支持 7 种目标架构 (`//build/toolchain/ohos/BUILD.gn`):

| 架构 | ABI Target | 库目录 |
|-----|------------|--------|
| arm | armv7-unknown-linux-ohos | usr/lib/arm-linux-ohos |
| arm64 | aarch64-unknown-linux-ohos | usr/lib/aarch64-linux-ohos |
| x86_64 | x86_64-unknown-linux-ohos | usr/lib/x86_64-linux-ohos |
| riscv64 | riscv64-unknown-linux-gnu | usr/lib/riscv64-linux-ohos |
| loongarch64 | loongarch64-linux-ohos | usr/lib/loongarch64-linux-ohos |
| mipsel | mipsel-unknown-linux-gnu | usr/lib/mipsel-linux-ohos |

### 3. 产物打包能力

- **系统镜像**: system, vendor, userdata, ramdisk
- **SDK**: 支持 Windows/Linux/macOS/OHOS 多平台
- **NDK**: Native Development Kit
- **HAP**: HarmonyOS Ability Package

### 4. 构建工具链

- **编译器**: Clang 15.0.4 (默认), GCC, ICCARM
- **链接器**: LLD
- **C 库**: musl libc
- **C++ 标准**: C++17

## 项目边界

### 包含范围

```
build/
├── 构建脚本 (build_scripts/)
├── 构建配置 (config/, toolchain/)
├── 构建模板 (templates/)
├── hb 构建工具 (hb/)
├── OpenHarmony 特定配置 (ohos/)
├── 工具脚本 (scripts/)
└── 轻量系统支持 (lite/)
```

### 不包含范围

- 具体产品配置（在 vendor/ 目录）
- 子系统源码（在 foundation/, base/ 等目录）
- 内核源码（在 kernel/ 目录）
- 第三方库源码（在 third_party/ 目录）

## 运行环境

### 主机要求

- **操作系统**: Ubuntu 18.04+ (推荐 20.04/22.04)
- **Python**: 3.8+
- **内存**: 建议 16GB+
- **磁盘**: 建议 200GB+ 可用空间

### 依赖工具

```bash
sudo apt-get install bison ccache default-jdk flex \
    gcc-arm-linux-gnueabi gcc-arm-none-eabi genext2fs \
    liblz4-tool libssl-dev libtinfo5 mtd-utils mtools \
    openssl ruby scons unzip u-boot-tools zip
```

## 关键概念

### 1. 子系统 (Subsystem)

OpenHarmony 按功能划分的顶层模块，如 `arkui`, `communication`, `multimedia` 等。

配置位置: `//build/subsystem_config.json`

### 2. 部件 (Part)

子系统的组成部分，定义在 `bundle.json` 或 `ohos.build` 中。

示例 (`//build/bundle.json`):
```json
{
  "component": {
    "name": "build_framework",
    "subsystem": "build",
    "build": {
      "sub_component": ["//build/common:common_packages"]
    }
  }
}
```

### 3. 模块 (Module)

最小的构建单元，使用 GN 模板定义，如 `ohos_shared_library("mylib")`。

### 4. 构建阶段 (Build Phase)

```
PRE_BUILD → PRE_LOAD → LOAD → PRE_TARGET_GENERATE → TARGET_GENERATE (gn gen)
→ POST_TARGET_GENERATE → PRE_TARGET_COMPILATION → TARGET_COMPILATION (ninja)
→ POST_TARGET_COMPILATION → POST_BUILD
```

## 快速开始

### 全量编译

```bash
./build.sh --product-name {product_name}
```

### 常用选项

```bash
--target-cpu=TARGET_CPU        # arm, arm64, x86_64
--build-target=BUILD_TARGET    # 指定编译目标
--gn-args=GN_ARGS             # 传递 GN 参数
--ninja-args=NINJA_ARGS       # 传递 Ninja 参数
--ccache                      # 启用 ccache
--jobs=JOBS                   # 编译线程数
--log-level=LOG_LEVEL         # debug, info, error
```

### 编译输出

```
out/{device_name}/
├── packages/phone/images/     # 系统镜像
├── sdk/                       # SDK 输出
└── build_configs/             # 构建配置
```

## 相关文档

- [目录结构](02_Directory_Structure.md) - 代码组织详解
- [构建系统详解](04_Build_System.md) - GN/Ninja 工作原理
- [GN Targets 梳理](05_GN_Targets.md) - 模板使用指南

## 相关仓库

- [build_lite](https://gitee.com/openharmony/build_lite) - 轻量系统构建
- [docs](https://gitee.com/openharmony/docs) - 官方文档

---

*文档生成时间: 2025-02-06*

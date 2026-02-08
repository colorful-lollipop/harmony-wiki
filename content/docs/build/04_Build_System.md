# 构建系统详解

本文档详细介绍 OpenHarmony 构建系统的工作原理，包括 GN/Ninja 构建流程、工具链配置和构建原理。

## 构建系统概述

OpenHarmony 构建系统采用 **GN (Generate Ninja) + Ninja** 的经典组合：

1. **GN**: 元构建系统，读取 `.gn` 和 `.gni` 文件，生成 Ninja 构建文件
2. **Ninja**: 高性能构建系统，执行实际的编译链接操作
3. **hb**: OpenHarmony 构建工具，封装 GN/Ninja 调用流程

## GN 构建系统

### GN 简介

GN (Generate Ninja) 是 Chromium 项目开发的元构建系统，特点：
- **声明式**: 描述构建目标，而非构建步骤
- **快速**: 生成 Ninja 文件速度快
- **可读**: 类似 Python 的语法
- **可维护**: 支持模板和导入

### GN 文件类型

| 扩展名 | 用途 | 示例 |
|--------|------|------|
| `.gn` | 构建定义 | `BUILD.gn`, `.gn` |
| `.gni` | 包含文件 | `ohos.gni`, `cxx.gni` |

### GN 核心概念

#### 1. Target (目标)

构建的基本单元，常见类型：

```gn
# 可执行文件
executable("my_app") {
  sources = [ "main.cc" ]
}

# 共享库
shared_library("my_lib") {
  sources = [ "lib.cc" ]
}

# 静态库
static_library("my_static") {
  sources = [ "static.cc" ]
}

# 源码集合
source_set("my_sources") {
  sources = [ "a.cc", "b.cc" ]
}

# 分组
group("all") {
  deps = [ ":my_app", ":my_lib" ]
}
```

#### 2. Template (模板)

OpenHarmony 定义了大量自定义模板：

```gn
# C/C++ 模板 (//build/templates/cxx/cxx.gni)
ohos_shared_library("my_lib") {
  sources = [ "my_lib.cpp" ]
  part_name = "my_part"
}

ohos_executable("my_app") {
  sources = [ "main.cpp" ]
  deps = [ ":my_lib" ]
  part_name = "my_part"
}

# Rust 模板 (//build/templates/rust/rust_template.gni)
ohos_rust_executable("my_rust_app") {
  sources = [ "main.rs" ]
  crate_name = "my_rust_app"
}
```

#### 3. Scope (作用域)

GN 使用 `{}` 定义作用域：

```gn
config("my_config") {
  defines = [ "MY_DEFINE" ]
  include_dirs = [ "//include" ]
}

ohos_shared_library("my_lib") {
  configs = [ ":my_config" ]
}
```

### GN 变量和函数

#### 预定义变量

```gn
# 路径变量
root_build_dir    # 构建输出目录
root_out_dir      # 输出目录
target_os         # 目标操作系统 ("ohos", "android", "linux")
target_cpu        # 目标 CPU ("arm", "arm64", "x86_64")

# 当前上下文
target_name       # 当前目标名
target_gen_dir    # 当前目标生成目录
```

#### 常用函数

```gn
# 声明参数
declare_args() {
  my_flag = true
}

# 条件判断
if (target_os == "ohos") {
  defines += [ "OHOS" ]
}

# 循环
foreach(source, sources) {
  # 处理每个 source
}

# 执行脚本
result = exec_script("my_script.py", [ arg1, arg2 ], "value")
```

### GN 命令

```bash
# 生成构建文件
gn gen out/my_build

# 带参数生成
gn gen out/my_build --args='target_os="ohos" target_cpu="arm64"'

# 查看目标
gn ls out/my_build

# 查看目标详情
gn desc out/my_build //path/to:target

# 查找依赖路径
gn path out/my_build //from:target //to:target

# 查找反向依赖
gn refs out/my_build //path/to:target

# 格式化 GN 文件
gn format //path/to/BUILD.gn

# 清理
gn clean out/my_build
```

## Ninja 构建系统

### Ninja 简介

Ninja 是 Google 开发的高性能构建系统，特点：
- **速度快**: 专注于执行速度
- **并行**: 自动并行化构建任务
- **增量**: 只构建变更的文件
- **简洁**: 构建文件由 GN 生成，人工不直接编辑

### Ninja 文件结构

```ninja
# 变量定义
cc = clang
cxx = clang++

# 规则定义
rule cc
  command = $cc $cflags -c $in -o $out
  description = CC $out

rule link
  command = $cxx $ldflags $in -o $out
  description = LINK $out

# 构建目标
build obj/main.o: cc src/main.cpp
build obj/lib.o: cc src/lib.cpp
build my_app: link obj/main.o obj/lib.o

# 默认目标
default my_app
```

### Ninja 命令

```bash
# 构建默认目标
ninja -C out/my_build

# 构建特定目标
ninja -C out/my_build target_name

# 并行构建
ninja -C out/my_build -j8

# 详细输出
ninja -C out/my_build -v

# 干运行（不实际执行）
ninja -C out/my_build -n

# 查看构建计划
ninja -C out/my_build -t commands target_name
```

## OpenHarmony 构建流程

### 完整构建流程

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: 配置解析                                             │
│  ├── 读取 subsystem_config.json                               │
│  ├── 读取 vendor/{product}/config.json                        │
│  └── 确定目标产品、CPU、OS                                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: 预加载 (Preloader)                                   │
│  ├── 扫描所有子系统的 bundle.json                              │
│  ├── 生成 parts.json (部件清单)                               │
│  ├── 生成 features.json (特性配置)                            │
│  ├── 生成 syscap.json (系统能力)                              │
│  └── 生成 build_config.json (构建配置)                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: 加载 (Loader)                                        │
│  ├── 验证部件依赖                                              │
│  ├── 生成 subsystem_config.gni                                │
│  └── 生成 target_platform.gn                                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: GN 生成                                              │
│  ├── 执行 gn gen                                              │
│  ├── 解析所有 BUILD.gn                                        │
│  ├── 展开模板                                                 │
│  └── 生成 build.ninja                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: Ninja 编译                                           │
│  ├── 解析 build.ninja                                         │
│  ├── 计算依赖图                                               │
│  ├── 并行编译                                                 │
│  └── 生成 .o, .so, .a, executable                            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 6: 打包                                                 │
│  ├── 收集模块到 system/vendor 目录                            │
│  ├── 生成系统镜像 (ext4/f2fs)                                 │
│  └── 可选: 打包 SDK/NDK                                       │
└─────────────────────────────────────────────────────────────┘
```

### 构建配置传递

```
命令行参数
    │
    ├──→ hb/main.py
    │       │
    │       ├──→ Arg.parse_all_args() → Arg 对象
    │       │
    │       ├──→ Config (单例)
    │       │       ├──→ product
    │       │       ├──→ target_cpu
    │       │       └───→ target_os
    │       │
    │       └──→ BuildArgsResolver
    │               │
    │               ├──→ resolve_product()
    │               ├──→ resolve_target_cpu()
    │               └──→ resolve_gn_args()
    │
    └──→ gn gen --args="..."
                │
                ├──→ declare_args() in .gni files
                │
                ├──→ template expansion
                │
                └──→ build.ninja
```

## 工具链配置

### 编译器配置

#### Clang 配置 (`//build/toolchain/toolchain.gni`)

```gn
clang_version = "15.0.4"
toolchains_dir = "//prebuilts/clang/ohos"
clang_base_path = "${toolchains_dir}/${host_platform_dir}/llvm"
```

#### 工具链定义 (`//build/toolchain/ohos/BUILD.gn`)

```gn
# arm64 工具链示例
ohos_clang_toolchain("ohos_clang_arm64") {
  toolchain_cpu = "arm64"
  toolchain_os = "ohos"
  abi_target = "aarch64-unknown-linux-ohos"
}
```

### 编译选项

#### 默认编译标志 (`//build/config/compiler/BUILD.gn`)

```gn
# 安全标志
ldflags += [ "-Wl,-z,noexecstack" ]  # 不可执行栈
ldflags += [ "-Wl,-z,now" ]          # 立即绑定
ldflags += [ "-Wl,-z,relro" ]        # RELRO 保护

# 优化标志
cflags += [ "-fPIC" ]                # 位置无关代码
cflags_cc += [ "-std=c++17" ]        # C++17 标准

# 链接器
ldflags += [ "-fuse-ld=lld" ]        # 使用 LLD
```

#### 架构特定标志 (`//build/config/arm.gni`)

```gn
if (target_cpu == "arm") {
  arm_arch = "armv7-a"
  arm_float_abi = "softfp"
  arm_fpu = "neon"
} else if (target_cpu == "arm64") {
  arm_arch = "armv8-a"
  arm_cpu = "cortex-a55"
  arm_float_abi = "hard"
}
```

## 构建依赖管理

### 内部依赖 (deps)

```gn
ohos_shared_library("my_lib") {
  deps = [
    "//path/to:other_lib",      # 同一仓库内的目标
    "//foundation/arkui/napi:napi",
  ]
}
```

### 外部依赖 (external_deps)

```gn
ohos_shared_library("my_lib") {
  external_deps = [
    "part_name:module_name",     # 跨部件依赖
    "napi:libnapi",
  ]
}
```

依赖解析流程：
1. 解析 `external_deps` 中的部件名
2. 查找部件的 `inner_kits` 定义
3. 映射到实际的 GN 目标路径

## 构建产物

### 编译产物类型

| 类型 | 扩展名 | 模板 | 安装位置 |
|------|--------|------|----------|
| 共享库 | .so | `ohos_shared_library` | system/lib64/ |
| 静态库 | .a | `ohos_static_library` | obj/ (不安装) |
| 可执行文件 | - | `ohos_executable` | system/bin/ |
| Rust 动态库 | .so | `ohos_rust_shared_library` | system/lib64/ |
| Rust 静态库 | .rlib | `ohos_rust_static_library` | obj/ |
| Ark Bytecode | .abc | `ohos_abc` | 应用包内 |

### 镜像产物

| 镜像 | 文件系统 | 大小 | 内容 |
|------|---------|------|------|
| system.img | ext4 | 1.5GB | 系统库、可执行文件 |
| vendor.img | ext4 | 256MB | 芯片厂商库 |
| userdata.img | f2fs | 1.4GB | 用户数据 |
| ramdisk.img | - | - | 启动内存盘 |

## 高级主题

### Sanitizer 支持

```gn
# Address Sanitizer
declare_args() {
  is_asan = false
}

# 在模板中使用
if (defined(sanitize) && sanitize != []) {
  # 应用 sanitizer 标志
}
```

### 组件构建

```gn
declare_args() {
  is_component_build = false  # 设为 true 启用组件构建
}
```

### 独立编译

```bash
# 独立编译模式
hb indep_build --part part_name
```

## 调试构建

### 调试选项

```bash
# 仅 GN 生成，不编译
./build.sh --product-name xxx --build-only-gn

# 详细输出
./build.sh --product-name xxx --verbose

# 指定日志级别
./build.sh --product-name xxx --log-level debug

# 保留 Ninja 继续执行（即使出错）
./build.sh --product-name xxx --keep-ninja-going
```

### 分析构建

```bash
# 查看 GN 目标
gn ls out/my_build

# 查看目标详情
gn desc out/my_build //path/to:target

# 查看 Ninja 构建图
ninja -C out/my_build -t graph | dot -Tpng > build.png

# 查看编译时间
ninja -C out/my_build -d stats
```

---

*文档生成时间: 2025-02-06*

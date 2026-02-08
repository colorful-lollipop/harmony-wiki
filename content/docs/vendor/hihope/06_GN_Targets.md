# GN Targets 与构建系统

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库中的 GN 构建文件、Targets 结构、依赖关系和编译产物
- **适用范围**：vendor_hihope 仓库中的 BUILD.gn 和 .gni 文件
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库包含 103 个 BUILD.gn 文件和 35 个 .gni 文件
  - 所有标准产品（9 个）采用一致的构建结构
  - rk3568 产品包含最完整的构建配置

## GN 构建系统概述

### GN 概述

GN（Generate Ninja）是 OpenHarmony 使用的构建系统，基于 Chromium GN 构建系统。

**核心概念**：
- **Target（目标）**：构建的基本单元（可执行文件、库、配置等）
- **Dependence（依赖）**：Targets 之间的关系
- **Toolchain（工具链）**：编译器和链接器
- **Output（输出）**：构建产物（.so, .a, .hap 等）

## vendor_hihope 仓库构建统计

### 总体统计

| 文件类型 | 数量 | 说明 |
|---------|------|------|
| **BUILD.gn** | 103 | 所有产品的构建文件 |
| **.gni** | 35 | 配置和模板文件 |
| **总计** | 138 | GN 相关文件 |

### 按产品分类

| 产品 | BUILD.gn 数量 | .gni 数量 | 说明 |
|------|---------------|----------|------|
| **2in1_core_system** | 9 | 4 | 标准产品 |
| **dayu210** | 10 | 4 | 标准产品 |
| **default_core_system** | 10 | 4 | 标准产品 |
| **ipcamera_core_system** | 10 | 4 | 标准产品 |
| **rk3568** | 10 | 4 | 最完整的标准产品 |
| **rk3568_mini_system** | 4 | 3 | 精简标准产品 |
| **tablet_core_system** | 10 | 4 | 标准产品 |
| **tv** | 9 | 4 | 标准产品 |
| **wearable** | 10 | 4 | 标准产品 |
| **nearlink_dk_3863** | 43 | 0 | IoT 产品（含教程） |
| **neptune_iotlink_demo** | 5 | 0 | IoT 产品 |
| **nearlink_dk_3863_xts** | 1 | 0 | 测试产品 |

## 标准产品构建结构

### 典型 BUILD.gn 结构（以 rk3568 为例）

**根 BUILD.gn**：`rk3568/BUILD.gn`

```gn
import("//build/ohos.gni")

group("rk3568") {
  deps = [
    # 功能区域依赖
    ":product_rk3568",           # 产品配置
    ":hdf_audio_config",         # HDF 音频配置
    ":hdf_codec_config",          # HDF 编解码配置
    ":default_app_config",         # 默认应用
    ":preinstall_config",           # 预安装配置
    ":resourceschedule",           # 资源调度
    ":hdf_config",               # HDF 配置
    ":product_etc",               # 系统配置
    ":window_config",              # 窗口配置
    ":security_config",            # 安全配置
  ]
}
```

### 典型子目录 BUILD.gn

#### bluetooth/BUILD.gn

**职责**：蓝牙模块构建配置

```gn
import("//build/ohos.gni")

group("bluetooth") {
  deps = [
    # 依赖蓝牙 HAL
    "//drivers/peripheral/bluetooth/hal:libbluetooth_hal.z.so",
    "//foundation/communication/bluetooth_server:libbluetooth_server.z.so",
  ]
}
```

#### hals/audio/BUILD.gn（音频 HAL 构建）

**职责**：音频 HAL 配置文件构建

**Target 类型**（证据：`rk3568/hals/audio/BUILD.gn`）：

| Target 名称 | 类型 | 产物 | 说明 |
|----------|------|------|------|
| `hdf_alsa_paths_json` | ohos_prebuilt_etc | ALSA 路径配置 |
| `hdf_alsa_adapter_json` | ohos_prebuilt_etc | ALSA 适配器配置 |
| `hdf_audio_effect_json` | ohos_prebuilt_etc | 音效配置 |
| `hdf_audio_path_json` | ohos_prebuilt_etc | 音频路径配置 |
| `hdf_audio_adapter_json` | ohos_prebuilt_etc | 音频适配器配置 |
| `audio_policy_config` | ohos_prebuilt_etc | 音频策略配置（arm/arm64） |
| `audio_policy_config_new` | ohos_prebuilt_etc | 新音频策略配置 |

**配置参数**：
- `install_images = [chipset_base_dir]` - 安装到 chipset 基础目录
- `relative_install_dir = "audio"` - 相对安装路径
- `subsystem_name = "product_rk3568"` - 子系统名称
- `part_name = "product_rk3568"` - 部件名称

#### hals/codec/BUILD.gn（编解码器 HAL 构建）

**职责**：编解码器 HAL 构建配置

**Target 类型**：
- 通常为 group 类型，不直接产出可执行文件或库
- 聚合其他配置 targets

#### hals/codec/BUILD.gn（编解码器 HAL 构建）

**职责**：编解码器 HAL 构建配置

**Target 类型**：
- 通常为 group 类型，不直接产出可执行文件或库
- 聚合其他配置 targets

#### default_app_config/BUILD.gn（默认应用配置）

**职责**：默认应用列表构建

**Target 类型**：`ohos_prebuilt_etc` - 预配置文件

#### preinstall-config/BUILD.gn（预安装配置）

**职责**：预安装应用配置构建

**Target 类型**（证据：`rk3568/preinstall-config/BUILD.gn`）：
- `install_list` - 应用列表
- `install_list_permissions` - 权限列表
- `install_list_capability` - 能力列表

#### resourceschedule/BUILD.gn（资源调度配置）

**职责**：资源调度配置构建

**子目录 Targets**：
- `cgroup_sched` - Cgroup 调度配置
- `ressched` - 资源调度器配置
- `soc_perf` - SoC 性能配置

#### security_config/BUILD.gn（安全配置）

**职责**：安全配置构建

**Target 类型**（证据：`rk3568/security_config/BUILD.gn`）：
- `critical_reboot_process_list` - 关键重启进程列表
- `high_privilege_process_list` - 高权限进程列表
- `sanitizer_check_list` - Security sanitizer 配置（.gni 文件）

#### hdf_config/uhdf/BUILD.gn（HDF 用户态配置）

**职责**：HDF 用户态配置文件构建

**Target 类型**：`ohos_prebuilt_etc` - HDF 配置文件

#### window_config/BUILD.gn（窗口管理配置）

**职责**：窗口管理器配置构建

**Target 类型**：`ohos_prebuilt_etc` - 窗口配置文件

## OpenHarmony GN 模板

### ohos_* 模板

vendor_hihope 仓库使用 OpenHarmony 定义的 GN 模板，而非原生 GN 模板。

| OpenHarmony 模板 | 原生 GN 模板 | 用途 |
|-----------------|---------------|------|
| `ohos_shared_library` | `shared_library` | 共享库，支持 external_deps |
| `ohos_static_library` | `static_library` | 静态库 |
| `ohos_executable` | `executable` | 可执行文件 |
| `ohos_source_set` | `source_set` | 源文件集合 |
| `ohos_group` | `group` | 目标组 |
| `ohos_prebuilt_etc` | （自定义）预构建配置文件 |
| `ohos_prebuilt_shared_library` | `prebuilt_shared_library` | 预构建共享库 |

### 典型 Target 定义

#### 配置文件 Target

```gn
ohos_prebuilt_etc("audio_policy_config") {
  source = "config/arm/audio_policy_config.xml"
  relative_install_dir = "audio"
  install_images = [chipset_base_dir]
  subsystem_name = "product_rk3568"
  part_name = "product_rk3568"
}
```

**关键参数**：
- `source` - 源文件路径
- `relative_install_dir` - 相对安装目录
- `install_images` - 安载目标镜像
- `subsystem_name` - 子系统名称
- `part_name` - 部件名称

#### 库 Target

```gn
ohos_shared_library("my_lib") {
  sources = ["my_lib.cpp"]
  include_dirs = ["include"]
  deps = [
    "//foundation/ability/ability_runtime:ability_runtime",
    "//utils/native/base:base",
  ]
  external_deps = [
    "hilog:libhilog.z.so",
    "napi:libace_napi.z.so",
  ]
  subsystem_name = "my_subsystem"
  part_name = "my_part"
}
```

**关键参数**：
- `sources` - 源文件列表
- `include_dirs` - 头文件包含目录
- `deps` - 依赖（同 part 内）
- `external_deps` - 依赖（跨 part）
- `subsystem_name` - 子系统名称
- `part_name` - 部件名称

## .gni 配置文件

### product.gni（产品配置）

**职责**：定义产品级变量和公共配置

**典型内容**（证据：`rk3568/product.gni`）：
```gni
# 产品信息
product_name = "rk3568"
product_config_dir = "//device/board/hihope/rk3568"

# 构建标志
enable_ohos_framework_netmanager = true
enable_ohos_framework_wifi = true
```

### sanitizer_check_list.gni（安全 sanitizer 配置）

**职责**：定义 Security sanitizer 的 CFI bypass 模块列表

**典型内容**（证据：`rk3568/security_config/sanitizer_check_list.gni`）：
```gni
# Security sanitizer 模块
sanitizer_check_list = [
  "socket_permission",
  "power_permission",
  "uri_permission_mgr",
  "dlp_permission_service",
  "ohdlp_permission",
  "selinux_adapter",
]
```

### build_cfg.gni（更新构建配置）

**职责**：定义 OTA 更新相关的构建配置

## 构建产物说明

### 主要产物类型

| 产物类型 | 扩展名 | 生成位置 | 安装路径 |
|----------|---------|----------|------|
| **共享库** | .so | out/{product}/libs/ | /usr/lib/ |
| **静态库** | .a | out/{product}/obj/ | 链接到应用 |
| **配置文件** | .json, .xml | out/{product}/etc/ | /etc/ |
| **镜像** | .bin, .img | out/{product}/packages/ | / |
| **HAP 包** | .hap | out/{product}/packages/phone/ | /data/app/ |
| **可执行文件** | 无（标准系统） | - | - |

### 产品特定产物

#### rk3568 产物

**配置文件产物**（证据：`rk3568/hals/audio/BUILD.gn`）：

| Target | 产物 | 安装路径 |
|--------|------|---------|
| `hdf_alsa_paths_json` | hdfconfig/alsa_paths.json | /etc/hdfconfig/ |
| `hdf_alsa_adapter_json` | hdfconfig/alsa_adapter.json | /etc/hdfconfig/ |
| `hdf_audio_effect_json` | hdfconfig/audio_effect.json | /etc/hdfconfig/ |
| `hdf_audio_path_json` | hdfconfig/audio_paths.json | /etc/hdfconfig/ |
| `audio_policy_config` | audio_policy_config.xml | /etc/audio/ |
| `audio_policy_config_new` | audio_policy_config_new.xml | /etc/audio/ |

#### nearlink_dk_3863 产物

**教程可执行文件**（证据：`nearlink_dk_3863/ws63_sample/*/BUILD.gn`）：
- 28 个教程项目各自产生可执行文件
- 安装路径：通常在 /data/ 或 /usr/bin/

#### neptune_iotlink_demo 产物

**系统镜像**：生成 OpenHarmony LiteOS-M 系统镜像

## 构建命令

### hb（Harmony Build）命令

```bash
# 设置产品
hb set -p {product_path}
hb set -p {product_path} {product_name}

# 选择产品交互式
hb set

# 编译产品
hb build
hb build -f {target_name}           # 编译指定 target
hb build --ccache              # 使用编译缓存
hb build --build-ninja         # 使用 Ninja 构建
hb build --fast-rebuild        # 快速重新编译

# 清理
hb clean
hb clean {target_name}
hb clean --build-ninja
```

### GN 构建命令（直接使用 GN）

```bash
# 生成 Ninja 文件
gn gen out/{product}

# 编译
ninja -C out/{product}

# 格式化
gn format path/to/BUILD.gn
```

## 依赖关系

### deps vs external_deps

| 依赖类型 | 作用 | 作用域 | 示例 |
|---------|------|--------|------|
| `deps` | 同 part 内依赖 | 同一 part | "//foundation/..." |
| `external_deps` | 跨 part 依赖 | 跨不同 part | "hilog:libhilog.z.so" |

### 依赖方向原则

1. **单向依赖**：上层依赖下层，避免循环
2. **明确声明**：所有依赖必须在 deps/external_deps 中声明
3. **传递性**：依赖的 deps 会被传递给依赖者
4. **避免过度依赖**：仅依赖实际需要的 targets

## 构建优化

### 编译缓存

- ✅ 使用 `hb build --ccache` 启用编译缓存
- ✅ 使用 `--fast-rebuild` 快速重新编译未变更的 target
- ⚠️ 缓存位置：通常在 `~/.hb-cache` 或 `.cache/`

### 增量编译

- ✅ GN 只重新编译变更的 targets
- ✅ Ninja 按需重新编译
- 📝 文件变更检测：时间戳比较

### 并行构建

- ✅ GN 默认支持并行分析
- ✅ Ninja 支持并行编译（-j 参数）
- 🎯 CPU 核心数：通常设置为逻辑核心数

## 构建调试

### 构建日志

```bash
# 详细构建日志
hb build --verbose
hb build -v -v

# Ninja 详细输出
ninja -C out/{product} -v
```

### GN 调试命令

```bash
# 查看依赖关系
gn desc out/{product} path/to/target

# 检查依赖
gn check path/to/BUILD.gn

# 输出依赖图
gn graph path/to/BUILD.gn --output=out/graph.dot
```

### 常见构建问题

| 问题 | 可能原因 | 解决方法 |
|------|---------|---------|
| **Target 未找到** | 路径错误或名称拼写错误 | 检查 BUILD.gn 中的路径 |
| **依赖缺失** | deps 中引用的 target 不存在 | 添加缺失的依赖或 external_deps |
| **循环依赖** | A 依赖 B，B 依赖 A | 重新组织依赖关系 |
| **编译错误** | 语法错误或头文件缺失 | 修复语法或添加 include_dirs |
| **链接错误** | 未定义符号或库缺失 | 检查 deps/external_deps |
| **安装失败** | 权限不足或路径不存在 | 检查安装路径和权限 |

## 相关跳转

- [返回 Wiki 首页](SUMMARY.md)
- [OpenHarmony 构建文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-gn-coding-style-and-best-practice.md)

# 目录结构

本文档详细说明 OpenHarmony build 仓库的目录结构和各模块职责。

## 顶层目录

```
build/                           # 编译构建主目录
├── build_scripts/               # 编译入口脚本
├── common/                      # 通用模块
├── config/                      # 编译配置
├── core/                        # 核心构建逻辑
├── docs/                        # 文档
├── hb/                          # hb 构建工具
├── lite/                        # 轻量系统构建
├── misc/                        # 杂项
├── ohos/                        # OpenHarmony 特定配置
├── rust/                        # Rust 支持
├── scripts/                     # 工具脚本
├── templates/                   # 编译模板
├── toolchain/                   # 工具链配置
├── tools/                       # 常用工具
├── bundle.json                  # HPM 部件配置
├── ohos.gni                     # 主 GNI 入口
├── ohos_var.gni                 # 构建变量
├── subsystem_config.json        # 子系统配置
└── test.gni                     # 测试配置
```

## 详细说明

### build_scripts/ - 编译入口脚本

**职责**: 提供编译命令入口

| 文件 | 说明 |
|------|------|
| `build.sh` | Shell 编译入口，支持所有编译选项 |
| `build.py` | Python 包装器，查找仓库根目录并调用 hb |
| `env_setup.sh` | Docker 环境设置脚本 |

**调用链**:
```
build.sh → tools_checker.py → hb/main.py → OHOSBuildModule
```

### common/ - 通用模块

**职责**: 提供通用构建功能

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 通用包定义 |

### config/ - 编译配置

**职责**: 定义编译器选项、架构配置、功能开关

| 文件/目录 | 说明 |
|-----------|------|
| `BUILDCONFIG.gn` | 主构建配置 (1196行) |
| `config.gni` | 配置入口 |
| `compiler/` | 编译器详细配置 |
| `clang/` | Clang 特定配置 |
| `ohos/` | OHOS 平台配置 |
| `sanitizers/` | Sanitizer 配置 |
| `arm.gni` | ARM 架构配置 |

**关键配置**:
- 编译器标志: `//build/config/compiler/BUILD.gn`
- 架构定义: `//build/config/arm.gni`
- 安全检查: `//build/config/sanitizers/sanitizers.gni`

### core/ - 核心构建逻辑

**职责**: GN 配置和脚本执行白名单

| 文件/目录 | 说明 |
|-----------|------|
| `gn/BUILD.gn` | GN 构建配置 |
| `gn/dotfile.gn` | GN dotfile 配置 |
| `gn/ohos_exec_script_allowlist.gni` | 脚本执行白名单 |
| `build_scripts/verify_notice.sh` | 声明文件验证脚本 |

### hb/ - hb 构建工具

**职责**: OpenHarmony 命令行构建工具

```
hb/
├── main.py                    # 主入口
├── setup.py                   # 包安装配置
├── containers/                # 数据容器
│   ├── arg.py                # 参数定义、构建阶段枚举
│   └── status.py             # 状态管理
├── modules/                   # 功能模块
│   ├── ohos_build_module.py  # 构建模块
│   ├── ohos_set_module.py    # 配置模块
│   ├── ohos_clean_module.py  # 清理模块
│   └── ...                   # 其他模块
├── resolver/                  # 参数解析器
│   ├── build_args_resolver.py
│   └── ...
├── resources/                 # 资源配置
│   ├── config.py             # 配置单例
│   └── args/default/         # 参数定义 JSON
├── services/                  # 核心服务
│   ├── gn.py                 # GN 服务
│   ├── ninja.py              # Ninja 服务
│   ├── preloader.py          # 预加载服务
│   └── loader.py             # 加载服务
└── util/                      # 工具函数
```

**支持的命令**:
- `hb build` - 构建
- `hb set` - 设置产品/板卡
- `hb clean` - 清理
- `hb env` - 环境检查
- `hb tool` - GN 工具操作
- `hb indep_build` - 独立构建
- `hb install/package/publish/update/push` - HPM 相关

### lite/ - 轻量系统构建

**职责**: 轻量/小型系统 (mini/small) 构建支持

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 轻量系统构建入口 |
| `ohos_var.gni` | 轻量系统变量 |
| `lite_target_list.gni` | 目标列表 |

### ohos/ - OpenHarmony 特定配置

**职责**: OpenHarmony 特定的构建、打包流程

```
ohos/
├── ace/                       # ACE (ArkUI) 构建
├── app/                       # HAP 应用构建
├── common/                    # 通用工具
├── hisysevent/               # HiSysEvent 配置
├── images/                    # 镜像制作
├── kits/                      # Kits 接口检查
├── native_stub/              # Native Stub
├── ndk/                       # NDK 构建
├── notice/                    # 开源声明处理
├── packages/                  # 系统打包
├── sa_profile/               # SA 配置处理
├── sdk/                       # SDK 构建
├── sbom/                      # SBOM 物料清单
├── taihe_idl/                # Taihe IDL
├── ohos_kits.gni             # InnerKits 模板
├── ohos_part.gni             # 部件模板
└── ohos_test.gni             # 测试配置
```

#### ohos/images/ - 镜像制作

| 镜像 | 配置 | 大小 | 文件系统 |
|------|------|------|---------|
| system | `system_image_conf.txt` | 1.5GB | ext4 |
| vendor | `vendor_image_conf.txt` | 256MB | ext4 |
| userdata | `userdata_image_conf.txt` | 1.4GB | f2fs |
| ramdisk | `ramdisk_image_conf.txt` | - | - |

#### ohos/ndk/ - NDK 构建

- `ndk.gni` - NDK 模板定义 (616行)
- `BUILD.gn` - NDK 构建主配置 (440行)
- `cmake/` - CMake 工具链配置

#### ohos/sdk/ - SDK 构建

- `sdk.gni` - SDK 模板定义 (406行)
- `BUILD.gn` - SDK 构建主配置 (315行)
- `ohos_sdk_description_std.json` - SDK 模块清单 (1673行)

### rust/ - Rust 支持

**职责**: Rust 工具链配置

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | Rust std 库预构建目标 |
| `rustc_toolchain.gni` | Rust 工具链配置 |

### scripts/ - 工具脚本

**职责**: 构建辅助工具

| 脚本 | 功能 |
|------|------|
| `compile_app.py` | 编译应用 |
| `build_js_assets.py` | 构建 JS 资源 |
| `generate_js_bytecode.py` | 生成 JS 字节码 |
| `hapbuilder.py` | HAP 构建 |
| `app_sign.py` | 应用签名 |
| `cargo2gn.py` | Cargo 到 GN 转换 |
| `code_release.py` | 代码发布 |

### templates/ - 编译模板

**职责**: 定义各种语言的构建模板

```
templates/
├── abc/                       # Ark Bytecode
├── bpf/                       # eBPF
├── cangjie/                   # 仓颉语言
├── common/                    # 通用模板
├── cxx/                       # C/C++
├── idl/                       # IDL
├── kernel/                    # 内核
├── metadata/                  # 元数据
├── rust/                      # Rust
└── update/                    # 更新
```

#### templates/cxx/ - C/C++ 模板

| 模板 | 文件 | 行号 |
|------|------|------|
| `ohos_executable` | `cxx.gni` | 38 |
| `ohos_shared_library` | `cxx.gni` | 574 |
| `ohos_static_library` | `cxx.gni` | 1438 |
| `ohos_source_set` | `cxx.gni` | 1773 |
| `ohos_prebuilt_*` | `prebuilt.gni` | - |

#### templates/rust/ - Rust 模板

| 模板 | 文件 | 行号 |
|------|------|------|
| `ohos_rust_executable` | `rust_template.gni` | 263 |
| `ohos_rust_shared_library` | `rust_template.gni` | 356 |
| `ohos_rust_static_library` | `rust_template.gni` | 449 |
| `ohos_rust_shared_ffi` | `rust_template.gni` | 538 |
| `ohos_rust_static_ffi` | `rust_template.gni` | 629 |
| `ohos_rust_proc_macro` | `rust_template.gni` | 718 |
| `rust_bindgen` | `rust_bindgen.gni` | 16 |
| `rust_cxx` | `rust_cxx.gni` | 14 |

### toolchain/ - 工具链配置

**职责**: 定义编译工具链

| 文件/目录 | 说明 |
|-----------|------|
| `BUILD.gn` | 工具链目标定义 |
| `toolchain.gni` | 工具链基础配置 |
| `gcc_toolchain.gni` | GCC/Clang 工具链模板 (895行) |
| `ohos/` | OpenHarmony 工具链 |
| `linux/` | Linux 主机工具链 |
| `mac/` | Mac 主机工具链 |
| `mingw/` | MinGW 工具链 |

**支持的 OHOS 工具链** (`//build/toolchain/ohos/BUILD.gn`):
- `ohos_clang_arm`
- `ohos_clang_arm64`
- `ohos_clang_x86_64`
- `ohos_clang_riscv64`
- `ohos_clang_loongarch64`
- `ohos_clang_mipsel`

## 关键文件索引

### 入口文件

| 文件 | 用途 |
|------|------|
| `build_scripts/build.sh` | Shell 编译入口 |
| `hb/main.py` | hb 工具入口 |
| `ohos.gni` | GN 模板主入口 |

### 配置文件

| 文件 | 用途 |
|------|------|
| `bundle.json` | HPM 部件配置 |
| `subsystem_config.json` | 子系统映射 |
| `ohos_var.gni` | 构建变量 |
| `common.gni` | 通用配置 |

### 模板文件

| 文件 | 用途 |
|------|------|
| `templates/cxx/cxx.gni` | C/C++ 模板 |
| `templates/rust/rust_template.gni` | Rust 模板 |
| `ohos/app/app.gni` | 应用模板 |
| `ohos/ndk/ndk.gni` | NDK 模板 |
| `ohos/sdk/sdk.gni` | SDK 模板 |

---

*文档生成时间: 2025-02-06*

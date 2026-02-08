# 项目概览

> **阅读时间**: 5 分钟 | **目标**: 快速理解 TEE SDK 是什么、能做什么

---

## 一句话定义

**tee_dev_kit** 是 OpenHarmony TEE (Trusted Execution Environment) 的官方 SDK 开发工具包，为开发者提供完整的可信应用 (Trusted Application, TA) 开发工具链，包括编译框架、签名工具、配置管理和示例代码。

**证据**: `README.md:6` - "The TEE SDK development kit supports independent development of TA"

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| **TEE** | Trusted Execution Environment | 可信执行环境，提供硬件隔离的安全执行区域，运行在主处理器安全扩展中 |
| **TA** | Trusted Application | 可信应用，运行在 TEE 中的安全应用，处理敏感数据和加密操作 |
| **CA** | Client Application | 客户端应用，运行在 REE (Rich Execution Environment) 中，通过 TEE Client API 与 TA 通信 |
| **SDK** | Software Development Kit | 软件开发工具包 |
| **GP TEE** | GlobalPlatform TEE | 国际标准化组织定义的 TEE 接口规范，本 SDK 遵循此标准 |

**证据**: `README.md:8-12` - 术语定义表

---

## 能力边界

### ✅ 提供的能力

| 能力 | 说明 | 证据位置 |
|------|------|---------|
| **TA 头文件** | GP TEE 标准接口头文件、OpenTrustee 扩展 API | `sysroot/usr/include/` |
| **编译框架** | CMake、Make、GN 三种构建系统支持 | `sdk/build/cmake/`, `sdk/build/mk/`, `sdk/build/BUILD.gn` |
| **签名工具** | TA 镜像签名生成、密钥管理、算法配置 | `sdk/build/script/signtool_sec.py:1-100` |
| **配置管理** | XML 配置解析、Manifest 生成、权限配置 | `sdk/build/script/manifest.py:1-200` |
| **示例代码** | 完整的 CA 和 TA 配对示例 | `sdk/src/CA/`, `sdk/src/TA/` |
| **链接脚本** | 32位/64位 TA 链接配置 | `sdk/build/ld/ta_link.ld`, `sdk/build/ld/ta_link_64.ld` |

### ❌ 不提供的能力

| 不提供 | 说明 |
|-------|------|
| **TEE 操作系统** | 内核代码在 `base/tee/tee_os_framework` 独立仓库 |
| **运行时环境** | TEE 运行时由 OpenTrustee OS 提供 |
| **N-API 接口** | 本项目为交叉编译工具包，不涉及 JavaScript 接口 |
| **设备端调试工具** | 调试工具在单独的 debug 仓库 |

**证据**: `README.md:14-18` - 项目边界说明

---

## 核心能力详解

### 1. TA 编译框架

**证据**: `sdk/build/mk/README.md:1-20`, `sdk/build/cmake/README.md:1-30`

支持三种构建系统，满足不同开发场景：

#### 构建系统对比

| 构建系统 | 适用场景 | 配置方式 | 证据位置 |
|---------|---------|---------|---------|
| **CMake** | 跨平台项目、现代 C++ 项目 | `CMakeLists.txt` | `sdk/build/cmake/common.cmake:1-50` |
| **Make** | 传统嵌入式项目 | `Makefile` | `sdk/build/mk/common.mk:1-60` |
| **GN** | OpenHarmony 系统构建 | `BUILD.gn` | `sdk/build/BUILD.gn:1-80` |

#### 工具链支持

| 架构 | 工具链 | 标志 | 证据位置 |
|------|-------|------|---------|
| **ARM (32位)** | ARM LLVM/GCC | 无特殊标志 | `sdk/build/cmake/arm_toolchain.cmake:1-30` |
| **AArch64 (64位)** | AArch64 LLVM/GCC | `TARGET_S_SARM64=y` | `sdk/build/cmake/aarch64_toolchain.cmake:1-30` |

**关键配置**: `sdk/build/cmake/common_flags.cmake:1-60` - 定义了安全编译标志（-fpie, -fstack-protector-strong）

### 2. TA 签名工具

**证据**: `sdk/build/script/signtool_sec.py:1-200`, `sdk/build/config/ta_sign_algo_config.ini`

签名工具链是 TEE 安全体系的核心组件：

#### 签名流程

```
输入文件 → 配置解析 → 镜像生成 → Hash 计算 → RSA 签名 → 输出 .sec 包
```

#### 签名配置

**证据**: `sdk/build/config/ta_sign_algo_config.ini:1-20`

```ini
[Sdefault]
sign_algo = RSA           # 签名算法
key_length = 4096         # 密钥长度（支持 2048/3072/4096）
hash_algo = SHA256        # Hash 算法
```

#### 密钥管理

| 密钥类型 | 路径 | 用途 | 安全等级 |
|---------|------|------|---------|
| **调试私钥** | `sdk/build/signkey/ta_sign_priv_key.pem` | 开发调试 | ❌ 仅供调试，禁止生产使用 |
| **验证公钥** | TEE OS 端 `/base/tee/tee_os_framework/lib/syslib/libelf_verify_key/src/common/ta_verify_key.c` | 运行时验证 | ✅ 生产环境需替换 |

**警告**: `README.md:83-85` - "The TEE SDK has a preset private key for signing TA files, which can only be used for debugging."

### 3. TA 配置管理

**证据**: `sdk/build/script/manifest.py:1-150`, `sdk/build/TA_demo/configs.xml:1-30`

#### 配置结构

**证据**: `sdk/build/TA_demo/configs.xml:1-25`

```xml
<ConfigInfo>
  <TA_Basic_Info>
    <service_name>demo-ta</service_name>
    <uuid>e3d37f4a-f24c-48d0-8884-3bdd6c44e988</uuid>
  </TA_Basic_Info>
  <TA_Manifest_Info>
    <instance_keep_alive>false</instance_keep_alive>
    <stack_size>8192</stack_size>
    <heap_size>81920</heap_size>
    <multi_session>false</multi_session>
    <single_instance>true</single_instance>
  </TA_Manifest_Info>
</ConfigInfo>
```

#### 配置项说明

| 配置项 | 类型 | 范围 | 说明 |
|-------|------|------|------|
| `service_name` | String | ≤64 字符 | TA 名称，仅支持数字、字母、`_`、`-` |
| `uuid` | UUID | - | TA 唯一标识符 |
| `stack_size` | Integer | 1024-1048576 | 每会话栈空间（默认 8192 字节） |
| `heap_size` | Integer | 0-10485760 | TA 实例堆空间（默认 0 字节） |
| `single_instance` | Bool | true/false | 是否单实例运行（当前仅支持 true） |

---

## 快速开始

### 步骤 1: 环境准备

```bash
# 1. 克隆 OpenHarmony build 仓库获取 LLVM 工具链
git clone git@gitee.com:openharmony/build.git
cd build
./build/prebuilts_download.sh

# 2. 声明 LLVM 工具链路径
export PATH=openharmony/prebuilts/clang/ohos/linux-x86_64/15.0.4/llvm/bin:$PATH

# 3. 安装 Python 依赖
pip install pycryptodome defusedxml
```

**证据**: `README.md:44-99`

### 步骤 2: 导入第三方头文件

```bash
# 克隆第三方仓库
git clone git@gitee.com:openharmony/third_party_musl.git
git clone git@gitee.com:openharmony/third_party_bounds_checking_function.git

# 执行导入脚本
./tee_dev_kit/sdk/thirdparty/open_source/import_open_source_header.sh
```

**证据**: `README.md:66-79`

### 步骤 3: 编译示例 TA

```bash
# 进入示例目录
cd tee_dev_kit/sdk/build/TA_demo

# 执行一键编译脚本
./build_ta.sh

# 验证输出
ls -la *.sec  # 查看生成的 TA 安装包
```

**证据**: `sdk/build/TA_demo/build_ta.sh:1-100`, `README.md:174-175`

### 预期输出

编译成功后在当前目录生成 `*.sec` 文件：

```
e3d37f4a-f24c-48d0-8884-3bdd6c44e988.sec
```

---

## 运行环境

### 开发环境要求

**证据**: `README.md:40-99`

| 要求 | 说明 | 证据位置 |
|------|------|---------|
| Python 3+ | 构建脚本运行环境 | `README.md:91` |
| LLVM 工具链 | 交叉编译工具 | `README.md:42-59` |
| pycryptodome | 签名工具依赖 | `README.md:96` |
| defusedxml | XML 解析依赖 | `README.md:98` |

### 目标平台

| 平台 | 架构 | 说明 |
|------|------|------|
| OpenHarmony TEE | ARM (32位) | 嵌入式设备 |
| OpenHarmony TEE | AArch64 (64位) | 高性能设备 |

### 头文件依赖

**证据**: `bundle.json:20-23`, `thirdparty/open_source/import_open_source_header.sh`

- `third_party_musl` - C 标准库头文件
- `third_party_bounds_checking_function` - 安全函数库头文件

---

## 目录结构

```
tee_dev_kit/
├── sdk/
│   ├── build/                           # 构建系统核心
│   │   ├── BUILD.gn                    # GN 构建入口
│   │   ├── cmake/                      # CMake 配置
│   │   │   ├── common.cmake           # 通用配置
│   │   │   ├── common_flags.cmake     # 编译标志
│   │   │   ├── arm_toolchain.cmake    # ARM 工具链
│   │   │   ├── aarch64_toolchain.cmake # 64位工具链
│   │   │   └── security_features.cmake # 安全特性
│   │   ├── mk/                        # Make 配置
│   │   │   ├── common.mk             # 通用 Makefile
│   │   │   ├── common_flags.mk       # 编译标志
│   │   │   └── security_features.mk  # 安全特性
│   │   ├── ld/                       # 链接脚本
│   │   │   ├── ta_link.ld            # 32位链接脚本
│   │   │   └── ta_link_64.ld         # 64位链接脚本
│   │   ├── script/                   # 构建脚本
│   │   │   ├── signtool_sec.py       # 签名工具
│   │   │   ├── manifest.py           # 配置解析
│   │   │   ├── build_ta.sh          # 一键编译
│   │   │   └── generate_signature.py # 签名生成
│   │   ├── config/                   # 配置文件
│   │   │   ├── ta_sign_algo_config.ini
│   │   │   └── config_ta_public.ini
│   │   ├── signkey/                  # 密钥目录
│   │   │   └── ta_sign_priv_key.pem
│   │   ├── tools/                    # 辅助工具
│   │   │   └── calc_ca_caller_hash.py
│   │   └── TA_demo/                 # TA 示例
│   │       ├── configs.xml           # TA 配置
│   │       ├── Makefile             # 示例 Makefile
│   │       ├── build_ta.sh          # 编译脚本
│   │       └── ta_demo.c            # 示例源码
│   └── src/                          # 示例源码
│       ├── TA/                       # TA 端代码
│       │   ├── helloworld_demo/
│       │   ├── aes_demo/
│       │   ├── rsa_demo/
│       │   ├── mac_demo/
│       │   └── secstorage_demo/
│       └── CA/                       # CA 端代码
│           ├── helloworld_demo/
│           ├── aes_demo/
│           ├── rsa_demo/
│           ├── mac_demo/
│           └── secstorage_demo/
├── sysroot/
│   └── usr/include/                   # TA 头文件
│       ├── tee_api.h                  # GP TEE 标准 API
│       ├── tee_ext_api.h             # OpenTrustee 扩展
│       └── ...
├── thirdparty/
│   └── open_source/
│       └── import_open_source_header.sh
├── bundle.json                        # 组件配置
├── README.md                          # 项目说明
└── wiki/                             # 本 Wiki
```

---

## 快速导航

### 继续阅读

| 文档 | 阅读时间 | 目标 |
|------|---------|------|
| [01_Architecture.md](01_Architecture.md) | 15 分钟 | 理解 CA ↔ TEE 通信架构 |
| [02_TA_Development_Guide.md](02_TA_Development_Guide.md) | 30 分钟 | 开发第一个 TA |
| [03_Build_System.md](03_Build_System.md) | 20 分钟 | 理解构建和签名流程 |
| [04_Security_Review.md](04_Security_Review.md) | 30 分钟 | 安全风险分析 |
| [05_Examples.md](05_Examples.md) | 20 分钟 | 参考示例代码 |

### 问题排查

遇到问题？请参考 [06_Troubleshooting.md](06_Troubleshooting.md)

---

## 变更历史

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0.0 | 2026-02-07 | 初始版本 |

---

*最后更新: 2026-02-07*

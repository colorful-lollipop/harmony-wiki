# 目录结构

## 目的

本文档介绍 hapsigner 项目的目录组织结构，帮助开发者快速定位代码和理解模块职责。

## 适用范围

- 需要阅读或修改 hapsigner 源码的开发者
- 需要了解模块依赖关系的架构师

---

## 顶层目录结构

```
developtools_hapsigner/
├── autosign/                # 一键签名脚本（Python）
├── binary_sign_tool/        # C++ 二进制签名工具（轻量版）
├── dist/                    # SDK 预配置文件（证书、模板、JAR）
├── figures/                 # 文档图片资源
├── hapsigntool/             # Java 签名工具（完整版）
│   ├── hap_sign_tool/       # CLI 入口模块
│   └── hap_sign_tool_lib/   # 核心库模块
├── hapsigntool_cpp/         # C++ 签名工具（完整版）
├── hapsigntool_cpp_test/    # 测试代码（本文档不覆盖）
├── tools/                   # 自动测试脚本
├── BUILD.gn                 # 根构建文件
├── bundle.json              # OpenHarmony 组件配置
├── LICENSE                  # Apache 2.0 许可证
├── NOTICE                   # 版权声明
├── README.md                # 英文说明文档
└── README_ZH.md             # 中文说明文档
```

---

## hapsigntool (Java 实现)

**路径**: `hapsigntool/`

**职责**: 提供完整的签名工具功能，是 SDK 的主要分发版本。

**构建方式**: Maven

### 子模块结构

```
hapsigntool/
├── pom.xml                          # Maven 父 POM
├── hap_sign_tool/                   # CLI 入口模块
│   ├── pom.xml
│   └── src/main/java/com/ohos/
│       ├── hapsigntool/
│       │   └── HapSignTool.java     # 主入口类
│       ├── hapsigntoolcmd/          # 命令处理
│       │   ├── CmdUtil.java
│       │   ├── HelpDocument.java
│       │   ├── Params.java
│       │   └── ParamsTrustlist.java
│       └── entity/                  # 参数实体
│           ├── RetMsg.java
│           ├── SignAppParameters.java
│           └── VerifyAppParameters.java
│
└── hap_sign_tool_lib/               # 核心库模块
    ├── pom.xml
    └── src/main/java/com/ohos/
        ├── hapsigntool/
        │   ├── api/                 # API 接口层
        │   │   ├── ServiceApi.java
        │   │   └── SignToolServiceImpl.java
        │   ├── cert/                # 证书操作
        │   │   └── CertBuilder.java
        │   ├── codesigning/         # 代码签名
        │   │   ├── fsverity/
        │   │   │   ├── FsVerityGenerator.java
        │   │   │   └── MerkleTreeBuilder.java
        │   │   └── sign/
        │   │       └── CodeSigning.java
        │   ├── entity/              # 实体类
        │   │   ├── ContentDigestAlgorithm.java
        │   │   ├── Options.java
        │   │   └── ParamConstants.java
        │   ├── error/               # 错误处理
        │   │   ├── CustomException.java
        │   │   └── ERROR.java
        │   ├── hap/                 # HAP 签名/验签
        │   │   ├── config/
        │   │   ├── entity/
        │   │   ├── provider/
        │   │   ├── sign/
        │   │   ├── utils/
        │   │   └── verify/
        │   ├── profile/             # Profile 签名/验签
        │   │   ├── ProfileSignTool.java
        │   │   └── VerifyHelper.java
        │   ├── signer/              # 签名器实现
        │   │   ├── ISigner.java
        │   │   ├── LocalSigner.java
        │   │   ├── RemoteSigner.java
        │   │   └── SignerFactory.java
        │   ├── utils/               # 工具类
        │   │   ├── CertUtils.java
        │   │   ├── FileUtils.java
        │   │   └── KeyStoreHelper.java
        │   └── zip/                 # ZIP 操作
        │       └── Zip.java
```

### 关键文件

| 文件路径 | 职责 |
|---------|------|
| `hap_sign_tool/src/main/java/com/ohos/hapsigntool/HapSignTool.java` | Java CLI 主入口，命令分发 |
| `hap_sign_tool_lib/src/main/java/com/ohos/hapsigntool/api/ServiceApi.java` | 服务接口定义 |
| `hap_sign_tool_lib/src/main/java/com/ohos/hapsigntool/api/SignToolServiceImpl.java` | 服务实现 |

---

## hapsigntool_cpp (C++ 完整版)

**路径**: `hapsigntool_cpp/`

**职责**: C++ 实现的完整签名工具，用于系统工具链集成。

**构建方式**: GN/Ninja

### 子模块结构

```
hapsigntool_cpp/
├── BUILD.gn                              # 构建配置
├── signature_tools.gni                   # 路径定义
├── main.cpp                              # 程序入口
│
├── api/                                  # API 层
│   ├── include/
│   │   ├── service_api.h                 # 服务接口
│   │   └── sign_tool_service_impl.h      # 服务实现头文件
│   └── src/
│       ├── sign_tool_service_impl.cpp    # 服务实现
│       └── cert_tools.cpp                # 证书工具
│
├── cmd/                                  # 命令处理
│   ├── include/
│   │   ├── cmd_util.h
│   │   ├── params.h
│   │   ├── params_run_tool.h             # 命令运行器
│   │   └── params_trust_list.h
│   └── src/
│       ├── cmd_util.cpp
│       ├── params.cpp
│       ├── params_run_tool.cpp           # 命令处理实现
│       └── params_trust_list.cpp
│
├── codesigning/                          # 代码签名
│   ├── datastructure/                    # 数据结构
│   │   ├── include/
│   │   │   ├── code_sign_block.h
│   │   │   ├── fs_verity_info_segment.h
│   │   │   └── merkle_tree_extension.h
│   │   └── src/
│   │       ├── code_sign_block.cpp
│   │       ├── fs_verity_info_segment.cpp
│   │       └── merkle_tree_extension.cpp
│   ├── fsverity/                         # fsverity 实现
│   │   ├── include/
│   │   │   ├── fs_verity_generator.h
│   │   │   └── merkle_tree_builder.h
│   │   └── src/
│   │       ├── fs_verity_generator.cpp
│   │       └── merkle_tree_builder.cpp
│   ├── sign/                             # 签名实现
│   │   ├── include/
│   │   │   ├── bc_signeddata_generator.h
│   │   │   ├── code_signing.h
│   │   │   └── verify_code_signature.h
│   │   └── src/
│   │       ├── bc_signeddata_generator.cpp
│   │       ├── code_signing.cpp
│   │       └── verify_code_signature.cpp
│   └── utils/                            # 工具函数
│       ├── include/
│       │   ├── cms_utils.h
│   │   └── src/
│       │   └── cms_utils.cpp
│
├── common/                               # 通用组件
│   ├── include/
│   │   ├── byte_buffer.h
│   │   ├── constant.h
│   │   └── options.h                     # 参数选项
│   └── src/
│       ├── byte_buffer.cpp
│       └── constant.cpp
│
├── hap/                                  # HAP 签名/验签
│   ├── config/                           # 签名配置
│   │   ├── include/signer_config.h
│   │   └── src/signer_config.cpp
│   ├── entity/                           # 实体定义
│   │   ├── include/
│   │   │   ├── block_data.h
│   │   │   ├── content_digest_algorithm.h
│   │   │   ├── sign_block_info.h
│   │   │   ├── sign_head.h
│   │   │   └── signing_block.h
│   │   └── src/
│   ├── provider/                         # 签名提供者
│   │   ├── include/
│   │   │   ├── local_sign_provider.h
│   │   │   ├── remote_sign_provider.h
│   │   │   └── sign_provider.h
│   │   └── src/
│   ├── sign/                             # 签名实现
│   │   ├── include/
│   │   │   ├── bc_pkcs7_generator.h
│   │   │   ├── sign_bin.h
│   │   │   ├── sign_elf.h
│   │   │   └── sign_hap.h
│   │   └── src/
│   ├── utils/                            # HAP 工具
│   │   ├── include/dynamic_lib_handle.h
│   │   └── src/
│   └── verify/                           # 验签实现
│       ├── include/
│       │   ├── matching_result.h
│       │   ├── verify_bin.h
│       │   ├── verify_elf.h
│       │   └── verify_hap.h
│       └── src/
│
├── profile/                              # Profile 签名
│   └── src/
│       ├── pkcs7_data.cpp
│       ├── profile_info.cpp
│       ├── profile_sign_tool.cpp
│       └── profile_verify.cpp
│
├── signer/                               # 签名器
│   ├── include/
│   │   ├── local_signer.h
│   │   ├── signer.h
│   │   └── signer_factory.h
│   └── src/
│       ├── local_signer.cpp
│       └── signer_factory.cpp
│
├── utils/                                # 通用工具
│   ├── include/
│   │   ├── file_utils.h
│   │   ├── string_utils.h
│   │   └── key_store_helper.h
│   └── src/
│
└── zip/                                  # ZIP 处理
    ├── include/
    └── src/
```

### 关键文件

| 文件路径 | 职责 |
|---------|------|
| `main.cpp` | C++ 程序入口 |
| `api/include/service_api.h` | C++ 服务接口定义 |
| `cmd/include/params_run_tool.h` | 命令处理核心 |
| `codesigning/sign/include/code_signing.h` | 代码签名接口 |

---

## binary_sign_tool (C++ 轻量版)

**路径**: `binary_sign_tool/`

**职责**: 精简的 C++ 签名工具，仅支持二进制文件签名/验签。

**构建方式**: GN/Ninja

### 子模块结构

```
binary_sign_tool/
├── BUILD.gn                              # 构建配置
├── signature_tools.gni                   # 路径定义
├── main.cpp                              # 程序入口
│
├── api/                                  # API 层（简化版）
│   ├── include/service_api.h             # 简化接口：仅 Sign/Verify
│   └── src/sign_tool_service_impl.cpp
│
├── cmd/                                  # 命令处理
│   └── src/
│       ├── cmd_util.cpp
│       ├── params.cpp
│       └── params_run_tool.cpp
│
├── codesigning/                          # 代码签名
│   ├── fsverity/                         # fsverity 实现
│   │   └── src/
│   │       ├── fs_verity_generator.cpp
│   │       └── merkle_tree_builder.cpp
│   └── sign/
│       └── src/code_signing.cpp
│
├── common/                               # 通用组件
│   ├── include/
│   │   ├── constant.h
│   │   ├── options.h
│   │   └── password_guard.h
│   └── src/options.cpp
│
├── hap/                                  # HAP/ELF 处理
│   ├── entity/
│   ├── provider/                         # 签名提供者
│   │   ├── include/
│   │   │   ├── local_sign_provider.h
│   │   │   ├── self_sign_sign_provider.h
│   │   │   └── sign_provider.h
│   │   └── src/
│   ├── sign/                             # ELF 签名
│   │   └── src/sign_elf.cpp
│   ├── utils/
│   └── verify/                           # ELF 验签
│       └── src/verify_elf.cpp
│
├── profile/                              # Profile 处理
│   └── src/
│       ├── profile_info.cpp
│       └── profile_sign_tool.cpp
│
├── signer/                               # 签名器（引用 hapsigntool_cpp）
│
└── utils/                                # 工具
    └── src/file_utils.cpp
```

### 与 hapsigntool_cpp 的区别

| 特性 | hapsigntool_cpp | binary_sign_tool |
|------|----------------|------------------|
| API | 完整（10+ 方法） | 精简（Sign/Verify） |
| 证书生成 | 支持 | 不支持 |
| Profile 签名 | 完整 | 简化 |
| 代码签名 | 支持 | 支持 |
| OpenSSL | 动态链接 | 静态链接 |
| 依赖 | 较多 | 精简 |

---

## dist (SDK 预配置文件)

**路径**: `dist/`

**职责**: 存放 SDK 预置的证书和模板文件。

```
dist/
├── hap-sign-tool.jar                   # Java 签名工具 JAR
├── OpenHarmony.p12                     # 示例 Keystore
├── OpenHarmonyApplication.pem          # 应用证书
├── OpenHarmonyProfileDebug.pem         # Debug Profile 证书
├── OpenHarmonyProfileRelease.pem       # Release Profile 证书
├── SgnedReleaseProfileTemplate.p7b     # 签名后的 Profile 模板
├── UnsgnedDebugProfileTemplate.json    # Debug Profile 模板
└── UnsgnedReleasedProfileTemplate.json # Release Profile 模板
```

---

## autosign (一键签名脚本)

**路径**: `autosign/`

**职责**: 提供 Python 脚本实现一键式签名流程。

```
autosign/
├── create_root.sh / create_root.bat           # 创建根证书脚本
├── create_appcert_sign_profile.sh / .bat    # 创建应用证书和 Profile
├── sign_hap.sh / sign_hap.bat               # 签名 HAP
├── sign_elf.sh / sign_elf.bat               # 签名 ELF
├── createAppCertAndProfile.config           # 应用证书配置
├── createRootAndSubCert.config              # 根证书配置
├── signHap.config                           # HAP 签名配置
├── signElf.config                           # ELF 签名配置
└── *.json / *.p7b / *.pem                   # 示例证书和模板
```

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         入口层                                   │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │ HapSignTool  │  │ hapsigntool_cpp  │  │binary_sign_tool  │  │
│  │   (Java)     │  │   (C++ 完整版)    │  │  (C++ 轻量版)     │  │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │
└─────────┼───────────────────┼─────────────────────┼────────────┘
          │                   │                     │
          ▼                   ▼                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                         API 层                                   │
│  ┌────────────────────────┐  ┌────────────────────────────────┐ │
│  │   ServiceApi (Java)    │  │   ServiceApi (C++)             │ │
│  │  - SignToolServiceImpl │  │  - SignToolServiceImpl         │ │
│  └───────────┬────────────┘  └───────────────┬────────────────┘ │
└──────────────┼───────────────────────────────┼──────────────────┘
               │                               │
               ▼                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      业务逻辑层                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────┐ │
│  │   Signer    │  │   Profile   │  │ CodeSigning │  │  HAP   │ │
│  │  (签名器)    │  │  (Profile)  │  │  (代码签名)  │  │(HAP包) │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └───┬────┘ │
└─────────┼────────────────┼────────────────┼─────────────┼──────┘
          │                │                │             │
          ▼                ▼                ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      基础能力层                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │   Cert      │  │    ZIP      │  │        OpenSSL          │ │
│  │  (证书操作)  │  │  (ZIP操作)   │  │     (加密/签名)          │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 相关链接

- [项目概览](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 理解系统设计
- [构建系统](04_Build_System.md) - 查看构建配置

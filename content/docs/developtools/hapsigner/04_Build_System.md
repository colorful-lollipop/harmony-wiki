# 构建系统

## 目的

本文档介绍 hapsigner 的构建系统配置，包括 GN 构建目标和 Maven 构建配置。

## 适用范围

- 需要构建项目的开发者
- 需要修改构建配置的维护者

---

## 构建系统概述

hapsigner 使用两套构建系统：

| 组件 | 构建工具 | 说明 |
|------|----------|------|
| Java 版本 | Maven | 开发工具/SDK 分发 |
| C++ 版本 | GN/Ninja | 系统工具链集成 |

---

## Maven 构建

### 项目结构

**代码证据**: `hapsigntool/pom.xml`

```xml
<project>
    <groupId>com.ohos</groupId>
    <artifactId>hapsigntool</artifactId>
    <version>4.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>hap_sign_tool</module>      <!-- CLI 入口 -->
        <module>hap_sign_tool_lib</module>  <!-- 核心库 -->
    </modules>
</project>
```

### 构建命令

```bash
# 进入项目目录
cd hapsigntool

# 编译打包
mvn package

# 运行测试
mvn test

# 安装到本地仓库
mvn install

# 清理构建产物
mvn clean
```

### 构建产物

| 产物 | 路径 | 说明 |
|------|------|------|
| hap-sign-tool.jar | `hap_sign_tool/target/` | 可执行 JAR 包 |
| hap_sign_tool_lib.jar | `hap_sign_tool_lib/target/` | 核心库 JAR 包 |

### 依赖管理

**代码证据**: `hapsigntool/pom.xml:26-61`

```xml
<dependencyManagement>
    <dependencies>
        <!-- JSON 处理 -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.11.0</version>
        </dependency>
        
        <!-- 加密库 -->
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcpkix-jdk18on</artifactId>
            <version>1.79</version>
        </dependency>
        
        <!-- 日志 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.23.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## GN 构建

### 构建目标概览

| Target | 类型 | 位置 | 说明 |
|--------|------|------|------|
| `lib` | ohos_copy | `BUILD.gn` | 复制 SDK 文件 |
| `copy_signature_tools_resource` | ohos_copy | `hapsigntool_cpp/BUILD.gn` | 复制签名资源 |
| `hap-sign-tool` | ohos_executable | `hapsigntool_cpp/BUILD.gn` | C++ 完整版签名工具 |
| `binary-sign-tool` | ohos_executable | `binary_sign_tool/BUILD.gn` | C++ 轻量版签名工具 |

### 根 BUILD.gn

**代码位置**: `BUILD.gn`

```gn
import("//build/ohos.gni")
import("//build/ohos/ace/ace.gni")

ohos_copy("lib") {
  if (build_public_version) {
    sources = [
      "dist/OpenHarmony.p12",
      "dist/OpenHarmonyProfileDebug.pem",
      "dist/OpenHarmonyProfileRelease.pem",
      "dist/UnsgnedDebugProfileTemplate.json",
      "dist/UnsgnedReleasedProfileTemplate.json",
      "dist/hap-sign-tool.jar",
    ]
  } else {
    sources = "dist/hap-sign-tool.jar"
  }
  outputs = [ target_out_dir + "/$target_name/{{source_file_part}}" ]
}
```

**功能**: 将 dist 目录下的预置文件复制到输出目录。

### hapsigntool_cpp BUILD.gn

**代码位置**: `hapsigntool_cpp/BUILD.gn`

```gn
# 导入模块定义
import("cmd/signature_tools_cmd.gni")
import("codesigning/signature_tools_codesigning.gni")
import("common/signature_tools_common.gni")
import("hap/signature_tools_hap.gni")
import("profile/signature_tools_profile.gni")
import("signature_tools.gni")
import("utils/signature_tools_utils.gni")
import("zip/signature_tools_zip.gni")

import("//build/ohos.gni")

# 复制资源文件
ohos_copy("copy_signature_tools_resource") {
  sources = [
    "../dist/OpenHarmony.p12",
    "../dist/OpenHarmonyApplication.pem",
    "../dist/OpenHarmonyProfileDebug.pem",
    "../dist/OpenHarmonyProfileRelease.pem",
    "../dist/SgnedReleaseProfileTemplate.p7b",
    "../dist/UnsgnedDebugProfileTemplate.json",
    "../dist/UnsgnedReleasedProfileTemplate.json",
  ]
  outputs = ["${target_out_dir}/toolchains/hapsigntool_pc/{{source_file_part}}"]
  part_name = "hapsigner"
  subsystem_name = "developtools"
}

# 可执行文件目标
ohos_executable("hap-sign-tool") {
  include_dirs = [
    "${signature_tools_api}/include",
    "${signature_tools_signer}/include",
    "//third_party/openssl/include",
    "//third_party/openssl/crypto/pkcs12",
    # ... 其他 include 路径
  ]
  
  sources = [
    "main.cpp",
    "${signature_tools_api}/src/sign_tool_service_impl.cpp",
    "${signature_tools_api}/src/cert_tools.cpp",
    "${signature_tools_signer}/src/signer_factory.cpp",
    "${signature_tools_signer}/src/local_signer.cpp",
    # ... 其他源文件（通过 .gni 引入）
  ]
  
  deps = [
    "//third_party/bzip2:libbz2",
    "//third_party/openssl:libcrypto_shared",
    "//third_party/openssl:libssl_shared",
  ]
  
  external_deps = [
    "c_utils:utils",
    "cJSON:cjson_static",
    "zlib:shared_libz",
  ]
  
  cflags_cc = [
    "-std=c++17",
    "-fno-rtti",
  ]
  
  cflags = [
    "-fno-rtti",
    "-Wno-c++20-extensions",
  ]
  
  install_images = ["system"]
  install_enable = false
  part_name = "hapsigner"
  subsystem_name = "developtools"
}
```

### binary_sign_tool BUILD.gn

**代码位置**: `binary_sign_tool/BUILD.gn`

```gn
ohos_executable("binary-sign-tool") {
  include_dirs = [
    "${signature_tools_api}/include",
    "//third_party/openssl/include",
    "//third_party/openssl/crypto/pkcs12",
    # ... 其他 include 路径
  ]
  
  sources = [
    "main.cpp",
    "${signature_tools_api}/src/sign_tool_service_impl.cpp",
    # ... 其他源文件
  ]
  
  external_deps = [
    "bounds_checking_function:libsec_static",
    "elfio:elfio",
    "cJSON:cjson_static",
    "openssl:libcrypto_static",
    "openssl:libssl_static",
  ]
  
  cflags_cc = [
    "-std=c++17",
    "-fno-rtti",
  ]
  
  remove_configs = ["//build/config:executable_config"]
  install_images = ["system"]
  install_enable = false
  part_name = "hapsigner"
  subsystem_name = "developtools"
}
```

### 模块 GNI 文件

项目使用 .gni 文件组织源文件，实现模块化构建：

| GNI 文件 | 说明 | 源文件数量 |
|----------|------|-----------|
| `cmd/signature_tools_cmd.gni` | 命令处理模块 | 4 |
| `codesigning/signature_tools_codesigning.gni` | 代码签名模块 | 23 |
| `common/signature_tools_common.gni` | 通用组件 | 9 |
| `hap/signature_tools_hap.gni` | HAP 签名模块 | 24 |
| `profile/signature_tools_profile.gni` | Profile 签名模块 | 4 |
| `utils/signature_tools_utils.gni` | 工具函数 | 9 |
| `zip/signature_tools_zip.gni` | ZIP 处理 | 10 |

**示例**: `hapsigntool_cpp/hap/signature_tools_hap.gni`

```gn
signature_tools_hap_include = [
  "${signature_tools_hap}/verify/include",
  "${signature_tools_hap}/config/include",
  "${signature_tools_hap}/entity/include",
  "${signature_tools_hap}/provider/include",
  "${signature_tools_hap}/sign/include",
  "${signature_tools_hap}/utils/include",
]

signature_tools_hap_src = [
  "${signature_tools_hap}/verify/src/verify_hap.cpp",
  "${signature_tools_hap}/verify/src/verify_bin.cpp",
  "${signature_tools_hap}/verify/src/verify_elf.cpp",
  "${signature_tools_hap}/config/src/signer_config.cpp",
  "${signature_tools_hap}/entity/src/content_digest_algorithm.cpp",
  # ... 其他源文件
]
```

---

## 编译产物

### Java 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| hap-sign-tool.jar | `hapsigntool/hap_sign_tool/target/` | 可执行 JAR |
| hap_sign_tool_lib.jar | `hapsigntool/hap_sign_tool_lib/target/` | 核心库 |

### C++ 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| hap-sign-tool | `${target_out_dir}/` | C++ 完整版可执行文件 |
| binary-sign-tool | `${target_out_dir}/` | C++ 轻量版可执行文件 |
| OpenHarmony.p12 | `${target_out_dir}/toolchains/hapsigntool_pc/` | 示例 Keystore |
| *.pem | `${target_out_dir}/toolchains/hapsigntool_pc/` | 证书文件 |
| *.json | `${target_out_dir}/toolchains/hapsigntool_pc/` | Profile 模板 |

### bundle.json 配置

**代码位置**: `hapsigntool_cpp/bundle.json`

```json
{
  "component": {
    "name": "hapsigner",
    "subsystem": "developtools",
    "adapted_system_type": ["standard"],
    "deps": {
      "components": [
        "bounds_checking_function",
        "c_utils",
        "cJSON",
        "elfio",
        "openssl",
        "zlib",
        "hilog"
      ],
      "third_party": [
        "bzip2",
        "openssl"
      ]
    },
    "build": {
      "sub_component": [
        "//developtools/hapsigner/binary_sign_tool:binary-sign-tool",
        "//developtools/hapsigner/hapsigntool_cpp:hap-sign-tool"
      ]
    }
  }
}
```

---

## 编译选项

### C++ 编译标志

| 标志 | 值 | 说明 |
|------|-----|------|
| `-std=c++17` | C++17 | C++ 标准版本 |
| `-fno-rtti` | - | 禁用运行时类型信息 |
| `-Wno-c++20-extensions` | - | 禁用 C++20 扩展警告 |

### 依赖库

#### hap-sign-tool 依赖

```gn
deps = [
    "//third_party/bzip2:libbz2",           # bzip2 压缩
    "//third_party/openssl:libcrypto_shared",  # OpenSSL 加密（动态）
    "//third_party/openssl:libssl_shared",     # OpenSSL SSL（动态）
]

external_deps = [
    "c_utils:utils",                        # C 工具库
    "cJSON:cjson_static",                   # JSON 解析
    "zlib:shared_libz",                     # zlib 压缩
]
```

#### binary-sign-tool 依赖

```gn
external_deps = [
    "bounds_checking_function:libsec_static",  # 安全函数库
    "elfio:elfio",                           # ELF 文件处理
    "cJSON:cjson_static",                    # JSON 解析
    "openssl:libcrypto_static",              # OpenSSL 加密（静态）
    "openssl:libssl_static",                 # OpenSSL SSL（静态）
]
```

---

## 构建命令

### 完整构建

```bash
# 进入 OpenHarmony 源码根目录
source build/envsetup.sh

# 编译 hapsigner
build.sh --product {product_name} --build-target hapsigner

# 或编译整个 developtools 子系统
build.sh --product {product_name} --build-target developtools
```

### 单独构建

```bash
# 使用 GN 直接构建
gn gen out/default
ninja -C out/default hap-sign-tool
ninja -C out/default binary-sign-tool
```

### 安装产物

```bash
# 产物安装路径
${target_out_dir}/toolchains/hapsigntool_pc/
```

---

## 相关链接

- [目录结构](01_Directory_Structure.md) - 了解代码组织
- [架构说明](02_Architecture.md) - 理解系统设计
- [API 参考](03_API_Reference.md) - 查看接口详情

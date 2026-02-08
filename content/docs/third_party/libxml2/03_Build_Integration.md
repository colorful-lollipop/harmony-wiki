# OpenHarmony 构建适配

## 概述

OpenHarmony 使用 GN (Generate Ninja) 构建系统替代 libxml2 原始的 autotools (configure/make) 构建系统。此适配通过 `BUILD.gn`、Python 脚本和配置文件实现，保持了上游源码的 pristine 状态。

---

## BUILD.gn 结构

### 构建目标

OpenHarmony 的 libxml2 提供三个构建目标：

#### 1. 共享库目标
```gn
ohos_shared_library("libxml2") {
    branch_protector_ret = "pac_ret"
    # ... 其他配置
    innerapi_tags = [
        "chipsetsdk",
        "platformsdk",
        "sasdk",
    ]
    install_images = [
        "updater",
        "system",
    ]
    part_name = "libxml2"
    subsystem_name = "thirdparty"
}
```

**关键属性**:
- `branch_protector_ret = "pac_ret"`: ARM Pointer Authentication 保护
- `innerapi_tags`: API 级别标记，用于 SDK 分类
- `install_images`: 安装到 system 和 updater 镜像

#### 2. 静态库目标
```gn
ohos_static_library("static_libxml2") {
    # 与共享库类似的源文件配置
    part_name = "libxml2"
    subsystem_name = "thirdparty"
}
```

**用途**: 用于需要静态链接的场景（如跨平台 ArkUI-X 测试）。

#### 3. iOS Framework 目标（条件编译）
```gn
if (current_os == "ios") {
    ohos_combine_darwin_framework("libxml2_shared") {
        deps = [ ":libxml2" ]
        subsystem_name = "thirdparty"
        part_name = "libxml2"
    }
}
```

**用途**: 为 iOS 平台提供 framework 打包。

---

## 配置生成系统

### 头文件生成流程

OpenHarmony 不使用 autotools 的 `configure` 脚本，而是使用 **Python 脚本生成配置头文件**：

```
libxml2-2.14.0.tar.xz
    ↓ (install.py 解压)
libxml2-2.14.0/
    config.h.cmake.in   ← generate_header.py
    include/libxml/xmlversion.h.in  ← generate_header.py
        ↓
${target_gen_dir}/include/config.h
${target_gen_dir}/include/libxml/xmlversion.h
```

### generate_header.py 脚本

**位置**: `/Volumes/lexar/code/d/work/oh/third_party/libxml2/generate_header.py`

**功能**: 将 JSON 配置文件和 `.in` 模板转换为 C 头文件

**类结构**:

```python
class ConfigHeader:
    """处理 config.h.cmake.in 生成 config.h"""
    def parse_file(self, file_path):
        # 读取 .in 文件
        # 替换 #cmakedefine 为 #define 或 #undef

class XmlVersionHeader:
    """处理 xmlversion.h.in 生成 xmlversion.h"""
    def parse_file(self, file_path):
        # 读取 .in 文件
        # 替换 @WITH_THREADS@, @VERSION@ 等变量
```

**BUILD.gn 调用**:
```gn
action("libxml2_generate_header") {
    script = "generate_header.py"
    inputs = [
        config_in_path,
        xmlversion_in_path,
        config_json_path,
        xml_version_json_path,
    ]
    outputs = [
        config_path,
        xmlversion_path,
    ]
    args = [
        "--config-input-path", config_in_path,
        "--config-path", config_path,
        "--xmlversion-input-path", xmlversion_in_path,
        "--xmlversion-path", xmlversion_path,
        "--config-json", config_json_path,
        "--xmlversion-json", xml_version_json_path,
    ]
}
```

---

## 平台配置文件

### config_linux.json (Linux 平台)

**位置**: `/Volumes/lexar/code/d/work/oh/third_party/libxml2/config_linux.json`

**用途**: 定义 Linux 平台的 `HAVE_*` 宏

**关键宏** (部分):

| 宏 | 值 | 说明 |
|-----|------|------|
| `HAVE_CONFIG_H` | "1" | 使用生成的 config.h |
| `HAVE_PTHREAD_H` | "1" | 支持 pthread |
| `HAVE_LIBPTHREAD` | "1" | 链接 pthread |
| `HAVE_MMAP` | "1" | 支持 mmap |
| `HAVE_DLOPEN` | "1" | 支持 dlopen 动态加载 |
| `HAVE_GETADDRINFO` | "1" | 支持 getaddrinfo |
| `HAVE_NETINET_IN_H` | "1" | 支持 socket 头文件 |
| `ICONV_CONST` | "const" | iconv const 限定符 |
| `_REENTRANT` | (未设置) | 线程安全标志 |
| `_GNU_SOURCE` | "1" (is_linux) | GNU 扩展 |

**平台特定配置** (BUILD.gn):
```gn
if (is_linux) {
    defines += [ "_GNU_SOURCE" ]
    libs = [ "dl" ]
}
```

### config_win.json (Windows/Mingw 平台)

**位置**: `/Volumes/lexar/code/d/work/oh/third_party/libxml2/config_win.json`

**用途**: 定义 Windows/Mingw 平台的 `HAVE_*` 宏

**关键差异** (与 Linux 对比):

| 宏 | Windows 值 | Linux 值 | 说明 |
|-----|-----------|-----------|------|
| `XML_SOCKLEN_T` | "socklen_t" | "socklen_t" | Socket 长度类型 |
| `HAVE_DIRENT_H` | 未设置 | "1" | Windows 不使用 dirent |
| `HAVE_UNISTD_H` | 未设置 | "1" | Windows 不使用 unistd |
| `HAVE_SYS_TIMEB_H` | "1" | 未设置 | Windows 使用 timeb.h |

**平台特定配置** (BUILD.gn):
```gn
if (is_mingw) {
    libs = [
        "bcrypt",
        "ws2_32",
    ]
}

ohos_static_library("static_libxml2") {
    # ...
    if (is_mingw) {
        defines = [ "LIBXML_STATIC" ]
    }
}
```

### xml_version.json (功能配置)

**位置**: `/Volumes/lexar/code/d/work/oh/third_party/libxml2/xml_version.json`

**用途**: 定义 libxml2 编译时启用的功能

**功能标志** (部分):

| 功能 | 值 | 说明 |
|------|------|------|
| `VERSION` | "2.9.13" | 版本号（注意与 bundle.json 中的 2.14.0 不同）|
| `LIBXML_VERSION_NUMBER` | "209013" | 版本号数值 |
| `WITH_THREADS` | "1" | 线程支持 |
| `WITH_TREE` | "1" | DOM 树支持 |
| `WITH_OUTPUT` | "1" | 输出序列化 |
| `WITH_READER` | "1" | XmlReader API |
| `WITH_PATTERN` | "1" | 模式匹配 |
| `WITH_WRITER` | "1" | XmlWriter API |
| `WITH_SAX1` | "1" | SAX1 接口 |
| `WITH_FTP` | "1" | FTP 支持（2.14.0 已移除，但标志仍保留）|
| `WITH_HTTP` | "1" | HTTP 支持 |
| `WITH_VALID` | "1" | DTD 验证 |
| `WITH_HTML` | "1" | HTML 解析 |
| `WITH_LEGACY` | "1" | 遗留 API |
| `WITH_C14N` | "1" | 规范化 |
| `WITH_CATALOG` | "1" | Catalog 支持 |
| `WITH_DOCB` | "1" | DocBook 支持 |
| `WITH_XPATH` | "1" | XPath 支持 |
| `WITH_XPTR` | "1" | XPointer 支持 |
| `WITH_XINCLUDE` | "1" | XInclude 支持 |
| `WITH_ISO8859X` | "1" | ISO-8859 编码 |
| `WITH_DEBUG` | "1" | 调试支持 |
| `WITH_REGEXPS` | "1" | 正则表达式 |
| `WITH_SCHEMAS` | "1" | XSD Schema 支持 |
| `WITH_SCHEMATRON` | "1" | Schematron 支持 |
| `WITH_MODULES` | "1" | 动态模块 |
| `MODULE_EXTENSION` | ".so" | 模块扩展名 |

**注意**: `VERSION` 在 xml_version.json 中为 "2.9.13"，但 bundle.json 显示版本为 2.14.0。这可能是配置文件的版本号未更新。

---

## 源码处理流程

### install.py 脚本

**位置**: `/Volumes/lexar/code/d/work/oh/third_party/libxml2/install.py`

**功能**:
1. 解压 `libxml2-2.14.0.tar.xz` 到 `${target_gen_dir}`
2. 应用所有 Patch 文件到解压后的源码

**BUILD.gn 调用**:
```gn
action("libxml2_install_action") {
    script = "//third_party/libxml2/install.py"
    outputs = [
        "${target_gen_dir}/libxml2-2.14.0/HTMLparser.c",
        "${target_gen_dir}/libxml2-2.14.0/parser.c",
        # ... 所有源文件
    ]
    inputs = [
        "//third_party/libxml2/libxml2-2.14.0.tar.xz"
    ]
    inputs += [
        "Backport-CVE-2025-32414-...",
        "Backport-CVE-2025-32415-...",
        # ... 共 12 个 Patch 文件
    ]
    args = [
        "--gen-dir", rebase_path("${target_gen_dir}", root_build_dir),
        "--source-file", rebase_path("//third_party/libxml2"),
    ]
}
```

**Patch 清单** (按应用顺序):
```python
patch_file = [
    "Backport-CVE-2025-32414-python-Read-at-most-len-4-ch-c.patch",
    "Backport-CVE-2025-32415-schemas-Fix-heap-buffer-over-c.patch",
    "Fix_XML_PARSE_NOBLANKS_dropping_non-whitespace_text.patch",
    "Backport-CVE-2025-6021-tree-Fix-integer-overflow-in-xmlBuildQName-c.patch",
    "Fix-relaxng-is-parsed-to-an-infinite-attrs-next-loop.patch",
    "Backport-CVE-2025-6170-Fix-potential-buffer-overflow-of-interactive-shell.patch",
    "Fix-CVE-2025-49794-CVE-2025-49796-memory-safety-issues-in-xmlSchematronReportOutput.patch",
    "Fix-CVE-2025-49795-null-pointer-dereference-leading-to-DoS.patch",
    "Fix-CVE-2025-8732-Prevent-infinite-recursion-in-xmlCatalogList.patch",
    "Fix-CVE-2026-0990-catalog-prevent-inf-recursion-in-xmlCatalogXMLResolveURI.patch",
    "Fix-CVE-2026-0992-catalog-Ignore-repeated-nextCatalog-entries.patch",
    "Fix-CVE-2026-0989-Add-RelaxNG-include-limit.patch",
]
```

---

## 编译配置

### 公共配置

#### libxml2_config (使用方配置)
```gn
config("libxml2_config") {
    include_dirs = [
        get_label_info(":libxml2_generate_header", "target_out_dir") + "/include",
        get_label_info(":libxml2_install_action", "target_gen_dir") +
            "/libxml2-2.14.0/include",
    ]
}
```

**用途**: 为依赖 libxml2 的模块提供头文件路径。

#### libxml2_static_config (静态库使用方配置)
```gn
config("libxml2_static_config") {
    include_dirs = [
        get_label_info(":libxml2_generate_header", "target_out_dir") + "/include",
        get_label_info(":libxml2_install_action", "target_gen_dir") +
            "/libxml2-2.14.0/include",
    ]
    if (is_mingw) {
        defines = [ "LIBXML_STATIC" ]
    } else {
        defines = [
            "HAVE_CONFIG_H",
            "_REENTRANT",
        ]
    }
}
```

**用途**: 为依赖静态库的模块提供头文件路径和必要的宏。

### 私有配置

#### libxml2_private_config (共享库内部配置)
```gn
config("libxml2_private_config") {
    visibility = [ ":*" ]
    cflags = [
        "-Wno-empty-body",
        "-Wno-incompatible-pointer-types",
        "-Wno-missing-field-initializers",
        "-Wno-self-assign",
        "-Wno-sign-compare",
        "-Wno-tautological-pointer-compare",
        "-Wno-unused-function",
        "-Wno-enum-compare",
        "-Wno-int-conversion",
        "-Wno-uninitialized",
        "-Wno-implicit-fallthrough",
    ]
    defines = [
        "HAVE_CONFIG_H",
        "_REENTRANT",
    ]
    if (is_linux) {
        defines += [ "_GNU_SOURCE" ]
    }
}
```

**用途**: 禁用 libxml2 源码中的编译警告，确保构建通过 OH 的 `-Werror` 标志。

#### libxml2_static_private_config (静态库内部配置)
```gn
config("libxml2_static_private_config") {
    cflags = [
        "-Wno-implicit-fallthrough",
        "-Wno-implicit-function-declaration",
        "-Wno-int-conversion",
        "-Wno-uninitialized",
        "-Wno-sometimes-uninitialized",
    ]
    cflags_cc = [ "-std=c++17" ]
}
```

**用途**: 静态库的额外编译标志（注意 C++ 标准设置，可能用于测试）。

---

## 源文件列表

### 包含的源文件 (BUILD.gn outputs)

```gn
outputs = [
    "${target_gen_dir}/libxml2-2.14.0/HTMLparser.c",
    "${target_gen_dir}/libxml2-2.14.0/HTMLtree.c",
    "${target_gen_dir}/libxml2-2.14.0/SAX2.c",
    "${target_gen_dir}/libxml2-2.14.0/buf.c",
    "${target_gen_dir}/libxml2-2.14.0/c14n.c",
    "${target_gen_dir}/libxml2-2.14.0/catalog.c",
    "${target_gen_dir}/libxml2-2.14.0/chvalid.c",
    "${target_gen_dir}/libxml2-2.14.0/debugXML.c",
    "${target_gen_dir}/libxml2-2.14.0/dict.c",
    "${target_gen_dir}/libxml2-2.14.0/encoding.c",
    "${target_gen_dir}/libxml2-2.14.0/entities.c",
    "${target_gen_dir}/libxml2-2.14.0/error.c",
    "${target_gen_dir}/libxml2-2.14.0/globals.c",
    "${target_gen_dir}/libxml2-2.14.0/hash.c",
    "${target_gen_dir}/libxml2-2.14.0/list.c",
    "${target_gen_dir}/libxml2-2.14.0/nanohttp.c",
    "${target_gen_dir}/libxml2-2.14.0/parser.c",
    "${target_gen_dir}/libxml2-2.14.0/parserInternals.c",
    "${target_gen_dir}/libxml2-2.14.0/pattern.c",
    "${target_gen_dir}/libxml2-2.14.0/relaxng.c",
    "${target_gen_dir}/libxml2-2.14.0/schematron.c",
    "${target_gen_dir}/libxml2-2.14.0/threads.c",
    "${target_gen_dir}/libxml2-2.14.0/tree.c",
    "${target_gen_dir}/libxml2-2.14.0/uri.c",
    "${target_gen_dir}/libxml2-2.14.0/valid.c",
    "${target_gen_dir}/libxml2-2.14.0/xinclude.c",
    "${target_gen_dir}/libxml2-2.14.0/xlink.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlIO.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlmemory.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlmodule.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlreader.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlregexp.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlsave.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlschemas.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlschemastypes.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlstring.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlunicode.c",
    "${target_gen_dir}/libxml2-2.14.0/xmlwriter.c",
    "${target_gen_dir}/libxml2-2.14.0/xpath.c",
    "${target_gen_dir}/libxml2-2.14.0/xpointer.c",
    "${target_gen_dir}/libxml2-2.14.0/xzlib.c",
    "${target_gen_dir}/libxml2-2.14.0/shell.c",
    "${target_gen_dir}/libxml2-2.14.0/xmllint.c",
]
```

**源文件过滤**:
```gn
sources = filter_exclude(output_values,
                 [
                   "*.h.cmake.in",
                   "*.h.in",
                 ])
```

排除头文件模板（`.h.in`, `.h.cmake.in`），仅编译 C 源文件。

---

## 与上游构建系统的差异

| 对比项 | OH 实现 | 上游实现 |
|--------|---------|----------|
| **构建系统** | GN (BUILD.gn) | autotools (configure/make) |
| **配置生成** | Python 脚本 + JSON | autoconf/configure |
| **头文件路径** | `${root_gen_dir}/third_party/libxml2/libxml2-2.14.0/include` | /usr/local/include/libxml2 |
| **库安装路径** | 通过 install_images 配置 | --prefix 配置 |
| **平台检测** | GN 变量 (is_linux, is_mingw, current_os) | configure 检测脚本 |
| **功能开关** | xml_version.json (编译时) | configure --enable/disable-xxx |
| **源码修改** | Patch 机制 | 直接修改上游源码 |

### 关键差异说明

#### 1. Python 脚本替代 autoconf
**OH**: 使用 `generate_header.py` 读取 JSON 配置，生成 `config.h` 和 `xmlversion.h`
**上游**: 使用 `configure` 脚本检测系统功能，生成 `config.h`

**优势**:
- 可控性强：配置完全由 OH 控制
- 跨平台一致：Linux 和 Windows 使用相同脚本
- 避免依赖：不需要 autoconf 工具

#### 2. Patch 机制
**OH**: 源码包 pristine，通过 `install.py` 应用 Patch
**上游**: 直接修改源码仓库

**优势**:
- 清晰追踪：所有 OH 修改都在 Patch 文件中
- 易升级：只需更新 Patch 列表
- 合并友好：可评估哪些 Patch 可推向上游

#### 3. GN 构建目标
**OH**: 使用 `ohos_shared_library` 和 `ohos_static_library`
**上游**: 使用 automake `lib_LTLIBRARIES` 和 `check_PROGRAMS`

**优势**:
- 集成 OH 构建系统
- 支持条件编译（is_linux, is_mingw 等）
- 统一依赖管理

---

## 构建输出

### 库文件

| 目标 | 输出文件 | 安装位置 |
|------|----------|---------|
| `libxml2:libxml2` | `libxml2.so` | `/system/lib/` |
| `libxml2:static_libxml2` | `libxml2.a` | 不安装，供静态链接 |

### 头文件

**生成位置**: `${target_gen_dir}/include/`

**公开头文件**:
- `config.h` (自动生成)
- `libxml/xmlversion.h` (自动生成)
- `libxml/*.h` (从 libxml2-2.14.0/include/libxml/ 复制)

**使用方包含** (bundle.json):
```json
{
    "header": {
        "header_files": [],
        "header_base": "${root_gen_dir}/third_party/libxml2/libxml2-2.14.0/include"
    }
}
```

---

## 特殊构建选项

### ARM Pointer Authentication (PAC)
```gn
ohos_shared_library("libxml2") {
    branch_protector_ret = "pac_ret"
}
```

**说明**: 启用返回地址的指针认证，增强安全性（仅在 ARM 架构有效）。

### API 级别标记
```gn
innerapi_tags = [
    "chipsetsdk",    # 芯片 SDK
    "platformsdk",   # 平台 SDK
    "sasdk",        # 系统 SDK
]
```

**说明**: 用于 OH SDK 打包和发布流程，标记 API 可见性。

### 安装镜像
```gn
install_images = [
    "updater",  # 更新器镜像
    "system",    # 系统镜像
]
```

**说明**: libxml2 安装到 system 镜像和 updater 镜像，支持系统启动和更新。

---

## 构建依赖

### 外部依赖

| 平台 | 依赖库 | 用途 |
|------|---------|------|
| Linux | `dl` | 动态链接 |
| Windows/Mingw | `bcrypt`, `ws2_32` | 加密和 Socket |
| 所有平台 | 无 (纯 C) | — |

### 内部依赖

| 目标 | 依赖目标 |
|------|---------|
| `libxml2` | `:libxml2_generate_header`, `:libxml2_install_action` |
| `static_libxml2` | `:libxml2_generate_header`, `:libxml2_install_action` |

---

## 调试与优化

### 禁用的警告 (libxml2_private_config)
```gn
cflags = [
    "-Wno-empty-body",
    "-Wno-incompatible-pointer-types",
    "-Wno-missing-field-initializers",
    "-Wno-self-assign",
    "-Wno-sign-compare",
    "-Wno-tautological-pointer-compare",
    "-Wno-unused-function",
    "-Wno-enum-compare",
    "-Wno-int-conversion",
    "-Wno-uninitialized",
    "-Wno-implicit-fallthrough",
]
```

**说明**: 这些警告来源于 libxml2 源码，但 OH 构建系统启用 `-Werror`（警告视为错误），因此需要禁用。

### 线程安全宏
```gn
defines = [
    "HAVE_CONFIG_H",
    "_REENTRANT",  # 启用线程安全函数
]
```

**说明**: `_REENTRANT` 宏使 libxml2 使用线程安全的系统函数（如 `strtok_r`）。

---

## 总结

### 关键特性
1. **Python 驱动的配置生成**: 替代 autotools，提高可控性
2. **Patch 机制**: 保持上游源码 pristine，所有修改可见
3. **GN 深度集成**: 充分利用 OH 构建系统特性
4. **平台支持**: 明确支持 Linux、Windows/Mingw、iOS
5. **安全加固**: 启用 PAC、安装到多个镜像

### 最佳实践
- **使用共享库**: 默认使用 `//third_party/libxml2:libxml2`
- **静态库用于测试**: 使用 `//third_party/libxml2:static_libxml2` 进行单元测试
- **头文件路径**: 通过 `libxml2_config` 自动包含，无需手动配置

---

*最后更新: 2026-02-07*

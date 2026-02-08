# libxml2 API 差异分析

## 概述

**OpenHarmony 的 libxml2 没有添加任何 OH 特有的 API**。

所有适配都是**外部包装层**（BUILD.gn、Python 脚本、Patch 机制），保持上游源码的完整性。因此，OH 版本的 libxml2 API 与上游版本完全兼容。

---

## API 兼容性

### 完全兼容的 API
所有 libxml2 的公开 API 都是上游原始实现，包括：

#### 核心 API
- **DOM API**: `xmlDocPtr`, `xmlNodePtr`, `xmlNewDoc()`, `xmlFreeDoc()` 等
- **SAX API**: `xmlSAXHandler`, `xmlParseDocument()` 等
- **XMLReader API**: `xmlTextReaderPtr`, `xmlReaderForFile()` 等
- **XPath**: `xmlXPathEval()`, `xmlXPathNewContext()` 等
- **序列化**: `xmlDocDumpMemory()`, `xmlSaveFile()` 等

#### 扩展 API
- **XSD Schema**: `xmlSchemaParse()`, `xmlSchemaValidateDoc()` 等
- **RELAX NG**: `xmlRelaxNGParse()`, `xmlRelaxNGNewValidCtxt()` 等
- **Schematron**: `xmlSchematronParse()`, `xmlSchematronValidateDoc()` 等
- **Catalog**: `xmlLoadCatalog()`, `xmlCatalogResolve()` 等
- **XInclude**: `xmlXIncludeProcess()`, `xmlXIncludeProcessTree()` 等

### 无 OH 特有 API
- ❌ **新增 API**: 没有添加任何 OH 特有的函数或宏
- ❌ **修改 API**: 没有修改任何现有 API 的行为
- ❌ **废弃 API**: 没有废弃任何上游 API
- ❌ **平台宏**: 没有使用 `#ifdef OHOS`、`#ifndef OHOS` 等平台条件编译

---

## 构建系统差异

### BUILD.gn vs Autotools

| 对比项 | OH 实现 | 上游实现 |
|--------|---------|----------|
| **配置生成** | `generate_header.py` + `config_*.json` | `configure` 脚本 |
| **头文件路径** | `${root_gen_dir}/third_party/libxml2-2.14.0/include` | `/usr/local/include/libxml2` |
| **库安装** | 通过 `install_images` 配置 | `make install` |
| **功能开关** | `xml_version.json` 编译时 | `./configure --enable-xxx` |
| **平台检测** | GN 变量 (`is_linux`, `is_mingw`) | `configure` 检测 |

### 源码处理
**OH**:
- 保持上游源码包 `libxml2-2.14.0.tar.xz` pristine
- 通过 `install.py` 脚本应用 12 个 Patch
- Patch 机制确保所有修改可见和可追踪

**上游**:
- 直接修改源码仓库
- 通过 Git commit 追踪变更

---

## 配置宏差异

### OH vs 上游配置

#### 完全相同的宏
大多数配置宏与上游保持一致：
- `WITH_THREADS`, `WITH_TREE`, `WITH_OUTPUT` 等功能标志
- `HAVE_PTHREAD_H`, `HAVE_MMAP`, `HAVE_GETADDRINFO` 等平台检测宏
- `_REENTRANT`, `HAVE_CONFIG_H` 等编译选项

#### OH 特定的宏路径
```gn
// OH 构建
include_dirs = [
    get_label_info(":libxml2_generate_header", "target_out_dir") + "/include",
    get_label_info(":libxml2_install_action", "target_gen_dir") +
        "/libxml2-2.14.0/include",
]

// 上游构建
// 默认头文件路径: /usr/local/include/libxml2
// 或: --includedir=/custom/path
```

**差异**: OH 使用构建生成的头文件路径，上游使用安装路径。

---

## 行为差异

### 无行为修改
所有 API 行为与上游完全一致，包括：
- 错误处理机制
- 内存分配策略
- 字符编码处理
- 解析器选项行为
- Schema 验证逻辑

### 安全修复不影响 API
OH 应用的安全 Patch 修复了内部实现问题，但**不改变 API 接口**：
- CVE-2025-32414: 修复 Python 绑定内部逻辑
- CVE-2025-32415: 修复 Schema 验证内部逻辑
- CVE-2025-6021: 修复 QName 构建内部逻辑
- CVE-2025-49794/95/96: 修复 Schematron 内部逻辑
- CVE-2025-8732, CVE-2026-0989/92: 修复 Catalog 内部逻辑
- CVE-2026-0989: 修复 RELAX NG 内部逻辑

**结果**: 开发者无需修改代码，只需升级库即可获得安全修复。

---

## 使用方式差异

### 链接方式

#### 共享库链接（推荐）
```gn
// BUILD.gn
ohos_shared_library("my_module") {
    deps = [ "//third_party/libxml2:libxml2" ]
}

// 代码中
#include <libxml/parser.h>  // 自动通过 libxml2_config 包含
```

**优势**:
- 运行时动态链接
- 多模块共享，节省空间

#### 静态库链接（特殊情况）
```gn
// BUILD.gn
ohos_shared_library("my_test") {
    deps = [ "//third_party/libxml2:static_libxml2" ]
}

// 代码中
#include <libxml/parser.h>  // 自动通过 libxml2_static_config 包含
```

**优势**:
- 单元测试稳定性
- 跨平台兼容性

**注意**: 静态库使用 `LIBXML_STATIC` 宏（Windows/Mingw）。

### 头文件包含

OH 自动通过 `libxml2_config` 或 `libxml2_static_config` 配置包含头文件：

```gn
public_configs = [ "//third_party/libxml2:libxml2_config" ]
```

**开发者无需手动配置**，直接使用：
```c
#include <libxml/parser.h>
#include <libxml/tree.h>
#include <libxml/xpath.h>
```

---

## 总结

### 关键发现

1. **API 完全兼容**: OH 版本的 libxml2 API 与上游完全相同
2. **无 OH 特有 API**: 没有添加任何 OH 特有的函数或宏
3. **外部包装适配**: 所有 OH 适配都是外部包装层（BUILD.gn、脚本、Patch）
4. **源码保持 pristine**: 不修改上游源码，通过 Patch 应用变更
5. **升级友好**: 升级到新上游版本只需替换源码包和更新 Patch

### 开发者影响
- **无代码修改**: 现有代码无需修改
- **透明升级**: 升级 libxml2 对应用透明（API 兼容）
- **统一接口**: OH 与上游使用相同的 API 规范

### 建议与注意事项
1. **使用标准 API**: 遵循官方文档中的 API 使用方式
2. **检查 Patch**: 升级时检查哪些 OH Patch 可移除（已合入上游）
3. **关注上游变更**: 2.15+ 将移除部分功能，提前评估影响

---

*最后更新: 2026-02-07*

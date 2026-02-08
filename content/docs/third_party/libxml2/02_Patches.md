# libxml2 Patch 详细分析

## 概述

OpenHarmony 版本的 libxml2 共应用 **12 个 Patch**，主要用于：
- **安全修复** (11 个): 修复上游 2.14.2-2.14.6 版本的 CVE 漏洞
- **Bug 修复** (1 个): 修复上游 2.14.0 的功能回归
- **OH 特有贡献** (1 个): 华为工程师贡献的 RelaxNG 无限循环修复

所有 Patch 都在构建时通过 `install.py` 脚本自动应用到解压后的源码中。

---

## Patch 清单

| 序号 | Patch 文件 | CVE | 修改文件 | 类型 | 严重程度 |
|------|-----------|------|---------|------|----------|
| 1 | `Fix_XML_PARSE_NOBLANKS_dropping_non-whitespace_text.patch` | — | parser.c, testparser.c | Bug | — |
| 2 | `Backport-CVE-2025-32414-python-Read-at-most-len-4-ch-c.patch` | CVE-2025-32414 | python/libxml.c | Security | High (7.5) |
| 3 | `Backport-CVE-2025-32415-schemas-Fix-heap-buffer-over-c.patch` | CVE-2025-32415 | xmlschemas.c | Security | High |
| 4 | `Backport-CVE-2025-6021-tree-Fix-integer-overflow-in-xmlBuildQName-c.patch` | CVE-2025-6021 | tree.c | Security | High (7.5) |
| 5 | `Backport-CVE-2025-6170-Fix-potential-buffer-overflow-of-interactive-shell.patch` | CVE-2025-6170 | shell.c | Security | Medium |
| 6 | `Fix-CVE-2025-49794-CVE-2025-49796-memory-safety-issues-in-xmlSchematronReportOutput.patch` | CVE-2025-49794, CVE-2025-49796 | schematron.c | Security | Critical (9.1) |
| 7 | `Fix-CVE-2025-49795-null-pointer-dereference-leading-to-DoS.patch` | CVE-2025-49795 | schematron.c | Security | High (7.5) |
| 8 | `Fix-CVE-2025-8732-Prevent-infinite-recursion-in-xmlCatalogList.patch` | CVE-2025-8732 | catalog.c | Security | High |
| 9 | `Fix-CVE-2026-0990-catalog-prevent-inf-recursion-in-xmlCatalogXMLResolveURI.patch` | CVE-2026-0990 | catalog.c | Security | High |
| 10 | `Fix-CVE-2026-0992-catalog-Ignore-repeated-nextCatalog-entries.patch` | CVE-2026-0992 | catalog.c | Security | Medium |
| 11 | `Fix-CVE-2026-0989-Add-RelaxNG-include-limit.patch` | CVE-2026-0989 | relaxng.c | Security | High |
| 12 | `Fix-relaxng-is-parsed-to-an-infinite-attrs-next-loop.patch` | — | relaxng.c | Bug | — |

---

## 详细分析

### Patch 1: XML_PARSE_NOBLANKS Bug 修复

**文件名**: `Fix_XML_PARSE_NOBLANKS_dropping_non-whitespace_text.patch`

**来源**: 上游 2.14.2

**修改文件**: `parser.c`, `testparser.c`

**原始问题**:
在 libxml2 2.14.0 中，使用 `XML_PARSE_NOBLANKS` 选项时，解析器会错误地删除某些非空白文本节点。这是提交 1f5b5371 引入的回归问题。

**修改内容**:

```c
// 修改前: xmlCharacters() 函数未区分空白上下文
static void
xmlCharacters(xmlParserCtxtPtr ctxt, const xmlChar *buf, int size) {
    // ...
    if ((checkBlanks) &&
        (areBlanks(ctxt, buf, size, 1))) {  // 总是使用 isBlank=1
        // 跳过空白
    }
}

// 修改后: 根据 context 区分是否为空白
static void
xmlCharacters(xmlParserCtxtPtr ctxt, const xmlChar *buf, int size,
              int isBlank) {  // 新增参数
    // ...
    if ((checkBlanks) &&
        (areBlanks(ctxt, buf, size, isBlank))) {  // 使用传入的 isBlank
        // 跳过空白
    }
}
```

**关键变更**:
1. `xmlCharacters()` 函数新增 `isBlank` 参数
2. 所有调用点根据上下文传递正确的 `isBlank` 值 (0 或 1)
3. 添加测试用例 `testNoBlanks()` 验证修复

**OH 需求**:
修复解析器功能回归，确保 `XML_PARSE_NOBLANKS` 选项在 OH 中正常工作。此选项常用于配置文件解析，需要精确的空白处理。

**升级建议**:
此为上游 Bug 修复，升级到 libxml2 2.14.2+ 后无需保留该 Patch。

---

### Patch 2: CVE-2025-32414 Python 缓冲区溢出

**文件名**: `Backport-CVE-2025-32414-python-Read-at-most-len-4-ch-c.patch`

**来源**: 上游 2.14.2

**修改文件**: `python/libxml.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-32414
- **CVSS**: 7.5 (High)
- **类型**: 缓冲区溢出
- **影响**: Python 绑定接口

**原始问题**:
Python 绑定中的 `xmlPythonFileReadRaw()` 和 `xmlPythonFileRead()` 函数存在缓冲区溢出漏洞。当 Python `read()` 返回字符串（字符）而非字节时，长度计算错误，导致读取过多数据到缓冲区。

**修改内容**:

```c
// 修改前: 直接请求 len 字节
ret = PyObject_CallMethod(file, (char *) "read", (char *) "(i)", len);

// 修改后: 最多请求 len/4 字符（为 UTF-8 编码留出空间）
ret = PyObject_CallMethod(file, (char *) "read", (char *) "(i)", len / 4);
```

关键变更:
1. 读取长度从 `len` 改为 `len / 4`（UTF-8 最长 4 字节/字符）
2. 添加 `lenread` 范围检查: `if (lenread < 0 || lenread > len)`
3. 移除不安全的 `memcpy()` 逻辑

**OH 需求**:
修复 Python 绑定安全漏洞。

**OH 风险评估**:
**极低** - OpenHarmony 通常不使用 libxml2 的 Python 绑定，此 Patch 主要是防御性措施。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.2+ 后无需保留该 Patch。

---

### Patch 3: CVE-2025-32415 Schema 堆缓冲区下溢

**文件名**: `Backport-CVE-2025-32415-schemas-Fix-heap-buffer-over-c.patch`

**来源**: 上游 2.14.2

**修改文件**: `xmlschemas.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-32415
- **CVSS**: High
- **类型**: 缓冲区下溢
- **影响**: XSD Schema 验证

**原始问题**:
`xmlSchemaIDCFillNodeTables()` 函数在处理 Identity Constraints (IDC) 时使用错误的循环条件 `j < nbNodeTable`，导致读取 `nbDupls` 值而非 `bind->nbNodes`，触发堆缓冲区下溢。

**修改内容**:

```c
// 修改前: 错误的循环变量
while (j < nbNodeTable) {  // nbNodeTable 来自 nbDupls
    // ...
    j++;
}

// 修改后: 正确的循环变量
while (j < bind->nbNodes) {  // 使用 bind->nbNodes
    // ...
    j++;
}
```

关键变更:
1. 两处循环条件从 `j < nbNodeTable` 改为 `j < bind->nbNodes`
2. 修复范围: `xmlSchemaIDCFillNodeTables()` 函数中的两个循环

**OH 需求**:
修复 XSD Schema 验证中的内存安全问题。OH 中的配置验证可能使用 XSD Schema。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.2+ 后无需保留该 Patch。

---

### Patch 4: CVE-2025-6021 QName 整数溢出

**文件名**: `Backport-CVE-2025-6021-tree-Fix-integer-overflow-in-xmlBuildQName-c.patch`

**来源**: 上游 2.14.5

**修改文件**: `tree.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-6021
- **CVSS**: 7.5 (High)
- **类型**: 整数溢出
- **影响**: QName 构建

**原始问题**:
`xmlBuildQName()` 函数在计算 `ncname` 和 `prefix` 组合后的长度时，使用 `int` 类型，存在整数溢出风险。当两个字符串长度接近 INT_MAX 时，分配的缓冲区不足，导致缓冲区溢出。

**修改内容**:

```c
// 修改前: 使用 int 类型，无溢出检查
int lenn, lenp;
// ...
if ((memory == NULL) || (len < lenn + lenp + 2)) {
    ret = xmlMalloc(lenn + lenp + 2);  // 可能整数溢出
}

// 修改后: 使用 size_t 类型，添加溢出检查
size_t lenn, lenp;  // 改为 size_t
// ...
if (lenn >= SIZE_MAX - lenp - 1)  // 溢出检查
    return(NULL);
if ((memory == NULL) || ((size_t) len < lenn + lenp + 2)) {
    ret = xmlMalloc(lenn + lenp + 2);
}
```

关键变更:
1. `lenn` 和 `lenp` 类型从 `int` 改为 `size_t`
2. 添加整数溢出检查: `if (lenn >= SIZE_MAX - lenp - 1)`
3. 添加 `len` 非负检查: `if ((ncname == NULL) || (len < 0))`

**OH 需求**:
修复 QName 构建中的内存安全问题。QName 是 XML 中带命名空间的元素名，广泛使用。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.5+ 后无需保留该 Patch。

---

### Patch 5: CVE-2025-6170 Shell 缓冲区溢出

**文件名**: `Backport-CVE-2025-6170-Fix-potential-buffer-overflow-of-interactive-shell.patch`

**来源**: 上游 2.14.5

**修改文件**: `shell.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-6170
- **CVSS**: Medium
- **类型**: 缓冲区溢出
- **影响**: xmllint 交互式 shell

**原始问题**:
xmllint 交互式 shell 在读取命令行时未限制输入长度，存在栈缓冲区溢出风险。`prompt`、`command` 和 `arg` 缓冲区大小为 500/100/400，但 `fgets()` 和循环读取未严格限制。

**修改内容**:

```c
// 新增常量定义
#define MAX_PROMPT_SIZE     500
#define MAX_ARG_SIZE        400
#define MAX_COMMAND_SIZE    100

// 修改前: 无长度限制的循环
while ((*cur != '\n') && (*cur != '\r') && (*cur != 0)) {
    command[i++] = *cur++;  // 可能越界
}

// 修改后: 添加边界检查
while ((*cur != '\n') && (*cur != '\r') &&
       (*cur != 0) && (i < (MAX_COMMAND_SIZE - 1))) {
    command[i++] = *cur++;  // 限制在 MAX_COMMAND_SIZE-1
}
```

关键变更:
1. 定义明确的缓冲区大小常量
2. 修复 `xmllintShellReadline()` 中的 `fgets()` 缓冲区大小
3. 为 `command` 和 `arg` 添加循环边界检查
4. 修复缓冲区截断逻辑

**OH 需求**:
修复 xmllint 工具的安全漏洞。

**OH 风险评估**:
**极低** - xmllint 是调试工具，不是系统关键组件。OH 中 xmllint 使用受限。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.5+ 后无需保留该 Patch。

---

### Patch 6: CVE-2025-49794/49796 Schematron 内存安全

**文件名**: `Fix-CVE-2025-49794-CVE-2025-49796-memory-safety-issues-in-xmlSchematronReportOutput.patch`

**来源**: 上游 2.14.5

**修改文件**: `schematron.c`

**CVE 信息**:
- **CVE IDs**: CVE-2025-49794, CVE-2025-49796
- **CVSS**: 9.1 (Critical)
- **类型**: Use-After-Free, Type Confusion
- **影响**: Schematron 验证

**原始问题**:

1. **CVE-2025-49794 (Use-After-Free)**:
   `xmlSchematronGetNode()` 在获取 XPath 结果后立即调用 `xmlXPathFreeObject()`，然后使用已释放的内存。

2. **CVE-2025-49796 (Type Confusion)**:
   `xmlSchematronFormatReport()` 假设 XPath 结果总是节点集合，但可能是其他类型（字符串、数字等），导致类型混淆和内存损坏。

**修改内容**:

```c
// 修改前: 立即释放 XPath 对象
static xmlNodePtr
xmlSchematronGetNode(xmlSchematronValidCtxtPtr ctxt, xmlNodePtr cur, const xmlChar *xpath) {
    // ...
    ret = xmlXPathEval(xpath, ctxt->xctxt);
    if (ret == NULL)
        return(NULL);
    if ((ret->type == XPATH_NODESET) && (ret->nodesetval != NULL) && (ret->nodesetval->nodeNr > 0))
        node = ret->nodesetval->nodeTab[0];
    xmlXPathFreeObject(ret);  // 立即释放
    return(node);  // 使用可能已释放的 node
}

// 修改后: 返回 XPath 对象，调用者负责释放
static xmlXPathObjectPtr  // 返回类型改为 xmlXPathObjectPtr
xmlSchematronGetNode(xmlSchematronValidCtxtPtr ctxt, xmlNodePtr cur, const xmlChar *xpath) {
    // ...
    return(xmlXPathEval(xpath, ctxt->xctxt));  // 返回对象
}

// 调用者正确使用
obj = xmlSchematronGetNode(ctxt, cur, path);
if ((obj != NULL) &&
    (obj->type == XPATH_NODESET) &&  // 检查类型
    (obj->nodesetval != NULL) &&
    (obj->nodesetval->nodeNr > 0))
    node = obj->nodesetval->nodeTab[0];
xmlXPathFreeObject(obj);  // 使用后释放
```

关键变更:
1. `xmlSchematronGetNode()` 返回类型从 `xmlNodePtr` 改为 `xmlXPathObjectPtr`
2. 调用者负责检查类型和释放对象
3. `xmlSchematronFormatReport()` 添加类型检查 switch 语句
4. 新增测试用例验证 CVE-2025-49794/49796

**OH 需求**:
修复 Schematron 验证中的关键内存安全问题。

**OH 风险评估**:
**高** - 如果 OH 使用 Schematron 进行配置/数据验证，恶意 XML 文件可能导致远程代码执行。

**升级建议**:
此为上游 Critical 安全修复，升级到 libxml2 2.14.5+ 后无需保留该 Patch。

**特别说明**:
上游计划在 2.15+ 版本移除 Schematron 功能（已多次出现严重安全问题）。OH 应评估是否继续使用此功能。

---

### Patch 7: CVE-2025-49795 空指针解引用

**文件名**: `Fix-CVE-2025-49795-null-pointer-dereference-leading-to-DoS.patch`

**来源**: 上游 2.14.5

**修改文件**: `schematron.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-49795
- **CVSS**: 7.5 (High)
- **类型**: 空指针解引用 (DoS)
- **影响**: Schematron 验证

**原始问题**:
`xmlSchematronFormatReport()` 在处理 XPath 表达式结果时未检查 `eval` 是否为 NULL。当 XPath 包含无效函数（如 `falae()` 而非 `false()`）时，`xmlXPathCompiledEval()` 返回 NULL，导致空指针解引用和崩溃。

**修改内容**:

```c
// 修改前: 未检查 eval 是否为 NULL
select = xmlGetNoNsProp(child, BAD_CAST "select");
comp = xmlXPathCtxtCompile(ctxt->xctxt, select);
eval = xmlXPathCompiledEval(comp, ctxt->xctxt);  // 可能返回 NULL

switch (eval->type) {  // eval 为 NULL 时崩溃
case XPATH_NODESET: { ... }
}

// 修改后: 添加 NULL 检查
select = xmlGetNoNsProp(child, BAD_CAST "select");
comp = xmlXPathCtxtCompile(ctxt->xctxt, select);
eval = xmlXPathCompiledEval(comp, ctxt->xctxt);

if (eval == NULL) {  // 新增检查
    xmlXPathFreeCompExpr(comp);
    xmlFree(select);
    return ret;
}

switch (eval->type) {
case XPATH_NODESET: { ... }
}
```

关键变更:
1. 添加 `eval` NULL 检查
2. NULL 时正确清理资源（释放 `comp` 和 `select`）
3. 返回而非继续执行
4. 新增测试用例验证修复

**OH 需求**:
修复 Schematron 验证中的空指针解引用 DoS。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.5+ 后无需保留该 Patch。

---

### Patch 8-10: Catalog 无限递归防护

#### Patch 8: CVE-2025-8732 - xmlCatalogList 无限递归

**文件名**: `Fix-CVE-2025-8732-Prevent-infinite-recursion-in-xmlCatalogList.patch`

**来源**: 上游 2.14.5+

**修改文件**: `catalog.c`

**CVE 信息**:
- **CVE ID**: CVE-2025-8732
- **CVSS**: High
- **类型**: 无限递归 DoS
- **影响**: SGML Catalog 解析

**原始问题**:
`xmlExpandCatalog()` 和 `xmlParseSGMLCatalog()` 在递归处理 `<delegate>` 和 `<nextCatalog>` 条目时未检查递归深度。恶意 catalog 文件可以创建循环引用，导致栈溢出和 DoS。

**修改内容**:

```c
// 修改前: 无递归深度检查
static int
xmlExpandCatalog(xmlCatalogPtr catal, const char *filename) {
    // ...
    if (catal->type == XML_SGML_CATALOG_TYPE) {
        // ...
        ret = xmlParseSGMLCatalog(catal, content, filename, 0);  // 递归
    }
}

// 修改后: 添加 depth 参数和检查
static int
xmlExpandCatalog(xmlCatalogPtr catal, const char *filename, int depth) {  // 新增 depth
    // ...
    if (depth > MAX_CATAL_DEPTH) {  // 深度检查
        return(-1);
    }
    if (catal->type == XML_SGML_CATALOG_TYPE) {
        // ...
        ret = xmlParseSGMLCatalog(catal, content, filename, 0, depth + 1);  // 传递 depth+1
    }
}
```

关键变更:
1. `xmlExpandCatalog()` 新增 `depth` 参数
2. `xmlParseSGMLCatalog()` 新增 `depth` 参数
3. 添加递归深度检查: `if (depth > MAX_CATAL_DEPTH)`
4. 所有调用点传递正确的 depth 值

**MAX_CATAL_DEPTH**: 默认值通常为 100。

---

#### Patch 9: CVE-2026-0990 - xmlCatalogXMLResolveURI 无限递归

**文件名**: `Fix-CVE-2026-0990-catalog-prevent-inf-recursion-in-xmlCatalogXMLResolveURI.patch`

**来源**: 上游 2.14.6+

**修改文件**: `catalog.c`

**CVE 信息**:
- **CVE ID**: CVE-2026-0990
- **CVSS**: High
- **类型**: 无限递归 DoS
- **影响**: XML Catalog 解析

**原始问题**:
`xmlCatalogListXMLResolveURI()` 在遍历 `<nextCatalog>` 条目时未检查递归深度，可能导致栈溢出。

**修改内容**:

```c
// 修改前: 无递归深度检查
static xmlChar *
xmlCatalogListXMLResolveURI(xmlCatalogEntryPtr catal, const xmlChar *URI) {
    while (catal != NULL) {
        if (catal->type == XML_CATA_CATALOG) {
            if (catal->children != NULL) {
                ret = xmlCatalogXMLResolveURI(catal->children, URI);  // 递归
            }
        }
        catal = catal->next;
    }
}

// 修改后: 添加 depth 跟踪和检查
static xmlChar *
xmlCatalogListXMLResolveURI(xmlCatalogEntryPtr catal, const xmlChar *URI) {
    if (catal->depth > MAX_CATAL_DEPTH) {  // 深度检查
        xmlCatalogErr(catal, NULL, XML_CATALOG_RECURSION,
                  "Detected recursion in catalog %s\n",
                  catal->name, NULL, NULL);
        return(NULL);
    }
    catal->depth++;  // 递增 depth

    cur = catal;
    while (cur != NULL) {
        if (cur->type == XML_CATA_CATALOG) {
            if (cur->children != NULL) {
                ret = xmlCatalogXMLResolveURI(cur->children, URI);
                if (ret != NULL) {
                    catal->depth--;  // 递减 depth
                    return(ret);
                }
            }
        }
        cur = cur->next;
    }

    catal->depth--;  // 递减 depth
    return(ret);
}
```

关键变更:
1. 使用 `catalogEntryPtr->depth` 字段跟踪递归深度
2. 添加递归深度检查
3. 正确管理 depth 的递增和递减

---

#### Patch 10: CVE-2026-0992 - Catalog 重复 nextCatalog

**文件名**: `Fix-CVE-2026-0992-catalog-Ignore-repeated-nextCatalog-entries.patch`

**来源**: 上游 2.14.6+

**修改文件**: `catalog.c`

**CVE 信息**:
- **CVE ID**: CVE-2026-0992
- **CVSS**: Medium
- **类型**: Catalog 解析优化/DoS 缓解
- **影响**: XML Catalog 解析

**原始问题**:
XML Catalog 中的重复 `<nextCatalog>` 条目会导致多次解析相同的 catalog 文件，增加 DoS 攻击面。

**修改内容**:

```c
// 修改前: 允许重复 nextCatalog
} else if (xmlStrEqual(cur->name, BAD_CAST "nextCatalog")) {
    entry = xmlParseXMLCatalogOneNode(cur, XML_CATA_NEXT_CATALOG,
            BAD_CAST "nextCatalog", NULL,
            BAD_CAST "catalog", prefer, cgroup);
}

// 修改后: 检查并忽略重复项
} else if (xmlStrEqual(cur->name, BAD_CAST "nextCatalog")) {
    xmlCatalogEntryPtr prev = parent->children;

    entry = xmlParseXMLCatalogOneNode(cur, XML_CATA_NEXT_CATALOG,
            BAD_CAST "nextCatalog", NULL,
            BAD_CAST "catalog", prefer, cgroup);

    /* Avoid duplication of nextCatalog */
    while (prev != NULL) {
        if ((prev->type == XML_CATA_NEXT_CATALOG) &&
            (xmlStrEqual(prev->URL, entry->URL)) &&  // 检查 URL
            (prev->prefer == entry->prefer) &&     // 检查 prefer
            (prev->group == entry->group)) {       // 检查 group
            if (xmlDebugCatalogs)
                xmlCatalogPrintDebug(
                    "Ignoring repeated nextCatalog %s\n", entry->URL);
            xmlFreeCatalogEntry(entry, NULL);
            entry = NULL;
            break;
        }
        prev = prev->next;
    }
}
```

关键变更:
1. 在添加 `nextCatalog` 条目前遍历现有条目
2. 检查重复条件: `type`、`URL`、`prefer`、`group` 全部相同
3. 发现重复时释放新条目并忽略
4. 添加调试日志输出

**OH 需求**:
缓解 Catalog 解析中的 DoS 风险。

**升级建议**:
这些为上游安全修复，升级到 libxml2 2.14.6+ 后无需保留这些 Patch。

---

### Patch 11: CVE-2026-0989 RelaxNG 包含限制

**文件名**: `Fix-CVE-2026-0989-Add-RelaxNG-include-limit.patch`

**来源**: 上游 2.14.6+

**修改文件**: `include/libxml/relaxng.h`, `relaxng.c`, `runtest.c`

**CVE 信息**:
- **CVE ID**: CVE-2026-0989
- **CVSS**: High
- **类型**: 栈溢出 DoS
- **影响**: RELAX NG 解析

**原始问题**:
`xmlRelaxNGIncludePush()` 在处理 `<include>` 指令时未限制递归深度。恶意 RelaxNG schema 可以创建无限包含链，导致栈溢出和 DoS。

**修改内容**:

```c
// 新增默认包含限制
static const int _xmlRelaxNGIncludeLimit = 1000;

// 解析器上下文新增 incLimit 字段
struct _xmlRelaxNGParserCtxt {
    // ...
    int incLimit;  /* Include limit, to avoid stack-overflow on parse */
};

// 新增 API: 设置包含限制
int
xmlRelaxParserSetIncLImit(xmlRelaxNGParserCtxt *ctxt, int limit)
{
    if (ctxt == NULL) return(-1);
    if (limit < 0) return(-1);
    ctxt->incLimit = limit;
    return(0);
}

// 修改 include 推入逻辑
static int
xmlRelaxNGIncludePush(xmlRelaxNGParserCtxtPtr ctxt, ...) {
    // ...
    if (ctxt->incNr >= ctxt->incLimit) {  // 添加限制检查
        xmlRngPErr(ctxt, (xmlNodePtr)value->doc, XML_RNGP_PARSE_ERROR,
                   "xmlRelaxNG: inclusion recursion limit reached\n", NULL, NULL);
        return(-1);
    }
    // ...
}

// 解析时初始化 incLimit
xmlRelaxNGParse(xmlRelaxNGParserCtxtPtr ctxt) {
    // ...
    if (ctxt->incLimit == 0) {
        ctxt->incLimit = _xmlRelaxNGIncludeLimit;  // 默认 1000
        if (include_limit_env != NULL) {
            // 支持环境变量配置
            char *strEnd;
            unsigned long val = 0;
            errno = 0;
            val = strtoul(include_limit_env, &strEnd, 10);
            if (errno != 0 || *strEnd != 0 || val > INT_MAX) {
                xmlRngPErr(ctxt, NULL, XML_RNGP_PARSE_ERROR,
                           "xmlRelaxNGParse: invalid RNG_INCLUDE_LIMIT %s\n",
                           (const xmlChar*)include_limit_env, NULL);
                return(NULL);
            }
            if (val)
                ctxt->incLimit = val;
        }
    }
    // ...
}
```

关键变更:
1. 默认包含限制: `_xmlRelaxNGIncludeLimit = 1000`
2. 环境变量支持: `RNG_INCLUDE_LIMIT` 可动态调整
3. 新增 API: `xmlRelaxParserSetIncLImit()` 允许程序设置
4. `xmlRelaxNGIncludePush()` 添加超限检查
5. 新增测试用例验证 3 层包含（limit=2 应失败，limit=3 应成功）

**OH 需求**:
防止 RelaxNG schema 解析中的栈溢出 DoS。

**升级建议**:
此为上游安全修复，升级到 libxml2 2.14.6+ 后无需保留该 Patch。

---

### Patch 12: RelaxNG 无限循环修复（OH 特有贡献）

**文件名**: `Fix-relaxng-is-parsed-to-an-infinite-attrs-next-loop.patch`

**来源**: 华为工程师 (l30034438@notesmail.huawei.com)

**修改文件**: `relaxng.c`

**提交**: Change-Id: I2b7b37c628b20bd234b84d8938d241dd722af087

**原始问题**:
`xmlRelaxNGSimplify()` 在简化 RelaxNG schema 时可能创建无限的 `attrs->next` 循环。特定模式的 schema（如包含递归引用的无用 `<group>`）在简化过程中触发此问题。

**修改内容**:

```c
// 修改前: 简化逻辑未检查重复
if (prev == NULL) {
    parent->content = cur->content;
    cur->content->next = cur->next;
    cur = cur->content;  // 可能已发生过，导致循环
}

// 修改后: 检查是否已简化过
if (prev == NULL) {
    /*
     * this simplification may already have happened
     * if this is done twice this leads to an infinite loop of attrs->next
     */
    if (parent->content != cur->content) {  // 新增检查
        parent->content = cur->content;
        cur->content->next = cur->next;
        cur = cur->content;
    }
}
```

关键变更:
1. 在简化前检查 `parent->content != cur->content`
2. 如果已简化过，跳过简化步骤
3. 添加注释说明问题根因

**测试用例**:
```xml
<!-- useless_group.rng -->
<grammar xmlns="http://relaxng.org/ns/structure/1.0">
  <start>
    <ref name="listOfLists"/>
  </start>
  <define name="listOfLists">
    <element name="listOfLists">
      <group>
        <zeroOrMore>
          <ref name="listOfLists"/>
        </zeroOrMore>
      </group>
      <optional>
        <attribute name="fail"/>
      </optional>
    </element>
  </define>
</grammar>
```

**OH 需求**:
修复 RelaxNG schema 解析中的无限循环 DoS。此是 OH 发现并贡献给上游的修复。

**升级建议**:
此为 OH 特有贡献，已推向上游。升级到包含此修复的上游版本后无需保留该 Patch。建议检查上游 issue/commit 跟踪状态。

---

## Patch 分类总结

### 按严重程度

| 严重程度 | 数量 | CVEs |
|---------|------|-------|
| Critical (9.1) | 2 | CVE-2025-49794, CVE-2025-49796 |
| High (7.5+) | 7 | CVE-2025-32414, CVE-2025-32415, CVE-2025-6021, CVE-2025-49795, CVE-2025-8732, CVE-2026-0989, CVE-2026-0990, CVE-2026-0990 |
| Medium | 2 | CVE-2025-6170, CVE-2026-0992 |
| 非安全 | 2 | XML_PARSE_NOBLANKS, RelaxNG 无限循环 |

### 按影响模块

| 模块 | CVEs 数量 | 关键漏洞 |
|------|-----------|---------|
| Schematron | 3 | CVE-2025-49794/95/96 (Critical) |
| Catalog | 3 | CVE-2025-8732, CVE-2026-0990/92 |
| RelaxNG | 2 | CVE-2026-0989, 无限循环 |
| XML Schema | 1 | CVE-2025-32415 |
| Tree/QName | 1 | CVE-2025-6021 |
| Python | 1 | CVE-2025-32414 |
| Shell | 1 | CVE-2025-6170 |
| Parser | 1 | XML_PARSE_NOBLANKS (非 CVE) |

---

## 上游版本对应关系

| OH Patch | 对应上游版本 | 说明 |
|---------|------------|------|
| XML_PARSE_NOBLANKS | 2.14.2 | Bug 修复 |
| CVE-2025-32414, CVE-2025-32415 | 2.14.2 | 安全修复 |
| CVE-2025-49794/95/96, CVE-2025-6021, CVE-2025-6170 | 2.14.5 | 安全修复 |
| CVE-2025-8732, CVE-2026-0989/0990/0992 | 2.14.6 | 安全修复 |
| RelaxNG 无限循环 | OH 贡献 | 待合入上游 |

---

## 升级建议

### 短期（维持当前版本）
1. **持续跟踪**: 监控上游 2.14.x 系列的新安全更新
2. **及时 Backport**: 发现新 CVE 时立即评估并应用 Patch
3. **评估 Schematron**: 由于多次 Critical 安全问题，考虑在 OH 中禁用该功能

### 中期（升级到 2.14.6）
1. **直接升级**: libxml2 2.14.6 包含所有已应用的 Patch
2. **保留 OH 特有 Patch**: 仅保留 RelaxNG 无限循环修复（如未合入上游）
3. **验证测试**: 全面测试所有依赖模块（80+ 模块）

### 长期（升级到 2.15+）
1. **提前评估**: 2.15+ 将移除多项功能（HTTP, Schematron, Modules API 等）
2. **迁移计划**: 为移除的功能寻找替代方案
3. **ABI 破坏**: 2.15+ 可能破坏二进制兼容性，需要全系统重新编译

---

*最后更新: 2026-02-07*

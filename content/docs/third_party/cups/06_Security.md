# CUPS 安全风险分析

## CVE 修复记录

本章节记录 CUPS v2.4.14 在 OpenHarmony 中的所有安全修复。

---

## CVE 修复清单

| CVE 编号 | 严重程度 | 漏洞类型 | 修复状态 |
|---------|---------|---------|---------|
| CVE-2022-26691 | 待确认 | 认证绕过 | ✅ 已修复 |
| CVE-2023-32324 | 待确认 | 缓冲区处理 | ✅ 已修复 |
| CVE-2023-34241 | 待确认 | Use-after-free | ✅ 已修复 |
| CVE-2023-4504 | 待确认 | 缓冲区溢出 | ✅ 已修复 |
| CVE-2024-35235 | 待确认 | 路径验证 | ✅ 已修复 |
| CVE-2025-59364 | 待确认 | IPP 协议解析 | ✅ 已修复 |

---

## CVE 详细分析

### CVE-2022-26691 - 认证绕过

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2022-26691.patch` |
| **影响文件** | `scheduler/cert.c` |
| **漏洞类型** | 字符串比较逻辑缺陷 |
| **CVSS 评分** | 待查询 NVD |

**漏洞描述**：

`ctcompare()` 证书字符串比较函数未正确处理不同长度的字符串。

**问题代码**：

```c
// 修改前
static int ctcompare(const char *a, const char *b) {
    int result = strcmp(a, b);
    return (result);  // 问题：短字符串可能被错误匹配
}
```

**修复代码**：

```c
// 修改后
static int ctcompare(const char *a, const char *b) {
    int result = strcmp(a, b);
    return (result | *a | *b);  // 确保两字符串都到达 NULL 终止符
}
```

**安全影响**：

攻击者可能通过构造特定字符串绕过证书验证。

---

### CVE-2023-32324 - 缓冲区处理缺陷

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2023-32324.patch` |
| **影响文件** | `cups/string.c` |
| **漏洞类型** | 边界条件处理 |
| **CVSS 评分** | 待查询 NVD |

**漏洞描述**：

`_cups_strlcpy()` 函数未处理 `size` 参数为 0 的情况。

**修复代码**：

```c
// string.c

// 新增边界检查
size_t _cups_strlcpy(char *dst, const char *src, size_t size) {
    if (size == 0) {
        return 0;  // 防止未定义行为
    }
    // ... 原有逻辑
}
```

---

### CVE-2023-34241 - Use-after-free

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2023-34241.patch` |
| **影响文件** | `scheduler/client.c` |
| **漏洞类型** | 内存释放后使用 |
| **CVSS 评分** | 待查询 NVD |

**漏洞描述**：

`httpClose()` 释放 `con->http` 内存后，`httpGetHostname()` 仍尝试访问已释放内存。

**修复方案**：

```c
// 修改前
void client_cleanup(client_t *con) {
    httpClose(con->http);    // 释放内存
    const char *hostname = httpGetHostname(con->http, ...);  // 访问已释放内存
    log_message("closed %s", hostname);
}

// 修改后
void client_cleanup(client_t *con) {
    const char *hostname = httpGetHostname(con->http, ...);  // 先获取
    httpClose(con->http);                                    // 后释放
    log_message("closing %s", hostname);                     // 日志调整
}
```

---

### CVE-2023-4504 - 缓冲区溢出

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2023-4504.patch` |
| **影响文件** | `cups/raster-interpret.c` |
| **漏洞类型** | 缓冲区越界读取 |
| **CVSS 评分** | 待查询 NVD |

**漏洞描述**：

PostScript 光栅图像解析时未检查缓冲区末尾。

**修复代码**：

```c
// raster-interpret.c

// 解析转义字符
while (*cur) {
    if (*cur == '\\') {  // 转义字符
        cur++;
        if (!*cur) {     // 新增：检查缓冲区末尾
            *ptr = NULL;
            return NULL;
        }
    }
    *ptr++ = *cur++;
}
```

---

### CVE-2024-35235 - 域套接字路径验证 (OH 特有)

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2024-35235.patch` |
| **影响文件** | `cups/http-addr.c`, `scheduler/conf.c` |
| **漏洞类型** | 路径验证缺陷 |
| **CVSS 评分** | 待查询 NVD |
| **备注** | OH 特有回溯 (含 Change-Id) |

**漏洞描述**：

域套接字处理存在路径验证问题，可能导致路径遍历攻击。

**修复代码**：

```c
// http-addr.c

// 新增 unlink 状态检查
if (unlink(socket_path) < 0) {
    if (errno != ENOENT) {  // 忽略 "文件不存在" 错误
        return -1;
    }
}

// conf.c

// 新增路径长度验证
size_t max_path = sizeof(addr->addr.un.sun_path) - 1;
if (socket_path_len > max_path) {
    return CUPS_SET_ERROR_STR(ERROR_TOO_LONG,
                              "Domain socket path too long");
}
```

---

### CVE-2025-59364 - IPP 协议解析

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-CVE-2025-59364.patch` |
| **影响文件** | `cups/ipp.c` |
| **漏洞类型** | IPP 标签解析 |
| **CVSS 评分** | 待查询 NVD |

**漏洞描述**：

IPP 扩展标签处理存在验证绕过漏洞。

**修复代码**：

```c
// ipp.c

// 移除有问题的 32 位扩展标签读取逻辑
// 新增错误处理
if (ippTagOf(name) == IPP_TAG_ZERO) {
   cupsdLogMessage(CUPSD_LOG_ERROR,
                   "Unable to read IPP attribute name");
    return NULL;
}
```

---

## 非 CVE 安全修复

### 内存泄漏修复

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `backport-Fix-memory-leak-in-cupsConvertOption.patch` |
| **影响文件** | `cups/ppd-cache.c` |
| **问题** | `ippAddCollection()` 创建的 collection 未释放 |

**修复代码**：

```c
// ppd-cache.c

// 添加 collection 释放
ipp_t *media_size = ippNew();
ippAddInteger(media_size, IPP_TAG_PRINTER, "media-size-supported", size);
// ... 添加内容
ippAddCollection(output, IPP_TAG_PRINTER, "media-size-supported", media_size);
ippDelete(media_size);  // 新增：释放内存
```

---

## OHOS 特有安全功能

### 日志数据脱敏

**Patch**: `cups-log-datamasking.patch`

**功能**：

| 脱敏字段 | 脱敏方式 |
|---------|---------|
| `job-name` | 首字符保留，其余用 `*` 替换 |
| `document-name` | 首字符保留，其余用 `*` 替换 |
| `requesting-user-name` | 替换为 `<anonymous>` |
| 密码相关 | 替换为 `<masked>` |

**实现文件**：

- `scheduler/datamasking.c` - 脱敏逻辑
- `scheduler/datamasking.h` - 头文件

**修改的文件**：

- `backend/ipp.c`
- `scheduler/auth.c`
- `scheduler/ipp.c`
- `scheduler/job.c`
- `scheduler/process.c`

**合规价值**：

- 符合 OHOS 隐私保护要求
- 防止 PII (个人身份信息) 泄露
- 满足安全审计要求

---

### 凭证存储修改

**Patch**: `ohos-ipp-authenticate.patch`

**修改前**：

- 凭证存储在文件 `/var/spool/cups/aXXXXX`
- 凭证以 root 权限保存
- 支持持久化凭证

**修改后**：

- 凭证仅存储在内存 `job->auth_env[]`
- 无文件操作
- 无 root 权限凭证文件

**安全优势**：

| 优势 | 说明 |
|-----|------|
| 防止凭证泄露 | 无持久化存储 |
| 容器友好 | 无特权文件依赖 |
| 最小权限 | 无 root 文件访问 |

---

## 安全风险评估

### 已知风险

| 风险项 | 级别 | 说明 |
|-------|------|------|
| CVE Patch 状态 | 低 | 所有已知 CVE 已修复 |
| 日志脱敏 | 低 | 敏感信息已脱敏 |
| 凭证存储 | 低 | 已适配 OHOS 安全模型 |
| USB 传输 | 中 | USB 传输使用 OH 服务，需确保服务安全 |

### 建议措施

1. **定期 CVE 检查**
   - 订阅 OpenPrinting CUPS 安全公告
   - 每月检查新 CVE

2. **上游同步**
   - 跟踪上游 v2.4.x 安全版本
   - 及时回溯新 CVE 修复

3. **OH 特有 Patch 安全审查**
   - `ohos-usb-*.patch` - USB 传输安全
   - `hilog` 集成 - 日志安全

4. **测试覆盖**
   - 启用所有模糊测试
   - 定期安全扫描

---

## 升级建议

### CVE Patch 处理策略

| Patch 类型 | 策略 |
|-----------|------|
| **上游 CVE** | 及时回溯，与上游同步 |
| **OH 特有** | 单独维护，跟踪 OH USB API 变化 |

### 安全测试

```bash
# 运行 CUPS 模糊测试
# 位置：base/print/print_fwk/test/fuzztest/

# 编译并运行
# - printcupsclient_fuzzer
# - printcupsattribute_fuzzer
# - printservicefunction_fuzzer
# ...
```

---

## 相关文档

- [02_Patches.md](02_Patches.md) - Patch 详解
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用情况

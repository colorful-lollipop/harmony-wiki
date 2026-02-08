# fsverity-utils 安全风险分析

## 6.1 安全概述

### 风险评估总结

| 风险维度 | 等级 | 说明 |
|----------|------|------|
| 自身安全 | 低 | 库本身无已知漏洞 |
| 依赖安全 | 中 | 依赖 OpenSSL |
| 攻击面 | 低 | 仅被核心签名模块使用 |
| 安全更新 | 低 | 可快速跟随上游 |

---

## 6.2 已知漏洞分析

### 上游漏洞状态

**结论**：fsverity-utils 本身**无已知 CVE 漏洞**。

原因分析：
1. 库代码量小，功能单一
2. 不涉及复杂的安全逻辑
3. 主要依赖 OpenSSL 的安全实现

### 依赖组件漏洞

| 组件 | 漏洞状态 | 缓解措施 |
|------|----------|----------|
| OpenSSL | 需关注 CVE | OH 安全更新机制 |

---

## 6.3 攻击面分析

### 代码攻击面

```
┌─────────────────────────────────────────────────────────────┐
│                   fsverity-utils 攻击面                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  输入点                                                      │
│  ├── 用户提供的文件描述符                                     │
│  ├── 读取回调函数                                            │
│  ├── Merkle 树参数                                           │
│  ├── 盐值                                                    │
│  ├── 签名参数（密钥文件、证书）                                │
│  └── 签名数据                                                │
│                                                              │
│  风险评估                                                    │
│  ├── 文件描述符：低风险（仅读）                               │
│  ├── 读取回调：低风险（用户控制）                              │
│  ├── 参数验证：中风险（需严格验证）                            │
│  ├── 盐值：低风险（仅用于哈希）                               │
│  ├── 签名参数：低风险（文件路径）                              │
│  └── 签名数据：低风险（仅验证）                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 潜在风险点

| 风险点 | 风险等级 | 原因 |
|--------|----------|------|
| 缓冲区溢出 | 低 | 使用安全字符串函数 |
| 整数溢出 | 中 | 需验证算术运算 |
| 空指针解引用 | 低 | 有参数检查 |
| 资源耗尽 | 中 | 大文件处理 |
| 路径遍历 | 低 | 不涉及文件路径 |

---

## 6.4 安全机制

### 1. 参数验证

```c
// 源文件：lib/compute_digest.c

// 版本检查
if (params->version != 1) {
    libfsverity_error_msg("unsupported version (%u)",
                          params->version);
    return -EINVAL;
}

// 块大小验证
if (!is_power_of_2(block_size)) {
    libfsverity_error_msg("unsupported block size (%u)",
                          block_size);
    return -EINVAL;
}

// 盐值大小验证
if (params->salt_size > sizeof(desc.salt)) {
    libfsverity_error_msg("unsupported salt size (%u)",
                          params->salt_size);
    return -EINVAL;
}

// 保留字段验证
if (!libfsverity_mem_is_zeroed(params->reserved1,
                               sizeof(params->reserved1)) ||
    !libfsverity_mem_is_zeroed(params->reserved2,
                               sizeof(params->reserved2))) {
    libfsverity_error_msg("reserved bits set in merkle_tree_params");
    return -EINVAL;
}
```

### 2. 内存安全

```c
// 使用安全的内存分配
ctx = libfsverity_zalloc(sizeof(*ctx));  // 零初始化

// 内存复制安全
*digest_ret = libfsverity_memdup(pkcs7_data, pkcs7_data_len);

// 始终检查返回值
if (!ctx->md_ctx) {
    libfsverity_error_msg("failed to allocate EVP_MD_CTX");
    goto err1;
}
```

### 3. 哈希算法安全

```c
// 源文件：lib/hash_algs.c

// 只使用经过验证的哈希算法
static const struct fsverity_hash_alg fsverity_hash_algs[] = {
    [FS_VERITY_HASH_ALG_SHA256] = {
        .name = "sha256",
        .digest_size = 32,
        .block_size = 64,
        .create_ctx = create_sha256_ctx,
    },
    [FS_VERITY_HASH_ALG_SHA512] = {
        .name = "sha512",
        .digest_size = 64,
        .block_size = 128,
        .create_ctx = create_sha512_ctx,
    },
};
```

### 4. 错误处理

```c
// 统一的错误回调机制
static void (*libfsverity_error_cb)(const char *msg);

LIBEXPORT void
libfsverity_set_error_callback(void (*cb)(const char *msg))
{
    libfsverity_error_cb = cb;
}

// 错误信息格式化
void libfsverity_error_msg(const char *format, ...)
{
    va_list va;

    va_start(va, format);
    libfsverity_do_error_msg(format, va);
    va_end(va);
}
```

---

## 6.5 OpenSSL 安全集成

### OpenSSL 使用

| 功能 | OpenSSL 组件 | 安全考虑 |
|------|--------------|----------|
| 哈希计算 | EVP API | 使用现代 EVP 接口 |
| X.509 证书 | X509, EVP_PKEY | 证书验证由调用者处理 |
| PKCS#7 签名 | PKCS7 API | 使用标准 PKCS#7 格式 |

### OpenSSL 版本要求

```gn
# BUILD.gn 中依赖声明
external_deps = [ "openssl:libcrypto_shared" ]
```

**最低版本**：OpenSSL 1.0.0

### 安全最佳实践

```c
// ✅ 正确：使用现代 EVP API
const EVP_MD *md = EVP_get_digestbyname(hash_alg->name);
EVP_DigestInit_ex(ctx->md_ctx, md, NULL);

// ❌ 避免：使用废弃的、低层 API
// SHA256_Update(), SHA256_Final() 等
```

---

## 6.6 在 OH 中的安全考量

### 使用场景安全

fsverity-utils 在 OH 中主要用于 **代码签名** 场景：

```
┌─────────────────────────────────────────────────────────────┐
│                   代码签名安全流程                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 证书验证                                                  │
│     └─► 验证证书链、有效期、撤销状态                          │
│                                                              │
│  2. 密钥安全                                                  │
│     └─► 私钥存储在安全区域                                    │
│                                                              │
│  3. 摘要计算                                                  │
│     └─► 使用 fsverity-utils libfsverity_compute_digest()    │
│                                                              │
│  4. 签名生成                                                  │
│     └─► 使用 fsverity-utils libfsverity_sign_digest()        │
│                                                              │
│  5. 启用 fs-verity                                           │
│     └─► 使用 fsverity-utils libfsverity_enable_with_sig()    │
│                                                              │
│  6. 内核验证                                                  │
│     └─► FS_IOC_ENABLE_VERITY ioctl                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### OH 安全增强

| 安全机制 | 实现位置 | 说明 |
|----------|----------|------|
| 参数验证 | FsverityUtilsHelper | 在调用 libfsverity 前验证 |
| 错误处理 | FsverityUtilsHelper | 统一错误处理和日志 |
| 资源管理 | FsverityUtilsHelper | RAII 风格的资源管理 |

---

## 6.7 安全测试

### 测试类型

| 测试类型 | 覆盖范围 | 执行频率 |
|----------|----------|----------|
| 单元测试 | API 正确性 | 每次构建 |
| 集成测试 | 与内核交互 | 每次发布 |
| Fuzz 测试 | 边界条件、异常输入 | 持续 |
| 安全审计 | 代码审查 | 定期 |

### Fuzz 测试

```gn
# fuzztest/BUILD.gn
fuzztest("initlocalcertificate_fuzzer") {
  sources = [ "initlocalcertificate_fuzzer.cpp" ]
  external_deps = [
    "fsverity-utils:libfsverity_utils",
    ...
  ]
}

fuzztest("signlocalcode_fuzzer") {
  sources = [ "signlocalcode_fuzzer.cpp" ]
  external_deps = [
    "fsverity-utils:libfsverity_utils",
    ...
  ]
}
```

---

## 6.8 安全建议

### 对开发者的建议

#### 1. 参数验证

```cpp
// ✅ 总是验证输入参数
bool FsverityUtilsHelper::ComputeDigest(
    int fd,
    const libfsverity_merkle_tree_params &params,
    struct libfsverity_digest **digest)
{
    // 验证文件描述符
    if (fd < 0) {
        LOG_ERROR("Invalid file descriptor");
        return false;
    }

    // 验证参数版本
    if (params.version != 1) {
        LOG_ERROR("Unsupported version: %u", params.version);
        return false;
    }

    // 验证文件大小
    if (params.file_size == 0) {
        LOG_WARN("Empty file");
    }

    return libfsverity_compute_digest(...) == 0;
}
```

#### 2. 错误处理

```cpp
// ✅ 设置错误回调
libfsverity_set_error_callback([](const char *msg) {
    LOG_ERROR("fsverity error: %s", msg);
    // 可选：上报安全事件
    SecurityAudit::ReportSecurityEvent("fsverity_error", msg);
});
```

#### 3. 资源清理

```cpp
// ✅ 使用 RAII
class DigestGuard {
public:
    DigestGuard(struct libfsverity_digest *d) : digest(d) {}
    ~DigestGuard() { if (digest) free(digest); }

    struct libfsverity_digest *get() { return digest; }

private:
    struct libfsverity_digest *digest;
};

// 使用
DigestGuard digest(nullptr);
if (libfsverity_compute_digest(..., &digest.get()) != 0) {
    return false;
}
// 自动清理
```

### 对安全审计的建议

#### 检查清单

```
□ API 调用前参数验证
□ 错误回调设置
□ 内存正确释放
□ 线程安全保证
□ 密钥文件权限
□ 证书链验证
```

#### 关注点

| 关注点 | 说明 |
|--------|------|
| 输入验证 | 所有外部输入需要验证 |
| 错误处理 | 确保错误不会被忽略 |
| 资源管理 | 避免资源泄漏 |
| 加密操作 | 确保使用安全的加密原语 |

---

## 6.9 安全更新策略

### 更新流程

```
┌─────────────────────────────────────────────────────────────┐
│                   安全更新流程                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 漏洞发现                                                 │
│     ├── OpenSSL CVE 公告                                     │
│     ├── 上游 fsverity-utils 更新                             │
│     └── OH 安全团队审计                                      │
│                                                              │
│  2. 评估影响                                                 │
│     ├── 分析漏洞影响范围                                     │
│     ├── 评估是否影响 OH                                      │
│     └── 确定更新优先级                                       │
│                                                              │
│  3. 实施更新                                                 │
│     ├── 更新 OpenSSL 依赖                                    │
│     └── 更新 fsverity-utils（如需要）                         │
│                                                              │
│  4. 测试验证                                                 │
│     ├── 单元测试                                            │
│     ├── 集成测试                                            │
│     └── 安全回归测试                                         │
│                                                              │
│  5. 发布更新                                                 │
│     ├── 安全更新发布                                         │
│     └── 安全公告                                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### OpenSSL 更新

| 场景 | 操作 |
|------|------|
| OpenSSL 安全修复 | 通过 OH 安全更新机制 |
| OpenSSL 功能更新 | 评估兼容性后更新 |

### fsverity-utils 更新

| 场景 | 操作 |
|------|------|
| 上游安全修复 | 评估后合并 |
| 上游功能更新 | 定期同步 |
| OH 特定问题 | 创建 Patch |

---

## 6.10 总结

### 安全状态

| 状态 | 评估 |
|------|------|
| 已知漏洞 | 无 |
| 高风险组件 | 无 |
| 依赖风险 | OpenSSL |
| 综合风险 | 低 |

### 安全优势

```
✓ 代码量小，攻击面小
✓ 无复杂安全逻辑
✓ 依赖成熟的 OpenSSL
✓ 参数验证完善
✓ 内存安全使用
```

### 持续关注

```
1. 定期检查 OpenSSL 安全公告
2. 跟随上游安全更新
3. 保持 fuzz 测试覆盖
4. 定期安全审计
```

---

## 参考资源

### 安全标准

- [CWE - Common Weakness Enumeration](https://cwe.mitre.org/)
- [CVSS - Common Vulnerability Scoring System](https://www.first.org/cvss/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

### 相关文档

- [README.md](README.md) - 项目概述
- [01_Overview.md](01_Overview.md) - 原始库介绍
- [02_Patches.md](02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
- [上游安全公告](https://www.kernel.org/)
- [OpenSSL 安全公告](https://www.openssl.org/news/)

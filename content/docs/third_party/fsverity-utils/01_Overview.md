# fsverity-utils 原始库介绍

## 1.1 库概述

**fsverity-utils** 是 Linux kernel **fs-verity** 功能的官方用户空间工具集，提供完整的文件完整性验证解决方案。

### 核心特性

| 特性 | 描述 |
|------|------|
| Merkle 树计算 | 计算文件的分层哈希树结构 |
| 多哈希算法 | 支持 SHA-256 和 SHA-512 |
| 签名支持 | PKCS#7 格式的摘要签名 |
| 核集成 | 通过 ioctl 与 Linux 内核交互 |

### 项目信息

| 属性 | 值 |
|------|-----|
| **上游版本** | v1.6 |
| **许可证** | MIT License |
| **上游地址** | https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git |
| **维护者** | Eric Biggers (ebiggers@google.com) |
| **首次发布** | 2018 年 |

---

## 1.2 fs-verity 技术背景

### 什么是 fs-verity？

**fs-verity** 是 Linux 内核提供的一种文件系统级完整性验证机制，具有以下特点：

```
┌─────────────────────────────────────────────────────────┐
│                   fs-verity 工作原理                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  文件数据 ──► Merkle 树 ──► 根哈希 ──► 内核验证         │
│                  │                                    │
│                  ▼                                    │
│            隐藏的哈希树                                 │
│            (不存储在文件数据中)                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 核心概念

| 概念 | 说明 |
|------|------|
| **Merkle 树** | 分层哈希结构，每个节点是其子节点的哈希 |
| **根哈希** | Merkle 树的根节点，作为文件的唯一标识 |
| **块大小** | Merkle 树的叶子节点大小，默认 4096 字节 |
| **盐值** | 可选的随机值，增加哈希抗碰撞能力 |

### 与其他技术的比较

| 技术 | 级别 | 特点 |
|------|------|------|
| **fs-verity** | 文件级 | 透明验证，不修改文件内容 |
| **dm-verity** | 块设备级 | 整个设备校验，需单独分区 |
| **IMA/EVM** | 系统级 | 扩展属性校验，需 LSM 支持 |

---

## 1.3 库架构

### 目录结构

```
fsverity-utils/
├── include/
│   └── libfsverity.h          # 公共 C API
├── lib/
│   ├── compute_digest.c        # 摘要计算实现
│   ├── enable.c                # fs-verity 启用
│   ├── hash_algs.c            # 哈希算法
│   ├── sign_digest.c          # 签名功能
│   └── utils.c                # 工具函数
├── common/
│   ├── common_defs.h           # 公共定义
│   └── fsverity_uapi.h         # 内核 API 定义
└── programs/
    ├── fsverity.c              # 命令行工具
    └── ...
```

### 模块依赖

```
libfsverity.h
    │
    ├── lib/compute_digest.c
    │       └── hash_algs.c (OpenSSL)
    │
    ├── lib/enable.c
    │       └── fsverity_uapi.h (ioctl)
    │
    ├── lib/sign_digest.c
    │       └── hash_algs.c (OpenSSL)
    │
    └── lib/utils.c
```

---

## 1.4 核心 API

### 摘要计算

```c
/**
 * libfsverity_compute_digest() - 计算文件的 fs-verity 摘要
 * @fd: 文件描述符上下文
 * @read_fn: 文件数据读取回调
 * @params: Merkle 树参数
 * @digest_ret: 返回的摘要指针
 *
 * 返回: 0 成功，负数错误码
 */
int libfsverity_compute_digest(
    void *fd,
    libfsverity_read_fn_t read_fn,
    const struct libfsverity_merkle_tree_params *params,
    struct libfsverity_digest **digest_ret
);
```

### 签名

```c
/**
 * libfsverity_sign_digest() - 对摘要进行签名
 * @digest: 待签名的摘要
 * @sig_params: 签名参数（证书和密钥）
 * @sig_ret: 返回的签名指针
 * @sig_size_ret: 返回的签名大小
 *
 * 返回: 0 成功，负数错误码
 */
int libfsverity_sign_digest(
    const struct libfsverity_digest *digest,
    const struct libfsverity_signature_params *sig_params,
    uint8_t **sig_ret,
    size_t *sig_size_ret
);
```

### 启用 fs-verity

```c
/**
 * libfsverity_enable() - 在文件上启用 fs-verity
 * @fd: 只读文件描述符
 * @params: Merkle 树参数
 *
 * 返回: 0 成功，负数错误码
 */
int libfsverity_enable(
    int fd,
    const struct libfsverity_merkle_tree_params *params
);

/**
 * libfsverity_enable_with_sig() - 启用 fs-verity（带签名）
 */
int libfsverity_enable_with_sig(
    int fd,
    const struct libfsverity_merkle_tree_params *params,
    const uint8_t *sig,
    size_t sig_size
);
```

---

## 1.5 使用流程

### 基本使用流程

```c
// 1. 设置 Merkle 树参数
struct libfsverity_merkle_tree_params params = {
    .version = 1,
    .hash_algorithm = FS_VERITY_HASH_ALG_SHA256,
    .file_size = file_size,
    .block_size = 4096,
};

// 2. 计算文件摘要
struct libfsverity_digest *digest = NULL;
libfsverity_compute_digest(fd, read_fn, &params, &digest);

// 3. 可选：对摘要进行签名
uint8_t *signature = NULL;
size_t sig_size = 0;
libfsverity_sign_digest(digest, &sig_params, &signature, &sig_size);

// 4. 启用 fs-verity
libfsverity_enable_with_sig(fd, &params, signature, sig_size);
```

### 典型应用场景

| 场景 | 描述 |
|------|------|
| **只读文件保护** | 保护系统只读文件不被篡改 |
| **应用签名** | 验证应用完整性 |
| **OTA 更新** | 验证更新包的完整性 |
| **可信执行** | 在执行前验证代码完整性 |

---

## 1.6 命令行工具

### 常用命令

```bash
# 启用 fs-verity
fsverity enable <file>

# 测量文件摘要
fsverity measure <file>

# 对文件签名
fsverity sign <file> --key=key.pem --cert=cert.pem

# 转储元数据
fsverity dump metadata <file>

# 计算摘要
fsverity digest <file>
```

### 签名示例

```bash
# 1. 生成密钥和证书
openssl req -newkey rsa:4096 -nodes -keyout key.pem -x509 -out cert.pem

# 2. 签名文件
fsverity sign file.bin --key=key.pem --cert=cert.pem --output=file.sig

# 3. 启用 fs-verity 并附加签名
fsverity enable file.bin --signature=file.sig
```

---

## 1.7 哈希算法

### 支持的算法

| 算法 | ID | 摘要大小 | 块大小 |
|------|-----|----------|--------|
| SHA-256 | 1 | 32 字节 | 64 字节 |
| SHA-512 | 2 | 64 字节 | 128 字节 |

### API 查询

```c
// 按名称查找算法
uint32_t alg = libfsverity_find_hash_alg_by_name("sha256");

// 获取摘要大小
int size = libfsverity_get_digest_size(alg);

// 获取算法名称
const char *name = libfsverity_get_hash_name(alg);
```

---

## 1.8 与 OpenHarmony 的关系

### OH 中的定位

在 OpenHarmony 中，fsverity-utils 被 **code_signature** 模块使用，提供：

1. **文件摘要计算**：为本地代码签名提供文件 Merkle 树根哈希
2. **完整性验证**：支持系统只读文件的完整性保护
3. **签名支持**：为签名机制提供底层哈希和签名功能

### OH 适配策略

| 维度 | OH 适配 |
|------|---------|
| 源码修改 | 无（干净导入） |
| 构建适配 | BUILD.gn 配置 |
| 头文件 | 直接使用上游 |
| 依赖 | OpenSSL |

---

## 1.9 局限性

### 当前限制

| 限制 | 描述 |
|------|------|
| **只读文件** | fs-verity 仅适用于只读文件 |
| **文件系统支持** | 需底层文件系统支持 fs-verity |
| **性能开销** | 首次启用时有计算开销 |
| **签名管理** | 需自行管理证书和密钥 |

### 安全考虑

```
⚠️ 重要提示：

fs-verity 仅提供完整性验证，不提供：

❌ 访问控制
❌ 加密保护
❌ 密钥管理

要获得完整的保护，需要：
✓ 结合签名验证
✓ 密钥安全存储
✓ 策略强制执行
```

---

## 1.10 参考资源

### 官方文档

- [Linux fs-verity 文档](https://www.kernel.org/doc/html/latest/filesystems/fsverity.html)
- [上游项目源码](https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git)
- [内核源码文档](Documentation/filesystems/fsverity.rst)

### 相关项目

| 项目 | 说明 |
|------|------|
| IMA/EVM | Linux 完整性子系统 |
| dm-verity | 块设备验证 |
| trustedfs | 用户空间验证工具 |

---

## 1.11 总结

fsverity-utils 是 Linux fs-verity 功能的权威实现，提供：

✅ 完整的 Merkle 树计算
✅ 多哈希算法支持
✅ PKCS#7 签名集成
✅ 简洁的 C API
✅ 平台无关设计

在 OpenHarmony 中，该库被代码签名模块用于提供文件完整性验证功能，采用干净导入策略，无 OH 特定代码修改。

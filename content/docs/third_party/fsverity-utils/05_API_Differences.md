# fsverity-utils API 差异分析

## 5.1 差异概述

### 差异总结

| 维度 | 状态 | 说明 |
|------|------|------|
| 公共 API | 无差异 | 直接使用上游 API |
| 头文件 | 无差异 | 与上游完全一致 |
| 功能 | 无差异 | 功能完整移植 |
| 行为 | 无差异 | 行为与上游相同 |

### 结论

**fsverity-utils 在 OH 中完全遵循上游 API，无任何差异。**

这得益于：
1. 上游代码的平台无关设计
2. OH 适配仅在构建层
3. 无 OH 特定功能需求

---

## 5.2 公共 API 清单

### 核心 API（与上游完全一致）

| API | 说明 | 状态 |
|-----|------|------|
| `libfsverity_compute_digest()` | 计算文件摘要 | ✅ 无差异 |
| `libfsverity_sign_digest()` | 对摘要签名 | ✅ 无差异 |
| `libfsverity_enable()` | 启用 fs-verity | ✅ 无差异 |
| `libfsverity_enable_with_sig()` | 启用 fs-verity（带签名） | ✅ 无差异 |
| `libfsverity_find_hash_alg_by_name()` | 按名称查找哈希算法 | ✅ 无差异 |
| `libfsverity_get_digest_size()` | 获取摘要大小 | ✅ 无差异 |
| `libfsverity_get_hash_name()` | 获取算法名称 | ✅ 无差异 |
| `libfsverity_set_error_callback()` | 设置错误回调 | ✅ 无差异 |

### 数据结构（与上游完全一致）

| 结构体 | 说明 | 状态 |
|--------|------|------|
| `libfsverity_merkle_tree_params` | Merkle 树参数 | ✅ 无差异 |
| `libfsverity_digest` | 摘要结构 | ✅ 无差异 |
| `libfsverity_signature_params` | 签名参数 | ✅ 无差异 |
| `libfsverity_metadata_callbacks` | 元数据回调 | ✅ 无差异 |

### 常量（与上游完全一致）

| 常量 | 值 | 说明 |
|------|-----|------|
| `FS_VERITY_HASH_ALG_SHA256` | 1 | SHA-256 算法 |
| `FS_VERITY_HASH_ALG_SHA512` | 2 | SHA-512 算法 |
| `FSVERITY_UTILS_MAJOR_VERSION` | 1 | 主版本号 |
| `FSVERITY_UTILS_MINOR_VERSION` | 6 | 次版本号 |

---

## 5.3 API 详细对比

### libfsverity_compute_digest()

#### 函数签名（上游）

```c
int libfsverity_compute_digest(
    void *fd,
    libfsverity_read_fn_t read_fn,
    const struct libfsverity_merkle_tree_params *params,
    struct libfsverity_digest **digest_ret
);
```

#### 函数签名（OH）

**完全相同，无差异**

#### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `fd` | `void *` | 文件上下文，传递给 read_fn |
| `read_fn` | `libfsverity_read_fn_t` | 文件数据读取回调 |
| `params` | `const struct libfsverity_merkle_tree_params *` | Merkle 树参数 |
| `digest_ret` | `struct libfsverity_digest **` | 返回的摘要指针 |

#### 返回值

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| -EINVAL | 无效参数 |
| -ENOMEM | 内存分配失败 |
| 负数 | read_fn 返回的错误 |

### libfsverity_sign_digest()

#### 函数签名（上游）

```c
int libfsverity_sign_digest(
    const struct libfsverity_digest *digest,
    const struct libfsverity_signature_params *sig_params,
    uint8_t **sig_ret,
    size_t *sig_size_ret
);
```

#### 函数签名（OH）

**完全相同，无差异**

### libfsverity_enable()

#### 函数签名（上游）

```c
int libfsverity_enable(
    int fd,
    const struct libfsverity_merkle_tree_params *params
);
```

#### 函数签名（OH）

**完全相同，无差异**

---

## 5.4 数据结构对比

### libfsverity_merkle_tree_params

#### 结构体定义（上游）

```c
struct libfsverity_merkle_tree_params {
    uint32_t version;                     /* 必须为 1 */
    uint32_t hash_algorithm;              /* 哈希算法，0 为默认 SHA256 */
    uint64_t file_size;                  /* 文件大小 */
    uint32_t block_size;                  /* 块大小，0 为默认 4096 */
    uint32_t salt_size;                  /* 盐值大小 */
    const uint8_t *salt;                 /* 盐值 */
    uint64_t reserved1[8];               /* 必须为 0 */
    const struct libfsverity_metadata_callbacks *metadata_callbacks;
    uintptr_t reserved2[7];              /* 必须为 0 */
};
```

#### 结构体定义（OH）

**完全相同，无差异**

### libfsverity_digest

#### 结构体定义（上游）

```c
struct libfsverity_digest {
    uint16_t digest_algorithm;    /* 哈希算法 */
    uint16_t digest_size;         /* 摘要大小 */
    uint8_t digest[];             /* 摘要数据 */
};
```

#### 结构体定义（OH）

**完全相同，无差异**

---

## 5.5 宏定义对比

### 哈希算法定义

```c
/* 上游 */
#define FS_VERITY_HASH_ALG_SHA256       1
#define FS_VERITY_HASH_ALG_SHA512      2

/* OH - 完全相同 */
```

### 版本定义

```c
/* 上游 */
#define FSVERITY_UTILS_MAJOR_VERSION    1
#define FSVERITY_UTILS_MINOR_VERSION    6

/* OH - 完全相同 */
```

---

## 5.6 回调函数类型

### libfsverity_read_fn_t

```c
/* 上游 */
typedef int (*libfsverity_read_fn_t)(void *fd, void *buf, size_t count);

/* OH - 完全相同 */
```

#### 函数类型说明

| 参数 | 说明 |
|------|------|
| `fd` | 上下文指针（通常是文件描述符） |
| `buf` | 读取缓冲区 |
| `count` | 要读取的字节数 |

#### 返回值

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| 负数 | 错误码（errno 值） |

---

## 5.7 OH 特有的使用方式

虽然 API 无差异，但 OH 使用时有以下特点：

### 1. 封装层

```cpp
// OH 使用 C++ 封装
class FsverityUtilsHelper {
public:
    static FsverityUtilsHelper &GetInstance() {
        static FsverityUtilsHelper instance;
        return instance;
    }

    bool ComputeDigest(int fd,
                       const libfsverity_merkle_tree_params &params,
                       struct libfsverity_digest **digest) {
        return libfsverity_compute_digest(
            &fd,
            [](void *fd, void *buf, size_t count) -> int {
                return read(*(int *)fd, buf, count);
            },
            &params,
            digest
        ) == 0;
    }
};
```

### 2. 错误回调设置

```cpp
// 设置统一的错误回调
libfsverity_set_error_callback([](const char *msg) {
    LOG_ERROR("fsverity error: %s", msg);
});
```

### 3. 内存管理辅助

```cpp
// RAII 风格的摘要管理
class DigestHolder {
public:
    DigestHolder(struct libfsverity_digest *d) : digest(d) {}
    ~DigestHolder() { if (digest) free(digest); }

    struct libfsverity_digest *get() { return digest; }

private:
    struct libfsverity_digest *digest;
};
```

---

## 5.8 API 使用注意事项

### 1. 参数验证

```cpp
// ✅ 正确：验证输入参数
int ret = libfsverity_compute_digest(fd, read_fn, params, &digest);
if (ret != 0) {
    LOG_ERROR("Compute digest failed: %d", ret);
    return false;
}
```

### 2. 内存释放

```cpp
// ✅ 正确：释放分配的内存
struct libfsverity_digest *digest = nullptr;
int ret = libfsverity_compute_digest(..., &digest);
if (ret == 0 && digest) {
    // 使用 digest
    free(digest);  // 必须调用
}

// ❌ 错误：忘记释放
int ret = libfsverity_compute_digest(..., &digest);
// 直接返回，导致内存泄漏
```

### 3. 错误回调

```cpp
// ✅ 正确：设置错误回调
libfsverity_set_error_callback([](const char *msg) {
    LOG_ERROR("fsverity: %s", msg);
});

// ❌ 错误：不设置错误回调
// 错误信息无法获取
```

### 4. 线程安全

```cpp
// 注意：libfsverity API 本身不是线程安全的
// 如果多线程使用，需要外部同步

std::mutex fsverity_mutex;

bool ThreadSafeComputeDigest(...) {
    std::lock_guard<std::mutex> lock(fsverity_mutex);
    return libfsverity_compute_digest(...) == 0;
}
```

---

## 5.9 兼容性说明

### 上游版本兼容性

| OH 版本 | fsverity-utils 版本 | 兼容性 |
|---------|-------------------|--------|
| OH 4.0+ | v1.6 | 完全兼容 |
| OH 3.x | v1.5 | 需验证 |

### API 稳定性

```
┌─────────────────────────────────────────────────────────────┐
│                    API 稳定性承诺                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  公共 API（libfsverity.h）                                    │
│  ├── 稳定：计算摘要相关 API                                    │
│  ├── 稳定：签名相关 API                                       │
│  └── 稳定：启用相关 API                                       │
│                                                              │
│  内部 API（lib_private.h）                                    │
│  └── 不稳定：可能随时变化                                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 5.10 未来可能的变更

### 可能的 API 扩展

| 扩展项 | 描述 | 可能性 |
|--------|------|--------|
| 新哈希算法 | 如 SM3（国密） | 中 |
| 批量处理 | 批量计算摘要 | 低 |
| 异步 API | 异步计算接口 | 低 |

### 变更通知流程

1. **上游变更**：跟随上游公告
2. **OH 特有变更**：将在本文档中说明
3. **废弃通知**：提前两个版本通知

---

## 5.11 总结

### API 差异总结

| 维度 | 差异 | 影响 |
|------|------|------|
| 公共 API | 无 | 无需修改使用代码 |
| 头文件 | 无 | 无需额外包含路径（通过 public_configs） |
| 行为 | 无 | 与上游行为一致 |
| 文档 | 无 | 可参考上游文档 |

### 使用建议

```
1. 直接使用上游 API，无需特殊处理
2. 参考上游文档和示例
3. 注意错误处理和内存管理
4. 遵循线程安全规范
```

---

## 参考文档

- [README.md](README.md) - 项目概述
- [01_Overview.md](01_Overview.md) - 原始库介绍
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
- [上游 API 文档](https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git/tree/include/libfsverity.h)

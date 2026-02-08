# 对外 API

## 目的

本文档列出 HVB 组件的所有对外 C 接口，包括函数签名、参数说明、返回值和错误码，帮助 Bootloader/init 开发者集成 libhvb。

---

## 适用范围

- 目标读者：Bootloader 开发者、init 开发者、updater 开发者
- 知识储备：C 语言、Bootloader 工作原理

---

## 核心结论

1. **HVB 无 N-API/JS 绑定**，纯 C 库
2. **主接口**：`hvb_chain_verify()` - 链式校验 RVT 和镜像
3. **适配接口**：`struct hvb_ops` - 平台相关操作抽象
4. **同步调用**：所有接口为同步，无异步或回调
5. **错误处理**：通过 `enum hvb_errno` 返回错误码

---

## API 清单

### 1. 主 API（hvb.h）

| 函数名 | 文件位置 | 功能描述 | 同步/异步 |
|---------|----------|----------|-----------|
| `hvb_init_verified_data()` | `libhvb/include/hvb.h:91` | 初始化校验数据上下文 | 同步 |
| `hvb_chain_verify()` | `libhvb/include/hvb.h:92-94` | 链式校验 RVT 和镜像 | 同步 |
| `hvb_chain_verify_data_free()` | `libhvb/include/hvb.h:95` | 释放校验数据内存 | 同步 |

---

### 2. 平台适配接口（hvb_ops.h）

| 函数指针 | 文件位置 | 功能描述 | 必须实现 |
|---------|----------|----------|----------|
| `read_partition` | `libhvb/include/hvb_ops.h:40-41` | 读取分区数据 | ✅ 是 |
| `write_partition` | `libhvb/include/hvb_ops.h:42-43` | 写入分区数据 | ❌ 否 |
| `valid_rvt_key` | `libhvb/include/hvb_ops.h:44-46` | 验证公钥是否可信 | ✅ 是 |
| `read_rollback` | `libhvb/include/hvb_ops.h:47-49` | 读取防回滚索引 | ✅ 是 |
| `write_rollback` | `libhvb/include/hvb_ops.h:50-51` | 写入防回滚索引 | ✅ 是 |
| `read_lock_state` | `libhvb/include/hvb_ops.h:52` | 读取设备锁状态 | ❌ 否 |
| `get_partiton_size` | `libhvb/include/hvb_ops.h:53` | 获取分区大小 | ✅ 是 |

---

## 详细接口说明

### 1. hvb_init_verified_data

**原型**：
```c
struct hvb_verified_data *hvb_init_verified_data(void);
```

**文件位置**：`libhvb/src/auth/hvb.c:27-70`

**功能**：
- 分配校验数据上下文
- 初始化证书数组（最多 32 个）
- 初始化镜像数组（最多 32 个）
- 初始化启动参数缓冲区（4096 字节）
- 初始化防回滚索引数组（32 个位置）

**返回值**：
- 成功：返回 `struct hvb_verified_data*` 指针
- 失败：返回 `NULL`（内存不足）

**错误处理**：
- 调用者检查返回值是否为 NULL
- 若失败，无需清理（已内部清理）

---

### 2. hvb_chain_verify

**原型**：
```c
enum hvb_errno hvb_chain_verify(
    struct hvb_ops *ops,
    const char *rvt_ptn,
    const char *const *hash_ptn_list,
    struct hvb_verified_data **out_vd
);
```

**文件位置**：`libhvb/src/auth/hvb.c`（主函数，多个辅助函数）

**参数说明**：
| 参数 | 类型 | 说明 | 必填 |
|------|------|------|------|
| `ops` | `struct hvb_ops*` | 平台适配接口，包含分区读写、公钥验证等 | ✅ 是 |
| `rvt_ptn` | `const char*` | RVT 分区名称（如 "rvt"），可为 NULL | ❌ 否 |
| `hash_ptn_list` | `const char* const*` | 要校验的镜像分区列表（NULL 终止），最多 2 个（RVT + 镜像） | ✅ 是 |
| `out_vd` | `struct hvb_verified_data**` | 输出参数，返回校验数据上下文 | ✅ 是 |

**功能**：
1. 校验 RVT 公钥可信度（通过 `ops->valid_rvt_key()`）
2. 解析 RVT，获取各镜像的公钥
3. 逐个校验镜像：
   - 读取镜像 footer
   - 解析 verity 证书
   - 验证签名（RSA-PSS 或 SM2）
   - 校验哈希（SHA256 或 SM3）
   - 检查 rollback_index
4. 生成内核启动参数（通过 `hvb_creat_cmdline()`）

**返回值**：
- `HVB_OK`：校验成功
- `HVB_ERROR_OOM`：内存不足
- `HVB_ERROR_IO`：IO 错误
- `HVB_ERROR_VERIFY_SIGN`：签名验证失败
- `HVB_ERROR_VERIFY_HASH`：哈希验证失败
- `HVB_ERROR_ROLLBACK_INDEX`：防回滚索引错误
- `HVB_ERROR_PUBLIC_KEY_REJECTED`：公钥被拒绝
- `HVB_ERROR_INVALID_CERT_FORMAT`：证书格式无效
- `HVB_ERROR_INVALID_FOOTER_FORMAT`：Footer 格式无效
- `HVB_ERROR_UNSUPPORTED_VERSION`：不支持的版本
- `HVB_ERROR_INVALID_ARGUMENT`：参数无效

**输出**：
- `out_vd`：返回校验数据上下文，包含：
  - 已加载证书列表（`certs`）
  - 已加载镜像列表（`images`）
  - 启动参数字符串（`cmdline.buf`）
  - 防回滚索引数组（`rollback_indexes`）
  - 哈希算法类型（`algorithm`）
  - 是否匹配备用公钥（`match_backup_pubkey`）

**错误码定义**：`libhvb/include/hvb.h:41-53`

**注意**：
- 调用者需实现 `hvb_ops` 接口
- 校验失败时，`out_vd` 仍会被赋值，但内容可能不完整

---

### 3. hvb_chain_verify_data_free

**原型**：
```c
void hvb_chain_verify_data_free(struct hvb_verified_data *vd);
```

**文件位置**：`libhvb/src/auth/hvb.c`（声明在 hvb.h:95，实现位置待确认）

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `vd` | `struct hvb_verified_data*` | 校验数据上下文，可为 NULL |

**功能**：
- 释放证书数组内存（`vd->certs`）
- 释放镜像数组内存（`vd->images`）
- 释放启动参数缓冲区（`vd->cmdline.buf`）
- 释放 `vd` 本身

**返回值**：无（void）

**注意**：
- 即使校验失败，也要调用此函数清理内存
- 重复调用安全（检查 NULL）

---

## 平台适配接口（hvb_ops）

### read_partition

**原型**：
```c
enum hvb_io_errno (*read_partition)(
    struct hvb_ops *ops,
    const char *ptn,
    int64_t offset,
    uint64_t num_bytes,
    void *buffer,
    uint64_t *out_num_read
);
```

**功能**：从指定分区读取数据

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `ops` | `struct hvb_ops*` | ops 结构本身（包含 user_data） |
| `ptn` | `const char*` | 分区名称（如 "boot"、"system"） |
| `offset` | `int64_t` | 读取偏移量（字节） |
| `num_bytes` | `uint64_t` | 期望读取字节数 |
| `buffer` | `void*` | 输出缓冲区，大小至少 `num_bytes` |
| `out_num_read` | `uint64_t*` | 实际读取字节数 |

**返回值**：
- `HVB_IO_OK`：成功
- `HVB_IO_ERROR_OOM`：内存不足
- `HVB_IO_ERROR_IO`：IO 错误
- `HVB_IO_ERROR_NO_SUCH_PARTITION`：分区不存在
- `HVB_IO_ERROR_RANGE_OUTSIDE_PARTITION`：偏移超出分区范围
- `HVB_IO_ERROR_NO_SUCH_VALUE`：无此值
- `HVB_IO_ERROR_INVALID_VALUE_SIZE`：值大小无效
- `HVB_IO_ERROR_INSUFFICIENT_SPACE`：空间不足

**错误码定义**：`libhvb/include/hvb_ops.h:27-36`

**注意**：
- 调用者需确保 `buffer` 有足够空间
- `out_num_read` 必须被赋值（即使失败）

---

### valid_rvt_key

**原型**：
```c
enum hvb_io_errno (*valid_rvt_key)(
    struct hvb_ops *ops,
    const uint8_t *pubkey,
    uint64_t pubkey_length,
    const uint8_t *pubkey_metadata,
    uint64_t pubkey_metadata_length,
    bool *out_is_trusted
);
```

**功能**：验证公钥是否可信（与 Bootloader 可信根对比）

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `pubkey` | `const uint8_t*` | 公钥数据 |
| `pubkey_length` | `uint64_t` | 公钥长度 |
| `pubkey_metadata` | `const uint8_t*` | 公钥元数据（当前未使用） |
| `pubkey_metadata_length` | `uint64_t` | 元数据长度 |
| `out_is_trusted` | `bool*` | 输出参数，`true`=可信，`false`=不可信 |

**返回值**：
- `HVB_IO_OK`：成功
- 其他：IO 错误

**注意**：
- 实现逻辑由 OEM 厂商自定义
- 信任根存储在 TPM、EFUSE 等安全区域

---

### read_rollback / write_rollback

**原型**：
```c
enum hvb_io_errno (*read_rollback)(
    struct hvb_ops *ops,
    uint64_t rollback_index_location,
    uint64_t *out_rollback_index
);

enum hvb_io_errno (*write_rollback)(
    struct hvb_ops *ops,
    uint64_t rollback_index_location,
    uint64_t rollback_index
);
```

**功能**：读取/写入防回滚索引

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `rollback_index_location` | `uint64_t` | 索引位置（0-31） |
| `out_rollback_index` | `uint64_t*` | 读取时输出当前索引 |
| `rollback_index` | `uint64_t` | 写入时设置新索引 |

**返回值**：
- `HVB_IO_OK`：成功
- 其他：IO 错误

**注意**：
- 存储介质由厂商实现（TPM、EFUSE、安全存储区）
- 校验成功后，必须更新索引

---

### read_lock_state

**原型**：
```c
enum hvb_io_errno (*read_lock_state)(
    struct hvb_ops *ops,
    bool *lock_state
);
```

**功能**：读取设备锁状态

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `lock_state` | `bool*` | 输出参数，`true`=锁定，`false`=解锁 |

**返回值**：
- `HVB_IO_OK`：成功
- 其他：IO 错误

**注意**：
- 可选实现（某些设备可能不支持）
- 锁状态影响校验逻辑（解锁设备可跳过校验）

---

### get_partiton_size

**原型**：
```c
enum hvb_io_errno (*get_partiton_size)(
    struct hvb_ops *ops,
    const char *ptn,
    uint64_t *out_bytes
);
```

**功能**：获取分区大小

**参数说明**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `ptn` | `const char*` | 分区名称 |
| `out_bytes` | `uint64_t*` | 输出参数，分区大小（字节） |

**返回值**：
- `HVB_IO_OK`：成功
- 其他：IO 错误

**注意**：
- 用于边界检查，防止越界读取

---

## 参数校验规则

### 输入参数校验

| 参数 | 校验规则 | 错误码 |
|------|----------|----------|
| `ops` | 不能为 NULL | `HVB_ERROR_INVALID_ARGUMENT` |
| `ptn` | 长度 < 36 字符 | `HVB_ERROR_INVALID_ARGUMENT` |
| `offset` | >= 0 | `HVB_ERROR_INVALID_ARGUMENT` |
| `num_bytes` | > 0 | `HVB_ERROR_INVALID_ARGUMENT` |
| `buffer` | 不能为 NULL（当 num_bytes > 0） | `HVB_ERROR_INVALID_ARGUMENT` |

**证据**：`libhvb/src/auth/hvb.c:108-112`（`hvb_get_partition_image` 函数）

### 分区大小限制

- **最小**：4 KiB（`HVB_MIN_PARTITION_SIZE`）
- **最大**：64 GiB（`HVB_MAX_PARTITION_SIZE`）

**证据**：`libhvb/include/hvb.h:32-33`

### 数量限制

- **最大镜像数**：32（`HVB_MAX_NUMBER_OF_LOADED_IMAGES`）
- **最大证书数**：32（`HVB_MAX_NUMBER_OF_LOADED_CERTS`）
- **最大回滚索引位置**：32（`HVB_MAX_NUMBER_OF_ROLLBACK_INDEX_LOCATIONS`）

**证据**：`libhvb/include/hvb.h:26-29`

---

## 调用示例

### Bootloader 集成示例

```c
#include "hvb.h"
#include "hvb_ops.h"

// 1. 实现 hvb_ops
enum hvb_io_errno my_read_partition(...) { /* 实现读取逻辑 */ }
enum hvb_io_errno my_valid_rvt_key(...) { /* 实现公钥验证 */ }
// ... 其他 ops

struct hvb_ops ops = {
    .user_data = NULL,
    .read_partition = my_read_partition,
    .valid_rvt_key = my_valid_rvt_key,
    // ... 其他函数指针
};

// 2. 调用链式校验
const char *hash_ptn_list[] = {"rvt", "boot", NULL};
struct hvb_verified_data *vd = NULL;
enum hvb_errno ret = hvb_chain_verify(&ops, "rvt", hash_ptn_list, &vd);

if (ret != HVB_OK) {
    // 处理校验失败
    return;
}

// 3. 获取启动参数
char *cmdline = vd->cmdline.buf;
// 传递给内核...

// 4. 清理
hvb_chain_verify_data_free(vd);
```

---

## 相关跳转

- [项目定位与边界](01_Project_Scope.md) - HVB 能力范围
- [架构说明](03_Architecture.md) - 数据流与调用时序
- [内部 API](05_Internal_API.md) - 模块接口详情

---

*最后更新: 2026-02-06*

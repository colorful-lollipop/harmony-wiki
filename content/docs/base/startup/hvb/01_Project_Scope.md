# 项目定位与边界

## 目的

本文档明确 HVB 组件的定位、核心能力、边界和运行环境，帮助读者理解"什么在 HVB 范围内"、"什么不在"。

---

## 适用范围

- 目标读者：Bootloader 开发者、init 开发者、安全审计人员、构建工程师
- 适配系统：OpenHarmony 标准系统
- 知识储备：C 语言、加密算法基础、Bootloader 原理

---

## 核心结论

1. **HVB 只负责镜像校验**，不负责镜像加载、文件系统挂载等
2. **信任边界**：Bootloader 可信根 → libhvb → 镜像数据
3. **不涉及权限管理**，权限检查在 Bootloader 层面
4. **不支持动态加载**，所有代码静态链接到调用方
5. **同步调用模式**，无异步接口或回调

---

## 核心能力

### ✅ HVB 提供的功能

| 功能 | 说明 | 集成点 |
|------|------|----------|
| **镜像签名生成** | 编译时为镜像添加 verity footer（签名、哈希、公钥） | 构建系统（hvbtool.py） |
| **镜像校验** | 运行时校验镜像签名和哈希 | Bootloader、init |
| **信任链建立** | 从可信根到镜像数据的完整信任链 | Bootloader |
| **防回滚** | 通过 rollback_index 防止版本回滚 | Bootloader |
| **RVT 管理** | 统一管理多个镜像的公钥 | Bootloader |
| **启动参数生成** | 生成内核启动参数（传递 dm-verity 配置） | init |
| **多种加密算法** | 支持 RSA（2048/3072/4096）和 SM2（国密） | 所有场景 |

**证据**：
- 镜像签名：`tools/hvbtool.py`
- 镜像校验：`libhvb/src/auth/hvb.c:92-95`（`hvb_chain_verify`）
- RVT 管理：`libhvb/src/rvt/hvb_rvt.c`
- 启动参数：`libhvb/src/cmdline/hvb_cmdline.c:35`（`hvb_creat_cmdline`）

---

### ❌ HVB 不提供的功能

| 功能 | 说明 | 负责方 |
|------|------|----------|
| **镜像加载** | 从存储介质读取镜像数据到内存 | Bootloader |
| **分区管理** | 分区表读写、分区挂载 | Bootloader、内核 |
| **文件系统操作** | 文件系统挂载、文件读写 | 内核、init |
| **权限管理** | UID/GID 权限检查、访问控制 | 内核、沙箱 |
| **设备解锁** | 解锁设备的安全逻辑 | Bootloader（厂商实现） |
| **OTA 升级** | OTA 包下载、分区写入 | updater |
| **签名密钥管理** | 密钥生成、存储、轮换 | OEM 厂商 |
| **用户认证** | 密码、指纹、人脸等认证 | 用户空间应用 |

---

## 边界与接口

### 与 Bootloader 的边界

**HVB 提供**：
- 校验接口：`hvb_chain_verify()` - 链式校验 RVT 和多个镜像
- 依赖抽象：`struct hvb_ops` - 分区读写、公钥验证、回滚索引读写、锁状态查询

**Bootloader 提供**：
- 可信根（公钥存储与验证）
- 分区读写实现
- 安全存储（rollback_index）
- 设备锁状态

**接口定义**：`libhvb/include/hvb_ops.h:38-54`

```c
struct hvb_ops {
    void *user_data;
    enum hvb_io_errno (*read_partition)(...);
    enum hvb_io_errno (*write_partition)(...);
    enum hvb_io_errno (*valid_rvt_key)(...);  // 验证公钥是否可信
    enum hvb_io_errno (*read_rollback)(...);
    enum hvb_io_errno (*write_rollback)(...);
    enum hvb_io_errno (*read_lock_state)(...);
    enum hvb_io_errno (*get_partiton_size)(...);
};
```

---

### 与 init 的边界

**HVB 提供**：
- 校验接口：`hvb_chain_verify()` - 校验用户态镜像
- 启动参数生成：`hvb_creat_cmdline()` - 生成 dm-verity 参数

**init 提供**：
- 分区读写实现
- 内核启动参数传递
- 文件系统挂载（基于 fstab 配置）

**接口定义**：`libhvb/include/hvb_cmdline.h:26-36`

```c
#define CMD_LINE_SIZE (4096UL)

#define HVB_CMDLINE_VB_STATE       "ohos.boot.hvb.enable"
#define HVB_CMDLINE_HASH_ALG       "ohos.boot.hvb.hash_algo"
#define HVB_CMDLINE_CERT_DIGEST    "ohos.boot.hvb.digest"
#define HVB_CMDLINE_VERSION        "ohos.boot.hvb.version"

enum hvb_errno hvb_creat_cmdline(struct hvb_ops *ops, struct hvb_verified_data *vd);
```

---

### 与 updater 的边界

**HVB 提供**：
- 静态库：`libhvb_static_real.a` - 使用 libsec_static（完整边界检查）
- 校验接口：`hvb_chain_verify()` - OTA 升级时校验新镜像

**updater 提供**：
- 分区读写实现
- OTA 包解析
- 分区写入与重启

**接口**：`libhvb/BUILD.gn:57-71`（`libhvb_static_real` target）

---

## 关键概念

### 1. Verity（完整性校验）

**定义**：通过密码学哈希确保数据完整性的机制

**实现**：
- **Hash 模式**：对整个镜像计算哈希
- **Hashtree 模式**：对镜像构造哈希树，支持按需校验

**证据**：`libhvb/include/hvb_cert.h:49-54`

---

### 2. 签名（Signature）

**定义**：用私钥对哈希值进行加密，确保数据来源可信

**实现**：
- RSA-PSS 填充（2048/3072/4096 位）
- SM2 国密签名（256 位）

**验证**：用公钥解密签名，对比哈希值

**证据**：`libhvb/include/hvb_crypto.h:68-79`（`hvb_rsa_verify_pss`）

---

### 3. RVT（Root Verity Table）

**定义**：统一存储多个镜像公钥和校验信息的分区

**作用**：Bootloader 只需验证 RVT，即可信任所有镜像的公钥

**支持备用公钥**：每个镜像可有 1-2 个公钥（pubkey_num_per_ptn）

**证据**：`libhvb/include/hvb_rvt.h:40-55`

---

### 4. Rollback Index（防回滚索引）

**定义**：记录镜像版本号的安全存储值

**作用**：
- 防止设备回滚到有漏洞的历史版本
- 每次启动成功后更新索引

**存储**：由 Bootloader 实现的安全存储（如 TPM、EFUSE）

**接口**：`libhvb/include/hvb_ops.h:47-51`

---

### 5. 锁/解锁状态

**定义**：设备的安全状态，影响校验逻辑

**状态**：
- **Green（锁 + 校验成功）**：需要使能 HVB
- **Yellow/Red（锁 + 校验失败）**：不需要使能 HVB
- **Orange（解锁）**：不需要使能 HVB

**实现**：由 Bootloader 厂商自定义实现

**证据**：`README_zh.md:94-100`

---

## 运行环境

### 适配系统类型

| 系统类型 | 支持情况 | 说明 |
|----------|----------|------|
| Standard（标准系统） | ✅ 支持 | 完整功能支持 |
| Small（轻量系统） | ❌ 不支持 | 未测试 |
| Mini（小型系统） | ❌ 不支持 | 未测试 |

**证据**：`bundle.json:15-17`

---

### 依赖组件

| 组件 | 依赖类型 | 用途 |
|------|----------|------|
| **bounds_checking_function** | 外部依赖 | 内存边界检查（libsec_shared/libsec_static） |
| **OpenSSL** | 构建依赖 | hvbtool.py 的加密算法 |
| **dm-verity** | 内核依赖 | 按需校验（init 集成） |

**证据**：
- bounds_checking_function: `bundle.json:20-22`
- OpenSSL: `tools/hvbtool.py:169-176`（`get_sm3_digest`）
- dm-verity: `README_zh.md:39`

---

### 资源占用

| 资源 | 占用 | 说明 |
|------|------|------|
| **ROM** | 3072 KB | 静态库代码大小 |
| **RAM** | 3072 KB | 运行时内存占用 |

**证据**：`bundle.json:18-19`

---

## 约束与限制

### 功能限制

1. **仅支持标准系统**：未适配小型和轻量系统
2. **无动态加载**：代码静态链接，不支持插件
3. **同步调用**：所有接口为同步调用，无异步接口
4. **无权限管理**：不负责权限检查（由 Bootloader/内核负责）
5. **无配置文件**：所有配置通过编译时参数或函数参数传递

### 技术限制

1. **分区大小限制**：4 KiB - 64 GiB（`libhvb/include/hvb.h:32-33`）
2. **镜像数量限制**：最多 32 个（`HVB_MAX_NUMBER_OF_LOADED_IMAGES`）
3. **证书数量限制**：最多 32 个（`HVB_MAX_NUMBER_OF_LOADED_CERTS`）
4. **RVT 大小限制**：最多 64 KiB（`libhvb/include/hvb_rvt.h:37-38`）

**证据**：
- 分区大小：`libhvb/include/hvb.h:32-33`
- 镜像数量：`libhvb/include/hvb.h:28`
- 证书数量：`libhvb/include/hvb.h:27`
- RVT 大小：`libhvb/include/hvb_rvt.h:37-38`

---

## 相关跳转

- [项目概览](00_Overview.md) - HVB 组件介绍
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [对外 API](04_Public_API.md) - 如何使用 HVB 接口
- [安全分析](08_Security_Analysis.md) - 信任边界与攻击面

---

*最后更新: 2026-02-06*

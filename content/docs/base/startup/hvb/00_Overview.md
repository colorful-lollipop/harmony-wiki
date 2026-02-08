# HVB 项目概览

## 目的

本文档介绍 OpenHarmony Verified Boot (HVB) 组件的基本概念、核心功能和设计目标，帮助读者快速理解组件定位。

---

## 适用范围

- 面向对象：OpenHarmony 设备开发者、Bootloader 开发者、系统安全工程师
- 适用场景：OpenHarmony 标准系统（standard）的安全启动实现
- 文档版本：基于 2026-02-06 代码库版本

---

## 核心结论

1. **HVB 是纯 C 实现的安全启动校验库**，无 JS 绑定，不涉及 N-API
2. **两种校验模式**：整包校验（hash）和按需校验（hashtree + dm-verity）
3. **信任链**：Bootloader 可信根 → RVT 公钥 → 镜像签名 → 镜像哈希
4. **防回滚机制**：通过 rollback_index 防止回滚到有漏洞的版本
5. **支持国密算法**：SM2 签名、SM3 哈希

---

## HVB 是什么？

Open**H**armony **V**erified **B**oot (HVB) 是 OpenHarmony 系统的安全启动组件，用于校验和认证系统镜像，确保：

- ✅ 镜像**来源合法**（通过签名验证）
- ✅ 镜像**未被篡改**（通过哈希校验）
- ✅ 无法**回滚到有漏洞的历史版本**（通过防回滚机制）

---

## 核心模块

### 1. libhvb（运行时校验库）

**路径**: `libhvb/`

**集成方**：
- **Bootloader**：在设备启动初期校验初始镜像（如 boot.img、recovery.img）
- **init**：在系统启动时使能 dm-verity，校验文件系统镜像（如 system.img）

**输出**：
- `libhvb_static.a` - 静态库（Bootloader/init 使用）
- `libhvb_static_real.a` - 静态库（Updater 使用）

**证据**：`libhvb/BUILD.gn:48-71`

---

### 2. hvbtool（镜像签名工具）

**路径**: `tools/hvbtool.py`

**集成方**：
- **构建系统**：在编译时对系统镜像进行签名

**功能**：
- 为镜像生成 verity footer（包含签名、哈希、公钥）
- 制作 RVT（Root Verity Table）镜像
- 支持解析和擦除 HVB 格式的镜像

**支持的签名算法**：
- SHA256_RSA2048
- SHA256_RSA4096
- SM2（国密）

**证据**：`tools/readme.md`（中文说明）

---

## 校验模式

### 1. 整包校验（Hash）

**适用场景**：一次性加载的小型镜像（如 boot.img、recovery.img）

**原理**：
1. 编译时：计算整个镜像的 SHA256 哈希值
2. 签名：对哈希值进行 RSA/SM2 签名
3. 启动时：重新计算镜像哈希，与签名中的哈希对比

**优点**：实现简单
**缺点**：镜像过大时影响开机时间

**证据**：`libhvb/include/hvb_cert.h:49-54`（`HVB_IMAGE_TYPE_HASH`）

---

### 2. 按需校验（Hashtree）

**适用场景**：大型文件系统镜像（如 system.img、vendor.img）

**原理**：
1. 编译时：构造哈希树（hashtree），每个数据块都有对应的哈希值
2. 签名：对哈希树的根哈希进行签名
3. 启动时：通过 dm-verity 驱动按需校验数据块
4. 运行时：内核在访问存储块时实时校验

**优点**：
- 不需要在开机时校验整个镜像
- 只需校验哈希树根哈希，启动速度快

**依赖**：内核需使能 [dm-verity](https://www.kernel.org/doc/html/latest/admin-guide/device-mapper/verity.html) 特性

**证据**：`libhvb/include/hvb_cert.h:51-52`（`HVB_IMAGE_TYPE_HASHTREE`）

---

## Verity Footer 结构

每个经过 HVB 签名的镜像末尾都会追加一个 **verity footer**，包含：

| 字段 | 说明 |
|------|------|
| **signature_info** | 签名内容（签名算法、签名数据、公钥、user_id 等） |
| **hash_info** | 完整性信息（salt、digest、hashtree 数据、FEC 纠错码） |

**证据**：`README_zh.md:43-52`

---

## Root Verity Table (RVT)

对于有多个镜像的系统，可增加 **RVT 分区**统一存放各个镜像的公钥：

**作用**：
- Bootloader 只需校验 RVT 的合法性
- 通过 RVT 的公钥列表逐个校验各个镜像的 verity 信息

**结构**：
- Magic: `"rot"`
- verity_num: 镜像数量
- pubkey_num_per_ptn: 每个镜像的公钥数量（1 或 2，支持备用公钥）
- 公钥描述符数组：每个镜像的公钥信息

**证据**：`libhvb/include/hvb_rvt.h:26-69`

---

## 信任链建立

HVB 建立完整的信任链：

```
可信根（Bootloader 可信运行环境）
    ↓ 校验 RVT 签名
RVT 公钥列表
    ↓ 校验镜像签名
镜像公钥
    ↓ 校验镜像哈希
镜像哈希（digest）
    ↓ 对比实际镜像哈希
实际镜像数据
```

**关键点**：
1. **可信根**由 OEM 厂商在 Bootloader 初始启动时通过安全运行环境提供
2. **rollback_index** 存储在安全存储区，防止回滚
3. **锁/解锁状态**影响校验逻辑（解锁设备可跳过校验）

**证据**：`README_zh.md:91-106`

---

## 加密算法支持

### 标准算法
- **SHA256**：哈希算法（32 字节）
- **RSA2048/RSA3072/RSA4096**：签名算法（2048/3072/4096 位）
- **RSA-PSS**：签名填充模式（盐长度 32 字节）

### 国密算法
- **SM2**：椭圆曲线签名算法（256 位）
- **SM3**：哈希算法（32 字节）

**证据**：`libhvb/include/hvb_crypto.h`、`libhvb/include/hvb_sm2.h`

---

## 运行环境与约束

### 适配系统类型
- **标准系统**（standard）✅
- 小型系统（lite）❌
- 轻量系统（small）❌

**证据**：`bundle.json:15-17`

### 依赖组件
- **bounds_checking_function**：边界检查库（libsec_shared/libsec_static）
- **OpenSSL**：加密算法支持（hvbtool.py 使用）
- **dm-verity**：内核模块（init 集成）

**证据**：`bundle.json:20-22`、`libhvb/BUILD.gn:50`

### 资源占用
- **ROM**: 3072 KB
- **RAM**: 3072 KB

**证据**：`bundle.json:18-19`

---

## 相关跳转

- [项目定位与边界](01_Project_Scope.md) - 详细的能力范围与限制
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [架构说明](03_Architecture.md) - 组件图与数据流
- [对外 API](04_Public_API.md) - 如何使用 libhvb

---

*最后更新: 2026-02-06*

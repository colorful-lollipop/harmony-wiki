# 03 - 内部 API 与 HAL 接口

## 目的与适用范围

**目的**: 详细说明 `sys_installer_lite` 的内部模块划分、HAL 接口定义和芯片适配指南。

**适用范围**: 需要进行芯片平台适配的平台开发者、深度理解系统架构的工程师。

---

## 模块职责划分

### 模块关系图

```
┌─────────────────────────────────────────────────────────────┐
│                    sys_installer_lite                       │
├─────────────────────────────────────────────────────────────┤
│  Module          │  File              │  Responsibility      │
├──────────────────┼────────────────────┼──────────────────────┤
│  Updater Core    │ hota_updater.c     │ 状态机、包解析、流程   │
│  Verify          │ hota_verify.c      │ 签名/哈希验证入口      │
│  RSA Wrapper     │ app_rsa.c          │ RSA-PKCS1 签名验证     │
│  SHA256 Wrapper  │ app_sha256.c       │ SHA256 哈希计算        │
│  HAL Interface   │ hal_hota_board.h   │ 硬件抽象接口定义       │
└──────────────────┴────────────────────┴──────────────────────┘
```

### 各模块详细职责

#### 1. Updater Core (hota_updater.c)

**位置**: `frameworks/source/updater/hota_updater.c` (704 行)

**核心职责**:
- **状态机管理**: 维护 OTA 生命周期状态（`HOTA_NOT_INIT` → `HOTA_INITED` → ...）
- **包解析**: 解析 PkgBasicInfo、Component Table
- **数据流控制**: 分包接收、缓冲区管理、组件调度
- **版本校验**: 防回滚检查（`CheckPkgVersionValid`）
- **回调通知**: 错误和状态变更通知

**关键数据结构**:
```c
// 全局状态 (hota_updater.c:90-103)
static HotaStatus g_otaStatus;                    // 当前状态
static CurrentDloadComp g_currentDloadComp;       // 当前下载组件
static ComponentInfos g_componentInfos;           // 组件信息表
static unsigned char *g_infoCompBuff;             // Info组件缓冲区
```

**内部函数清单**:

| 函数名 | 行号 | 功能 |
|--------|------|------|
| `HotaResetStatus` | 105-117 | 重置全局状态 |
| `UpdateStatus` | 120-142 | 更新状态并通知回调 |
| `ReportErrorCode` | 144-149 | 报告错误码 |
| `IsDigest` | 151-161 | 检查字符串是否为纯数字 |
| `IsLatestVersion` | 163-208 | 版本比较（防回滚） |
| `CheckPkgVersionValid` | 210-225 | 验证包版本有效性 |
| `ParseHotaInfoComponent` | 227-283 | 解析并验证 Info Component |
| `InitDloadNextComp` | 285-308 | 初始化下一个组件下载 |
| `ParseHotaComponent` | 310-331 | 验证组件哈希 |
| `HotaIsRejected` | 333-336 | 检查是否被拒绝状态 |
| `ProcessInfoCompHeader` | 338-386 | 处理 Info Component 头部 |
| `DloadIsDone` | 388-397 | 检查是否全部完成 |
| `ProcessOneComponent` | 399-419 | 处理一个组件完成 |
| `CopyToDloadCompBuffer` | 421-437 | 拷贝到缓冲区 |
| `GetCurrentDloadCompPartition` | 439-457 | 获取当前组件对应分区 |
| `StashRecvDataToBuffer` | 459-484 | 存储接收到的数据 |
| `ProcessCompData` | 486-507 | 处理组件数据 |
| `HotaDefaultWrite` | 509-558 | 默认包格式写入处理 |

#### 2. Verify Module (hota_verify.c)

**位置**: `frameworks/source/verify/hota_verify.c` (159 行)

**核心职责**:
- **哈希管理**: 全局 SHA256 上下文管理（`g_sha256`）
- **签名验证入口**: `HotaSignVerify` 协调哈希计算和 RSA 验证
- **公钥获取**: 通过 HAL 获取公钥

**关键数据结构**:
```c
static uint8 g_hash[HASH_LENGTH] = {0};            // 哈希结果
static AppSha256Context g_sha256 = { 0 };          // SHA256 上下文
```

**导出函数**:

| 函数名 | 行号 | 功能 |
|--------|------|------|
| `HotaHashInit` | 28-37 | 初始化 SHA256 上下文 |
| `HotaHashCalc` | 39-48 | 更新 SHA256 计算 |
| `HotaGetHash` | 50-72 | 获取最终哈希值 |
| `HotaSignVerify` | 132-153 | 验证签名（入口函数） |
| `HotaGetPubKey` | 155-159 | 获取公钥（调用 HAL） |

**内部函数**:

| 函数名 | 行号 | 功能 |
|--------|------|------|
| `HotaCalcImageHash` | 74-101 | 计算数据块的 SHA256 |
| `HotaSignVerifyByHash` | 103-130 | 使用 RSA 验证哈希签名 |

#### 3. RSA Wrapper (app_rsa.c)

**位置**: `frameworks/source/verify/app_rsa.c` (80 行)

**核心职责**:
- **mbedtls 封装**: 封装 mbedtls 库提供 RSA 功能
- **公钥解析**: 解析 DER 格式公钥
- **PKCS1-v2.1 验证**: 使用 RSA-PSS-SHA256 验证签名

**关键数据结构**:
```c
typedef struct {
    mbedtls_pk_context context;
} AppRsaContext;
```

**导出函数**:

| 函数名 | 行号 | 功能 |
|--------|------|------|
| `AppRsaInit` | 22-29 | 初始化 RSA 上下文 |
| `AppRsaDecodePublicKey` | 31-43 | 解析 DER 格式公钥 |
| `AppVerifyData` | 45-69 | 验证签名数据 |
| `AppRsaFree` | 71-79 | 释放 RSA 上下文 |

**实现细节** (app_rsa.c:60):
```c
// 使用 RSA-PKCS1-v2.1 (PSS) + SHA256
mbedtls_rsa_set_padding(mbedtls_pk_rsa(rsa->context), 
                        MBEDTLS_RSA_PKCS_V21, 
                        MBEDTLS_MD_SHA256);
ret = mbedtls_rsa_pkcs1_verify(mbedtls_pk_rsa(rsa->context), 
                               MBEDTLS_MD_SHA256, 
                               plainBufLen, 
                               plainBuf, 
                               cipherBuf);
```

#### 4. SHA256 Wrapper (app_sha256.c)

**位置**: `frameworks/source/verify/app_sha256.c` (49 行)

**核心职责**:
- **mbedtls 封装**: 封装 mbedtls 库提供 SHA256 功能

**导出函数**:

| 函数名 | 行号 | 功能 |
|--------|------|------|
| `AppSha256Init` | 19-28 | 初始化 SHA256 上下文 |
| `AppSha256Update` | 30-38 | 更新哈希计算 |
| `AppSha256Finish` | 40-48 | 完成计算并输出结果 |

---

## HAL 接口定义

### HAL 概述

**位置**: `hals/hal_hota_board.h` (269 行)

HAL（Hardware Abstraction Layer）是框架层与硬件/芯片实现之间的抽象接口。芯片厂商需要实现这些接口以适配 `sys_installer_lite`。

**实现要求**:
- 接口由厂商在 `vendor/{vendor}/{board}/hals/update/` 或类似路径实现
- 编译时链接为 `libhal_update.so`（动态库）或静态库
- 实现必须保证线程安全（框架层非线程安全）

### HAL 函数清单（19个）

#### 1. 生命周期管理

**HotaHalInit**
```c
/**
 * @brief OTA module initialization.
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalInit(void);
```
- **调用时机**: `HotaInit()` 时调用
- **职责**: 初始化 Flash 驱动、分配资源
- **必须**: 幂等（多次调用安全）

**HotaHalDeInit**
```c
/**
 * @brief Release OTA module resource.
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalDeInit(void);
```
- **调用时机**: `HotaCancel()` 或 `HotaInit()` 失败时
- **职责**: 释放资源、关闭 Flash

#### 2. 分区操作

**HotaHalWrite**
```c
/**
 * @brief Write image to partition.
 * @param partition [in] partition ID or special value
 * @param buffer    [in] image buffer
 * @param offset    [in] The buffer offset of file
 * @param bufLen    [in] The Length of buffer
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalWrite(int partition, unsigned char *buffer, 
                 unsigned int offset, unsigned int bufLen);
```
- **特殊 partition 值**:
  - `PARTITION_INFO_COMP` (1): Info 组件专用分区
  - `0`: 默认分区（自定义模式）
  - 其他: 组件表映射的分区 ID
- **要求**: 支持任意偏移写入、幂等（可重复写入相同数据）

**HotaHalRead**
```c
/**
 * @brief read image of partition.
 * @param partition [in] partition ID
 * @param offset    [in] The buffer offset of file
 * @param bufLen    [in] The Length of buffer
 * @param buffer    [out] image buffer
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalRead(int partition, unsigned int offset, 
                unsigned int bufLen, unsigned char *buffer);
```
- **用途**: 自定义模式下读取已写入数据

**HotaHalGetPartitionInfo**
```c
/**
 * @brief Get partition info.
 * @return Returns pointer to ComponentTableInfo array, terminated by NULL.
 */
const ComponentTableInfo *HotaHalGetPartitionInfo(void);
```
- **返回**: 分区表数组，以 `{0, NULL, NULL, 0}` 结束
- **示例**:
```c
static ComponentTableInfo g_componentTable[] = {
    {0, "kernel", "/dev/mmcblk0p1", 0},
    {1, "rootfs", "/dev/mmcblk0p2", 0},
    {0, NULL, NULL, 0}  // 结束标记
};
```

#### 3. 启动控制

**HotaHalSetBootSettings**
```c
/**
 * @brief Write Boot Settings in order to notify device upgrade success
 *        or enter Recovery Part.
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalSetBootSettings(void);
```
- **职责**: 
  - 设置启动标志（如 Bootloader 可读取的 GPT 属性）
  - 切换 A/B 分区槽位
  - 设置升级完成标志

**HotaHalRestart**
```c
/**
 * @brief Restart after upgrade finish or go bootloader to upgrade.
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalRestart(void);
```
- **职责**: 触发系统重启
- **注意**: 通常不会返回

**HotaHalRollback**
```c
/**
 * @brief Rollback if ota failed.
 * @return OHOS_SUCCESS: Success, Others: Failure.
 */
int HotaHalRollback(void);
```
- **用途**: 升级失败后回滚到之前版本

#### 4. 密钥与验证

**HotaHalGetPubKey**
```c
/**
 * @brief Get public key.
 * @param length [out] pubkey length in bytes
 * @return Pointer to public key buffer (DER format), NULL if failed.
 */
unsigned char *HotaHalGetPubKey(unsigned int *length);
```
- **要求**: 
  - 返回 DER 格式 RSA 公钥
  - 密钥长度应与签名算法匹配（RSA2048→256B, RSA3072→384B）
- **安全**: 公钥应存储在受保护区域（如 OTP、eFuse、只读分区）

**HotaHalCheckVersionValid**
```c
/**
 * @brief check whether pkgVersion is valid.
 * @return Returns 1 if pkgVersion is valid compared to currentVersion;
 *         Returns 0 otherwise.
 */
int HotaHalCheckVersionValid(const char *currentVersion, 
                             const char *pkgVersion, 
                             unsigned int pkgVersionLength);
```
- **用途**: 可覆盖默认的版本检查逻辑
- **默认**: 框架层使用 `IsLatestVersion()` 进行字符串比较

#### 5. 能力查询

**HotaHalGetUpdateAbility**
```c
int HotaHalGetUpdateAbility(void);
```
- **返回**: 能力位图（见 hota_partition.h:33-40）

**HotaHalGetUpdateIndex**
```c
int HotaHalGetUpdateIndex(unsigned int *index);
```
- **返回**: 1=A分区, 2=B分区

**HotaHalIsDeviceCanReboot**
```c
int HotaHalIsDeviceCanReboot(void);
```

**HotaHalIsDevelopMode**
```c
int HotaHalIsDevelopMode(void);
```

#### 6. 元数据管理

**HotaHalGetMetaData / HotaHalSetMetaData**
```c
int HotaHalGetMetaData(UpdateMetaData *metaData);
int HotaHalSetMetaData(UpdateMetaData *metaData);
```
- **用途**: 持久化升级状态
- **存储位置**: 建议 Bootloader 可访问的区域（如 misc 分区、GPT 属性）

**数据结构** (hota_partition.h:54-62):
```c
typedef struct {
    unsigned char updateMode;       // 升级模式
    unsigned char runningPartition; // 当前运行分区
    unsigned char updatePartition;  // 升级目标分区
    unsigned char runningStatus;    // 运行状态
    unsigned char otaStatus;        // OTA 状态
    unsigned char rebootStatus;     // 重启状态
    unsigned char updateStatus;     // 升级状态标志
} UpdateMetaData;
```

#### 7. 维护功能

**HotaHalGetOtaPkgPath**
```c
int HotaHalGetOtaPkgPath(char *path, int len);
```

**HotaHalRebootAndCleanUserData**
```c
int HotaHalRebootAndCleanUserData(void);
```

**HotaHalRebootAndCleanCache**
```c
int HotaHalRebootAndCleanCache(void);
```

---

## 芯片适配指南

### 适配步骤

```
1. 创建适配目录
   vendor/{vendor}/{board}/hals/update/

2. 实现 hal_hota_board.h 接口
   - hal_hota_board.c

3. 创建 BUILD.gn
   - 编译为 libhal_update.so 或静态库

4. 配置分区表
   - 在 g_componentTable 中定义分区映射

5. 实现公钥存储
   - 从安全存储读取公钥（OTP/eFuse/只读分区）

6. 实现启动控制
   - 与 Bootloader 协调分区切换

7. 实现元数据存储
   - misc 分区或 GPT 属性
```

### 最小实现示例（伪代码）

```c
// hal_hota_board.c
#include "hal_hota_board.h"
#include "flash_driver.h"

// 分区表定义
static ComponentTableInfo g_partitions[] = {
    {0, "kernel", "/dev/mmcblk0p1", 0},
    {1, "rootfs", "/dev/mmcblk0p2", 0},
    {0, NULL, NULL, 0}
};

int HotaHalInit(void) {
    FlashInit();
    return OHOS_SUCCESS;
}

int HotaHalWrite(int partition, unsigned char *buffer, 
                 unsigned int offset, unsigned int bufLen) {
    // 根据 partition ID 获取分区地址
    uint32_t addr = GetPartitionAddr(partition) + offset;
    FlashWrite(addr, buffer, bufLen);
    return OHOS_SUCCESS;
}

const ComponentTableInfo *HotaHalGetPartitionInfo(void) {
    return g_partitions;
}

unsigned char *HotaHalGetPubKey(unsigned int *length) {
    static unsigned char pubkey[] = { /* DER 格式 RSA 公钥 */ };
    *length = sizeof(pubkey);
    return pubkey;
}

int HotaHalSetBootSettings(void) {
    // 设置启动标志，切换 A/B 槽位
    SetBootFlag(UPGRADE_COMPLETE);
    return OHOS_SUCCESS;
}

int HotaHalRestart(void) {
    SystemReset();
    return OHOS_SUCCESS;  // 不会执行到这里
}

// ... 其他接口实现
```

### 关键注意事项

1. **幂等性**: `HotaHalWrite` 必须支持重复写入相同数据
2. **原子性**: `HotaHalSetBootSettings` 应原子性地更新启动标志
3. **安全存储**: 公钥必须从受保护区域读取，防止篡改
4. **版本持久化**: 元数据必须持久化，Bootloader 可读取
5. **错误处理**: 写入失败必须返回错误，不能静默忽略

---

## 接口稳定性标注

| 接口/模块 | 稳定性 | 说明 |
|-----------|--------|------|
| hota_updater.h | **稳定** | 对外 C API，向后兼容 |
| hal_hota_board.h | **稳定** | HAL 接口，向后兼容 |
| hota_updater.c 内部函数 | **不稳定** | 可能随版本变更 |
| app_rsa.c / app_sha256.c | **内部** | 不建议外部直接调用 |
| 全局变量 | **内部** | 非线程安全，不推荐外部访问 |

---

## 相关跳转

- [项目架构详解 → 01_Architecture.md](01_Architecture.md)
- [对外 API 说明 → 02_Public_API.md](02_Public_API.md)
- [构建系统说明 → 04_GN_Build.md](04_GN_Build.md)
- [安全风险分析 → 05_Security.md](05_Security.md)

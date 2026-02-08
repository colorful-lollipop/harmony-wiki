# 02 - 对外 API（C 接口）

## 目的与适用范围

**目的**: 完整说明 `sys_installer_lite` 提供的 C 语言对外接口。

**适用范围**: 需要集成 OTA 功能的应用开发者。

**重要说明**: 本项目提供的是 C 原生 API，**非 N-API**（无 JavaScript 绑定）。API 定义在 `interfaces/kits/hota_updater.h`。

---

## API 概览

### 头文件位置

```c
#include "hota_updater.h"  // interfaces/kits/hota_updater.h
```

### API 分类

| 分类 | 函数 | 数量 |
|------|------|------|
| 配置类 | HotaSetPackageType, HotaGetUpdateIndex, HotaGetUpdateAbility | 3 |
| 生命周期类 | HotaInit, HotaCancel, HotaSetBootSettings, HotaRestart | 4 |
| 数据传输类 | HotaWrite, HotaRead | 2 |
| 查询类 | HotaGetOtaPkgPath, HotaIsDeviceCanReboot, HotaIsDevelopMode, HotaGetUpdateStatus | 4 |
| 维护类 | HotaRebootAndCleanUserData, HotaRebootAndCleanCache | 2 |
| **总计** | | **15** |

---

## 数据类型定义

### 状态枚举 (hota_updater.h:46-53)

```c
typedef enum {
    HOTA_NOT_INIT = 0X01,           // 未初始化
    HOTA_INITED = 0x02,             // 已初始化
    HOTA_TRANSPORT_INFO_DONE = 0x03, // 信息组件传输完成
    HOTA_TRANSPORT_ALL_DONE = 0x04,  // 所有组件传输完成
    HOTA_FAILED = 0x05,             // 失败
    HOTA_CANCELED = 0x06            // 已取消
} HotaStatus;
```

### 错误码枚举 (hota_updater.h:55-62)

```c
typedef enum {
    HOTA_DATA_COMMON_ERR = 0,       // 通用错误
    HOTA_DATA_WRITE_ERR = 1,        // 写入错误
    HOTA_VERSION_INVALID = 2,       // 版本无效（防回滚）
    HOTA_DATA_SIGN_CHECK_ERR = 3,   // 签名验证失败
    HOTA_DATA_VERIFY_HASH_ERR = 4,  // 哈希校验失败
    HOTA_DATA_COPY_TO_BUFFER_ERR = 5 // 缓冲区拷贝失败
} HotaErrorCode;
```

### 回调函数类型 (hota_updater.h:64-65)

```c
typedef void (*ErrorCallBackFunc)(HotaErrorCode errorCode);
typedef void (*StatusCallBackFunc)(HotaStatus status);
```

---

## API 详细说明

### 1. HotaInit - 初始化 OTA 模块

**文件位置**: `interfaces/kits/hota_updater.h:115`

**函数原型**:
```c
int HotaInit(ErrorCallBackFunc errorCallback, StatusCallBackFunc statusCallback);
```

**功能描述**:
初始化 OTA 模块，分配资源，设置回调函数。

**参数说明**:

| 参数 | 类型 | 描述 | 可空 |
|------|------|------|------|
| errorCallback | ErrorCallBackFunc | 错误回调函数 | 是（NULL 表示不接收错误通知） |
| statusCallback | StatusCallBackFunc | 状态变更回调函数 | 是（NULL 表示不接收状态通知） |

**返回值**:
- `0`: 成功
- `-1`: 失败（内存分配失败或 HAL 初始化失败）

**调用示例**:
```c
// 简单初始化（无回调）
int ret = HotaInit(NULL, NULL);
if (ret != 0) {
    printf("HotaInit failed\n");
    return -1;
}

// 带回调的初始化
void OnError(HotaErrorCode code) {
    printf("OTA Error: %d\n", code);
}
void OnStatus(HotaStatus status) {
    printf("OTA Status: %d\n", status);
}
int ret = HotaInit(OnError, OnStatus);
```

**实现细节** (hota_updater.c:580-603):
1. 保存回调函数指针到 `g_otaNotifier`
2. 调用 `HotaHalInit()` 进行 HAL 层初始化
3. 分配 `g_infoCompBuff` 缓冲区（1500 字节）
4. 获取分区信息表 `HotaHalGetPartitionInfo()`
5. 更新状态为 `HOTA_INITED`

**错误处理**:
- 如果 `malloc` 失败返回 -1
- 如果 `HotaHalInit()` 失败返回其错误码

---

### 2. HotaWrite - 写入升级数据

**文件位置**: `interfaces/kits/hota_updater.h:129`

**函数原型**:
```c
int HotaWrite(unsigned char *buffer, unsigned int offset, unsigned int buffSize);
```

**功能描述**:
将升级包数据写入 Flash。支持分包写入，内部自动处理包解析和验证。

**参数说明**:

| 参数 | 类型 | 描述 | 约束 |
|------|------|------|------|
| buffer | unsigned char* | 数据缓冲区 | 不可为 NULL |
| offset | unsigned int | 数据在包中的偏移 | 从 0 开始递增 |
| buffSize | unsigned int | 数据大小 | 最大 4KB (MAX_TRANSPORT_BUFF_SIZE) |

**返回值**:
- `0`: 成功
- `-1`: 失败（参数错误、写入失败、验证失败）

**调用示例**:
```c
FILE *fp = fopen("ota_package.bin", "rb");
unsigned char buffer[1024];
size_t bytesRead;
unsigned int offset = 0;

while ((bytesRead = fread(buffer, 1, sizeof(buffer), fp)) > 0) {
    int ret = HotaWrite(buffer, offset, bytesRead);
    if (ret != 0) {
        printf("Write failed at offset %u\n", offset);
        HotaCancel();
        break;
    }
    offset += bytesRead;
}
fclose(fp);
```

**实现细节** (hota_updater.c:620-627):
- 根据 `g_useDefaultPkgFlag` 选择模式
- 默认模式: `HotaDefaultWrite()` - 自动解析包结构
- 自定义模式: `HotaHalWrite(0, ...)` - 直接透传数据

**HotaDefaultWrite 内部流程** (hota_updater.c:509-558):
1. 参数校验（buffer 非空，buffSize ≤ 4KB）
2. 检查状态是否被拒绝（`HotaIsRejected()`）
3. 处理 PkgBasicInfo（前 176 字节）
4. 累积 Info Component 数据
5. 分包写入数据组件到对应分区

**限制与约束**:
- 必须在 `HotaInit` 成功后调用
- 如果状态为 `HOTA_FAILED` 或 `HOTA_CANCELED` 会拒绝写入
- 数据大小不能超过 4KB/次
- 偏移必须连续递增

---

### 3. HotaRead - 读取已写入数据

**文件位置**: `interfaces/kits/hota_updater.h:146`

**函数原型**:
```c
int HotaRead(unsigned int offset, unsigned int bufLen, unsigned char *buf);
```

**功能描述**:
读取已写入 Flash 的数据，用于自定义包格式时的数据校验。

**参数说明**:

| 参数 | 类型 | 描述 | 约束 |
|------|------|------|------|
| offset | unsigned int | 读取偏移 | 必须有效 |
| bufLen | unsigned int | 读取长度 | 必须 ≤ buf 大小 |
| buf | unsigned char* | 输出缓冲区 | 不可为 NULL |

**返回值**:
- `0`: 成功
- `-1`: 失败（未使用自定义模式、参数错误、读取失败）

**使用条件**:
仅在 `NOT_USE_DEFAULT_PKG` 模式下可用（需先调用 `HotaSetPackageType(0)`）。

**调用示例**:
```c
// 必须在自定义模式下
HotaSetPackageType(NOT_USE_DEFAULT_PKG);

unsigned char readBuf[1024];
int ret = HotaRead(0, sizeof(readBuf), readBuf);
if (ret == 0) {
    // 校验 readBuf 数据...
}
```

**实现细节** (hota_updater.c:605-618):
```c
if (g_useDefaultPkgFlag == NOT_USE_DEFAULT_PKG) {
    return HotaHalRead(0, offset, bufLen, buf);
}
printf("UseOHOSPkgFlag is not open!");
return OHOS_FAILURE;
```

---

### 4. HotaCancel - 取消升级

**文件位置**: `interfaces/kits/hota_updater.h:158`

**函数原型**:
```c
int HotaCancel(void);
```

**功能描述**:
取消当前升级，释放资源，恢复状态。

**返回值**:
- `0`: 成功
- 其他: HAL 层错误码

**调用时机**:
- 升级失败需要重置时
- 用户主动取消升级时

**实现细节** (hota_updater.c:629-634):
1. 更新状态为 `HOTA_CANCELED`
2. 调用 `HotaResetStatus()` 重置所有全局变量
3. 调用 `HotaHalDeInit()` 释放 HAL 资源

---

### 5. HotaSetBootSettings - 设置启动参数

**文件位置**: `interfaces/kits/hota_updater.h:170`

**函数原型**:
```c
int HotaSetBootSettings(void);
```

**功能描述**:
设置系统启动参数，标记升级完成，准备重启。

**返回值**:
- `0`: 成功
- `-1`: 失败（OTA 未完成或 HAL 失败）

**前置条件**:
默认模式下必须达到 `HOTA_TRANSPORT_ALL_DONE` 状态。

**调用示例**:
```c
int ret = HotaSetBootSettings();
if (ret != 0) {
    printf("Failed to set boot settings\n");
    return -1;
}
// 接下来可以调用 HotaRestart()
```

**实现细节** (hota_updater.c:652-667):
1. 检查 `g_useDefaultPkgFlag` 模式
2. 默认模式下检查 `data.otaStatus == HOTA_TRANSPORT_ALL_DONE`
3. 调用 `HotaHalSetBootSettings()` 通知 HAL 层

---

### 6. HotaRestart - 重启系统

**文件位置**: `interfaces/kits/hota_updater.h:182`

**函数原型**:
```c
int HotaRestart(void);
```

**功能描述**:
重启设备完成升级。

**返回值**:
- `0`: 成功（通常不会返回，系统已重启）
- `-1`: 失败

**前置条件**:
必须先调用 `HotaSetBootSettings()`。

**实现细节** (hota_updater.c:636-650):
1. 检查状态（默认模式下）
2. 调用 `HotaHalRestart()` 触发重启

---

### 7. HotaSetPackageType - 设置包格式类型

**文件位置**: `interfaces/kits/hota_updater.h:86`

**函数原型**:
```c
int HotaSetPackageType(unsigned int flag);
```

**参数说明**:

| 值 | 宏定义 | 含义 |
|----|--------|------|
| 1 | `USE_DEFAULT_PKG` | 使用默认包格式（自动解析、签名验证） |
| 0 | `NOT_USE_DEFAULT_PKG` | 使用自定义包格式（透传到 HAL） |

**返回值**:
- `0`: 成功
- `-1`: 参数错误（非 0/1）

**默认行为**:
默认值为 `USE_DEFAULT_PKG` (1)，即自动解析和验证。

**实现细节** (hota_updater.c:560-569):
```c
if (flag != USE_DEFAULT_PKG && flag != NOT_USE_DEFAULT_PKG) {
    printf("flag is invalid. value = %u\r\n", flag);
    return OHOS_FAILURE;
}
g_useDefaultPkgFlag = flag;
```

---

### 8. HotaGetUpdateIndex - 获取升级分区索引

**文件位置**: `interfaces/kits/hota_updater.h:101`

**函数原型**:
```c
int HotaGetUpdateIndex(unsigned int *index);
```

**功能描述**:
在 A/B 分区场景下，获取当前应该升级的分区索引（1=A, 2=B）。

**参数说明**:

| 参数 | 类型 | 描述 | 约束 |
|------|------|------|------|
| index | unsigned int* | 输出分区索引 | 不可为 NULL |

**返回值**:
- `0`: 成功（index 被填充）
- `-1`: 失败（参数错误或 HAL 错误）

**实现细节** (hota_updater.c:571-578):
直接调用 `HotaHalGetUpdateIndex(index)`。

---

### 9. HotaGetUpdateAbility - 获取升级能力

**文件位置**: `interfaces/kits/hota_updater.h:194`

**函数原型**:
```c
int HotaGetUpdateAbility(void);
```

**功能描述**:
获取设备支持的升级能力位图。

**返回值**:
位图值（定义在 hota_partition.h:33-40）:

| 位 | 宏定义 | 能力 |
|----|--------|------|
| 0 | `ABILITY_DIFF_UPDATE` | 支持差分升级 |
| 1 | `ABILITY_PATCH_UPDATE` | 支持补丁升级 |
| 2 | `ABILITY_PKG_SEARCH` | 支持包搜索 |
| 3 | `ABILITY_PKG_DLOAD` | 支持包下载 |
| 4 | `ABILITY_UPDATE_AUTH` | 支持升级认证 |
| 5 | `ABILITY_AUTO_UPDATE` | 支持自动升级 |
| 6 | `ABILITY_FLOW_INSTALL` | 支持流式安装 |
| 7 | `ABILITY_AB_PART_INSTALL` | 支持 A/B 分区 |

**实现细节** (hota_updater.c:668-671):
直接调用 `HotaHalGetUpdateAbility()`，由 HAL 层返回能力位图。

---

### 10. HotaGetOtaPkgPath - 获取 OTA 包路径

**文件位置**: `interfaces/kits/hota_updater.h:209`

**函数原型**:
```c
int HotaGetOtaPkgPath(char *path, int len);
```

**功能描述**:
获取 OTA 升级包的存储路径。

**参数说明**:

| 参数 | 类型 | 描述 | 约束 |
|------|------|------|------|
| path | char* | 输出路径缓冲区 | 不可为 NULL |
| len | int | 缓冲区长度 | 必须足够容纳路径 |

**返回值**:
- `0`: 成功
- `-1`: 失败

---

### 11. HotaIsDeviceCanReboot - 检查设备是否可自动重启

**文件位置**: `interfaces/kits/hota_updater.h:221`

**函数原型**:
```c
int HotaIsDeviceCanReboot(void);
```

**返回值**:
- `1`: 设备支持自动重启
- `0`: 设备不支持自动重启

---

### 12. HotaIsDevelopMode - 检查是否为开发模式

**文件位置**: `interfaces/kits/hota_updater.h:233`

**函数原型**:
```c
int HotaIsDevelopMode(void);
```

**返回值**:
- `1`: 开发模式
- `0`: 非开发模式（正式版）

**安全意义**:
开发模式可能放宽某些安全检查，生产环境应确保返回 0。

---

### 13. HotaGetUpdateStatus - 获取升级状态

**文件位置**: `interfaces/kits/hota_updater.h:245`

**函数原型**:
```c
int HotaGetUpdateStatus(void);
```

**返回值**:
- `1`: 有正在进行的升级（`updateStatus != 0`）
- `0`: 无升级进行中

**实现细节** (hota_updater.c:688-693):
```c
UpdateMetaData data = {0};
HotaHalGetMetaData(&data);
return data.updateStatus ? 1 : 0;
```

---

### 14. HotaRebootAndCleanUserData - 重启并清理用户数据

**文件位置**: `interfaces/kits/hota_updater.h:257`

**函数原型**:
```c
int HotaRebootAndCleanUserData(void);
```

**功能描述**:
重启设备并清除用户数据分区（恢复出厂设置）。

**使用场景**:
- 升级失败后的恢复
- 用户主动恢复出厂设置

**返回值**:
- `0`: 成功
- `-1`: 失败

---

### 15. HotaRebootAndCleanCache - 重启并清理缓存

**文件位置**: `interfaces/kits/hota_updater.h:269`

**函数原型**:
```c
int HotaRebootAndCleanCache(void);
```

**功能描述**:
重启设备并清除缓存分区。

**返回值**:
- `0`: 成功
- `-1`: 失败

---

## API 调用链汇总

### 标准升级流程调用链

```
应用层
  │
  ├──► HotaInit(cbError, cbStatus)
  │      └──► [hota_updater.c:580] HotaHalInit()
  │             └──► [厂商实现] 硬件初始化
  │
  ├──► HotaWrite(buffer, offset, len) [多次调用]
  │      └──► [hota_updater.c:620] HotaDefaultWrite()
  │             ├──► [hota_updater.c:348] ProcessInfoCompHeader()
  │             │      └──► [hota_updater.c:210] CheckPkgVersionValid()
  │             ├──► [hota_updater.c:460] StashRecvDataToBuffer()
  │             │      └──► [hals/hal_hota_board.h:74] HotaHalWrite()
  │             └──► [hota_updater.c:227] ParseHotaInfoComponent()
  │                    └──► [hota_verify.c:132] HotaSignVerify()
  │                           ├──► [hota_verify.c:74] HotaCalcImageHash()
  │                           │      └──► [app_sha256.c] AppSha256Init/Update/Finish
  │                           └──► [hota_verify.c:103] HotaSignVerifyByHash()
  │                                  └──► [app_rsa.c:45] AppVerifyData()
  │                                         └──► [mbedtls] mbedtls_rsa_pkcs1_verify()
  │
  ├──► HotaSetBootSettings()
  │      └──► [hals/hal_hota_board.h:101] HotaHalSetBootSettings()
  │
  └──► HotaRestart()
         └──► [hals/hal_hota_board.h:112] HotaHalRestart()
                └──► [厂商实现] 系统重启
```

---

## 相关跳转

- [项目架构详解 → 01_Architecture.md](01_Architecture.md)
- [HAL 接口适配 → 03_Inner_API.md](03_Inner_API.md)
- [安全风险分析 → 05_Security.md](05_Security.md)

# 06 - 常见问题与调试

## 目的与适用范围

**目的**: 汇总 `sys_installer_lite` 开发、集成、调试过程中的常见问题及解决方案。

**适用范围**: 应用开发者、平台集成工程师。

---

## 构建问题

### Q1: 编译报错 "undefined reference to HotaHalInit"

**现象**:
```
linking libhota.so
undefined reference to `HotaHalInit'
undefined reference to `HotaHalWrite'
...
```

**原因**: 未链接 HAL 实现库。

**解决**:
1. 确认厂商已实现 HAL 接口（`hals/update/`）
2. 确认 `BUILD.gn` 中正确添加了依赖：
```gn
# 对于动态库
deps += [ "$ohos_board_adapter_dir/update:hal_update" ]
ldflags = [ "-lhal_update" ]

# 对于静态库（liteos_m）
deps += [ "$ohos_board_adapter_dir/hals/update:hal_update_static" ]
```

---

### Q2: 编译报错 "mbedtls/xxx.h not found"

**现象**:
```
frameworks/source/verify/app_rsa.c:18:10: fatal error: 
'mbedtls/entropy.h' file not found
```

**原因**: 未添加 mbedtls 头文件路径。

**解决**:
确认 `BUILD.gn` 包含：
```gn
include_dirs += [ "//third_party/mbedtls/include" ]
deps += [ "//third_party/mbedtls:mbedtls" ]
```

---

### Q3: 如何开启调试日志？

**解决**:
当前代码使用 `printf` 输出日志，可通过以下方式查看：

1. **串口输出**: 确保串口驱动已初始化
2. **文件重定向**: 在测试程序中重定向 stderr：
```c
freopen("/tmp/ota.log", "w", stderr);
HotaInit(NULL, NULL);
```

**日志级别**: 当前无日志级别控制，所有 `printf` 都会输出。

---

## 运行问题

### Q4: HotaInit 返回 -1

**排查步骤**:

1. **检查 HAL 实现**:
```c
// 确认 HotaHalInit 返回 OHOS_SUCCESS (0)
int ret = HotaHalInit();
printf("HotaHalInit ret = %d\n", ret);
```

2. **检查内存分配**:
```c
// HotaInit 内部 malloc 1500 字节
g_infoCompBuff = (unsigned char *)malloc(MAX_BUFFER_SIZE);  // 1500
```
检查系统是否有足够堆内存。

3. **检查分区表**:
```c
// 确认 HotaHalGetPartitionInfo 返回有效数据
const ComponentTableInfo *table = HotaHalGetPartitionInfo();
if (table == NULL || table[0].componentName == NULL) {
    printf("Partition table is empty!\n");
}
```

---

### Q5: HotaWrite 返回 -1

**排查步骤**:

1. **检查状态**:
```c
// 确认已调用 HotaInit
// 确认状态不是 HOTA_FAILED 或 HOTA_CANCELED
```

2. **检查参数**:
```c
// buffSize 不能超过 4KB
if (buffSize > 4 * 1024) {
    printf("buffSize too large: %u\n", buffSize);
}

// buffer 不能为 NULL
if (buffer == NULL) {
    printf("buffer is NULL\n");
}
```

3. **检查 HAL 写入**:
```c
// 在 HAL 实现中添加调试日志
int HotaHalWrite(int partition, unsigned char *buffer, 
                 unsigned int offset, unsigned int bufLen) {
    printf("HotaHalWrite: partition=%d, offset=%u, len=%u\n", 
           partition, offset, bufLen);
    // ...
}
```

---

### Q6: 签名验证失败（HOTA_DATA_SIGN_CHECK_ERR）

**排查步骤**:

1. **检查公钥**:
```c
// 确认 HotaHalGetPubKey 返回正确的 DER 格式公钥
unsigned int len = 0;
unsigned char *key = HotaHalGetPubKey(&len);
printf("PubKey length: %u\n", len);
// RSA2048: 256 bytes
// RSA3072: 384 bytes
```

2. **检查升级包签名算法**:
```c
// 确认 PkgBasicInfo.type 与公钥匹配
// SIGN_ARITHMETIC_RSA2048 = 0x0001
// SIGN_ARITHMETIC_RSA3072 = 0x0011
```

3. **验证签名数据位置**:
```c
// 签名数据位于 Info Component 末尾
// g_signStartAddr 根据 type 自动计算
```

4. **使用工具验证**:
```bash
# 使用 openssl 验证签名
openssl dgst -sha256 -verify public_key.pem -signature sign.bin data.bin
```

---

### Q7: 哈希校验失败（HOTA_DATA_VERIFY_HASH_ERR）

**排查步骤**:

1. **检查组件数据完整性**:
```c
// 确认 HotaWrite 写入了完整数据
// 检查网络/存储传输是否完整
```

2. **检查组件表 SHA256**:
```c
// ComponentInfo.shaData 应为组件数据的 SHA256
// 使用 sha256sum 工具验证
sha256sum component.bin
```

3. **检查分块哈希计算**:
```c
// 调试 hota_verify.c:39-48 HotaHashCalc
void HotaHashCalc(const uint8 *buffer, uint32 length) {
    printf("HashCalc: length=%u\n", length);
    // ...
}
```

---

### Q8: 版本检查失败（HOTA_VERSION_INVALID）

**排查步骤**:

1. **检查版本号格式**:
```c
// 版本号应为数字和点号组成，如 "1.0.2"
// 分隔符: ".| " (点、竖线、空格)
```

2. **检查当前版本**:
```c
// 确认 GetIncrementalVersion() 返回正确值
const char *current = GetIncrementalVersion();
printf("Current version: %s\n", current);
```

3. **检查包版本**:
```c
// 确认 PkgBasicInfo.version 正确设置
// 必须大于当前版本
```

4. **检查 HAL 覆盖**:
```c
// 如果实现了 HotaHalCheckVersionValid，确认逻辑正确
```

---

## HAL 适配问题

### Q9: 如何适配新的 Flash 类型？

**示例**:
```c
// hal_hota_board.c
#include "hal_hota_board.h"
#include "my_flash_driver.h"

int HotaHalWrite(int partition, unsigned char *buffer, 
                 unsigned int offset, unsigned int bufLen) {
    // 将 partition ID 映射到 Flash 地址
    uint32_t baseAddr = GetPartitionBaseAddr(partition);
    uint32_t writeAddr = baseAddr + offset;
    
    // 擦除 Flash（按 sector）
    if (offset % SECTOR_SIZE == 0) {
        FlashEraseSector(writeAddr);
    }
    
    // 写入数据
    FlashWrite(writeAddr, buffer, bufLen);
    
    return OHOS_SUCCESS;
}
```

---

### Q10: A/B 分区如何切换？

**方案 1: Bootloader 控制**:
```c
int HotaHalSetBootSettings(void) {
    // 设置启动标志，通知 Bootloader 切换分区
    SetBootFlag(BOOT_ALTERNATE);
    return OHOS_SUCCESS;
}

int HotaHalRestart(void) {
    // 重启后 Bootloader 读取标志并切换
    SystemReset();
    return OHOS_SUCCESS;
}
```

**方案 2: GPT 属性**:
```c
int HotaHalSetBootSettings(void) {
    // 修改 GPT 分区属性
    SetPartitionActive(updatePartition);
    return OHOS_SUCCESS;
}
```

---

### Q11: 公钥存储在哪里？

**推荐方案**:

1. **OTP/eFuse**（最安全）:
```c
unsigned char *HotaHalGetPubKey(unsigned int *length) {
    static unsigned char pubkey[256];
    // 从 OTP 读取
    OTP_Read(OTP_ADDR_PUBKEY, pubkey, 256);
    *length = 256;
    return pubkey;
}
```

2. **只读分区**:
```c
unsigned char *HotaHalGetPubKey(unsigned int *length) {
    // 从只读分区读取
    static unsigned char *pubkey = NULL;
    if (pubkey == NULL) {
        pubkey = mmap(NULL, 256, PROT_READ, MAP_PRIVATE, 
                      fd, PUBKEY_PARTITION_OFFSET);
    }
    *length = 256;
    return pubkey;
}
```

---

## 调试技巧

### 启用详细日志

在源码中添加调试日志：

```c
// hota_updater.c 关键位置添加
#define OTA_DEBUG 1

#if OTA_DEBUG
#define OTA_LOG(fmt, ...) printf("[OTA] " fmt "\r\n", ##__VA_ARGS__)
#else
#define OTA_LOG(fmt, ...)
#endif

// 使用示例
OTA_LOG("HotaWrite: offset=%u, size=%u", offset, buffSize);
```

### 状态机追踪

添加状态变更日志：

```c
static void UpdateStatus(HotaStatus status) {
    OTA_LOG("Status change: %d -> %d", g_otaStatus, status);
    // ...
}
```

### 数据转储

调试时转储关键数据：

```c
// 转储缓冲区内容
void DumpBuffer(const char *name, unsigned char *buf, int len) {
    printf("%s (%d bytes): ", name, len);
    for (int i = 0; i < len && i < 32; i++) {
        printf("%02x ", buf[i]);
    }
    printf("...\n");
}

// 在 HotaSignVerify 中使用
DumpBuffer("Image Hash", hashOut, HASH_LENGTH);
DumpBuffer("Signature", imageSign, signLen);
```

### 使用 GDB 调试

```bash
# 1. 编译时添加 -g
cflags = [ "-g", "-O0" ]

# 2. 使用 gdbserver 远程调试
gdbserver :2345 ./ota_app

# 3. 本地连接
gdb ./ota_app
target remote device_ip:2345
break HotaWrite
continue
```

---

## 性能优化

### 写入速度优化

1. **增大传输缓冲区**:
```c
// 默认 4KB，可增大到 Flash page 大小
#define MAX_TRANSPORT_BUFF_SIZE (64 * 1024)  // 64KB
```

2. **批量擦除**:
```c
// 在 HotaHalWrite 中缓存擦除操作
int HotaHalWrite(...) {
    // 延迟擦除，批量处理
}
```

### 内存使用优化

1. **减小缓冲区**（内存受限设备）:
```c
// 默认 1500，最小可设为 512
#define MAX_BUFFER_SIZE 512
```

2. **静态分配**:
```c
// 避免动态分配
static unsigned char g_infoCompBuff[MAX_BUFFER_SIZE];
```

---

## 相关跳转

- [项目架构详解 → 01_Architecture.md](01_Architecture.md)
- [对外 API 说明 → 02_Public_API.md](02_Public_API.md)
- [HAL 接口适配 → 03_Inner_API.md](03_Inner_API.md)
- [安全风险分析 → 05_Security.md](05_Security.md)

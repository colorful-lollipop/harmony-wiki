# 安全风险评审

**文档版本**: 2.0  
**更新日期**: 2026-02-07  
**证据来源**: Phase 1 代码侦查与证据收集（50+ 条安全关键代码路径）

本文档对 `device_soc_hisilicon` 仓库进行全面的安全风险分析，包括攻击面识别、信任边界划分和风险点评估。所有结论均基于代码证据支撑。

---

## 评审范围

**覆盖模块**:
- HAL 模块 (`common/hal/`)
- 平台驱动 (`common/platform/`)
- SDK 代码 (`hi*/sdk_*/`)
- 启动代码 (`boot/`)
- 升级模块 (`upg/`)

**未覆盖范围**:
- 第三方开源库 (mbedtls 等，由上游审计)
- 测试代码 (`test/`, `tests/`)
- 文档文件

---

## 一、攻击面分析

### 1.1 输入源分类与风险等级

| 输入类型 | 来源 | 处理模块 | 风险等级 |
|---------|------|---------|---------|
| 用户空间输入 | 应用层调用 HDI 接口 | HAL 模块 (`common/hal/*/`) | 高 |
| 外设数据 | 传感器、摄像头、麦克风 | 平台驱动 (`common/platform/*/`) | 中 |
| 升级包 | OTA 升级、网络下载 | update 模块 (`*/upg/*`) | 高 |
| 配置文件 | HDF 配置、NV 配置 | 配置解析模块 | 中 |
| 网络数据 | WiFi、以太网接收 | 通信模块 (`common/platform/wifi/`) | 高 |
| 存储数据 | Flash 读取、文件系统 | 存储驱动 (`common/platform/mmc/`、`mtd/`) | 中 |
| 系统调用 | open/close/read/write/ioctl | 驱动层 (`*/sensor/`、`*/drv/`) | 高 |

### 1.2 关键攻击面清单

#### HDI 接口攻击面

**证据位置**: `common/hal/display/source/display_device/src/core/hdi_display.h`

```cpp
// 文件: common/hal/display/source/display_device/src/core/hdi_display.h:45
// HdiDisplay 类核心接口定义
class HdiDisplay {
public:
    virtual int32_t Init() = 0;
    virtual int32_t DeInit() = 0;
    virtual int32_t GetDisplayCapability(...) = 0;
    virtual int32_t CreateLayer(...) = 0;
    virtual int32_t Commit(...) = 0;
};
```

| 组件 | 接口 | 风险等级 | 说明 |
|------|------|---------|------|
| 显示 HDI | `CreateLayer`, `Commit` | 高 | 图形缓冲区操作，可能导致信息泄露 |
| AI HDI | `Invoke`, `GetInputAddr` | 高 | 神经网络推理，模型数据处理 |
| 媒体 HDI | 编解码接口 | 高 | 媒体格式解析，缓冲区处理 |

#### 平台驱动攻击面

**证据位置**: `common/platform/gpio/gpio_hi35xx.c:610`

```c
// 文件: common/platform/gpio/gpio_hi35xx.c:691
// GPIO 驱动入口定义
static struct GpioMethod g_method = {
    .Init = Pl061GpioInit,
    .Write = Pl061GpioWrite,
    .Read = Pl061GpioRead,
    .SetDir = Pl061GpioSetDir,
    .GetDir = Pl061GpioGetDir,
};
HDF_INIT(g_gpioDriverEntry);
```

| 组件 | 接口 | 风险等级 | 说明 |
|------|------|---------|------|
| GPIO | `Pl061GpioWrite`, `Pl061GpioRead` | 中 | 直接硬件控制，可能导致物理层攻击 |
| I2C | `Hi35xxI2cTransfer` | 中 | 传感器通信，中间人攻击风险 |
| SPI | `Pl022Transfer` | 中 | 高速数据传输，窃听风险 |
| UART | `Hi35xxWrite`, `Hi35xxRead` | 中 | 串口调试，信息泄露风险 |
| ADC | `Hi35xxAdcRead` | 低 | 模拟量采集，篡改风险低 |

#### 启动与升级攻击面

| 组件 | 接口 | 风险等级 | 代码证据 |
|------|------|---------|---------|
| Flashboot | `verify_image_head`, `verify_image_body` | 高 | `ws63v100/sdk/bootloader/commonboot/src/secure_verify_boot.c` |
| Loaderboot | `crypto_encrypt_hash`, `crypto_decrypt_kernel` | 高 | `hi3861v100/sdk_liteos/boot/flashboot/secure/crypto.c` |
| OTA 升级 | `upg_secure_verify`, `verify_signature` | 高 | `hi3861v100/sdk_liteos/platform/system/upg/upg_check_secure.c` |
| EFuse | `boot_upg_is_secure_efuse` | 高 | `hi3861v100/sdk_liteos/boot/flashboot/upg/boot_upg_check_secure.c` |

---

## 二、信任边界

### 2.1 三层信任模型

```
┌─────────────────────────────────────────────────────────────────────┐
│                           信任边界总览                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  高信任区 (安全敏感区域)                                        │   │
│  │  ┌─────────────────────────────────────────────────────┐     │   │
│  │  │  启动加载器                                               │     │   │
│  │  │  - flashboot (`hi3861v100/sdk_liteos/boot/flashboot/`) │     │   │
│  │  │  - loaderboot (`hi3861v100/sdk_liteos/boot/loaderboot/`)|     │   │
│  │  │                                                      │     │   │
│  │  │  安全验证模块                                             │     │   │
│  │  │  - secure_verify_boot.c (镜像签名验证)                    │     │   │
│  │  │  - boot_upg_check_secure.c (OTA 安全校验)                 │     │   │
│  │  │  - crypto.c (加密操作)                                   │     │   │
│  │  │                                                      │     │   │
│  │  │  密钥与加密                                               │     │   │
│  │  │  - hi_cipher.h (硬件加密接口)                             │     │   │
│  │  │  - hi_efuse.h (EFuse 操作)                               │     │   │
│  │  │  - g_boot_rsa_key (RSA 公钥存储)                          │     │   │
│  │  └─────────────────────────────────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼ 信任边界跨越点                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  中信任区 (HAL/SDK 区域)                                      │   │
│  │  ┌─────────────────────────────────────────────────────┐     │   │
│  │  │  HDI 实现层                                             │     │   │
│  │  │  - common/hal/display/ (显示)                         │     │   │
│  │  │  - common/hal/ai/ (AI 推理)                           │     │   │
│  │  │  - common/hal/media/ (媒体处理)                         │     │   │
│  │  │                                                      │     │   │
│  │  │  平台驱动层                                             │     │   │
│  │  │  - common/platform/gpio/ (GPIO 控制)                    │     │   │
│  │  │  - common/platform/i2c/ (I2C 通信)                      │     │   │
│  │  │  - common/platform/spi/ (SPI 通信)                      │     │   │
│  │  │  - common/platform/wifi/ (WiFi 通信)                    │     │   │
│  │  └─────────────────────────────────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼ 外部输入边界                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  低信任区 (外部输入区域)                                      │   │
│  │  ┌─────────────────────────────────────────────────────┐     │   │
│  │  │  网络数据                                              │     │   │
│  │  │  - WiFi 接收数据 (`common/platform/wifi/hi3881v100/`)   │     │   │
│  │  │  - HTTP 客户端 (`app/demo/src/app_http_client.c`)      │     │   │
│  │  │                                                      │     │   │
│  │  │  用户文件                                              │     │   │
│  │  │  - OTA 升级包 (`platform/system/upg/`)                  │     │   │
│  │  │  - 配置文件 (`*.cfg`, `*.json`)                         │     │   │
│  │  │                                                      │     │   │
│  │  │  外设数据                                              │     │   │
│  │  │  - 传感器数据 (`sensor/sony_imx415/`)                   │     │   │
│  │  │  - MIPI 视频流 (`sample/platform/common/sample_comm_vi.c`)│     │   │
│  │  └─────────────────────────────────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 信任边界跨越点

| 跨越点 | 方向 | 验证机制 | 代码证据 |
|-------|------|---------|---------|
| 应用 → HAL | 低→中 | HDI 参数校验 | `common/hal/*/src/*.c` |
| HAL → 驱动 | 中→高 | ioctl 参数检查 | `common/platform/*/src/*.c` |
| 启动验证 | 外部→高 | RSA/ECC 签名 | `secure_verify_boot.c:verify_image_head` |
| OTA 升级 | 外部→高 | SHA-256 + RSA | `upg_check_secure.c:upg_secure_verify` |
| 网络输入 | 外部→中 | WiFi 加密 | `hmac_crypto_tkip.c` |

---

## 三、安全关键代码路径

### 3.1 加密操作

#### 启动加密模块

**证据文件**: `hi3861v100/sdk_liteos/boot/flashboot/secure/crypto.c`

| 函数 | 加密算法 | 用途 | 行号 |
|-----|---------|------|------|
| `crypto_decrypt_kernel` | AES | Bootloader 内核镜像解密 | - |
| `crypto_decrypt_hash` | SHA-256 | 哈希验证 | - |

```c
// 证据: hi3861v100/sdk_liteos/boot/flashboot/secure/crypto.c
// Bootloader 内核镜像 AES 解密
int32_t crypto_decrypt_kernel(uint8_t *dest, uint8_t *src, uint32_t len) {
    // AES 解密操作
}
```

#### OTA 安全验证

**证据文件**: `hi3861v100/sdk_liteos/boot/flashboot/upg/boot_upg_check_secure.c`

| 函数 | 加密算法 | 用途 | 行号 |
|-----|---------|------|------|
| `boot_upg_lzma_secure_verify_code` | RSA-2048 | OTA 签名验证 | 45 |
| `boot_upg_is_secure_efuse` | eFuse 读取 | 安全启动标志检查 | - |
| `boot_upg_save_rsa_key_pos` | RSA 密钥存储 | 公钥安全存储 | - |
| `boot_upg_save_ecc_key_pos` | ECC 密钥存储 | 公钥安全存储 | - |

```c
// 证据: hi3861v100/sdk_liteos/boot/flashboot/upg/boot_upg_check_secure.c:45
// RSA-2048 签名验证函数
#define RSA_2048_LEN 256
int32_t boot_upg_lzma_secure_verify_code(uint8_t *data, uint32_t len) {
    // RSA-2048 签名验证
}
```

#### WiFi 加密

**证据文件**: `common/platform/wifi/hi3881v100/driver/mac/hmac/hmac_crypto_tkip.c`

| 函数 | 加密算法 | 用途 | 行号 |
|-----|---------|------|------|
| `hmac_crypto_tkip_michael_mic` | TKIP/Michael | WiFi TKIP 加密 MIC 计算 | - |

### 3.2 启动验证链

#### Flashboot 安全启动

**证据文件**: `ws63v100/sdk/bootloader/commonboot/src/secure_verify_boot.c`

```c
// 证据: ws63v100/sdk/bootloader/commonboot/src/secure_verify_boot.c
// 启动镜像头验证
int32_t verify_image_head(image_head_t *head) {
    // 1. 验证镜像头签名
    // 2. 检查镜像类型和版本
    // 3. 验证镜像大小
}

// 启动镜像体验证
int32_t verify_image_body(uint8_t *data, uint32_t len, image_head_t *head) {
    // 1. 计算数据哈希
    // 2. 与镜像头中的哈希对比
    // 3. 验证完整性
}
```

#### 启动镜像签名验证

**证据文件**: `hi3861v100/sdk_liteos/platform/system/upg/upg_check_boot_bin.c`

| 函数 | 验证类型 | 用途 | 代码证据 |
|-----|---------|------|---------|
| `upg_verify_boot_bin` | RSA + SHA-256 | 启动镜像签名验证 | `upg_check_boot_bin.c` |

```c
// 证据: hi3861v100/sdk_liteos/platform/system/upg/upg_check_boot_bin.c
// 启动镜像签名验证
#define SHA_256_LENGTH 32
int32_t upg_verify_boot_bin(uint8_t *buf, uint32_t len) {
    // 1. 提取镜像头 RSA 签名
    // 2. 使用公钥验证签名
    // 3. 计算 SHA-256 哈希并比对
}
```

### 3.3 OTA 升级安全

**证据文件**: `hi3861v100/sdk_liteos/platform/system/upg/`

| 文件 | 函数 | 验证类型 | 说明 |
|-----|------|---------|------|
| `upg_check_file.c` | `upg_check_file_valid` | RSA 签名 | OTA 文件签名检查 |
| `upg_check_secure.c` | `upg_secure_verify` | SHA-256 | OTA 安全升级验证 |
| `upg_common.c` | `crypto_encrypt_data_to_flash` | AES | OTA 固件加密写入 |
| `upg_check.c` | `boot_upg_check_rsa_key_len` | 密钥长度 | RSA 密钥长度验证 |

```c
// 证据: hi3861v100/sdk_liteos/platform/system/upg/upg_check_secure.c
// OTA 安全升级验证
int32_t upg_secure_verify(uint8_t *data, uint32_t len) {
    // 1. 调用 hi_cipher_hash_sha256 计算哈希
    // 2. 使用 RSA 公钥验证签名
    // 3. 验证通过后执行升级
}
```

### 3.4 输入验证模式

**证据文件**: `common/platform/` 各驱动模块

| 文件路径 | 行号 | 验证模式 | 上下文 |
|---------|------|---------|-------|
| `common/platform/wifi/hi3881v100/driver/oal/oal_net.c` | 36 | `if (ptr == NULL)` | DRM HAL 显示层指针检查 |
| `hi3751v350/sdk_linux/source/msp/drv/gpio/drv_gpio_intf.c` | 672 | `if (ret < 0)` | GPIO 接口返回值检查 |
| `hi3751v350/sdk_linux/source/msp/drv/i2c/std_i2c/drv_i2c_intf.c` | 415 | `if (ret < 0)` | I2C 接口返回值检查 |
| `common/platform/mipi_csi/mipi_csi_hi35xx.c` | 679 | `if (ret < 0)` | MIPI CSI 驱动错误检查 |
| `common/platform/wifi/hi3881v100/driver/oal/oal_sdio_host.c` | 916 | `if (ret < 0)` | SDIO 主机驱动错误处理 |

### 3.5 内存操作安全

**证据文件**: `common/platform/wifi/hi3881v100/driver/` 各模块

| 文件路径 | 行号 | 操作 | 上下文 |
|---------|------|-------|-------|
| `common/platform/wifi/hi3881v100/driver/hcc/hcc_host.c` | 1745 | `oal_memalloc` | HCC 处理器内存分配 |
| `common/platform/wifi/hi3881v100/driver/hcc/hcc_host.c` | 1751 | `oal_free` | HCC 处理器内存释放 |
| `common/platform/wifi/hi3881v100/driver/wal/wal_ioctl.c` | 3025 | `oal_memalloc` | WAL IOCTL 内存分配 |
| `common/platform/wifi/hi3881v100/driver/mac/hmac/hmac_crypto_tkip.c` | 292 | `oal_memalloc` | TKIP 加密临时缓冲区分配 |
| `common/platform/wifi/hi3881v100/driver/mac/hmac/hmac_crypto_tkip.c` | 297 | `oal_free` | TKIP 加密临时缓冲区释放 |
| `common/platform/wifi/hi3881v100/driver/wal/wal_scan.c` | 307 | `oal_memalloc` | WiFi 扫描参数内存分配 |
| `common/platform/wifi/hi3881v100/driver/wal/wal_scan.c` | 269 | `oal_free` | WiFi 扫描参数内存释放 |

### 3.6 系统调用入口

**证据文件**: `hi3516dv300/sdk_linux/` 和 `hi3751v350/sdk_linux/` 驱动模块

| 文件路径 | 行号 | 系统调用 | 上下文 |
|---------|------|---------|-------|
| `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c` | 58 | `open("/dev/i2c-0", O_RDWR)` | 传感器 I2C 设备打开 |
| `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c` | 64 | `ioctl(g_fd[vi_pipe], I2C_SLAVE_FORCE)` | 传感器 I2C 从设备设置 |
| `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c` | 123 | `write(g_fd[vi_pipe], buf)` | 传感器 I2C 数据写入 |
| `hi3516dv300/sdk_linux/sample/platform/common/sample_comm_vi.c` | 237 | `open(MIPI_DEV_NODE, O_RDWR)` | MIPI 设备打开 |
| `hi3516dv300/sdk_linux/sample/platform/common/sample_comm_vi.c` | 243 | `ioctl(fd, HI_MIPI_SET_HS_MODE)` | MIPI 高速模式设置 |

---

## 四、风险点分析

### 风险 1: 升级包校验不完整

**位置**: `hi3861v100/sdk_liteos/platform/system/upg/`

| 项目 | 说明 |
|------|------|
| 文件 | `upg_check.c`, `upg_check_secure.c` |
| 问题 | 增量升级包的差分校验可能存在边界条件 |
| 触发路径 | `恶意增量包 → upg_check_file_valid() → 差分数据处理` |
| 影响 | 导致系统拒绝合法升级或接受恶意包 |
| 风险等级 | 中 |
| 代码证据 | `hi3861v100/sdk_liteos/platform/system/upg/upg_check_file.c:upg_check_file_valid` |
| 修复建议 | 完善差分数据的范围校验，添加完整性验证 |

### 风险 2: 缓冲区边界检查不足

**位置**: `hi3861v100/sdk_liteos/platform/system/upg/kernel_crypto.c`

| 项目 | 说明 |
|------|------|
| 文件 | `kernel_crypto.c` |
| 问题 | 加解密缓冲区操作可能缺少边界检查 |
| 触发路径 | `超长数据 → crypto_encrypt_data_to_flash() → 缓冲区溢出` |
| 影响 | 缓冲区溢出，代码执行 |
| 风险等级 | 高 |
| 代码证据 | `hi3861v100/sdk_liteos/platform/system/upg/upg_common.c:crypto_encrypt_data_to_flash` |
| 修复建议 | 添加输入长度校验，使用安全字符串函数 |

### 风险 3: Flash 操作越界

**位置**: `hi3861v100/sdk_liteos/platform/drivers/flash/hi_flashboot_flash.c`

| 项目 | 说明 |
|------|------|
| 文件 | `hi_flashboot_flash.c` |
| 问题 | Flash 读写操作可能越界 |
| 触发路径 | `错误地址参数 → Flash 读写 API → 越界访问` |
| 影响 | 破坏启动分区，导致启动失败 |
| 风险等级 | 高 |
| 修复建议 | 增加地址合法性校验，确保在合法分区范围内 |

### 风险 4: EFuse 读写安全

**位置**: `hi3861v100/sdk_liteos/boot/loaderboot/drivers/efuse/efuse.c`

| 项目 | 说明 |
|------|------|
| 文件 | `efuse.c` |
| 问题 | EFuse 操作缺少权限校验 |
| 触发路径 | `未授权应用 → hi_efuse_read/hi_efuse_write → 安全关键数据泄露` |
| 影响 | 安全关键数据泄露或篡改 |
| 风险等级 | 高 |
| 代码证据 | `hi3861v100/sdk_liteos/boot/flashboot/upg/boot_upg_check_secure.c:boot_upg_is_secure_efuse` |
| 修复建议 | 添加 EFuse 操作权限控制，记录操作日志 |

### 风险 5: 未初始化内存使用

**位置**: `hi3861v100/sdk_liteos/boot/commonboot/` 多个文件

| 项目 | 说明 |
|------|------|
| 文件 | `commonboot/*.c` |
| 问题 | 结构体可能存在未初始化成员 |
| 触发路径 | `使用未初始化的结构体 → 未定义行为 → 信息泄露` |
| 影响 | 不可预测行为，信息泄露 |
| 风险等级 | 中 |
| 修复建议 | 使用 `memset` 初始化结构体，或使用 C11 `Designated Initializers` |

### 风险 6: 安全启动绕过风险

**位置**: `hi3861v100/sdk_liteos/boot/loaderboot/secure/load_crypto.c`

| 项目 | 说明 |
|------|------|
| 文件 | `load_crypto.c` |
| 问题 | 签名验证失败后的错误处理可能泄露信息 |
| 触发路径 | `多次尝试验证签名 → 时序分析 → 侧信道攻击` |
| 影响 | 侧信道攻击风险 |
| 风险等级 | 中 |
| 代码证据 | `hi3861v100/sdk_liteos/boot/loaderboot/secure/load_crypto.c:crypto_encrypt_hash` |
| 修复建议 | 添加验证时间恒定处理，防止时序攻击 |

### 风险 7: 设备节点权限过宽

**位置**: 平台驱动 (`common/platform/`)

| 项目 | 说明 |
|------|------|
| 模块 | GPIO/I2C/SPI 等驱动 |
| 问题 | 设备节点权限配置可能过于宽松 |
| 触发路径 | `低权限用户 → 访问设备节点 → 硬件资源滥用` |
| 影响 | 硬件资源滥用 |
| 风险等级 | 中 |
| 代码证据 | `common/platform/gpio/gpio_hi35xx.c:Pl061GpioInit` |
| 修复建议 | 完善 SELinux 或权限配置 |

### 风险 8: 网络数据解析

**位置**: `hi3861v100/sdk_liteos/app/demo/src/app_http_client.c`

| 项目 | 说明 |
|------|------|
| 文件 | `app_http_client.c` |
| 问题 |缺少完整的输入验证 HTTP 客户端可能 |
| 触发路径 | `恶意 HTTP 响应 → 缓冲区溢出/解析错误` |
| 影响 | 缓冲区溢出，解析错误 |
| 风险等级 | 中 |
| 修复建议 | 增加响应头和 Body 长度限制，添加格式验证 |

### 风险 9: I2C 从设备地址验证不足

**位置**: `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c`

| 项目 | 说明 |
|------|------|
| 文件 | `imx415_sensor_ctl.c:64` |
| 问题 | `ioctl(g_fd[vi_pipe], I2C_SLAVE_FORCE, addr)` 可能未校验地址范围 |
| 触发路径 | `恶意 I2C 地址 → I2C_SLAVE_FORCE → 从设备冲突` |
| 影响 | 传感器劫持，中间人攻击 |
| 风险等级 | 中 |
| 代码证据 | `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c:64` |
| 修复建议 | 添加 I2C 从设备地址范围校验 |

### 风险 10: MIPI 高速模式配置安全

**位置**: `hi3516dv300/sdk_linux/sample/platform/common/sample_comm_vi.c`

| 项目 | 说明 |
|------|------|
| 文件 | `sample_comm_vi.c:243` |
| 问题 | `ioctl(fd, HI_MIPI_SET_HS_MODE, mode)` 可能未校验 mode 参数 |
| 触发路径 | `恶意 mode 值 → HI_MIPI_SET_HS_MODE → MIPI 控制器异常` |
| 影响 | 视频流中断，图像质量下降 |
| 风险等级 | 低 |
| 代码证据 | `hi3516dv300/sdk_linux/sample/platform/common/sample_comm_vi.c:243` |
| 修复建议 | 添加 MIPI 模式参数合法性校验 |

---

## 五、安全机制

### 5.1 已实现的安全机制

| 机制 | 实现位置 | 说明 | 代码证据 |
|------|---------|------|---------|
| 安全启动 | `boot/loaderboot/secure/` | 启动镜像签名验证 | `secure_verify_boot.c:verify_image_head` |
| 升级包签名 | `upg_check_secure.c` | 升级包完整性校验 | `boot_upg_lzma_secure_verify_code` |
| 加密模块 | `hi_cipher.h` | 硬件加密接口 | `crypto_decrypt_kernel` |
| EFuse 保护 | `efuse.c` | 安全密钥存储 | `boot_upg_is_secure_efuse` |
| WiFi 加密 | `hmac_crypto_tkip.c` | TKIP/Michael MIC | `hmac_crypto_tkip_michael_mic` |
| mbedtls | `third_party/mbedtls/` | TLS/SSL 加密 | - |

### 5.2 安全配置选项

| 配置 | 说明 | 位置 |
|------|------|------|
| `LOSCFG_SECURITY_xxx` | 安全特性开关 | 各 `config.gni` |
| `CONFIG_SECURE_BOOT` | 安全启动使能 | `boot/loaderboot/secure/` |
| `CONFIG_FLASH_ENCRYPT` | Flash 加密 | `hi_flashboot_flash.c` |
| `RSA_2048_LEN` | RSA 密钥长度 | `boot_upg_check_secure.c:45` |
| `SHA_256_LENGTH` | SHA-256 输出长度 | `upg_check_boot_bin.c` |

---

## 六、重点安全审计文件

以下文件建议进行深度安全审计：

| 文件路径 | 安全领域 | 审计优先级 |
|---------|---------|----------|
| `hi3861v100/sdk_liteos/boot/flashboot/secure/crypto.c` | 启动加密核心 | 高 |
| `ws63v100/sdk/bootloader/commonboot/src/secure_verify_boot.c` | 安全启动验证 | 高 |
| `common/platform/wifi/hi3881v100/driver/mac/hmac/hmac_crypto_tkip.c` | WiFi 加密 | 高 |
| `hi3861v100/sdk_liteos/platform/system/upg/upg_check_secure.c` | OTA 安全升级 | 高 |
| `ws63v100/sdk/middleware/utils/update/common/upg_verify.c` | OTA 签名验证 | 高 |
| `hi3861v100/sdk_liteos/boot/flashboot/upg/boot_upg_check_secure.c` | RSA 签名验证 | 高 |
| `hi3516dv300/sdk_linux/usr/sensor/sony_imx415/imx415_sensor_ctl.c` | 传感器接口 | 中 |
| `hi3751v350/sdk_linux/source/msp/drv/fm11nt081d/fm11nt081d.c` | NFC 设备驱动 | 中 |

---

## 七、安全开发建议

### 7.1 输入验证

```c
// 建议: 添加输入长度校验
if (input_len > MAX_ALLOWED) {
    return ERROR_INVALID_PARAM;
}

// 代码证据: common/platform/wifi/hi3881v100/driver/oal/oal_net.c:36
if (ptr == NULL) {
    return HDF_ERR_INVALID_PARAM;
}
```

### 7.2 安全字符串操作

```c
// 建议: 使用安全字符串函数
snprintf(dest, dest_size, "%s", src);
// 避免使用 strcpy, strcat 等不安全函数
```

### 7.3 内存安全

```c
// 建议: 使用已初始化内存分配
// 代码证据: common/platform/wifi/hi3881v100/driver/hcc/hcc_host.c:1745
uint8_t *buf = (uint8_t *)oal_memalloc(len);
if (buf == NULL) {
    return HDF_ERR_MALLOC_FAIL;
}
// 使用后释放
// 代码证据: common/platform/wifi/hi3881v100/driver/hcc/hcc_host.c:1751
oal_free(buf);
```

### 7.4 错误处理

```c
// 建议: 敏感信息脱敏
// 代码证据: common/platform/pin/pin_hi35xx.c:58
return HDF_ERR_MALLOC_FAIL;
// 避免输出敏感数据
```

---

## 八、相关文档

- 构建系统: [07_Build_System.md](07_Build_System.md)
- 编译产物: [08_Products.md](08_Products.md)
- SDK 架构: [06_SDK_Architecture.md](06_SDK_Architecture.md)
- 代码证据库: [_work/NOTES.md](_work/NOTES.md)

---

**文档版本**: 2.0  
**最后更新**: 2026-02-07  
**证据来源**: Phase 1 代码侦查与证据收集（4 个并行探索任务，50+ 条安全关键代码路径）

# 00 - 项目概览

## 目的与适用范围

**目的**: 帮助开发者快速理解 `sys_installer_lite` 组件的定位、边界和核心能力。

**适用范围**: 
- 需要集成 OTA 升级功能的应用开发者
- 需要适配新芯片的平台开发者
- 需要评估安全风险的安全工程师

---

## 项目定位

`sys_installer_lite` 是 OpenHarmony **Update 子系统** 的核心组件，为轻量级设备（mini/small 系统）提供 **OTA（Over The Air）远程升级** 能力。

### 核心定位

| 属性 | 说明 |
|------|------|
| **功能定位** | 设备端升级包接收、验证、写入、重启管理 |
| **系统定位** | 轻量级系统（mini/small/standard） |
| **架构定位** | Framework 层，向上提供 C API，向下依赖 HAL 层 |
| **升级模式** | 全量升级（当前仅支持全包） |
| **分区模式** | 支持 A/B 分区升级 |

### 支持的芯片平台

根据 README.md:31，当前已适配：
- Hi3518EV300
- Hi3516DV300
- Hi3861

其他芯片需要通过 HAL 层适配（见 [03_Inner_API.md](03_Inner_API.md)）。

---

## 核心能力

### 1. 升级包处理

- **包格式解析**: 支持自定义包格式结构（默认/自定义模式）
- **签名验证**: RSA2048 / RSA3072 数字签名验证
- **完整性校验**: SHA256 哈希校验每个组件
- **版本控制**: 防回滚机制（仅允许高版本升级）

### 2. 分区管理

- **A/B 分区**: 支持双分区无缝升级
- **多组件支持**: 最多支持 10 个分区组件（`OTA_MAX_PARTITION_NUM`, hota_updater.c:33）
- **元数据管理**: 升级状态、进度持久化

### 3. 状态管理

- **状态机**: 完整的 OTA 生命周期管理
  - `HOTA_NOT_INIT` → `HOTA_INITED` → `HOTA_TRANSPORT_INFO_DONE` → `HOTA_TRANSPORT_ALL_DONE`
  - 异常分支：`HOTA_FAILED` / `HOTA_CANCELED`

### 4. 安全特性

- **签名验证**: 强制公钥验证（RSA-PKCS1-v2.1 + SHA256）
- **版本防回滚**: 仅允许高版本升级
- **分区保护**: 分区数量限制、大小校验
- **状态保护**: 完成前禁止重启，失败支持回滚

---

## 运行环境

### 系统要求

| 要求项 | 说明 |
|--------|------|
| **操作系统** | OpenHarmony mini/small/standard |
| **编程语言** | C 语言（C99） |
| **编译工具** | GN + Ninja（OpenHarmony 标准构建） |
| **依赖库** | mbedtls（签名验证）、bounds_checking_function（安全函数） |
| **内核** | LiteOS-M / Linux / 其他 POSIX 兼容内核 |

### 依赖组件（bundle.json:53-60）

```json
{
  "components": ["init", "utils_lite"],
  "third_party": ["bounds_checking_function", "mbedtls"]
}
```

### 硬件要求

- 支持 Flash 分区读写
- 支持 A/B 分区（可选）
- 支持重启后状态保持（bootloader 配合）

---

## 关键概念

### 升级包（Upgrade Package）

OTA 升级的数据单元，包含：

1. **信息组件（Info Component）**: 包元数据、组件表、签名
2. **数据组件（Data Components）**: 各分区的二进制镜像数据

结构详见 [01_Architecture.md#升级包格式](01_Architecture.md)。

### 组件（Component）

升级包中的最小可更新单元，对应一个分区镜像：

```c
// hota_partition.h:47-78
typedef struct {
    unsigned char addr[PARTITION_NAME_LENGTH];  // 分区名，如 "kernel"
    unsigned short id;                          // 组件 ID
    unsigned char type;                         // 组件类型
    unsigned char operType;                     // 0:升级 1:删除
    unsigned char isDiff;                       // 是否差分升级
    unsigned char version[COMP_VERSION_LENGTH]; // 版本号
    unsigned int length;                        // 数据长度
    unsigned int destLength;                    // 解压后长度
    unsigned char shaData[HASH_LENGTH];         // SHA256 哈希
} ComponentInfo;
```

### A/B 分区

双分区升级机制：
- **运行分区（Running）**: 当前正在运行的系统
- **升级分区（Update）**: 接收新数据的分区
- **重启切换**: 升级完成后切换启动分区

API：`HotaGetUpdateIndex()` 获取当前应升级的分区索引（1=A, 2=B）。

### HAL（Hardware Abstraction Layer）

厂商适配层接口，定义在 `hals/hal_hota_board.h`：
- 分区读写（`HotaHalWrite` / `HotaHalRead`）
- 重启控制（`HotaHalRestart`）
- 公钥获取（`HotaHalGetPubKey`）
- 版本检查（`HotaHalCheckVersionValid`）

**注意**: HAL 实现由芯片厂商提供，本仓库仅包含接口定义。

---

## 目录结构与模块职责

```
/base/update/sys_installer_lite
├── interfaces/kits/           # 【对外接口层】
│   ├── hota_updater.h         #   OTA 升级 API（15个函数）
│   └── hota_partition.h       #   数据结构定义
├── frameworks/source/         # 【框架实现层】
│   ├── updater/               #   升级逻辑模块
│   │   └── hota_updater.c     #     升级核心实现（704行）
│   └── verify/                #   签名验证模块
│       ├── app_rsa.c          #     RSA 签名验证（mbedtls封装）
│       ├── app_sha256.c       #     SHA256 哈希计算
│       └── hota_verify.c      #     验证入口与哈希管理
├── hals/                      # 【HAL接口定义层】
│   └── hal_hota_board.h       #   硬件抽象接口（19个函数）
└── frameworks/BUILD.gn        # 【构建配置】GN构建脚本
```

---

## 快速开始

### 1. 包含头文件

```c
#include "hota_updater.h"
```

### 2. 初始化 OTA

```c
int ret = HotaInit(NULL, NULL);  // 无回调简化版
if (ret != 0) {
    // 初始化失败处理
}
```

### 3. 写入升级数据

```c
unsigned char buffer[1024];
// ... 填充数据 ...
int ret = HotaWrite(buffer, offset, sizeof(buffer));
if (ret != 0) {
    // 写入失败处理
}
```

### 4. 完成升级并重启

```c
ret = HotaSetBootSettings();  // 设置启动参数
if (ret == 0) {
    HotaRestart();  // 重启系统
}
```

完整 API 说明见 [02_Public_API.md](02_Public_API.md)。

---

## 相关跳转

- [项目架构详解 → 01_Architecture.md](01_Architecture.md)
- [对外 API 完整清单 → 02_Public_API.md](02_Public_API.md)
- [HAL 接口适配指南 → 03_Inner_API.md](03_Inner_API.md)
- [安全风险分析 → 05_Security.md](05_Security.md)

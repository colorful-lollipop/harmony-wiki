# HAL（硬件抽象层）实现说明

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库中的 HAL 实现、接口定义和适配模式
- **适用范围**：vendor/hihope 仓库中的 hals/ 目录
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库仅包含基础 HAL 实现（sys_param, token）
  - 标准产品的 HAL（audio, codec）主要为配置文件，实际实现在 OpenHarmony 框架中
  - HAL 接口通过 HDF 层暴露给上层系统

## HAL 概述

### HAL 定义

HAL（Hardware Abstraction Layer，硬件抽象层）是 OpenHarmony 架构中连接系统框架与厂商硬件的中间层。

**职责**：
- 屏蔽底层硬件差异
- 为上层框架提供统一接口
- 实现平台相关的硬件驱动适配

**在 vendor 仓库中的位置**：
- 标准产品：`{product}/hals/` 目录
- IoT 产品：`{product}/hals/utils/` 目录

## HAL 实现分类

### 1. Utils HAL（实用 HAL）

**适用产品**：
- neptune_iotlink_demo
- nearlink_dk_3863

**位置**：`{product}/hals/utils/`

#### 1.1 Token HAL（令牌管理）

**职责**：提供设备令牌（Token）和访问密钥（Access Key）的读写接口。

**文件结构**：
```
hals/utils/token/
├── hal_token.c         # Token HAL 实现
├── hal_token.h         # Token HAL 头文件
└── BUILD.gn           # 构建文件
```

**接口函数**（证据：`nearlink_dk_3863/hals/utils/token/hal_token.c`）：

| 函数名 | 签名 | 参数 | 返回值 | 说明 |
|---------|--------|------|--------|------|
| OEMReadToken | `static int OEMReadToken(char *token, unsigned int len)` | token, len | EC_SUCCESS | OEM 读取令牌（需厂商实现） |
| OEMWriteToken | `static int OEMWriteToken(const char *token, unsigned int len)` | token, len | EC_SUCCESS | OEM 写入令牌（需厂商实现） |
| OEMGetAcKey | `static int OEMGetAcKey(char *acKey, unsigned int len)` | acKey, len | EC_SUCCESS | OEM 获取访问密钥（需厂商实现） |
| OEMGetProdId | `static int OEMGetProdId(char *productId, unsigned int len)` | productId, len | EC_SUCCESS | OEM 获取产品 ID（需厂商实现） |

**实现状态**：
- ⚠️ 所有函数目前为 stub 实现（仅返回 EC_SUCCESS）
- 📝 TODO: 厂商需要在实际硬件上实现令牌存储和读取逻辑

**包含头文件**：
```c
#include "hal_token.h"
#include "ohos_errno.h"
#include "ohos_types.h"
```

#### 1.2 System Parameter HAL（系统参数 HAL）

**职责**：提供系统参数（如设备序列号）的读取接口。

**文件结构**：
```
hals/utils/sys_param/
├── hal_sys_param.c    # 系统参数 HAL 实现
├── vendor.para         # 系统参数配置
└── BUILD.gn           # 构建文件
```

**接口函数**（证据：`nearlink_dk_3863/hals/utils/sys_param/hal_sys_param.c`）：

| 函数名 | 签名 | 参数 | 返回值 | 说明 |
|---------|--------|------|--------|------|
| HalGetSerial | `const char *HalGetSerial(void)` | 无 | char* | 读取设备序列号（UDID） |

**实现细节**（证据：`hal_sys_param.c:18-21`）：
```c
#define UDID_SIZE 20
static uint8_t udid[UDID_SIZE];

const char *HalGetSerial(void)
{
    uapi_soc_read_id(udid, UDID_SIZE);  // 读取 SoC eFuse 中的 UDID
    return (const char *) udid;
}
```

**包含头文件**：
```c
#include "hal_sys_param.h"
#include "efuse.h"
```

**相关配置**：
- `vendor.para`：系统参数配置文件

### 2. Audio HAL（音频 HAL）

**适用产品**：所有标准产品（9 个）

**位置**：`{product}/hals/audio/`

**职责**：提供音频采集、播放、音效处理和编解码的 HAL 接口。

**文件结构**：
```
hals/audio/
├── BUILD.gn                      # 音频 HAL 构建文件
├── audio_adapter.json             # 音频适配器配置
├── audio_effect.json              # 音效配置
├── audio_paths.json               # 音频路径配置
├── product.gni                   # 产品特定配置
└── config/                       # 音频策略配置
    ├── arm/
    │   └── audio_policy_config.xml
    └── arm64/
        └── audio_policy_config.xml
```

**配置文件说明**：

#### 2.1 audio_adapter.json

**职责**：定义音频适配器配置，如 ALSA 设备路径。

**证据**：`rk3568/hals/audio/BUILD.gn:16-23`（ohos_prebuilt_etc target）

#### 2.2 audio_effect.json

**职责**：定义支持的音效类型和参数。

**证据**：`rk3568/hals/audio/BUILD.gn:34-40`（ohos_prebuilt_etc target）

#### 2.3 audio_paths.json

**职责**：定义音频设备节点路径。

**证据**：`rk3568/hals/audio/BUILD.gn:43-49`（ohos_prebuilt_etc target）

#### 2.4 audio_policy_config.xml

**职责**：定义音频策略（如音量、路由、焦点）。

**证据**：`rk3568/hals/audio/config/arm/audio_policy_config.xml`（XML 格式的策略配置）

**HAL 接口说明**：

| 接口类别 | 说明 | 实现位置 |
|----------|------|----------|
| **音频流接口** | 提供音频流（PCM）采集和播放 | OpenHarmony 框架 |
| **音频控制接口** | 提供音量、路由、静音等控制 | OpenHarmony 框架 |
| **音效接口** | 提供音效处理（EQ、混响等） | OpenHarmony 框架 |
| **编解码接口** | 提供音频编解码能力 | HDF Codec 驱动 |

**实际实现**：
- ⚠️ vendor_hihope 仓库中的 Audio HAL 主要为配置文件
- ✅ 实际 Audio HAL 实现在 OpenHarmony 框架中（foundation/multimedia/audio_framework）
- ✅ vendor 仓库负责提供平台特定的配置和适配

### 3. Codec HAL（编解码器 HAL）

**适用产品**：所有标准产品（9 个）

**位置**：`{product}/hals/codec/`

**职责**：提供音频、视频编解码的 HAL 接口。

**文件结构**：
```
hals/codec/
├── BUILD.gn      # 编解码器 HAL 构建文件
└── product.gni    # 编解码器产品配置
```

**HAL 接口说明**：

| 接口类别 | 说明 | 实现位置 |
|----------|------|----------|
| **音频编解码接口** | 提供音频编解码（AAC, MP3, FLAC 等） | HDF Codec 驱动 |
| **视频编解码接口** | 提供视频编解码（H.264, H.265, VP8 等） | HDF Codec 驱动 |
| **图像编解码接口** | 提供图像编解码（JPEG, PNG 等） | HDF Codec 驱动 |

**实际实现**：
- ⚠️ vendor_hihope 仓库中的 Codec HAL 主要为配置文件
- ✅ 实际 Codec HAL 实现在 OpenHarmony 框架中（foundation/multimedia/camera_framework）
- ✅ vendor 仓库负责提供平台特定的配置和适配

## HAL 实现模式

### 1. 实现位置

| HAL 类型 | vendor 仓库实现 | 实际实现位置 | 说明 |
|---------|----------------|----------|------|
| **Utils HAL** | ✅ 有实现 | - | neptune_iotlink_demo, nearlink_dk_3863 |
| **Audio HAL** | ❌ 仅配置 | OpenHarmony 框架 | vendor 提供配置文件 |
| **Codec HAL** | ❌ 仅配置 | OpenHarmony 框架 | vendor 提供配置文件 |

### 2. 实现方式

**IoT 产品（Utils HAL）**：
- ✅ 提供 C 语言实现（.c 文件）
- ✅ 提供头文件定义接口（.h 文件）
- ✅ 使用 BUILD.gn 构建为静态或共享库
- ⚠️ 当前为 stub 实现，需要厂商完善

**标准产品（Audio/Codec HAL）**：
- ✅ 仅提供 JSON/XML 配置文件
- ✅ 实际 HAL 逻辑在 OpenHarmony 框架中
- ✅ vendor 仓库负责平台适配层

### 3. HAL 与上层框架关系

```
┌─────────────────────────────────────┐
│  应用层（ArkTS/C++）       │  [OpenHarmony 框架]
└────────────┬────────────────────┘
             │
        ┌──────────▼───────────┐
        │  框架层           │  [OpenHarmony 框架]
        │  (Foundation 框架)     │
        │                       │
        └──────────┬───────────┘
             │
        ┌──────────▼───────────┐
        │   HAL 接口层         │  [OpenHarmony 框架]
        │  （定义接口）          │
        │                       │
        └──────────┬───────────┘
             │
        ┌──────────▼───────────┐
        │   HAL 实现层         │  [vendor 仓库]
        │  （实际实现）          │
        │                       │
        └──────────┬───────────┘
             │
        ┌──────────▼───────────┐
        │    HDF 驱动层         │  [OpenHarmony HDF]
        │  （底层硬件访问）      │
        └──────────────────────────┘
```

## HAL 开发指南

### IoT 产品 HAL 开发

对于 neptune_iotlink_demo 和 nearlink_dk_3863 等 IoT 产品：

1. **确定 HAL 接口需求**
   - 参考 OpenHarmony HAL 接口规范
   - 确定需要实现的函数和回调

2. **在 hals/ 目录创建实现**
   ```
   {product}/hals/
   ├── my_hal/
   │   ├── my_hal.c
   │   ├── my_hal.h
   │   └── BUILD.gn
   └── ...
   ```

3. **编写 BUILD.gn**
   ```gn
   import("//build/ohos.gni")

   ohos_shared_library("my_hal") {
     sources = ["my_hal.c"]
     include_dirs = ["include"]
     external_deps = [
       "hdf_core:libhdf",
       "hilog:libhilog",
     ]
     subsystem_name = "product_{product}"
     part_name = "product_{product}"
   }
   ```

4. **测试 HAL 接口**
   - 编写单元测试
   - 使用 OpenHarmony 测试框架
   - 在实际硬件上验证功能

### 标准产品 HAL 配置

对于标准产品（rk3568, wearable 等）：

1. **修改 HAL 配置文件**
   - 更新 `hals/audio/audio_adapter.json`
   - 更新 `hals/audio/audio_effect.json`
   - 更新 `hals/audio/audio_paths.json`

2. **修改音频策略**
   - 编辑 `hals/audio/config/arm/audio_policy_config.xml`
   - 编辑 `hals/audio/config/arm64/audio_policy_config.xml`

3. **验证配置**
   - 编译系统
   - 测试音频功能
   - 检查日志输出

## HAL 调试技巧

### 1. 日志调试

使用 OpenHarmony 日志系统查看 HAL 行为：

```c
#include "hilog/log.h"

void MyHalFunction() {
    OH_LOG_Print(LOG_APP, "My HAL function called");
    // HAL 逻辑
    OH_LOG_Print(LOG_DEBUG, "Parameter: %{public}d", param_value);
}
```

### 2. HDF 调试

查看 HDF 驱动加载和服务注册：

```bash
# 查看 HDF 服务列表
hdc shell
hdf -h

# 查看特定服务信息
hdf -s service_name

# 查看 HDF 日志
hdc shell hilog -T HDF
```

### 3. 常见问题

| 问题 | 可能原因 | 解决方法 |
|------|---------|---------|
| HAL 加载失败 | .so 文件不存在或路径错误 | 检查 `BUILD.gn` 和配置文件 |
| 服务未注册 | HDF 配置错误或 priority 不对 | 检查 `device_info.hcs` 中的服务定义 |
| 权限被拒绝 | 进程权限不足 | 检查 `high_privilege_process_list.json` |
| 配置未生效 | 配置文件格式错误 | 使用 GN 格式化工具检查 JSON/XML |

## 相关跳转

- [HDF 配置详解](04_HDF_Configuration.md)
- [目录结构详解](02_Directory_Structure.md)
- [返回 Wiki 首页](SUMMARY.md)

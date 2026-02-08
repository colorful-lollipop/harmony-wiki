# 项目概览

> 最后更新：2026-02-07
> 版本：v3.0.0

## 1.1 项目定位

### 定义

**Device Security Level Management (DSLM)** 是 OpenHarmony 安全子系统的核心组件，负责管理设备的系统安全能力等级，并为分布式设备间的数据流转提供安全决策依据。

### 核心职责

DSL M 模块承担以下核心职责：

| 职责 | 描述 | 证据 |
|------|------|------|
| **安全等级定义** | 定义 SL1-SL5 五级安全能力体系 | `bundle.json:15` 定义组件名称为 `device_security_level` |
| **等级查询** | 提供查询设备安全等级的 SDK 接口 | `interfaces/inner_api/include/device_security_info.h:42-68` |
| **凭证验证** | 验证远端设备的身份凭证 | `oem_property/common/dslm_credential.c:45-67` |
| **信任决策** | 为分布式数据流转提供安全决策 | `README.md:29` "The security level of each device in a Super Device provides the decision-making criteria" |

###解决的问题

在 OpenHarmony 分布式技术场景中，多个设备可以融合形成"超级虚拟终端"。当处理或流转各类用户数据时，需要确保各个节点不因安全能力薄弱而成为整个系统的薄弱点。DSL M 模块通过以下机制解决这一问题：

```
┌─────────────────────────────────────────────────────────────┐
│                    超级虚拟终端                              │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│  │ 设备 A  │◄─►│ 设备 B  │◄─►│ 设备 C  │◄─►│ 设备 D  │   │
│  │  SL3    │   │  SL5    │   │  SL2    │   │  SL4    │   │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘   │
│       │              │              │              │        │
│       └──────────────┼──────────────┼──────────────┘        │
│                      │                                  │
│              ┌───────▼───────┐                          │
│              │  DSLM 模块    │                          │
│              │  安全决策中心  │                          │
│              └───────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.2 安全等级体系

### SL1-SL5 分级定义

OpenHarmony 参考业界权威的安全分级模型，结合实际业务场景和设备分类，将设备安全能力划分为五个等级：

| 等级 | 名称 | 核心特征 | 典型设备 |
|------|------|----------|----------|
| **SL1** | 基础安全 | 软件完整性保护、消除常见错误 | 低端物联网设备 |
| **SL2** | 自主访问控制 | DAC、基础抗渗透、隔离环境 | 智能家居设备 |
| **SL3** | 强制访问控制 | MAC、关键元素保护、抗漏洞利用 | 智能手机、平板 |
| **SL4** | 精简可信基 | 简化 TCB、防篡改、充分仲裁 | 高端智能终端 |
| **SL5** | 形式化验证 | 核心模块形式化验证、防物理攻击 | 高安全敏感场景 |

### 等级包含关系

在 OpenHarmony 生态体系中，**高一级的设备安全能力默认包含低一级的所有能力**：

```
SL5 ─────────────────────────────────────────►
     包含 SL4 + SL3 + SL2 + SL1 全部能力

SL4 ───────────────────────────────────────►
     包含 SL3 + SL2 + SL1 全部能力

SL3 ────────────────────────────────────►
     包含 SL2 + SL1 全部能力

SL2 ────────────────────────────────►
     包含 SL1 全部能力

SL1 ───────►
     基础安全能力
```

### 等级评估依据

设备安全等级取决于以下系统安全能力：

| 能力域 | 具体能力 | SL 要求 |
|--------|----------|---------|
| **启动信任根 (RoT)** | 安全启动链、镜像签名验证 | SL3+ 必须 |
| **存储信任根** | 安全存储、密钥保护 | SL2+ 必须 |
| **计算信任根** | TEE/安全 enclave | SL4+ 必须 |
| **完整性保护** | 系统完整性监控 | SL1+ 必须 |
| **访问控制** | DAC + MAC | SL2+ 必须 DAC，SL3+ 必须 MAC |
| **抗渗透能力** | 漏洞利用缓解 | SL2+ 必须基础能力 |
| **形式化验证** | 核心模块形式化证明 | SL5 必须 |

**证据**：`README.md:9-10` "The security level of an OpenHarmony device depends on the system security capabilities of the device. The OpenHarmony system security capabilities are based on the root of trust (RoT) for boot, RoT for storage, and RoT for compute on the hardware."

---

## 1.3 核心能力

### SDK 能力

DSL M 对外提供一套 C 原生接口，供系统级组件查询设备安全等级：

| 接口 | 类型 | 功能描述 |
|------|------|----------|
| `RequestDeviceSecurityInfo()` | 同步 | 同步获取设备安全等级信息 |
| `RequestDeviceSecurityInfoAsync()` | 异步 | 异步获取设备安全等级信息 |
| `FreeDeviceSecurityInfo()` | 释放 | 释放安全等级信息对象 |
| `GetDeviceSecurityLevelValue()` | 提取 | 从信息对象中提取等级值 |

**证据**：`interfaces/inner_api/include/device_security_info.h:29-68`

### 服务能力

| 能力 | 描述 | 证据 |
|------|------|------|
| **SA 注册** | 作为 System Ability (ID: 3511) 提供服务 | `services/sa/standard/dslm_service.cpp:38` |
| **IPC 通信** | 支持跨进程安全等级查询 | `services/sa/standard/dslm_service.cpp:126-144` |
| **设备发现** | 感知设备上下线状态 | `services/dslm/dslm_core_process.c:183` |
| **凭证管理** | 生成和验证设备凭证 | `oem_property/ohos/common/dslm_ohos_verify.c` |

### 安全能力

| 能力 | 描述 | 证据 |
|------|------|------|
| **凭证签名** | 基于 ECDSA 的凭证签名验证 | `oem_property/common/dslm_credential_utils.c:522-577` |
| **证书链验证** | 多级证书链验证 | `oem_property/ohos/common/external_interface_adapter.c:107-161` |
| **挑战响应** | 防止重放攻击的挑战机制 | `oem_property/ohos/common/dslm_ohos_verify.c:68-167` |
| **硬件绑定** | HUKS 硬件密钥 attestation | `oem_property/ohos/common/hks_adapter.c` |

---

## 1.4 运行环境

### 系统形态支持

DSL M 模块支持 OpenHarmony 的三种系统形态：

| 系统形态 | 架构 | 实现语言 | IPC 机制 |
|----------|------|----------|----------|
| **Standard** | 富设备 (L2) | C++ | Binder IPC |
| **Small** | 轻设备 (L1) | C | Binder/Socket |
| **Mini** | 微设备 (L0) | C | 内部调用 (无 IPC) |

**证据**：`bundle.json:26-30`
```json
"adapted_system_type": [
    "standard",
    "small", 
    "mini"
]
```

### 依赖组件

DSL M 运行依赖以下 OpenHarmony 组件：

| 依赖类型 | 组件 | 用途 |
|----------|------|------|
| **安全组件** | HUKS | 密钥管理和硬件 attestation |
| **安全组件** | Device Auth | 设备认证和公钥管理 |
| **通信组件** | DSoftBus | 分布式设备通信 |
| **通信组件** | Device Manager | 设备发现和管理 |
| **系统组件** | SAFWK/SAMGR | System Ability 框架 |
| **系统组件** | IPC | 进程间通信 |

**证据**：`bundle.json:33-49`
```json
"deps": {
    "components": [
        "cJSON", "c_utils", "device_auth", "device_manager",
        "dsoftbus", "hilog", "hisysevent", "hitrace",
        "huks", "ipc", "safwk", "samgr", "openssl", "access_token"
    ]
}
```

### 资源占用

| 资源类型 | 大小 | 说明 |
|----------|------|------|
| **ROM** | 200KB | 代码和只读数据 |
| **RAM** | 2500KB | 运行时内存 |

**证据**：`bundle.json:31-32`

---

## 1.5 快速开始

### 添加编译依赖

在目标模块的 `BUILD.gn` 中添加依赖：

```gn
external_deps += [ "device_security_level:dslm_sdk" ]
```

### 包含头文件

```cpp
#include "device_security_defines.h"  // 数据结构和错误码定义
#include "device_security_info.h"     // API 函数声明
```

### 同步调用示例

```cpp
#include <stdio.h>
#include "device_security_defines.h"
#include "device_security_info.h"

void CheckDeviceSecurityLevel(const DeviceIdentify *device)
{
    DeviceSecurityInfo *info = NULL;
    
    // 同步获取设备安全等级信息
    int32_t ret = RequestDeviceSecurityInfo(device, NULL, &info);
    if (ret != SUCCESS) {
        printf("获取安全等级失败，错误码: %d\n", ret);
        return;
    }
    
    // 提取安全等级值
    int32_t level = 0;
    ret = GetDeviceSecurityLevelValue(info, &level);
    if (ret != SUCCESS) {
        printf("提取安全等级失败，错误码: %d\n", ret);
        FreeDeviceSecurityInfo(info);
        return;
    }
    
    printf("设备安全等级: SL%d\n", level);
    
    // 检查是否满足安全要求
    if (level >= 3) {
        printf("设备安全等级满足要求 (SL3+)\n");
    } else {
        printf("设备安全等级不足，需要 SL3 或更高\n");
    }
    
    // 释放资源
    FreeDeviceSecurityInfo(info);
}
```

### 异步调用示例

```cpp
#include <stdio.h>
#include "device_security_defines.h"
#include "device_security_info.h"

// 异步回调函数
void OnDeviceSecurityInfoReceived(const DeviceIdentify *identify, 
                                   DeviceSecurityInfo *info)
{
    int32_t level = 0;
    int32_t ret = GetDeviceSecurityLevelValue(info, &level);
    
    if (ret == SUCCESS) {
        printf("异步获取成功，设备安全等级: SL%d\n", level);
    } else {
        printf("异步获取失败，错误码: %d\n", ret);
    }
    
    // 必须释放资源
    FreeDeviceSecurityInfo(info);
}

void AsyncQueryExample(const DeviceIdentify *device)
{
    // 异步请求
    int32_t ret = RequestDeviceSecurityInfoAsync(device, NULL, 
                                                  OnDeviceSecurityInfoReceived);
    if (ret != SUCCESS) {
        printf("发起异步请求失败，错误码: %d\n", ret);
        return;
    }
    
    printf("异步请求已发起，等待回调通知...\n");
}
```

### 默认配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `challenge` | 0 | 默认挑战值，由服务生成 |
| `timeout` | 45 秒 | 超时时间，可设置 1-300 秒 |
| `extra` | 0 | 保留字段 |

---

## 1.6 版本信息

| 属性 | 值 |
|------|-----|
| **当前版本** | v3.0.0 |
| **主版本号** | 3 (`VERSION_MAJOR = 3U`) |
| **次版本号** | 0 (`VERSION_MINOR = 0U`) |
| **修订版本号** | 0 (`VERSION_PATCH = 0U`) |
| **版本格式** | `(major << 16) + (minor << 8) + patch` |

**证据**：`services/dslm/dslm_core_defines.h:34-36`

---

## 1.7 相关资源

### 代码仓库

- **主仓库**: `base/security/device_security_level`
- **代码路径**: `foundation/security/device_security_level`

### 关联组件

| 组件 | 关系 | 用途 |
|------|------|------|
| [Data Transfer Management](https://gitee.com/openharmony/security_dataclassification) | 上游 | 数据风险等级与设备安全等级映射 |
| [HUKS](https://gitee.com/openharmony/security_huks) | 依赖 | 硬件密钥服务 |
| [Device Authentication](https://gitee.com/openharmony/security_device_auth) | 依赖 | 设备认证服务 |
| [SELinux](https://gitee.com/openharmony/security_selinux) | 关联 | 强制访问控制 |

---

## 下一章

- [02_Architecture.md](./02_Architecture.md) - 架构与数据流
- [03_CodeMap.md](./03_CodeMap.md) - 目录结构与代码地图
- [04_Interface.md](./04_Interface.md) - 对外接口文档

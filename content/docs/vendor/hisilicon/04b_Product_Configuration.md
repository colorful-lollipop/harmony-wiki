# 产品配置详解

本文档详细说明 `config.json` 产品配置文件的结构和字段含义。

---

## 4.b.1 配置概述

**格式**：JSON

**位置**：`*/config.json`

**用途**：定义产品的基本信息、子系统配置、组件列表等。

---

## 4.b.2 完整配置示例

### LiteOS-M 产品配置

**文件**：`hispark_pegasus/config.json` [证据]

```json
{
    "product_name": "wifiiot_hispark_pegasus",        // [1] 产品名称
    "type": "mini",                                   // [2] 系统类型
    "version": "3.0",                                 // [3] 版本
    "ohos_version": "OpenHarmony 1.0",               // [4] OpenHarmony 版本
    "device_company": "hisilicon",                   // [5] 芯片厂商
    "device_build_path": "device/board/hisilicon/hispark_pegasus",
    "board": "hispark_pegasus",                      // [6] 开发板名称
    "kernel_type": "liteos_m",                       // [7] 内核类型
    "kernel_is_prebuilt": true,                      // [8] 是否预编译内核
    "kernel_version": "",
    "subsystems": [                                  // [9] 子系统列表
      {
        "subsystem": "applications",
        "components": [
          { "component": "wifi_iot_sample_app", "features":[] }
        ]
      },
      {
        "subsystem": "iothardware",
        "components": [
          { "component": "peripheral", "features":[] }
        ]
      },
      {
        "subsystem": "hiviewdfx",
        "components": [
          { "component": "hilog_lite", "features":[] },
          { "component": "hievent_lite", "features":[] },
          { "component": "blackbox_lite", "features":[] },
          { "component": "hidumper_lite", "features":[] }
        ]
      },
      {
        "subsystem": "systemabilitymgr",
        "components": [
          { "component": "samgr_lite", "features":[] }
        ]
      },
      {
        "subsystem": "security",
        "components": [
          { "component": "device_auth", "features":[] },
          { "component": "huks", "features": [...] }
        ]
      },
      {
        "subsystem": "thirdparty",
        "components": [
          { "component": "mbedtls", "features": [...] }
        ]
      },
      {
        "subsystem": "startup",
        "components": [
          { "component": "bootstrap_lite", "features":[] },
          { "component": "init", "features": [...] }
        ]
      },
      {
        "subsystem": "communication",
        "components": [
          { "component": "wifi_lite", "features":[] },
          { "component": "dsoftbus", "features":[] },
          { "component": "wifi_aware", "features":[]}
        ]
      },
      {
        "subsystem": "updater",
        "components": [
          { "component": "sys_installer_lite", "features":[] }
        ]
      },
      {
        "subsystem": "commonlibrary",
        "components": [
          { "component": "utils_lite", "features":[ ... ] }
        ]
      },
      {
       "subsystem": "xts",
       "components": [
         { "component": "acts", "features": [...] },
         { "component": "tools", "features":[] },
         { "component": "device_attest_lite", "features":[] }
       ]
      },
      {
        "subsystem": "developtools",
        "components": [
          { "component": "syscap_codec", "features":[] }
        ]
      }
    ],
    "third_party_dir": "//device/soc/hisilicon/hi3861v100/sdk_liteos/third_party",
    "product_adapter_dir": "//vendor/hisilicon/hispark_pegasus/hals"
}
```

### 标准系统产品配置

**文件**：`hispark_taurus_standard/config.json` [证据]

```json
{
  "product_name": "hispark_taurus_standard",
  "device_company": "hisilicon",
  "device_build_path": "device/board/hisilicon/hispark_taurus/linux",
  "target_cpu": "arm",
  "type": "standard",
  "version": "3.0",
  "board": "hispark_taurus",
  "inherit": [ "productdefine/common/base/standard_system.json",
               "productdefine/common/inherit/ipcamera.json"
  ],
  "enable_ramdisk": true,
  "subsystems": [
    {
      "subsystem": "hisilicon_products",
      "components": [
        {
          "component": "hisilicon_products",
          "features": []
        }
      ]
    },
    {
      "subsystem": "arkui",
      "components": [
        {
          "component": "ui_lite",
          "features": []
        }
      ]
    },
    {
      "subsystem": "security",
      "components": [
        {
          "component": "certificate_manager"
        }
      ]
    },
    {
      "subsystem": "hdf",
      "components": [
        {
          "component": "drivers_peripheral_wlan",
          "features": [
            "drivers_peripheral_wlan_feature_enable_HDF_NL80211 = true",
            "drivers_peripheral_wlan_feature_enable_HDF_UT = true"
          ]
        }
      ]
    }
  ]
}
```

---

## 4.b.3 字段说明

### 4.b.3.1 必填字段

| 字段 | 类型 | 说明 |
|-----|------|------|
| product_name | string | 产品唯一标识 |
| type | string | 系统类型 (mini/light/standard/linux) |
| device_company | string | 芯片厂商 |
| board | string | 开发板名称 |
| subsystems | array | 子系统配置列表 |

### 4.b.3.2 可选字段

| 字段 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| version | string | "1.0" | 产品版本 |
| ohos_version | string | - | OpenHarmony 版本 |
| device_build_path | string | - | 设备构建路径 |
| kernel_type | string | - | 内核类型 |
| kernel_is_prebuilt | boolean | false | 是否预编译内核 |
| kernel_version | string | - | 内核版本 |
| target_cpu | string | - | 目标 CPU |
| inherit | array | [] | 继承的配置 |
| enable_ramdisk | boolean | false | 启用 RAMDISK |
| third_party_dir | string | - | 第三方目录 |
| product_adapter_dir | string | - | 产品适配目录 |

---

## 4.b.3.3 系统类型

| type 值 | 内核 | 说明 |
|--------|------|------|
| mini | LiteOS-M | 轻量级 IoT 系统 |
| light | LiteOS | 轻量级设备系统 |
| standard | Linux | 标准系统 |
| linux | Linux | Linux 系统 |

---

## 4.b.3.4 子系统配置

```json
{
  "subsystem": "subsystem_name",       // 子系统名称
  "components": [
    {
      "component": "component_name",    // 组件名称
      "features": [                    // 特性列表（可选）
        "feature1 = value1",
        "feature2 = value2"
      ]
    }
  ]
}
```

---

## 4.b.4 常用子系统

| 子系统 | 组件 | 说明 |
|-------|------|------|
| applications | wifi_iot_sample_app | 示例应用 |
| iothardware | peripheral | 外设支持 |
| hiviewdfx | hilog_lite, hievent_lite, ... | 调试日志 |
| security | device_auth, huks | 安全框架 |
| communication | wifi_lite, dsoftbus, wifi_aware | 通信能力 |
| startup | bootstrap_lite, init | 启动框架 |
| arkui | ui_lite | UI 框架 |
| hdf | drivers_peripheral_* | 驱动组件 |

---

## 4.b.5 相关文档

- [配置体系](./04_Configuration.md)
- [HDF 配置详解](./04a_HDF_Configuration.md)
- [产品系列](./03_Products.md)

# SystemUI 构建指南

## 1. 构建系统概述

### 1.1 构建工具

| 工具 | 版本/要求 | 用途 |
|------|-----------|------|
| **hvigor** | 最新版本 | 构建系统入口 |
| **Node.js** | 16+ | 运行时环境 |
| **OpenHarmony SDK** | 12 | 编译 SDK |
| **@ohos/hvigor-ohos-plugin** | - | HAP/HAR 编译插件 |

### 1.2 构建产物类型

| 类型 | 说明 | 扩展名 |
|------|------|--------|
| **HAP** | 可安装的应用包 | `.hap` |
| **HAR** | 共享模块 | `.har` |

### 1.3 配置文件

| 文件 | 用途 |
|------|------|
| `build-profile.json5` | 项目构建配置 |
| `hvigorfile.js` | hvigor 入口脚本 |
| `hvigor/hvigor-wrapper.js` | hvigor 包装器 |
| `module.json5` | 模块配置 |

**证据**: `build-profile.json5:1-198`

## 2. 构建配置详解

### 2.1 项目配置 (build-profile.json5)

```json5
{
  "app": {
    "products": [
      {
        "name": "default",           // 产品名称
        "signingConfig": "release",  // 签名配置
        "compileSdkVersion": 12,     // 编译 SDK 版本
        "compatibleSdkVersion": 12    // 兼容 SDK 版本
      }
    ]
  }
}
```

### 2.2 模块配置结构

每个模块在 `build-profile.json5` 中定义：

```json5
{
  "modules": [
    {
      "name": "phone_entry",           // 模块名
      "srcPath": "./entry/phone",      // 源码路径
      "targets": [
        {
          "name": "default",          // 目标配置
        }
      ]
    },
    {
      "name": "batterycomponent",      // HAR 模块
      "srcPath": "./features/batterycomponent"
    }
  ]
}
```

### 2.3 模块类型

| 类型 | 配置 | 编译产物 |
|------|------|----------|
| entry | `"type": "entry"` | HAP |
| feature | `"type": "feature"` | HAP |
| har | 默认或 `"type": "har"` | HAR |

**证据**: `entry/phone/src/main/module.json5:2-4`

```json5
{
  "module": {
    "name": "phone_entry",
    "type": "entry",
    "srcEntrance": "./ets/Application/AbilityStage.ts"
  }
}
```

## 3. 模块清单

### 3.1 entry 模块 (2 个)

| 模块名 | 路径 | 类型 | 产物 |
|--------|------|------|------|
| `phone_entry` | `entry/phone/` | entry | phone_entry.hap |
| `pc_entry` | `entry/pc/` | entry | pc_entry.hap |

### 3.2 features 模块 (18 个 HAR)

| 模块名 | 路径 | 产物 |
|--------|------|------|
| `airplanecomponent` | `features/airplanecomponent/` | airplanecomponent.har |
| `autorotatecomponent` | `features/autorotatecomponent/` | autorotatecomponent.har |
| `batterycomponent` | `features/batterycomponent/` | batterycomponent.har |
| `bluetoothcomponent` | `features/bluetoothcomponent/` | bluetoothcomponent.har |
| `brightnesscomponent` | `features/brightnesscomponent/` | brightnesscomponent.har |
| `capsulecomponent` | `features/capsulecomponent/` | capsulecomponent.har |
| `clockcomponent` | `features/clockcomponent/` | clockcomponent.har |
| `controlcentercomponent` | `features/controlcentercomponent/` | controlcentercomponent.har |
| `locationcomponent` | `features/locationcomponent/` | locationcomponent.har |
| `managementcomponent` | `features/managementcomponent/` | managementcomponent.har |
| `navigationservice` | `features/navigationservice/` | navigationservice.har |
| `nfccomponent` | `features/nfccomponent/` | nfccomponent.har |
| `noticeitem` | `features/noticeitem/` | noticeitem.har |
| `ringmodecomponent` | `features/ringmodecomponent/` | ringmodecomponent.har |
| `signalcomponent` | `features/signalcomponent/` | signalcomponent.har |
| `statusbarcomponent` | `features/statusbarcomponent/` | statusbarcomponent.har |
| `volumecomponent` | `features/volumecomponent/` | volumecomponent.har |
| `volumepanelcomponent` | `features/volumepanelcomponent/` | volumepanelcomponent.har |
| `wificomponent` | `features/wificomponent/` | wificomponent.har |

### 3.3 product 模块 (9 个)

| 模块名 | 路径 | 类型 | 产物 |
|--------|------|------|------|
| `default_navigationBar` | `product/default/navigationBar/` | feature | default_navigationBar.hap |
| `default_notificationmanagement` | `product/default/notificationmanagement/` | feature | default_notificationmanagement.hap |
| `default_volumepanel` | `product/default/volumepanel/` | feature | default_volumepanel.hap |
| `default_dialog` | `product/default/dialog/` | feature | default_dialog.hap |
| `pc_controlpanel` | `product/pc/controlpanel/` | feature | pc_controlpanel.hap |
| `pc_notificationpanel` | `product/pc/notificationpanel/` | feature | pc_notificationpanel.hap |
| `pc_statusbar` | `product/pc/statusbar/` | feature | pc_statusbar.hap |
| `phone_dropdownpanel` | `product/phone/dropdownpanel/` | feature | phone_dropdownpanel.hap |
| `phone_statusbar` | `product/phone/statusbar/` | feature | phone_statusbar.hap |

### 3.4 common 模块

| 模块名 | 路径 | 类型 | 产物 |
|--------|------|------|------|
| `common` | `common/` | har | common.har |

## 4. 构建命令

### 4.1 常用命令

```bash
# 安装依赖
npm install

# 构建所有模块
hvigor assembleHap

# 构建手机版本
hvigor --product phone --target default assembleHap

# 构建 PC 版本
hvigor --product pc --target default assembleHap

# 构建 release 版本
hvigor assembleHap --release

# 清理构建产物
hvigor clean
```

### 4.2 构建输出目录

```
out/
└── [product]/
    └── [target]/
        └── default/
            ├── phone_entry/
            │   └── default/
            │       └── unsigned
            │           ├── phone_entry.hap
            │           └── phone_entryhapsign.hap
            └── pc_entry/
                └── default/
                    └── unsigned
                        ├── pc_entry.hap
                        └── pc_entryhapsign.hap
```

### 4.3 hvigorfile.js 配置

**证据**: `entry/phone/hvigorfile.js:1-18`

```javascript
/**
 * Copyright (c) 2022 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 */

module.exports = require('@ohos/hvigor-ohos-plugin').hapTasks
```

## 5. 模块依赖关系

### 5.1 依赖图

```
entry (HAP)
├── common (HAR)
├── product/phone/* (feature HAP)
│   ├── features/* (HAR)
│   └── common (HAR)
└── product/pc/* (feature HAP)
    ├── features/* (HAR)
    └── common (HAR)
```

### 5.2 典型依赖

| 消费者 | 提供者 | 依赖类型 |
|--------|--------|----------|
| entry/phone | common | HAR |
| entry/pc | common | HAR |
| product/* | features/* | HAR |
| product/* | common | HAR |
| features/* | common | HAR |

### 5.3 依赖声明

HAR 依赖在模块的 `module.json5` 中声明：

```json5
{
  "module": {
    "dependencies": [
      {
        "moduleName": "common",
        "bundleName": "com.ohos.systemui"
      }
    ]
  }
}
```

## 6. 产物说明

### 6.1 HAP 结构

```
[module].hap
├── config.json          # 模块配置
├── libs/                # native 库
├── resources/           # 资源文件
└── classes.ets          # ArkTS 字节码
```

### 6.2 HAR 结构

```
[module].har
├── index.ets            # 入口
├── libs/                # native 库
├── resources/           # 资源文件
└── classes.ets          # ArkTS 字节码
```

### 6.3 安装位置

| 产物类型 | 安装路径 |
|----------|----------|
| entry HAP | `/system/apps/com.ohos.systemui/` |
| feature HAP | `/system/apps/com.ohos.systemui/` |
| HAR | `/system/module/` |

## 7. 签名配置

### 7.1 签名文件

| 文件 | 位置 |
|------|------|
| 签名证书 | `signature/` 目录 |
| 签名配置 | `build-profile.json5` 中的 `signingConfig` |

### 7.2 签名配置示例

```json5
{
  "products": [
    {
      "name": "default",
      "signingConfig": "release"
    }
  ]
}
```

## 8. 常见问题

### 8.1 构建失败

| 问题 | 解决方案 |
|------|----------|
| Node.js 版本不兼容 | 使用 Node.js 16+ |
| SDK 未配置 | 设置 `OHOS_SDK` 环境变量 |
| 依赖缺失 | 运行 `npm install` |
| 内存不足 | 增加 hvigor 内存: `--hm-const-param=MAX_HEAP_SIZE=4096` |

### 8.2 构建产物验证

```bash
# 查看 HAP 信息
ha dump -h [module].hap

# 验证签名
ha sign-verify -i [module].hap
```

## 9. 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [架构设计](03_Architecture.md) - 系统架构图
- [内部 API](04_API_Inner.md) - 模块接口
- [安全评审](06_Security.md) - 安全考虑

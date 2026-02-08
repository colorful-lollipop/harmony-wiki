# 11_Module_Config - 模块配置

## 概述

本文档描述 Authentication Widget 的模块配置文件结构和配置项含义。

**证据**: `bundle.json`, `module.json`, `app.json`

## bundle.json

**证据**: `bundle.json` 行 1-33

模块清单文件，定义模块的元信息和构建配置。

### 完整配置

```json
{
  "name": "@ohos/auth_widget",
  "version": "4.0",
  "description": "User Authentication capability",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "applications/standard/auth_widget"
  },
  "dirs": {},
  "scripts": {},
  "component": {
    "name": "auth_widget",
    "subsystem": "applications",
    "features": ["auth_widget_enabled"],
    "adapted_system_type": ["standard"],
    "rom": "860KB",
    "ram": "0KB",
    "deps": {
      "components": [],
      "third_party": []
    },
    "build": {
      "sub_component": [
        "//applications/standard/auth_widget:auth_widget"
      ],
      "inner_kits": [],
      "test": []
    }
  }
}
```

### 配置项说明

| 配置项 | 值 | 描述 |
|--------|-----|------|
| `name` | `@ohos/auth_widget` | 模块名（用于包管理） |
| `version` | `4.0` | 模块版本 |
| `description` | `User Authentication capability` | 模块描述 |
| `license` | `Apache License 2.0` | 开源协议 |
| `publishAs` | `code-segment` | 发布类型 |
| `component.name` | `auth_widget` | 组件名 |
| `component.subsystem` | `applications` | 所属子系统 |
| `component.features` | `["auth_widget_enabled"]` | 功能开关 |
| `component.adapted_system_type` | `["standard"]` | 适配系统类型 |
| `component.rom` | `860KB` | ROM 占用 |
| `component.ram` | `0KB` | RAM 占用 |

## module.json

**证据**: `entry/src/main/module.json` 行 1-44

模块配置文件，定义模块的能力和组件。

### 完整配置

```json
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "userauthuiextensionability",
    "deviceTypes": ["default", "tablet"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "extensionAbilities": [
      {
        "name": "userauthuiextensionability",
        "srcEntry": "./ets/extensionability/UserAuthAbility.ts",
        "icon": "$media:app_icon",
        "label": "$string:EntryAbility_label",
        "type": "sysDialog/userAuth",
        "metadata": [
          {
            "name": "ohos.extension.servicetype",
            "value": "commonDialog"
          }
        ]
      }
    ],
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_PIN_AUTH"
      },
      {
        "name": "ohos.permission.ACCESS_BIOMETRIC"
      },
      {
        "name": "ohos.permission.SUPPORT_USER_AUTH"
      },
      {
        "name": "ohos.permission.PRIVACY_WINDOW"
      }
    ]
  }
}
```

### 配置项说明

| 配置项 | 值 | 描述 |
|--------|-----|------|
| `module.name` | `entry` | 模块名 |
| `module.type` | `entry` | 模块类型 |
| `module.mainElement` | `userauthuiextensionability` | 主入口组件 |
| `module.deviceTypes` | `["default", "tablet"]` | 支持设备类型 |
| `module.deliveryWithInstall` | `true` | 安装时交付 |
| `module.installationFree` | `false` | 非免安装 |
| `module.pages` | `$profile:main_pages` | 页面配置 |

### ExtensionAbility 配置

| 配置项 | 值 | 描述 |
|--------|-----|------|
| `name` | `userauthuiextensionability` | 能力名 |
| `srcEntry` | `./ets/extensionability/UserAuthAbility.ts` | 源码入口 |
| `type` | `sysDialog/userAuth` | 能力类型 |
| `metadata.name` | `ohos.extension.servicetype` | 元数据名 |
| `metadata.value` | `commonDialog` | 元数据值 |

### 权限配置

**证据**: `module.json` 行 29-42

| 权限名 | 描述 | 用途 |
|--------|------|------|
| `ohos.permission.ACCESS_PIN_AUTH` | PIN 认证访问 | PIN 输入和处理 |
| `ohos.permission.ACCESS_BIOMETRIC` | 生物特征认证访问 | 指纹/人脸认证 |
| `ohos.permission.SUPPORT_USER_AUTH` | 用户认证支持 | 认证框架集成 |
| `ohos.permission.PRIVACY_WINDOW` | 隐私窗口模式 | 隐私保护窗口 |

## app.json

**证据**: `AppScope/app.json` 行 1-15

应用配置文件，定义应用级别的基本信息。

```json
{
  "app": {
    "bundleName": "com.ohos.useriam.authwidget",
    "vendor": "example",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name",
    "minAPIVersion": 10,
    "targetAPIVersion": 10,
    "distributedNotificationEnabled": true,
    "apiReleaseType": "Beta5"
  }
}
```

### 配置项说明

| 配置项 | 值 | 描述 |
|--------|-----|------|
| `bundleName` | `com.ohos.useriam.authwidget` | 应用包名 |
| `vendor` | `example` | 厂商名 |
| `versionCode` | `1000000` | 版本码 |
| `versionName` | `1.0.0` | 版本名 |
| `icon` | `$media:app_icon` | 应用图标 |
| `label` | `$string:app_name` | 应用标签 |
| `minAPIVersion` | `10` | 最低 API 版本 |
| `targetAPIVersion` | `10` | 目标 API 版本 |
| `apiReleaseType` | `Beta5` | API 发布类型 |

## build-profile.json5

**证据**: `build-profile.json5` 行 1-41

构建配置，定义产品配置和模块配置。

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        {
          "name": "default",
          "applyToProducts": ["default"]
        }
      ]
    }
  ]
}
```

### 配置项说明

| 配置项 | 值 | 描述 |
|--------|-----|------|
| `products[0].name` | `default` | 产品名 |
| `products[0].compileSdkVersion` | `23` | 编译 SDK 版本 |
| `products[0].compatibleSdkVersion` | `23` | 兼容 SDK 版本 |
| `modules[0].name` | `entry` | 模块名 |
| `modules[0].srcPath` | `./entry` | 模块源码路径 |

## 资源管理

### 资源目录结构

**证据**: `entry/src/main/resources/` 目录分析

```
resources/
├── base/
│   ├── element/           # 基础元素
│   │   ├── color.json    # 颜色定义
│   │   ├── string.json   # 字符串定义
│   │   ├── float.json    # 浮点数定义
│   │   └── image.json    # 图片定义
│   └── profile/
│       └── main_pages.json # 页面配置
├── en_US/                # 英文资源
│   └── element/
│       └── string.json
└── zh_CN/               # 中文资源
    └── element/
        └── string.json
```

### 多语言支持

| 语言 | 路径 | 用途 |
|------|------|------|
| 默认 | `base/` | 默认语言资源 |
| 英文 | `en_US/` | 英文界面 |
| 中文 | `zh_CN/` | 中文界面 |

## 相关文档

- [GN 构建配置](10_GN_Build.md) - BUILD.gn targets 详解
- [概览](00_Overview.md) - 模块在系统中的定位
- [安全风险评审](20_Security_Review.md) - 权限安全分析

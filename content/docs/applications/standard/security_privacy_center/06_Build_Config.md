# 构建配置

## 构建系统概述

本项目使用 **hvigor** 作为构建工具，它是基于 Gradle 的 OpenHarmony 项目构建系统。

| 属性 | 值 |
|-----|---|
| 构建工具 | hvigor |
| 构建语言 | TypeScript（hvigorfile.ts） |
| 模块系统 | OpenHarmony Module（.hap） |
| 编译 SDK | compileSdkVersion 23 |

## 配置文件结构

### 工程级配置

| 文件 | 路径 | 用途 |
|-----|------|------|
| `build-profile.json5` | 根目录 | 工程级构建配置（签名、产物、SDK 版本） |
| `hvigorfile.ts` | 根目录 | 工程级构建任务脚本 |
| `hvigor-config.json5` | hvigor/ | hvigor 自身配置 |

### 模块级配置

| 文件 | 路径 | 用途 |
|-----|------|------|
| `build-profile.json5` | entry/ | 模块级构建配置（模块列表、目标） |
| `hvigorfile.ts` | entry/ | 模块级构建任务脚本 |
| `module.json5` | entry/src/main/ | 模块能力声明（页面、权限、Ability） |

## build-profile.json5 详解

### 工程级配置

**文件路径**：`build-profile.json5`

```json5
{
  "app": {
    "signingConfigs": [           // 签名配置
      {
        "name": "default",
        "material": {
          "storePassword": "******",  // 密钥库密码（敏感）
          "certpath": "signature/OpenHarmonyApplication.cer",
          "keyAlias": "openharmony application release",
          "keyPassword": "******",    // 密钥密码（敏感）
          "profile": "signature/privacyCenter.p7b",
          "signAlg": "SHA256withECDSA",
          "storeFile": "signature/OpenHarmony.p12"
        }
      }
    ],
    "products": [                // 构建产物配置
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23,
        "runtimeOS": "OpenHarmony"
      }
    ],
    "buildModeSet": [            // 构建模式
      { "name": "debug" },
      { "name": "release" }
    ]
  },
  "modules": [                   // 模块列表
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

**关键配置项**：

| 配置项 | 值 | 说明 |
|-------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| signingConfig | default | 使用默认签名 |
| signAlg | SHA256withECDSA | 签名算法 |

**证据来源**：`build-profile.json5:17-64`

### 模块级配置

**文件路径**：`entry/build-profile.json5`

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["default"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": ["pages/Index", "pages/locationServices", "pages/UiExtensionPage"],
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "exported": true,
        // ... 其他配置
      }
    ],
    "requestPermissions": [
      // 权限声明列表
    ]
  }
}
```

**证据来源**：`entry/src/main/module.json5:15-79`

## module.json5 详解

### 模块声明

| 属性 | 值 | 说明 |
|-----|-----|------|
| name | entry | 模块名称 |
| type | entry | 模块类型（entry/externalAbility/library） |
| description | $string:module_desc | 模块描述 |
| mainElement | EntryAbility | 主入口元素 |
| deliveryWithInstall | true | 安装时交付 |
| pages | $profile:main_pages | 页面配置 |

**证据来源**：`module.json5:16-26`

### Ability 声明

**EntryAbility 配置**：

| 属性 | 值 | 说明 |
|-----|-----|------|
| name | EntryAbility | Ability 名称 |
| srcEntry | ./ets/entryability/EntryAbility.ets | 源码入口 |
| exported | true | 是否导出 |
| permissions | ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER | 需要权限 |

**技能配置（Skills）**：

```json
{
  "skills": [
    {
      "entities": ["entity.system.home"],
      "actions": ["action.system.home"]
    }
  ]
}
```

**证据来源**：`module.json5:28-49`

### 权限声明

本项目声明了 7 个权限：

| 权限 | 敏感级别 | 用途 |
|-----|---------|------|
| ohos.permission.MANAGE_SECURE_SETTINGS | 高 | 管理安全设置 |
| ohos.permission.ACCESS_BUNDLE_DIR | 高 | 访问应用目录 |
| ohos.permission.GET_BUNDLE_INFO | 中 | 获取应用信息 |
| ohos.permission.GET_INSTALLED_BUNDLE_LIST | 中 | 获取已安装列表 |
| ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER | 高 | 访问安全隐私中心 |
| ohos.permission.ACCESS_CERT_MANAGER | 高 | 访问证书管理 |
| ohos.permission.CONTROL_LOCATION_SWITCH | 高 | 控制位置开关 |

**证据来源**：`module.json5:51-77`

## 页面配置

### main_pages.json

**文件路径**：`entry/src/main/resources/base/profile/main_pages.json`

```json
{
  "src": [
    "pages/Index",
    "pages/locationServices",
    "pages/UiExtensionPage"
  ]
}
```

**页面路由映射**：

| 路由路径 | 对应文件 | 功能 |
|---------|---------|------|
| pages/Index | Index.ets | 首页 |
| pages/locationServices | locationServices.ets | 位置服务页 |
| pages/UiExtensionPage | UiExtensionPage.ets | UIExtension 页 |

## 资源配置文件

### 字符串资源

| 文件 | 语言 | 说明 |
|-----|------|------|
| resources/base/element/string.json | 默认 | 默认字符串 |
| resources/zh_CN/element/string.json | 简体中文 | 中文翻译 |
| resources/en_US/element/string.json | 英文 | 英文翻译 |

**证据来源**：`entry/src/main/resources/`

### 颜色资源

**文件路径**：`entry/src/main/resources/base/element/color.json`

定义应用使用的颜色常量。

## 构建命令

### 本地构建

```bash
# 使用 DevEco Studio
# Build → Build Haps/App(s) → Build Hap(s)

# 或使用命令行
hpm build
```

### 构建模式

| 模式 | 说明 |
|-----|------|
| debug | 调试模式，包含调试信息 |
| release | 发布模式，优化大小 |

## 构建配置变更影响

| 配置项变更 | 影响范围 |
|-----------|---------|
| compileSdkVersion | 需要对应版本的 SDK |
| signingConfig | 影响签名验证 |
| permissions | 影响运行时权限校验 |
| pages | 影响路由映射 |
| deviceTypes | 影响支持的设备范围 |

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [05_Inner_API.md](./05_Inner_API.md) → 内部 API
- [07_Build_Outputs.md](./07_Build_Outputs.md) → 编译产物
- [08_Security_Review.md](./08_Security_Review.md) → 安全评审

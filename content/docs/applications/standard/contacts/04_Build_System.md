# 构建系统

## 概述

Contacts 应用使用 **hvigor** 作为构建工具，基于 Gradle 体系构建 OpenHarmony 应用。

## 构建配置

### 项目级配置

**文件**: `build-profile.json5`

```json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "default",
        "material": {
          "storePassword": "****",  // 敏感信息已脱敏
          "certpath": "sign/IDE.cer",
          "keyAlias": "OpenHarmony Application CA",
          "keyPassword": "****",
          "profile": "sign/contacts.p7b",
          "signAlg": "SHA256withECDSA",
          "storeFile": "sign/OpenHarmony_stage.p12"
        }
      }
    ],
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
    { "name": "entry", "srcPath": "./entry" },
    { "name": "common", "srcPath": "./common" },
    { "name": "phonenumber", "srcPath": "./feature/phonenumber" },
    { "name": "contact", "srcPath": "./feature/contact" },
    { "name": "account", "srcPath": "./feature/account" },
    { "name": "call", "srcPath": "./feature/call" },
    { "name": "dialpad", "srcPath": "./feature/dialpad" }
  ]
}
```

> **证据来源**: `build-profile.json5`

### 模块配置

| 模块 | 类型 | 签名配置 |
|------|------|----------|
| entry | entry | default |
| common | feature | - |
| phonenumber | feature | - |
| contact | feature | - |
| account | feature | - |
| call | feature | - |
| dialpad | feature | - |

### SDK 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |

## hvigor 配置

### hvigorfile.js

```javascript
// 证据来源: hvigorfile.js
// 模块级构建入口
module.exports = require('@ohos/hvigor-ohos-plugin').staticComponentTasks
```

### hvigor 目录

```
hvigor/
├── hvigorfile.js    # 构建入口脚本
├── hvigorw          # Linux/Mac 构建脚本
└── hvigorw.bat      # Windows 构建脚本
```

## 模块结构

### entry 模块配置

**文件**: `entry/src/main/module.json5`

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntry": "./ets/Application/MyAbilityStage.ts",
    "mainElement": "com.ohos.contacts.MainAbility",
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "com.ohos.contacts.MainAbility",
        "srcEntry": "./ets/MainAbility/MainAbility.ts",
        "icon": "$media:ic_contact_icon",
        "label": "$string:app_name"
      }
    ],
    "requestPermissions": [
      // 权限声明列表
    ]
  }
}
```

### feature 模块配置

每个 feature 模块包含 `module.json5`：

| 模块 | module.json5 路径 |
|------|-------------------|
| contact | `feature/contact/src/main/module.json5` |
| call | `feature/call/src/main/module.json5` |
| dialpad | `feature/dialpad/src/main/module.json5` |
| phonenumber | `feature/phonenumber/src/main/module.json5` |
| account | `feature/account/src/main/module.json5` |
| common | `common/src/main/module.json5` |

## 签名配置

### 签名文件

| 文件 | 用途 |
|------|------|
| `sign/IDE.cer` | 签名证书 |
| `sign/OpenHarmony_stage.p12` | 密钥库 |
| `sign/contacts.p7b` | 签名 profile |

### 签名算法

- **算法**: SHA256withECDSA
- **密钥别名**: OpenHarmony Application CA

## 构建产物

### 预期产物

| 产物类型 | 说明 | 位置 |
|----------|------|------|
| .hap | Harmony Ability Package | `entry/build/default/outputs/default/` |
| .hsp | Harmony Shared Package | `feature/*/build/default/outputs/default/` |

### 产物结构

```
out/
└── default/
    ├── entry/
    │   └── build/
    │       └── default/
    │           └── outputs/
    │               └── default/
    │                   └── entry.hap
    ├── contact/
    │   └── build/
    │       └── default/
    │           └── outputs/
    │               └── default/
    │                   └── contact.hsp
    └── ...
```

## 构建命令

### 常用命令

```bash
# 构建所有模块
hvigor build

# 构建指定模块
hvigor build --module-name entry

# 清理构建
hvigor clean

# 查看帮助
hvigor --help
```

### 构建选项

| 选项 | 说明 |
|------|------|
| `--module-name` | 指定模块名 |
| `--product` | 指定产品配置 |
| `--debug` | Debug 构建 |
| `--release` | Release 构建 |

## 构建优化

### 资源编译

| 配置 | 值 |
|------|-----|
| 资源目录 | `src/main/resources/` |
| 多语言支持 | `zh_CN/`, `en_US/` |
| 资源类型 | element, media, layout, etc. |

### 编译配置

```json5
{
  "module": {
    "metadata": [
      {
        "name": "ArkTSPartialUpdate",
        "value": "true"
      }
    ]
  }
}
```

## 常见问题

### 1. 签名失败

**症状**: 构建时报签名错误

**解决方案**: 
- 检查签名文件是否存在
- 验证 `build-profile.json5` 中的路径配置
- 确保密钥密码正确

### 2. SDK 版本不匹配

**症状**: 提示 SDK 版本不兼容

**解决方案**:
- 检查 `compileSdkVersion` 和 `compatibleSdkVersion`
- 确保本地 SDK 版本 >= 配置版本

### 3. 模块依赖缺失

**症状**: 找不到模块依赖

**解决方案**:
- 检查 `modules` 配置中的 `srcPath`
- 确保模块目录存在且包含 `module.json5`

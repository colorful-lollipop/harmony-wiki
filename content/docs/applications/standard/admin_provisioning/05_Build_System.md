# 05_构建系统

> 本文档基于代码证据编写，证据来源见各章节引用。

## 构建系统概述

### 构建工具链

| 工具 | 版本 | 证据文件 |
|-----|-----|---------|
| **hvigor** | 1.0.6 | `package.json:14` |
| **hvigor-ohos-plugin** | 1.0.6 | `package.json:13` |
| **ohos.gni** | - | `BUILD.gn:14` |

### 构建配置

**构建入口脚本**: `hvigorfile.js`

**构建产物**: HAP (Harmony Ability Package)

### 构建命令

```bash
# Debug 构建
hvigor --mode module -p product=phone assembleHapDebug --strict-mode

# Release 构建
hvigor --mode module -p product=phone assembleHapRelease --strict-mode
```

**证据** (`hvigorfile.js`):
```javascript
// 标准 Hvigor 构建脚本
```

---

## GN 构建配置

### 构建入口

**文件**: `BUILD.gn`

**证据** (`BUILD.gn:14-27`):

```gn
import("//build/ohos.gni")

ohos_hap("adminprovisioning_hap") {
  hap_profile = "entry/src/main/module.json"
  deps = [
    ":adminprovisioning_js_assets",
    ":adminprovisioning_resources",
  ]
  certificate_profile = "signature/adminprovisioning.p7b"
  hap_name = "adminprovisioning"
  subsystem_name = "applications"
  part_name = "prebuilt_hap"
  module_install_dir = "app/com.ohos.adminprovisioning"
}
```

### Target 清单

| Target 名称 | 类型 | 用途 | 证据 |
|------------|------|-----|-----|
| `adminprovisioning_hap` | ohos_hap | HAP 包构建主入口 | `BUILD.gn:16` |
| `adminprovisioning_js_assets` | ohos_js_assets | ArkTS/JS 源码编译 | `BUILD.gn:29` |
| `adminprovisioning_resources` | ohos_resources | 资源文件处理 | `BUILD.gn:39` |
| `adminprovisioning_app_profile` | ohos_app_scope | 应用配置 | `BUILD.gn:34` |

---

### 详细 Target 配置

#### 1. adminprovisioning_hap

**类型**: `ohos_hap`

**用途**: 构建 HAP 安装包

**配置**:

| 属性 | 值 | 说明 |
|-----|-----|-----|
| `hap_profile` | `entry/src/main/module.json` | 模块配置文件 |
| `deps` | `:adminprovisioning_js_assets`, `:adminprovisioning_resources` | 依赖 |
| `certificate_profile` | `signature/adminprovisioning.p7b` | 签名文件 |
| `hap_name` | `adminprovisioning` | HAP 名称 |
| `subsystem_name` | `applications` | 子系统名 |
| `part_name` | `prebuilt_hap` | Part 名称 |
| `module_install_dir` | `app/com.ohos.adminprovisioning` | 安装路径 |

**证据** (`BUILD.gn:16-27`):
```gn
ohos_hap("adminprovisioning_hap") {
  hap_profile = "entry/src/main/module.json"
  deps = [
    ":adminprovisioning_js_assets",
    ":adminprovisioning_resources",
  ]
  certificate_profile = "signature/adminprovisioning.p7b"
  hap_name = "adminprovisioning"
  subsystem_name = "applications"
  part_name = "prebuilt_hap"
  module_install_dir = "app/com.ohos.adminprovisioning"
}
```

---

#### 2. adminprovisioning_js_assets

**类型**: `ohos_js_assets`

**用途**: 编译 ArkTS/JS 源码为 ABC 字节码

**配置**:

| 属性 | 值 | 说明 |
|-----|-----|-----|
| `ets2abc` | `true` | 启用 ETS 到 ABC 编译 |
| `source_dir` | `entry/src/main/ets` | 源码目录 |

**证据** (`BUILD.gn:29-32`):
```gn
ohos_js_assets("adminprovisioning_js_assets") {
  ets2abc = true
  source_dir = "entry/src/main/ets"
}
```

---

#### 3. adminprovisioning_resources

**类型**: `ohos_resources`

**用途**: 编译资源文件

**配置**:

| 属性 | 值 | 说明 |
|-----|-----|-----|
| `sources` | `entry/src/main/resources` | 资源目录 |
| `deps` | `:adminprovisioning_app_profile` | 依赖 |
| `hap_profile` | `entry/src/main/module.json` | 模块配置 |

**证据** (`BUILD.gn:39-43`):
```gn
ohos_resources("adminprovisioning_resources") {
  sources = [ "entry/src/main/resources" ]
  deps = [ ":adminprovisioning_app_profile" ]
  hap_profile = "entry/src/main/module.json"
}
```

---

#### 4. adminprovisioning_app_profile

**类型**: `ohos_app_scope`

**用途**: 处理应用级配置

**配置**:

| 属性 | 值 | 说明 |
|-----|-----|-----|
| `app_profile` | `AppScope/app.json` | 应用配置 |
| `sources` | `AppScope/resources` | 资源目录 |

**证据** (`BUILD.gn:34-37`):
```gn
ohos_app_scope("adminprovisioning_app_profile") {
  app_profile = "AppScope/app.json"
  sources = [ "AppScope/resources" ]
}
```

---

## 模块配置

### module.json5

**文件**: `entry/src/main/module.json5`

**证据** (`module.json5:1-81`):

```json5
{
  "module": {
    "name": "adminprovisioning",
    "type": "entry",
    "srcEntrance": "./ets/Application/AbilityStage.ts",
    "description": "$string:description_adminProvisioning",
    "mainElement": "com.ohos.adminprovisioning.MainAbility",
    "deviceTypes": ["default", "tablet", "2in1"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "uiSyntax": "ets",
    "abilities": [
      // MainAbility 配置
      {
        "name": "com.ohos.adminprovisioning.MainAbility",
        "srcEntrance": "./ets/MainAbility/MainAbility.ts",
        // ...
      },
      // AutoManagerAbility 配置
      {
        "name": "com.ohos.automanager.AutoManagerAbility",
        "srcEntrance": "./ets/MainAbility/AutoManagerAbility.ts",
        "permissions": ["ohos.permission.PROVISIONING_MESSAGE"],
        // ...
      }
    ],
    "extensionAbilities": [
      {
        "name": "MDMUIExtensionAbility",
        "type": "ui",
        "srcEntrance": "./ets/MainAbility/UIExtensionAbility.ets"
      }
    ],
    "requestPermissions": [
      // 权限声明列表
    ]
  }
}
```

---

### app.json

**文件**: `AppScope/app.json`

**证据** (`AppScope/app.json:1-13`):

```json
{
  "app": {
    "bundleName": "com.ohos.adminprovisioning",
    "vendor": "ohos",
    "versionCode": 1000001,
    "versionName": "1.0.1",
    "icon": "$media:app_icon",
    "label": "$string:app_name",
    "distributedNotificationEnabled": true,
    "minAPIVersion": 9,
    "targetAPIVersion": 9
  }
}
```

---

## 构建产物

| 产物 | 路径 | 说明 |
|-----|-----|-----|
| **Debug HAP** | `build/outputs/hap/debug/phone/` | 未签名调试版本 |
| **Release HAP** | `build/outputs/hap/release/phone/` | 签名发布版本 |

**证据** (`doc/Instructions.md:120-135`):
```
编译完成后，hap包会生成在工程目录下的 `\build\outputs\hap\debug\phone\` 路径下
```

---

## 构建变体

### 构建类型

| 类型 | 说明 | 配置 |
|-----|-----|-----|
| **Debug** | 调试构建，未签名 | `OhosBuild Variants` → debug |
| **Release** | 发布构建，需签名 | `OhosBuild Variants` → release |

### 签名配置

**签名文件**: `signature/adminprovisioning.p7b`

**配置位置**: DevEco Studio → Project Structure → Signing Configs

**证据** (`doc/Instructions.md:90-105`):
1. 拷贝 OpenHarmony 标准版工程的 `prebuilts/signcenter` 目录
2. 拷贝 `signature/adminprovisioning.p7b` 到该目录
3. 在 DevEco Studio 中配置签名

---

## 依赖关系图

```
adminprovisioning_hap
    ├── deps
    │   ├── adminprovisioning_js_assets
    │   │   └── source_dir: entry/src/main/ets
    │   │
    │   └── adminprovisioning_resources
    │       ├── deps
    │       │   └── adminprovisioning_app_profile
    │       │       ├── app_profile: AppScope/app.json
    │       │       └── sources: AppScope/resources
    │       └── hap_profile: entry/src/main/module.json
    │
    └── certificate_profile: signature/adminprovisioning.p7b
```

---

*文档版本: 1.0*

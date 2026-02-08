# GN Targets 与编译产物

> 本文档描述用户证书管理部件的 GN 构建配置、Target 定义和编译产物

---

## 目的

本文档帮助构建工程师理解 GN 构建配置、Targets 定义和编译产物映射。

## 适用范围

- OpenHarmony 构建工程师
- 需要了解构建系统的开发者
- 需要修改构建配置的技术人员

## 关键结论

1. **单 HAP 产物**: 最终产物为 `CertificateManager.hap`
2. **主要 Target**: `user_cert_manager` (ohos_app)
3. **依赖简单**: 仅依赖 JS 资源和资源目标
4. **无 C/C++ 模块**: 全部为 ArkTS，无需编译 native 代码
5. **安装路径**: `app/com.ohos.certificatemanager`

## 相关跳转

- [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 目录结构
- [07_Build_Artifacts.md](wiki/07_Build_Artifacts.md) - 编译产物

---

## 1. GN 构建配置

### 1.1 根 BUILD.gn

#### 1.1.1 文件路径

```
/Volumes/lexar/code/d/work/oh/applications/standard/user_certificate_manager/BUILD.gn
```

**证据位置**: `BUILD.gn:1-49`

---

## 2. Target 定义

### 2.1 user_cert_manager - 主应用 Target

#### 2.1.1 Target 定义

```gn
ohos_app("user_cert_manager") {
  deps = [
    ":cert_manager_js_assets",
    ":cert_manager_resources",
  ]
  publicity_file = "publicity.xml"
  certificate_profile = "signature/privacyCenter.p7b"
  hap_name = "CertificateManager"
  part_name = "user_certificate_manager"
  subsystem_name = "applications"
  js_build_mode = "release"
  module_install_dir = "app/com.ohos.certificatemanager"
  sdk_home = "//prebuilts/ohos-sdk/linux"
  sdk_type_name = [ "sdk.dir" ]
  assemble_type = "assembleHap"
  build_level = "module"
  build_modules = [ "CertManager" ]
}
```

#### 2.1.2 Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `ohos_app` | 目标类型：应用 |
| `hap_name` | `CertificateManager` | HAP 文件名 |
| `part_name` | `user_certificate_manager` | 部件名称 |
| `subsystem_name` | `applications` | 子系统名称 |
| `js_build_mode` | `release` | JS 构建模式 |
| `module_install_dir` | `app/com.ohos.certificatemanager` | 安装路径 |
| `certificate_profile` | `signature/privacyCenter.p7b` | 签名证书 |
| `assemble_type` | `assembleHap` | 汇编类型：HAP |

#### 2.1.3 依赖

| Target | 类型 | 说明 |
|--------|------|------|
| `:cert_manager_js_assets` | `ohos_js_assets` | JS 资源 |
| `:cert_manager_resources` | `ohos_resources` | 资源文件 |

**证据位置**: `BUILD.gn:15-32`

---

### 2.2 user_cert_manager_app_profile - 应用配置 Target

#### 2.2.1 Target 定义

```gn
ohos_app_scope("user_cert_manager_app_profile") {
  app_profile = "AppScope/app.json"
  sources = [ "AppScope/resources" ]
}
```

#### 2.2.2 Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `ohos_app_scope` | 目标类型：应用配置 |
| `app_profile` | `AppScope/app.json` | 应用配置文件 |
| `sources` | `[ "AppScope/resources" ]` | 资源目录 |

**证据位置**: `BUILD.gn:34-37`

---

### 2.3 cert_manager_js_assets - JS 资源 Target

#### 2.3.1 Target 定义

```gn
ohos_js_assets("cert_manager_js_assets") {
  ets2abc = true
  source_dir = "certmanager/src/main/ets"
}
```

#### 2.3.2 Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `ohos_js_assets` | 目标类型：JS 资源 |
| `ets2abc` | `true` | 启用 ETS 转 ABC |
| `source_dir` | `certmanager/src/main/ets` | 源目录 |

**证据位置**: `BUILD.gn:39-42`

---

### 2.4 cert_manager_resources - 资源 Target

#### 2.4.1 Target 定义

```gn
ohos_resources("cert_manager_resources") {
  sources = [ "certmanager/src/main/resources" ]
  deps = [ ":user_cert_manager_app_profile" ]
  hap_profile = "certmanager/src/main/module.json"
}
```

#### 2.4.2 Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `ohos_resources` | 目标类型：资源 |
| `sources` | `[ "certmanager/src/main/resources" ]` | 资源目录 |
| `deps` | `[ ":user_cert_manager_app_profile" ]` | 依赖：应用配置 |
| `hap_profile` | `certmanager/src/main/module.json` | 模块配置 |

**证据位置**: `BUILD.gn:44-48`

---

## 3. 依赖关系图

```
user_cert_manager (ohos_app) ⭐
├── cert_manager_js_assets (ohos_js_assets)
│   └── source: certmanager/src/main/ets
└── cert_manager_resources (ohos_resources)
    ├── source: certmanager/src/main/resources
    └── user_cert_manager_app_profile (ohos_app_scope)
        ├── app_profile: AppScope/app.json
        └── sources: AppScope/resources
```

**证据位置**: `BUILD.gn:15-48`

---

## 4. bundle.json 部件配置

### 4.1 配置内容

```json
{
  "name": "@ohos/user_certificate_manager",
  "description": "user_certificate_manager",
  "version": "5.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "applications/standard/user_certificate_manager"
  },
  "dirs": {},
  "scripts": {},
  "author": {},
  "repository": "",
  "component": {
    "name": "user_certificate_manager",
    "subsystem": "applications",
    "syscap": [],
    "features": [],
    "adapted_system_type": ["standard"],
    "hisysevent_config": [],
    "rom": "1MB",
    "ram": "1MB",
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [
        "//applications/standard/user_certificate_manager:user_cert_manager"
      ],
      "inner_kits": [],
      "test": []
    }
  }
}
```

### 4.2 关键配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | `@ohos/user_certificate_manager` | 组件名称 |
| `subsystem` | `applications` | 子系统 |
| `adapted_system_type` | `["standard"]` | 适配系统类型 |
| `rom` | `"1MB"` | ROM 占用 |
| `ram` | `"1MB"` | RAM 占用 |
| `sub_component` | `["//applications/standard/user_certificate_manager:user_cert_manager"]` | 子部件 |

**证据位置**: `bundle.json:1-36`

---

## 5. build-profile.json5 构建配置

### 5.1 配置内容

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23,
        "runtimeOS": "OpenHarmony",
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
    },
    {
      "name": "CertManager",
      "srcPath": "./certmanager",
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

### 5.2 关键配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `compileSdkVersion` | `23` | 编译 SDK 版本 |
| `compatibleSdkVersion` | `23` | 兼容 SDK 版本 |
| `runtimeOS` | `OpenHarmony` | 运行时 OS |
| `modules` | `entry`, `CertManager` | 模块列表 |

**证据位置**: `build-profile.json5:16-53`

---

## 6. module.json 模块配置

### 6.1 配置内容

```json
{
  "module": {
    "name": "CertManager",
    "type": "feature",
    "description": "$string:mainability_description",
    "mainElement": "MainAbility",
    "deviceTypes": ["default"],
    "metadata": [
      {
        "name": "ArkTSPartialUpdate",
        "value": "true"
      },
      {
        "name": "partialUpdateStrictCheck",
        "value": "all"
      }
    ],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "MainAbility",
        "srcEntry": "./ets/MainAbility/MainAbility.ts",
        "description": "$string:mainability_description",
        "icon": "$media:icon",
        "label": "$string:entry_MainAbility",
        "exported": true,
        "launchType": "singleton",
        "startWindowIcon": "$media:icon",
        "startWindowBackground": "$color:color_1",
        "orientation": "auto_rotation_restricted",
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "extensionAbilities": [
      {
        "name": "MainExtensionAbility",
        "srcEntry": "./ets/MainAbility/MainExtensionAbility.ts",
        "description": "$string:mainability_description",
        "icon": "$media:icon",
        "label": "$string:entry_MainAbility",
        "exported": true,
        "permissions": ["ohos.permission.ACCESS_CERT_MANAGER"],
        "type": "sys/commonUI",
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home", "action.access.privacy.center", "ohos.want.action.viewData"]
          }
        ],
        "metadata": [
          {
            "name": "metadata.access.privacy.center",
            "value": "security_privacy.json"
          }
        ]
      },
      {
        "name": "CertPickerUIExtAbility",
        "srcEntry": "./ets/MainAbility/CertPickerUiExtAbility.ets",
        "exported": true,
        "permissions": ["ohos.permission.ACCESS_CERT_MANAGER"],
        "type": "sys/commonUI"
      }
    ],
    "requestPermissions": [
      {"name": "ohos.permission.GET_BUNDLE_INFO"},
      {"name": "ohos.permission.ACCESS_CERT_MANAGER_INTERNAL"},
      {"name": "ohos.permission.ACCESS_CERT_MANAGER"},
      {"name": "ohos.permission.GET_BUNDLE_RESOURCES"},
      {"name": "ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER"},
      {"name": "ohos.permission.ACCESS_BIOMETRIC"},
      {"name": "ohos.permission.PRIVACY_WINDOW"},
      {"name": "ohos.permission.ACCESS_SYSTEM_APP_CERT"},
      {"name": "ohos.permission.ACCESS_USER_TRUSTED_CERT"},
      {"name": "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED"}
    ]
  }
}
```

### 6.2 关键配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | `CertManager` | 模块名称 |
| `type` | `feature` | 模块类型 |
| `mainElement` | `MainAbility` | 主入口 |
| `abilities` | `MainAbility` | Ability 列表 |
| `extensionAbilities` | `MainExtensionAbility`, `CertPickerUIExtAbility` | Extension Ability 列表 |
| `requestPermissions` | 10 个权限 | 申请的权限列表 |

**证据位置**: `certmanager/src/main/module.json:1-117`

---

## 7. Target 产物映射

### 7.1 Target → 产物映射

| Target | 类型 | 输出产物 | 说明 |
|--------|------|----------|------|
| `user_cert_manager` | `ohos_app` | `CertificateManager.hap` | 最终 HAP 文件 |
| `cert_manager_js_assets` | `ohos_js_assets` | `*.abc` 文件 | 编译后的 JS 文件 |
| `cert_manager_resources` | `ohos_resources` | 资源包 | 资源文件打包 |
| `user_cert_manager_app_profile` | `ohos_app_scope` | 应用配置 | 应用元数据 |

### 7.2 最终产物

```
out/rk3568/obj/applications/standard/user_certificate_manager/user_cert_manager/
└── CertificateManager.hap
```

**安装路径**: `app/com.ohos.certificatemanager`

**证据位置**: `README.md:103`, `BUILD.gn:26`

---

## 8. 编译命令

### 8.1 单仓编译

```bash
./build.sh --product-name rk3568 --ccache --build-target user_certificate_manager
```

**参数说明**:
- `--product-name`: 产品名称（如 rk3568）
- `--ccache`: 启用编译缓存
- `--build-target`: 编译目标

**证据位置**: `README.md:89-91`

### 8.2 应用安装

```bash
hdc install CertificateManager.hap
```

**证据位置**: `README.md:106-109`

---

## 9. 构建配置文件清单

| 文件路径 | 类型 | 作用 |
|---------|------|------|
| `BUILD.gn` | GN 构建配置 | 定义 Targets |
| `bundle.json` | Bundle 元数据 | 部件定义 |
| `build-profile.json5` | 构建 Profile | 模块配置、SDK 版本 |
| `certmanager/BUILD.gn` | 模块 GN 配置 | 模块 Targets |
| `certmanager/oh-package.json5` | 模块依赖 | 模块依赖管理 |
| `certmanager/src/main/module.json` | 模块配置 | Ability、权限 |
| `AppScope/app.json` | 应用配置 | Bundle 名称、版本 |

---

**END OF 06_GN_Targets.md**

# 构建指南

## 1. 构建系统概述

### 1.1 构建工具

| 工具 | 版本 | 用途 |
|-----|------|-----|
| hvigor | - | 模块构建工具 |
| hb (HarmonyOS Build) | - | 全局构建工具 |
| Node.js | - | JavaScript 运行时 |
| npm | - | 包管理工具 |

### 1.2 构建流程

```
源代码 (.ets)
     │
     ▼
┌─────────────────┐
│  ETS 编译器      │  ArkTS → 中间字节码
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ABC 虚拟机      │  字节码 → ARM/其他
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  资源编译器      │  资源文件编译
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  HAP 打包工具   │  生成可安装包
└────────┬────────┘
         │
         ▼
   HAP 安装包
```

## 2. 构建配置

### 2.1 根目录构建文件

```
/Volumes/lexar/code/d/work/oh/applications/standard/call/
├── BUILD.gn              ← GN 构建主配置
├── build-profile.json5   ← hvigor 构建配置
├── hvigorfile.js         ← hvigor 任务脚本
├── hvigorw               ← hvigor 启动脚本
└── hvigorw.bat           ← Windows 启动脚本
```

### 2.2 BUILD.gn 配置

```gn
# 文件: BUILD.gn

import("//build/ohos.gni")

# 通话 UI HAP
ohos_hap("callui_hap") {
  hap_profile = "entry/src/main/module.json"
  deps = [
    ":callui_js_assets",
    ":callui_resources",
  ]
  certificate_profile = "signature/callui.p7b"
  hap_name = "CallUI"
  subsystem_name = "applications"
  part_name = "prebuilt_hap"
  module_install_dir = "app/com.ohos.callui"
}

# JS 资源编译
ohos_js_assets("callui_js_assets") {
  ets2abc = true                    # 启用 ETS → ABC 编译
  source_dir = "entry/src/main/ets"
}

# 应用级配置
ohos_app_scope("callui_app_profile") {
  app_profile = "AppScope/app.json"
  sources = [ "AppScope/resources" ]
}

# 资源编译
ohos_resources("callui_resources") {
  sources = [ "entry/src/main/resources" ]
  deps = [ ":callui_app_profile" ]
  hap_profile = "entry/src/main/module.json"
}

# 移动数据设置 HAP
ohos_hap("mobileDataSettings_hap") {
  hap_profile = "mobiledatasettings/src/main/module.json"
  deps = [
    ":mobiledatasettings_js_assets",
    ":mobiledatasettings_resources",
  ]
  certificate_profile = "signature/callui.p7b"
  hap_name = "MobileDataSettings"
  subsystem_name = "applications"
  part_name = "prebuilt_hap"
  module_install_dir = "app/com.ohos.callui"
}

ohos_js_assets("mobiledatasettings_js_assets") {
  ets2abc = true
  source_dir = "mobiledatasettings/src/main/ets"
}

ohos_resources("mobiledatasettings_resources") {
  sources = [ "mobiledatasettings/src/main/resources" ]
  deps = [ ":callui_app_profile" ]
  hap_profile = "mobiledatasettings/src/main/module.json"
}
```

### 2.3 build-profile.json5

```json5
// 文件: build-profile.json5
{
  "app": {
    "signingConfigs": [],           // 签名配置
    "compileSdkVersion": 9,        // 编译 SDK 版本
    "compatibleSdkVersion": 9,     // 兼容 SDK 版本
    "targetSdkVersion": 9,         // 目标 SDK 版本
    "products": [
      {
        "name": "default",         // 默认产品
        "signingConfig": "default",
        "compatibleSdkVersion": "9.0.0",
        "targetSdkVersion": "9.0.0",
        "compileSdkVersion": "9.0.0"
      }
    ],
    "modules": [
      {
        "name": "entry",
        "srcPath": "entry",
        "targets": [
          {
            "name": "default",
            "applyToProducts": ["default"]
          }
        ]
      },
      {
        "name": "mobiledatasettings",
        "srcPath": "mobiledatasettings",
        "targets": [
          {
            "name": "default",
            "applyToProducts": ["default"]
          }
        ]
      },
      {
        "name": "common",
        "srcPath": "common",
        "targets": [
          {
            "name": "default",
            "applyToProducts": ["default"]
          }
        ]
      }
    ]
  }
}
```

## 3. 模块配置

### 3.1 entry 模块配置

```json5
// 文件: entry/src/main/module.json
{
  "module": {
    "name": "callui",
    "type": "entry",                    // 模块类型: entry
    "srcEntrance": "./ets/Application/MyAbilityStage.ts",
    "description": "$string:callui_description",
    "mainElement": "com.ohos.callui.ServiceAbility",
    "deviceTypes": ["default", "tablet"],
    "deliveryWithInstall": true,       // 安装时部署
    "installationFree": false,         // 非免安装
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "com.ohos.callui.MainAbility",
        "srcEntrance": "./ets/MainAbility/MainAbility.ts",
        "description": "$string:callui_description",
        "icon": "$media:app_icon",
        "label": "$string:callui_description",
        "startWindowIcon": "$media:app_start_window_icon",
        "startWindowBackground": "$color:black",
        "visible": false,              // 对用户不可见
        "backgroundModes": ["voip"]    // VoIP 后台模式
      }
    ],
    "extensionAbilities": [
      {
        "name": "com.ohos.callui.ServiceAbility",
        "type": "service",
        "visible": true,
        "srcEntrance": "./ets/ServiceAbility/ServiceAbility.ts",
        "permissions": ["ohos.permission.PLACE_CALL"]
      }
    ],
    "requestPermissions": [/* 权限列表 */]
  }
}
```

### 3.2 mobiledatasettings 模块配置

```json5
// 文件: mobiledatasettings/src/main/module.json
{
  "module": {
    "name": "mobiledatasettings",
    "type": "feature",                 // 模块类型: feature
    "srcEntrance": "./ets/Application/MyAbilityStage.ts",
    "description": "$string:app_name",
    "mainElement": "com.ohos.mobiledatasettings.MainAbility",
    "deviceTypes": ["default", "tablet"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "com.ohos.mobiledatasettings.MainAbility",
        "srcEntrance": "./ets/MainAbility/MainAbility.ts",
        "description": "$string:mainability_description",
        "visible": true
      }
    ],
    "requestPermissions": [
      { "name": "ohos.permission.GET_NETWORK_INFO" },
      { "name": "ohos.permission.SET_TELEPHONY_STATE" },
      { "name": "ohos.permission.GET_TELEPHONY_STATE" },
      { "name": "ohos.permission.MANAGE_SECURE_SETTINGS" }
    ]
  }
}
```

### 3.3 common 模块配置

```json5
// 文件: common/src/main/module.json5
{
  "module": {
    "name": "common",
    "type": "har",                    // 模块类型: HAR
    "deviceTypes": ["default", "tablet"]
  }
}
```

## 4. 构建产物

### 4.1 产物清单

| 模块 | 产物类型 | 产物名称 | 路径 |
|-----|---------|---------|------|
| entry | HAP | CallUI.hap | `out/default/apps/default/com.ohos.callui/CallUI.hap` |
| mobiledatasettings | HAP | MobileDataSettings.hap | `out/default/apps/default/com.ohos.callui/MobileDataSettings.hap` |
| common | HAR | common.har | `out/default/libs/common.har` |

### 4.2 产物结构

```
out/default/
├── apps/
│   └── default/
│       └── com.ohos.callui/
│           ├── CallUI.hap              ← entry 产物
│           │   ├── config.json         ← 模块配置
│           │   ├── index.dev            ← 入口点
│           │   ├── libs/               ← 共享库
│           │   ├── resources/          ← 资源文件
│           │   └── modules/            ← 字节码模块
│           │       └── default/
│           │           ├── AB/*.abc    ← 编译后的字节码
│           │           └── ...
│           │
│           └── MobileDataSettings.hap  ← mobiledatasettings 产物
│
└── libs/
    └── common.har                      ← common 静态库
```

### 4.3 HAP 包结构

```
CallUI.hap/
├── META-INF/
│   └── MANIFEST.MF                    ← 包清单
├── config.json                        ← 模块配置
├── index.dev                          ← 入口描述
├── libs/
│   └── lib*.so                        ← Native 库 (如有)
├── resources/
│   ├── base/
│   │   ├── media/                     ← 图片资源
│   │   ├── element/                   ← 字符串/颜色等
│   │   └── layout/                    ← 布局文件
│   └── zh_CN/                         ← 中文资源
│
└── modules/
    └── default/
        ├── App.abc                    ← 应用字节码
        ├── MainAbility.abc            ← Ability 字节码
        └── ...
```

## 5. 构建命令

### 5.1 全量构建

```bash
# 使用 hb 构建
hb build -f

# 使用 hvigor 构建
hvigor --mode module -p product=default assembleHap --strict-mode=true
```

### 5.2 单模块构建

```bash
# 构建 entry 模块
hvigor --mode module -p product=default assembleHap -p module=entry --strict-mode=true

# 构建 mobiledatasettings 模块
hvigor --mode module -p product=default assembleHap -p module=mobiledatasettings --strict-mode=true
```

### 5.3 清理构建

```bash
# 清理构建产物
hb clean

# 或手动删除
rm -rf out/
```

### 5.4 调试构建

```bash
# 调试模式构建
hvigor --mode module -p product=default assembleHap --debug --strict-mode=true

# 查看详细日志
hvigor --mode module -p product=default assembleHap --verbose --strict-mode=true
```

## 6. 签名配置

### 6.1 签名文件

| 文件 | 路径 | 用途 |
|-----|------|-----|
| 签名证书 | `signature/callui.p7b` | 应用签名 |
| 调试证书 | (IDE 自动生成) | 调试签名 |

### 6.2 签名配置

```json5
// build-profile.json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "default",
        "material": {
          "certAlias": "ohos_app",                    // 证书别名
          "keyAlias": "ohos_app",                     // 密钥别名
          "keyPassword": "xxxxxx",                    // 密钥密码
          "storeFile": "signature/callui.p7b",        // 证书文件
          "storePassword": "xxxxxx"                   // 证书密码
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "default"
      }
    ]
  }
}
```

## 7. 依赖管理

### 7.1 模块依赖

```
entry (entry)
├── depends on: common (har)
├── depends on: @ohos.telephony.call (系统 API)
├── depends on: @ohos.telephony.sim (系统 API)
├── depends on: @ohos.telephony.radio (系统 API)
├── depends on: @ohos.telephony.sms (系统 API)
├── depends on: @ohos.app.ability (系统 API)
├── depends on: @ohos.rpc (系统 API)
├── depends on: @ohos.commonEvent (系统 API)
└── depends on: @ohos.notification (系统 API)

mobiledatasettings (feature)
├── depends on: common (har)
├── depends on: @ohos.telephony.radio (系统 API)
└── depends on: @ohos.telephony.sim (系统 API)

common (har)
└── 无外部依赖
```

### 7.2 npm 依赖

```json5
// oh-package.json5
{
  "dependencies": {
    // 系统 API 由 SDK 提供，无需声明
  }
}
```

## 8. 常见问题

### Q1: 构建失败，提示权限错误

**错误信息**:
```
error: permission denied: ohos.permission.PLACE_CALL
```

**解决方案**:
1. 在 `module.json` 中正确声明权限
2. 确保权限已添加到 `requestPermissions` 列表

### Q2: HAP 安装失败

**错误信息**:
```
error: install failed due to different signatures
```

**解决方案**:
1. 检查签名证书是否一致
2. 确认 `certificate_profile` 配置正确

### Q3: 资源编译失败

**错误信息**:
```
error: resource not found: @media:app_icon
```

**解决方案**:
1. 检查 `resources` 目录结构
2. 确认 `element` 目录下有 `string.json` 和 `media` 目录

## 9. 相关文档

- [项目概述](01_Overview.md)
- [架构设计](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [安全评审](05_Security_Review.md)

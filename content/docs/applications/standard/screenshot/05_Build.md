# 构建配置

## 构建系统概述

| 项目 | 配置 |
|-----|------|
| 构建工具 | hvigor |
| API 类型 | stageMode |
| 编译 SDK | 23 |
| 目标 SDK | 23 |

## 构建配置结构

```
screenshot/
├── hvigorfile.js              # 根构建脚本 → appTasks
├── build-profile.json5         # 根构建配置
├── AppScope/                  # 应用全局配置
│   └── app.json5
├── common/                    # HAR 模块
│   ├── hvigorfile.js          # → hapTasks
│   ├── build-profile.json5
│   └── src/main/
├── features/screenshot/       # HAR 模块
│   ├── hvigorfile.js          # → hapTasks
│   ├── build-profile.json5
│   └── src/main/
└── product/phone/            # Entry HAP 模块
    ├── hvigorfile.js          # → hapTasks
    ├── build-profile.json5
    └── src/main/
```

## 根构建配置

**文件**: `build-profile.json5`

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "release",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23,
        "targetSdkVersion": 23,
        "runtimeOS": "OpenHarmony"
      }
    ],
    "signingConfigs": []
  },
  "modules": [
    {
      "name": "phone",
      "srcPath": "./product/phone",
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

**关键配置**:
- `signingConfig`: release
- `runtimeOS`: OpenHarmony
- `modules[0].name`: phone
- `modules[0].srcPath`: `./product/phone`

## 模块构建配置

### common 模块 (HAR)

**文件**: `common/build-profile.json5`

```json5
{
  "apiType": "stageMode",
  "targets": [{ "name": "default" }]
}
```

**hvigorfile.js**: `module.exports = require('@ohos/hvigor-ohos-plugin').hapTasks`

### features/screenshot 模块 (HAR)

**文件**: `features/screenshot/build-profile.json5`

```json5
{
  "apiType": "stageMode",
  "targets": [{ "name": "default" }]
}
```

### product/phone 模块 (Entry HAP)

**文件**: `product/phone/build-profile.json5`

```json5
{
  "apiType": "stageMode",
  "targets": [{ "name": "default" }]
}
```

**模块配置**: `product/phone/src/main/module.json5`

```json5
{
  "module": {
    "name": "phone",
    "type": "entry",
    "srcEntrance": "./ets/Application/AbilityStage.ts",
    "mainElement": "com.ohos.screenshot.ServiceExtAbility",
    "uiSyntax": "ets",
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages"
  }
}
```

## 模块类型

| 模块 | 类型 | 说明 |
|-----|------|------|
| product/phone | entry | 可安装的 HAP 包 |
| features/screenshot | har | 静态库，被 entry 引用 |
| common | har | 静态库，被其他模块引用 |

## 依赖关系

```
phone (entry)
├── depends: screenshot (har)
└── depends: common (har)

screenshot (har)
└── depends: common (har)

common (har)
└── 无外部依赖
```

**证据**:
- `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets:20-21` (导入 screenshot)
- `product/phone/src/main/ets/vm/ViewModel.ets:17` (导入 screenshot)
- `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:21` (导入 common)

## 签名配置

**签名文件位置**: `signature/screenshot.p7b`

**配置方式**: 在 `product/phone/build.gradle` 中配置 `signingConfigs`

**使用说明**:
1. 针对 `product/phone` 模块配置 `build.gradle` 中的 `signingConfigs`
2. 将 `signature/screenshot.p7b` 放到配置指定路径

**证据**: `README.md:40-43`

## 构建命令

### 本地构建

```bash
# 在项目根目录执行
hvigor --mode module -p product=default -p debuggable=false assembleHap
```

### 调试构建

```bash
hvigor --mode module -p product=default assembleHap --debug
```

### 构建特定模块

```bash
# 构建 phone 模块
hvigor --mode module -p product=default -p module=phone assembleHap
```

## 构建产物

### HAP 包

| 模块 | 产物路径 | 说明 |
|-----|---------|------|
| phone | `product/phone/build/default/outputs/default/phone.hap` | 主应用包 |

### HAR 包

| 模块 | 产物路径 | 说明 |
|-----|---------|------|
| screenshot | `features/screenshot/build/default/outputs/default/screenshot.har` | 截屏功能库 |
| common | `common/build/default/outputs/default/common.har` | 通用工具库 |

### 安装路径

构建产物将安装在设备上的：
- `/data/app/{bundleName}/` 目录
- 系统应用可能预装在 `/system/app/` 或 `/system/haps/`

## 资源文件

### 应用资源

| 目录 | 说明 |
|-----|------|
| `AppScope/resources/` | 全局应用资源 |
| `product/phone/src/main/resources/` | phone 模块资源 |
| `features/screenshot/src/main/resources/` | screenshot 模块资源 |

### 资源类型

| 目录 | 用途 |
|-----|------|
| `base/element/` | 元素资源 (string, color, float) |
| `base/media/` | 媒体资源 (图片) |
| `zh_CN/` | 中文本地化 |
| `en_US/` | 英文本地化 |

**证据**: `product/phone/src/main/resources/` 目录结构

## 页面配置

**文件**: `product/phone/src/main/resources/base/profile/main_pages.json`

```json5
{
  "src": [
    "pages/index"
  ]
}
```

## 依赖包配置

**文件**: `oh-package.json5`

```json5
{
  "modelVersion": "5.0.1",
  "devDependencies": {
    "@ohos/hypium": "1.0.9"
  },
  "dependencies": {
    "@ohos/hypium": "^1.0.9"
  }
}
```

## 构建配置特点

1. **Stage 模型**: 使用标准的 OpenHarmony Stage 应用模型
2. **模块化设计**: 功能拆分为多个 HAR 模块
3. **配置分离**: 模块配置独立，支持单独编译
4. **签名机制**: 支持 release 签名配置

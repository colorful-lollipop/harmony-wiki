# 构建系统

## 概述

`app_samples` 仓库使用 **hvigor** 作为构建工具，配置通过 `build-profile.json5` 文件管理。

> **证据**: 各示例目录下的 `build-profile.json5` 文件

## 构建配置

### build-profile.json5 结构

```json5
{
  "app": {
    "signingConfigs": [],           // 签名配置
    "products": [
      {
        "name": "default",          // 产品名称
        "signingConfig": "default", // 签名配置
        "arkTSVersion": "1.2",      // ArkTS 版本
        "compatibleSdkVersion": "6.0.0(20)",  // 兼容 SDK 版本
        "targetSdkVersion": "6.0.0(20)",     // 目标 SDK 版本
        "runtimeOS": "HarmonyOS",   // 运行时系统
        "buildOption": {
          "strictMode": {
            "caseSensitiveCheck": true,    // 大小写敏感检查
            "useNormalizedOHMUrl": true     // 规范化 OHM URL
          }
        }
      }
    ],
    "buildModeSet": [
      { "name": "debug" },          // 调试模式
      { "name": "release" }         // 发布模式
    ]
  },
  "modules": [
    {
      "name": "entry",              // 模块名称
      "srcPath": "./entry",         // 源码路径
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

| 配置项 | 说明 | 示例值 |
|-------|------|--------|
| `arkTSVersion` | ArkTS 语言版本 | "1.2" |
| `compatibleSdkVersion` | 兼容的 SDK 版本 | "6.0.0(20)" |
| `targetSdkVersion` | 目标 SDK 版本 | "6.0.0(20)" |
| `runtimeOS` | 运行时操作系统 | "HarmonyOS" |
| `strictMode.caseSensitiveCheck` | 大小写敏感检查 | true |
| `strictMode.useNormalizedOHMUrl` | 规范化 OHM URL | true |

## 模块配置

### module.json5 结构

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": [
      "default",
      "tablet"
    ],
    "deliveryWithInstall": true,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:entry_ability_desc",
        "icon": "$media:icon",
        "label": "$string:entry_ability_label",
        "startWindowIcon": "$media:icon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "continuable": false
      }
    ],
    "metadata": [
      {
        "name": "router",
        "items": [
          {
            "name": "pages/Index",
            "bundleData": []
          }
        ]
      }
    ]
  }
}
```

### 配置项说明

| 配置项 | 说明 |
|-------|------|
| `module.name` | 模块名称 |
| `module.type` | 模块类型（entry、feature） |
| `module.mainElement` | 主入口组件 |
| `module.deviceTypes` | 支持的设备类型 |
| `module.pages` | 页面路由配置 |
| `module.abilities` | Ability 配置列表 |

## 构建工具

### hvigor

hvigor 是 OpenHarmony 的构建系统，基于 Gradle 构建脚本。

```
hvigor/
├── hvigor.jar           # hvigor 核心库
└── hvigor-config.json5  # hvigor 配置
```

### hvigorfile.ts

```typescript
import { tasks, defaults } from '@ohos/hvigor';

export default {
  system: tasks.androidAware,  // 使用 Android 感知任务
  options: {
    // 构建选项
  },
  tasks: {
    // 自定义任务
  },
  defaults: {
    // 默认配置
  }
}
```

## 构建命令

### Debug 构建

```bash
# 使用 hvigorw 构建
./hvigorw assembleDebug --product default

# 或使用 hvigor
hvigor assembleDebug
```

### Release 构建

```bash
./hvigorw assembleRelease --product default
```

### 清理构建

```bash
./hvigorw clean
```

## 产物输出

### HAP 包结构

```
build/default/outputs/default/
└── entry-default-signed.hap
    ├── config.json              # 模块配置
    ├── resources.index          # 资源索引
    ├── libs/                    # 依赖库
    ├── classes.dex              # ArkTS 字节码
    └── modules/                 # 模块文件
```

### 产物安装路径

| 路径 | 说明 |
|-----|------|
| `/data/app/el2/100/base/{bundleName}/haps/entry/` | 应用沙箱目录 |
| `/system/app/{bundleName}/` | 系统应用目录（系统应用） |

## 代码检查

### code-linter.json5

```json5
{
  "exclude": ["node_modules", "build", "*.hsp"],
  "include": ["*.ets", "*.ts"],
  "rule": {
    "typescript": {
      "typescript-rules": {
        "no-unnecessary-type-constraint": "error",
        "no-empty-interface": "error"
      }
    }
  }
}
```

### 检查命令

```bash
./hvigorw lint
```

## 依赖管理

### oh-package.json5

```json5
{
  "name": "entry",
  "version": "1.0.0",
  "description": "OpenHarmony Project",
  "main": "",
  "author": "",
  "license": "ISC",
  "dependencies": {},
  "devDependencies": {}
}
```

## 构建问题排查

### 常见错误

| 错误 | 原因 | 解决方案 |
|-----|------|---------|
| SDK 版本不匹配 | compatibleSdkVersion 与安装 SDK 不符 | 检查 DevEco Studio SDK 版本 |
| ArkTS 版本不兼容 | 代码使用了更高版本语法 | 使用 ArkTS 1.2 语法 |
| 签名配置缺失 | release 构建缺少签名 | 配置签名文件 |
| 资源文件缺失 | 引用了不存在的资源 | 检查 resources 目录 |

### 调试方法

```bash
# 查看详细日志
./hvigorw assembleDebug --stacktrace

# 仅检查配置
./hvigorw tasks --info
```

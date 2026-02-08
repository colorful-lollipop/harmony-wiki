# 构建配置

> 本章档说明本项目的 GN Targets 构建配置（本项目使用 hvigor + Node.js）。

## 构建系统

| 构建工具 | 版本要求 | 说明 |
|----------|----------|------|
| **hvigor** | 任意稳定版本 | 基于 Node.js 的构建工具 |
| **Node.js** | 14+ | hvigor 运行时环境 |

## 构建入口

### 根构建配置

| 文件 | 用途 |
|------|------|
| `build-profile.json5` | 项目级构建配置 |
| `hvigorfile.js` | hvigor 构建脚本 |

**代码位置**: `build-profile.json5:1-38`

```json5
{
  "app": {
    "compileSdkVersion": 9,
    "compatibleSdkVersion": 9,
    "products": [
      {
        "name": "default",
        "signingConfig": "default"
      }
    ]
  },
  "modules": [
    {
      "name": "phone-wallpaper",
      "srcPath": "./product/phone",
      "targets": [
        {
          "name": "default",
          "applyToProducts": ["default"]
        }
      ]
    },
    {
      "name": "pad-wallpaper",
      "srcPath": "./product/pad",
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

## 模块配置

### phone-wallpaper 模块

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **name** | `phone-wallpaper` | 模块名 |
| **srcPath** | `./product/phone` | 源码路径 |
| **类型** | feature | 功能模块 |
| **设备类型** | phone | 仅手机 |

**代码位置**: `build-profile.json5:14-24`

### pad-wallpaper 模块

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **name** | `pad-wallpaper` | 模块名 |
| **srcPath** | `./product/pad` | 源码路径 |
| **类型** | feature | 功能模块 |
| **设备类型** | pad | 仅平板 |

**代码位置**: `build-profile.json5:26-36`

## 产品配置

### default 产品

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **name** | `default` | 产品名 |
| **signingConfig** | `default` | 签名配置 |

**代码位置**: `build-profile.json5:7-10`

### 产品级构建配置

| 产品 | 配置文件 |
|------|----------|
| phone | `product/phone/build-profile.json5` |
| pad | `product/pad/build-profile.json5` |

**代码位置**: `product/phone/build-profile.json5`

```json5
{
  "app": {
    "signingConfig": "default"
  },
  "modules": [
    {
      "name": "phone-wallpaper",
      "srcPath": "./src/main",
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

## 签名配置

### 签名配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **配置文件** | `signature/signatureProfile.json` | 签名配置 |
| **引用方式** | `signingConfig: "default"` | 产品配置引用 |

**代码位置**: `signature/signatureProfile.json`

## 构建目标 (Targets)

### 默认构建目标

| Target | 适用产品 | 输出 |
|--------|----------|------|
| **default** | default | phone-wallpaper.hap / pad-wallpaper.hap |

### 构建命令

```bash
# 构建 phone 版本
hvigor --mode module -p product=default -p module=phone-wallpaper assembleHap

# 构建 pad 版本
hvigor --mode module -p product=default -p module=pad-wallpaper assembleHap

# 同时构建所有模块
hvigor assembleHap
```

## 构建选项

### 常用构建选项

| 选项 | 说明 | 使用场景 |
|------|------|----------|
| `--mode module` | 模块构建模式 | 构建单个模块 |
| `-p product=<name>` | 指定产品 | 选择构建产品 |
| `-p module=<name>` | 指定模块 | 选择构建模块 |
| `assembleHap` | 构建 Hap 包 | 生成安装包 |

### SDK 版本配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **compileSdkVersion** | 9 | 编译 SDK 版本 |
| **compatibleSdkVersion** | 9 | 兼容 SDK 版本 |

## 依赖配置

### 模块间依赖

本项目为独立模块，**无模块间依赖**。

### 系统 API 依赖

| 依赖 | 用途 | 声明方式 |
|------|------|----------|
| `@ohos.wallpaper` | 壁纸服务 | import 语句 |
| `@ohos.window` | 窗口管理 | import 语句 |
| `@ohos.WallpaperExtension` | 壁纸扩展基类 | import 语句 |
| `@ohos.application.Ability` | 能力基类 | import 语句 |
| `@ohos.application.AbilityStage` | 能力舞台基类 | import 语句 |

## 构建产物配置

### 模块配置文件

| 文件 | 用途 |
|------|------|
| `product/phone/src/main/module.json5` | phone 模块配置 |
| `product/pad/src/main/module.json5` | pad 模块配置 |

**代码位置**: `product/phone/src/main/module.json5:1-35`

```json5
{
  "module": {
    "name": "phone-wallpaper",
    "type": "feature",
    "srcEntrance": "./ets/Application/AbilityStage.ts",
    "mainElement": "MainAbility",
    "deviceTypes": ["phone"],
    "deliveryWithInstall": true,
    "pages": "$profile:main_pages"
  }
}
```

## 相关文档

- [编译产物](07_Artifacts.md)
- [目录结构](02_Directory_Structure.md)
- [问题定位](09_Troubleshooting.md)

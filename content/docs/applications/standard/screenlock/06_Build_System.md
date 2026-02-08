# 06. 构建系统

> 目的: 详细说明ScreenLock的构建配置和流程  
> 适用范围: 构建工程师、开发者

---

## 1. 构建系统概述

### 1.1 构建工具

| 工具 | 版本 | 用途 |
|------|------|------|
| Hvigor | 2.x+ | 构建编排工具 |
| Node.js | 14.x+ | 运行环境 |
| OpenHarmony SDK | 9+ | 编译SDK |

### 1.2 构建配置文件

| 文件 | 用途 |
|------|------|
| `build-profile.json5` | 项目级构建配置，定义模块和SDK版本 |
| `hvigorfile.js` | 项目级Hvigor脚本 |
| `oh-package.json5` | 包管理配置 |
| `entry/hvigorfile.js` | Entry模块Hvigor脚本 |
| `common/oh-package.json5` | Common模块包配置 |
| `*/build-profile.json5` | 模块级构建配置 |

---

## 2. build-profile.json5详解

### 2.1 根配置

**文件**: `/build-profile.json5`

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
    // 13个模块定义
  ]
}
```

**配置项说明**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `products[0].name` | "default" | 产物名称 |
| `products[0].signingConfig` | "default" | 签名配置 |
| `products[0].compileSdkVersion` | 23 | 编译SDK版本 |
| `products[0].compatibleSdkVersion` | 23 | 兼容SDK版本 |

### 2.2 模块定义

**全部模块** (13个):

```json5
{
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [{ "name": "default", "applyToProducts": ["default"] }]
    },
    {
      "name": "pc",
      "srcPath": "./product/pc",
      "targets": [{ "name": "default", "applyToProducts": ["default"] }]
    },
    {
      "name": "phone",
      "srcPath": "./product/phone",
      "targets": [{ "name": "default", "applyToProducts": ["default"] }]
    },
    {
      "name": "batterycomponent",
      "srcPath": "./features/batterycomponent"
    },
    {
      "name": "clockcomponent",
      "srcPath": "./features/clockcomponent"
    },
    {
      "name": "datetimecomponent",
      "srcPath": "./features/datetimecomponent"
    },
    {
      "name": "noticeitem",
      "srcPath": "./features/noticeitem"
    },
    {
      "name": "screenlock",
      "srcPath": "./features/screenlock"
    },
    {
      "name": "shortcutcomponent",
      "srcPath": "./features/shortcutcomponent"
    },
    {
      "name": "signalcomponent",
      "srcPath": "./features/signalcomponent"
    },
    {
      "name": "wallpapercomponent",
      "srcPath": "./features/wallpapercomponent"
    },
    {
      "name": "wificomponent",
      "srcPath": "./features/wificomponent"
    },
    {
      "name": "common",
      "srcPath": "./common"
    }
  ]
}
```

**模块类型**:

| 类型 | 数量 | 模块 | 说明 |
|------|------|------|------|
| entry | 1 | entry | 应用入口 |
| feature | 2 | phone, pc | 产品形态 |
| feature | 9 | features/* | 功能组件 |
| feature | 1 | common | 公共模块 |

---

## 3. 模块配置详解

### 3.1 Entry模块配置

**文件**: `entry/src/main/module.json5`

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntrance": "./ets/Application/AbilityStage.ts",
    "description": "$string:entry_desc",
    "mainElement": "MainAbility",
    "deviceTypes": ["phone", "tablet"],
    "pages": "$profile:main_pages",
    "uiSyntax": "ets",
    "abilities": [
      {
        "srcEntry": "./ets/MainAbility/MainAbility.ts",
        "name": "MainAbility",
        "description": "$string:MainAbility_desc",
        "icon": "$media:icon",
        "label": "$string:MainAbility_label",
        "visible": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ]
  }
}
```

**关键配置**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `type` | "entry" | Entry类型模块 |
| `mainElement` | "MainAbility" | 主Ability |
| `deviceTypes` | ["phone", "tablet"] | 支持的设备 |
| `uiSyntax` | "ets" | UI语法 |

### 3.2 Phone模块配置

**文件**: `product/phone/src/main/module.json5`

```json5
{
  "module": {
    "name": "phone",
    "type": "feature",
    "srcEntrance": "./ets/Application/AbilityStage.ts",
    "mainElement": "com.ohos.systemui.screenlock.ServiceExtAbility",
    "deviceTypes": ["phone"],
    "requestPermissions": [
      // 21个权限声明
    ],
    "extensionAbilities": [
      {
        "name": "com.ohos.systemui.screenlock.ServiceExtAbility",
        "srcEntrance": "./ets/ServiceExtAbility/ServiceExtAbility.ts",
        "type": "service"
      }
    ]
  }
}
```

**关键配置**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `type` | "feature" | Feature类型模块 |
| `mainElement` | "ServiceExtAbility" | 主Ability |
| `deviceTypes` | ["phone"] | 仅手机 |
| `requestPermissions` | 21个权限 | 系统权限声明 |
| `extensionAbilities` | ServiceExtAbility | 服务扩展Ability |

---

## 4. Hvigor构建脚本

### 4.1 项目级脚本

**文件**: `hvigorfile.js`

```javascript
module.exports = {
    system: harTasks,  // HAR模块构建任务
    plugins: []
}
```

### 4.2 模块级脚本示例

**文件**: `common/hvigorfile.js`

```javascript
const { harTasks } = require('@ohos/hvigor-ohos-plugin');

module.exports = {
    system: harTasks,  // HAR构建任务
    plugins: []
}
```

**构建任务类型**:

| 任务 | 用途 | 适用模块 |
|------|------|----------|
| `harTasks` | HAR库构建 | common, features/* |
| `hapTasks` | HAP包构建 | entry, phone, pc |

---

## 5. 包管理配置

### 5.1 项目级包配置

**文件**: `oh-package.json5`

```json5
{
  "license": "ISC",
  "devDependencies": {
    "@ohos/hypium": "1.0.6"
  },
  "name": "systemui",
  "description": "example description",
  "version": "1.0.0"
}
```

### 5.2 模块级包配置

**文件**: `common/oh-package.json5`

```json5
{
  "name": "@ohos/common",
  "version": "1.0.0",
  "description": "Common utilities for ScreenLock",
  "main": "index.ts"
}
```

**包名规范**:

| 模块 | 包名 | 导入方式 |
|------|------|----------|
| common | `@ohos/common` | `import { X } from '@ohos/common'` |
| screenlock | `@ohos/screenlock` | `import { X } from '@ohos/screenlock'` |
| 其他features | `@ohos/{name}` | 类似 |

---

## 6. 构建流程

### 6.1 构建命令

```bash
# 安装依赖
npm install

# 编译项目
hvigor assemble

# 编译并签名
hvigor assemble --mode=release

# 清理构建
hvigor clean
```

### 6.2 构建流程图

```
┌─────────────────────────────────────────────────────────────┐
│                      构建流程                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 依赖解析                                                  │
│     └── 解析oh-package.json5依赖                              │
│                                                             │
│  2. 模块编译（并行）                                           │
│     ├── Common模块 (HAR)                                      │
│     │   └── 编译TypeScript/ArkTS                              │
│     │                                                         │
│     ├── Features模块 (HAR)                                    │
│     │   └── 编译TypeScript/ArkTS                              │
│     │                                                         │
│     └── Product模块 (HAP)                                     │
│         ├── 编译TypeScript/ArkTS                              │
│         └── 打包HAP                                           │
│                                                             │
│  3. 资源处理                                                  │
│     └── 编译资源文件（图片、字符串等）                          │
│                                                             │
│  4. 签名打包                                                  │
│     └── 使用签名文件签名HAP包                                  │
│                                                             │
│  5. 产物输出                                                  │
│     └── build/default/outputs/                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 构建产物

| 产物类型 | 文件扩展名 | 说明 |
|----------|------------|------|
| HAP | `.hap` | 应用包（entry, phone, pc） |
| HAR | `.har` | 库包（common, features） |

---

## 7. 签名配置

### 7.1 签名文件

**位置**: `signature/systemui.p7b`

### 7.2 签名配置

签名配置在OpenHarmony SDK的签名工具中配置，包括：

| 配置项 | 说明 |
|--------|------|
| `keyStore` | 密钥库文件 |
| `keyAlias` | 密钥别名 |
| `signAlg` | 签名算法（如SHA256withECDSA） |
| `storePassword` | 密钥库密码 |
| `keyPassword` | 密钥密码 |

---

## 8. 构建产物目录

### 8.1 输出目录结构

```
build/
└── default/
    ├── outputs/
    │   ├── default/           # 默认产物
    │   │   ├── entry.hap     # Entry HAP包
    │   │   ├── phone.hap     # Phone HAP包
    │   │   └── pc.hap        # PC HAP包
    │   └── common/            # Common HAR包
    │       └── common.har
    ├── intermediates/         # 中间产物
    │   ├── entry/
    │   ├── phone/
    │   └── pc/
    └── logs/                  # 构建日志
```

---

## 9. 构建优化

### 9.1 增量构建

Hvigor支持增量构建，只编译变更的文件。

### 9.2 并行构建

模块间并行编译，加快构建速度。

### 9.3 缓存

启用编译缓存，避免重复编译未变更的代码。

---

## 10. 常见问题

### 10.1 构建失败

| 问题 | 原因 | 解决 |
|------|------|------|
| SDK版本不匹配 | compileSdkVersion与实际SDK不符 | 检查SDK安装和版本配置 |
| 依赖找不到 | oh-package.json5配置错误 | 检查模块名和路径 |
| 签名失败 | 签名配置错误 | 检查签名文件和密码 |

### 10.2 运行时错误

| 问题 | 原因 | 解决 |
|------|------|------|
| 权限拒绝 | 权限未声明或未被授予 | 检查module.json5权限声明 |
| 找不到Ability | Ability路径配置错误 | 检查srcEntry路径 |

---

*关键结论: ScreenLock使用Hvigor构建系统，包含13个模块（1 entry + 2 product + 9 features + 1 common），输出HAP和HAR包。构建配置主要集中在build-profile.json5和module.json5中。*

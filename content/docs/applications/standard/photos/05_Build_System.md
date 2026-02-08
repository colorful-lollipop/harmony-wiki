# 构建系统

## 1. 构建工具链

### 1.1 构建工具

| 工具 | 版本/来源 | 用途 |
|------|-----------|------|
| **hvigor** | `@ohos/hvigor-ohos-plugin` | 构建入口 |
| **DevEco Studio** | - | IDE 构建 |
| **Node.js** | - | JavaScript 运行时 |

> **证据**: `hvigorfile.js:16` - `require('@ohos/hvigor-ohos-plugin').appTasks`

### 1.2 构建命令

```bash
# Debug 构建
hvigor assembleHap --product default --debug

# Release 构建
hvigor assembleHap --product default --release

# 清理构建
hvigor clean
```

---

## 2. 构建配置文件

### 2.1 根目录配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `build-profile.json5` | `/build-profile.json5` | 构建配置（签名、SDK 版本） |
| `hvigorfile.js` | `/hvigorfile.js` | hvigor 构建脚本 |
| `module.json5` | 各模块目录 | 单个模块配置 |

### 2.2 配置文件详解

#### 2.2.1 build-profile.json5

```json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "debug",
        "material": {
          "certpath": "signature/OpenHarmonyApplication.cer",
          "storePassword": "...",
          "keyAlias": "OpenHarmony Application Release",
          "keyPassword": "...",
          "profile": "signature/photos.p7b",
          "signAlg": "SHA256withECDSA",
          "storeFile": "signature/OpenHarmony.p12"
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "debug",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23
      }
    ]
  },
  "modules": [
    {
      "name": "photos_common",
      "srcPath": "./common",
      "targets": [...]
    },
    {
      "name": "photos_browser",
      "srcPath": "./feature/browser",
      "targets": [...]
    },
    // ... 其他模块
  ]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **signingConfigs.name** | `debug` | 调试签名配置 |
| **signingConfigs.signAlg** | `SHA256withECDSA` | 签名算法 |
| **products.name** | `default` | 产品名 |
| **products.compileSdkVersion** | `23` | 编译 SDK 版本 |
| **products.compatibleSdkVersion** | `23` | 兼容 SDK 版本 |

> **证据**: `build-profile.json5:1-122`

---

#### 2.2.2 hvigorfile.js

```javascript
module.exports = require('@ohos/hvigor-ohos-plugin').appTasks;
```

| 配置项 | 说明 |
|--------|------|
| `@ohos/hvigor-ohos-plugin` | OpenHarmony hvigor 插件 |

> **证据**: `hvigorfile.js:16`

---

## 3. 模块配置

### 3.1 模块清单

| 模块名 | 类型 | srcPath | 目标设备 |
|--------|------|---------|----------|
| `photos_common` | HAR | `./common` | default, tablet |
| `photos_browser` | HAR | `./feature/browser` | default, tablet |
| `photos_editor` | HAR | `./feature/editor` | default |
| `photos_formAbility` | HAR | `./feature/formAbility` | default |
| `photos_thirdselect` | HAR | `./feature/thirdselect` | default |
| `photos_timeline` | HAR | `./feature/timeline` | default |
| `phone_photos` | entry | `./product/phone` | default |

> **证据**: `build-profile.json5:34-121`

---

### 3.2 模块配置详情

#### 3.2.1 photos_common (HAR)

```json5
{
  "name": "photos_common",
  "type": "har",
  "deviceTypes": ["default", "tablet"]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **type** | `har` | HAR 静态共享包 |
| **deviceTypes** | `["default", "tablet"]` | 支持设备 |

---

#### 3.2.2 phone_photos (entry)

```json5
{
  "name": "phone_photos",
  "type": "entry",
  "srcEntry": "./ets/Application/AbilityStage.ts",
  "mainElement": "MainAbility",
  "deviceTypes": ["default", "tablet"],
  "deliveryWithInstall": true,
  "installationFree": false,
  "pages": "$profile:main_pages"
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **type** | `entry` | 入口模块 |
| **srcEntry** | `./ets/Application/AbilityStage.ts` | 源码入口 |
| **mainElement** | `MainAbility` | 主 Ability |
| **deliveryWithInstall** | `true` | 随安装分发 |
| **installationFree** | `false` | 非免安装 |

> **证据**: `product/phone/src/main/module.json5:2-14`

---

## 4. 签名配置

### 4.1 签名文件

| 文件 | 路径 | 用途 |
|------|------|------|
| `OpenHarmonyApplication.cer` | `signature/` | 调试证书 |
| `OpenHarmony.p12` | `signature/` | 密钥库 |
| `photos.p7b` | `signature/` | Profile 文件 |

### 4.2 签名参数

| 参数 | 值 | 说明 |
|------|-----|------|
| **signAlg** | `SHA256withECDSA` | 签名算法 |
| **keyAlias** | `OpenHarmony Application Release` | 密钥别名 |
| **storePassword** | 密文 | 密钥库密码 |

> **证据**: `build-profile.json5:8-21`

---

## 5. 依赖管理

### 5.1 模块间依赖

```
phone_photos (entry)
    │
    ├── photos_common (har) ─── 必需
    ├── photos_browser (har) ─── 必需
    ├── photos_editor (har) ─── 必需
    ├── photos_formAbility (har) ─── 必需
    ├── photos_thirdselect (har) ─── 必需
    └── photos_timeline (har) ─── 必需
```

### 5.2 外部依赖

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| `@ohos/hypium` | `oh-package.json5` | 测试框架 |

> **证据**: `oh-package.json5:5`

---

## 6. 设备适配

### 6.1 设备类型

| 类型 | 说明 | 支持模块 |
|------|------|----------|
| `default` | 标准设备（手机） | 全部模块 |
| `tablet` | 平板设备 | common, browser, phone |

### 6.2 屏幕适配

| 适配点 | 实现方式 |
|--------|----------|
| 屏幕尺寸 | `ScreenManager` |
| 断点系统 | `BreakPointSystem` |
| 布局适配 | 基于 `gridColumns` 的响应式布局 |

> **证据**: `MainAbility.ts:29` - `ScreenManager.getInstance()`

---

## 7. 构建变体

### 7.1 产品变体

| 变体名 | 签名配置 | SDK 版本 |
|--------|----------|----------|
| `default` | debug | 23 |
| `tablet` | debug | 23 |

### 7.2 构建类型

| 类型 | 说明 | 用途 |
|------|------|------|
| debug | 调试构建 | 开发调试 |
| release | 发布构建 | 生产发布 |

---

## 8. 常见构建问题

### 8.1 构建失败排查

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 签名验证失败 | 签名文件缺失 | 配置签名文件 |
| SDK 版本不匹配 | compileSdkVersion 错误 | 检查 build-profile.json5 |
| 模块找不到 | srcPath 配置错误 | 检查模块路径 |
| 资源找不到 | resources 目录缺失 | 检查资源文件 |

### 8.2 日志查看

```bash
# 查看构建日志
hvigor assembleHap --product default --debug --verbose

# 查看详细日志
hvigor --stacktrace
```

---

## 9. 相关文档

| 文档 | 说明 |
|------|------|
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 问题排查 |
| [README_zh.md](../README_zh.md) | 原始开发说明 |

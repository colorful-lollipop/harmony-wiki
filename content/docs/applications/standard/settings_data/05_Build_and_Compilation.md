# 构建与编译

## 概述

SettingsData 使用 OpenHarmony 官方的 **hvigor** 构建系统进行编译，构建产物为 HAP（Harmony Ability Package）应用包。

---

## 构建环境

### SDK 版本要求

| 配置项 | 值 | 说明 |
|--------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| targetSdkVersion | 23 | 目标 SDK 版本 |

**证据**: `build-profile.json5:9-11`

```json
{
  "products": [
    {
      "name": "default",
      "compileSdkVersion": 23,
      "compatibleSdkVersion": 23,
      "targetSdkVersion": 23
    }
  ]
}
```

### 构建工具

| 工具 | 版本 | 说明 |
|------|------|------|
| hvigor | - | OpenHarmony 构建系统 |
| Node.js | - | hvigor 运行时依赖 |
| ArkTS 编译器 | - | 编译 ets 文件 |

---

## 构建配置

### 构建入口文件

| 文件 | 用途 |
|------|------|
| `hvigorfile.js` | hvigor 构建脚本入口 |
| `build-profile.json5` | 项目构建配置 |
| `hvigor/` | hvigor 工具目录 |

**hvigorfile.js** (`hvigorfile.js:1-14`):

```javascript
// 导入 hvigor 模块
module.exports = require('@ohos/hvigor');

// 定义模块导出
export default {
  system: plugin => plugin,
  app: plugin => plugin,
}
```

### 构建产物配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 模块名 | entry | 主模块名称 |
| 模块类型 | entry | 入口模块 |
| srcEntrance | ./ets/Application/DataAbilityStage.ts | 源码入口 |
| deliveryWithInstall | true | 随安装包交付 |
| installationFree | false | 不支持免安装 |

**证据**: `entry/src/main/module.json5:3-14`

---

## 构建命令

### 常用命令

```bash
# 构建 HAP 包（调试版本）
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default

# 构建 HAP 包（发布版本）
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default --release

# 清理构建产物
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data clean

# 查看帮助
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data --help
```

### 构建选项

| 选项 | 说明 |
|------|------|
| `--product <name>` | 指定产品配置 |
| `--debug` | 调试版本（默认） |
| `--release` | 发布版本 |
| `--analyze` | 分析构建依赖 |
| `--daemon` | 使用守护进程构建 |

---

## 构建产物

### HAP 包结构

构建生成的 HAP 包位于：

```
build/default/outputs/default/
└── entry-default-signed.hap    # 签名的 HAP 包
```

### HAP 包内容

HAP 包是一个压缩文件，包含以下内容：

```
entry-default-signed.hap
├── ets/                        # ArkTS 字节码
│   ├── index.abc              # 入口字节码
│   └── ...
├── resources/                  # 资源文件
│   ├── indexer.json
│   └── ...
├── module.json                 # 模块配置
├── pack.info                   # 打包信息
└── signature/                  # 签名信息
    ├── cert.pem
    ├── chain.pem
    └── sig_data
```

### 产物映射

| 源文件/目录 | 构建产物位置 |
|-------------|--------------|
| `entry/src/main/ets/` | `ets/` 目录（字节码） |
| `entry/src/main/resources/` | `resources/` 目录 |
| `entry/src/main/module.json5` | `module.json` |
| 签名文件 | `signature/` 目录 |

---

## 签名配置

### 签名配置项

| 配置项 | 值 | 说明 |
|--------|-----|------|
| signingConfigs | [] | 签名配置列表 |
| signingConfig | default | 默认签名配置 |

**证据**: `build-profile.json5:3-4`

### 签名文件位置

| 文件 | 用途 |
|------|------|
| `signature/` | 签名密钥和证书目录 |

---

## 模块配置详解

### module.json5 配置

**证据**: `entry/src/main/module.json5`

```json
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntrance": "./ets/Application/DataAbilityStage.ts",
    "description": "$string:entry_desc",
    "mainElement": "MainAbility",
    "deviceTypes": ["default", "tablet"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "metadata": [
      {
        "name": "ArkTSPartialUpdate",
        "value": "false"
      }
    ],
    "uiSyntax": "ets",
    "extensionAbilities": [
      {
        "srcEntrance": "./ets/DataAbility/DataExtAbility.ets",
        "name": "DataExtAbility",
        "icon": "$media:icon",
        "description": "$string:description_datashareextability",
        "type": "dataShare",
        "uri": "datashare://com.ohos.settingsdata.DataAbility",
        "visible": true,
        "writePermission": "ohos.permission.MANAGE_SECURE_SETTINGS",
        "metadata": [{"name": "ohos.extension.dataShare", "resource": "$profile:data_share_config"}]
      },
      {
        "name": "UserChangeStaticSubscriber",
        "type": "staticSubscriber",
        "visible": true,
        "description": "UserChangeStaticSubscriber",
        "icon": "$media:icon",
        "metadata": [{
          "name": "ohos.extension.staticSubscriber",
          "resource": "$profile:static_subscriber_config"
        }],
        "srcEntrance": "./ets/StaticSubscriber/UserChangeStaticSubscriber.ets"
      }
    ]
  }
}
```

### ExtensionAbility 配置说明

| ExtensionAbility | 类型 | URI | 权限 | 入口文件 |
|------------------|------|-----|------|----------|
| DataExtAbility | dataShare | `datashare://com.ohos.settingsdata.DataAbility` | `MANAGE_SECURE_SETTINGS` | `DataAbility/DataExtAbility.ets` |
| UserChangeStaticSubscriber | staticSubscriber | - | - | `StaticSubscriber/UserChangeStaticSubscriber.ets` |

---

## 资源编译配置

### 资源文件目录

```
entry/src/main/resources/
├── base/
│   ├── element/
│   │   ├── color.json          # 颜色资源
│   │   └── string.json         # 字符串资源
│   ├── media/
│   │   └── icon.png            # 图标资源
│   └── profile/
│       ├── data_share_config.json    # DataShare 配置
│       ├── main_pages.json           # 页面配置
│       └── static_subscriber_config.json  # 静态订阅配置
└── rawfile/
    └── default_settings.json    # 默认设置值
```

### 资源编译说明

| 资源类型 | 编译方式 |
|----------|----------|
| element/*.json | 编译为二进制格式 |
| media/* | 直接复制 |
| profile/*.json | 编译为二进制格式 |
| rawfile/* | 直接复制 |

---

## 构建优化建议

### 1. 增量构建

hvigor 支持增量构建，只重新编译修改过的文件：

```bash
# 使用增量构建（默认行为）
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default
```

### 2. 并行构建

hvigor 支持并行任务执行：

```bash
# 指定并行度
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default --parallel 4
```

### 3. 构建缓存

```bash
# 清理缓存并重新构建
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data clean
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default
```

---

## 常见构建问题

### 问题 1: SDK 版本不匹配

**现象**: 编译错误，提示 SDK 版本不兼容

**解决方案**: 确保 `compileSdkVersion` 与实际安装的 SDK 版本一致

```bash
# 查看已安装的 SDK 版本
echo "检查 SDK 版本: 23"
```

### 问题 2: 签名配置缺失

**现象**: 构建 HAP 时提示签名错误

**解决方案**: 配置有效的签名文件

1. 在 AppScope/resources/base/profile/ 下配置签名
2. 或使用默认调试签名

### 问题 3: 资源文件路径错误

**现象**: 运行时找不到资源文件

**解决方案**: 检查 `module.json5` 中的资源路径配置

```json
"icon": "$media:icon"  // 引用 resources/base/media/icon.png
```

---

## 构建产物验证

### 验证 HAP 包

```bash
# 解压 HAP 包检查内容
unzip entry-default-signed.hap -d hap_contents/
ls -la hap_contents/
```

### 验证签名

```bash
# 使用 hapverify 工具验证签名
hapverify entry-default-signed.hap
```

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [01_Project_Overview.md](./01_Project_Overview.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

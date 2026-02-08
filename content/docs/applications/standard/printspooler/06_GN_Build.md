# 构建配置

## 目的

本文档说明 PrintSpooler 项目的 Hvigor 构建配置，包括模块列表、配置参数和依赖关系。

## 适用范围

本文档适用于：

- DevOps 工程师
- 构建系统维护人员
- 需要修改构建配置的开发者

**注意**：项目使用 Hvigor 构建系统，而非 GN。因此本文档不包含 BUILD.gn 或 .gni 文件的内容。

## 关键结论

1. **构建系统**：Hvigor
2. **4 个模块**：entry（HAP）、common（HAR）、ippPrint（HAR）、driverEntry（Feature）
3. **SDK 版本**：API 18（编译），API 11（兼容）
4. **产品配置**：default, tablet

---

## 根构建配置

### build-profile.json5

**路径**：根目录 `build-profile.json5`

**配置内容**：

```json5
{
  "app": {
    "signingConfigs": [],
    "products": [
      {
        "name": "default",
        "compileSdkVersion": 18,
        "compatibleSdkVersion": 11,
        "runtimeOS": "OpenHarmony"
      },
      {
        "name": "tablet",
        "compileSdkVersion": 18,
        "compatibleSdkVersion": 11,
        "runtimeOS": "OpenHarmony"
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
      "name": "common",
      "srcPath": "./common"
    },
    {
      "name": "ippPrint",
      "srcPath": "./feature/ippPrint"
    },
    {
      "name": "driverEntry",
      "srcPath": "./driverEntry",
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

**关键参数**：
- `compileSdkVersion`: 18 - 编译使用的 SDK 版本
- `compatibleSdkVersion`: 11 - 兼容的最低 SDK 版本
- `runtimeOS`: "OpenHarmony" - 运行时系统

**证据**：根目录 `build-profile.json5`

### oh-package.json5

**路径**：根目录 `oh-package.json5`

**配置内容**：

```json5
{
  "modelVersion": "5.0.2",
  "license": "ISC",
  "devDependencies": {
    "@ohos/hypium": "1.0.21"
  },
  "name": "hwprintspooler",
  "description": "example description",
  "repository": {},
  "version": "1.0.0",
  "dependencies": {}
}
```

**证据**：根目录 `oh-package.json5`

---

## 模块配置

### Entry 模块

**类型**：HAP（Harmony Ability Package）

**配置文件**：`entry/build-profile.json5`

**模块配置**：`entry/src/main/module.json5`

#### Ability 配置

| Ability | 类型 | 路径 | 可见性 |
|---------|------|------|--------|
| MainAbility | UIExtensionAbility | srcEntry: "./ets/MainAbility/MainAbility.ets" | visible: false |
| JobManagerAbility | UIExtensionAbility | srcEntry: "./ets/MainAbility/JobManagerAbility.ts" | visible: false |

**证据**：`entry/src/main/module.json5:24-45`

#### ExtensionAbility 配置

| ExtensionAbility | 类型 | 导出 | 权限 |
|----------------|------|------|------|
| PrintExtension | PrintExtensionAbility | type: "print", visible: false | - |
| PrintServiceExtAbility | sysDialog/print | exported: true | ohos.permission.PRINT |

**证据**：`entry/src/main/module.json5:47-64`

#### 权限声明

共 9 个权限（详见 [安全风险评审](08_Security_Review.md)）

**证据**：`entry/src/main/module.json5:66-161`

---

### Common 模块

**类型**：HAR（Harmony Archive）

**配置文件**：`common/build-profile.json5`

**模块配置**：`common/src/main/module.json5`

**配置内容**：

```json5
{
  "module": {
    "name": "common",
    "type": "har",
    "deviceTypes": ["default", "tablet"]
  }
}
```

**证据**：`common/src/main/module.json5`

---

### IPPPrint 模块

**类型**：HAR（Harmony Archive）

**配置文件**：`feature/ippPrint/build-profile.json5`

**模块配置**：`feature/ippPrint/src/main/module.json5`

**配置内容**：

```json5
{
  "module": {
    "name": "ippPrint",
    "type": "har",
    "deviceTypes": ["default", "tablet"]
  }
}
```

**证据**：`feature/ippPrint/src/main/module.json5`

---

### DriverEntry 模块

**类型**：Feature

**配置文件**：`driverEntry/build-profile.json5`

**模块配置**：`driverEntry/src/main/module.json5`

#### ExtensionAbility 配置

| ExtensionAbility | 类型 | 元数据 |
|----------------|------|--------|
| DriverExtensionAbility | Driver (type="driver") | desc: cupsFilter, cupsPpd, cupsBackend, saneBackend |

**元数据配置**（证据：`driverEntry/src/main/module.json5:22-50`）：

```json5
"metadata": [
  {
    "name": "desc",
    "value": "xxx printer driver"
  },
  {
    "name": "vendor",
    "value": "xx company"
  },
  {
    "name": "cupsFilter",
    "value": "/print_service/cups/serverbin/filter",
    "resource": "/libs/arm64-v8a/rastertopwg"
  },
  {
    "name": "cupsPpd",
    "value": "/print_service/cups/datadir/model",
    "resource": "/libs/arm64-v8a/HUAWEI_PixLab_xxx.ppd"
  },
  {
    "name": "saneBackend",
    "value": "/print_service/sane/backend",
    "resource": "/libs/arm64-v8a/libsane-pantumxxx.so"
  },
  {
    "name": "cupsBackend",
    "value": "/print_service/cups/serverbin/backend",
    "resource": "/libs/arm64-v8a/lpd"
  }
]
```

---

## 模块依赖关系

### Entry 模块依赖

**依赖配置**：`entry/oh-package.json5`

```json5
{
  "name": "entry",
  "version": "1.0.0",
  "description": "description",
  "main": "",
  "author": "",
  "license": "",
  "dependencies": {
    "@ohos/common": "file:../common",
    "@ohos/ippprint": "file:../feature/ippPrint"
  }
}
```

**依赖**：
- `@ohos/common`: 本地文件依赖 `../common`
- `@ohos/ippprint`: 本地文件依赖 `../feature/ippPrint`

**证据**：`entry/oh-package.json5`

### IPPPrint 模块依赖

**依赖配置**：`feature/ippPrint/oh-package.json5`

**依赖**：
- `@ohos/common`: 本地文件依赖 `../common`

**证据**：`feature/ippPrint/oh-package.json5`

### DriverEntry 模块依赖

**依赖配置**：`driverEntry/oh-package.json5`

**依赖**：无

**证据**：`driverEntry/oh-package.json5`

---

## 构建产物类型

| 模块 | 类型 | 产物文件 | 说明 |
|------|------|---------|------|
| entry | HAP | entry-default.hap | 应用包（安装到系统） |
| common | HAR | common.har | 公共库（被 entry 和 ippPrint 引用） |
| ippPrint | HAR | ippPrint.har | IPP 功能库（被 entry 引用） |
| driverEntry | Feature | driverEntry-default.hap | 驱动扩展包 |

**证据**：根据模块类型推断

---

## 产品配置

| 产品名称 | 用途 | 应用模块 |
|---------|------|---------|
| default | 默认设备（手机等） | entry (default target) |
| tablet | 平板设备 | entry (default target) |

**证据**：`build-profile.json5:5-20`

---

## 构建命令

### 标准构建

```bash
# 构建 default 产品
hvigorw --mode module -p module=entry@default

# 构建 tablet 产品
hvigorw --mode module -p module=entry@tablet
```

### 清理构建

```bash
# 清理构建产物
hvigorw clean
```

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 模块组织
- [编译产物](07_Build_Artifacts.md) - 输出文件和安装路径
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用

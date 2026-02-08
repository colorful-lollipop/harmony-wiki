# 构建系统 (GN)

## 目的

本文档描述DLP Manager的构建系统配置，包括GN目标、编译产物和构建流程。

## 适用范围

- 需要进行构建配置的开发人员
- 需要了解编译产物的运维人员
- 进行CI/CD配置的工程师

---

## 构建系统概览

DLP Manager使用OpenHarmony的GN（Generate Ninja）构建系统。

### 构建配置文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 主构建脚本（`entry/src/main/module.json`） |
| `bundle.json` | 项目元数据和组件定义（`bundle.json`） |
| `build-profile.json5` | 编译配置（`build-profile.json5`） |
| `signature/dlpm.gni` | 签名配置导入（`signature/dlpm.gni`） |

---

## GN目标分析

### 1. 主目标: dlp_manager

**定义位置**: `BUILD.gn:17-39`

```gn
ohos_app("dlp_manager") {
  deps = [
    ":dlp_manager_js_assets",
    ":dlp_manager_resources",
  ]
  publicity_file = "publicity.xml"
  js_build_mode = "release"
  certificate_profile = "signature/dlpmanager.p7b"
  hap_name = "dlp_manager"
  part_name = "dlp_manager"
  subsystem_name = "applications"
  module_install_dir = "app/com.ohos.dlpmanager"
  sdk_home = "//prebuilts/ohos-sdk/linux"
  sdk_type_name = [ "sdk.dir" ]
  assemble_type = "assembleHap"
  build_level = "module"
  build_modules = [ "entry" ]
}
```

#### 目标属性

| 属性 | 值 | 说明 |
|------|-----|------|
| 目标名 | dlp_manager | HAP包名称 |
| 目标类型 | ohos_app | OpenHarmony应用目标 |
| 子系统 | applications | 所属子系统 |
| 部件名 | dlp_manager | 部件名称 |
| 安装路径 | app/com.ohos.dlpmanager | 系统安装路径 |
| 构建类型 | assembleHap | 构建HAP包 |
| SDK | //prebuilts/ohos-sdk/linux | SDK路径 |
| 签名配置 | signature/dlpmanager.p7b | 系统签名证书 |

#### 依赖关系

```
dlp_manager (主目标)
    ├── dlp_manager_js_assets (JS资源)
    └── dlp_manager_resources (资源文件)
            └── dlp_manager_app_profile (应用配置)
```

### 2. JS资源目标

**定义位置**: `BUILD.gn:46-49`

```gn
ohos_js_assets("dlp_manager_js_assets") {
  ets2abc = true          # ETS转ABC字节码
  source_dir = "entry/src/main/ets"
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| ets2abc | true | 将ETS源码编译为ABC字节码 |
| source_dir | entry/src/main/ets | 源码目录 |

### 3. 资源目标

**定义位置**: `BUILD.gn:51-55`

```gn
ohos_resources("dlp_manager_resources") {
  sources = [ "entry/src/main/resources" ]
  deps = [ ":dlp_manager_app_profile" ]
  hap_profile = "entry/src/main/module.json"
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| sources | entry/src/main/resources | 资源目录 |
| deps | dlp_manager_app_profile | 依赖应用配置 |
| hap_profile | entry/src/main/module.json | HAP配置文件 |

### 4. 应用配置目标

**定义位置**: `BUILD.gn:41-44`

```gn
ohos_app_scope("dlp_manager_app_profile") {
  app_profile = "AppScope/app.json"
  sources = [ "AppScope/resources" ]
}
```

---

## 编译产物

### 产物清单

| 产物类型 | 文件名 | 说明 |
|---------|--------|------|
| HAP包 | dlp_manager.hap | 应用安装包 |
| ABC字节码 | *.abc | ETS编译后的字节码 |
| 资源文件 | resources/**/*.json | 编译后的资源 |

### 产物路径

```
out/{product}/{variant}/
├── applications/standard/dlp_manager/
│   └── dlp_manager.hap          # HAP安装包
└── ...
```

### 安装路径

编译后HAP包安装到系统路径：

```
/system/app/com.ohos.dlpmanager/
├── dlp_manager.hap
└── ...
```

**配置来源**: `BUILD.gn:28` (`module_install_dir = "app/com.ohos.dlpmanager"`)

---

## 构建配置详解

### 1. bundle.json - 项目元数据

**文件位置**: `bundle.json`

```json
{
  "name": "@ohos/dlp_manager",
  "description": "dlp_manager",
  "version": "3.1.0",
  "license": "Apache License 2.0",
  "component": {
    "name": "dlp_manager",
    "subsystem": "applications",
    "adapted_system_type": ["standard"],
    "rom": "670KB",
    "build": {
      "sub_component": [
        "//applications/standard/dlp_manager:dlp_manager"
      ]
    }
  }
}
```

#### 关键配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | @ohos/dlp_manager | 组件名 |
| version | 3.1.0 | 组件版本 |
| subsystem | applications | 所属子系统 |
| adapted_system_type | ["standard"] | 适配系统类型（标准系统） |
| rom | 670KB | ROM占用估算 |
| build.sub_component | //applications/standard/dlp_manager:dlp_manager | 构建子组件 |

### 2. build-profile.json5 - 编译配置

**文件位置**: `build-profile.json5`

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23,
        "runtimeOS": "OpenHarmony"
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry"
    }
  ]
}
```

#### 关键配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| compileSdkVersion | 23 | 编译SDK版本 |
| compatibleSdkVersion | 23 | 兼容SDK版本 |
| runtimeOS | OpenHarmony | 运行时OS |

### 3. 签名配置

**签名配置导入**: `BUILD.gn:15`

```gn
import("signature/dlpm.gni")
```

**条件签名** (`BUILD.gn:34-38`):

```gn
if (defined(sign_hap_py_path)) {
  certificate_profile = "${certificate_profile_path}"
  key_alias = "dlpmanager V2"
  compatible_version = "9"
}
```

**说明**: 系统应用需要系统签名证书，开发时使用调试签名，发布时使用正式签名。

---

## 构建流程

### 构建步骤

```bash
# 1. 编译应用
hb build //applications/standard/dlp_manager:dlp_manager

# 或编译整个子系统
hb build --subsystem applications

# 2. 生成的HAP包路径
out/{product}/applications/standard/dlp_manager/dlp_manager.hap
```

### 构建流程图

```
开始构建
    │
    ▼
┌─────────────────────┐
│ 1. 解析BUILD.gn     │
│    加载依赖          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. 编译ETS源码      │
│    ets2abc转换      │
│    entry/src/main/ets
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. 处理资源文件     │
│    entry/src/main/resources
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 4. 打包HAP          │
│    合并ABC+资源     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 5. 签名             │
│    使用系统证书签名  │
└──────────┬──────────┘
           │
           ▼
       构建完成
    dlp_manager.hap
```

---

## 构建开关与配置

### 1. 编译配置

| 配置 | 文件 | 说明 |
|------|------|------|
| API版本 | AppScope/app.json | minAPIVersion: 12, targetAPIVersion: 12 |
| 设备类型 | entry/src/main/module.json | default, tablet, 2in1 |
| 构建模式 | BUILD.gn | js_build_mode = "release" |

### 2. 特性配置

**ArkTSPartialUpdate**: `entry/src/main/module.json:15-19`

```json
"metadata": [
  {
    "name": "ArkTSPartialUpdate",
    "value": "true"
  }
]
```

---

## 运行时加载关系

### HAP包结构

```
dlp_manager.hap
├── ets/                      # ABC字节码
│   └── modules.abc
├── resources/                # 资源文件
│   ├── base/
│   ├── zh_CN/
│   └── ...
├── module.json               # 模块配置
└── ability.json              # Ability配置
```

### 运行时加载流程

```
系统启动
    │
    ▼
┌─────────────────────┐
│ 1. 加载HAP包        │
│    /system/app/     │
│    com.ohos.dlpmanager/
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. 解析module.json  │
│    注册Ability      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. 加载ABC字节码    │
│    ets/modules.abc  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 4. 初始化Ability    │
│    AbilityStage     │
└─────────────────────┘
```

---

## 常见构建问题

### 问题1: 签名失败

**现象**: 构建时报签名错误

**解决**:
1. 确认签名证书存在：`signature/dlpmanager.p7b`
2. 开发时可使用调试签名
3. 正式构建需要系统签名证书

### 问题2: 资源编译失败

**现象**: resources编译错误

**解决**:
1. 检查资源文件格式
2. 检查json配置文件语法
3. 清理缓存重新构建

### 问题3: ETS编译失败

**现象**: ets2abc转换错误

**解决**:
1. 检查ETS语法
2. 确认SDK版本匹配
3. 检查类型定义

---

## 关键结论

1. **单HAP结构**: DLP Manager为单HAP应用，Entry模式
2. **系统签名**: 需要系统签名证书才能正常安装运行
3. **ABC字节码**: ETS源码编译为ABC字节码运行
4. **标准系统**: 仅适配OpenHarmony标准系统
5. **release模式**: 构建配置为release模式（非debug）

---

## 相关链接

- [目录结构](20_Directory_Structure.md) - 查看代码组织
- [调试指南](70_Debugging.md) - 查看安装调试方法

# 配置字段参考

## 目的

本文档提供 `productdefine/common` 仓库 JSON 配置文件的**完整字段参考**，包括字段说明、取值范围、使用示例。

**适用范围**: 需要查阅配置字段定义的开发者。

---

## 顶层字段

### version

**类型**: `string`  
**必填**: ✅ 是  
**取值**: `"3.0"`  
**说明**: 配置文件格式版本

**示例**:
```json
{
  "version": "3.0"
}
```

---

### product_name

**类型**: `string`  
**必填**: ✅ products/ 目录下必填  
**格式**: `system-{target_cpu}-{variant}` 或自定义名称  
**说明**: 产品标识名，用于编译参数 `--product-name`

**示例**:
```json
{
  "product_name": "system-arm64-default"
}
```

**命名规范**:
- 使用小写字母
- 单词间使用连字符 `-`
- 避免使用特殊字符

---

### device_company

**类型**: `string`  
**必填**: ✅ products/ 目录下必填  
**取值**: `"ohos"` 或其他厂商名  
**说明**: 设备厂商标识

**示例**:
```json
{
  "device_company": "ohos"
}
```

---

### target_cpu

**类型**: `string`  
**必填**: ✅ products/ 目录下必填  
**取值**: `"arm64"`, `"arm"`, `"x86_64"`, `"riscv64"` 等  
**说明**: 目标 CPU 架构

**示例**:
```json
{
  "target_cpu": "arm64"
}
```

---

### board

**类型**: `string`  
**必填**: ✅ products/ 目录下必填  
**取值**: 通常与 `target_cpu` 相同  
**说明**: 目标板级平台

**示例**:
```json
{
  "board": "arm64"
}
```

---

### type

**类型**: `string`  
**必填**: ✅ products/ 目录下必填  
**取值**: `"mini"`, `"small"`, `"standard"`  
**说明**: 系统类型

| 取值 | 说明 | 适用场景 |
|------|------|----------|
| `mini` | 轻量系统 | 资源极度受限设备 |
| `small` | 小型系统 | 中等资源设备 |
| `standard` | 标准系统 | 富设备（手机/平板/PC）|

**示例**:
```json
{
  "type": "standard"
}
```

---

### inherit

**类型**: `array of string`  
**必填**: ❌ 可选  
**格式**: `productdefine/common/{path}/{file}.json`  
**说明**: 继承的配置文件列表

**示例**:
```json
{
  "inherit": [
    "productdefine/common/inherit/rich.json",
    "productdefine/common/inherit/phone.json"
  ]
}
```

**规则**:
- 按数组顺序加载
- 后加载的配置覆盖先加载的同名配置
- 当前文件的 `subsystems` 最后加载，优先级最高

---

### subsystems

**类型**: `array of object`  
**必填**: ✅ 是  
**说明**: 子系统配置列表

**示例**:
```json
{
  "subsystems": [
    {
      "subsystem": "security",
      "components": [...]
    }
  ]
}
```

---

### enable_ramdisk

**类型**: `boolean`  
**必填**: ❌ 可选  
**取值**: `true`, `false`  
**说明**: 是否启用 ramdisk  
**适用**: products/ 目录

**示例**:
```json
{
  "enable_ramdisk": true
}
```

---

### build_selinux

**类型**: `boolean`  
**必填**: ❌ 可选  
**取值**: `true`, `false`  
**说明**: 是否编译 SELinux  
**适用**: products/ 目录

**示例**:
```json
{
  "build_selinux": true
}
```

---

## 子系统对象字段

### subsystem

**类型**: `string`  
**必填**: ✅ 是  
**说明**: 子系统名称

**常用子系统列表**:

| 子系统名 | 说明 |
|----------|------|
| `arkui` | ArkUI 框架 |
| `security` | 安全子系统 |
| `communication` | 通信子系统 |
| `multimedia` | 多媒体子系统 |
| `hiviewdfx` | 调测子系统 |
| `startup` | 启动子系统 |
| `distributeddatamgr` | 分布式数据管理 |
| `bundlemanager` | 包管理子系统 |
| `ability` | Ability 框架 |
| `hdf` | 硬件驱动框架 |
| `systemabilitymgr` | 系统能力管理 |
| `global` | 全球化 |
| `powermgr` | 电源管理 |
| `resourceschedule` | 资源调度 |
| `graphic` | 图形子系统 |
| `window` | 窗口管理 |
| `inputmethod` | 输入法框架 |

**示例**:
```json
{
  "subsystem": "security"
}
```

---

### components

**类型**: `array of object`  
**必填**: ✅ 是  
**说明**: 部件配置列表

**示例**:
```json
{
  "components": [
    {
      "component": "huks",
      "features": []
    }
  ]
}
```

---

## 部件对象字段

### component

**类型**: `string`  
**必填**: ✅ 是  
**说明**: 部件名称

**常用部件示例**:

| 部件名 | 所属子系统 | 说明 |
|--------|------------|------|
| `huks` | security | 通用密钥库 |
| `access_token` | security | 访问令牌管理 |
| `ace_engine` | arkui | ArkUI 引擎 |
| `bundle_framework` | bundlemanager | 包管理框架 |
| `ability_runtime` | ability | Ability 运行时 |
| `hilog` | hiviewdfx | 日志系统 |
| `wifi` | communication | WiFi 服务 |
| `bluetooth` | communication | 蓝牙服务 |

**示例**:
```json
{
  "component": "huks"
}
```

---

### features

**类型**: `array of string`  
**必填**: ❌ 可选  
**格式**: `{component_name}_{feature_name}={value}`  
**说明**: 部件特性配置列表

**示例**:
```json
{
  "features": [
    "wifi_feature_non_seperate_p2p=true",
    "wifi_feature_p2p_random_mac_addr=false"
  ]
}
```

**常见 Feature 模式**:

| Feature 模式 | 说明 | 示例 |
|--------------|------|------|
| `*_feature_*=true/false` | 布尔开关 | `ace_engine_feature_enable_web=true` |
| `*_support_*=true/false` | 功能支持 | `hitrace_support_executable_file=false` |
| `*_enable_*=true/false` | 使能控制 | `graphic_2d_feature_ace_enable_gpu=true` |

---

### syscap

**类型**: `array of string`  
**必填**: ❌ 可选  
**格式**: `SystemCapability.{Domain}.{Ability} = {value}`  
**说明**: 系统能力声明

**示例**:
```json
{
  "syscap": [
    "SystemCapability.Multimedia.AVSession.ExtendedDisplayCast = false"
  ]
}
```

---

## 完整配置示例

### 产品配置示例

```json
{
  "product_name": "system-arm64-default",
  "device_company": "ohos",
  "target_cpu": "arm64",
  "board": "arm64",
  "type": "standard",
  "version": "3.0",
  "enable_ramdisk": true,
  "build_selinux": true,
  "inherit": [
    "productdefine/common/inherit/rich.json"
  ],
  "subsystems": [
    {
      "subsystem": "security",
      "components": [
        {
          "component": "selinux_adapter",
          "features": []
        }
      ]
    },
    {
      "subsystem": "hdf",
      "components": [
        {
          "component": "drivers_peripheral_codec",
          "features": []
        },
        {
          "component": "drivers_peripheral_display",
          "features": []
        },
        {
          "component": "drivers_peripheral_input",
          "features": [
            "drivers_peripheral_input_feature_model = true"
          ]
        }
      ]
    }
  ]
}
```

### 继承模版示例

```json
{
  "version": "3.0",
  "subsystems": [
    {
      "subsystem": "security",
      "components": [
        {
          "component": "huks",
          "features": []
        },
        {
          "component": "access_token",
          "features": []
        }
      ]
    },
    {
      "subsystem": "arkui",
      "components": [
        {
          "component": "ace_engine",
          "features": [
            "ace_engine_feature_enable_accessibility = true",
            "ace_engine_feature_enable_web = true"
          ]
        },
        {
          "component": "napi",
          "features": []
        }
      ]
    }
  ]
}
```

### 基础配置示例

```json
{
  "subsystems": [
    {
      "subsystem": "hiviewdfx",
      "components": [
        { "component": "hilog_lite" },
        { "component": "hievent_lite" },
        { "component": "hiview_lite" }
      ]
    },
    {
      "subsystem": "startup",
      "components": [
        { "component": "bootstrap_lite" },
        { "component": "init" }
      ]
    }
  ]
}
```

---

## 字段对照表

### 按目录类型的字段要求

| 字段 | base/ | inherit/ | products/ |
|------|-------|----------|-----------|
| `version` | ❌ | ✅ | ✅ |
| `product_name` | ❌ | ❌ | ✅ |
| `device_company` | ❌ | ❌ | ✅ |
| `target_cpu` | ❌ | ❌ | ✅ |
| `board` | ❌ | ❌ | ✅ |
| `type` | ❌ | ❌ | ✅ |
| `inherit` | ❌ | ⚠️ | ⚠️ |
| `subsystems` | ✅ | ✅ | ✅ |
| `enable_ramdisk` | ❌ | ❌ | ⚠️ |
| `build_selinux` | ❌ | ❌ | ⚠️ |

**图例**:
- ✅ 必须
- ❌ 不支持
- ⚠️ 可选

---

## 相关跳转

- [项目概览](01_Overview.md) - 理解项目定位
- [目录结构](02_Structure.md) - 了解目录组织
- [配置继承](03_Inheritance.md) - 学习继承机制
- [产品配置](04_Products.md) - 查看具体配置

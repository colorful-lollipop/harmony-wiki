# 10_GN_Build - GN 构建配置

## 概述

Authentication Widget 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统进行编译。

**证据**: `BUILD.gn` 行 14

```gn
import("//build/ohos.gni")
```

## 构建入口

### BUILD.gn

**证据**: `BUILD.gn` 行 1-48

完整路径: `/Volumes/lexar/code/d/work/oh/applications/standard/auth_widget/BUILD.gn`

```
auth_widget/
├── declare_args()           # 参数声明
├── ohos_app_scope()         # 应用范围配置
├── ohos_js_assets()         # JS/ArkTS 资源
├── ohos_resources()         # 资源文件
└── ohos_hap()              # HAP 产物
```

## Targets 清单

### 1. authwidget_app_profile

**证据**: `BUILD.gn` 行 20-23

| 属性 | 值 |
|------|-----|
| Target 类型 | `ohos_app_scope` |
| 输出 | 应用配置目录 |
| Sources | `./AppScope/resources` |
| 依赖 | 无 |

```gn
ohos_app_scope("authwidget_app_profile") {
  app_profile = "./AppScope/app.json"
  sources = [ "./AppScope/resources" ]
}
```

### 2. authwidget_js_assets

**证据**: `BUILD.gn` 行 25-28

| 属性 | 值 |
|------|-----|
| Target 类型 | `ohos_js_assets` |
| 输出 | .abc 字节码文件 |
| Sources | `entry/src/main/ets` |
| 开关 | ets2abc = true |

```gn
ohos_js_assets("authwidget_js_assets") {
  ets2abc = true
  source_dir = "entry/src/main/ets"
}
```

### 3. authwidget_resources

**证据**: `BUILD.gn` 行 30-34

| 属性 | 值 |
|------|-----|
| Target 类型 | `ohos_resources` |
| 输出 | 资源打包目录 |
| Sources | `entry/src/main/resources` |
| 依赖 | `:authwidget_app_profile` |
| Hap Profile | `entry/src/main/module.json` |

```gn
ohos_resources("authwidget_resources") {
  sources = [ "entry/src/main/resources" ]
  deps = [ ":authwidget_app_profile" ]
  hap_profile = "entry/src/main/module.json"
}
```

### 4. auth_widget (最终产物)

**证据**: `BUILD.gn` 行 36-47

| 属性 | 值 |
|------|-----|
| Target 类型 | `ohos_hap` |
| 输出 | `.hap` 安装包 |
| Hap Profile | `entry/src/main/module.json` |
| Certificate | `signature/auth_widget.p7b` |
| Hap Name | `AuthWidget` |
| Subsystem | `applications` |
| Part Name | `auth_widget` |
| Module Install Dir | `app/com.ohos.useriam.authwidget` |
| 依赖 | `:authwidget_js_assets`, `:authwidget_resources` |

```gn
ohos_hap("auth_widget") {
  hap_profile = "entry/src/main/module.json"
  deps = [
    ":authwidget_js_assets",
    ":authwidget_resources",
  ]
  certificate_profile = "signature/auth_widget.p7b"
  hap_name = "AuthWidget"
  subsystem_name = "applications"
  part_name = "auth_widget"
  module_install_dir = "app/com.ohos.useriam.authwidget"
}
```

## 依赖关系图

```
auth_widget (ohos_hap)
     │
     ├── authwidget_js_assets (ohos_js_assets)
     │       │
     │       └── entry/src/main/ets/
     │
     └── authwidget_resources (ohos_resources)
             │
             ├── authwidget_app_profile (ohos_app_scope)
             │       │
             │       └── AppScope/resources/
             │
             └── entry/src/main/resources/
                     │
                     └── module.json
```

## 构建开关

### auth_widget_enabled

**证据**: `BUILD.gn` 行 16-18

```gn
declare_args() {
  auth_widget_enabled = true
}
```

**启用方式**: 在产品配置中设置 `auth_widget_enabled = true`

## 产物清单

### 编译产物

| 产物类型 | 预计路径 | 描述 |
|----------|----------|------|
| `.hap` | `out/.../packages/` | 最终安装包 |
| `.abc` | `out/.../libs/` | ArkTS 字节码 |
| `.pak` | `out/.../` | 资源打包文件 |

### 安装路径

**证据**: `BUILD.gn` 行 46

```gn
module_install_dir = "app/com.ohos.useriam.authwidget"
```

- **HAP 安装路径**: `/system/app/com.ohos.useriam.authwidget/`

## 构建命令

### 标准构建

**证据**: `README.md` 行 34-35

```bash
./build.sh --product-name rk3568 --ccache --build-target auth_widget
```

| 参数 | 描述 |
|------|------|
| `--product-name` | 产品名称，如 `rk3568` |
| `--ccache` | 启用缓存编译 |
| `--build-target` | 构建目标，`auth_widget` |

### 参数说明

| 参数 | 必填 | 描述 |
|------|------|------|
| `--product-name` | 是 | 指定产品 |
| `--ccache` | 否 | 加速编译 |
| `--build-target` | 是 | 指定组件 |

## 模块配置

### bundle.json

**证据**: `bundle.json` 行 1-33

| 配置项 | 值 |
|--------|-----|
| 模块名 | `@ohos/auth_widget` |
| 版本 | 4.0 |
| 子系统 | `applications` |
| 组件名 | `auth_widget` |
| 特性 | `auth_widget_enabled` |
| ROM | 860KB |
| RAM | 0KB |
| 子组件 | `//applications/standard/auth_widget:auth_widget` |

```json
{
  "name": "@ohos/auth_widget",
  "version": "4.0",
  "component": {
    "name": "auth_widget",
    "subsystem": "applications",
    "features": ["auth_widget_enabled"],
    "adapted_system_type": ["standard"],
    "build": {
      "sub_component": [
        "//applications/standard/auth_widget:auth_widget"
      ]
    }
  }
}
```

## 构建产物验证

### 预期输出目录结构

```
out/{product}/
└── packages/
    └── phone/
        └── system/
            └── app/
                └── com.ohos.useriam.authwidget/
                    └── AuthWidget.hap
```

## 相关文档

- [模块配置](11_Module_Config.md) - bundle.json 与 module.json 详解
- [架构说明](01_Architecture.md) - 构建在架构中的位置
- [常见问题](90_FAQ.md) - 构建问题排查

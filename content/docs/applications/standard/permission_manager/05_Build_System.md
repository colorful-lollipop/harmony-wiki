# 构建系统

> **目的**: 详细说明PermissionManager的GN构建配置、Targets、编译产物和安装路径  
> **适用范围**: 构建工程师、系统集成人员  
> **最后更新**: 2026-02-05

---

## 构建系统概述

PermissionManager使用 **GN (Generate Ninja)** 构建系统，这是OpenHarmony的标准构建工具。

### 构建流程

```
┌─────────────────────────────────────────────────────────────────────┐
│                         源码文件                                     │
│  *.ets, *.ts, *.json, resources/                                    │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       GN配置解析                                     │
│  BUILD.gn → 定义targets、依赖、产物                                  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Ninja构建                                       │
│  - ArkTS编译 (ets2abc)                                              │
│  - 资源打包                                                         │
│  - HAP包组装                                                        │
│  - 签名                                                             │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      编译产物                                        │
│  permission_manager.hap → 安装到 /system/app/                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## BUILD.gn 详细分析

**文件位置**: `/applications/standard/permission_manager/BUILD.gn`

### 整体结构

```gn
# 导入OpenHarmony构建配置
import("//build/ohos.gni")
import("signature/pm.gni")

# Target 1: 主应用Target (生成HAP包)
ohos_app("permission_manager") {
  # ...
}

# Target 2: 应用级资源配置
ohos_app_scope("permission_manager_app_profile") {
  # ...
}

# Target 3: ArkTS代码编译
ohos_js_assets("permission_manager_js_assets") {
  # ...
}

# Target 4: 资源文件打包
ohos_resources("permission_manager_resources") {
  # ...
}
```

---

### Target 1: permission_manager (主应用)

```gn
ohos_app("permission_manager") {
  # 依赖的其他targets
  deps = [
    ":permission_manager_js_assets",      # ArkTS编译产物
    ":permission_manager_resources",      # 资源文件
  ]
  
  # 公开性配置文件
  publicity_file = "publicity.xml"
  
  # 签名证书
  certificate_profile = "signature/pm.p7b"
  
  # HAP包名称
  hap_name = "permission_manager"
  
  # 部件名称
  part_name = "permission_manager"
  
  # 子系统名称
  subsystem_name = "applications"
  
  # JS构建模式
  js_build_mode = "release"
  
  # 模块安装目录
  module_install_dir = "app/com.ohos.permissionmanager"
  
  # SDK配置
  sdk_home = "//prebuilts/ohos-sdk/linux"
  sdk_type_name = [ "sdk.dir" ]
  
  # 构建类型
  assemble_type = "assembleHap"
  
  # 构建级别
  build_level = "module"
  
  # 构建的模块
  build_modules = [ "permissionmanager" ]
  
  # 条件签名配置（构建时传入）
  if (defined(sign_hap_py_path)) {
    certificate_profile = "${certificate_profile_path}"
    key_alias = "PermissionManager"
    private_key_path = "PermissionManager"
    compatible_version = "9"
  }
}
```

**关键配置说明**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `hap_name` | permission_manager | 生成的HAP包文件名（不含扩展名） |
| `module_install_dir` | app/com.ohos.permissionmanager | 设备上的安装路径 |
| `subsystem_name` | applications | 所属子系统 |
| `part_name` | permission_manager | 所属部件 |
| `js_build_mode` | release | 发布模式（非调试） |
| `certificate_profile` | signature/pm.p7b | 签名证书路径 |

---

### Target 2: permission_manager_app_profile

```gn
ohos_app_scope("permission_manager_app_profile") {
  # 应用配置文件
  app_profile = "AppScope/app.json"
  
  # 全局资源
  sources = [ "AppScope/resources" ]
}
```

**职责**: 配置应用级信息和全局资源。

**关联文件**:
- `AppScope/app.json` - 应用基本信息（bundleName, version等）
- `AppScope/resources/` - 全局资源文件

---

### Target 3: permission_manager_js_assets

```gn
ohos_js_assets("permission_manager_js_assets") {
  # 启用ArkTS编译（ets2abc）
  ets2abc = true
  
  # ArkTS源码目录
  source_dir = "permissionmanager/src/main/ets"
}
```

**职责**: 编译ArkTS源代码。

**编译过程**:
1. 将 `.ets` 文件编译为方舟字节码 (ABC)
2. 输出到 `ets/` 目录

---

### Target 4: permission_manager_resources

```gn
ohos_resources("permission_manager_resources") {
  # 资源源目录
  sources = [ "permissionmanager/src/main/resources" ]
  
  # 依赖应用级profile
  deps = [ ":permission_manager_app_profile" ]
  
  # 模块配置文件
  hap_profile = "permissionmanager/src/main/module.json"
}
```

**职责**: 打包资源文件（字符串、图片、布局等）。

**关联文件**:
- `permissionmanager/src/main/resources/` - 模块资源
- `permissionmanager/src/main/module.json` - 模块配置

---

## Targets 清单

| Target | 类型 | 输出 | 依赖 | 描述 |
|--------|------|------|------|------|
| `permission_manager` | ohos_app | .hap | js_assets, resources | 主应用包 |
| `permission_manager_app_profile` | ohos_app_scope | - | - | 应用级配置 |
| `permission_manager_js_assets` | ohos_js_assets | ets/ | - | ArkTS编译 |
| `permission_manager_resources` | ohos_resources | resources/ | app_profile | 资源打包 |

---

## 编译产物

### 1. HAP包 (HarmonyOS Ability Package)

**产物名称**: `permission_manager.hap`

**生成位置**: `out/{product}/packages/phone/system/app/com.ohos.permissionmanager/`

**安装路径**: `/system/app/com.ohos.permissionmanager/permission_manager.hap`

**HAP包内部结构**:

```
permission_manager.hap
├── ets/                        # ArkTS编译产物
│   ├── pages/                  # 页面字节码
│   ├── MainAbility/            # Ability字节码
│   ├── ServiceExtAbility/
│   └── ...
├── resources/                  # 资源文件
│   ├── base/
│   │   ├── element/            # 字符串、颜色等
│   │   ├── media/              # 图片
│   │   └── profile/            # 页面配置
│   └── rawfile/                # 原始文件
├── module.json                 # 模块配置（编译后）
├── ability.json                # Ability配置
├── config.json                 # 应用配置
└── signature/                  # 签名信息
    └── ...
```

### 2. 中间产物

| 产物 | 位置 | 说明 |
|------|------|------|
| ArkTS字节码 | `out/{product}/obj/.../ets/` | .abc文件 |
| 资源索引 | `out/{product}/obj/.../resources.index` | 资源索引文件 |
| 编译日志 | `out/{product}/build.log` | 构建日志 |

---

## 构建命令

### 完整构建

```bash
# 进入OpenHarmony源码根目录
cd {ohos_source_root}

# 执行构建
./build.sh --product-name rk3568 --build-target //applications/standard/permission_manager:permission_manager
```

**参数说明**:
- `--product-name`: 产品名称（如 rk3568, phone等）
- `--build-target`: 要构建的GN target路径

### 仅编译模块

```bash
# 只编译PermissionManager
./build.sh --product-name rk3568 --build-target permission_manager
```

### 清理构建

```bash
# 清理产物
rm -rf out/rk3568/applications/standard/permission_manager/

# 重新构建
./build.sh --product-name rk3568 --build-target permission_manager
```

---

## 签名配置

### 开发签名

**证书文件**: `signature/pm.p7b`

**GN配置**:
```gn
certificate_profile = "signature/pm.p7b"
```

### 发布签名

在构建时通过参数传入签名信息：

```gn
if (defined(sign_hap_py_path)) {
  certificate_profile = "${certificate_profile_path}"
  key_alias = "PermissionManager"
  private_key_path = "PermissionManager"
  compatible_version = "9"
}
```

**签名工具**: OpenHarmony SDK提供的 `sign_hap.py`

---

## 依赖关系

### GN Target依赖图

```
permission_manager (ohos_app)
    ├── permission_manager_js_assets (ohos_js_assets)
    │   └── source_dir: permissionmanager/src/main/ets
    │       ├── *.ets (64个源文件)
    │       └── ets2abc编译
    │
    └── permission_manager_resources (ohos_resources)
        ├── permission_manager_app_profile (ohos_app_scope)
        │   ├── AppScope/app.json
        │   └── AppScope/resources/
        │
        ├── permissionmanager/src/main/resources/
        └── permissionmanager/src/main/module.json
```

### 外部依赖

PermissionManager依赖以下OpenHarmony子系统：

| 子系统 | 用途 |
|--------|------|
| ability | Ability框架 |
| bundle_manager | 应用包管理 |
| window_manager | 窗口管理 |
| account | 系统账号 |
| access_token | 权限访问控制 |

---

## Feature Flags

当前项目未定义特殊的Feature Flags。所有功能通过代码中的条件编译控制：

```typescript
// 设备类型检查
if (deviceInfo.deviceType === 'wearable') {
    this.context.terminateSelf();
    return;
}
```

---

## 构建问题排查

### 常见问题

#### 1. 编译失败：找不到依赖

**症状**:
```
ERROR: //applications/standard/permission_manager:permission_manager
Target //some/other:target not found
```

**解决**:
- 检查 `deps` 中的路径是否正确
- 确保依赖的模块已编译

#### 2. 签名失败

**症状**:
```
ERROR: Sign failed
```

**解决**:
- 检查 `signature/pm.p7b` 是否存在
- 检查证书是否过期
- 确认 `sign_hap_py_path` 是否正确设置

#### 3. 资源打包失败

**症状**:
```
ERROR: Resource compilation failed
```

**解决**:
- 检查 `resources/` 目录结构
- 确认资源文件格式正确
- 检查 `module.json` 配置

#### 4. ArkTS编译失败

**症状**:
```
ERROR: ets2abc failed
```

**解决**:
- 检查ArkTS语法错误
- 确认导入路径正确
- 检查类型定义

---

## 安装和验证

### 安装到设备

```bash
# 推送HAP包到设备
hdc file send out/rk3568/packages/phone/system/app/com.ohos.permissionmanager/permission_manager.hap /data/local/tmp/

# 安装HAP
hdc shell bm install -p /data/local/tmp/permission_manager.hap

# 或者覆盖安装（开发时）
hdc shell mount -o remount,rw /
hdc shell rm -rf /system/app/com.ohos.permissionmanager
hdc file send permission_manager.hap /system/app/com.ohos.permissionmanager/
hdc shell reboot
```

### 验证安装

```bash
# 检查HAP是否存在
hdc shell ls -la /system/app/com.ohos.permissionmanager/

# 查看应用信息
hdc shell bm dump -n com.ohos.permissionmanager

# 检查日志
hdc shell hilog | grep PermissionManager
```

---

## 产物映射表

| Source | GN Target | Output | Install Path |
|--------|-----------|--------|--------------|
| `permissionmanager/src/main/ets/*.ets` | permission_manager_js_assets | `ets/*.abc` | `permission_manager.hap/ets/` |
| `permissionmanager/src/main/resources/` | permission_manager_resources | `resources/` | `permission_manager.hap/resources/` |
| `permissionmanager/src/main/module.json` | permission_manager_resources | `module.json` | `permission_manager.hap/module.json` |
| `AppScope/*` | permission_manager_app_profile | - | `permission_manager.hap/` |
| `signature/pm.p7b` | permission_manager | 签名信息 | `permission_manager.hap/signature/` |

---

*上一篇: [内部API说明](04_Inner_API.md) | 返回 [README](README.md) | 下一篇: [安全风险评审](06_Security_Analysis.md)*

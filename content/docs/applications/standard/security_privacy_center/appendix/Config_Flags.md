# 配置开关

## 概述

本文档描述安全隐私中心模块中的关键配置项，包括常量化配置、功能开关等。

## 代码中的常量定义

### 业务常量

**文件位置**：`entry/src/main/ets/common/constants/ComConstant.ets`

```typescript
export default class ComConstant {
  // 宽度高度全屏
  static readonly WIDTH_HEIGHT_FULL_SCREEN: string = '100%';
  
  // 应用信息列表 Key
  static readonly INIT_APP_INFO_LIST: string = 'initAppInfoList';
}
```

**用途**：统一管理全局业务常量，避免硬编码。

### 路由常量

**文件位置**：`entry/src/main/ets/common/constants/RouterConstant.ets`

```typescript
// 路由相关常量定义
```

### 数据共享常量

**文件位置**：`entry/src/main/ets/common/constants/DataShareConstant.ets`

```typescript
export default class DataShareConstants {
  // RDB 表名常量
  static readonly ANTO_MENU_TABLE_V2 = {
    tableName: 'anto_menu_table_v2'
  };
}
```

### 系统事件常量

**文件位置**：`entry/src/main/ets/common/constants/HiSysEventConstant.ets`

```typescript
// HiSysEvent 系统事件相关常量
```

## 模块配置

### 目标 Ability 模式

**文件位置**：`entry/src/main/ets/main/auto_menu/AutoMenuModel.ets`

```typescript
// UIAbility 访问方法
const DST_UIABILITY_MODE = 0;

// 跳转页面模式
const DST_PAGE_MODE = 2;

// UIExtensionAbility 模式（需要继承 UIExtensionAbility）
const DST_ABILITY_MODE = 1;
```

**用途**：定义菜单点击后的跳转目标类型。

| 常量 | 值 | 说明 |
|-----|-----|------|
| DST_UIABILITY_MODE | 0 | 通过 startAbility 启动 UIAbility |
| DST_ABILITY_MODE | 1 | 通过 UIExtensionComponent 承载 |
| DST_PAGE_MODE | 2 | 通过 router 跳转本地页面 |

### 菜单展示模式

**文件位置**：`entry/src/main/ets/main/auto_menu/AutoMenuViewModel.ets`

```typescript
const DISPLAY_MODEL_LIST = 'list';
const DISPLAY_MODEL_CARD = 'card';
```

**用途**：定义菜单在首页的展示形式。

| 常量 | 值 | 说明 |
|-----|-----|------|
| DISPLAY_MODEL_LIST | 'list' | 列表模式展示 |
| DISPLAY_MODEL_CARD | 'card' | 卡片模式展示 |

## 构建配置

### build-profile.json5

**文件位置**：`build-profile.json5`

```json5
{
  "products": [
    {
      "name": "default",
      "compileSdkVersion": 23,
      "compatibleSdkVersion": 23,
      "runtimeOS": "OpenHarmony"
    }
  ],
  "buildModeSet": [
    { "name": "debug" },
    { "name": "release" }
  ]
}
```

| 配置项 | 值 | 说明 |
|-------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| buildMode | debug/release | 构建模式 |

### module.json5

**文件位置**：`entry/src/main/module.json5`

```json5
{
  "module": {
    "deviceTypes": ["default"],
    "deliveryWithInstall": true,
    "installationFree": false
  }
}
```

| 配置项 | 值 | 说明 |
|-------|-----|------|
| deviceTypes | default | 目标设备类型 |
| deliveryWithInstall | true | 安装时交付 |
| installationFree | false | 非免安装模块 |

## 权限配置

### 声明的权限

**文件位置**：`entry/src/main/module.json5:51-77`

| 权限 | 用途 | 是否高敏感 |
|-----|------|-----------|
| ohos.permission.MANAGE_SECURE_SETTINGS | 管理安全设置 | 是 |
| ohos.permission.ACCESS_BUNDLE_DIR | 访问应用目录 | 是 |
| ohos.permission.GET_BUNDLE_INFO | 获取应用信息 | 否 |
| ohos.permission.GET_INSTALLED_BUNDLE_LIST | 获取已安装列表 | 否 |
| ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER | 访问安全隐私中心 | 是 |
| ohos.permission.ACCESS_CERT_MANAGER | 访问证书管理 | 是 |
| ohos.permission.CONTROL_LOCATION_SWITCH | 控制位置开关 | 是 |

## 功能开关（Feature Flags）

本项目未使用显式的 feature flags，但以下配置可视为功能开关：

### 1. 菜单显示模式

通过 `displayedMode` 控制菜单展示形式：
- `'list'`：列表模式
- `'card'`：卡片模式

**证据来源**：`AutoMenuViewModel.ets:43-52`

### 2. 菜单可点击状态

通过 `isClickable` 控制菜单是否可点击：
- `0`：不可点击
- `1`：可点击

**证据来源**：`MenuInfo.ets:32`

### 3. 菜单支持状态

通过 `isSupport` 控制菜单是否展示：
- `0`：不支持
- `1`：支持

**证据来源**：`MenuInfo.ets:29`

## 资源配置

### 字符串资源

**文件位置**：`entry/src/main/resources/base/element/string.json`

```json
{
  "app": {
    "EntryAbility_desc": "安全隐私中心",
    "EntryAbility_label": "隐私",
    "module_desc": "安全隐私中心模块"
  }
}
```

### 颜色资源

**文件位置**：`entry/src/main/resources/base/element/color.json`

应用颜色配置。

### 尺寸资源

**文件位置**：`entry/src/main/resources/base/element/float.json`

应用尺寸配置。

## 返回导航

- [SUMMARY.md](../SUMMARY.md) → 文档导航
- [06_Build_Config.md](../06_Build_Config.md) → 构建配置
- [09_FAQ.md](../09_FAQ.md) → 常见问题

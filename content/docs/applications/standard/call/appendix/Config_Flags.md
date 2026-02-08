# 配置项说明

## 1. 模块配置

### 1.1 entry 模块配置

**文件**: `entry/src/main/module.json`

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `module.name` | `callui` | 模块名称 |
| `module.type` | `entry` | 模块类型 (entry/feature/har) |
| `module.srcEntrance` | `./ets/Application/MyAbilityStage.ts` | 入口文件 |
| `module.mainElement` | `com.ohos.callui.ServiceAbility` | 主组件 |
| `module.deviceTypes` | `["default", "tablet"]` | 支持设备类型 |
| `module.deliveryWithInstall` | `true` | 安装时部署 |
| `module.installationFree` | `false` | 非免安装 |

### 1.2 Ability 配置

| 配置项 | MainAbility 值 | ServiceAbility 值 | 说明 |
|-------|---------------|------------------|------|
| `name` | `com.ohos.callui.MainAbility` | `com.ohos.callui.ServiceAbility` | Ability 名称 |
| `type` | `ability` | `service` | Ability 类型 |
| `visible` | `false` | `true` | 是否可见 |
| `srcEntrance` | `./ets/MainAbility/MainAbility.ts` | `./ets/ServiceAbility/ServiceAbility.ts` | 入口文件 |
| `backgroundModes` | `["voip"]` | - | 后台模式 |

## 2. 应用配置

### 2.1 AppScope 配置

**文件**: `AppScope/app.json`

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `bundleName` | `com.ohos.callui` | 包名 |
| `vendor` | `example` | 厂商 |
| `versionCode` | `10004032` | 版本码 |
| `versionName` | `1.0.4.032` | 版本名 |
| `minAPIVersion` | `9` | 最低 API Level |
| `targetAPIVersion` | `9` | 目标 API Level |

## 3. 构建配置

### 3.1 BUILD.gn Targets

| Target | 类型 | 输入 | 输出 | 说明 |
|-------|------|-----|------|------|
| `callui_hap` | ohos_hap | js_assets, resources | `.hap` | 通话 UI HAP |
| `callui_js_assets` | ohos_js_assets | `entry/src/main/ets` | 中间字节码 | JS 资源编译 |
| `callui_resources` | ohos_resources | resources | 资源包 | 资源编译 |
| `callui_app_profile` | ohos_app_scope | AppScope | 应用配置 | 应用级配置 |
| `mobileDataSettings_hap` | ohos_hap | js_assets, resources | `.hap` | 移动数据 HAP |
| `mobiledatasettings_js_assets` | ohos_js_assets | `mobiledatasettings/src/main/ets` | 中间字节码 | JS 资源编译 |
| `mobiledatasettings_resources` | ohos_resources | resources | 资源包 | 资源编译 |

### 3.2 hvigor 配置

**文件**: `build-profile.json5`

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `compileSdkVersion` | `9` | 编译 SDK 版本 |
| `compatibleSdkVersion` | `9.0.0` | 兼容 SDK 版本 |
| `targetSdkVersion` | `9` | 目标 SDK 版本 |

## 4. 权限配置

### 4.1 entry 模块权限

| 权限名称 | 用途 | 敏感度 |
|---------|------|-------|
| `ohos.permission.PLACE_CALL` | 发起通话 | 高 |
| `ohos.permission.ANSWER_CALL` | 接听电话 | 高 |
| `ohos.permission.GET_TELEPHONY_STATE` | 获取通话状态 | 中 |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置通话状态 | 高 |
| `ohos.permission.READ_CONTACTS` | 读取联系人 | 高 |
| `ohos.permission.SEND_MESSAGES` | 发送短信 | 高 |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | 中 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取包信息 | 中 |
| `ohos.permission.NOTIFICATION_CONTROLLER` | 通知控制 | 中 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 后台启动 | 中 |
| `ohos.permission.KEEP_BACKGROUND_RUNNING` | 后台保持 | 中 |
| `ohos.permission.VIBRATE` | 振动 | 低 |
| `ohos.permission.ACCELEROMETER` | 加速计 | 低 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 安全设置 | 高 |
| `ohos.permission.RUNNING_LOCK` | 运行锁 | 中 |

### 4.2 mobiledatasettings 模块权限

| 权限名称 | 用途 | 敏感度 |
|---------|------|-------|
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | 中 |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置通话状态 | 高 |
| `ohos.permission.GET_TELEPHONY_STATE` | 获取通话状态 | 中 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 安全设置 | 高 |

## 5. 功能开关

### 5.1 模块类型开关

| 模块 | type 值 | 说明 |
|-----|---------|------|
| entry | `entry` | 主入口模块，可独立安装 |
| mobiledatasettings | `feature` | 功能模块，需依赖 entry |
| common | `har` | 静态库，可被任意模块依赖 |

### 5.2 页面配置

**文件**: `entry/src/main/module.json`

```json
{
  "pages": "$profile:main_pages"
}
```

**文件**: `entry/src/main/resources/base/profile/main_pages.json`

```json
{
  "src": [
    "pages/index"
  ]
}
```

## 6. 资源引用

### 6.1 支持的设备类型

| 类型 | 说明 |
|-----|------|
| `default` | 默认设备 (手机) |
| `tablet` | 平板设备 |

### 6.2 支持的语言

| 语言代码 | 说明 |
|---------|------|
| `zh_CN` | 简体中文 |
| `base` | 默认资源 |

## 7. 编译选项

### 7.1 ETS 编译选项

| 选项 | 值 | 说明 |
|-----|-----|------|
| `ets2abc` | `true` | 启用 ETS 到 ABC 字节码编译 |

### 7.2 HAP 构建选项

| 选项 | 值 | 说明 |
|-----|-----|------|
| `hap_name` | `CallUI` / `MobileDataSettings` | HAP 名称 |
| `subsystem_name` | `applications` | 子系统名称 |
| `part_name` | `prebuilt_hap` | 部件名称 |
| `module_install_dir` | `app/com.ohos.callui` | 安装路径 |

## 8. 签名配置

### 8.1 证书配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `certificate_profile` | `signature/callui.p7b` | 签名证书路径 |
| `hap_name` | - | HAP 名称 |
| `signingConfig` | `default` | 签名配置名称 |

## 9. 运行时配置

### 9.1 系统能力依赖

| 能力 | 说明 |
|-----|------|
| `ohos.permission.PLACE_CALL` | 通话能力 |
| `ohos.permission.ANSWER_CALL` | 接听能力 |
| `ohos.permission.READ_CONTACTS` | 联系人读取 |
| `ohos.permission.NOTIFICATION_CONTROLLER` | 通知控制 |

### 9.2 后台模式

| 模式 | MainAbility | ServiceAbility |
|-----|-------------|----------------|
| `voip` | ✅ 支持 | - |

## 10. 相关文档

- [构建指南](04_Build_Guide.md)
- [API 参考](03_API_Reference.md)
- [架构设计](02_Architecture.md)

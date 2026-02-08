# 系统资源说明

## 目的

本文档描述 `system_resources` 模块提供的系统资源类型，包括系统字体、系统资源包内容、权限定义以及资源组织方式，帮助开发者理解可用的系统级资源。

## 适用范围

- **模块**: `/base/global/system_resources`
- **资源类型**: 字体、权限、多语言字符串、分层参数

## 系统字体

### 字体概览

| 字体名称 | 文件 | 字重/变体 | 支持语言 | 支持设备 |
|----------|------|-----------|----------|----------|
| HarmonyOS Sans | HarmonyOS_Sans.ttf | Thin/Light/Regular/Medium/Bold/Black | 105+ 语言 | default, watch |
| HarmonyOS Sans SC | HarmonyOS_Sans_SC.ttf | 6 字重 | 简体中文 | default, watch |
| HarmonyOS Sans TC | HarmonyOS_Sans_TC.ttf | 6 字重 | 繁体中文 | default, watch |
| HarmonyOS Sans Arabic | HarmonyOS_Sans_Naskh_Arabic.ttf | Regular | 阿拉伯语 | default, watch |
| HMSymbolVF | HMSymbolVF.ttf | Variable | 系统图标 | default |
| HMSymbolVF Watch | HMSymbolVF_watch.ttf | Variable | 系统图标 | watch |
| HMOS Color Emoji | HMOSColorEmojiCompat.ttf | - | Emoji | default, watch |

### HarmonyOS Sans 特性

**证据**: `README.md:9-15`

- **可变字体技术**: 单个文件支持多个字重
- **6 种字重**: Thin, Light, Regular, Medium, Bold, Black
- **105+ 语言**: 覆盖中文、拉蒂文、西里尔文、希腊文、阿拉伯文
- **商业免费**: Apache 2.0 许可证 (部分条款)

### 字体安装路径

```
/system/fonts/
├── HarmonyOS_Sans.ttf
├── HarmonyOS_Sans_Italic.ttf
├── HarmonyOS_Sans_SC.ttf
├── HarmonyOS_Sans_TC.ttf
├── HMSymbolVF.ttf
└── ...
```

## 系统资源包

### Hap 包结构

**证据**: `systemres/BUILD.gn:28-42`

| 产物 | 名称 | 安装路径 |
|------|------|----------|
| SystemResources.hap | SystemResources | app/ohos.global.systemres |

### 包内容

```
SystemResources.hap
├── module.json              # 模块配置 + 权限定义
├── AppScope/
│   └── resources/          # 应用范围资源
└── main/
    ├── module.json         # 权限定义引用
    └── resources/          # 主资源
        ├── default/        # 默认设备
        ├── tablet/         # 平板
        ├── car/            # 车机
        ├── wearable/       # 可穿戴
        ├── tv/             # 电视
        ├── th-wearable/    # 穿戴 Thin
        ├── 2in1/           # 二合一
        └── [40+ 语言]/     # 多语言资源
```

## 权限定义

### 权限统计

**证据**: `systemres/main/module.json:17-1301+`

| 权限级别 | 数量 | 授权方式 |
|----------|------|----------|
| normal | ~20 | system_grant/user_grant |
| system_basic | ~100+ | system_grant/user_grant |
| system_core | ~30 | system_grant |

### 权限分类示例

#### normal 级别权限

| 权限名称 | 用途 | grantMode | 证据 |
|----------|------|-----------|------|
| ohos.permission.INTERNET | 网络访问 | system_grant | module.json:205-214 |
| ohos.permission.CAMERA | 相机拍照 | user_grant | module.json:561-570 |
| ohos.permission.BLUETOOTH | 蓝牙使用 | system_grant | module.json:122-129 |
| ohos.permission.VIBRATE | 振动 | system_grant | module.json:752-761 |
| ohos.permission.MICROPHONE | 麦克风 | user_grant | module.json:355-364 |

#### system_basic 级别权限

| 权限名称 | 用途 | grantMode | 证据 |
|----------|------|-----------|------|
| ohos.permission.READ_CONTACTS | 读取联系人 | user_grant | module.json:278-287 |
| ohos.permission.LOCATION | 位置信息 | user_grant | module.json:479-488 |
| ohos.permission.READ_CALL_LOG | 读取通话记录 | user_grant | module.json:256-265 |
| ohos.permission.GET_PHONE_NUMBERS | 获取电话号码 | system_grant | module.json:300-309 |

#### system_core 级别权限

| 权限名称 | 用途 | grantMode | 证据 |
|----------|------|-----------|------|
| ohos.permission.POWER_MANAGER | 电源管理 | system_grant | module.json:694-701 |
| ohos.permission.INSTALL_BUNDLE | 安装应用 | system_grant | module.json:1049-1056 |
| ohos.permission.MANAGE_USER_IDM | 用户身份管理 | system_grant | module.json:1293-1300 |
| ohos.permission.REBOOT | 重启系统 | system_grant | module.json:603-610 |

### 权限定义属性

**证据**: `module.json:17-42`

| 属性 | 说明 | 示例 |
|------|------|------|
| `name` | 权限完整名称 | "ohos.permission.CAMERA" |
| `grantMode` | 授权模式 | "system_grant" / "user_grant" |
| `availableLevel` | 可用级别 | "normal" / "system_basic" / "system_core" |
| `since` | API 起始版本 | 7, 8, 9... |
| `provisionEnable` | 是否可预置 | true / false |
| `distributedSceneEnable` | 是否支持分布式 | true / false |
| `label` | 用户可见标签引用 | "$string:ohos_lab_camera" |
| `description` | 权限描述引用 | "$string:ohos_desc_camera" |

## 多语言资源

### 支持语言

**证据**: `systemres/main/resources/` 目录结构

| 语言代码 | 语言名称 | 状态 |
|----------|----------|------|
| zh_CN | 简体中文 | ✅ 支持 |
| zh_TW | 繁体中文 | ✅ 支持 |
| en_US | 美式英语 | ✅ 支持 |
| ja_JP | 日语 | ✅ 支持 |
| ko_KR | 韩语 | ✅ 支持 |
| ... | 40+ 语言 | ✅ 支持 |

### 资源类型

| 资源类型 | 路径 | 用途 |
|----------|------|------|
| 字符串 | `element/string.json` | UI 显示文本 |
| 颜色 | `element/color.json` | UI 颜色值 |
| 浮点数 | `element/float.json` | UI 数值参数 |
| 颜色系统 | `element/color_sys.json` | 系统颜色主题 |
| 符号 | `element/symbol.json` | 系统符号定义 |
| 复数 | `element/plurals.json` | 复数形式字符串 |

## 分层参数

### 用途

系统级 UI 参数，用于统一界面风格和交互反馈。

**证据**: `README.md:19`

### 参数类型

| 类型 | 文件 | 用途 |
|------|------|------|
| color_sys | color_sys.json | 系统颜色主题 |
| float_sys | float_sys.json | 系统浮点参数 |
| 其他 | *.json | 布局/尺寸参数 |

## 设备适配

### 支持的设备类型

**证据**: `module.json:7-14`

| 设备类型 | 标识符 | 说明 |
|----------|--------|------|
| 默认设备 | default | 手机/平板通用 |
| 电视 | tv | 电视设备 |
| 车机 | car | 汽车中控 |
| 手表 | wearable | 智能手表 |
| 平板 | tablet | 平板电脑 |
| 二合一 | 2in1 | PC平板二合一 |

### 适配策略

- **默认适配**: `resources/default/` 适用于所有设备
- **设备特定**: `resources/[device]/` 覆盖默认配置
- **语言特定**: `resources/[lang]/` 提供本地化

## 相关文档

| 文档 | 链接 |
|------|------|
| 全局导航 | [SUMMARY.md](SUMMARY.md) |
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 构建系统 | [03_Build_System.md](03_Build_System.md) |
| 安全评审 | [05_Security.md](05_Security.md) |

## 更新日志

| 日期 | 变更 | 负责人 |
|------|------|--------|
| 2026-02-06 | 初始版本 | Wiki Generator |

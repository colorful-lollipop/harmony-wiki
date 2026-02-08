# 目录结构

## 目的

本文档描述 `system_resources` 模块的目录结构、各目录职责以及文件分类规范，帮助开发者快速定位所需资源文件。

## 适用范围

- **仓库路径**: `/base/global/system_resources`
- **排除范围**: 测试相关目录 (`test/`, `tests/`, `*_test.*`)

## 完整目录树

```
/base/global/system_resources/
├── fonts/                           # [1] 系统字体目录
│   ├── HarmonyOS_Sans.ttf           # 主字体 (默认+手表)
│   ├── HarmonyOS_Sans_Italic.ttf   # 斜体字体
│   ├── HarmonyOS_Sans_Condensed.ttf # 窄体字体
│   ├── HarmonyOS_Sans_Condensed_Italic.ttf
│   ├── HarmonyOS_Sans_Naskh_Arabic.ttf      # 阿拉伯语
│   ├── HarmonyOS_Sans_Naskh_Arabic_UI.ttf   # 阿拉伯语 UI
│   ├── HarmonyOS_Sans_SC.ttf        # 简体中文
│   ├── HarmonyOS_Sans_TC.ttf        # 繁体中文
│   ├── HarmonyOS_Sans_Digit.ttf     # 数字字体
│   ├── HarmonyOS_Sans_Digit_Medium.ttf
│   ├── HMOSColorEmojiCompat.ttf     # Emoji 兼容
│   ├── HMOSEmojiFlags.ttf           # 国旗 Emoji
│   ├── HMSymbolVF.ttf              # 系统图标 (默认)
│   ├── HMSymbolVF_watch.ttf        # 系统图标 (手表)
│   ├── hm_symbol_config.json        # 图标配置 (默认)
│   ├── hm_symbol_config_next.json   # 图标配置新版 (默认)
│   ├── hm_symbol_config_next_watch.json # 图标配置新版 (手表)
│   └── HarmonyOS_Sans_Notdef.ttf   # Fallback 字体
│
├── systemres/                       # [2] 系统资源包
│   ├── AppScope/                   # [2.1] 应用范围配置
│   │   ├── app.json                # 应用信息配置
│   │   └── resources/              # 多语言资源
│   │       ├── base/               # 默认语言
│   │       │   └── element/
│   │       │       └── string.json
│   │       └── zh_CN/              # 简体中文
│   │           └── element/
│   │               └── string.json
│   │
│   ├── main/                       # [2.2] 主资源
│   │   ├── module.json             # 模块配置 + 100+ 权限定义
│   │   └── resources/              # 设备/语言适配资源
│   │       ├── default/            # 默认设备
│   │       ├── tablet/            # 平板
│   │       ├── car/                # 车机
│   │       ├── wearable/           # 可穿戴
│   │       ├── tv/                 # 电视
│   │       ├── th-wearable/        # 穿戴 Thin
│   │       ├── 2in1/               # 二合一设备
│   │       └── [40+ 语言]/          # 多语言支持
│   │           ├── en_US/
│   │           ├── zh_CN/
│   │           ├── zh_TW/
│   │           ├── ja_JP/
│   │           ├── ko_KR/
│   │           └── ...
│   │
│   └── BUILD.gn                     # Hap 构建配置
│
├── BUILD.gn                        # [3] 根构建配置
├── bundle.json                     # [4] Bundle 配置
├── systemres.gni                   # [5] GN 包含配置 (字体列表)
├── README.md                       # [6] 项目说明 (英文)
├── README_zh.md                    # [7] 项目说明 (中文)
├── LICENSE                         # [8] 主许可证 (Apache-2.0)
├── LICENSE_Fonts                   # [9] 字体许可证
└── OAT.xml                         # [10] 开源审计配置
```

## 目录职责详解

### 1. fonts/ - 系统字体目录

**职责**: 存放系统级字体文件

| 子项 | 类型 | 用途 |
|------|------|------|
| `HarmonyOS_Sans*.ttf` | 可变字体 | 主字体，支持 6 种字重 |
| `*_SC.ttf` | 字体 | 简体中文变体 |
| `*_TC.ttf` | 字体 | 繁体中文变体 |
| `*_Arabic*.ttf` | 字体 | 阿拉伯语变体 |
| `HMSymbolVF*.ttf` | 符号字体 | 系统图标符号 |
| `hm_symbol_config*.json` | 配置 | 图标字体配置 |

**证据**: `BUILD.gn:28-36` 定义了字体文件的构建 target

**不含测试**: fonts 目录不包含任何测试文件

### 2. systemres/ - 系统资源包

**职责**: 存放系统资源包的源文件

#### 2.1 AppScope/ - 应用范围配置

**用途**: 定义应用级别的配置和资源

| 文件 | 用途 | 证据 |
|------|------|------|
| `app.json` | 应用基本信息 | bundleName: ohos.global.systemres |
| `resources/` | 多语言资源 | - |

**app.json 结构**:
```json
{
  "app": {
    "bundleName": "ohos.global.systemres",
    "icon": "$media:ohos_app_icon",
    "label": "$string:ohos_app_name",
    "singleton": true
  }
}
```

#### 2.2 main/ - 主资源模块

**用途**: 定义模块级别的资源

| 资源类型 | 用途 |
|----------|------|
| `module.json` | 模块配置 + 权限定义 |
| `resources/default/` | 默认设备资源 |
| `resources/[device]/` | 特定设备资源 |
| `resources/[lang]/` | 特定语言资源 |

**module.json 关键内容**:
- 设备类型适配 (default, tv, car, wearable, tablet, 2in1)
- 权限定义 (100+ 权限元数据)
- 模块类型 (entry)

**证据**: `module.json:1-42` 定义了模块配置

### 3. BUILD.gn - 根构建配置

**职责**: 定义顶层构建 targets

| Target | 类型 | 输出 |
|--------|------|------|
| `systemres_hap` | ohos_hap | SystemResources.hap |
| `ohos_fonts` | ohos_shared_headers | 头文件集合 |
| `copy_preview_fonts` | ohos_copy | 预览器字体 |
| `copy_preview_fonts_ext` | ohos_copy | 扩展预览器字体 |

**证据**: `BUILD.gn:1-79`

### 4. bundle.json - Bundle 配置

**用途**: 定义模块的元信息

| 字段 | 值 |
|------|-----|
| name | @ohos/system_resources |
| version | 4.0 |
| subsystem | global |
| part_name | system_resources |

**证据**: `bundle.json:1-67`

### 5. systemres.gni - GN 包含配置

**用途**: 定义被其他模块包含的变量

**关键变量**:
- `system_resources_support_ext`: 扩展支持开关
- `system_resources_font_feature_product`: 产品字体特性
- `sys_fonts_list`: 字体列表配置

**证据**: `systemres.gni:14-173`

## 文件分类规范

### 按文件类型分类

| 类型 | 文件 | 数量 |
|------|------|------|
| 字体文件 | `*.ttf` | 15 |
| 字体配置 | `*.json` | 3 |
| 构建配置 | `BUILD.gn`, `*.gni` | 3 |
| 模块配置 | `module.json`, `app.json` | 2 |
| Bundle 配置 | `bundle.json` | 1 |
| 许可证 | `LICENSE*` | 2 |
| 文档 | `README*.md` | 2 |

### 按功能分类

| 功能 | 相关文件 |
|------|----------|
| 字体管理 | `fonts/`, `BUILD.gn`, `systemres.gni` |
| 资源管理 | `systemres/`, `BUILD.gn` |
| 权限定义 | `module.json` |
| 构建打包 | `BUILD.gn`, `bundle.json` |

## 排除的目录

以下目录**不包含**在 Wiki 文档引用中（测试相关内容）:

| 目录模式 | 说明 |
|----------|------|
| `test/` | 测试目录 |
| `tests/` | 测试目录 |
| `unittest/` | 单元测试 |
| `unit_test/` | 单元测试 |
| `fuzz/` | 模糊测试 |
| `fuzztest/` | 模糊测试 |
| `*_test.*` | 测试文件 |
| `*_fuzzer.*` | 模糊测试文件 |

## 相关文档

| 文档 | 链接 |
|------|------|
| 全局导航 | [SUMMARY.md](SUMMARY.md) |
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 构建系统 | [03_Build_System.md](03_Build_System.md) |
| 系统资源 | [04_Resources.md](04_Resources.md) |

## 更新日志

| 日期 | 变更 | 负责人 |
|------|------|--------|
| 2026-02-06 | 初始版本 | Wiki Generator |

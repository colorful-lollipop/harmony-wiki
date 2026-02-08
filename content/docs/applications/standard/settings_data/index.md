# SettingsData - OpenHarmony 系统设置数据管理应用

## 项目简介

**SettingsData** 是 OpenHarmony 系统中预置的系统应用，为用户提供系统设置数据的访问存取服务。该应用负责存储和读取系统属性，如时间格式、屏幕亮度、音量等关键设置项。

### 核心能力

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| 数据持久化 | 使用 RDB 关系数据库存储设置数据 | `SettingsDBHelper.ets:162-174` |
| DataShare 数据共享 | 通过 DataAbility 对外提供数据访问接口 | `module.json5:22-32` |
| 多用户支持 | 支持创建/删除用户时管理对应数据表 | `UserChangeStaticSubscriber.ets:33-79` |
| 权限管控 | 敏感设置需 `MANAGE_SECURE_SETTINGS` 权限 | `DataExtAbility.ets:271-298` |
| 默认值加载 | 系统首次启动时从 `default_settings.json` 加载默认值 | `SettingsDBHelper.ets:273-323` |

### 技术栈

| 层级 | 技术/框架 | 用途 |
|------|----------|------|
| 应用框架 | ArkTS + Ability | 应用主框架 |
| 数据持久化 | `@ohos.data.relationalStore` | RDB 关系数据库 |
| 数据共享 | `@ohos.application.DataShareExtensionAbility` | DataAbility 实现 |
| 事件监听 | `@ohos.commonEventManager` | 用户变更事件订阅 |
| 配置管理 | `@ohos.data.preferences` | 首选项存储 |
| 音频控制 | `@ohos.multimedia.audio` | 音量设置操作 |

### 运行环境

| 环境要求 | 说明 |
|----------|------|
| SDK 版本 | compileSdkVersion 23 / compatibleSdkVersion 23 |
| 系统版本 | OpenHarmony 标准系统 |
| 设备类型 | default, tablet |
| 安全区域 | EL1 / EL2（根据数据库状态自动选择） |

---

## 快速开始

### 项目结构

```
settingsdata/
├── entry/src/main/
│   ├── ets/
│   │   ├── Application/
│   │   │   └── DataAbilityStage.ts      # DataAbility 入口
│   │   ├── DataAbility/
│   │   │   └── DataExtAbility.ets       # 数据能力实现
│   │   ├── StaticSubscriber/
│   │   │   └── UserChangeStaticSubscriber.ets  # 用户变更监听
│   │   ├── Utils/
│   │   │   ├── SettingsDBHelper.ets     # 数据库帮助器
│   │   │   ├── GlobalContext.ets        # 全局上下文
│   │   │   └── SettingsDataConfig.ets   # 配置常量
│   │   └── common/
│   │       └── Common.ts                # 公共类型定义
│   └── resources/
│       ├── base/
│       │   ├── element/                 # 资源文件
│       │   └── profile/
│       │       ├── data_share_config.json    # DataShare 配置
│       │       └── static_subscriber_config.json
│       └── rawfile/
│           └── default_settings.json    # 默认设置值
├── hvigor/                              # 构建工具
└── build-profile.json5                  # 构建配置
```

### 构建命令

```bash
# 使用 hvigor 构建 HAP 包
hvigor --path /Volumes/lexar/code/d/work/oh/applications/standard/settings_data assembleHap --product default
```

### 安装部署

构建生成的 HAP 包位于：
```
build/default/outputs/default/entry-default-signed.hap
```

---

## 文档导航

| 主题 | 链接 | 说明 |
|------|------|------|
| 项目详情 | [项目概览](./01_Project_Overview.md) | 核心能力与技术选型详解 |
| 代码结构 | [目录结构](./02_Directory_Structure.md) | 模块划分与文件组织 |
| 架构设计 | [架构设计](./03_Architecture.md) | 组件图、数据流、时序图 |
| API 接口 | [API 参考](./04_API_Reference.md) | DataShare 接口与 URI 配置 |
| 构建流程 | [构建与编译](./05_Build_and_Compilation.md) | hvigor 构建配置 |
| 安全分析 | [安全评审](./06_Security_Review.md) | 威胁模型与风险点 |

---

## 相关资源

- **OpenHarmony 官方文档**: https://developer.harmonyos.com
- **DataAbility 开发指南**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/database/dataability-guidelines.md
- **关系型数据库 RDB**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/database/rdb-guidelines.md

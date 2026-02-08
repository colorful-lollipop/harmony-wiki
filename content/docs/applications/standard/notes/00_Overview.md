# 项目概览

## 1. 项目定位

### 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **项目名称** | @ohos/notes (备忘录) |
| **所属子系统** | applications (系统应用) |
| **目标设备** | 标准系统设备 (RK3568等) |
| **应用类型** | 系统应用 (System App) |
| **License** | Apache License 2.0 |
| **版本** | 3.0 |

**证据位置**: `bundle.json:2-5`

```json
{
  "name": "@ohos/notes",
  "description": "notes app for standard system.",
  "version": "3.0",
  "license": "Apache License 2.0"
}
```

### 1.2 核心能力

备忘录应用提供以下核心功能：

| 功能 | 描述 | 代码位置 |
|------|------|----------|
| **笔记创建与编辑** | 支持富文本编辑、插入图片 | `features/src/main/ets/components/NoteContentComp.ets` |
| **文件夹管理** | 文件夹的增删改查、颜色标识 | `common/utils/src/main/ets/default/model/databaseModel/FolderData.ets` |
| **数据持久化** | 使用 relationalStore 存储笔记数据 | `common/utils/src/main/ets/default/baseUtil/RdbStoreUtil.ets` |
| **跨设备流转** | 支持设备间笔记迁移与同步 | `product/default/src/main/ets/MainAbility/MainAbility.ts:163-214` |
| **分布式文件同步** | 图片等资源的分布式存储 | `product/default/src/main/ets/MainAbility/MainAbility.ts:246-258` |
| **响应式布局** | 自适应手机/平板/折叠屏 | `product/default/src/main/ets/MainAbility/MainAbility.ts:260-275` |
| **收藏与置顶** | 笔记收藏、置顶功能 | `common/utils/src/main/ets/default/model/databaseModel/NoteData.ets:29-31` |

### 1.3 适用范围

| 维度 | 范围 |
|------|------|
| **操作系统** | OpenHarmony 标准系统 |
| **设备类型** | 手机、平板、折叠屏 |
| **API 版本** | API 14 (5.0.2) |
| **开发环境** | DevEco Studio 5.0.2 Release |

**证据位置**: `README_zh.md:141-145`

```
1.本示例仅支持标准系统上运行,支持设备RK3568。
2.本示例已适配API14版本SDK,SDK版本号(API Version 14 5.0.2)。
3.本示例需要使用DevEco Studio 5.0.2 Release版本才可编译运行。
```

## 2. 运行环境

### 2.1 系统依赖

本应用依赖以下系统能力：

| 依赖组件 | 用途 | 证据位置 |
|----------|------|----------|
| `ability_base` | Ability 基础能力 | `bundle.json:24` |
| `ability_runtime` | Ability 运行时 | `bundle.json:25` |
| `relational_store` | 关系型数据库 | `bundle.json:26` |
| `hiviewdfx_hilog_native` | 日志输出 | `bundle.json:27` |
| `web_webview` | 富文本编辑 (WebView) | `bundle.json:28` |
| `resourceManager` | 资源管理 | `bundle.json:29` |
| `medialibrary_standard` | 媒体库访问 | `bundle.json:30` |

**证据位置**: `bundle.json:22-33`

```json
"deps": {
  "components": [
    "ability_base",
    "ability_runtime",
    "relational_store",
    "hiviewdfx_hilog_native",
    "web_webview",
    "resourceManager",
    "medialibrary_standard"
  ]
}
```

### 2.2 运行时权限

本应用声明的权限（待补充，需检查 module.json5）

## 3. 关键概念

### 3.1 Stage 模型

本应用采用 OpenHarmony 的 **Stage 模型** 作为应用框架：

- **UIAbility**: `MainAbility.ts` - 应用的主入口 Ability
- **AbilityStage**: `AbilityStage.ts` - Stage 生命周期管理
- **ExtensionAbility**: （如有）用于特定扩展场景

**证据位置**: `product/default/src/main/ets/MainAbility/MainAbility.ts:31`

```typescript
export default class MainAbility extends UIAbility {
```

### 3.2 数据模型

| 模型 | 用途 | 主要字段 |
|------|------|----------|
| `NoteData` | 笔记数据 | id, title, uuid, folder_uuid, content_text, content_img |
| `FolderData` | 文件夹数据 | id, name, uuid, color, folder_type |
| `EnumData` | 枚举定义 | NoteType, Favorite, Delete, Top, FolderType |

**证据位置**: `common/utils/src/main/ets/default/model/databaseModel/`

### 3.3 存储结构

```
应用数据目录:
├── filesDir          # 应用私有文件目录
├── distributedFilesDir  # 分布式文件目录
└── database          # relationalStore 数据库
    └── Note.db       # 笔记数据库
```

## 4. 相关跳转

| 目标 | 链接 |
|------|------|
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 内部 API | [03_Inner_API.md](03_Inner_API.md) |
| 构建配置 | [04_GN_Targets.md](04_GN_Targets.md) |
| 安全评估 | [06_Security_Review.md](06_Security_Review.md) |

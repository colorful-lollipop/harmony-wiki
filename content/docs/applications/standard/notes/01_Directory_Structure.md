# 目录结构

## 1. 总体目录树

```
/applications/standard/notes
│
├── common/                          # 公共模块 (HAR)
│   ├── component/                   # 公共组件
│   │   └── src/main/ets/default
│   │
│   └── utils/                       # 工具类库 ⭐ 核心模块
│       └── src/main/ets/default
│           ├── baseUtil/            # 基础工具类
│           │   ├── LogUtil.ets      # 日志工具
│           │   ├── DateUtil.ets     # 日期工具
│           │   ├── NoteUtil.ets     # 笔记工具
│           │   ├── FolderUtil.ets   # 文件夹工具
│           │   ├── RdbStoreUtil.ets # 数据库工具
│           │   └── GlobalResourceManager.ets
│           │
│           ├── model/                # 数据模型
│           │   ├── databaseModel/   # 数据库相关模型
│           │   │   ├── NoteData.ets     # 笔记数据类
│           │   │   ├── FolderData.ets   # 文件夹数据类
│           │   │   ├── AttachmentData.ets # 附件数据类
│           │   │   ├── EnumData.ets      # 枚举定义
│           │   │   └── SysDefData.ets    # 系统默认数据
│           │   │
│           │   └── searchModel/
│           │       └── SearchModel.ets
│           │
│           └── access/               # 访问控制
│               └── MediaLibraryAccess.ets
│
├── features/                        # 功能模块 (HAR)
│   ├── src/main/ets/components/     # UI 组件
│   │   ├── NoteListComp.ets         # 笔记列表组件
│   │   ├── NoteContentComp.ets      # 笔记内容组件
│   │   ├── NoteContentCompPortrait.ets # 竖屏笔记内容
│   │   ├── FolderListComp.ets       # 文件夹列表组件
│   │   └── CusDialogComp.ets        # 自定义对话框
│   │
│   ├── src/main/resources/          # 资源文件
│   └── module.json5                 # HAR 模块配置
│
├── product/                         # 产品配置
│   └── default/                     # 默认产品
│       └── src/main/ets
│           ├── MainAbility/          # 主 Ability
│           │   └── MainAbility.ts
│           │
│           ├── Application/          # 应用配置
│           │   └── AbilityStage.ts
│           │
│           └── pages/                # 页面
│               ├── MyNoteHome.ets    # 我的笔记主页
│               ├── NoteHome.ets      # 笔记主页
│               ├── NoteHomePortrait.ets # 竖屏笔记主页
│               └── NoteContentHome.ets # 笔记内容页
│
├── signature/                       # 签名配置
├── AppScope/                        # 应用作用域资源
├── wiki/                            # 工程文档 ⭐
└── build-profile.json5             # 构建配置
```

## 2. 模块职责

### 2.1 common/utils - 工具类模块

**类型**: HAR (Static Library)
**路径**: `common/utils/`
**职责**: 提供数据模型、数据库操作、工具方法

| 子目录 | 文件 | 职责 |
|--------|------|------|
| `baseUtil/` | `LogUtil.ets` | 日志输出封装 |
| | `DateUtil.ets` | 日期格式化 |
| | `NoteUtil.ets` | 笔记业务逻辑 |
| | `FolderUtil.ets` | 文件夹业务逻辑 |
| | `RdbStoreUtil.ets` | relationalStore 数据库操作 |
| | `GlobalResourceManager.ets` | 资源管理 |
| `model/databaseModel/` | `NoteData.ets` | 笔记数据模型 |
| | `FolderData.ets` | 文件夹数据模型 |
| | `AttachmentData.ets` | 附件数据模型 |
| | `EnumData.ets` | 枚举类型定义 |
| | `SysDefData.ets` | 系统默认数据 |
| `model/searchModel/` | `SearchModel.ets` | 搜索模型 |
| `access/` | `MediaLibraryAccess.ets` | 媒体库访问控制 |

**证据位置**: `common/utils/src/main/ets/` 目录结构

### 2.2 features - 功能组件模块

**类型**: HAR (Static Library)
**路径**: `features/`
**职责**: 提供 UI 组件库

| 组件文件 | 职责 |
|----------|------|
| `NoteListComp.ets` | 笔记列表展示与交互 |
| `NoteContentComp.ets` | 笔记富文本内容编辑 |
| `NoteContentCompPortrait.ets` | 竖屏模式笔记内容 |
| `FolderListComp.ets` | 文件夹列表管理 |
| `CusDialogComp.ets` | 通用对话框组件 |

**证据位置**: `features/src/main/ets/components/`

### 2.3 product/default - 主应用模块

**类型**: Feature Ability
**路径**: `product/default/`
**职责**: 应用入口、页面路由、生命周期管理

| 目录/文件 | 职责 |
|----------|------|
| `MainAbility/` | UIAbility 入口，生命周期管理 |
| `AbilityStage/` | Stage 生命周期 |
| `pages/` | 页面组件 |
| `resources/` | 资源文件 |

## 3. 资源目录结构

### 3.1 资源分类

```
resources/
├── base/                      # 默认资源
│   ├── element/              # 基础元素
│   │   ├── string.json       # 字符串资源
│   │   └── color.json        # 颜色资源
│   │
│   ├── profile/              # 配置文件
│   │   └── main_pages.json   # 页面路由配置
│   │
│   └── float.json            # 浮点值资源
│
├── zh_CN/                     # 中文资源
│   └── element/
│       └── string.json
│
└── en_US/                    # 英文资源
    └── element/
        └── string.json
```

### 3.2 页面路由配置

**文件**: `product/default/src/main/resources/base/profile/main_pages.json`

```json
{
  "src": [
    "pages/MyNoteHome",
    "pages/NoteContentHome",
    "pages/NoteHome",
    "pages/NoteHomePortrait"
  ]
}
```

## 4. 数据流概览

```
┌─────────────────────────────────────────────────────────────┐
│                    用户交互层                                │
│  pages/ (NoteHome, NoteContentHome, MyNoteHome, etc.)      │
└────────────────────────┬────────────────────────────────────┘
                         │ 页面路由
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    组件层                                    │
│  features/components/ (NoteListComp, NoteContentComp, etc.)│
└────────────────────────┬────────────────────────────────────┘
                         │ 组件调用
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    业务逻辑层                                │
│  common/utils/baseUtil/ (NoteUtil, FolderUtil, etc.)       │
└────────────────────────┬────────────────────────────────────┘
                         │ 业务方法
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    数据层                                    │
│  common/utils/baseUtil/RdbStoreUtil.ets                     │
│  └── relationalStore (Note.db)                              │
└─────────────────────────────────────────────────────────────┘
```

## 5. 相关跳转

| 目标 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 内部 API | [03_Inner_API.md](03_Inner_API.md) |

# 目录结构

> 模块划分与职责说明

## 顶层目录

```
/applications/standard/launcher/
├── common/                           # 公共能力层 (HAR)
├── feature/                          # 公共特性层 (HAR)
│   ├── appcenter/                   # 应用中心
│   ├── bigfolder/                   # 智能文件夹
│   ├── form/                         # 桌面卡片
│   ├── gesturenavigation/            # 手势导航
│   ├── numbadge/                    # 数字角标
│   ├── pagedesktop/                 # 工作区
│   ├── recents/                     # 最近任务
│   ├── settings/                    # 桌面设置
│   └── smartdock/                   # Dock 工具栏
├── product/                          # 业务形态层 (HAP)
│   ├── phone/                       # 手机形态
│   └── pad/                          # 平板形态
├── signature/                        # 签名证书
├── AppScope/                         # 应用范围资源
├── docs/                             # 开发文档
├── figures/                          # 文档图片
└── wiki/                             # Wiki 文档
```

## 层级职责

### 1. common/ - 公共能力层

**职责**：提供所有桌面形态必须依赖的基础能力。

| 子目录 | 职责 |
|--------|------|
| `utils/` | 工具类（Log、Trace、CheckEmptyUtils） |
| `manager/` | 管理器（RdbStoreManager、PreferencesHelper） |
| `model/` | 数据模型 |
| `viewmodel/` | 视图模型基类 |
| `config/` | 配置管理 |
| `constants/` | 常量定义（CommonConstants、StyleConstants） |
| `interface/` | 接口定义 |
| `bean/` | 业务实体类 |
| `cache/` | 缓存管理 |
| `uicomponents/` | 公共 UI 组件 |

**导出 API**：
```typescript
// 通过 @ohos/common 引用
import { Log, Trace, CommonConstants, windowManager } from '@ohos/common';
import { AppGrid, AppIcon, FolderComponent } from '@ohos/common/component';
```

### 2. feature/ - 公共特性层

**职责**：提供可复用的功能组件，被各桌面形态引用。

| 模块 | 职责 | 导出 API |
|------|------|----------|
| `appcenter` | 应用中心网格 | `AppGridViewModel`, `AppListViewModel`, `appCenterPreLoader` |
| `bigfolder` | 智能文件夹 | `BigFolderModel`, `BigFolderViewModel`, `BigFolderStyleConfig` |
| `form` | 桌面卡片 | `FormViewModel`, `FormStyleConfig`, `formPreLoader` |
| `gesturenavigation` | 手势导航 | `GestureNavigationManager` |
| `numbadge` | 数字角标 | `NumBadgeManager` |
| `pagedesktop` | 工作区 | `PageDesktopViewModel`, `PageDesktopDragHandler` |
| `recents` | 最近任务 | `RecentMissionsViewModel`, `RecentMissionsStage` |
| `settings` | 桌面设置 | `SettingItemsManager` |
| `smartdock` | Dock 栏 | `SmartDockStyleConfig`, `SmartDockLayoutConfig` |

### 3. product/ - 业务形态层

**职责**：定义具体设备的桌面实现。

| 模块 | 职责 | 类型 |
|------|------|------|
| `phone` | 手机桌面 | entry (HAP) |
| `pad` | 平板桌面 | entry (HAP) |

**子结构**：
```
product/phone/src/main/
├── ets/
│   ├── MainAbility/           # 主入口
│   │   └── MainAbility.ts    # ServiceExtension 实现
│   ├── Application/          # 应用级初始化
│   └── pages/                # 页面组件
│       ├── EntryView.ets     # 桌面入口
│       ├── RecentView.ets    # 最近任务
│       └── ...
└── resources/                # 资源文件
    ├── base/                 # 默认资源
    ├── zh_CN/                # 中文资源
    └── en_US/                # 英文资源
```

## 典型模块内部结构

```
feature/pagedesktop/
├── index.ts                  # 模块导出入口
└── src/main/ets/
    ├── default/
    │   ├── common/           # 公共逻辑
    │   │   ├── PageDesktopPreLoader.ts
    │   │   ├── PageDesktopDragHandler.ts
    │   │   └── PageDesktopGridStyleConfig.ts
    │   ├── viewmodel/        # 视图模型
    │   │   └── PageDesktopViewModel.ts
    │   ├── layout/          # 布局组件
    │   │   └── PageDesktopLayout.ets
    │   ├── common/
    │   │   └── components/  # 子组件
    │   │       ├── AppItem.ets
    │   │       ├── FolderItem.ets
    │   │       └── ...
    │   └── pages/           # 页面
    └── resources/           # 资源
```

## 资源目录结构

```
AppScope/                         # 应用级资源
└── resources/
    ├── base/
    │   ├── element/             # 元素资源（颜色、字符串等）
    │   └── media/               # 媒体资源（图片等）

feature/*/src/main/resources/     # 模块级资源
├── base/
│   ├── element/
│   ├── profile/                 # 配置文件
│   └── media/
├── zh_CN/
└── en_US/

product/phone/src/main/resources/  # 产品级资源
├── base/
│   ├── element/
│   ├── profile/                 # 页面路由配置
│   └── media/
└── zh_CN/
```

## 配置文件

| 文件 | 作用 |
|------|------|
| `module.json5` | 模块配置（入口、权限、组件） |
| `main_pages.json` | 页面路由配置 |
| `build-profile.json5` | 构建配置（模块依赖、签名） |

## 相关文档

| 文档 | 链接 |
|------|------|
| 架构说明 | [02_Architecture.md](02_Architecture.md) |
| 对外 API | [03_APIs.md](03_APIs.md) |
| 构建配置 | [05_Build.md](05_Build.md) |

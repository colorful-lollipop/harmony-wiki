# 对外 API

> ArkTS 模块导出清单

## 概述

本文档记录 applications_launcher 项目中各模块导出的公共 API。由于本项目为 ArkTS 应用，不包含传统 N-API（C/C++ 导出层），所有 API 通过 ArkTS 模块系统导出。

> **证据**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:44` - MainAbility 继承 ServiceExtension，无 N-API 调用

## 模块导出清单

### common 模块

**引用方式**: `import { ... } from '@ohos/common'`

common 模块是 Launcher 项目的核心公共能力层，提供以下导出：

#### 工具类

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 工具类 | `Log` | class | 日志输出（showInfo/showDebug/showWarn/showError/showFatal） |
| 工具类 | `Trace` | class | 性能追踪 |
| 工具类 | `PinyinSort` | class | 拼音排序 |
| 工具类 | `CheckEmptyUtils` | class | 空值检查 |
| 工具类 | `ObjectCopyUtil` | class | 对象拷贝工具 |

> **证据**: `common/index.ts:16-18`

#### 常量

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 常量 | `CommonConstants` | class | 公共常量（BundleName、AbilityName 等） |
| 常量 | `StyleConstants` | class | 样式常量 |
| 常量 | `EventConstants` | class | 事件常量 |
| 常量 | `PresetStyleConstants` | class | 预设样式常量 |
| 常量 | `RecentsStyleConstants` | class | 最近任务样式常量 |
| 常量 | `FormConstants` | class | 卡片常量 |
| 常量 | `NumberConstants` | class | 数字常量 |

> **证据**: `common/index.ts:43-50`

#### 管理器

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 管理器 | `windowManager` | instance | 窗口管理器 |
| 管理器 | `RdbStoreManager` | class | RDB 数据库管理器 |
| 管理器 | `launcherAbilityManager` | instance | Launcher 能力管理器 |
| 管理器 | `FormListInfoCacheManager` | class | 卡片列表缓存 |
| 管理器 | `ResourceManager` | class | 资源管理器 |
| 管理器 | `localEventManager` | instance | 本地事件管理器 |
| 管理器 | `DisplayManager` | class | 显示管理器 |
| 管理器 | `navigationBarCommonEventManager` | instance | 导航栏事件管理 |
| 管理器 | `PreferencesHelper` | class | 首选项助手 |
| 管理器 | `FormManager` | class | 卡片管理器 |
| 管理器 | `BadgeManager` | class | 角标管理器 |
| 管理器 | `amsMissionManager` | class | AMS 任务管理器 |
| 管理器 | `InputMethodManager` | class | 输入法管理器 |
| 管理器 | `settingsDataManager` | class | 设置数据管理器 |
| 管理器 | `CloseAppManager` | class | 关闭应用管理器 |
| 管理器 | `layoutConfigManager` | instance | 布局配置管理器 |

> **证据**: `common/index.ts:52-66`, `common/index.ts:83-94`

#### 视图模型

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 视图模型 | `LayoutViewModel` | class | 布局视图模型基类 |
| 视图模型 | `SettingsModel` | class | 设置模型 |
| 视图模型 | `AppModel` | class | 应用模型 |
| 视图模型 | `FormModel` | class | 卡片模型 |
| 视图模型 | `PageDesktopModel` | class | 工作区模型 |
| 视图模型 | `RecentMissionsModel` | class | 最近任务模型 |
| 视图模型 | `SettingsModelObserver` | class | 设置模型观察者 |
| 视图模型 | `AtomicServiceAppModel` | class | 原子服务应用模型 |

> **证据**: `common/index.ts:72-80`

#### 数据实体

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 数据实体 | `AppItemInfo` | class | 应用项信息 |
| 数据实体 | `MissionInfo` | class | 任务信息 |
| 数据实体 | `CardItemInfo` | class | 卡片项信息 |
| 数据实体 | `FolderData` | class | 文件夹数据 |
| 数据实体 | `FolderItemInfo` | class | 文件夹项信息 |
| 数据实体 | `MenuInfo` | class | 菜单信息 |
| 数据实体 | `DockItemInfo` | class | Dock 项信息 |
| 数据实体 | `RecentMissionInfo` | class | 最近任务信息 |
| 数据实体 | `RecentBundleMissionInfo` | class | 最近任务 Bundle 信息 |
| 数据实体 | `LauncherDragItemInfo` | class | 拖拽项信息 |
| 数据实体 | `SnapShotInfo` | class | 快照信息 |
| 数据实体 | `SettingItemInfo` | class | 设置项信息 |

> **证据**: `common/index.ts:29-42`

#### 接口定义

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 接口 | `DragArea` | interface | 拖拽区域接口 |
| 接口 | `DragItemPosition` | interface | 拖拽项位置接口 |
| 接口 | `GridLayoutInfo` | interface | 网格布局信息 |
| 接口 | `ILayoutConfig` | interface | 布局配置接口 |
| 接口 | `FormLayoutConfig` | interface | 卡片布局配置 |
| 接口 | `FolderLayoutConfig` | interface | 文件夹布局配置 |
| 接口 | `AppListStyleConfig` | interface | 应用列表样式配置 |
| 接口 | `AppGridStyleConfig` | interface | 应用网格样式配置 |
| 接口 | `RecentsModeConfig` | interface | 最近任务模式配置 |
| 接口 | `PageDesktopModeConfig` | interface | 工作区模式配置 |
| 接口 | `PageDesktopLayoutConfig` | interface | 工作区布局配置 |
| 接口 | `PageDesktopAppModeConfig` | interface | 工作区应用模式配置 |
| 接口 | `LauncherLayoutStyleConfig` | interface | 启动器布局样式配置 |
| 接口 | `FormDetailLayoutConfig` | interface | 卡片详情布局配置 |

> **证据**: `common/index.ts:68-94`

#### 基类

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 基类 | `BaseStage` | class | 舞台基类 |
| 基类 | `BaseDragHandler` | class | 拖拽处理器基类 |
| 基类 | `BaseViewModel` | class | 视图模型基类 |
| 基类 | `BaseModulePreLoader` | class | 模块预加载器基类 |
| 基类 | `BaseCloseAppHandler` | class | 关闭应用处理器基类 |
| 基类 | `BaseStartAppHandler` | class | 启动应用处理器基类 |

> **证据**: `common/index.ts:20-27`

#### 配置管理

| 导出类型 | 符号名 | 类型 | 说明 |
|----------|--------|------|------|
| 配置 | `SettingItemsConfig` | class | 设置项配置 |
| 配置 | `SettingItemsManager` | class | 设置项管理器 |
| 配置 | `SettingItemOptionsChecker` | class | 设置项选项检查器 |

> **证据**: `common/index.ts:100-103`

### UI 组件

**引用方式**: `import { ... } from '@ohos/common/component'`

| 组件名 | 类型 | 说明 |
|--------|------|------|
| `AppIcon` | component | 应用图标 |
| `AppName` | component | 应用名称 |
| `AppGrid` | component | 应用网格 |
| `AppBubble` | component | 应用气泡 |
| `AppMenu` | component | 应用菜单 |
| `FolderComponent` | component | 文件夹组件 |
| `FolderItem` | component | 文件夹项 |
| `FormItemComponent` | component | 卡片项 |
| `FormManagerDialog` | component | 卡片管理对话框 |
| `RemoveFormDialog` | component | 删除卡片对话框 |
| `UninstallDialog` | component | 卸载对话框 |
| `ScrollerComponent` | component | 滚动组件 |
| `OverlayAppIcon` | component | 叠加应用图标 |
| `RemoteWindowWrapper` | component | 远程窗口包装器 |

---

### pagedesktop 模块

**引用方式**: `import { ... } from '@ohos/pagedesktop'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `pageDesktopPreLoader` | function | 工作区预加载器 |
| `PageDesktopDragHandler` | class | 工作区拖拽处理器 |
| `PageDesktopGridStyleConfig` | class | 工作区网格样式配置 |
| `PageDesktopViewModel` | class | 工作区视图模型 |

**引用组件**: `import { ... } from '@ohos/pagedesktop/component'`

| 组件 | 说明 |
|------|------|
| `PageDesktopLayout` | 工作区布局 |

---

### smartdock 模块

**引用方式**: `import { ... } from '@ohos/smartdock'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `smartDockPreLoader` | function | Dock 预加载器 |
| `SmartDockStyleConfig` | class | Dock 样式配置 |
| `SmartDockLayoutConfig` | class | Dock 布局配置 |

**引用组件**: `import { ... } from '@ohos/smartdock/component'`

| 组件 | 说明 |
|------|------|
| `SmartDock` | Dock 栏组件 |

---

### recents 模块

**引用方式**: `import { ... } from '@ohos/recents'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `RecentMissionsStage` | class | 最近任务舞台 |
| `RecentsStyleConstants` | class | 最近任务样式常量 |
| `RecentMissionsViewModel` | class | 最近任务视图模型 |

**引用组件**: `import { ... } from '@ohos/recents/component'`

| 组件 | 说明 |
|------|------|
| `RecentMissionsSingleLayout` | 单列布局 |
| `RecentMissionsDoubleLayout` | 双列布局 |

---

### form 模块

**引用方式**: `import { ... } from '@ohos/form'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `formPreLoader` | function | 卡片预加载器 |
| `FormStyleConfig` | class | 卡片样式配置 |
| `FormViewModel` | class | 卡片视图模型 |
| `FormDetailLayoutConfig` | class | 卡片详情布局配置 |

**引用组件**: `import { ... } from '@ohos/form/component'`

| 组件 | 说明 |
|------|------|
| `FormManagerComponent` | 卡片管理组件 |
| `FormServiceComponent` | 卡片服务组件 |

---

### appcenter 模块

**引用方式**: `import { ... } from '@ohos/appcenter'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `AppGridViewModel` | class | 应用网格视图模型 |
| `AppListViewModel` | class | 应用列表视图模型 |
| `appCenterPreLoader` | function | 应用中心预加载器 |

**引用组件**: `import { ... } from '@ohos/appcenter/component'`

| 组件 | 说明 |
|------|------|
| `AppGridLayout` | 应用网格布局 |

---

### bigfolder 模块

**引用方式**: `import { ... } from '@ohos/bigfolder'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `BigFolderModel` | class | 智能文件夹模型 |
| `BigFolderViewModel` | class | 智能文件夹视图模型 |
| `bigFolderPreLoader` | function | 智能文件夹预加载器 |
| `BigFolderStyleConfig` | class | 智能文件夹样式配置 |
| `BigFolderConstants` | class | 智能文件夹常量 |
| `BigFolderStyleConstants` | class | 智能文件夹样式常量 |

**引用组件**: `import { ... } from '@ohos/bigfolder/component'`

| 组件 | 说明 |
|------|------|
| `FolderOpenComponent` | 文件夹打开组件 |
| `FolderAppListDialog` | 文件夹应用列表对话框 |

---

### settings 模块

**引用方式**: `import { ... } from '@ohos/settings'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `SettingItemsManager` | class | 设置项管理器 |

---

### gesturenavigation 模块

**引用方式**: `import { ... } from '@ohos/gesturenavigation'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `GestureNavigationManager` | class | 手势导航管理器 |

---

### numbadge 模块

**引用方式**: `import { ... } from '@ohos/numbadge'`

| 导出 | 类型 | 说明 |
|------|------|------|
| `NumBadgeManager` | class | 数字角标管理器 |

---

## 系统 API 引用

### 常用系统 API

| 模块 | 用途 | 引用示例 |
|------|------|---------|
| `@ohos.app.ability.ServiceExtensionAbility` | ServiceExtension 基类 | `import ServiceExtension from '@ohos.app.ability.ServiceExtensionAbility';` |
| `@ohos.app.ability.Want` | 意图对象 | `import Want from '@ohos.app.ability.Want';` |
| `@ohos.window` | 窗口管理 | `import window from '@ohos.window';` |
| `@ohos.display` | 显示管理 | `import display from '@ohos.display';` |
| `@ohos.hilog` | 日志系统 | `import hilog from '@ohos.hilog';` |
| `@ohos.multimodalInput.inputConsumer` | 输入消费 | `import inputConsumer from '@ohos.multimodalInput.inputConsumer';` |
| `@ohos.multimodalInput.keyCode` | 按键码 | `import { KeyCode } from '@ohos.multimodalInput.keyCode';` |

---

## API 使用示例

### 日志输出

```typescript
import { Log } from '@ohos/common';

Log.showInfo(TAG, `onCreate start`);
Log.showDebug(TAG, `window created: ${windowName}`);
Log.showError(TAG, `failed to load: ${error}`);
```

### 窗口管理

```typescript
import { windowManager } from '@ohos/common';

// 创建窗口
windowManager.createWindow(
    globalThis.desktopContext,
    windowManager.DESKTOP_WINDOW_NAME,
    windowManager.DESKTOP_RANK,
    'pages/desktop',
    true,
    callback
);
```

### 视图模型

```typescript
import { PageDesktopViewModel } from '@ohos/pagedesktop';

const viewModel = PageDesktopViewModel.getInstance();
viewModel.updateDesktopInfo();
```

### 事件通信

```typescript
import { localEventManager, EventConstants } from '@ohos/common';

// 发送事件
localEventManager.sendLocalEventSticky(
    EventConstants.EVENT_REQUEST_FORM_ITEM_VISIBLE, 
    null
);

// 监听事件
localEventManager.on(
    EventConstants.EVENT_REQUEST_FORM_ITEM_VISIBLE,
    (data) => {
        // 处理事件
    }
);
```

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 内部 API | [04_Inner_API.md](04_Inner_API.md) |
| 架构说明 | [02_Architecture.md](02_Architecture.md) |
| 调用链图谱 | [appendix/Callgraphs.md](appendix/Callgraphs.md) |

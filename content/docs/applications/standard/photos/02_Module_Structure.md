# 模块结构

## 1. 目录结构总览

```
photos/
├── AppScope/                          # 应用级资源配置
│   └── resources/
│       ├── base/
│       │   ├── element/               # 字符串、图片等资源
│       │   └── media/                  # 应用图标
│       └── rawfile/                   # 原始文件
│
├── common/                             # 共享基础模块 (HAR)
│   └── src/main/ets/
│       └── default/
│           ├── interface/              # 接口定义
│           ├── model/                   # 数据模型
│           │   └── browser/            # 浏览器相关模型
│           │       ├── album/           # 相册模型
│           │       ├── dataObserver/   # 数据观察者
│           │       └── photo/          # 照片模型
│           ├── access/                 # 访问控制
│           └── utils/                  # 工具类
│
├── feature/                            # 功能模块 (HAR)
│   ├── browser/                        # 图片浏览功能
│   │   └── src/main/ets/default/
│   ├── editor/                         # 图片编辑功能
│   │   └── src/main/ets/default/
│   │       ├── base/                   # 编辑基础
│   │       ├── crop/                   # 裁剪组件
│   │       ├── utils/                  # 编辑工具
│   │       └── view/                   # 编辑视图
│   ├── formAbility/                    # FA 卡片编辑
│   │   └── src/main/ets/default/view/
│   ├── thirdselect/                    # 第三方选择
│   │   └── src/main/ets/
│   │       ├── default/utils/          # 选择工具
│   │       └── view/                   # 选择视图
│   └── timeline/                        # 日视图功能
│       └── src/main/ets/
│           └── view/
│
├── product/                            # 产品模块 (entry)
│   └── phone/
│       └── src/main/ets/
│           ├── Application/            # 应用入口
│           │   └── AbilityStage.ts     # 应用级别初始化
│           ├── MainAbility/            # 主能力
│           │   └── MainAbility.ts      # 页面路由分发
│           ├── FormAbility/            # FA 卡片
│           ├── ServiceExt/            # 服务扩展
│           ├── picker/                 # UI 扩展选择器
│           ├── DeleteAbility/          # 删除对话框
│           ├── SaveAbility/            # 保存对话框
│           ├── common/                 # phone 私有通用
│           ├── view/                   # phone 私有视图
│           ├── workers/                # Worker 线程
│           └── pages/                   # 页面组件
│
├── signature/                          # 签名配置
│   └── material/                       # 签名材料
│
├── wiki/                               # 本文档目录
│
├── hvigor/                             # hvigor 配置
│   └── hvigorfile.js                   # 构建脚本
│
├── build-profile.json5                 # 构建配置
├── module.json5                        # 模块配置 (各模块)
└── bundle.json                         # 组件配置
```

---

## 2. 模块详细说明

### 2.1 photos_common (共享基础模块)

**类型**: HAR (静态共享包)  
**路径**: `common/src/main/`  
**职责**: 提供所有模块共用的基础能力

```
common/src/main/ets/default/
├── interface/                          # 接口定义
│   ├── BrowserDataInterface.ts
│   ├── BrowserDataFactory.ts
│   ├── BrowserOperationFactory.ts
│   └── MenuOperationFactory.ts
│
├── model/                             # 数据模型
│   ├── browser/
│   │   ├── album/                     # 相册模型
│   │   │   ├── AlbumSetDataSource.ts
│   │   │   ├── AlbumSetCallback.ts
│   │   │   ├── AlbumDataImpl.ts
│   │   │   └── AlbumInfo.ts
│   │   ├── dataObserver/              # 数据观察
│   │   │   ├── MediaObserver.ts       # 媒体变化观察
│   │   │   ├── MediaObserverCallback.ts
│   │   │   └── CommonObserverCallback.ts
│   │   ├── photo/                     # 照片模型
│   │   │   ├── UriDataSource.ts
│   │   │   ├── FifoCache.ts           # 缓存
│   │   │   ├── EventPipeline.ts
│   │   │   └── JumpSourceToMain.ts
│   │   ├── AbsDataSource.ts           # 数据源基类
│   │   ├── BrowserDataImpl.ts
│   │   ├── LoadingListener.ts
│   │   ├── SelectManager.ts           # 选择管理
│   │   └── SelectionState.ts
│   │
│   └── AbstractAlbumDataSource.ts
│
├── access/                             # 访问控制
│   └── UserFileManagerAccess.ts       # 媒体文件访问
│
└── utils/                              # 工具类
    ├── Log.ts                          # 日志工具
    ├── BreakPointSystem.ts             # 断点系统
    ├── DateUtil.ts                     # 日期工具
    ├── ImageUtil.ts                   # 图片工具
    ├── StringUtil.ts                  # 字符串工具
    ├── MathUtil.ts                    # 数学工具
    ├── UiUtil.ts                      # UI 工具
    ├── DataStoreUtil.ts                # 数据存储
    ├── ReportToBigDataUtil.ts         # 大数据上报
    ├── ErrUtil.ts                     # 错误工具
    ├── BroadCast.ts                   # 广播工具
    ├── WindowUtil.ts                  # 窗口工具
    └── TraceControllerUtils.ts        # 追踪控制
```

**导出内容**:

| 导出项 | 类型 | 说明 |
|--------|------|------|
| `@ohos/common` | 包引用 | 包含常量、工具、Model 等 |

**关键类**:

| 类名 | 职责 | 路径 |
|------|------|------|
| `UserFileManagerAccess` | 媒体文件访问封装 | `access/` |
| `MediaObserver` | 媒体数据变化监听 | `model/browser/dataObserver/` |
| `SelectManager` | 照片选择状态管理 | `model/browser/` |
| `BroadCastManager` | 跨组件消息分发 | `utils/` |

> **证据**: `MainAbility.ts:32` - `import { ..., Log, ..., UserFileManagerAccess } from '@ohos/common'`

---

### 2.2 photos_browser (图片浏览模块)

**类型**: HAR  
**路径**: `feature/browser/src/main/`  
**职责**: 提供图片/视频浏览的核心功能

```
feature/browser/src/main/ets/default/
├── view/                              # 视图组件
└── (更多子目录和组件)
```

**导出内容**: 
- 浏览相关 UI 组件
- 浏览数据处理逻辑

---

### 2.3 photos_editor (图片编辑模块)

**类型**: HAR  
**路径**: `feature/editor/src/main/`  
**职责**: 提供图片编辑能力（裁剪、滤镜等）

```
feature/editor/src/main/ets/default/
├── base/                              # 基础组件
├── crop/                              # 裁剪功能
│   └── CropView.ts                    # 裁剪视图
├── utils/                             # 编辑工具
└── view/                              # 编辑视图组件
```

**关键组件**:

| 组件 | 职责 |
|------|------|
| `CropView` | 图片裁剪 |
| `FilterView` | 滤镜效果 |
| `EditPanel` | 编辑面板 |

---

### 2.4 photos_formAbility (FA 卡片模块)

**类型**: HAR  
**路径**: `feature/formAbility/src/main/`  
**职责**: 提供 FA 卡片相关的编辑功能

```
feature/formAbility/src/main/ets/default/view/
└── FormEditorView/                    # FA 编辑视图
```

---

### 2.5 photos_thirdselect (第三方选择模块)

**类型**: HAR  
**路径**: `feature/thirdselect/src/main/`  
**职责**: 提供第三方应用图片选择功能

```
feature/thirdselect/src/main/ets/
├── default/
│   └── utils/
│       └── SmartPickerUtils.ts        # 智能选择器工具
└── view/                              # 选择视图组件
```

**关键工具**:

| 工具 | 职责 | 证据 |
|------|------|------|
| `SmartPickerUtils` | 初始化选择器参数 | `MainAbility.ts:122,131` |

---

### 2.6 photos_timeline (日视图模块)

**类型**: HAR  
**路径**: `feature/timeline/src/main/`  
**职责**: 提供按日期展示的日视图功能

```
feature/timeline/src/main/ets/
└── view/                              # 时间线视图
    └── TimelinePage/                   # 日视图页面
```

**关键类**:

| 类名 | 职责 | 证据 |
|------|------|------|
| `TimelineDataSourceManager` | 日视图数据源管理 | `MainAbility.ts:72-73` |

---

### 2.7 phone_photos (主入口模块)

**类型**: entry (HAP)  
**路径**: `product/phone/src/main/`  
**职责**: 集成所有 HAR 模块，提供应用入口

```
product/phone/src/main/ets/
├── Application/
│   └── AbilityStage.ts                # 应用级别初始化
│
├── MainAbility/
│   └── MainAbility.ts                 # 页面路由分发
│
├── FormAbility/
│   └── FormAbility.ts                  # FA 卡片入口
│
├── ServiceExt/
│   └── ServiceExtAbility.ts            # 服务扩展
│
├── picker/
│   └── PickerUIExtensionAbility.ts    # UI 扩展选择器
│
├── DeleteAbility/
│   └── DeleteUIExtensionAbility.ts    # 删除对话框
│
├── SaveAbility/
│   └── SaveUIExtensionAbility.ts      # 保存对话框
│
├── common/                            # phone 私有通用
│   └── ...
│
├── view/                              # phone 私有视图
│   └── ...
│
├── workers/                           # Worker 线程
│
└── pages/                             # 页面组件
    ├── index.ets                       # 首页
    ├── PhotoBrowser.ets                # 照片浏览页
    ├── PhotoGridPage.ets               # 宫格页
    ├── ThirdSelectPhotoGridPage.ets   # 第三方选择页
    ├── FormEditorPage.ets              # FA 编辑页
    └── ...
```

**入口组件**:

| 组件 | 类型 | 职责 |
|------|------|------|
| `AbilityStage` | 应用级别 | 全局上下文初始化 |
| `MainAbility` | Page | 外部调用路由分发 |
| `FormAbility` | Form | FA 卡片入口 |
| `ServiceExtAbility` | Service | 后台服务 |

---

## 3. 页面清单

### 3.1 主应用页面

| 页面 | 路由 | 职责 |
|------|------|------|
| **首页** | `pages/index` | 图库主入口，展示相册/日视图 |
| **照片浏览** | `pages/PhotoBrowser` | 大图浏览，支持缩放/滑动 |
| **宫格视图** | `pages/PhotoGridPage` | 图片网格展示 |
| **新建相册** | `pages/NewAlbumPage` | 创建新相册 |
| **默认照片页** | `pages/DefaultPhotoPage` | FA 默认展示页 |
| **FA 编辑页** | `pages/FormEditorPage` | FA 卡片编辑 |
| **视频浏览** | `pages/VideoBrowser` | 视频播放 |

> **证据**: `MainAbility.ts:203` - `setUIContent(this.context, 'pages/index', null)`

### 3.2 第三方选择页面

| 页面 | 路由 | 职责 |
|------|------|------|
| **第三方选择宫格** | `pages/ThirdSelectPhotoGridPage` | 第三方选择图片 |
| **第三方选择相册** | `pages/ThirdSelectAlbumSetPage` | 第三方选择相册页 |
| **第三方浏览** | `pages/ThirdSelectPhotoBrowser` | 第三方图片浏览 |

> **证据**: `MainAbility.ts:243-275` - 路由到 `ThirdSelectPhotoGridPage`

### 3.3 UI 扩展页面

| 页面 | 类型 | 职责 |
|------|------|------|
| **删除对话框** | UIExtension | 图片删除确认 |
| **保存对话框** | UIExtension | 图片保存确认 |
| **编辑对话框** | UIExtension | 编辑操作面板 |
| **资源删除页** | UIExtension | 删除资源确认 |

---

## 4. 模块依赖矩阵

| 从 → 到 | common | browser | editor | formAbility | thirdselect | timeline | phone |
|---------|--------|---------|--------|-------------|-------------|----------|-------|
| **common** | - | 依赖 | 依赖 | 依赖 | 依赖 | 依赖 | 依赖 |
| **browser** | 被依赖 | - | 可选 | 可选 | 可选 | 可选 | 被依赖 |
| **editor** | 依赖 | 可选 | - | 可选 | 可选 | 可选 | 被依赖 |
| **formAbility** | 依赖 | 可选 | 可选 | - | 可选 | 可选 | 被依赖 |
| **thirdselect** | 依赖 | 可选 | 可选 | 可选 | - | 可选 | 被依赖 |
| **timeline** | 依赖 | 可选 | 可选 | 可选 | 可选 | - | 被依赖 |
| **phone** | 依赖 | 依赖 | 依赖 | 依赖 | 依赖 | 依赖 | - |

---

## 5. 命名约定

### 5.1 文件命名

| 类型 | 约定 | 示例 |
|------|------|------|
| 页面 | `PageName.ets` | `PhotoBrowser.ets` |
| 组件 | `ComponentName.ets` | `ActionBar.ets` |
| 工具类 | `FunctionName.ts` | `DateUtil.ts` |
| 数据模型 | `DataName.ts` | `MediaDataItem.ts` |
| 数据源 | `FeatureDataSource.ts` | `AlbumSetDataSource.ts` |

### 5.2 路径约定

| 组件类型 | 路径模式 |
|----------|----------|
| 页面 | `product/phone/src/main/ets/pages/` |
| 主视图 | `feature/[name]/src/main/ets/view/` |
| Model | `common/src/main/ets/default/model/` |
| 工具 | `common/src/main/ets/default/utils/` |

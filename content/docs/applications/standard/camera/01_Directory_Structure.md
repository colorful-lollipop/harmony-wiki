# 目录结构与模块职责

## 仓库根目录结构

```
applications/standard/camera/
├── AppScope/                    # 应用级配置
│   └── app.json5               # 应用配置（bundleName、版本等）
├── bundle.json                  # OpenHarmony 组件配置
├── build-profile.json5          # 构建配置（签名、模块、SDK版本）
├── oh-package.json5            # 依赖管理
├── hvigorfile.js               # 构建脚本入口
├── common/                      # Common 层（HAR）
├── features/                    # Feature 层（HAR）
├── product/                     # Product 层（Entry）
├── signature/                   # 签名配置
└── figures/                     # 文档图片
```

**证据**: `ls -la` 根目录

## 模块类型说明

| 模块 | 类型 | 路径 | 职责 |
|------|------|------|------|
| phone | entry | `product/phone/` | 手机形态应用入口 |
| tablet | entry | `product/tablet/` | 平板形态应用入口 |
| common | har | `common/` | 公共服务与组件 |
| photo | har | `features/photo/` | 拍照功能模块 |
| video | har | `features/video/` | 录像功能模块 |
| multi | har | `features/multi/` | 多机位协同模块 |

**证据**: `build-profile.json5:41-82`

```json
{
  "modules": [
    { "name": "phone", "srcPath": "./product/phone" },
    { "name": "tablet", "srcPath": "./product/tablet" },
    { "name": "common", "srcPath": "./common" },
    { "name": "multi", "srcPath": "./features/multi" },
    { "name": "photo", "srcPath": "./features/photo" },
    { "name": "video", "srcPath": "./features/video" }
  ]
}
```

## Common 层详细结构

```
common/
├── src/main/ets/default/
│   ├── camera/                 # 相机核心服务
│   │   ├── CameraService.ts   # 相机能力封装（主类）
│   │   ├── CameraPlatformCapability.ts  # 平台能力检测
│   │   ├── ThumbnailGetter.ts # 缩略图获取
│   │   └── SaveCameraAsset.ts # 媒体资源保存
│   ├── featurecommon/          # 通用 UI 组件
│   │   ├── animate/           # 动画效果
│   │   ├── assistivegridview/ # 辅助网格
│   │   ├── bigtext/           # 大文字组件
│   │   ├── cameraswitcher/    # 相机切换
│   │   ├── geolocation/       # 地理位置
│   │   ├── moreList/          # 更多列表
│   │   ├── playsound/         # 播放声音
│   │   ├── preferences/       # 偏好设置存储
│   │   ├── screenlock/        # 屏幕锁定管理
│   │   ├── settingview/       # 设置界面
│   │   │   ├── model/         # 数据模型
│   │   │   ├── phone/         # 手机端组件
│   │   │   └── tablet/        # 平板端组件
│   │   ├── shutterbutton/     # 快门按钮
│   │   ├── tabbar/            # 标签栏
│   │   ├── thumbnail/         # 缩略图显示
│   │   ├── timelapseview/     # 延时视图
│   │   └── zoomview/          # 缩放视图
│   ├── featureservice/         # Feature 管理服务
│   │   ├── FeatureManager.ts  # Feature 管理器
│   │   ├── FunctionId.ts      # 功能标识
│   │   ├── IModeMap.ts        # 模式映射接口
│   │   └── ModeAssembler.ts   # 模式组装
│   ├── function/               # 业务功能
│   │   ├── BaseFunction.ts    # 功能基类
│   │   ├── CameraBasicFunction.ts  # 相机基础功能
│   │   ├── CaptureFunction.ts # 拍照功能
│   │   ├── RecordFunction.ts  # 录像功能
│   │   └── ZoomFunction.ts    # 缩放功能
│   ├── redux/                  # 状态管理
│   │   ├── actions/           # Actions
│   │   ├── reducers/          # Reducers
│   │   └── store.ets          # Store 定义
│   ├── setting/                # 设置管理
│   │   ├── SettingManager.ts  # 设置管理器
│   │   ├── settingitem/       # 设置项定义
│   │   └── storage/           # 存储实现
│   ├── utils/                  # 工具类
│   │   ├── Log.ts             # 日志工具
│   │   ├── Constants.ts       # 常量定义
│   │   ├── GlobalContext.ts   # 全局上下文
│   │   ├── ComponentPosition.ts  # 组件位置
│   │   ├── ComponentIdKeys.ts    # 组件 ID
│   │   ├── DateTimeUtil.ts    # 日期时间
│   │   └── ReportUtil.ts      # 上报工具
│   └── worker/                 # 多线程/事件
│       ├── CameraWorker.ts    # Camera Worker
│       ├── WorkerManager.ts   # Worker 管理
│       └── eventbus/          # 事件总线
│           ├── EventBus.ts
│           └── EventBusManager.ts
├── src/main/resources/         # 资源文件
└── index.ets                   # 模块导出入口
```

**统计**: Common 层约 94 个 .ets/.ts 文件

**证据**: `find ./common/src/main/ets/default -type f \( -name "*.ets" -o -name "*.ts" \) | wc -l`

## Feature 层结构

### Photo 模块
```
features/photo/
└── src/main/ets/photo/
    ├── PhotoMode.ts           # 拍照模式主类
    └── PhotoModeParam.ts      # 拍照参数
```

### Video 模块
```
features/video/
└── src/main/ets/video/
    ├── VideoMode.ts           # 录像模式主类
    └── VideoModeParam.ts      # 录像参数
```

### Multi 模块
```
features/multi/
└── src/main/ets/multi/
    ├── MultiMode.ts           # 多机位模式主类
    └── MultiModeParam.ts      # 多机位参数
```

## Product 层结构

### Phone 模块
```
product/phone/
└── src/main/
    ├── ets/
    │   ├── Application/
    │   │   └── AbilityStage.ts    # AbilityStage
    │   ├── MainAbility/
    │   │   ├── MainAbility.ts     # 主 Ability
    │   │   └── ExtensionPickerAbility.ts  # 扩展选择器
    │   ├── FormAbility/
    │   │   └── FormAbility.ts     # 卡片 Ability
    │   ├── pages/                 # 页面
    │   │   ├── index.ets          # 主页面
    │   │   ├── Control.ets        # 控制栏
    │   │   ├── FootBar.ets        # 底部栏
    │   │   ├── PreviewArea.ets    # 预览区
    │   │   ├── SettingView.ets    # 设置视图
    │   │   ├── BigVideoTimer.ets  # 大视频定时器
    │   │   ├── SmallVideoTimer.ets # 小视频定时器
    │   │   └── ThirdPreviewView.ets # 第三方预览
    │   └── common/                # 产品级公共代码
    │       ├── ModeMap.ts
    │       └── ModeConfig.ts
    ├── resources/                 # 资源
    │   ├── base/                  # 基础资源
    │   ├── en_US/                 # 英文资源
    │   ├── zh_CN/                 # 中文资源
    │   └── rawfile/               # 原始文件
    └── module.json5               # 模块配置
```

### Tablet 模块
结构与 Phone 类似，页面组件带有 "Land" 后缀表示横屏适配：
- `indexLand.ets` - 平板主页面
- `ControlLand.ets` - 平板控制栏
- `FootBarLand.ets` - 平板底部栏
- `PreviewAreaLand.ets` - 平板预览区

## 模块依赖关系

```
                    ┌─────────────┐
                    │    Phone    │
                    └──────┬──────┘
                           │ depends on
                    ┌──────┴──────┐
                    │    Tablet   │
                    └──────┬──────┘
                           │ depends on
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────┴────┐       ┌─────┴─────┐      ┌─────┴─────┐
   │  Photo  │       │   Video   │      │   Multi   │
   └────┬────┘       └─────┬─────┘      └─────┬─────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │ depends on
                    ┌──────┴──────┐
                    │    Common   │
                    └─────────────┘
```

**依赖声明位置**:
- Phone/Tablet 通过 `oh-package.json5` 或 hvigor 配置依赖 Common 和 Features
- Feature 模块依赖 Common

## 关键配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| bundle.json | `./bundle.json` | 组件发布配置 |
| build-profile.json5 | `./build-profile.json5` | 构建配置 |
| module.json5 | `product/phone/src/main/module.json5` | Phone 模块配置 |
| module.json5 | `product/tablet/src/main/module.json5` | Tablet 模块配置 |
| oh-package.json5 | `./oh-package.json5` | 依赖管理 |
| hvigorfile.ts | `*/hvigorfile.ts` | 模块构建脚本 |

## 代码组织规范

### 文件命名
- **类文件**: 大驼峰命名，如 `CameraService.ts`
- **组件文件**: 大驼峰命名，如 `PreviewArea.ets`
- **工具文件**: 大驼峰命名，如 `Log.ts`
- **配置文件**: 小写 + 下划线，如 `module.json5`

### 目录命名
- **层级目录**: 小写，如 `default/`, `main/`
- **功能目录**: 小写，如 `camera/`, `utils/`
- **组件目录**: 小写，如 `shutterbutton/`

### 导出规范
- Common 层统一在 `index.ets` 中导出公共接口
- 导出格式: `export { ClassName } from './path/to/Class'`

**证据**: `common/index.ets:16-106`

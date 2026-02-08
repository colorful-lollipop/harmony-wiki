# 目录结构

## 目的

本文档说明 FilePicker 应用的目录结构、模块职责和文件组织规范，帮助开发者快速定位代码。

## 适用范围

本文档适用于：
- 需要修改 FilePicker 功能的开发者
- 需要理解代码组织的维护人员

## 关键结论

1. **项目采用模块化设计**：entry（主模块）和 audiopicker（音频选择器模块）
2. **清晰的职责划分**：entryability（能力入口）、pages（UI 页面）、base（基础工具）、databases（数据模型）、taskpool（任务池）
3. **纯 ArkTS/TypeScript 实现**：所有源代码为 .ets 或 .ts 文件，无 C/C++ 代码

## 项目目录树

```
/Volumes/lexar/code/d/work/oh/applications/standard/filepicker/
├── AppScope/                      # 应用级配置
│   └── app.json5                  # bundle 配置（bundleName、版本等）
├── entry/                         # 主模块（文件选择器）
│   ├── src/main/
│   │   ├── ets/                   # ArkTS/TypeScript 源码
│   │   │   ├── entryability/       # Ability 入口
│   │   │   │   ├── MainAbility.ets              # UIAbility 主入口
│   │   │   │   └── FilePickerUIExtAbility.ets  # UIExtensionAbility 入口
│   │   │   ├── pages/              # UI 页面
│   │   │   │   ├── browser/storage/  # 浏览器页面
│   │   │   │   │   ├── MyPhone.ets              # 文件选择主页面
│   │   │   │   │   └── FilePickerBatchAuth.ets  # 批量授权页面
│   │   │   │   ├── component/       # UI 组件
│   │   │   │   │   ├── common/          # 通用组件
│   │   │   │   │   │   ├── TopBar.ets       # 顶部导航栏
│   │   │   │   │   │   ├── FilesList.ets    # 文件列表
│   │   │   │   │   │   ├── Loading.ets      # 加载组件
│   │   │   │   │   │   └── DialogComponent.ets # 对话框组件
│   │   │   │   │   ├── myphone/        # MyPhone 专用组件
│   │   │   │   │   │   ├── BreadCrumb.ets  # 面包屑导航
│   │   │   │   │   │   └── FilesList.ets    # 文件列表
│   │   │   │   │   └── dialog/         # 对话框组件
│   │   │   │   │       ├── DownloadDialog.ets      # 下载对话框
│   │   │   │   │       ├── FileMkdirDialog.ets     # 新建文件夹对话框
│   │   │   │   │       └── FileMoveDialog.ets     # 移动文件对话框
│   │   │   │   ├── PathPicker.ets          # 路径选择页面
│   │   │   │   └── DownloadAuth.ets        # 下载授权页面
│   │   │   ├── databases/          # 数据模型
│   │   │   │   └── model/
│   │   │   │       ├── FileData.ets            # 文件数据模型
│   │   │   │       ├── FileAssetModel.ets     # 文件资源模型
│   │   │   │       ├── BasicDataSource.ets    # 基础数据源
│   │   │   │       ├── MimeType.ets           # MIME 类型模型
│   │   │   │       └── base/
│   │   │   │           └── FileBase.ts             # 文件基类
│   │   │   ├── taskpool/           # 任务池
│   │   │   │   ├── manager/
│   │   │   │   │   └── TaskManager.ts               # 任务池管理器
│   │   │   │   ├── base/
│   │   │   │   │   ├── BaseTask.ts               # 任务基类
│   │   │   │   │   └── TaskExecutor.ts          # 任务执行器
│   │   │   │   ├── const/
│   │   │   │   │   └── TaskConst.ts             # 任务常量
│   │   │   │   ├── util/
│   │   │   │   │   └── TaskpoolUtils.ets         # 任务池工具
│   │   │   │   └── task/
│   │   │   │       ├── BatchGrantPermissionTask.ets  # 批量授权任务
│   │   │   │       └── BatchAuthFilterTask.ets      # 批量过滤任务
│   │   │   ├── base/               # 基础工具类
│   │   │   │   ├── utils/                 # 工具函数
│   │   │   │   │   ├── FilePickerUtil.ets     # FilePicker 工具
│   │   │   │   │   ├── FileAccessExec.ets     # FileAccess 执行器
│   │   │   │   │   ├── FileMimeTypeUtil.ets   # MIME 类型工具
│   │   │   │   │   ├── FileUriUtil.ets       # URI 工具
│   │   │   │   │   ├── FileUtil.ets           # 文件操作工具（TODO: 文件未找到）
│   │   │   │   │   ├── FsUtil.ets            # 文件系统工具
│   │   │   │   │   ├── Tools.ets             # 通用工具
│   │   │   │   │   ├── Common.ets            # 通用函数
│   │   │   │   │   ├── StringUtil.ets        # 字符串工具
│   │   │   │   │   ├── ObjectUtil.ets        # 对象工具
│   │   │   │   │   ├── ArrayUtil.ets         # 数组工具
│   │   │   │   │   ├── DateTimeUtil.ets      # 日期时间工具
│   │   │   │   │   ├── UiUtil.ets           # UI 工具
│   │   │   │   │   ├── EventBus.ets         # 事件总线
│   │   │   │   │   └── AbilityCommonUtil.ets  # Ability 通用工具
│   │   │   │   ├── constants/            # 常量定义
│   │   │   │   │   ├── Constant.ts           # 通用常量
│   │   │   │   │   ├── ErrorCodeConst.ts     # 错误码常量
│   │   │   │   │   ├── FilePickerItems.ts    # FilePicker 项目常量
│   │   │   │   │   ├── UiConstant.ets       # UI 常量
│   │   │   │   │   ├── FolderRecord.ts      # 文件夹记录常量
│   │   │   │   │   └── PageRouteConst.ts     # 路由常量
│   │   │   │   ├── config/               # 配置
│   │   │   │   │   └── AppConfig.ts        # 应用配置
│   │   │   │   └── model/               # 数据模型
│   │   │   │       └── StartModeOptions.ets  # 启动模式选项
│   │   │   ├── application/         # 应用入口
│   │   │   │   └── PickerAbilityStage.ets          # AbilityStage
│   │   │   └── log/                # 日志
│   │   │       └── Logger.ts           # 日志工具
│   │   └── resources/           # 资源文件
│   │       ├── base/               # 基础资源
│   │       ├── zh_CN/             # 中文资源
│   │       └── en_US/             # 英文资源
│   └── hvigorfile.js           # Hvigor 构建脚本
├── audiopicker/                   # 音频选择器模块
│   └── src/main/
│       ├── ets/                   # ArkTS 源码
│       │   ├── audiopickerability/ # AudioPicker Ability 入口
│       │   │   └── AudioPickerUIExtensionAbility.ets
│       │   ├── pages/              # UI 页面
│       │   │   ├── card/
│       │   │   │   ├── AudioPickerView.ets           # 音频选择视图
│       │   │   │   └── AudioPickerExtension.ets     # AudioPicker 扩展
│       │   │   ├── dialog/
│       │   │   │   └── SafetyTipDialog.ets         # 安全提示对话框
│       │   │   └── viewmodel/
│       │   │       └── AudioPickerViewModel.ets     # 视图模型
│       │   ├── basemvvm/           # MVVM 基类
│       │   │   ├── AbsBaseViewData.ets
│       │   │   ├── AbsBaseViewModel.ets
│       │   │   └── ViewState.ets
│       │   ├── localresource/       # 本地资源管理
│       │   │   ├── localaudio/
│       │   │   │   ├── LocalAudioManager.ets   # 本地音频管理
│       │   │   │   └── LocalAudioFile.ets      # 本地音频文件
│       │   │   └── LocalResourceManager.ets  # 本地资源管理
│       │   ├── data/               # 数据
│       │   │   └── ObservedArray.ets
│       │   ├── constant/           # 常量
│       │   │   └── Constants.ets
│       │   └── audiopreference/    # 音频首选项
│       │       └── AudioPickerPreference.ets
│       └── resources/           # 资源文件
├── doc/                          # 文档
├── figures/                      # 架构图
├── signature/                    # 签名证书
├── hvigor/                      # Hvigor 工具
├── wiki/                        # Wiki 文档（本目录）
│   ├── README.md
│   ├── SUMMARY.md
│   ├── 00_Overview.md
│   ├── 01_Directory_Structure.md
│   ├── 02_Architecture.md
│   ├── 03_Public_API.md
│   ├── 04_Internal_API.md
│   ├── 05_Build_System.md
│   ├── 06_Permissions.md
│   ├── 07_Security_Review.md
│   ├── 08_FAQ.md
│   ├── _work/
│   │   ├── NOTES.md
│   │   └── PLAN.md
│   └── appendix/
│       ├── Callgraphs.md
│       └── Config_Flags.md
├── build-profile.json5            # 应用级构建配置
├── oh-package.json5              # 包配置
├── hvigorfile.js                # 构建入口
├── README.md                    # 项目说明
├── LICENSE                      # Apache 2.0 许可证
└── .git/                       # Git 仓库
```

## 模块职责

### entry 模块（主模块）

**定位**：文件选择和保存的核心功能模块

| 子目录 | 职责 | 关键文件 | 代码行数（约） |
|--------|-------|-----------|---------------|
| entryability | Ability 入口和生命周期管理 | MainAbility.ets, FilePickerUIExtAbility.ets | 250 |
| pages | UI 页面和用户交互 | MyPhone.ets, PathPicker.ets, DownloadAuth.ets | 800 |
| component | 可复用的 UI 组件 | TopBar.ets, FilesList.ets, DialogComponent.ets | 500 |
| databases | 数据模型和数据源 | FileData.ets, FileAssetModel.ets | 300 |
| taskpool | 并发任务处理 | TaskManager.ts, BatchGrantPermissionTask.ets | 250 |
| base | 基础工具和常量 | FilePickerUtil.ets, FileAccessExec.ets | 1500 |

**证据**：目录结构基于实际文件扫描

### audiopicker 模块（音频选择器）

**定位**：独立的音频文件选择功能模块

| 子目录 | 职责 | 关键文件 | 代码行数（约） |
|--------|-------|-----------|---------------|
| audiopickerability | AudioPicker Ability 入口 | AudioPickerUIExtensionAbility.ets | 70 |
| pages | UI 页面和 MVVM 架构 | AudioPickerView.ets, AudioPickerViewModel.ets | 400 |
| localresource | 本地音频资源管理 | LocalAudioManager.ets, LocalAudioFile.ets | 200 |
| basemvvm | MVVM 基类 | AbsBaseViewModel.ets, ViewState.ets | 100 |

**证据**：`audiopicker/src/main/module.json5:54` - type: "sysPicker/audioPicker"

## 文件统计

| 模块 | .ets 文件数 | .ts 文件数 | 总计 |
|--------|-------------|-------------|-------|
| entry | 52 | 26 | 78 |
| audiopicker | 15 | 0 | 15 |
| **总计** | **67** | **26** | **93** |

**证据**：通过 `find` 命令统计（见 NOTES.md）

## 代码组织规范

### 命名规范

| 类型 | 规范 | 示例 |
|------|-------|------|
| Ability 入口 | XAbility.ets | MainAbility.ets, FilePickerUIExtAbility.ets |
| 页面 | PascalCase.ets | MyPhone.ets, PathPicker.ets |
| 组件 | PascalCase.ets | TopBar.ets, FilesList.ets |
| 工具类 | XUtil.ets | FilePickerUtil.ets, FileAccessExec.ets |
| 数据模型 | PascalCase.ets | FileData.ets, StartModeOptions.ets |
| 常量 | PascalCase.ts | Constant.ts, ErrorCodeConst.ts |

### 目录组织原则

1. **按职责分组**：entryability、pages、component、databases、taskpool、base
2. **公共代码下沉**：base 目录包含跨模块共享的工具类
3. **UI 组件独立**：component 目录按功能细分（common、myphone、dialog）
4. **数据模型集中**：databases/model 统一管理数据结构

**证据**：实际目录结构分析

## 关键文件索引

### Ability 入口

| 文件 | 作用 | 行数 |
|------|------|------|
| MainAbility.ets | UIAbility 主入口，处理 choose/save 模式 | 101 |
| FilePickerUIExtAbility.ets | UIExtensionAbility 入口，支持模态弹窗模式 | 145 |
| AudioPickerUIExtensionAbility.ets | AudioPicker UIExtensionAbility 入口 | 66 |

### 页面

| 文件 | 作用 | 行数 |
|------|------|------|
| MyPhone.ets | 文件选择主页面 | 374 |
| PathPicker.ets | 路径选择页面，处理文件保存 | 262 |
| DownloadAuth.ets | 下载授权页面 | 117 |
| FilePickerBatchAuth.ets | 批量授权页面 | 100+ |

### 工具类

| 文件 | 作用 | 行数 |
|------|------|------|
| FilePickerUtil.ets | Want 解析、结果返回、权限授权 | 362 |
| FileAccessExec.ets | FileAccess API 封装、文件列表获取 | 214 |
| TaskManager.ts | 任务池管理、并发控制 | 134 |

### 数据模型

| 文件 | 作用 | 行数 |
|------|------|------|
| StartModeOptions.ets | 启动模式选项数据模型 | TODO |
| FileData.ets | 文件数据模型、数据源 | TODO |
| FileAssetModel.ets | 文件资源模型、媒体类型工具 | TODO |

## 相关跳转

- [概览](00_Overview.md) - 了解项目定位
- [架构设计](02_Architecture.md) - 深入理解架构
- [对外 API](03_Public_API.md) - 学习调用方式

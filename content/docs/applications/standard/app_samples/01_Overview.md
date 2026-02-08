# 项目概述

## 仓库定位

`app_samples` 是 OpenHarmony 应用示例仓库，提供一系列**独立完整的应用示例**，帮助开发者快速熟悉：

- OpenHarmony SDK API 的使用方法
- Stage 应用模型的开发模式
- ArkTS 声明式 UI 开发范式
- 常见业务场景的实现方案

> **证据**: [README.md](README.md) - 仓库根目录说明文件

## 快速开始

### 环境要求

| 组件 | 要求 |
|-----|------|
| DevEco Studio | 6.0.0.43 或更高版本 |
| SDK | API 20（6.0.0(20)） |
| ArkTS | 1.2 版本 |
| 运行环境 | 标准系统设备（Phone） |

### 导入示例

1. 打开 DevEco Studio
2. 选择 **Import Project**
3. 选择示例目录（如 `code/ArkTS-Sta/ComponentSample`）
4. 等待构建完成
5. 连接设备并运行

### 独立下载示例

```bash
git init
git config core.sparsecheckout true
echo code/ArkTS1.2/[SampleName]/ > .git/info/sparse-checkout
git remote add origin https://gitee.com/openharmony/applications_app_samples.git
git pull
```

> **证据**: 各示例 README.md 中的下载说明

## 示例列表

| 名称 | 功能描述 | 关键技术点 |
|-----|---------|----------|
| **ComponentSample** | 组件复用示例 | 列表二级联动、自定义日历、跨文件组件复用、懒加载（LazyForEach） |
| **CameraSample** | 相机拍照示例 | startAbilityForResult 拉起相机、Image 组件、评论图片列表 |
| **FilesSample** | 文件操作示例 | 大文件复制、@ohos.zlib 压缩/解压、Rawfile 处理 |
| **BackgroundblurSample** | 背景模糊效果 | 待分析 |
| **ContainernestedslideSample** | 容器嵌套滑动 | 待分析 |
| **CustomanimationtabSample** | 自定义动画 Tab | 待分析 |
| **CustomviewSample** | 自定义视图 | 待分析 |
| **DownloadSample** | 下载功能 | 待分析 |

## 开发规范

### Stage 模型

所有示例均基于 **Stage 模型** 开发：

```
entry/src/main/
├── ets/
│   ├── entryability/     # 入口 Ability
│   └── pages/           # 页面组件
└── module.json5         # 模块配置
```

### ArkTS 编码规范

- 使用 **ArkTS 1.2** 语法
- 遵循 OpenHarmony SDK API 调用规范
- 错误处理使用 `BusinessError` 类型
- 日志使用 `hilog` 模块

### 目录结构规范

```
entry/src/main/ets/
└── pages/
    └── [FeatureName]/
        ├── *.ets              # 主页面
        ├── components/       # 子组件
        ├── view/            # 视图组件（可选）
        └── *.ets            # 辅助文件
```

### 资源管理

```
entry/src/main/resources/
├── base/
│   ├── element/          # 字符串、颜色、尺寸
│   ├── media/           # 图片资源
│   └── profile/         # 页面配置
└── ...
```

## 依赖说明

| 依赖类型 | 说明 |
|---------|------|
| 系统 API | 使用 OpenHarmony SDK 内置 API |
| 本地资源 | rawfile 资源打包到 HAP |
| 外部依赖 | 部分示例可能依赖系统相机等能力 |

## 权限要求

| 权限 | 说明 |
|-----|------|
| 无 | 大部分示例无需特殊权限 |
| 相机 | CameraSample 可能需要相机权限 |

> **证据**: 各示例 README.md 中的"相关权限"章节

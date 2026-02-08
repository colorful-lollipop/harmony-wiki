# 目录结构

## 仓库根目录

```
app_samples/
├── code/                     # 示例代码目录
│   └── ArkTS-Sta/           # ArkTS Stage 模型示例
│       ├── ComponentSample/  # 组件复用示例
│       ├── CameraSample/     # 相机示例
│       ├── FilesSample/      # 文件操作示例
│       ├── BackgroundblurSample/
│       ├── ContainernestedslideSample/
│       ├── CustomanimationtabSample/
│       ├── CustomviewSample/
│       └── DownloadSample/
├── wiki/                     # Wiki 文档目录
├── README.md                # 仓库说明
├── README_zh.md             # 中文说明
├── LICENSE                  # 开源协议
├── OAT.xml                 # OpenAI 规范检查配置
└── changelog.md            # 变更日志
```

## 单个示例目录结构

以 **ComponentSample** 为例：

```
ComponentSample/
├── AppScope/                   # 应用级配置
│   └── app.json5              # 应用配置
├── entry/                      # 主模块
│   └── src/main/
│       ├── ets/
│       │   ├── entryability/  # 入口 Ability
│       │   │   ├── EntryAbility.ets
│       │   │   └── GlobalContext.ets
│       │   └── pages/        # 页面组件
│       │       ├── Index.ets
│       │       ├── SecondaryLinkage/
│       │       ├── CustomCalendar/
│       │       ├── DynamicAttributes/
│       │       └── HighlyLoaded/
│       ├── resources/        # 资源目录
│       │   ├── base/
│       │   │   ├── element/  # 元素资源
│       │   │   │   ├── color.json
│       │   │   │   ├── float.json
│       │   │   │   ├── integer.json
│       │   │   │   └── string.json
│       │   │   ├── media/    # 媒体资源
│       │   │   └── profile/  # 配置文件
│       │   │       ├── backup_config.json
│       │   │       └── main_pages.json
│       │   └── dark/         # 深色主题
│       └── module.json5     # 模块配置
├── build-profile.json5        # 构建配置
├── code-linter.json5         # 代码检查配置
├── hvigor/                   # hvigor 构建工具
├── hvigorfile.ts            # hvigor 构建脚本
├── oh-package.json5         # 依赖配置
├── ohosTest.md              # 测试说明
└── README.md                # 示例说明
```

## 模块职责说明

### 顶层目录

| 目录/文件 | 职责 |
|---------|------|
| `code/` | 存放所有应用示例 |
| `wiki/` | 工程文档（本文档） |
| `README.md` | 仓库级别说明 |
| `OAT.xml` | 静态代码检查配置 |

### 示例模块（entry/）

| 路径 | 职责 |
|-----|------|
| `ets/entryability/` | 应用入口，负责初始化和生命周期管理 |
| `ets/pages/` | UI 页面，包含所有视图组件 |
| `resources/base/element/` | 字符串、颜色、尺寸等基础资源 |
| `resources/base/media/` | 图片资源 |
| `resources/base/profile/` | 页面路由配置 |
| `module.json5` | 模块声明、权限、依赖配置 |

### 构建配置

| 文件 | 职责 |
|-----|------|
| `build-profile.json5` | 定义产品、SDK 版本、构建模式 |
| `hvigorfile.ts` | hvigor 构建任务定义 |
| `code-linter.json5` | 代码规范检查配置 |

## 页面目录结构规范

```
pages/[FeatureName]/
├── *.ets                    # 主页面组件
├── components/              # 私有子组件
│   └── *.ets               # 组件实现
├── view/                   # 视图组件（可选）
│   └── *.ets
├── [DataType].ets          # 数据类型定义（可选）
└── [Helper].ets            # 辅助函数（可选）
```

### 示例：ComponentSample 页面结构

```
pages/
├── Index.ets                      # 首页
├── SecondaryLinkage/
│   ├── SecondaryLinkage.ets       # 二级联动页面
│   ├── SecondaryLinkExample.ets   # 实现页面
│   └── DataType.ets               # 数据类型
├── CustomCalendar/
│   ├── CustomCalendar.ets         # 日历页面
│   ├── CalendarView.ets           # 日历视图
│   ├── components/
│   │   ├── GetDate.ets            # 日期获取
│   │   └── MonthDataSource.ets     # 懒加载数据源
│   └── ...
├── DynamicAttributes/
├── HighlyLoaded/
```

## 资源目录规范

```
resources/
├── base/
│   ├── element/
│   │   ├── color.json            # 颜色定义
│   │   ├── float.json            # 浮点数定义
│   │   ├── integer.json          # 整数定义
│   │   └── string.json           # 字符串资源
│   ├── media/                    # 图片资源
│   │   ├── icon_*.png
│   │   └── *.jpg
│   └── profile/
│       ├── backup_config.json    # 备份配置
│       └── main_pages.json       # 页面路由
└── dark/                         # 深色主题
    └── element/
        └── color.json
```

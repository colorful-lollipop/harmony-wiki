# 目录结构

## 顶层目录

```
/security_privacy_center/
├── AppScope/                          # 应用级配置信息
├── entry/                             # 主模块目录（核心代码）
├── hvigor/                            # hvigor 构建配置
├── signature/                         # 签名材料
├── wiki/                              # 文档目录（本Wiki）
├── build-profile.json5                # 工程级构建配置
├── hvigorfile.ts                      # 工程级构建脚本
├── LICENSE                            # Apache 2.0 许可证
├── oh-package.json5                   # 工程依赖配置
├── OAT.xml                            # 开源合规配置
└── README.md                          # 项目说明
```

## entry 模块详细结构

```
entry/
├── build-profile.json5                # 模块级构建配置
├── hvigorfile.ts                      # 模块级构建脚本
├── oh-package.json5                   # 模块依赖配置
└── src/
    ├── main/
    │   ├── module.json5               # 模块能力声明
    │   ├── ets/                       # ArkTS/ETS 源码
    │   │   ├── common/                # 公共代码目录
    │   │   │   ├── base/              # MVI 架构基类
    │   │   │   │   ├── BaseIntent.ets
    │   │   │   │   ├── BaseModel.ets
    │   │   │   │   ├── BaseState.ets
    │   │   │   │   └── BaseViewModel.ets
    │   │   │   ├── bean/              # 数据对象
    │   │   │   │   ├── AccessTableTypedef.ets
    │   │   │   │   ├── BundleInfoBean.ets
    │   │   │   │   ├── DefaultMenuConfig.ets
    │   │   │   │   ├── MenuConfig.ets
    │   │   │   │   └── MenuInfo.ets
    │   │   │   ├── components/        # 公共组件
    │   │   │   │   ├── ComponentConfig.ets
    │   │   │   │   ├── TitleBarComponent.ets
    │   │   │   │   └── headComponent.ets
    │   │   │   ├── constants/         # 常量定义
    │   │   │   │   ├── ComConstant.ets
    │   │   │   │   ├── DataShareConstant.ets
    │   │   │   │   ├── HiSysEventConstant.ets
    │   │   │   │   └── RouterConstant.ets
    │   │   │   └── utils/             # 工具类
    │   │   │       ├── AutoMenuManager.ets
    │   │   │       ├── GetSelfBundleInfoUtils.ets
    │   │   │       ├── HiSysEventUtil.ets
    │   │   │       ├── Logger.ets
    │   │   │       ├── RawFileUtil.ets
    │   │   │       ├── RdbManager.ets
    │   │   │       ├── ResourceUtil.ets
    │   │   │       └── StringUtil.ets
    │   │   ├── entryability/          # 入口能力
    │   │   │   └── EntryAbility.ets
    │   │   ├── main/                  # 业务模块
    │   │   │   └── auto_menu/         # 接入菜单业务
    │   │   │       ├── AutoMenuIntent.ets
    │   │   │       ├── AutoMenuModel.ets
    │   │   │       ├── AutoMenuViewModel.ets
    │   │   │       └── AutoMenuViewState.ets
    │   │   ├── model/                 # 数据模型
    │   │   │   ├── bundleInfo/        # 应用包信息
    │   │   │   │   └── BundleInfoModel.ets
    │   │   │   └── locationServicesImpl/  # 位置服务
    │   │   │       ├── ListenerBean.ets
    │   │   │       ├── LocationService.ets
    │   │   │       └── LocationViewModel.ets
    │   │   ├── pages/                 # 页面组件
    │   │   │   ├── Index.ets          # 首页
    │   │   │   ├── locationServices.ets   # 位置服务页
    │   │   │   └── UiExtensionPage.ets   # UIExtension页
    │   │   └── view/                  # 视图组件
    │   │       └── privacy/
    │   │           └── PrivacyProtectionListView.ets
    │   └── resources/                 # 资源文件
    │       ├── base/
    │       │   ├── element/           # 元素资源
    │       │   │   ├── color.json
    │       │   │   ├── float.json
    │       │   │   └── string.json
    │       │   └── profile/           # 配置文件
    │       │       └── main_pages.json
    │       ├── en_US/                 # 英文资源
    │       │   └── element/string.json
    │       └── zh_CN/                 # 中文资源
    │           └── element/string.json
    └── ohosTest/                      # 测试代码（不纳入文档范围）
```

## 模块职责划分

### 核心业务模块

| 目录 | 职责 | 关键类/文件 |
|-----|------|------------|
| `ets/pages/` | 页面渲染与用户交互 | Index、locationServices、UiExtensionPage |
| `ets/model/` | 数据获取与业务逻辑 | LocationService、BundleInfoModel |
| `ets/main/auto_menu/` | 接入菜单业务 | AutoMenuModel、AutoMenuViewModel |
| `ets/view/` | 视图组件 | PrivacyProtectionListView |

### 公共支撑模块

| 目录 | 职责 | 说明 |
|-----|------|------|
| `ets/common/base/` | MVI 架构基类 | Intent、Model、State、ViewModel 抽象 |
| `ets/common/bean/` | 数据结构定义 | MenuInfo、MenuConfig 等接口 |
| `ets/common/components/` | 通用 UI 组件 | TitleBarComponent 等 |
| `ets/common/constants/` | 常量定义 | 业务常量、路由常量 |
| `ets/common/utils/` | 工具类 | Logger、RdbManager、ResourceUtil |

### 配置与资源

| 目录/文件 | 职责 |
|----------|------|
| `module.json5` | 模块能力声明、权限配置 |
| `resources/` | 多语言字符串、颜色、尺寸资源 |
| `AppScope/app.json5` | 应用级配置（包名、版本等） |
| `signature/` | 签名密钥与证书 |

## 代码统计（不含测试）

| 类型 | 数量 |
|-----|------|
| .ets 文件 | 约 40+ |
| .json5 配置文件 | 5 |
| 资源文件（json） | 10+ |
| 总代码行数 | 约 2000+ |

## 不纳入文档的范围

根据项目规范，以下目录**不纳入**Wiki 文档范围：

| 目录 | 原因 |
|-----|------|
| `ohosTest/` | 测试代码 |
| `signature/` | 签名材料（敏感信息） |
| `.git/` | 版本控制目录 |

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [01_Overview.md](./01_Overview.md) → 项目概览
- [03_Architecture.md](./03_Architecture.md) → 架构设计

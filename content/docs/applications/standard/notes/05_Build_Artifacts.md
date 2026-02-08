# 编译产物

## 1. 产物概览

### 1.1 主要产物

| 产物类型 | 模块 | 文件名 | 说明 |
|----------|------|--------|------|
| HAP | default | `default.hap` | 可安装的应用包 |
| HAR | utils | `utils.har` | 工具类库 |
| HAR | component | `component.har` | 功能组件库 |
| HAR | resources | `resources.har` | 资源库 |

### 1.2 产物类型说明

| 类型 | 说明 | 用途 |
|------|------|------|
| **HAP** | Harmony Ability Package | 可直接安装到设备的应用程序包 |
| **HAR** | Harmony Ability Archive | 静态库，可被其他模块引用 |

## 2. HAP 产物详情

### 2.1 默认 HAP

| 属性 | 值 |
|------|-----|
| **文件名** | `default.hap` |
| **模块** | `product/default` |
| **路径** | `build/outputs/hap/default/` |
| **签名** | 已签名 (如配置了签名) |
| **可安装设备** | 手机、平板、折叠屏 |

### 2.2 HAP 结构

```
default.hap
│
├── META-INF/
│   ├── MANIFEST.MF
│   ├── CERT.SF
│   └── CERT.RSA
│
├── libs/                          # 动态库
│   └── libentry.z.so              # 应用入口动态库
│
├── module.json5                   # 模块配置
│
├── resources.index                # 资源索引
│
├── resources/                     # 应用资源
│   ├── base/
│   │   ├── element/               # 基础元素
│   │   ├── media/                # 媒体资源
│   │   └── profile/              # 配置文件
│   │
│   └── zh_CN/                    # 中文资源
│       └── element/
│
└── ets/                           # ArkTS/ETS 字节码
    └── utils/                     # utils 模块
    └── components/                # components 模块
    └── pages/                     # 页面
```

### 2.3 签名配置

| 配置项 | 值/路径 |
|--------|---------|
| **签名文件** | `signature/` 目录 |
| **签名配置** | `build-profile.json5` 中 `signingConfig: "default"` |

**证据位置**: `build-profile.json5:22`

## 3. HAR 产物详情

### 3.1 utils.har

| 属性 | 值 |
|------|-----|
| **来源模块** | `common/utils` |
| **类型** | HAR (Static Library) |
| **导出 API** | RdbStoreUtil, NoteUtil, FolderUtil, LogUtil, DateUtil |

**导出结构**:
```
utils.har
│
├── module.json5
│
├── index.ets                      # 导出入口
│
├── baseUtil/                      # 基础工具类
│   ├── LogUtil.ets
│   ├── DateUtil.ets
│   ├── NoteUtil.ets
│   ├── FolderUtil.ets
│   ├── RdbStoreUtil.ets
│   └── GlobalResourceManager.ets
│
└── model/                         # 数据模型
    ├── databaseModel/
    │   ├── NoteData.ets
    │   ├── FolderData.ets
    │   ├── AttachmentData.ets
    │   ├── EnumData.ets
    │   └── SysDefData.ets
    │
    └── searchModel/
        └── SearchModel.ets
```

### 3.2 component.har

| 属性 | 值 |
|------|-----|
| **来源模块** | `features` |
| **类型** | HAR (Static Library) |
| **导出组件** | NoteListComp, NoteContentComp, FolderListComp, CusDialogComp |

**导出结构**:
```
component.har
│
├── module.json5
│
└── components/
    ├── NoteListComp.ets
    ├── NoteContentComp.ets
    ├── NoteContentCompPortrait.ets
    ├── FolderListComp.ets
    └── CusDialogComp.ets
```

## 4. 运行时加载关系

### 4.1 HAP 加载顺序

```
1. 系统加载 HAP
      │
      ▼
2. 解析 module.json5
      │
      ▼
3. 加载资源 (resources.index)
      │
      ▼
4. 加载模块依赖 (utils.har, component.har)
      │
      ▼
5. 初始化 AbilityStage
      │
      ▼
6. 创建 MainAbility
      │
      ▼
7. 加载页面 (MyNoteHome, NoteHome, etc.)
```

### 4.2 模块加载

| 模块 | 加载时机 | 引用方式 |
|------|----------|----------|
| utils | 应用启动 | HAR 静态链接 |
| component | 页面加载时 | import 语句导入 |

## 5. 安装路径

### 5.1 开发调试安装

| 路径 | 说明 |
|------|------|
| `hdc_std install <hap_path>` | 通过 hdc 命令安装 |
| DevEco Studio 直接安装 | 通过 IDE 部署 |

### 5.2 系统安装路径

| 路径 | 说明 |
|------|------|
| `/data/app/<bundle_name>/<module_name>/` | 应用私有目录 |
| `/system/app/<bundle_name>/` | 系统应用目录 (系统应用) |

**注意**: 具体路径取决于设备配置和系统版本

## 6. 相关跳转

| 目标 | 链接 |
|------|------|
| 构建配置 | [04_GN_Targets.md](04_GN_Targets.md) |
| 安全评估 | [06_Security_Review.md](06_Security_Review.md) |
| 故障排查 | [07_Troubleshooting.md](07_Troubleshooting.md) |

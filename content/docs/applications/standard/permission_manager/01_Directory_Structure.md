# 目录结构与模块职责

> **目的**: 详细说明PermissionManager源码的组织方式和各模块职责  
> **适用范围**: 开发者、代码维护人员  
> **最后更新**: 2026-02-05

---

## 顶层目录结构

```
applications/standard/permission_manager/
├── entry/                              # Entry模块（启动入口）
│   └── src/main/ets/
│       ├── Application/                # AbilityStage配置
│       │   └── AbilityStage.ts         # 生命周期管理
│       ├── MainAbility/                # Entry的MainAbility
│       │   └── MainAbility.ts
│       └── pages/                      # Entry页面
│           └── index.ets
├── permissionmanager/                  # 主模块（核心业务逻辑）
│   └── src/main/ets/
│       ├── Application/                # AbilityStage配置
│       ├── MainAbility/                # 权限管理主Ability
│       ├── ServiceExtAbility/          # 权限请求对话框
│       ├── GlobalExtAbility/           # 全局开关对话框
│       ├── SecurityExtAbility/         # 安全组件对话框
│       ├── PermissionSheet/            # Sheet类型Ability
│       ├── common/                     # 公共代码
│       └── pages/                      # 页面
├── AppScope/                           # 应用级配置
│   ├── app.json                        # 应用基本信息
│   └── resources/                      # 全局资源
├── signature/                          # 签名证书
│   ├── pm.p7b                          # 证书文件
│   └── pm.gni                          # GN签名配置
├── BUILD.gn                            # GN构建入口
└── LICENSE                             # Apache-2.0许可证
```

---

## Entry模块 (`entry/`)

**职责**: 应用启动入口，负责初始化和跳转到主模块

### 文件清单

| 文件 | 路径 | 职责 |
|------|------|------|
| AbilityStage.ts | `entry/src/main/ets/Application/AbilityStage.ts` | 管理Ability生命周期 |
| MainAbility.ts | `entry/src/main/ets/MainAbility/MainAbility.ts` | Entry主Ability |
| index.ets | `entry/src/main/ets/pages/index.ets` | 启动页面 |

**注意**: Entry模块逻辑较简单，主要职责是拉起主模块的MainAbility。

---

## 主模块 (`permissionmanager/`)

### 1. Application目录

**文件**: `permissionmanager/src/main/ets/Application/AbilityStage.ts`

**职责**:
- 管理Ability生命周期
- 全局上下文初始化

### 2. MainAbility目录

**文件**: `permissionmanager/src/main/ets/MainAbility/MainAbility.ts`

**职责**:
- UIAbility类型，单例模式
- 权限管理设置的主入口
- 监听应用包变更事件
- 加载权限管理主页面

**关键方法**:
- `onCreate()`: 初始化，存储caller bundleName
- `onWindowStageCreate()`: 创建窗口，加载主页面
- `getAllApplications()`: 获取所有已安装应用
- `getSperifiedApplication()`: 获取指定应用信息
- `permissionCheck()`: 检查是否拥有GET_INSTALLED_BUNDLE_LIST权限

### 3. ServiceExtAbility目录

**文件**:
- `ServiceExtAbility.ets` - 权限请求对话框Ability
- `GrantDialogModel.ets` - 对话框数据模型
- `GrantDialogViewModel.ets` - 视图模型
- `GrantDialogViewState.ets` - 视图状态
- `GrantDialogIntent.ets` - Intent处理

**职责**:
- 处理运行时权限请求
- 解析调用方信息
- 创建权限请求对话框窗口
- 通过IPC返回授权结果

**关键流程**:
```
onRequest(want)
  → createWindow()
  → getCallerAppInfo(want)  [解析调用方信息]
  → getGroupWithPermission() [权限分组]
  → loadContent('pages/dialogPlus')
  → showWindow()
```

### 4. GlobalExtAbility目录

**文件**:
- `GlobalExtAbility.ets` - 全局开关对话框Ability
- `GlobalDialogModel.ets` - 数据模型
- `GlobalDialogViewState.ets` - 视图状态

**职责**:
- 当权限全局开关关闭时显示提示对话框
- 请求用户启用全局开关

**安全要求**:
- 需要权限: `ohos.permission.GET_SENSITIVE_PERMISSIONS`

### 5. SecurityExtAbility目录

**文件**:
- `SecurityExtAbility.ets` - 安全组件对话框Ability

**职责**:
- 为安全组件提供授权对话框
- 绑定到目标窗口 (bindDialogTarget)
- 管理对话框生命周期

**关键特性**:
- 使用 `dialogSet` 防止重复对话框
- 支持窗口绑定和取消绑定回调

### 6. PermissionSheet目录

**文件**:
- `PermissionStateSheetAbility.ets` - 权限状态Sheet
- `PermissionStateSheetDialog.ets` - Sheet对话框UI
- `GlobalSwitchSheetAbility.ets` - 全局开关Sheet
- `GlobalSwitchSheetDialog.ets` - 全局开关Sheet UI

**职责**:
- 提供Sheet类型的权限管理界面
- 支持从应用或其他系统界面弹出

**技术特点**:
- 继承自 `UIExtensionAbility`
- 创建子窗口或浮动窗口
- 支持模态显示

### 7. common目录

#### 7.1 base/ - 基类

| 文件 | 职责 |
|------|------|
| `BasePermissionStrategy.ets` | 权限策略基类，定义策略接口 |
| `BaseModel.ets` | 数据模型基类 |
| `BaseViewModel.ets` | 视图模型基类 |
| `BaseIntent.ets` | Intent处理基类 |
| `BaseState.ets` | 状态管理基类 |

**BasePermissionStrategy核心方法**:
- `getPermissionGroupConfig()`: 获取权限组配置
- `getGroupTitle()`: 获取权限组标题
- `getReasonByPermission()`: 获取授权理由
- `grantHandle()`: 授权处理逻辑
- `denyHandle()`: 拒绝处理逻辑
- `preProcessingPermission()`: 预处理权限

#### 7.2 permissionGroupManager/ - 权限策略

| 文件 | 权限组 |
|------|--------|
| `PermissionGroupManager.ets` | 策略管理器（单例） |
| `LocationStrategy.ets` | 位置信息 |
| `CameraStrategy.ets` | 相机 |
| `MicrophoneStrategy.ets` | 麦克风 |
| `ContactsStrategy.ets` | 通讯录 |
| `CalendarStrategy.ets` | 日历 |
| `SportStrategy.ets` | 运动健身 |
| `HealthStrategy.ets` | 身体传感器 |
| `ImageAndVideosStrategy.ets` | 图片和视频 |
| `AudiosStrategy.ets` | 音频 |
| `DocumentsStrategy.ets` | 文件 |
| `AdsStrategy.ets` | 应用跟踪 |
| `GetInstalledBundleListStrategy.ets` | 读取已安装应用列表 |
| `DistributedDatasyncStrategy.ets` | 分布式数据同步 |
| `BluetoothStrategy.ets` | 蓝牙 |
| `PasteboardStrategy.ets` | 剪贴板 |
| `DownloadDirectoryStrategy.ets` | 下载文件夹 |
| `DesktopDirectoryStrategy.ets` | 桌面文件夹 |
| `DocumentsDirectoryStrategy.ets` | 文档文件夹 |
| `NearlinkStrategy.ets` | 星闪 |
| `ScreenCaptureStrategy.ets` | 屏幕录制 |

**PermissionGroupManager职责**:
- 管理所有权限策略实例
- 权限组配置查询
- 权限预处理协调
- 授权/拒绝操作分发

#### 7.3 model/ - 数据模型

| 文件 | 职责 |
|------|------|
| `definition.ets` | 枚举定义（Permission, PermissionGroup, ButtonStatus等） |
| `typedef.ets` | 类型定义和接口（AppInfo, CallerAppInfo, PermissionGroupConfig等） |
| `permissionGroup.ets` | 权限组相关数据结构 |

**definition.ets中定义的枚举**:
- `Permission`: 48个具体权限
- `PermissionGroup`: 21个权限组
- `ButtonStatus`: 6种按钮状态
- `ClickOption`: 3种点击选项
- `PermissionOption`: 3种权限操作选项
- `PermissionType`: 3种权限结果类型

#### 7.4 utils/ - 工具函数

| 文件 | 职责 |
|------|------|
| `utils.ets` | 通用工具函数（Log, 字符串处理等） |
| `permissionUtils.ets` | 权限相关工具 |
| `bundleInfoUtils.ets` | 应用包信息工具 |
| `commonUtils.ets` | 通用工具 |
| `deviceUtil.ets` | 设备信息工具 |
| `constant.ets` | 常量定义 |
| `globalContext.ets` | 全局上下文管理 |

#### 7.5 components/ - 可复用组件

| 文件 | 组件 |
|------|------|
| `alphabeticalIndex.ets` | 字母索引组件 |
| `backBar.ets` | 返回导航栏 |
| `location.ets` | 位置信息组件 |
| `search.ets` | 搜索组件 |

#### 7.6 observer/ - 事件观察者

| 文件 | 职责 |
|------|------|
| `EventObserver.ets` | 应用包变更事件监听 |

### 8. pages/ - 页面

| 文件 | 页面 |
|------|------|
| `authority-management.ets` | 权限管理主页面 |
| `authority-secondary.ets` | 权限二级页面 |
| `authority-tertiary.ets` | 权限三级页面（详情） |
| `authority-tertiary-groups.ets` | 权限组三级页面 |
| `application-secondary.ets` | 应用详情页面 |
| `application-tertiary.ets` | 应用权限详情 |
| `dialogPlus.ets` | 权限请求对话框 |
| `globalSwitch.ets` | 全局开关对话框 |
| `securityDialog.ets` | 安全组件对话框 |
| `permission-access-record.ets` | 权限访问记录 |
| `transition.ets` | 过渡动画页面 |

**页面导航关系**:

```
authority-management.ets
    ├── authority-secondary.ets
    │   └── authority-tertiary.ets
    │       └── authority-tertiary-groups.ets
    └── application-secondary.ets
        └── application-tertiary.ets
```

---

## AppScope目录

### 文件

| 文件 | 职责 |
|------|------|
| `app.json` | 应用基本信息（bundleName, vendor, version等） |
| `resources/` | 全局资源文件 |

---

## 模块依赖关系

```
pages/
    ├── 依赖 common/components/
    ├── 依赖 common/model/
    ├── 依赖 common/utils/
    └── 依赖 common/permissionGroupManager/

ServiceExtAbility/
    ├── 依赖 common/base/
    ├── 依赖 common/model/
    ├── 依赖 common/utils/
    └── 依赖 common/permissionGroupManager/

MainAbility/
    ├── 依赖 common/utils/
    └── 依赖 common/model/

permissionGroupManager/
    ├── 依赖 base/BasePermissionStrategy
    └── 依赖 model/
```

---

## 测试文件（忽略）

根据要求，以下目录中的测试文件不纳入文档范围：

```
entry/src/ohosTest/
permissionmanager/src/ohosTest/
```

包含的测试类型:
- `List.test.ets`
- `utils.test.ets`

---

## 源码文件统计

| 类别 | 数量 | 路径模式 |
|------|------|----------|
| Ability | 6 | `*/MainAbility/*`, `*ExtAbility/*` |
| 页面 | 15 | `*/pages/*.ets` |
| 策略类 | 21 | `*/permissionGroupManager/*Strategy.ets` |
| 数据模型 | 3 | `*/model/*.ets` |
| 工具类 | 7 | `*/utils/*.ets` |
| 组件 | 4 | `*/components/*.ets` |
| 基类 | 5 | `*/base/*.ets` |
| **总计** | **~64** | `permissionmanager/src/main/ets/**/*.ets` |

---

*上一篇: [项目概览](00_Overview.md) | 返回 [README](README.md) | 下一篇: [架构说明](02_Architecture.md)*

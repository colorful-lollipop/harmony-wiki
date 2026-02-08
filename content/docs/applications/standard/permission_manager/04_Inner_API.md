# 内部API说明

> **目的**: 详细说明PermissionManager内部模块的接口、依赖关系和稳定性  
> **适用范围**: 项目开发者、维护人员  
> **最后更新**: 2026-02-05

---

## 模块概览

PermissionManager内部模块遵循**分层架构**，从上到下依次为：

```
┌─────────────────────────────────────────────────────────────┐
│  UI层 (pages/)                                              │
│  - 页面组件，直接使用ViewModel和Model                        │
├─────────────────────────────────────────────────────────────┤
│  Ability层 (*ExtAbility/)                                   │
│  - Ability生命周期管理                                      │
│  - 协调Model和UI层                                          │
├─────────────────────────────────────────────────────────────┤
│  ViewModel层 (*ViewModel.ets)                               │
│  - 业务逻辑处理                                             │
│  - 数据转换和验证                                           │
├─────────────────────────────────────────────────────────────┤
│  Model层 (*Model.ets)                                       │
│  - 数据模型定义                                             │
│  - 系统API调用封装                                          │
├─────────────────────────────────────────────────────────────┤
│  策略层 (permissionGroupManager/)                           │
│  - 权限组策略实现                                           │
├─────────────────────────────────────────────────────────────┤
│  基础层 (common/)                                           │
│  - 工具函数、常量、类型定义                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. 基类接口 (common/base/)

### 1.1 BasePermissionStrategy

**文件**: `permissionmanager/src/main/ets/common/base/BasePermissionStrategy.ets`

**稳定性**: ⭐⭐⭐⭐⭐ (稳定接口，新增策略必须实现)

**接口定义**:

```typescript
export abstract class BasePermissionStrategy {
    // 必须实现
    abstract getPermissionGroupConfig(): PermissionGroupConfig;
    abstract getGroupTitle(appName: string, locationFlag: number): ResourceStr;
    
    // 默认实现，可覆盖
    getReadAndWrite(permissions: Set<Permission>): ResourceStr;
    getReasonByPermission(groupToPermissions, permissions, context, callerAppInfo, pasteBoardName): ResourceStr;
    getButtonList(): ButtonStatus[];
    isSupport(permission: Permission): boolean;
    grantHandle(permission, callerAppInfo, locationFlag, buttonStatus): PermissionOption;
    denyHandle(permission, callerAppInfo, locationFlag, buttonStatus): PermissionOption;
    isPermissionPendingGrant(permission, callerAppInfo): boolean;
    preProcessingPermission(permission, tokenId): Promise<PermissionWithOption[]>;
}
```

**使用场景**: 所有权限组策略必须继承此基类，实现 `getPermissionGroupConfig()` 和 `getGroupTitle()` 方法。

**示例**:
```typescript
export class CameraStrategy extends BasePermissionStrategy {
    getPermissionGroupConfig(): PermissionGroupConfig {
        return new PermissionGroupConfig({
            groupName: PermissionGroup.CAMERA,
            permissions: [Permission.CAMERA],
            icon: $r('app.media.camera'),
            title: $r('app.string.camera_title'),
            reason: $r('app.string.camera_reason'),
            buttonList: [ButtonStatus.DENY, ButtonStatus.ALLOW]
        });
    }
    
    getGroupTitle(appName: string, locationFlag: number): ResourceStr {
        return $r('app.string.camera_title');
    }
}
```

---

### 1.2 BaseModel

**文件**: `permissionmanager/src/main/ets/common/base/BaseModel.ets`

**稳定性**: ⭐⭐⭐ (中等稳定，可能扩展)

**说明**: 目前为空的基类，预留用于后续统一的数据模型接口。

---

### 1.3 BaseViewModel

**文件**: `permissionmanager/src/main/ets/common/base/BaseViewModel.ets`

**稳定性**: ⭐⭐⭐ (中等稳定)

**说明**: 视图模型基类，预留用于统一的视图逻辑处理。

---

## 2. 策略管理器接口

### 2.1 PermissionGroupManager

**文件**: `permissionmanager/src/main/ets/common/permissionGroupManager/PermissionGroupManager.ets`

**稳定性**: ⭐⭐⭐⭐⭐ (核心接口，高度稳定)

**类定义**:

```typescript
export class PermissionGroupManager {
    // 单例获取
    static getInstance(): PermissionGroupManager;
    
    // 获取所有权限组配置
    getAllGroupConfigs(): PermissionGroupConfig[];
    
    // 根据权限查找所属权限组
    getGroupNameByPermission(permission: Permission, allConfigs: PermissionGroupConfig[]): PermissionGroup | undefined;
    
    // 预处理权限
    preProcessPermission(permission: Permission, groupName: PermissionGroup, tokenId: number): Promise<PermissionWithOption[]>;
    
    // 获取权限组配置列表（用于UI展示）
    getGroupConfigs(callerAppInfo: CallerAppInfo, context: common.ServiceExtensionContext, 
                    appName: string, locationFlag: number, pasteBoardName: string): PermissionGroupConfig[];
    
    // 根据用户点击获取权限操作选项
    getPermissionOption(permission: Permission, groupName: PermissionGroup, 
                        callerAppInfo: CallerAppInfo, locationFlag: number, 
                        buttonStatus: ButtonStatus): PermissionOption;
    
    // 将按钮状态转换为点击选项
    getClickSituation(buttonStatus: ButtonStatus): ClickOption;
}
```

**依赖方向**:
```
PermissionGroupManager
    ├── BasePermissionStrategy (基类)
    ├── 20+ 具体策略类
    ├── definition.ets (Permission, PermissionGroup枚举)
    └── typedef.ets (PermissionGroupConfig, CallerAppInfo等)
```

**线程安全**: 单例模式，内部状态只读（策略Map在构造时初始化），**线程安全**。

---

## 3. 数据模型接口

### 3.1 CallerAppInfo

**文件**: `permissionmanager/src/main/ets/common/model/typedef.ets:484-493`

**稳定性**: ⭐⭐⭐⭐⭐ (核心数据结构)

**接口定义**:

```typescript
export interface CallerAppInfo {
    readonly bundleName: string;                    // 调用方包名
    readonly tokenId: number;                       // 调用方Token ID
    readonly reqPerms: Permission[];                // 请求的权限列表
    readonly reqPermsState: number[];               // 权限初始状态
    readonly reqPermsDetails: bundleManager.ReqPermissionDetail[];  // 权限详情
    readonly proxy: rpc.RemoteObject;               // IPC代理对象
    grantResult: number[];                          // 授权结果（可修改）
    groupWithPermission: Map<PermissionGroup, Set<Permission>>;  // 权限分组
}
```

**创建方式**: 通过 `GrantDialogModel.getCallerAppInfo(want)` 解析Want参数创建。

---

### 3.2 PermissionGroupConfig

**文件**: `permissionmanager/src/main/ets/common/model/typedef.ets:495-524`

**稳定性**: ⭐⭐⭐⭐⭐

**接口定义**:

```typescript
export class PermissionGroupConfig {
    public readonly groupName: PermissionGroup;      // 权限组名
    public readonly permissions: Permission[];       // 权限列表
    public icon: Resource;                           // 图标资源
    public readonly title: ResourceStr;              // 标题
    public readonly readAndWrite?: ResourceStr;      // 读写说明（可选）
    public readonly reason: ResourceStr;             // 授权理由
    public readonly buttonList: ButtonStatus[];      // 按钮列表
    
    constructor(param: GroupConfig);
}

export interface GroupConfig {
    readonly groupName: PermissionGroup;
    readonly permissions: Permission[];
    icon: Resource;
    readonly title: ResourceStr;
    readonly readAndWrite?: ResourceStr;
    readonly reason: ResourceStr;
    readonly buttonList: ButtonStatus[];
}
```

---

### 3.3 PermissionWithOption

**文件**: `permissionmanager/src/main/ets/common/model/typedef.ets:526-529`

**稳定性**: ⭐⭐⭐⭐⭐

```typescript
export interface PermissionWithOption {
    permission: Permission;              // 权限
    permissionOption: PermissionOption;  // 操作选项（GRANT/REVOKE/SKIP）
}
```

---

## 4. GrantDialogModel接口

**文件**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets`

**稳定性**: ⭐⭐⭐⭐ (相对稳定，可能新增功能)

**核心方法**:

```typescript
export class GrantDialogModel extends BaseModel {
    // 从Want解析调用方信息
    getCallerAppInfo(want: Want): Promise<CallerAppInfo>;
    
    // 创建权限请求对话框窗口
    createWindow(context: common.ServiceExtensionContext, name: string, 
                 rect: display.Rect, want: Want): Promise<void>;
    
    // 获取剪贴板信息来源应用名
    getPasteBoardInfo(): string;
    
    // 获取应用显示名称
    getAppName(bundleName: string): string;
    
    // 初始化位置权限组标志
    initLocationFlag(callerAppInfo: CallerAppInfo): number;
    
    // 获取授权权限组配置
    getGrantGroups(callerAppInfo: CallerAppInfo, context: common.ServiceExtensionContext,
                   appName: string, locationFlag: number, pasteBoardName: string): PermissionGroupConfig[];
    
    // 处理用户点击事件
    clickHandle(groupConfig: PermissionGroupConfig, callerAppInfo: CallerAppInfo, 
                locationFlag: number, buttonStatus: ButtonStatus): Promise<void>;
    
    // 写入授权结果
    writeToGrantResult(permission: Permission, callerAppInfo: CallerAppInfo, 
                       state: abilityAccessCtrl.GrantStatus): void;
    
    // 关闭对话框并返回结果
    terminateWithResult(context: common.ServiceExtensionContext, win: window.Window, 
                        callerAppInfo: CallerAppInfo): Promise<void>;
}
```

**依赖关系**:
```
GrantDialogModel
    ├── BaseModel (继承)
    ├── PermissionGroupManager (权限策略)
    ├── definition.ets (枚举)
    ├── typedef.ets (类型)
    ├── utils.ets (工具函数)
    ├── constant.ets (常量)
    ├── @ohos.rpc (IPC)
    ├── @ohos.window (窗口)
    ├── @ohos.pasteboard (剪贴板)
    ├── @ohos.bundle.bundleManager (应用包)
    └── @ohos.abilityAccessCtrl (权限控制)
```

---

## 5. 工具函数接口

### 5.1 utils.ets

**文件**: `permissionmanager/src/main/ets/common/utils/utils.ets`

**稳定性**: ⭐⭐⭐⭐ (稳定，通用工具)

**导出函数**:

```typescript
// 日志输出
export function Log: {
    info(message: string): void;
    error(message: string): void;
    debug(message: string): void;
}

// 字符串截断
export function titleTrim(title: string, maxLength?: number): string;

// 获取支持的权限列表
export function supportPermission(): Permission[];

// 对话框异常处理
export function PermissionDialogException(errCode: number, session?: UIExtensionContentSession): void;

// 销毁处理
export function destruction(want: Want, permissionType: PermissionType): void;

// 设置避让区域
export function setAvoidArea(extensionWinProxy: uiExtensionHost.UIExtensionHostWindowProxy): void;
```

---

### 5.2 GlobalContext

**文件**: `permissionmanager/src/main/ets/common/utils/globalContext.ets`

**稳定性**: ⭐⭐⭐⭐⭐ (核心工具，全局状态管理)

**接口定义**:

```typescript
export class GlobalContext {
    // 单例获取
    static getContext(): GlobalContext;
    
    // 存储全局状态
    store(key: string, value: Object): void;
    
    // 加载全局状态
    load(key: string): Object | undefined;
    
    // 设置并获取窗口数量
    setAndGetWindowNum(num: number): number;
    
    // 增加窗口数量并返回
    increaseAndGetWindowNum(): number;
    
    // 减少窗口数量并返回
    decreaseAndGetWindowNum(): number;
}
```

**使用场景**: 在Ability和页面间共享状态，如窗口计数、应用信息等。

---

## 6. 模块依赖图

### 6.1 完整依赖图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              UI层 (pages)                               │
│  authority-management.ets, dialogPlus.ets, ...                         │
│                                │                                        │
│          ┌─────────────────────┼─────────────────────┐                  │
│          ▼                     ▼                     ▼                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐           │
│  │   common/     │    │   common/     │    │   common/     │           │
│  │   model/      │    │    utils/     │    │ permissionGroupManager │  │
│  └───────────────┘    └───────────────┘    └───────────────┘           │
│          │                     │                     │                  │
└──────────┼─────────────────────┼─────────────────────┼──────────────────┘
           │                     │                     │
           ▼                     ▼                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           Ability层                                      │
│  MainAbility.ts, ServiceExtAbility.ets, GlobalExtAbility.ets, ...       │
│                                │                                        │
│          ┌─────────────────────┼─────────────────────┐                  │
│          ▼                     ▼                     ▼                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐           │
│  │   *Model.ets  │    │   *ViewModel  │    │   common/     │           │
│  │               │    │               │    │    utils/     │           │
│  └───────────────┘    └───────────────┘    └───────────────┘           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          系统服务层                                      │
│  @ohos.abilityAccessCtrl, @ohos.bundle.bundleManager, @ohos.window, ...│
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.2 禁止的依赖方向

为避免循环依赖，以下依赖方向**禁止**：

| 禁止的依赖 | 原因 |
|-----------|------|
| common/ → pages/ | 基础层不应依赖UI层 |
| common/ → *ExtAbility/ | 基础层不应依赖Ability层 |
| *Model.ets → pages/ | Model不应依赖UI |
| 策略类 → pages/ | 策略类不应依赖UI |

---

## 7. 可替换点

### 7.1 可替换的模块

| 模块 | 替换方式 | 影响范围 |
|------|----------|----------|
| 具体策略类 | 继承BasePermissionStrategy实现新策略 | 该权限组的处理逻辑 |
| GrantDialogModel | 继承BaseModel重新实现 | 权限请求对话框行为 |
| 工具函数 | 修改utils.ets中的实现 | 全局工具行为 |

### 7.2 不可替换的模块

| 模块 | 原因 |
|------|------|
| PermissionGroupManager | 核心管理器，单例模式 |
| Ability生命周期方法 | 系统回调，固定接口 |
| 数据模型接口 | 跨模块依赖，修改会导致连锁反应 |

---

## 8. API稳定性分级

| 分级 | 含义 | 示例 |
|------|------|------|
| ⭐⭐⭐⭐⭐ | 高度稳定，不会修改 | BasePermissionStrategy, PermissionGroupManager, 数据模型接口 |
| ⭐⭐⭐⭐ | 稳定，可能新增功能 | GrantDialogModel, 工具函数 |
| ⭐⭐⭐ | 中等稳定，可能调整 | BaseViewModel, 具体策略实现 |
| ⭐⭐ | 不稳定，可能重构 | UI层组件内部方法 |
| ⭐ | 实验性，随时可能修改 | 新开发功能 |

---

## 9. 接口版本历史

| 版本 | 修改内容 | 时间 |
|------|----------|------|
| 1.0 | 初始版本，包含基本权限请求功能 | 2022 |
| 1.1 | 新增权限组策略模式 | 2023 |
| 1.2 | 新增SecurityExtAbility支持安全组件 | 2023 |
| 1.3 | 新增Sheet类型Ability | 2024 |
| 1.4 | 新增多个权限组策略（目录访问等） | 2025 |

---

*上一篇: [对外API参考](03_API_Reference.md) | 返回 [README](README.md) | 下一篇: [构建系统](05_Build_System.md)*

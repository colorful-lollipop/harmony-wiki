# 内部 API

## 概述

本文档描述安全隐私中心模块内部的接口定义，包括模块间的数据流向、接口契约以及稳定性标注。所有接口均可追溯到代码证据。

## 模块接口总览

| 模块 | 导出接口 | 依赖方 | 稳定性 |
|-----|---------|-------|--------|
| AutoMenuModel | getMenuInfoListFromRdb | AutoMenuViewModel | 稳定 |
| AutoMenuModel | getMenuInfoListFromBms | AutoMenuViewModel | 稳定 |
| AutoMenuModel | handleMenuClick | AutoMenuViewModel | 稳定 |
| LocationService | getServiceState | LocationViewModel | 稳定 |
| LocationService | enableLocation | LocationViewModel | 稳定 |
| LocationService | disableLocation | LocationViewModel | 稳定 |
| BundleInfoModel | getAllBundleInfoByFunctionAccess | Index | 稳定 |

## 核心接口详细说明

### 1. AutoMenuModel 接口

**文件位置**：`entry/src/main/ets/main/auto_menu/AutoMenuModel.ets`

#### 1.1 getMenuInfoListFromRdb

```typescript
async getMenuInfoListFromRdb(context: Context): Promise<MenuInfo[]>
```

**功能**：从本地 RDB 缓存获取菜单信息列表

**参数**：
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| context | Context | 是 | UIAbility 上下文 |

**返回值**：
| 类型 | 说明 |
|-----|------|
| MenuInfo[] | 菜单信息列表，可能为空数组 |

**实现逻辑**：
1. 创建 RDB 查询谓词（`DataShareConstants.ANTO_MENU_TABLE_V2`）
2. 执行查询并遍历结果集
3. 构建 MenuInfo 对象数组

**异常处理**：返回空数组而非抛出异常

**调用来源**：`AutoMenuViewModel.processIntentWithModel`（`AutoMenuInitIntent`）

**证据来源**：`AutoMenuModel.ets:40-84`

#### 1.2 getMenuInfoListFromBms

```typescript
async getMenuInfoListFromBms(context: Context): Promise<MenuInfo[]>
```

**功能**：从 BMS（Bundle Manager Service）获取最新菜单配置

**参数**：
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| context | Context | 是 | UIAbility 上下文 |

**返回值**：
| 类型 | 说明 |
|-----|------|
| MenuInfo[] | 从 BMS 获取并转换的菜单列表 |

**实现逻辑**：
1. 获取当前用户 ID
2. 从 BMS 获取 MenuConfig 列表
3. 转换为 MenuInfo 列表
4. 按优先级排序
5. 刷新 RDB 缓存
6. 返回结果

**调用来源**：`AutoMenuViewModel.processIntentWithModel`（`AutoMenuRefreshIntent`）

**证据来源**：`AutoMenuModel.ets:86-105`

#### 1.3 handleMenuClick

```typescript
handleMenuClick(menuInfo: MenuInfo): void
```

**功能**：处理菜单项点击事件，根据 dstAbilityMode 执行不同跳转

**参数**：
| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| menuInfo | MenuInfo | 是 | 被点击的菜单信息 |

**dstAbilityMode 枚举**：

| 值 | 常量名 | 行为 |
|---|--------|------|
| 0 | DST_UIABILITY_MODE | 通过 startAbility 启动目标 UIAbility |
| 1 | DST_ABILITY_MODE | 跳转到 UiExtensionPage 承载第三方 UI |
| 2 | DST_PAGE_MODE | 通过 router 跳转到本地页面 |

**实现逻辑**：
- 根据 `menuInfo.dstAbilityMode` 判断跳转类型
- 调用对应的跳转 API
- 错误通过 Logger 记录

**调用来源**：`AutoMenuViewModel.processIntentWithModel`（`AutoMenuClickIntent`）

**证据来源**：`AutoMenuModel.ets:107-145`

### 2. LocationService 接口

**文件位置**：`entry/src/main/ets/model/locationServicesImpl/LocationService.ets`

#### 2.1 getServiceState

```typescript
async getServiceState(): Promise<void>
```

**功能**：获取当前位置服务开关状态

**参数**：无

**返回值**：无（通过 Listener 回调更新状态）

**实现逻辑**：
1. 调用 `geolocation.isLocationEnabled()` 获取状态
2. 通过注册的 Listener 回调更新 UI

**事件回调**：
```typescript
registerListener(listener: ListenerBean): void
mListener?.updateServiceState(state)
```

**调用来源**：`LocationViewModel.initViewModel`

**证据来源**：`LocationService.ets:45-55`

#### 2.2 enableLocation

```typescript
enableLocation(): void
```

**功能**：开启位置服务

**参数**：无

**返回值**：无

**实现逻辑**：
```typescript
geolocation.enableLocation()
  .then((res) => Logger.info(TAG, `enable location, result: ${JSON.stringify(res)}`))
  .catch((error) => Logger.info(TAG, `enable location, error: ${error}`));
```

**调用来源**：`LocationVM.enableLocation`

**证据来源**：`LocationService.ets:57-67`

#### 2.3 disableLocation

```typescript
disableLocation(): void
```

**功能**：关闭位置服务

**参数**：无

**返回值**：无

**实现逻辑**：
```typescript
geolocation.disableLocation();
```

**调用来源**：`LocationVM.disableLocation`

**证据来源**：`LocationService.ets:69-76`

#### 2.4 状态监听

```typescript
startService(): void
geolocation.on('locationEnabledChange', (isChanged: boolean) => { ... });
```

**功能**：监听系统位置开关变化事件

**触发条件**：用户通过系统设置或其他方式改变位置开关

**证据来源**：`LocationService.ets:28-38`

### 3. BundleInfoModel 接口

**文件位置**：`entry/src/main/ets/model/bundleInfo/BundleInfoModel.ets`

#### 3.1 getAllBundleInfoByFunctionAccess

```typescript
getAllBundleInfoByFunctionAccess(): Promise<bundleManager.BundleInfo[]>
```

**功能**：获取具有功能访问权限的应用包信息列表

**参数**：无

**返回值**：
| 类型 | 说明 |
|-----|------|
| Promise\<bundleManager.BundleInfo[]\> | 应用包信息数组 |

**调用来源**：`Index.getBundleInfo`

**证据来源**：`Index.ets:64`

## 数据结构定义

### MenuInfo 接口

**文件位置**：`entry/src/main/ets/common/bean/MenuInfo.ets`

```typescript
export default interface MenuInfo {
  businessId: string;           // 业务 ID
  intents: string;              // Intent 字符串
  userId: number;               // 用户 ID
  iconBackgroundColorResource: string;  // 图标背景色
  priority: number;             // 优先级（越大越靠前）
  isSupport: number;            // 是否支持（0：不支持，1：支持）
  isClickable: number;          // 是否可点击（0：不可，1：可）
  displayedMode: string;        // 展示模式（list/card）
  iconResource: string;         // 图标资源
  mainTitleResource: string;    // 主标题资源
  subTitleResource: string;     // 副标题资源
  showControlAbilityUri: string; // 控制 Ability URI
  dstAbilityMode: number;       // 目标 Ability 模式
  dstAbilityName: string;       // 目标 Ability 名称
  dstBundleName: string;        // 目标包名
  bundleName: string;           // 当前包名
  titleString: string;          // 解析后的主标题
  subTitleString: string;       // 解析后的副标题
}
```

### ListenerBean 接口

**文件位置**：`entry/src/main/ets/model/locationServicesImpl/ListenerBean.ets`

```typescript
class ListenerBean {
  updateServiceState(state: boolean): void;
}
```

**用途**：位置服务状态变更的回调接口

## 稳定性标注

### 稳定接口（Stable）

以下接口为稳定接口，可安全依赖：

| 接口 | 标注依据 |
|-----|---------|
| AutoMenuModel 所有方法 | `common/` 目录，公共模块 |
| LocationService 所有方法 | `model/` 目录，业务核心 |
| BundleInfoModel 所有方法 | `model/` 目录，业务核心 |

### 不可变接口原则

- 不应直接修改公共模块的接口参数
- 新增功能应通过扩展实现，而非修改现有接口
- 接口变更应遵循版本兼容原则

## 依赖方向图

```
Index (pages)
    │
    ├──▶ AutoMenuViewModel (main/auto_menu)
    │         │
    │         └──▶ AutoMenuModel
    │                   │
    │                   ├──▶ AutoMenuManager (utils)
    │                   ├──▶ RdbManager (utils)
    │                   └──▶ ResourceUtil (utils)
    │
    ├──▶ LocationViewModel (model/locationServicesImpl)
    │         │
    │         └──▶ LocationService
    │                   │
    │                   └──▶ @ohos.geoLocationManager (系统 API)
    │
    └──▶ BundleInfoModel (model/bundleInfo)
              │
              └──▶ @ohos.bundle.bundleManager (系统 API)
```

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [03_Architecture.md](./03_Architecture.md) → 架构设计
- [04_NAPI.md](./04_NAPI.md) → N-API 接口
- [06_Build_Config.md](./06_Build_Config.md) → 构建配置

# 03_Components - 组件清单与 API

## 概述

本文档详细描述 advanced_ui_component 中的所有组件及其 API。

## 组件总览

| 组件 | 类型 | N-API 模块 | ABC 导出 | JS API |
|------|------|------------|----------|--------|
| AtomServiceNavigation | 原子化服务 | ✅ | ✅ | ❌ |
| AtomServiceSearch | 原子化服务 | ✅ | ✅ | ❌ |
| AtomServiceTabs | 原子化服务 | ✅ | ✅ | ❌ |
| AtomServiceWeb | 原子化服务 | ✅ | ✅ | ✅ |
| CustomAppBar | 原子化服务 | ✅ | ✅ | ❌ |
| CustomAppBarMenuBar | 原子化服务 | ✅ | ✅ | ✅ |
| FullScreenLaunchComponent | 启动组件 | ❌ | ✅ | ❌ |
| HalfScreenLaunchComponent | 启动组件 | ❌ | ✅ | ❌ |
| InnerFullScreenLaunchComponent | 启动组件 | ❌ | ✅ | ❌ |
| InterstitialDialogAction | 弹窗组件 | ✅ | ✅ | ❌ |
| NavPushPathHelper | 导航助手 | ✅ | ✅ | ✅ |

---

## 原子化服务组件

### AtomServiceNavigation

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.AtomicServiceNavigation` |
| **路径** | `atomicservicenavigation/` |
| **源码** | `atomicservicenavigation/source/atomicservicenavigation.ets` |
| **N-API 实现** | `atomicservicenavigation/interfaces/atomicservicenavigation.cpp` |

**功能**: 提供原子化服务场景的导航组件。

**使用示例**:
```typescript
import { AtomicServiceNavigation } from '@ohos/atomicService';

@Entry
@Component
struct Index {
  build() {
    AtomicServiceNavigation() {
      // 导航内容
    }
  }
}
```

**证据来源**: `atomicservicenavigation/interfaces/atomicservicenavigation.cpp:36-51`

---

### AtomServiceSearch

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.AtomicServiceSearch` |
| **路径** | `atomicservicesearch/` |
| **源码** | `atomicservicesearch/source/atomicservicesearch.ets` |
| **N-API 实现** | `atomicservicesearch/interfaces/atomicservicesearch.cpp` |

**功能**: 提供原子化服务场景的搜索组件。

---

### AtomServiceTabs

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.AtomicServiceTabs` |
| **路径** | `atomicservicetabs/` |
| **源码** | `atomicservicetabs/source/atomicservicetabs.ets` |
| **N-API 实现** | `atomicservicetabs/interfaces/atomicservicetabs.cpp` |

**功能**: 提供原子化服务场景的标签页组件，支持多内容切换。

---

### AtomServiceWeb

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.AtomicServiceWeb` |
| **路径** | `atomicserviceweb/` |
| **源码** | `atomicserviceweb/source/atomicserviceweb.ets` |
| **N-API 实现** | `atomicserviceweb/interfaces/atomicserviceweb.cpp` |

#### JS API 清单

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `checkUrl(bundleName, domainType, url)` | string, string, string | number | URL 策略校验 |

#### checkUrl 详细说明

**C++ 实现**: `atomicserviceweb/interfaces/atomicserviceweb.cpp:27-71`

**参数**:
| 参数 | 类型 | 说明 | 校验 |
|------|------|------|------|
| bundleName | string | 包名 | 非空字符串, 最大 1024 字符 |
| domainType | string | 域名类型 | 非空字符串 |
| url | string | 待校验 URL | 非空字符串, 最大 1024 字符 |

**返回值**:
| 值 | 说明 |
|---|------|
| 0 | URL 校验通过 |
| 非 0 | URL 校验失败, 具体含义由系统策略定义 |

**错误处理**:
- 参数类型错误抛出 `TypeError`
- 参数不足抛出 `Error`

**调用链**:
```
JS: atomicservice.AtomicServiceWeb.checkUrl()
    │
    ▼
N-API: CheckUrl(env, info)
    │
    ▼
ApiPolicyAdapter::CheckUrl()
    │
    ▼
dlopen("/system/lib64/platformsdk/libapipolicy_client.z.so")
    │
    ▼
系统 API: CheckUrl()
```

**证据来源**:
- 注册点: `atomicserviceweb/interfaces/atomicserviceweb.cpp:97-105`
- API 实现: `atomicserviceweb/interfaces/atomicserviceweb.cpp:27-71`
- 策略适配: `atomicserviceweb/interfaces/api_policy_adapter.cpp:21-26`

---

### CustomAppBar

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.CustomAppBar` |
| **路径** | `customappbar/` |
| **N-API 实现** | `customappbar/interfaces/` |

**功能**: 提供自定义应用栏组件。

---

### CustomAppBarMenuBar

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.AtomServiceMenuBar` |
| **路径** | `customappbar/atomicservicemenubar/` |
| **头文件** | `customappbar/atomicservicemenubar/include/menubar_api_implement.h` |

#### JS API 清单

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `setMenubarVisible(visible)` | boolean | void | 设置菜单栏可见性 |

**C++ 实现**: `MenubarAPIImplement::setMenubarVisible()`

**依赖**:
- `ace_engine:ace_ndk`
- `ace_engine:ace_uicontent`

**证据来源**: `customappbar/atomicservicemenubar/include/menubar_api_implement.h:27`

---

## 启动组件

### FullScreenLaunchComponent

| 属性 | 值 |
|------|-----|
| **路径** | `fullscreenlaunchcomponent/` |
| **源码** | `fullscreenlaunchcomponent/source/fullscreenlaunchcomponent.ets` |

**功能**: 全屏启动组件，应用启动时展示全屏启动页。

---

### HalfScreenLaunchComponent

| 属性 | 值 |
|------|-----|
| **路径** | `halfscreenlaunchcomponent/` |
| **源码** | `halfscreenlaunchcomponent/source/halfscreenlaunchcomponent.ets` |

**功能**: 半屏启动组件，应用启动时展示半屏启动页。

---

### InnerFullScreenLaunchComponent

| 属性 | 值 |
|------|-----|
| **路径** | `innerfullscreenlaunchcomponent/` |
| **源码** | `innerfullscreenlaunchcomponent/source/innerfullscreenlaunchcomponent.ets` |

**功能**: 内部全屏启动组件，用于子窗口场景的启动页。

---

## 弹窗组件

### InterstitialDialogAction

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.InterstitialDialogAction` |
| **路径** | `interstitialdialogaction/` |
| **源码** | `interstitialdialogaction/source/interstitialdialogaction.ets` |

**功能**: 插屏弹窗组件，用于在应用运行过程中展示广告或重要信息。

---

## 导航助手

### NavPushPathHelper

| 属性 | 值 |
|------|-----|
| **N-API 模块** | `atomicservice.NavPushPathHelper` |
| **路径** | `navpushpathhelper/` |
| **头文件** | `navpushpathhelper/include/hsp_silentinstall_napi.h` |

#### JS API 清单

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `silentInstall(moduleName, callback?)` | string, function? | Promise/void | HSP 静默安装 |
| `isHspExist(moduleName)` | string | boolean | 检查 HSP 是否存在 |
| `updateRouteMap(routeMap, callback?)` | object, function? | Promise/void | 更新路由表 |

#### silentInstall 详细说明

**C++ 实现**: `HspSilentInstallNapi::SilentInstall()`

**参数**:
| 参数 | 类型 | 说明 | 必需 |
|------|------|------|------|
| moduleName | string | HSP 模块名 | 是 |
| callback | function | 回调函数 (可选) | 否 |

**回调签名**:
```typescript
interface InstallCallback {
  (errCode: number, errorMessage: string): void;
}
```

**异步模式**:
- 使用 `uv_work_t` 进行异步工作
- 支持成功/失败双回调
- 自动管理回调引用生命周期

**证据来源**: `navpushpathhelper/include/hsp_silentinstall_napi.h:27-50`

#### isHspExist 详细说明

**C++ 实现**: `HspSilentInstallNapi::IsHspExist()`

**参数**:
| 参数 | 类型 | 说明 | 必需 |
|------|------|------|------|
| moduleName | string | HSP 模块名 | 是 |

**返回值**:
| 类型 | 说明 |
|------|------|
| boolean | true: 存在, false: 不存在 |

#### updateRouteMap 详细说明

**C++ 实现**: `HspSilentInstallNapi::UpdateRouteMap()`

**参数**:
| 参数 | 类型 | 说明 | 必需 |
|------|------|------|------|
| routeMap | object | 路由映射表 | 是 |
| callback | function | 回调函数 (可选) | 否 |

---

## 组件依赖矩阵

| 组件 | hilog | napi | ace_ndk | ability | bundle | window | ipc | samgr |
|------|-------|------|--------|---------|--------|-------|-----|------|
| AtomServiceNavigation | ✅ | ✅ | - | - | - | - | - | - |
| AtomServiceSearch | ✅ | ✅ | - | - | - | - | - | - |
| AtomServiceTabs | ✅ | ✅ | - | - | - | - | - | - |
| AtomServiceWeb | ✅ | ✅ | - | - | - | - | - | - |
| CustomAppBar | ✅ | ✅ | - | - | - | - | - | - |
| CustomAppBarMenuBar | ✅ | ✅ | ✅ | - | - | - | - | - |
| FullScreenLaunchComponent | - | - | - | - | - | - | - | - |
| HalfScreenLaunchComponent | - | - | - | - | - | - | - | - |
| InnerFullScreenLaunchComponent | - | - | - | - | - | - | - | - |
| InterstitialDialogAction | - | - | - | - | - | - | - | - |
| NavPushPathHelper | ✅ | ✅ | - | ✅ | ✅ | - | ✅ | ✅ |

**证据来源**: 各组件 `interfaces/BUILD.gn` 文件

---

## 附录: N-API 头文件索引

| 组件 | 头文件 | 主要内容 |
|------|--------|----------|
| AtomServiceWeb | `api_policy_adapter.h` | ApiPolicyAdapter 类 |
| NavPushPathHelper | `hsp_silentinstall_napi.h` | HspSilentInstallNapi 类 |
| NavPushPathHelper | `hsp_silentinstall.h` | HSP 安装相关 |
| NavPushPathHelper | `silent_install_callback.h` | 回调定义 |
| CustomAppBarMenuBar | `menubar_api_implement.h` | MenubarAPIImplement 类 |
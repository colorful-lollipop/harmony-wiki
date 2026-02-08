# Inner API

> 本文档描述 Form Fwk 的 Inner API，包括模块接口、依赖方向、稳定性标注

## Inner API 概览

| 接口 | Broker Descriptor | 路径 | 稳定性 |
|------|-------------------|------|--------|
| **IFormMgr** | `ohos.appexecfwk.FormMgr` | `interfaces/inner_api/include/` | 稳定 |
| **IFormProvider** | `ohos.appexecfwk.FormProvider` | `interfaces/inner_api/include/` | 稳定 |
| **IFormHost** | `ohos.appexecfwk.FormHost` | `interfaces/inner_api/include/` | 稳定 |
| **IFormSupply** | `ohos.appexecfwk.FormSupply` | `interfaces/inner_api/include/` | 稳定 |
| **IFormRender** | `ohos.appexecfwk.FormRender` | `interfaces/inner_api/include/` | 稳定 |
| **IJsFormStateObserver** | `ohos.aafwk.IJsFormStateObserver` | `interfaces/inner_api/include/` | 稳定 |
| **IFormPublishInterceptor** | `ohos.appexecfwk.FormPublishInterceptor` | `interfaces/inner_api/include/` | 稳定 |

## IFormMgr 接口

**头文件**: `interfaces/inner_api/include/form_mgr_interface.h`
**IPC ID 范围**: 3001-3500

### 核心方法

| 方法 | IPC ID | 功能 |
|------|--------|------|
| AddForm | 3001 | 添加卡片 |
| DeleteForm | 3003 | 删除卡片 |
| UpdateForm | 3005 | 更新卡片 |
| RequestForm | 3007 | 请求卡片 |
| ReleaseForm | 3009 | 释放卡片 |
| SetNextRefreshTime | 3013 | 设置刷新时间 |
| AcquireFormState | 3045 | 获取卡片状态 |
| GetAllFormsInfo | 3049 | 获取所有卡片信息 |
| GetFormsInfoByApp | 3051 | 按应用获取卡片 |
| GetFormsInfoByModule | 3053 | 按模块获取卡片 |
| ShareForm | 3079 | 分享卡片 |
| CheckFMSReady | 3097 | 检查 FMS 就绪 |

### 依赖方向

```
IFormMgr
    ├── IPC: IFormProvider (回调)
    ├── IPC: IFormHost (通知)
    ├── IPC: IFormRender (渲染)
    ├── SAMgr: ABILITY_MGR_SERVICE_ID
    ├── SAMgr: BUNDLE_MGR_SERVICE_ID
    └── SAMgr: APP_MGR_SERVICE_ID
```

## IFormProvider 接口

**头文件**: `interfaces/inner_api/include/form_provider_interface.h`
**IPC ID 范围**: 3051-3062

### 核心方法

| 方法 | IPC ID | 功能 |
|------|--------|------|
| AcquireFormData | 3051 | 获取卡片数据 |
| EventNotify | 3053 | 事件通知 |
| UpdateForm | 3055 | 更新表单 |
| MessageEvent | 3057 | 消息事件 |
| RouterEvent | 3059 | 路由事件 |
| BackgroundEvent | 3061 | 后台事件 |

## IFormHost 接口

**头文件**: `interfaces/inner_api/include/form_host_interface.h`
**IPC ID 范围**: 3681-3693

### 核心方法

| 方法 | IPC ID | 功能 |
|------|--------|------|
| AcquireProviderData | 3681 | 获取提供者数据 |
| FormEventNotify | 3683 | 卡片事件通知 |
| AcquireFormState | 3685 | 获取卡片状态 |
| ReleaseForm | 3687 | 释放卡片 |
| DeleteForm | 3689 | 删除卡片 |

## IFormSupply 接口

**头文件**: `interfaces/inner_api/include/form_supply_interface.h`
**IPC ID 范围**: 3201-3214

### 核心方法

| 方法 | IPC ID | 功能 |
|------|--------|------|
| OnAcquireData | 3201 | 数据获取回调 |
| OnEventNotify | 3203 | 事件通知回调 |
| OnUpdateForm | 3205 | 更新卡片回调 |
| OnMessageEvent | 3207 | 消息事件回调 |
| OnRouterEvent | 3209 | 路由事件回调 |

## IFormRender 接口

**头文件**: `interfaces/inner_api/include/form_render_interface.h`
**IPC ID 范围**: 3101-3113

### 核心方法

| 方法 | IPC ID | 功能 |
|------|--------|------|
| RenderForm | 3101 | 渲染卡片 |
| UpdateForm | 3103 | 更新卡片 |
| ReleaseRenderer | 3105 | 释放渲染器 |
| RecycleRenderer | 3107 | 回收渲染器 |
| RecoverRenderer | 3109 | 恢复渲染器 |

## Inner API 实现位置

### Proxy 实现

| 接口 | Proxy 文件 | 说明 |
|------|------------|------|
| IFormMgr | `interfaces/inner_api/src/form_mgr_proxy.cpp` | 客户端代理 |
| IFormProvider | `interfaces/inner_api/src/form_provider_proxy.cpp` | 客户端代理 |
| IFormHost | `interfaces/inner_api/src/form_host_proxy.cpp` | 客户端代理 |
| IFormSupply | `interfaces/inner_api/src/form_supply_proxy.cpp` | 客户端代理 |
| IFormRender | `interfaces/inner_api/src/form_render_proxy.cpp` | 客户端代理 |

### Stub 实现

| 接口 | Stub 文件 | 说明 |
|------|-----------|------|
| IFormMgr | `interfaces/inner_api/src/form_mgr_stub.cpp` | 服务端存根 |
| IFormProvider | `interfaces/inner_api/src/form_provider_stub.cpp` | 服务端存根 |
| IFormHost | `interfaces/inner_api/src/form_host_stub.cpp` | 服务端存根 |
| IFormSupply | `interfaces/inner_api/src/form_supply_stub.cpp` | 服务端存根 |
| IFormRender | `interfaces/inner_api/src/form_render_stub.cpp` | 服务端存根 |

## 稳定性标注

### 稳定接口（Stable）

以下接口为系统内部稳定接口：

| 接口 | 标记位置 | 说明 |
|------|----------|------|
| IFormMgr | `innerapi_tags = ["platformsdk"]` | 平台 SDK |
| IFormProvider | `innerapi_tags = ["platformsdk"]` | 平台 SDK |
| IFormHost | `innerapi_tags = ["platformsdk"]` | 平台 SDK |
| IFormSupply | `innerapi_tags = ["platformsdk"]` | 平台 SDK |
| IFormRender | `innerapi_tags = ["platformsdk"]` | 平台 SDK |

### 稳定性说明

| 标记 | 含义 |
|------|------|
| platformsdk | 平台 SDK 接口，供系统应用使用 |
| - | 内部接口，仅限系统组件使用 |

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                        FormMgrService                            │
│                     (System Ability 403)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌──────────────┐    IPC    ┌──────────────┐                │
│    │  IFormProvider│◀──────────│  IFormSupply │                │
│    │  (Proxy)     │           │  (Stub)      │                │
│    └──────┬───────┘           └──────────────┘                │
│           │ IPC 调用                                              │
│           ▼                                                      │
│    ┌─────────────────────────────────────────────────────┐    │
│    │                    IPC 分发层                          │    │
│    │  OnRemoteRequest() → 方法路由 → 各业务处理            │    │
│    └─────────────────────────────────────────────────────┘    │
│                                                                  │
│    ┌──────────────┐    IPC    ┌──────────────┐                │
│    │  IFormHost   │──────────▶│  IFormHost   │                │
│    │  (Stub)      │           │  (Proxy)    │                │
│    └──────────────┘           └──────────────┘                │
│                                                                  │
│    ┌──────────────┐    IPC    ┌──────────────┐                │
│    │  IFormRender │──────────▶│  IFormRender │                │
│    │  (Stub)     │           │  (Proxy)     │                │
│    └──────────────┘           └──────────────┘                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

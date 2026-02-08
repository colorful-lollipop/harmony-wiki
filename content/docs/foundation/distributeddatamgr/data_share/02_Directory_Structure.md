# Data Share 目录结构

## 目的

本文档介绍 Data Share 项目的目录组织和模块职责，帮助开发者快速定位代码。

## 适用范围

- 需要理解代码组织的新加入开发者
- 进行代码审查或修改的维护者
- 需要查找特定功能的开发者

## 顶层目录结构

```
foundation/distributeddatamgr/data_share
├── common/                    # 公共代码（ANI/Rust）
├── frameworks/                # 框架实现
│   ├── cj/ffi/               # Cangjie FFI 接口
│   ├── ets/ani/              # ArkTS ANI 实现
│   ├── js/                   # JavaScript/NAPI 实现
│   └── native/               # Native C++ 实现
├── interfaces/                # 接口定义
│   └── inner_api/            # 内部 API
└── test/                     # 测试代码（本文档不覆盖）
```

## 详细目录说明

### common/ - 公共代码

**路径**: `common/`

| 子目录 | 说明 | 关键文件 |
|--------|------|----------|
| `ani_rs/` | Rust ANI 绑定 | `src/ani/ani_rs_bind.cpp` |
| `ani_rs_macros/` | Rust 过程宏 | `src/lib.rs` |
| `ani_sys/` | ANI 系统接口 | `src/ani_sys.cpp` |

**证据**: 代码路径 `common/ani_rs/src/ani_rs_bind.cpp:12`

### frameworks/cj/ffi/ - Cangjie FFI

**路径**: `frameworks/cj/ffi/`

| 子目录 | 说明 | 关键文件 |
|--------|------|----------|
| `data_share_predicates/` | 谓词 FFI 实现 | `src/data_share_predicates_impl.cpp` |

为 Cangjie 语言提供 DataSharePredicates 的 FFI 绑定。

**证据**: 代码路径 `frameworks/cj/ffi/data_share_predicates/src/data_share_predicates_impl.cpp:12`

### frameworks/ets/ani/ - ArkTS ANI

**路径**: `frameworks/ets/ani/`

| 子目录 | 说明 | 关键文件 |
|--------|------|----------|
| `ets/` | ArkTS 类型定义 | `datasharehelper.ets` |
| `include/` | C++ 头文件 | `ani_datashare_helper.h` |
| `src/` | C++ 实现 | `cxx/ani_subscriber.cpp` |

为 ArkTS 语言提供 ANI (ArkTS Native Interface) 绑定。

**证据**: 代码路径 `frameworks/ets/ani/src/cxx/ani_subscriber.cpp:12`

### frameworks/js/ani/ - JS ANI 适配

**路径**: `frameworks/js/ani/`

| 子目录 | 说明 | 关键文件 |
|--------|------|----------|
| `common/` | 公共代码 | `src/ani_utils.cpp` |
| `dataShare/` | DataShare ANI 实现 | `src/ani_datashare_helper.cpp` |
| `dataShareResultSet/` | ResultSet ANI 实现 | `src/data_share_result_set.cpp` |

为 JavaScript 提供 ANI 适配层。

**证据**: 代码路径 `frameworks/js/ani/dataShare/src/ani_datashare_helper.cpp:12`

### frameworks/js/napi/ - NAPI 绑定

**路径**: `frameworks/js/napi/`

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `common/` | NAPI 公共代码 | `src/datashare_predicates_proxy.cpp` |
| `dataShare/` | DataShare NAPI 实现 | `src/napi_datashare_helper.cpp` |
| `datashare_ext_ability/` | ExtensionAbility NAPI | `datashare_ext_ability_module.cpp` |
| `datashare_ext_ability_context/` | Context NAPI | `datashare_ext_ability_context_module.cpp` |
| `observer/` | 观察者 NAPI | `src/napi_observer.cpp` |

**核心文件**:

| 文件路径 | 说明 | 关键代码 |
|----------|------|----------|
| `dataShare/src/native_datashare_module.cpp` | data.dataShare 模块入口 | `napi_module_register(&_module)` line 64 |
| `dataShare/src/napi_datashare_helper.cpp` | DataShareHelper NAPI 实现 | `Napi_CreateDataShareHelper()` line 186 |
| `dataShare/src/native_datashare_predicates_module.cpp` | Predicates 模块入口 | `DataSharePredicatesProxy::Init()` line 32 |
| `common/src/datashare_predicates_proxy.cpp` | 谓词代理实现 | `CreateConstructor()` line 40 |

**证据**:
- `frameworks/js/napi/dataShare/src/native_datashare_module.cpp:64` - 模块注册
- `frameworks/js/napi/dataShare/src/napi_datashare_helper.cpp:249-264` - Helper 类方法注册

### frameworks/native/ - Native C++ 实现

**路径**: `frameworks/native/`

#### native/common/ - 公共实现

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `src/datashare_itypes_utils.cpp` | IPC 类型工具 | 序列化/反序列化 |
| `src/ishared_result_set.cpp` | 共享结果集 | 跨进程结果集传输 |
| `src/datashare_uri_utils.cpp` | URI 工具 | URI 解析与验证 |
| `src/call_reporter.cpp` | 调用报告 | HiView 故障上报 |

**证据**: `frameworks/native/common/src/ishared_result_set.cpp:12`

#### native/consumer/ - 客户端实现

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `src/` | 客户端核心实现 | `datashare_helper_impl.cpp` |
| `controller/` | 控制器实现 | `general_controller_service_impl.cpp` |
| `include/` | 客户端头文件 | `datashare_proxy.h` |

**核心类**:

| 类名 | 文件 | 职责 |
|------|------|------|
| `DataShareHelper` | `interfaces/inner_api/consumer/include/datashare_helper.h:49` | 客户端主接口 |
| `DataShareHelperImpl` | `frameworks/native/consumer/src/datashare_helper_impl.cpp` | Helper 实现 |
| `DataShareProxy` | `frameworks/native/consumer/include/datashare_proxy.h:29` | IPC 客户端代理 |
| `GeneralControllerServiceImpl` | `frameworks/native/consumer/controller/service/src/general_controller_service_impl.cpp` | Silent 模式控制器 |
| `GeneralControllerProviderImpl` | `frameworks/native/consumer/controller/provider/src/general_controller_provider_impl.cpp` | Non-Silent 模式控制器 |

**证据**:
- `interfaces/inner_api/consumer/include/datashare_helper.h:49` - DataShareHelper 类定义
- `frameworks/native/consumer/include/datashare_proxy.h:29` - DataShareProxy 类定义

#### native/provider/ - 服务端实现

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `src/` | 服务端核心实现 | `datashare_stub_impl.cpp` |
| `include/` | 服务端头文件 | `datashare_stub.h` |

**核心类**:

| 类名 | 文件 | 职责 |
|------|------|------|
| `DataShareStub` | `frameworks/native/provider/include/datashare_stub.h:28` | IPC 服务端存根 |
| `DataShareStubImpl` | `frameworks/native/provider/src/datashare_stub_impl.cpp` | 服务端业务实现 |
| `DataShareExtAbility` | `frameworks/native/provider/src/datashare_ext_ability.cpp` | 扩展能力基类 |
| `JsDataShareExtAbility` | `frameworks/native/provider/src/js_datashare_ext_ability.cpp` | JS 扩展能力 |

**证据**:
- `frameworks/native/provider/include/datashare_stub.h:28` - DataShareStub 类定义
- `frameworks/native/provider/src/datashare_stub_impl.cpp:55` - CheckCallingPermission 实现

#### native/permission/ - 权限管理

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `src/` | 权限实现 | `data_share_permission.cpp` |
| `include/` | 权限头文件 | `data_share_permission.h` |

**核心类**:

| 类名 | 文件 | 职责 |
|------|------|------|
| `DataSharePermission` | `interfaces/inner_api/permission/include/data_share_permission.h:30` | 权限验证主类 |
| `DataShareCalledConfig` | `frameworks/native/permission/include/data_share_called_config.h` | Provider 信息获取 |

**证据**:
- `interfaces/inner_api/permission/include/data_share_permission.h:30` - DataSharePermission 类定义
- `frameworks/native/permission/src/data_share_permission.cpp:63` - VerifyPermission 实现

#### native/proxy/ - 代理实现

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `src/` | 代理实现 | `data_share_manager_impl.cpp` |
| `include/` | 代理头文件 | `data_share_service_proxy.h` |

**核心类**:

| 类名 | 文件 | 职责 |
|------|------|------|
| `DataShareManagerImpl` | `frameworks/native/proxy/src/data_share_manager_impl.cpp` | 管理器实现 |
| `DataShareServiceProxy` | `frameworks/native/proxy/src/data_share_service_proxy.cpp` | 系统服务代理 |

#### native/dfx/ - 诊断与监控

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `src/` | DFX 实现 | `hiview_datashare.cpp` |
| `include/` | DFX 头文件 | `hiview_datashare.h` |

### interfaces/inner_api/ - 内部 API

**路径**: `interfaces/inner_api/`

#### inner_api/common/ - 公共接口

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `include/datashare_errno.h` | 错误码定义 | `E_OK`, `E_ERROR` 等 |
| `include/datashare_predicates.h` | 谓词定义 | `DataSharePredicates` 类 |
| `include/datashare_values_bucket.h` | 值容器 | `DataShareValuesBucket` 类 |
| `include/datashare_observer.h` | 观察者接口 | `DataShareObserver` 类 |
| `include/datashare_type.h` | 类型定义 | 数据结构定义 |
| `include/datashare_template.h` | 模板定义 | 订阅模板 |

**证据**:
- `interfaces/inner_api/common/include/datashare_errno.h:30` - 错误码定义
- `interfaces/inner_api/common/include/datashare_predicates.h:29` - 谓词类定义

#### inner_api/consumer/ - 客户端接口

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `include/datashare_helper.h` | 主接口 | `DataShareHelper` 类 line 49 |
| `include/datashare_result_set.h` | 结果集 | `DataShareResultSet` 类 |
| `include/datashare_business_error.h` | 业务错误 | `DatashareBusinessError` 类 |
| `include/dataproxy_handle.h` | 代理句柄 | `DataProxyHandle` 类 |

**证据**: `interfaces/inner_api/consumer/include/datashare_helper.h:49`

#### inner_api/provider/ - 服务端接口

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `include/result_set_bridge.h` | 结果集桥接 | `ResultSetBridge` 类 line 24 |

**证据**: `interfaces/inner_api/provider/include/result_set_bridge.h:24`

#### inner_api/permission/ - 权限接口

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `include/data_share_permission.h` | 权限接口 | `DataSharePermission` 类 line 30 |

**证据**: `interfaces/inner_api/permission/include/data_share_permission.h:30`

## 代码文件统计

| 模块 | 头文件数 | 实现文件数 | 主要职责 |
|------|---------|-----------|----------|
| NAPI (JS) | ~10 | ~17 | JS 绑定层 |
| Native Consumer | ~8 | ~15 | 客户端实现 |
| Native Provider | ~6 | ~11 | 服务端实现 |
| Native Permission | ~3 | ~3 | 权限管理 |
| Native Common | ~6 | ~8 | 公共实现 |
| Inner API | ~23 | - | 接口定义 |

## 关键结论

1. **清晰的层次结构** - 从 NAPI/ANI 到 Native 再到 Inner API，层次分明
2. **双模式支持** - Consumer 目录下的 controller 子目录分别实现 Silent 和 Non-Silent 模式
3. **权限独立模块** - permission 目录独立管理权限相关逻辑
4. **多语言支持** - js/napi、ets/ani、cj/ffi 分别支持 JavaScript、ArkTS、Cangjie
5. **接口与实现分离** - interfaces/inner_api 只定义接口，实现在 frameworks/native

## 导航提示

- **查找 N-API 实现** → `frameworks/js/napi/`
- **查找客户端逻辑** → `frameworks/native/consumer/`
- **查找服务端逻辑** → `frameworks/native/provider/`
- **查找权限检查** → `frameworks/native/permission/`
- **查找接口定义** → `interfaces/inner_api/`
- **查找错误码** → `interfaces/inner_api/common/include/datashare_errno.h`

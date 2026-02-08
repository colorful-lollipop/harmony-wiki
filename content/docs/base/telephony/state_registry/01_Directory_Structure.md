# 目录结构

## 顶层结构

```
state_registry/
├── figures/                      # 文档用图
├── frameworks/                   # 框架层 [核心代码]
│   ├── native/                  # Native 框架
│   ├── js/                      # JS N-API 框架
│   ├── ets/                     # ETS (ArkUI Native) 框架
│   └── cj/                       # CJ FFI 框架
├── interfaces/                  # 接口层
│   ├── kits/                    # 外部接口 (JS API)
│   └── innerkits/               # 内部接口 (Native API)
├── services/                    # 服务层
│   ├── src/                     # 服务实现
│   ├── include/                # 服务头文件
│   └── telephony_ext_wrapper/   # 扩展包装器
├── sa_profile/                  # SA 配置文件
├── BUILD.gn                     # 根构建文件
├── bundle.json                  # 组件配置
├── README.md                    # 英文说明
└── README_zh.md                 # 中文说明
```

**证据来源**：仓库根目录文件列表及 `README.md` 目录结构说明。

## 框架层详解

### frameworks/native/

提供 Native 层的观察者实现，是整个模块的 Native 核心。

```
frameworks/native/
├── observer/
│   ├── include/
│   │   ├── telephony_observer_proxy.h     # 观察者代理
│   │   └── telephony_callback_observer.h  # 回调观察者
│   └── src/
│       └── telephony_observer_proxy.cpp   # 代理实现
└── common/
    └── include/
```

**关键文件**：
- `frameworks/native/observer/include/telephony_observer_proxy.h`：观察者代理接口定义
- `frameworks/native/observer/src/telephony_observer_proxy.cpp`：代理实现源码

### frameworks/js/

提供 JS N-API 绑定层，实现 JS 到 Native 的桥接。

```
frameworks/js/
└── napi/
    ├── observer/                   # 观察者 N-API 实现
    │   ├── napi_observer.cpp       # N-API 注册入口
    │   ├── napi_observer.h         # N-API 头文件
    │   ├── napi_observer_utils.cpp # 工具函数
    │   ├── napi_observer_utils.h   # 工具头文件
    │   ├── napi_observer_network.cpp # 网络事件绑定
    │   ├── napi_observer_call.cpp    # 通话事件绑定
    │   ├── napi_observer_signal.cpp   # 信号事件绑定
    │   ├── napi_observer_cell.cpp     # 小区事件绑定
    │   ├── napi_observer_sim.cpp      # SIM 事件绑定
    │   ├── napi_observer_data.cpp     # 数据连接事件绑定
    │   └── BUILD.gn                    # 构建配置
    └── BUILD.gn                        # 父级构建配置
```

**关键文件**：
- `frameworks/js/napi/observer/napi_observer.cpp`：N-API 模块注册入口

### frameworks/ets/

提供 ETS (ArkUI Native Interface) 接口支持。

```
frameworks/ets/
└── ani/
    └── observer/                   # ETS ANI 实现
        ├── observer.cpp            # ANI 实现
        ├── observer.h             # ANI 头文件
        └── BUILD.gn               # 构建配置
```

**证据来源**：`bundle.json` 中 `build.group_type.fwk_group` 包含 `//base/telephony/state_registry/frameworks/ets/ani/observer:observer_ani_group`。

### frameworks/cj/

提供 CJ (C++ JavaScript) FFI 接口支持。

```
frameworks/cj/
├── src/                          # CJ 实现源码
└── BUILD.gn                      # 构建配置
```

**证据来源**：`bundle.json` 中 `inner_kits` 包含 `//base/telephony/state_registry/frameworks/cj:cj_observer_ffi`。

## 接口层详解

### interfaces/kits/

发布对外 JS API 声明，供应用开发者使用。

```
interfaces/kits/
└── js/
    └── @ohos.telephony.observer.d.ts  # JS API 类型定义
```

**关键文件**：
- `interfaces/kits/js/@ohos.telephony.observer.d.ts`：定义 `observer` 模块的完整 JS API

### interfaces/innerkits/

发布内部 Native API 声明，供系统组件使用。

```
interfaces/innerkits/
└── observer/
    ├── telephony_observer.h            # 观察者接口
    └── telephony_observer_client.h      # 观察者客户端接口
```

**关键文件**：
- `interfaces/innerkits/observer/telephony_observer.h`：内部观察者接口
- `interfaces/innerkits/observer/telephony_observer_client.h`：客户端接口

## 服务层详解

### services/src/

服务核心实现代码。

```
services/src/
├── telephony_state_registry_service.cpp    # SA 服务主程序
├── telephony_state_registry_stub.cpp        # IPC Stub 实现
├── telephony_state_registry_record.cpp      # 记录管理
└── telephony_state_registry_dump_helper.cpp # Dump 辅助
```

**关键文件**：
- `services/src/telephony_state_registry_service.cpp`：系统服务主入口
- `services/src/telephony_state_registry_stub.cpp`：IPC 接口存根

### services/include/

服务头文件声明。

```
services/include/
├── telephony_state_registry_service.h  # 服务接口
├── telephony_state_registry_stub.h     # Stub 接口
├── telephony_state_registry_record.h   # 记录接口
└── telephony_state_registry_dump_helper.h # Dump 接口
```

### services/telephony_ext_wrapper/

电信扩展包装器实现。

```
services/telephony_ext_wrapper/
├── src/
│   └── telephony_ext_wrapper.cpp       # 包装器实现
└── include/
    └── telephony_ext_wrapper.h         # 头文件声明
```

## SA Profile

```
sa_profile/
└── state_registry_sa_profile.xml        # SA 配置文件
```

**证据来源**：`bundle.json` 中 `build.group_type.service_group` 包含 `//base/telephony/state_registry/sa_profile:state_registry_sa_profile`。

## 模块职责总结

| 目录 | 职责 | 稳定性 |
|------|------|--------|
| frameworks/native | Native 业务逻辑 | 稳定 |
| frameworks/js | JS N-API 绑定 | 稳定 |
| frameworks/ets | ETS ANI 接口 | 稳定 |
| frameworks/cj | CJ FFI 接口 | 稳定 |
| interfaces/kits | JS API 声明 | 稳定 |
| interfaces/innerkits | Native API 声明 | 稳定 |
| services | SA 服务实现 | 稳定 |
| sa_profile | SA 配置 | 稳定 |

## 相关文档

- [架构设计](02_Architecture.md)
- [JS API](03_JS_API.md)
- [GN 构建](05_GN_Build.md)

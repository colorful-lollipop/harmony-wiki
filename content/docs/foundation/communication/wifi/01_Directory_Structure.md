# 目录结构与模块职责

**目的**: 描述 WLAN 组件的目录结构、各模块的职责和主要文件

**适用范围**: 所有开发者、架构师、代码审查者

**生成时间**: 2026-02-06

---

## 目录树概览（排除测试）

```
wifi/
├── application/                    # 应用层
│   ├── portal_login/            # 门户登录应用
│   └── wifi_direct_demo/        # WiFi P2P 演示应用
├── base/                         # 基础工具
│   ├── cRPC/                   # cRPC 通信库
│   │   ├── include/
│   │   └── src/
│   ├── inner_api/               # 内部 API（接口定义）
│   │   └── *.h
│   ├── security_utils/           # 安全工具（加密、签名等）
│   │   ├── include/
│   │   └── src/
│   ├── shared_util/             # 共享工具函数
│   │   └── *.cpp
│   ├── state_machine/           # 状态机框架
│   │   ├── include/
│   │   │   └── wifi_state_machine.h
│   │   └── src/
│   └── utils/                  # 工具函数
│       ├── inc/
│       └── src/
├── frameworks/                    # 框架实现层
│   ├── cj/                     # Cangjie 语言绑定
│   │   ├── include/
│   │   └── src/
│   ├── ets/                    # ArkTS 实现
│   │   └── taihe/wifiManager/  # WiFi Manager ETS
│   │       ├── idl/
│   │       ├── include/
│   │       └── src/
│   ├── js/napi/                # JavaScript N-API 绑定
│   │   ├── inc/                # N-API 头文件
│   │   └── src/                # N-API 实现
│   ├── native/                 # 原生 C++ 实现
│   │   ├── interfaces/         # IDL 接口定义
│   │   │   └── *.idl
│   │   ├── include/
│   │   │   └── i_*.h
│   │   └── src/             # Proxy/Stub 实现
│   └── wifi_ndk/               # NDK（Native Development Kit）
│       ├── include/
│       │   └── oh_wifi.h
│       └── src/
│           └── oh_wifi.cpp
├── interfaces/                    # 接口定义层
│   ├── c_api/                  # C API
│   │   ├── include/
│   │   └── *.h
│   ├── inner_api/               # 内部 API（对外暴露）
│   │   └── *.h
│   └── kits/                    # JS Kit 接口（数据结构）
│       └── c/
│           └── *.h
├── relation_services/             # 关联服务
│   ├── dhcp_service/            # DHCP 服务适配
│   ├── common/                 # 通用代码
│   └── etc/                    # 配置文件
│       └── init/                 # 服务启动脚本
│           ├── default_conf/
│           ├── udp_socket/
│           └── unix_socket/
├── services/                     # 服务实现层
│   └── wifi_standard/         # 标准服务实现
│       ├── etc/                # 配置文件
│       │   ├── init/             # SA 启动配置
│       │   │   └── wifi_standard.cfg
│       │   ├── param/            # 系统参数
│       │   └── sa_profile/       # SA 配置文件
│       │       ├── 1120.json
│       │       ├── 1121.json
│       │       ├── 1123.json
│       │       └── 1124.json
│       ├── include/            # 服务头文件
│       └── wifi_framework/     # WiFi 框架核心实现
│           ├── wifi_manage/      # WiFi 管理（核心逻辑）
│           │   ├── wifi_sta/            # Station（STA）服务
│           │   │   ├── *sa/            # SA 实现和服务
│           │   │   ├── sta_interface.cpp
│           │   │   ├── sta_service.cpp
│           │   │   └── sta_state_machine.cpp
│           │   ├── wifi_sta_ext/        # STA 扩展服务
│           │   ├── wifi_ap/             # AP 服务
│           │   │   ├── *sa/
│           │   │   ├── ap_interface.cpp
│           │   │   └── ap_service.cpp
│           │   ├── wifi_p2p/            # P2P 服务
│           │   │   ├── *sa/
│           │   │   ├── p2p_interface.cpp
│           │   │   └── p2p_service.cpp
│           │   ├── wifi_scan/           # 扫描服务
│           │   │   ├── *sa/
│           │   │   ├── scan_interface.cpp
│           │   │   └── scan_service.cpp
│           │   ├── wifi_native/         # 原生接口适配
│           │   │   ├── client/
│           │   │   ├── hdi_client/
│           │   │   └── common/
│           │   ├── wifi_controller/   # 控制器
│           │   │   ├── concrete_clientmode_manager.cpp
│           │   │   ├── concrete_manager_state_machine.cpp
│           │   │   ├── multi_sta_manager.cpp
│           │   │   └── wifi_service_scheduler.cpp
│           │   └── wifi_common/         # 通用功能
│           │       ├── wifi_auth_center.cpp      # 认证中心
│           │       ├── wifi_permission_helper.cpp # 权限助手
│           │       ├── wifi_permission_utils.cpp  # 权限工具
│           │       ├── wifi_internal_event_dispatcher.cpp # 事件分发
│           │       ├── wifi_protect_manager.cpp   # 保护管理
│           │       ├── wifi_country_code/    # 国家码管理
│           │       ├── network_black_list/   # 网络黑名单
│           │       ├── network_status_history/ # 网络状态历史
│           │       ├── rdb/                 # 数据库适配
│           │       ├── net_eap/             # EAP 认证
│           │       ├── wifi_net_agent.cpp    # 网络代理
│           │       ├── wifi_app_state_aware.cpp # 应用状态感知
│           │       └── wifi_config_center.cpp     # 配置中心
│           ├── wifi_pro/              # WiFi Pro 功能
│           │   ├── perf_5g/         # 5G 性能优化
│           │   ├── wifi_intelligence/ # WiFi 智能功能
│           │   └── wifi_security_detect/ # WiFi 安全检测
│           ├── wifi_self_cure/        # WiFi 自愈功能
│           └── wifi_sta_ext/          # STA 扩展服务
│               ├── wifi_data_report/     # 数据报告
│               └── wifi_telephony_utils/ # 电话工具
│           ├── wifi_scan_sa/          # 扫描 SA
│           ├── wifi_sta_sa/           # STA SA
│           ├── wifi_ap_sa/            # AP SA
│           └── wifi_p2p_sa/          # P2P SA
│           └── wifi_toolkit/        # 工具包
│               ├── config/              # 配置管理
│               ├── net_helper/          # 网络辅助
│               └── utils/               # 工具函数
├── utils/                        # 工具函数
│   ├── extern_library/          # 外部库
│   ├── inc/                    # 工具头文件
│   └── src/                    # 工具源文件
├── BUILD.gn                     # 根构建文件
├── bundle.json                  # Bundle 配置
├── hisysevent.yaml              # HiSysEvent 配置
├── wifi.gni                    # 主配置文件
└── wifi_lite.gni               # 轻量级配置
```

---

## 模块职责详解

### 1. Application Layer（应用层）

#### `portal_login/`
**职责**: 门户认证登录应用
**主要文件**:
- `entry/src/main/ets/` - ArkTS 源码
- `AppScope/resources/` - 资源文件
**功能**: 提供 Web 门户认证界面，用于强制门户场景下的用户认证

#### `wifi_direct_demo/`
**职责**: WiFi Direct (P2P) 演示应用
**主要文件**:
- `entry/src/main/ets/` - ArkTS 源码
**功能**: 展示 P2P 设备发现、连接和组管理的示例实现

---

### 2. Base Layer（基础层）

#### `cRPC/`
**职责**: cRPC 通信库
**主要文件**:
- `include/wifi_crpc.h`
- `src/wifi_crpc.cpp`
**功能**: 提供跨语言（Cangjie 与 C++）间的远程过程调用机制

#### `security_utils/`
**职责**: 安全工具
**主要文件**:
- `include/wifi_encryption_util.h`
- `src/wifi_encryption_util.cpp`
**功能**: WiFi 配置加密工具（如 WPA 密码加密）

#### `state_machine/`
**职责**: 状态机框架
**主要文件**:
- `include/wifi_state_machine.h`
- `src/wifi_state_machine.cpp`
**功能**: 提供通用的状态机实现框架，用于 WiFi 连接、扫描等状态管理

#### `shared_util/`
**职责**: 共享工具函数
**主要文件**: 多个 `.cpp` 工具文件
**功能**: 提供日志、字符串处理等通用工具函数

#### `utils/`
**职责**: 工具函数库
**主要文件**:
- `src/wifi_intl_util.cpp` - 国际化工具
- `src/wifi_common_util.cpp` - 通用工具
**功能**: Bundle 信息获取、进程身份提取、系统服务访问等

---

### 3. Frameworks Layer（框架层）

#### `js/napi/`
**职责**: JavaScript N-API 绑定层
**主要文件**:
- `src/wifi_napi_entry.cpp` - 主 N-API 模块注册（70+ 导出函数）
- `src/wifi_ext_napi_entry.cpp` - 扩展 N-API 模块
- `src/wifi_napi_device.cpp` - STA 设备操作
- `src/wifi_napi_hotspot.cpp` - 热点操作
- `src/wifi_napi_p2p.cpp` - P2P 操作
- `src/wifi_napi_event.cpp` - 事件处理
- `src/wifi_napi_utils.cpp` - 工具函数
**功能**: 将 JS API 映射到 C++ 实现，处理异步操作、事件订阅

#### `native/`
**职责**: 原生 C++ 实现和 IPC 层
**主要文件**:
- `src/wifi_device.cpp` - 设备代理
- `src/wifi_hotspot.cpp` - 热点代理
- `src/wifi_p2p.cpp` - P2P 代理
- `src/wifi_scan.cpp` - 扫描代理
- `src/wifi_device_callback_stub.cpp` - 设备回调 Stub
- `src/wifi_sa_event.cpp` - SA 事件处理
**功能**: 实现 IPC Proxy/Stub 模式，通过 Binder 与 SA 通信

#### `ets/taihe/wifiManager/`
**职责**: ArkTS WiFi Manager 实现
**主要文件**:
- `src/ohos.wifiManager.impl.cpp`
**功能**: 提供 Taihe 语言的 WiFi Manager API

#### `cj/`
**职责**: Cangjie 语言 FFI（Foreign Function Interface）
**主要文件**:
- `src/wifi_ffi.cpp`
**功能**: 提供 Cangjie 语言的 WiFi 绑定

#### `wifi_ndk/`
**职责**: NDK（Native Development Kit）接口
**主要文件**:
- `include/oh_wifi.h`
- `src/oh_wifi.cpp`
**功能**: 为第三方开发者提供 C 接口访问 WiFi SA

---

### 4. Interfaces Layer（接口层）

#### `c_api/`
**职责**: C API 接口
**主要文件**:
- `include/wifi_device.h` - 设备 API
- `include/wifi_hotspot.h` - 热点 API
- `include/wifi_p2p.h` - P2P API
- `include/wifi_scan_info.h` - 扫描信息结构
- `include/wifi_event.h` - 事件接口
**功能**: 为 C/C++ 应用提供直接调用接口

#### `inner_api/`
**职责**: 内部 API（组件间接口）
**主要文件**:
- `wifi_device.h`
- `wifi_scan.h`
- `wifi_hotspot.h`
- `wifi_p2p.h`
- `wifi_hid2d.h`
**功能**: 定义 WiFi 服务间调用的内部接口

#### `kits/c/`
**职责**: JS Kit 数据结构
**主要文件**:
- `wifi_device_config.h` - 设备配置结构
- `wifi_linked_info.h` - 连接信息结构
- `wifi_p2p_config.h` - P2P 配置结构
- `wifi_hotspot_config.h` - 热点配置结构
- `wifi_error_code.h` - 错误码定义
**功能**: 定义 JS API 与 C++ 实现间的数据结构

---

### 5. Services Layer（服务层）

#### `wifi_standard/wifi_framework/wifi_manage/`

##### `wifi_sta/`（Station 服务）
**职责**: STA（Station）模式管理
**关键子模块**:
- `wifi_sta_sa/` - SA 实现（1120）
  - `wifi_device_mgr_service_impl.cpp` - 设备管理服务实现
- `sta_interface.cpp` - STA 接口
- `sta_service.cpp` - STA 服务
- `sta_state_machine.cpp` - STA 状态机
**核心功能**:
- WiFi 启用/禁用
- 网络扫描
- 网络连接/断开
- 配置管理
- 连接状态监控

##### `wifi_ap/`（AP 服务）
**职责**: AP（Access Point）模式管理
**关键子模块**:
- `wifi_ap_sa/` - SA 实现（1121）
  - `wifi_hotspot_mgr_service_impl.cpp` - 热点管理服务实现
- `ap_interface.cpp` - AP 接口
- `ap_service.cpp` - AP 服务
- `ap_state_machine.cpp` - AP 状态机
**核心功能**:
- 热点启用/禁用
- 站点管理
- 热点配置
- 热点状态监控

##### `wifi_p2p/`（P2P 服务）
**职责**: P2P（Peer-to-Peer）管理
**关键子模块**:
- `wifi_p2p_sa/` - SA 实现（1123）
  - `wifi_p2p_service_impl.cpp` - P2P 服务实现
- `p2p_interface.cpp` - P2P 接口
- `p2p_service.cpp` - P2P 服务
- `p2p_state_machine.cpp` - P2P 状态机
**核心功能**:
- 设备发现
- 组创建
- P2P 连接
- 持久化组管理

##### `wifi_scan/`（扫描服务）
**职责**: WiFi 网络扫描管理
**关键子模块**:
- `wifi_scan_sa/` - SA 实现（1124）
  - `wifi_scan_mgr_service_impl.cpp` - 扫描管理服务实现
- `scan_interface.cpp` - 扫描接口
- `scan_service.cpp` - 扫描服务
- `scan_state_machine.cpp` - 扫描状态机
**核心功能**:
- 发起扫描
- 扫描结果管理
- 扫描状态监控

##### `wifi_native/`
**职责**: 原生接口适配（HAL/HDI 层）
**关键子模块**:
- `client/hdi_client/` - HDI 客户端实现
- `common/` - 通用 HAL 接口
**核心功能**:
- 硬件接口抽象
- HAL 功能调用

##### `wifi_controller/`
**职责**: WiFi 控制器
**主要文件**:
- `wifi_service_scheduler.cpp` - 服务调度器
- `multi_sta_manager.cpp` - 多 STA 管理
- `concrete_manager_state_machine.cpp` - 管理器状态机
**核心功能**:
- 多 STA 实例管理
- 服务生命周期控制

##### `wifi_common/`
**职责**: 通用功能模块
**关键子模块**:
- `wifi_auth_center.cpp` - 认证中心（系统应用检查、令牌验证）
- `wifi_permission_helper.cpp` - 权限验证
- `wifi_permission_utils.cpp` - 权限工具
- `wifi_internal_event_dispatcher.cpp` - 事件分发
- `wifi_protect_manager.cpp` - WiFi 保护管理
- `wifi_config_center.cpp` - 配置中心
- `net_eap/` - EAP 认证支持
- `network_black_list/` - 网络黑名单
- `network_status_history/` - 网络状态历史
**核心功能**:
- 权限验证
- 事件分发
- 配置管理
- 网络策略

##### `wifi_pro/`
**职责**: WiFi Pro 增强功能
**主要文件**:
- `perf_5g/` - 5G 性能优化
- `wifi_intelligence/` - WiFi 智能功能
- `wifi_security_detect/` - 安全检测

##### `wifi_sta_ext/`
**职责**: STA 扩展服务
**主要文件**:
- `wifi_data_report/` - 数据报告
- `wifi_telephony_utils/` - 电话工具

##### `wifi_self_cure/`
**职责**: WiFi 自愈功能
**主要文件**:
- WiFi 自愈逻辑实现

##### `wifi_toolkit/`
**职责**: 工具包
**关键子模块**:
- `config/` - 配置管理工具
- `net_helper/` - 网络辅助工具
- `utils/` - 通用工具
**核心功能**:
- 配置文件解析
- 网络辅助功能
- 工具函数

---

### 6. Relation Services Layer（关联服务）

#### `relation_services/`
**职责**: 关联系统服务集成
**主要文件**:
- `dhcp_service/` - DHCP 服务适配
- `etc/init/` - 服务启动脚本
**核心功能**:
- DHCP 客户端集成
- WPA supplicant 集成
- 服务生命周期管理

---

### 7. Utils Layer（工具层）

#### `extern_library/`
**职责**: 外部库
**主要文件**:
- `src/wifi_intl_util.cpp` - 国际化工具
**功能**: 提供国际化支持的通用工具

#### `utils/`
**职责**: 工具函数库
**主要文件**:
- `inc/` - 头文件
- `src/` - 源文件
**功能**: 提供日志、字符串处理等通用工具

---

## 文件说明

### 构建文件

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根 GN 构建文件 |
| `wifi.gni` | 主配置文件（标准系统） |
| `wifi_lite.gni` | 轻量级配置文件 |
| `bundle.json` | 组件元数据、依赖、能力定义 |

### 配置文件

| 文件 | 说明 |
|------|------|
| `hisysevent.yaml` | HiSysEvent 事件定义 |
| `wifi/services/wifi_standard/etc/init/wifi_standard.cfg` | WiFi 服务启动配置 |
| `wifi/services/wifi_standard/sa_profile/*.json` | SA 配置文件（1120、1121、1123、1124） |
| `wifi/relation_services/etc/init/*.cfg` | 关联服务启动配置 |

---

## 模块间依赖关系

### 依赖层次
```
应用层 (Application)
    ↓
N-API 绑定层 (frameworks/js/napi)
    ↓
Native SDK 层 (frameworks/native)
    ↓
IPC Binder
    ↓
System Ability 层 (services/wifi_standard)
    ↓
HAL/HDI 层 (wifi_native)
    ↓
硬件驱动层
```

### 关键依赖

| 模块 | 依赖 |
|------|------|
| N-API 模块 | Native SDK (wifi_sdk) |
| Native SDK | SA Proxy |
| SA 服务 | WiFi 管理器 |
| WiFi 管理器 | 子服务（STA/AP/P2P/Scan） |
| 子服务 | 工具包和通用功能 |
| WiFi 管理器 | HAL 适配层 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 架构详解
- [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - N-API 接口文档
- [04_Inner_API.md](04_Inner_API.md) - 内部 API 文档
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建系统

---

**下一步**: 阅读 [02_Architecture.md](02_Architecture.md) 了解组件架构设计和数据流。

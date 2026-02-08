# 目录结构与模块职责

## 文档目的

本文档详细说明 DHCP 组件的目录结构及各模块的职责分工。

---

## 适用范围

- 根路径: `/foundation/communication/dhcp`
- 不包含: `test/` 目录（测试相关）

---

## 完整目录树

```
dhcp/
├── BUILD.gn                          # 根构建入口（Lite 版本）
├── bundle.json                       # 组件配置（版本 3.1.0）
├── dhcp.gni                          # 构建变量定义（标准版）
├── dhcp_lite.gni                     # 构建变量定义（Lite 版）
├── LICENSE                           # Apache License 2.0
├── README.md                         # 中文说明
├── README.en.md                      # 英文说明
├── OAT.xml                           # 开放能力声明
├── figures/                          # 文档图片
│   └── zh-cn_image_dhcp.png          # 架构图
├── frameworks/                       # 框架层（SDK）
│   ├── BUILD.gn                      # SDK 构建配置
│   ├── libdhcp_sdk.map               # SDK 导出符号表
│   ├── native/
│   │   ├── c_adapter/                # C 接口适配层
│   │   │   ├── inc/
│   │   │   │   └── dhcp_c_utils.h    # C 工具宏与类型转换
│   │   │   └── src/
│   │   │       ├── dhcp_c_service.cpp # C API 实现（17 个方法）
│   │   │       └── dhcp_c_utils.cpp   # 错误码转换
│   │   ├── include/                  # 公共头文件
│   │   │   ├── dhcp_errcode.h        # C++ 错误码定义
│   │   │   ├── dhcp_event.h          # 回调事件类定义
│   │   │   ├── dhcp_ipc_lite_adapter.h # Lite IPC 适配
│   │   │   ├── dhcp_manager_service_ipc_interface_code.h # IPC 命令码
│   │   │   └── dhcp_sdk_define.h    # SDK 常量定义
│   │   ├── interfaces/               # IPC 接口定义
│   │   │   ├── i_dhcp_client.h       # Client 接口（IRemoteBroker）
│   │   │   ├── i_dhcp_client_callback.h # Client 回调接口
│   │   │   ├── i_dhcp_server.h       # Server 接口
│   │   │   └── i_dhcp_server_callback.h # Server 回调接口
│   │   └── src/                      # SDK 实现代码
│   │       ├── dhcp_client.cpp       # C++ Client SDK 实现
│   │       ├── dhcp_client_impl.cpp  # Client 内部实现
│   │       ├── dhcp_client_proxy.cpp # Client Proxy（发送请求）
│   │       ├── dhcp_client_proxy_lite.cpp # Lite 版 Client Proxy
│   │       ├── dhcp_server.cpp       # C++ Server SDK 实现
│   │       ├── dhcp_server_impl.cpp  # Server 内部实现
│   │       ├── dhcp_server_proxy.cpp # Server Proxy
│   │       ├── dhcp_server_proxy_lite.cpp # Lite 版 Server Proxy
│   │       ├── dhcp_client_callback_stub.cpp # Client 回调 Stub
│   │       ├── dhcp_client_callback_stub_lite.cpp # Lite 版
│   │       ├── dhcp_server_callback_stub.cpp # Server 回调 Stub
│   │       └── dhcp_event.cpp        # 回调事件实现
│   └── interfaces/                   # （空，使用 native/interfaces）
├── interfaces/                       # 对外接口定义
│   ├── inner_api/                    # 内部 C++ API
│   │   ├── dhcp_client.h            # C++ Client 接口
│   │   ├── dhcp_server.h            # C++ Server 接口
│   │   └── include/
│   │       └── dhcp_define.h        # 常量与宏定义
│   └── kits/c/                       # C 语言 API（对外暴露）
│       ├── dhcp_c_api.h             # C API 头文件（17 个导出方法）
│       ├── dhcp_error_code.h        # C API 错误码
│       └── dhcp_result_event.h       # 数据结构与回调定义
├── services/                         # 服务实现层
│   ├── dhcp_client/                  # DHCP 客户端服务（SA 1126）
│   │   ├── BUILD.gn                  # Client 构建配置
│   │   ├── include/
│   │   │   ├── dhcp_client_callback_proxy.h # 回调 Proxy
│   │   │   ├── dhcp_client_death_recipient.h # 死亡通知
│   │   │   ├── dhcp_client_service_impl.h  # Client SA 实现
│   │   │   ├── dhcp_client_stub.h   # IPC Stub（接收请求）
│   │   │   └── dhcp_function.h      # 公共函数声明
│   │   └── src/
│   │       ├── dhcp_client_callback_proxy.cpp
│   │       ├── dhcp_client_death_recipient.cpp
│   │       ├── dhcp_client_service_impl.cpp # SA 主逻辑
│   │       ├── dhcp_client_state_machine.cpp # 客户端状态机
│   │       ├── dhcp_client_stub.cpp
│   │       ├── dhcp_function.cpp    # 公共函数实现
│   │       ├── dhcp_ipv6_client.cpp # IPv6 客户端
│   │       ├── dhcp_ipv6_dns_repository.cpp # IPv6 DNS 仓库
│   │       ├── dhcp_ipv6_event.cpp   # IPv6 事件处理
│   │       ├── dhcp_ipv6_info.cpp    # IPv6 信息管理
│   │       ├── dhcp_options.cpp     # DHCP 选项解析
│   │       ├── dhcp_result.cpp      # 结果数据处理
│   │       ├── dhcp_result_store_manager.cpp # 结果缓存
│   │       └── dhcp_socket.cpp      # Socket 通信
│   ├── dhcp_server/                  # DHCP 服务端（SA 1127）
│   │   ├── BUILD.gn                  # Server 构建配置
│   │   ├── etc/                      # 配置文件目录
│   │   │   └── dhcpd.conf            # dhcpd 配置示例
│   │   ├── include/
│   │   │   ├── address_utils.h       # 地址工具
│   │   │   ├── common_util.h         # 通用工具
│   │   │   ├── dhcp_address_pool.h   # 地址池管理
│   │   │   ├── dhcp_argument.h       # 参数解析
│   │   │   ├── dhcp_binding.h        # 租约绑定
│   │   │   ├── dhcp_config.h         # 配置管理
│   │   │   ├── dhcp_dhcpd.h          # dhcpd 核心逻辑
│   │   │   ├── dhcp_function.h       # 公共函数
│   │   │   ├── dhcp_option.h         # 选项处理
│   │   │   ├── dhcp_s_server.h       # Server 核心
│   │   │   ├── dhcp_server_callback_proxy.h # 回调 Proxy
│   │   │   ├── dhcp_server_death_recipient.h # 死亡通知
│   │   │   ├── dhcp_server_service_impl.h # SA 实现
│   │   │   └── dhcp_server_stub.h    # IPC Stub
│   │   └── src/
│   │       ├── address_utils.cpp
│   │       ├── common_util.cpp
│   │       ├── dhcp_address_pool.cpp
│   │       ├── dhcp_argument.cpp
│   │       ├── dhcp_binding.cpp
│   │       ├── dhcp_config.cpp
│   │       ├── dhcp_dhcpd.cpp
│   │       ├── dhcp_function.cpp
│   │       ├── dhcp_option.cpp
│   │       ├── dhcp_s_server.cpp
│   │       ├── dhcp_server_callback_proxy.cpp
│   │       ├── dhcp_server_death_recipient.cpp
│   │       ├── dhcp_server_service_impl.cpp
│   │       └── dhcp_server_stub.cpp
│   ├── sa_profile/                   # SystemAbility 配置
│   │   ├── BUILD.gn                  # SA 配置构建
│   │   ├── 1126.json                 # DHCP Client SA 配置
│   │   └── 1127.json                 # DHCP Server SA 配置
│   └── utils/                        # 工具库
│       ├── BUILD.gn                  # Utils 构建配置
│       ├── libdhcp_util.map          # Utils 导出符号表
│       ├── include/
│       │   ├── dhcp_arp_checker.h    # ARP 检查工具
│       │   ├── dhcp_common_utils.h  # 通用工具
│       │   ├── dhcp_permission_utils.h # 权限检查
│       │   ├── dhcp_sa_manager.h     # SA 管理工具
│       │   ├── dhcp_system_timer.h   # 定时器
│       │   └── dhcp_thread.h         # 线程池
│       └── src/
│           ├── dhcp_arp_checker.cpp
│           ├── dhcp_common_utils.cpp
│           ├── dhcp_permission_utils.cpp
│           ├── dhcp_sa_manager.cpp
│           ├── dhcp_system_timer.cpp
│           └── dhcp_thread.cpp
└── wiki/                             # 文档（新增）
    ├── README.md
    ├── SUMMARY.md
    ├── 00_Overview.md
    └── ...
```

---

## 模块职责详解

### frameworks/native - SDK 框架层

| 子模块 | 职责 | 关键文件 | 输出产物 |
|--------|------|----------|----------|
| c_adapter | C API 适配层，C→C++ 转换 | dhcp_c_service.cpp | libdhcp_sdk.z.so |
| interfaces | IPC 接口定义（IRemoteBroker） | i_dhcp_client.h | 接口头文件 |
| src | C++ SDK 实现，Proxy/Stub 通信 | dhcp_client_proxy.cpp | SDK 代码 |

**职责说明**:
- 提供对外 C/C++ 接口
- 封装 IPC 通信细节
- 处理错误码转换
- 管理回调机制

### services/dhcp_client - DHCP 客户端服务

| 子模块 | 职责 | 关键文件 | 输出产物 |
|--------|------|----------|----------|
| service_impl | SA 实现，管理 SA 生命周期 | dhcp_client_service_impl.cpp | libdhcp_client.z.so |
| state_machine | DHCP 客户端状态机 | dhcp_client_state_machine.cpp | - |
| ipv6_client | IPv6 地址获取 | dhcp_ipv6_client.cpp | - |
| socket | 网络包收发 | dhcp_socket.cpp | - |
| options | DHCP 选项解析 | dhcp_options.cpp | - |

**职责说明**:
- 实现 DHCP 客户端协议（RFC 2131）
- 管理 DHCPv4/IPv6 地址获取
- 处理 IP 获取成功/失败回调
- WiFi 场景 IP 缓存管理

### services/dhcp_server - DHCP 服务端

| 子模块 | 职责 | 关键文件 | 输出产物 |
|--------|------|----------|----------|
| service_impl | SA 实现 | dhcp_server_service_impl.cpp | libdhcp_server.z.so |
| dhcp_s_server | Server 核心逻辑 | dhcp_s_server.cpp | - |
| address_pool | IP 地址池管理 | dhcp_address_pool.cpp | - |
| binding | 租约绑定管理 | dhcp_binding.cpp | - |
| config | 配置文件解析 | dhcp_config.cpp | - |

**职责说明**:
- 实现 DHCPv4 服务端协议
- 管理地址池和租约表
- 响应客户端 DHCP 请求
- 租约超时与续约处理

### services/utils - 工具库

| 模块 | 职责 | 关键文件 | 输出产物 |
|------|------|----------|----------|
| permission_utils | 权限检查（TokenID + 权限验证） | dhcp_permission_utils.cpp | libdhcp_utils.z.so |
| sa_manager | SystemAbility 获取与管理 | dhcp_sa_manager.cpp | - |
| arp_checker | ARP 检测 IP 冲突 | dhcp_arp_checker.cpp | - |
| thread | 线程池管理 | dhcp_thread.cpp | - |
| system_timer | 定时器 | dhcp_system_timer.cpp | - |
| common_utils | 通用工具（文件操作等） | dhcp_common_utils.cpp | - |

**职责说明**:
- 提供跨模块共享的基础功能
- 权限验证逻辑集中管理
- SA 查找与连接管理

### interfaces - 对外接口层

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| inner_api | 内部 C++ 接口（子系统内部使用） | dhcp_client.h, dhcp_server.h |
| kits/c | C 语言 API（对外暴露给应用） | dhcp_c_api.h |

**职责说明**:
- 定义稳定的接口契约
- 区分内部 API 与公共 API

---

## 目录依赖关系

```
┌─────────────────────────────────────────┐
│        interfaces/kits/c (对外 API)      │
│         dhcp_c_api.h → C 接口           │
└──────────────┬──────────────────────────┘
               │ 调用
               ▼
┌─────────────────────────────────────────┐
│     frameworks/native (SDK 层)           │
│  dhcp_c_service.cpp → C 适配层          │
│  dhcp_client_proxy.cpp → IPC Proxy      │
└──────────────┬──────────────────────────┘
               │ IPC 调用
               ▼
┌─────────────────────────────────────────┐
│   services/dhcp_client (Client SA)       │
│   dhcp_client_service_impl.cpp → 实现   │
│   dhcp_client_stub.cpp → IPC Stub       │
└──────────────┬──────────────────────────┘
               │ 依赖
               ▼
┌─────────────────────────────────────────┐
│      services/utils (工具库)             │
│  dhcp_permission_utils.cpp → 权限       │
│  dhcp_sa_manager.cpp → SA 管理          │
└─────────────────────────────────────────┘
```

---

## 不包含的目录

| 目录 | 说明 | 理由 |
|------|------|------|
| test/ | 单元测试、Fuzz 测试 | 测试代码，不作为业务证据 |
| .git/ | Git 仓库 | 版本控制 |
| figures/ | 文档图片 | 非代码 |

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构详情
- [06_Build_System](06_Build_System.md) - 构建配置

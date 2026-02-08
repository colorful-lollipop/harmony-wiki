# 目录结构与模块职责

## 目的

本文档描述 `telephony_core_service` 的代码组织结构，帮助开发者快速定位功能代码。

---

## 顶层目录

```
/base/telephony/core_service
├── figures/              # 文档图片
├── frameworks/           # 框架层 (JS/C++ FFI)
├── interfaces/           # API 接口定义
├── sa_profile/           # SA 配置文件
├── services/             # 核心服务实现
├── telephonyres/         # 资源文件 (.hap)
├── utils/                # 工具库
├── test/                 # 测试代码 (Wiki 不覆盖)
├── BUILD.gn              # 主构建文件
├── bundle.json           # 组件配置
└── telephony_core_service.gni  # GN 导入文件
```

## frameworks/ - 框架层

### JS N-API 实现

| 目录 | 职责 | 输出产物 |
|------|------|----------|
| `frameworks/js/sim/` | SIM 卡相关 JS API | `sim.z.so` |
| `frameworks/js/network_search/` | 网络搜索相关 JS API | `radio.z.so` |
| `frameworks/js/esim/` | eSIM 相关 JS API | `esim.z.so` |
| `frameworks/js/vcard/` | vCard 联系人导入导出 | `vcard.z.so` |
| `frameworks/js/napi/` | N-API 工具函数 | (无独立产物) |

### ANI (ArkTS Native Interface)

| 目录 | 职责 |
|------|------|
| `frameworks/ets/ani/sim/` | ArkTS SIM API |
| `frameworks/ets/ani/radio/` | ArkTS Radio API |
| `frameworks/ets/ani/esim/` | ArkTS eSIM API |

### C FFI (Cangjie 语言支持)

| 目录 | 职责 |
|------|------|
| `frameworks/cj/telephony_sim/` | Cangjie SIM FFI |
| `frameworks/cj/telephony_radio/` | Cangjie Radio FFI |

### Native 框架

| 目录 | 职责 |
|------|------|
| `frameworks/native/src/` | CoreServiceClient 等客户端实现 |

**关键文件**:
- `core_service_client.cpp:1` - 核心服务客户端
- `core_service_proxy.cpp:1` - 核心服务代理
- `tel_ril_base_parcel.cpp:1` - RIL 数据包基础

## interfaces/ - 接口层

### innerkits/ - 内部 API

| 目录/文件 | 职责 |
|-----------|------|
| `include/i_core_service.h` | 核心服务接口定义 |
| `include/i_sim_manager.h` | SIM 管理器接口 |
| `include/i_network_search.h` | 网络搜索接口 |
| `include/i_tel_ril_manager.h` | RIL 管理器接口 |
| `include/core_service_ipc_interface_code.h` | IPC 命令码定义 (line 22-154) |
| `include/telephony_errors.h` | 错误码定义 (line 23-91) |
| `ims/include/` | IMS 服务接口 |
| `satellite/` | 卫星服务接口 |

### kits/ - 外部 API

| 目录 | 职责 |
|------|------|
| `kits/c/telephony_radio/` | C 语言 Radio API |

## services/ - 服务实现

### core/ - 核心服务

| 文件 | 职责 |
|------|------|
| `core_service.cpp:55` | 服务生命周期管理 (OnStart/OnStop) |
| `core_service_stub.cpp:1` | IPC Stub 实现 |
| `core_service_dump_helper.cpp:1` - Dump 工具 |

### sim/ - SIM 卡服务

| 文件类别 | 职责 |
|----------|------|
| `sim_manager.cpp:1` | SIM 管理器主类 |
| `sim_file_manager.cpp:1` | SIM 文件管理 |
| `sim_state_manager.cpp:1` | SIM 状态管理 |
| `multi_sim_controller.cpp:1` | 多卡控制 |
| `icc_file_controller.cpp:1` | ICC 文件控制 |
| `stk_controller.cpp:1` | STK 菜单控制 |
| `esim_*.cpp` | eSIM 相关实现 (条件编译) |

### network_search/ - 网络搜索

| 文件类别 | 职责 |
|----------|------|
| `network_search_manager.cpp:1` | 网络搜索管理器 |
| `network_search_handler.cpp:1` | 网络搜索事件处理 |
| `network_search_state.cpp:1` | 网络状态管理 |
| `operator_name.cpp:1` | 运营商名称处理 |
| `signal_info.cpp:1` | 信号信息处理 |
| `nitz_update.cpp:1` | NITZ 时间更新 |

### tel_ril/ - RIL 通信

| 文件类别 | 职责 |
|----------|------|
| `tel_ril_manager.cpp:1` | RIL 管理器 |
| `tel_ril_sim.cpp:1` | SIM 相关 RIL 命令 |
| `tel_ril_network.cpp:1` | 网络相关 RIL 命令 |
| `tel_ril_modem.cpp:1` | Modem 相关 RIL 命令 |
| `tel_ril_call.cpp:1` | 通话相关 RIL 命令 |
| `tel_ril_sms.cpp:1` | SMS 相关 RIL 命令 |
| `tel_ril_data.cpp:1` | 数据业务 RIL 命令 |

### ims_service_interaction/ - IMS 交互

| 文件 | 职责 |
|------|------|
| `ims_core_service_client.cpp:1` | IMS 服务客户端 |
| `ims_core_service_proxy.cpp:1` | IMS 服务代理 |

### satellite_service_interaction/ - 卫星交互

| 文件 | 职责 |
|------|------|
| `satellite_service_client.cpp:1` | 卫星服务客户端 |
| `satellite_service_proxy.cpp:1` | 卫星服务代理 |

## utils/ - 工具库

| 目录 | 职责 | 输出产物 |
|------|------|----------|
| `common/` | 通用工具（权限、配置、事件） | `libtel_common.z.so` |
| `log/` | 日志封装 | (配置) |
| `preferences/` | 偏好设置 | - |
| `vcard/` | vCard 编解码 | `libtel_vcard.z.so` |
| `codec/` | ASN.1 编解码 (eSIM) | - |

**关键文件**:
- `telephony_permission.cpp:70` - 权限检查实现
- `telephony_config.cpp:1` - 配置管理

## sa_profile/ - SA 配置

| 文件 | 职责 |
|------|------|
| `4010.json` | CoreService SA 配置 (SA ID: 4010) |

配置内容:
```json
{
    "process": "telephony",
    "systemability": [{
        "name": 4010,
        "libpath": "libtel_core_service.z.so",
        "run-on-create": true
    }]
}
```

## telephonyres/ - 资源

- 运营商配置资源
- 编译为 `telephonyres.hap`

## 相关链接

- [架构设计](./02_Architecture.md)
- [GN 构建](./05_GN_Build.md)

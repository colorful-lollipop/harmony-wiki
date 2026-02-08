# 目录结构

> **目的**: 理解 Bluetooth 模块的代码组织方式与各目录职责  
> **适用范围**: 代码导航、新人定位、功能归属判断

## 顶层目录结构

```
/foundation/communication/bluetooth/
├── interfaces/                    # [对外 API] API 接口定义
│   ├── inner_api/                # 内部系统服务 API (C++ 头文件)
│   │   └── include/
│   │       ├── bluetooth_*.h     # Profile 接口定义
│   │       └── c_header/         # C API 头文件
│   └── c_api/                    # C 原生 API（Mini/Small 系统）
│
├── frameworks/                    # [框架实现] 核心框架代码
│   ├── js/napi/                  # N-API 实现（ArkTS/JS 绑定层）
│   │   ├── src/
│   │   │   ├── ble/              # BLE N-API 模块
│   │   │   ├── a2dp/             # A2DP N-API 模块
│   │   │   ├── hfp/              # HFP N-API 模块
│   │   │   ├── hid/              # HID N-API 模块
│   │   │   ├── connection/       # 连接管理 N-API
│   │   │   ├── access/           # 访问控制 N-API
│   │   │   ├── socket/           # SPP Socket N-API
│   │   │   ├── pan/              # PAN N-API 模块
│   │   │   ├── base_profile/     # Base Profile N-API
│   │   │   ├── constant/         # 常量定义 N-API
│   │   │   ├── common/           # 公共模块
│   │   │   ├── opp/              # OPP N-API 模块
│   │   │   ├── map/              # MAP N-API 模块
│   │   │   ├── pbap/             # PBAP N-API 模块
│   │   │   ├── audio_manager/    # Audio Manager N-API
│   │   │   └── native_module.cpp # 主入口模块
│   │   └── include/              # N-API 公共头文件
│   │
│   ├── c_api/                    # C API 实现（对外）
│   │
│   ├── cj/                       # FFI 接口（ArkTS/TS）
│   │   ├── ble/
│   │   ├── a2dp/
│   │   ├── hfp/
│   │   ├── hid/
│   │   ├── access/
│   │   ├── connection/
│   │   └── socket/
│   │
│   ├── ets/taihe/                # ETS Taihe 框架（ArkTS）
│   │   ├── bluetooth_ble/
│   │   ├── bluetooth_a2dp/
│   │   ├── bluetooth_hfp/
│   │   ├── bluetooth_hid/
│   │   ├── bluetooth_connection/
│   │   ├── bluetooth_access/
│   │   └── common/
│   │
│   └── inner/                    # [核心框架] 内部实现
│       ├── include/              # 内部头文件
│       ├── src/                  # Profile 实现
│       │   ├── bluetooth_host.cpp        # 蓝牙主机管理
│       │   ├── bluetooth_ble_*.cpp      # BLE Profile 实现
│       │   ├── bluetooth_a2dp_*.cpp     # A2DP Profile 实现
│       │   └── ...
│       ├── ipc/                  # IPC 框架
│       │   ├── interface/        # IPC 接口定义 (IRemoteBroker)
│       │   ├── include/          # Proxy/Stub 头文件
│       │   └── src/              # Proxy/Stub 实现
│       └── c_adapter/             # C API 适配层
│
├── test/                         # [测试目录 - 不纳入 Wiki]
│   ├── unittest/                 # 单元测试
│   └── ...
│
├── sa_profile/                   # SA Profile 配置（服务注册）
├── services/                     # [不存在] 服务实现（在独立仓库）
├── bluetooth.gni                 # GN 构建配置
├── bundle.json                   # 模块配置
└── LICENSE                       # 许可证
```

## 目录职责说明

### `interfaces/` - 对外 API 层

**职责**: 定义上层应用调用的 API 接口

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `inner_api/include/` | 内部系统服务 API（C++ 头文件） | `bluetooth_host.h`, `bluetooth_gatt_*.h` |
| `inner_api/include/c_header/` | C API 头文件 | `ohos_bt_*.h` |
| `c_api/` | C 原生 API 实现（Mini/Small 系统） | - |

**稳定性标注**:
- `inner_api/` → **不稳定**（仅限系统应用内部使用）
- `c_api/` → **稳定**（对外暴露，可被第三方应用调用）

### `frameworks/js/napi/` - N-API 绑定层

**职责**: 将 C++ API 暴露给 ArkTS/JS 应用层

| 子目录 | 职责 | 关键注册点 |
|--------|------|-----------|
| `src/ble/` | BLE 相关 API | `native_module_ble.cpp:70` |
| `src/a2dp/` | A2DP 音频 Profile | `native_module_a2dp.cpp:60` |
| `src/hfp/` | HFP 免提 Profile | `native_module_hfp.cpp:65` |
| `src/hid/` | HID 人体学设备 | `native_module_hid.cpp:65` |
| `src/connection/` | 设备连接管理 | `native_module_connection.cpp:60` |
| `src/access/` | 蓝牙访问控制 | `native_module_access.cpp:63` |
| `src/socket/` | SPP 串口通信 | `native_module_socket.cpp:60` |
| `src/pan/` | PAN 网络共享 | `native_module_pan.cpp:62` |
| `src/base_profile/` | Base Profile 抽象 | `native_module_base_profile.cpp:61` |
| `src/constant/` | 常量定义 | `native_module_constant.cpp:61` |
| `src/common/` | 公共模块 | `module_common.cpp:61` |
| `src/native_module.cpp` | **主入口模块** | `napi_module_register()` |

**N-API 模块注册模式**:
```cpp
// 典型结构（以 ble 为例）
native_module_ble.cpp
├── Init() 函数          // 导出 JS 方法
├── napi_module 结构      // 模块元数据
└── RegisterModule()      // 自动注册（constructor 属性）
```

### `frameworks/inner/` - 核心框架层

**职责**: 实现蓝牙 Profile 管理与 IPC 通信

| 子目录 | 职责 | 关键类 |
|--------|------|--------|
| `src/` | 各 Profile 的核心实现 | `BluetoothHost`, `GattServer`, `A2dpSrc` |
| `ipc/interface/` | IPC 接口定义（IRemoteBroker） | `IBluetoothHost`, `IBluetoothGatt*` |
| `ipc/include/` | Proxy/Stub 头文件 | `bluetooth_host_proxy.h` |
| `ipc/src/` | Proxy/Stub 实现 | `bluetooth_host_proxy.cpp` |
| `c_adapter/` | C API 适配层 | `ohos_bt_*.cpp` |
| `include/` | 内部工具头文件 | `bluetooth_log.h`, `bluetooth_observer_list.h` |

**IPC 架构关键文件**:
```
ipc/interface/
├── i_bluetooth_host.h              # IBluetoothHost 接口
├── i_bluetooth_gatt_client.h       # GATT 客户端接口
├── i_bluetooth_gatt_server.h      # GATT 服务端接口
├── i_bluetooth_ble_*.h            # BLE 相关接口
├── i_bluetooth_a2dp_*.h           # A2DP 相关接口
├── i_bluetooth_hfp_*.h            # HFP 相关接口
├── ... (40+ 接口文件)
└── bluetooth_service_ipc_interface_code.h  # SAID 1130 定义

ipc/src/
├── bluetooth_host_proxy.cpp        # 客户端代理实现
├── bluetooth_host_stub.cpp         # 服务端存根实现
├── bluetooth_gatt_*_proxy.cpp     # 各 Profile 代理
├── bluetooth_gatt_*_stub.cpp      # 各 Profile 存根
└── ... (40+ 实现文件)
```

### `frameworks/cj/` - FFI 接口层

**职责**: 为 ArkTS 提供 FFI（Foreign Function Interface）绑定

| 子目录 | 职责 |
|--------|------|
| `ble/` | BLE FFI 实现 |
| `a2dp/` | A2DP FFI 实现 |
| `hfp/` | HFP FFI 实现 |
| `hid/` | HID FFI 实现 |
| `access/` | 访问控制 FFI |
| `connection/` | 连接管理 FFI |
| `socket/` | Socket FFI |

### `frameworks/ets/taihe/` - ETS Taihe 框架

**职责**: 为 ArkTS 提供声明式 API 封装

| 子目录 | 职责 |
|--------|------|
| `bluetooth_ble/` | BLE ETS 组件 |
| `bluetooth_a2dp/` | A2DP ETS 组件 |
| `bluetooth_hfp/` | HFP ETS 组件 |
| `bluetooth_hid/` | HID ETS 组件 |
| `bluetooth_connection/` | 连接管理 ETS |
| `bluetooth_access/` | 访问控制 ETS |
| `common/` | ETS 公共工具 |

## 模块与目录映射表

| Profile/能力 | N-API | Inner API | IPC 接口 |
|--------------|-------|-----------|----------|
| BLE 广播 | `js/napi/src/ble/` | `inner/src/bluetooth_ble_advertiser.cpp` | `i_bluetooth_ble_advertiser.h` |
| BLE 扫描 | `js/napi/src/ble/` | `inner/src/bluetooth_ble_central_manager.cpp` | `i_bluetooth_ble_central_manager.h` |
| GATT 客户端 | `js/napi/src/ble/` | `inner/src/bluetooth_gatt_client.cpp` | `i_bluetooth_gatt_client.h` |
| GATT 服务端 | `js/napi/src/ble/` | `inner/src/bluetooth_gatt_server.cpp` | `i_bluetooth_gatt_server.h` |
| A2DP 源端 | `js/napi/src/a2dp/` | `inner/src/bluetooth_a2dp_src.cpp` | `i_bluetooth_a2dp_src.h` |
| A2DP 接收端 | `js/napi/src/a2dp/` | `inner/src/bluetooth_a2dp_snk.cpp` | `i_bluetooth_a2dp_sink.h` |
| HFP 免提 | `js/napi/src/hfp/` | `inner/src/bluetooth_hfp_hf.cpp` | `i_bluetooth_hfp_hf.h` |
| HFP 网关 | `js/napi/src/hfp/` | `inner/src/bluetooth_hfp_ag.cpp` | `i_bluetooth_hfp_ag.h` |
| HID 主机 | `js/napi/src/hid/` | `inner/src/bluetooth_hid_host.cpp` | `i_bluetooth_hid_host.h` |
| HID 设备 | `js/napi/src/hid/` | `inner/src/bluetooth_hid_device.cpp` | `i_bluetooth_hid_device.h` |
| AVRCP 控制 | `js/napi/src/a2dp/` | `inner/src/bluetooth_avrcp_ct.cpp` | `i_bluetooth_avrcp_ct.h` |
| AVRCP 目标 | `js/napi/src/a2dp/` | `inner/src/bluetooth_avrcp_tg.cpp` | `i_bluetooth_avrcp_tg.h` |
| PAN | `js/napi/src/pan/` | `inner/src/bluetooth_pan.cpp` | `i_bluetooth_pan.h` |
| OPP | `js/napi/src/opp/` | `inner/src/bluetooth_opp.cpp` | `i_bluetooth_opp.h` |
| PBAP | `js/napi/src/pbap/` | `inner/src/bluetooth_pbap_pse.cpp` | `i_bluetooth_pbap_pse.h` |
| MAP | `js/napi/src/map/` | `inner/src/bluetooth_map_mse.cpp` | `i_bluetooth_map_mse.h` |
| SPP Socket | `js/napi/src/socket/` | `inner/src/bluetooth_socket.cpp` | `i_bluetooth_socket.h` |
| 主机管理 | `js/napi/src/native_module.cpp` | `inner/src/bluetooth_host.cpp` | `i_bluetooth_host.h` |

## 忽略的目录

以下目录在代码分析时忽略（不纳入 Wiki 文档）：

| 目录/模式 | 原因 |
|-----------|------|
| `test/` | 测试代码，仅供开发调试 |
| `*_test.*` | 测试文件 |
| `unittest/` | 单元测试 |
| `fuzz/` | 模糊测试 |
| `.git/` | Git 版本控制 |
| `out/` | 构建输出目录 |

---

**下一步**: [架构设计](02_Architecture.md) → 深入理解组件交互与数据流

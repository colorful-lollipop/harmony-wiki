# 目录结构

> connectivity_cangjie_wrapper 项目目录布局与模块职责说明

## 整体目录树

```
foundation/communication/connectivity_cangjie_wrapper/
├── figures/                                    # 架构图片目录
│   └── connectivity_cangjie_wrapper_architecture_en.png
│
├── kit/                                        # Kit 层代码
│   └── ConnectivityKit/                        # ConnectivityKit 统一出口
│       ├── index.cj                            # 模块入口，导出所有 API
│       └── BUILD.gn                            # 构建配置
│
├── ohos/                                       # Cangjie API 实现
│   ├── bluetooth/                              # 蓝牙模块
│   │   ├── a2dp/                               # A2DP Profile 接口
│   │   │   ├── a2dp.cj                         # A2DP API 实现
│   │   │   ├── native.cj                      # Native 绑定
│   │   │   └── BUILD.gn
│   │   ├── base_profile/                      # 基础 Profile 框架
│   │   │   ├── base_profile.cj
│   │   │   └── BUILD.gn
│   │   ├── ble/                               # BLE 接口（核心）
│   │   │   ├── ble.cj                         # BLE 扫描/广播 API
│   │   │   ├── gatt_client_device.cj         # GATT 客户端
│   │   │   ├── gatt_server.cj                # GATT 服务端
│   │   │   ├── gatt_util.cj                  # GATT 工具函数
│   │   │   ├── native.cj                     # Native 绑定
│   │   │   ├── util.cj                       # 工具函数
│   │   │   └── BUILD.gn
│   │   ├── connection/                       # 连接管理
│   │   │   ├── connection.cj
│   │   │   ├── native.cj
│   │   │   └── BUILD.gn
│   │   ├── constant/                         # 蓝牙常量
│   │   │   ├── constant.cj                   # 枚举定义
│   │   │   └── BUILD.gn
│   │   ├── hfp/                              # HFP Profile 接口
│   │   │   └── hands_free_audio_gateway_profile.cj
│   │   │       └── BUILD.gn
│   │   ├── callback_controller.cj            # 回调控制器
│   │   ├── error_message.cj                  # 错误消息
│   │   └── BUILD.gn                          # 蓝牙根 BUILD.gn
│   │
│   └── wifi_manager/                          # WLAN 模块
│       ├── common.cj                         # 公共函数
│       ├── error_code.cj                     # 错误码定义
│       ├── ip_info.cj                        # IP 信息
│       ├── wifi_info_elem.cj                 # WiFi 信息元素
│       ├── wifi_p2p_config.cj                # P2P 配置
│       ├── wifi_scan_info.cj                 # 扫描信息
│       ├── wifi.cj                           # WiFi/P2P API
│       └── BUILD.gn
│
├── mock/                                       # Mock 代码（跨平台支持）
│   ├── ohos.bluetooth.cj
│   ├── ohos.bluetooth.ble.cj
│   ├── ohos.bluetooth.a2dp.cj
│   ├── ohos.bluetooth.base_profile.cj
│   ├── ohos.bluetooth.connection.cj
│   ├── ohos.bluetooth.constant.cj
│   ├── ohos.bluetooth.hfp.cj
│   └── ohos.wifi_manager.cj
│
├── test/                                       # 测试代码
│   ├── bluetooth/
│   │   └── test/                             # 蓝牙测试项目
│   └── wifi_manager/
│       └── test/                             # WiFi 测试项目
│
└── wiki/                                       # 本 Wiki 文档
    ├── _work/                                # 工作目录
    │   ├── NOTES.md                          # 事实记录
    │   └── PLAN.md                           # 任务计划
    ├── README.md                             # Wiki 首页
    ├── SUMMARY.md                            # 全站导航
    ├── index.md                              # 项目首页
    ├── 01_Project_Overview.md               # 项目概览
    ├── 02_Directory_Structure.md            # 本文档
    ├── 03_Architecture.md                   # 架构说明
    ├── 04_N-API_Reference.md                # N-API 参考
    ├── 05_Inner_API.md                      # 内部 API
    ├── 06_Build_System.md                   # 构建系统
    ├── 07_Security_Review.md                # 安全评审
    ├── 08_FAQ_Troubleshooting.md            # FAQ 与排错
    └── appendix/
        └── Callgraphs.md                    # 关键调用链
```

## 模块职责说明

### kit/ConnectivityKit

| 文件 | 职责 |
|------|------|
| `index.cj` | 统一导出所有蓝牙和 WLAN API，作为 Kit 层出口 |

### ohos/bluetooth/

| 子模块 | 职责 |
|--------|------|
| `ble/` | BLE 核心功能：扫描、广播、GATT 服务/客户端 |
| `a2dp/` | A2DP Profile：音频流传输控制 |
| `hfp/` | HFP Profile：免提通话控制 |
| `base_profile/` | 基础 Profile 框架和接口 |
| `connection/` | 蓝牙设备连接管理 |
| `constant/` | 枚举常量定义（连接状态、Profile 类型等） |

### ohos/wifi_manager/

| 文件 | 职责 |
|------|------|
| `wifi.cj` | P2P 连接、扫描、状态监听 |
| `wifi_p2p_config.cj` | P2P 连接配置 |
| `wifi_scan_info.cj` | 扫描结果信息 |
| `ip_info.cj` | IP 信息（预留） |
| `error_code.cj` | WiFi 错误码定义 |

### mock/

Mock 代码用于在 **Windows (mingw)** 和 **macOS** 平台上提供桩实现，确保代码可以在这些平台上编译通过。

```gn
# BUILD.gn 中的跨平台判断
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.bluetooth.ble.cj" ]
} else {
    sources = [ "ble.cj", "gatt_client_device.cj", ... ]
}
```

## 文件命名规范

| 模式 | 说明 |
|------|------|
| `*.cj` | Cangjie 源代码文件 |
| `BUILD.gn` | GN 构建配置文件 |
| `native.cj` | Native 绑定/FFI 接口文件 |
| `util.cj` / `*_util.cj` | 工具函数文件 |
| `*.md` | Markdown 文档 |

## 代码组织原则

1. **按功能分模块**: 每个功能模块独立目录
2. **按类型分层**: kit → ohos → 具体模块
3. **关注点分离**: API 实现与 Native 绑定分离
4. **跨平台兼容**: mock 目录提供桩实现
5. **测试隔离**: 测试代码与业务代码分离

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Project_Overview.md) | 项目定位与核心能力 |
| [架构说明](./03_Architecture.md) | 组件关系与数据流 |
| [N-API 参考](./04_N-API_Reference.md) | API 详细说明 |
| [构建系统](./06_Build_System.md) | GN Targets 与产物 |

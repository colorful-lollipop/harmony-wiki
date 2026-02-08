# 代码地图

> 快速定位 RIL Adapter 核心代码文件

## 1. 目录结构总览

```
base/telephony/ril_adapter/
├── interfaces/                    # 内部接口定义层 (API 契约)
│   └── innerkits/
│       └── include/               # 12 个头文件
│
├── services/                     # 服务实现层
│   ├── hril/                      # HRIL 业务模块 (C++)
│   │   ├── include/              # 11 个头文件
│   │   └── src/                  # 10 个实现文件
│   │
│   ├── hril_hdf/                  # HDF 服务适配 (C)
│   │   ├── include/              # 2 个头文件
│   │   └── src/                  # 1 个实现文件
│   │
│   └── vendor/                    # 厂商抽象层 (C)
│       ├── include/              # 11 个头文件
│       └── src/                  # 11 个实现文件
│
└── utils/native/                  # 工具类
    └── include/                  # 2 个头文件
```

## 2. 核心文件定位

### 2.1 入口与初始化

| 功能 | 文件 | 重要性 |
|------|------|--------|
| **HDF 服务入口** | `services/hril_hdf/src/hril_hdf.c` | ⭐⭐⭐ |
| **HDF 接口定义** | `services/hril_hdf/include/hril_hdf.h` | ⭐⭐⭐ |
| **Modem 适配** | `services/hril_hdf/include/modem_adapter.h` | ⭐⭐ |
| **Vendor 库加载** | `services/vendor/src/vendor_adapter.c` | ⭐⭐⭐ |

### 2.2 核心管理模块

| 功能 | 头文件 | 实现文件 | 规模 |
|------|--------|----------|------|
| **管理器** | `services/hril/include/hril_manager.h` | `services/hril/src/hril_manager.cpp` | 49KB |
| **基类** | `services/hril/include/hril_base.h` | `services/hril/src/hril_base.cpp` | 5.5KB |
| **事件循环** | `services/hril/include/hril_event.h` | `services/hril/src/hril_event.cpp` | 8KB |
| **定时器** | `services/hril/include/hril_timer_callback.h` | `services/hril/src/hril_timer_callback.cpp` | 4KB |

### 2.3 业务模块

| 模块 | 头文件 | 实现文件 | 代码规模 | 核心职责 |
|------|--------|----------|----------|----------|
| **通话** | `hril_call.h` | `hril_call.cpp` | 45KB | 拨号/接听/挂断/保持 |
| **数据** | `hril_data.h` | `hril_data.cpp` | 35KB | PDP/数据连接 |
| **网络** | `hril_network.h` | `hril_network.cpp` | 79KB | 信号/注册/邻区 |
| **SIM** | `hril_sim.h` | `hril_sim.cpp` | 44KB | PIN/APDU/STK |
| **短信** | `hril_sms.h` | `hril_sms.cpp` | 41KB | 短信/广播 |
| **Modem** | `hril_modem.h` | `hril_modem.cpp` | 17KB | 射频/IMEI |

### 2.4 Vendor 抽象层

| 模块 | 头文件 | 实现文件 | 核心职责 |
|------|--------|----------|----------|
| **厂商适配** | `vendor_adapter.h` | `vendor_adapter.c` | 库加载/状态管理 |
| **AT 支持** | `at_support.h` | `at_support.c` | AT 命令工具 |
| **AT 通话** | `at_call.h` | `at_call.c` | 通话 AT 命令 |
| **AT 数据** | `at_data.h` | `at_data.c` | 数据 AT 命令 |
| **AT 网络** | `at_network.h` | `at_network.c` | 网络 AT 命令 |
| **AT SIM** | `at_sim.h` | `at_sim.c` | SIM AT 命令 |
| **AT 短信** | `at_sms.h` | `at_sms.c` | 短信 AT 命令 |
| **AT Modem** | `at_modem.h` | `at_modem.c` | Modem AT 命令 |
| **报告处理** | `vendor_report.h` | `vendor_report.c` | 响应解析 |
| **通道管理** | `vendor_channel.h` | `vendor_channel.c` | AT 通道 |
| **工具类** | `vendor_util.h` | `vendor_util.c` | 通用函数 |

### 2.5 接口定义层

| 文件 | 描述 | 关键内容 |
|------|------|----------|
| `hril.h` | 主 C API | `RilInitOps`, `HRilReport` |
| `hril_enum.h` | 枚举定义 | 错误码、状态枚举 |
| `hril_types.h` | 类型定义 | 基本类型、常量 |
| `hril_request.h` | 请求 ID | `HREQ_*` 宏定义 |
| `hril_notification.h` | 通知 ID | `HNOTI_*` 宏定义 |
| `hril_public_struct.h` | 公共结构体 | `ReqDataInfo`, `ReportInfo` |
| `hril_vendor_*.h` | 厂商请求结构 | 各模块的结构体定义 |

## 3. 代码导航图

### 3.1 查找功能对应的文件

| 功能需求 | 定位文件 |
|----------|----------|
| **通话功能实现** | `services/hril/src/hril_call.cpp` |
| **数据连接管理** | `services/hril/src/hril_data.cpp` |
| **网络注册查询** | `services/hril/src/hril_network.cpp` |
| **SIM 卡操作** | `services/hril/src/hril_sim.cpp` |
| **短信收发** | `services/hril/src/hril_sms.cpp` |
| **Modem 控制** | `services/hril/src/hril_modem.cpp` |
| **AT 命令发送** | `services/vendor/src/at_*.c` |
| **厂商库加载** | `services/vendor/src/vendor_adapter.c` |
| **事件循环** | `services/hril/src/hril_event.cpp` |
| **定时器管理** | `services/hril/src/hril_timer_callback.cpp` |

### 3.2 查找类型定义

| 类型 | 头文件 |
|------|--------|
| **错误码** | `interfaces/innerkits/include/hril_enum.h` |
| **请求参数** | `interfaces/innerkits/include/hril_vendor_*.h` |
| **响应结构** | `interfaces/innerkits/include/hril_public_struct.h` |
| **回调函数指针** | `interfaces/innerkits/include/hril.h` |

### 3.3 查找宏定义

| 宏类型 | 定义位置 |
|--------|----------|
| **请求 ID** | `interfaces/innerkits/include/hril_request.h` |
| **通知 ID** | `interfaces/innerkits/include/hril_notification.h` |
| **日志宏** | `utils/native/include/telephony_log_wrapper.h` |

## 4. 关键符号速查

### 4.1 全局变量/函数

| 符号 | 类型 | 文件 | 描述 |
|------|------|------|------|
| `HRilInit()` | 函数 | `hril_hdf.c` | RIL 初始化 |
| `InitRilAdapter()` | 函数 | `hril_hdf.c` | 适配器初始化 |
| `HRilRegOps()` | 函数 | `hril_hdf.c` | 注册厂商操作 |
| `ReleaseRilAdapter()` | 函数 | `hril_hdf.c` | 释放适配器 |
| `HRilManager::GetInstance()` | 函数 | `hril_manager.cpp` | 获取单例 |

### 4.2 关键类

| 类名 | 基类 | 头文件 | 主要职责 |
|------|------|--------|----------|
| `HRilManager` | `IHRilReporter` | `hril_manager.h` | 单例管理、请求路由 |
| `HRilBase` | - | `hril_base.h` | 基类、模板方法 |
| `HRilCall` | `HRilBase` | `hril_call.h` | 通话业务 |
| `HRilData` | `HRilBase` | `hril_data.h` | 数据业务 |
| `HRilNetwork` | `HRilBase` | `hril_network.h` | 网络业务 |
| `HRilSim` | `HRilBase` | `hril_sim.h` | SIM 业务 |
| `HRilSms` | `HRilBase` | `hril_sms.h` | 短信业务 |
| `HRilModem` | `HRilBase` | `hril_modem.h` | Modem 业务 |

### 4.3 关键回调

| 回调名 | 头文件 | 触发时机 |
|--------|--------|----------|
| `OnCallReport()` | `hril_hdf.h` | 通话事件发生 |
| `OnDataReport()` | `hril_hdf.h` | 数据事件发生 |
| `OnNetworkReport()` | `hril_hdf.h` | 网络事件发生 |
| `OnSimReport()` | `hril_hdf.h` | SIM 事件发生 |
| `OnSmsReport()` | `hril_hdf.h` | 短信事件发生 |
| `OnModemReport()` | `hril_hdf.h` | Modem 事件发生 |

## 5. 文件规模统计

### 5.1 按语言

| 语言 | 文件数 | 总代码量 |
|------|--------|----------|
| **C++** | 10 | ~260KB |
| **C** | 12 | ~350KB |
| **H** | ~40 | - |

### 5.2 按模块

| 模块 | 文件数 | 代码量 |
|------|--------|--------|
| HRIL 业务 | 20 | ~260KB |
| Vendor 抽象 | 22 | ~350KB |
| HDF 适配 | 3 | - |
| 接口定义 | 14 | - |

## 6. 排除目录

以下目录不纳入文档和代码分析范围：

| 目录 | 说明 |
|------|------|
| `test/` | 测试代码 |
| `figures/` | 图片资源 |
| `wiki/` | 本文档目录 |

## 7. 小结

代码地图的核心价值在于快速定位：

1. **业务逻辑** → `services/hril/src/hril_*.cpp`
2. **厂商适配** → `services/vendor/src/at_*.c`
3. **接口定义** → `interfaces/innerkits/include/`
4. **HDF 集成** → `services/hril_hdf/`

---

**相关文档**：
- [架构与数据流](02_Architecture.md) - 理解代码组织逻辑
- [接口文档](04_Interface.md) - 详细 API 参考
- [攻击面分析](05_AttackSurface.md) - 安全研究入口

# OpenHarmony Print Scan Framework - 目录结构

**目的**: 了解代码组织方式和各模块职责

**适用范围**: OpenHarmony Print Scan Framework 3.1

---

## 1. 顶层目录结构（不含测试）

```
print_print_fwk/
├── etc/                       # 系统配置
│   ├── init/                   # 初始化配置（rc 文件）
│   └── param/                  # 参数配置（para 文件）
├── figures/                    # 架构图和文档图片
├── frameworks/                 # 框架层实现
│   ├── ISaneBackends/          # SANE 后端接口定义
│   ├── ani_util/               # 动画工具
│   ├── helper/                 # 辅助工具（print_helper, scan_helper）
│   ├── innerkitsimpl/          # 内部实现（print_client, scan_client）
│   ├── kits/                   # 扩展框架（print_extension）
│   ├── models/                 # 数据模型（print_models, scan_models）
│   ├── ohprint/                # 打印接口定义
│   └── ohscan/                 # 扫描接口定义
├── interfaces/                 # 接口层定义
│   └── kits/                   # 开发接口
│       ├── ani/                   # ANI 接口
│       ├── jsnapi/                # JS N-API 扩展接口
│       │   ├── print_extension/     # 打印扩展能力 N-API
│       │   └── print_extensionctx/ # 打印扩展上下文 N-API
│       ├── ndk/                   # NDK 接口
│       │   ├── ohprint/            # 打印 NDK 接口
│       │   └── ohscan/             # 扫描 NDK 接口
│       └── napi/                  # N-API 接口
│           ├── print_napi/          # 打印 N-API 模块
│           └── scan_napi/           # 扫描 N-API 模块
├── profile/                    # Profile 配置文件
├── services/                   # 服务层实现
│   ├── print_service/           # 打印服务
│   ├── scan_service/            # 扫描服务
│   └── sane_service/            # SANE 服务
└── utils/                      # 工具库
    ├── include/                 # 公共头文件
    └── ...
```

---

## 2. 目录职责说明

### 2.1 接口层 (interfaces/)

**职责**: 定义面向外部的接口，包括 N-API（JS 接口）和 NDK（Native 接口）

#### interfaces/kits/napi/

| 模块 | 职责 | 主要内容 |
|--------|------|---------|
| **print_napi** | 打印 N-API 模块 | JS 打印 API、枚举常量 |
| **scan_napi** | 扫描 N-API 模块 | JS 扫描 API、枚举常量 |

#### interfaces/kits/jsnapi/

| 模块 | 职责 | 主要内容 |
|--------|------|---------|
| **print_extension** | 打印扩展能力 N-API | 扩展应用 JS 接口 |
| **print_extensionctx** | 打印扩展上下文 N-API | 扩展上下文 JS 接口 |

#### interfaces/kits/ndk/

| 模块 | 职责 | 主要内容 |
|--------|------|---------|
| **ohprint** | 打印 NDK 接口 | 打印 C++ 接口定义 |
| **ohscan** | 扫描 NDK 接口 | 扫描 C++ 接口 定义 |

---

### 2.2 框架层 (frameworks/)

**职责**: 框架核心实现，包括模型、辅助工具、内部实现和接口定义

#### frameworks/models/

| 模块 | 职责 |
|--------|------|
| **print_models** | 打印数据模型 | PrinterInfo, PrintJob, PrinterCapability 等 |
| **scan_models** | 扫描数据模型 | ScanDeviceInfo, ScanParameters, ScanOptionDescriptor 等 |

#### frameworks/helper/

| 模块 | 职责 |
|--------|------|
| **print_helper** | 打印辅助工具 | 打印相关工具函数 |
| **scan_helper** | 扫描辅助工具 | 扫描相关工具函数 |

#### frameworks/innerkitsimpl/

| 模块 | 职责 |
|--------|------|
| **print_impl** | 打印内部实现 | PrintServiceProxy, PrintCallbackStub（IPC 客户端/服务端） |
| **scan_impl** | 扫描内部实现 | ScanServiceProxy, ScanCallbackStub（IPC 客户端/服务端） |

#### frameworks/ohprint/ & frameworks/ohscan/

| 模块 | 职责 |
|--------|------|
| **ohprint** | 打印接口 | 打印相关常量、数据结构定义 |
| **ohscan** | 扫描接口 | 扫描相关常量、数据结构定义 |

#### frameworks/kits/ & frameworks/ISaneBackends/

| 模块 | 职责 |
|--------|------|
| **extension** | 打印扩展框架 | 扩展基础类和接口 |
| **ISaneBackends** | SANE 后端接口 | ISaneBackends 接口定义 |

---

### 2.3 服务层 (services/)

**职责**: 系统服务实现，作为 SA（System Ability）注册到系统中

#### services/print_service/

| 组件 | 职责 |
|--------|------|
| **PrintServiceAbility** | 打印系统服务 | 实现 `PrintServiceStub`，提供打印 IPC 服务 |
| **CUPS 客户端** | 打印后端集成 | 与 CUPS 打印系统交互 |
| **Vendor 管理器** | 厂商驱动管理 | 管理多种打印驱动（BSUNI, PPD, IPP Everywhere, SMB） |
| **事件系统** | 事件发布订阅 | 处理打印机事件和任务事件 |

#### services/scan_service/

| 组件 | 职责 |
|--------|------|
| **ScanServiceAbility** | 扫描系统服务 | 实现 `ScanServiceStub`，提供扫描 IPC 服务 |
| **SANE 管理器** | SANE 服务管理 | 管理 SANE 后端和扫描任务 |
| **USB 扫描器管理** | USB 设备发现 | 发现和管理 USB 扫描器 |
| **mDNS 服务** | 网络扫描器发现 | 通过 mDNS 发现网络扫描器 |

#### services/sane_service/

| 组件 | 职责 |
|--------|------|
| **SaneServerManager** | SANE 系统服务 | 实现 `SaneBackendsStub`，提供 SANE 后端服务 |
| **ESCL 驱动管理** | ESCL 驱动管理 | 管理 ESCL 扫描器驱动 |

---

### 2.4 配置和工具 (etc/, profile/, utils/)

#### etc/

| 目录 | 职责 |
|--------|------|
| **init/** | 系统初始化配置 | printservice.rc, scanservice.rc, saneservice.rc 等 |
| **param/** | 系统参数配置 | print.para, print.para.dac 等 |

#### profile/

| 目录 | 职责 |
|--------|------|
| **profile/** | SA Profile 配置 | print_sa_profiles.xml, scan_sa_profiles.xml, sane_sa_profiles.xml |

#### utils/

| 目录 | 职责 |
|--------|------|
| **include/** | 公共头文件 | print_constant.h, scan_constant.h 等常量定义 |

---

## 3. 模块依赖关系

### 3.1 依赖层次

```
应用层 (N-API)
    ↓
接口层 (interfaces/kits)
    ↓
内部实现层 (frameworks/innerkitsimpl)
    ↓
服务层 (services)
    ↓
后端层 (CUPS, SANE, Vendor Drivers)
```

### 3.2 数据流向

**打印流程**:
```
JS 应用
  → N-API (print_napi)
  → PrintServiceProxy (IPC 客户端)
  → PrintServiceAbility (SA 服务端)
  → PrintServiceStub (IPC 服务端)
  → PrintCallback (事件回调)
  → CUPS / Vendor Driver
  → 打印机
```

**扫描流程**:
```
JS 应用
  → N-API (scan_napi)
  → ScanServiceProxy (IPC 客户端)
  → ScanServiceAbility (SA 服务端)
  → ScanServiceStub (IPC 服务端)
  → ScanCallback (事件回调)
  → SaneServerManager (SANE 服务)
  → SANE Backend
  → 扫描器
```

---

## 4. 重要文件索引

### 4.1 N-API 注册文件

| 文件 | 用途 |
|------|------|
| `interfaces/kits/napi/print_napi/src/print_module.cpp` | 打印 N-API 模块注册 |
| `interfaces/kits/napi/scan_napi/src/scan_module.cpp` | 扫描 N-API 模块注册 |
| `interfaces/kits/jsnapi/print_extension/print_extension_module.cpp` | 打印扩展 N-API 模块注册 |

### 4.2 服务能力文件

| 文件 | 用途 |
|------|------|
| `services/print_service/include/print_service_ability.h` | 打印服务接口定义 |
| `services/scan_service/include/scan_service_ability.h` | 扫描服务接口定义 |
| `services/sane_service/include/sane_service_ability.h` | SANE 服务接口定义 |

### 4.3 常量定义文件

| 文件 | 用途 |
|------|------|
| `utils/include/print_constant.h` | 打印相关常量和错误码 |
| `utils/include/scan_constant.h` | 扫描相关常量和错误码 |

### 4.4 IPC 接口文件

| 文件 | 用途 |
|------|------|
| `frameworks/innerkitsimpl/print_impl/src/print_service_proxy.cpp` | 打印服务客户端代理 |
| `frameworks/innerkitsimpl/scan_impl/src/scan_service_proxy.cpp` | 扫描服务客户端代理 |
| `services/print_service/src/print_service_stub.cpp` | 打印服务服务端桩 |
| `services/scan_service/src/scan_service_stub.cpp` | 扫描服务服务端桩 |

---

**相关链接**:
- [项目概览](00_Overview.md)
- [架构设计](03_Architecture.md)
- [对外 API](04_External_API.md)

# HDF Core 目录结构

## 1. 顶层目录

```
drivers/hdf_core/
├── adapter/           # 适配层（KHDF/UHDF 实现）
├── figures/           # 文档图片
├── framework/         # 框架核心代码
├── interfaces/        # 接口定义
├── bundle.json        # 组件配置
├── README.md          # 英文说明
└── README_zh.md       # 中文说明
```

## 2. Adapter 目录（适配层）

```
adapter/
├── BUILD.gn                    # 构建入口
├── build/test_common/          # 测试公共代码（IGNORE）
├── khdf/                       # Kernel HDF（内核态）
│   ├── linux/                  # Linux 内核适配
│   ├── liteos/                 # LiteOS-A 内核适配
│   ├── liteos_m/               # LiteOS-M 内核适配
│   └── uniproton/              # UniProton 内核适配
├── platform/                   # 平台驱动适配
│   ├── gpio/                   # GPIO 平台适配
│   ├── i2c/                    # I2C 平台适配
│   ├── pwm/                    # PWM 平台适配
│   ├── spi/                    # SPI 平台适配
│   ├── uart/                   # UART 平台适配
│   ├── watchdog/               # Watchdog 平台适配
│   └── mipi_dsi/               # MIPI DSI 平台适配
├── uhdf/                       # User HDF（旧版，Lite 系统）
└── uhdf2/                      # User HDF（新版，Standard 系统）
    ├── config/                 # 配置解析模块
    ├── hdi/                    # HDI 实现
    ├── host/                   # DevHost 实现
    ├── ipc/                    # IPC 适配
    ├── manager/                # DevMgr 实现
    ├── osal/                   # OS 适配层
    ├── platform/               # 平台驱动接口
    ├── security/               # 安全模块
    ├── shared/                 # Host/Manager 共享代码
    └── uhdf.gni                # UHDF 构建配置
```

## 3. Framework 目录（框架核心）

```
framework/
├── core/                       # 核心框架
│   ├── adapter/                # 内核适配层
│   │   ├── syscall/            # 系统调用适配
│   │   └── vnode/              # VNode 适配
│   ├── common/                 # 公共基础代码
│   │   └── src/                # 通用实现
│   ├── host/                   # Host 模块（核心逻辑）
│   │   ├── include/            # 头文件
│   │   └── src/                # 实现
│   ├── manager/                # Manager 模块（核心逻辑）
│   │   ├── include/            # 头文件
│   │   └── src/                # 实现
│   ├── sec/                    # 安全相关
│   │   └── include/            # 安全头文件
│   └── shared/                 # Host/Manager 共享
│       ├── include/            # 共享头文件
│       └── src/                # 共享实现
├── include/                    # 对外头文件
│   ├── audio/                  # 音频接口
│   ├── bluetooth/              # 蓝牙接口
│   ├── camera/                 # 相机接口
│   ├── core/                   # 核心接口
│   ├── ethernet/               # 以太网接口
│   ├── net/                    # 网络接口
│   ├── osal/                   # OSAL 接口
│   ├── platform/               # 平台接口（GPIO/I2C/SPI等）
│   ├── utils/                  # 工具接口
│   └── wifi/                   # WLAN 接口
├── model/                      # 驱动模型
│   ├── audio/                  # 音频框架
│   ├── camera/                 # 相机框架
│   ├── display/                # 显示框架
│   ├── input/                  # 输入框架
│   ├── misc/                   # 杂项（dsoftbus/light/vibrator）
│   ├── network/                # 网络/WLAN 框架
│   ├── sensor/                 # 传感器框架
│   ├── storage/                # 存储框架
│   └── usb/                    # USB 框架
├── sample/                     # 示例代码
│   └── platform/               # 平台驱动示例
├── support/                    # 基础能力
│   ├── platform/               # 平台驱动框架
│   │   ├── include/can/        # CAN 接口
│   │   ├── include/fwk/        # 框架接口
│   │   ├── include/gpio/       # GPIO 接口
│   │   ├── include/hdmi/       # HDMI 接口
│   │   ├── include/i2c/        # I2C 接口
│   │   ├── include/pin/        # PIN 接口
│   │   ├── include/pwm/        # PWM 接口
│   │   ├── include/spi/        # SPI 接口
│   │   ├── include/timer/      # Timer 接口
│   │   └── src/                # 实现
│   └── posix/                  # POSIX 适配
│       └── src/                # Mem/Mutex/Sem/Spinlock/Thread/Time
├── test/                       # 测试代码（IGNORE）
├── tools/                      # 工具
│   ├── hc-gen/                 # HCS 配置编译器
│   ├── hdf_dbg/                # HDF 调试工具
│   ├── hdf_dev_eco_tool/       # 驱动开发工具
│   ├── hcs-view/               # HCS 可视化工具
│   └── hdi-gen/                # HDI 代码生成器
└── utils/                      # 基础数据结构
    ├── include/                # 头文件
    └── src/                    # 实现（hcs_parser等）
```

## 4. Interfaces 目录（接口定义）

```
interfaces/
└── inner_api/                  # 内部 API
    ├── core/                   # 核心接口
    │   ├── hdf_device_class.h
    │   ├── hdf_io_service.h
    │   ├── hdf_io_service_if.h
    │   ├── hdf_object.h
    │   ├── hdf_service_status.h
    │   ├── ioservstat_listener.h
    │   └── svcmgr_ioservice.h
    ├── hdi/                    # HDI 接口
    │   ├── base/               # HDI 基础（SMQ/Buffer）
    │   ├── devmgr_hdi.h
    │   ├── hdi_base.h
    │   ├── hdi_support.h
    │   ├── idevmgr_hdi.h
    │   ├── iservmgr_hdi.h
    │   ├── servmgr_hdi.h
    │   └── ...
    ├── host/                   # Host 接口
    │   ├── shared/             # 共享接口
    │   └── uhdf/               # UHDF 专用
    ├── ipc/                    # IPC 接口
    │   ├── hdf_dump_reg.h
    │   ├── hdf_remote_service.h
    │   ├── hdf_sbuf_ipc.h
    │   └── iproxy_broker.h
    ├── osal/                   # OSAL 接口
    │   ├── shared/             # 共享 OSAL
    │   └── uhdf/               # UHDF OSAL
    └── utils/                  # 工具接口
        ├── hdf_base.h
        ├── hdf_dlist.h
        ├── hdf_log.h
        ├── hdf_sbuf.h
        └── ...
```

## 5. 关键文件索引

### 5.1 驱动入口
| 文件 | 说明 |
|------|------|
| `interfaces/inner_api/host/shared/hdf_driver.h` | 驱动入口定义 |
| `interfaces/inner_api/host/shared/hdf_device_desc.h` | 设备描述符 |
| `interfaces/inner_api/host/shared/hdf_device_object.h` | 设备对象 |

### 5.2 核心框架
| 文件 | 说明 |
|------|------|
| `framework/core/manager/src/devmgr_service.c` | 设备管理服务 |
| `framework/core/host/src/devhost_service.c` | 驱动主机服务 |
| `framework/core/shared/src/hdf_object_manager.c` | 对象管理 |

### 5.3 UHDF 实现
| 文件 | 说明 |
|------|------|
| `adapter/uhdf2/manager/src/devmgr_service_stub.c` | DevMgr IPC Stub |
| `adapter/uhdf2/manager/src/devsvc_manager_stub.c` | Service Manager Stub |
| `adapter/uhdf2/host/src/devhost_service_full.c` | DevHost 实现 |
| `adapter/uhdf2/host/src/devhost_service_stub.c` | DevHost IPC Stub |
| `adapter/uhdf2/ipc/src/hdf_remote_adapter.cpp` | IPC 适配器 |

### 5.4 HDI 接口
| 文件 | 说明 |
|------|------|
| `interfaces/inner_api/hdi/iservmgr_hdi.h` | IServiceManager 接口 |
| `interfaces/inner_api/hdi/idevmgr_hdi.h` | IDeviceManager 接口 |
| `adapter/uhdf2/hdi/src/servmgr_client.c` | ServiceManager 客户端 |
| `adapter/uhdf2/hdi/src/devmgr_client.c` | DeviceManager 客户端 |

### 5.5 平台接口
| 文件 | 说明 |
|------|------|
| `framework/include/platform/gpio_if.h` | GPIO 接口 |
| `framework/include/platform/i2c_if.h` | I2C 接口 |
| `framework/include/platform/spi_if.h` | SPI 接口 |
| `framework/include/platform/uart_if.h` | UART 接口 |
| `framework/include/platform/pwm_if.h` | PWM 接口 |

### 5.6 安全模块
| 文件 | 说明 |
|------|------|
| `adapter/uhdf2/security/include/hdf_security.h` | 安全模块头文件 |
| `adapter/uhdf2/security/src/hdf_security.c` | 安全实现（权限管理） |

### 5.7 构建配置
| 文件 | 说明 |
|------|------|
| `adapter/BUILD.gn` | 主构建入口 |
| `adapter/uhdf2/uhdf.gni` | UHDF 构建配置 |
| `bundle.json` | 组件配置 |

## 6. 模块职责说明

### 6.1 Core 模块
| 模块 | 职责 |
|------|------|
| core/manager | 设备管理核心逻辑（加载、卸载、查询） |
| core/host | 驱动主机核心逻辑（设备生命周期） |
| core/shared | Host/Manager 共享代码 |
| core/adapter | 内核适配（syscall、vnode） |

### 6.2 UHDF2 模块
| 模块 | 职责 |
|------|------|
| uhdf2/manager | DevMgr 进程实现（设备管理、服务管理） |
| uhdf2/host | DevHost 进程实现（驱动容器） |
| uhdf2/ipc | IPC 适配（Binder 封装） |
| uhdf2/hdi | HDI 客户端实现 |
| uhdf2/security | 安全模块（权限管理） |
| uhdf2/platform | 平台驱动接口（用户态） |

### 6.3 Model 模块
| 模块 | 职责 |
|------|------|
| model/audio | 音频驱动框架 |
| model/display | 显示驱动框架 |
| model/input | 输入驱动框架 |
| model/network | 网络/WLAN 驱动框架 |
| model/sensor | 传感器驱动框架 |
| model/storage | 存储驱动框架 |
| model/usb | USB 驱动框架 |

## 7. 代码统计（估算）

| 目录 | 代码类型 | 说明 |
|------|----------|------|
| adapter/ | C/C++ | ~50K 行，适配层实现 |
| framework/core/ | C | ~30K 行，核心框架 |
| framework/model/ | C | ~100K 行，驱动模型 |
| framework/support/ | C | ~20K 行，平台支持 |
| framework/utils/ | C | ~10K 行，基础工具 |
| interfaces/ | C/C++ 头文件 | API 定义 |

## 8. 相关文档

- [项目概览](./01_Overview.md)
- [架构说明](./03_Architecture.md)
- [GN 构建系统](./06_GN_Build.md)

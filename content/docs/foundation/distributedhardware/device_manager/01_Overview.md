# 项目概览

## 1. 项目定位

**DeviceManager** 是 OpenHarmony 分布式硬件子系统的核心组件，提供账号无关的分布式设备的认证组网能力。

### 核心能力

- **设备发现**：发现周边的分布式设备
- **设备认证**：通过 PIN 码等方式认证设备
- **设备状态管理**：监听设备上下线状态
- **设备组网**：建立设备间的信任关系

### 适用场景

- 分布式数据同步
- 跨设备任务流转
- 多设备协同操作

## 2. 系统依赖

DeviceManager 依赖以下核心子系统：

| 依赖组件 | 作用 |
|---------|------|
| **dsoftbus** | 提供设备上下线通知、设备信息、设备认证通道、设备发现能力 |
| **deviceauth** | 提供设备群组管理和群组认证能力 |

> 证据来源：`README_zh.md:12-15`

## 3. 目录结构

```
device_manager/
├── common/                        # 公共能力头文件
│   └── include/ipc/model/         # IPC 功能模块头文件
├── display/                       # PIN 码显示 HAP（系统应用）
├── interfaces/
│   ├── cj/                        # 仓颉 FFI 接口
│   ├── inner_kits/                # 内部 Native 接口
│   │   └── native_cpp/           # IPC 核心实现
│   └── kits/                      # 外部 JS 接口
│       ├── js/                    # 传统 JS 接口
│       └── js4.0/                 # OpenHarmony 4.0+ JS 接口
├── sa_profile/                     # SA 进程配置
├── services/
│   ├── implementation/           # 服务实现核心代码
│   └── service/                   # SA 服务核心代码
└── utils/                         # 公共能力（加密、IPC、日志）

test/                              # 测试代码（*不纳入本文档范围*）
```

> 证据来源：`README_zh.md:19-117`

## 4. 技术栈

| 类别 | 技术 |
|-----|------|
| **编程语言** | C++, JavaScript |
| **运行环境** | OpenHarmony 设备（Hi3516DV300 等） |
| **系统能力** | System Ability (SA) |
| **通信机制** | IPC, DSoftBus |

> 证据来源：`README_zh.md:121-122`

## 5. 版本演进

### 传统 API（Legacy）

- **入口**：`createDeviceManager(bundleName, callback)`
- **接口文件**：`ohos.distributedHardware.deviceManager.d.ts`
- **调用方式**：Callback/Promise 混用

### OpenHarmony 4.0+ API（New）

- **入口**：`createDeviceManager(bundleName)` - 同步返回实例
- **接口文件**：`ohos.distributedDeviceManager.d.ts`
- **特性**：支持三方应用调用，需申请 `ohos.permission.DISTRIBUTED_DATASYNC` 权限

> 证据来源：`README_zh.md:125-225`

## 6. 权限要求

### 基础权限

| 权限名称 | 用途 |
|---------|------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 软总线中心 |

### 系统权限（仅系统应用）

| 权限名称 | 用途 |
|---------|------|
| `ohos.permission.ACCESS_SERVICE_DP` | 访问设备配置服务 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 管理安全设置 |

> 完整权限列表见 [SA 配置文件](sa_profile/device_manager.cfg)

## 7. 部署形态

DeviceManager 以 **System Ability (SA)** 形式运行：

- **进程**：`device_manager` 独立进程
- **UID**：device_manager
- **权限等级**：system_basic
- **启动方式**：按需启动（BOOT_COMPLETED 事件触发）

> 证据来源：`sa_profile/device_manager.cfg:10-60`

## 8. 相关文档

| 文档 | 链接 |
|-----|------|
| API 参考 | [distributedDeviceManager](../interfaces/kits/js4.0/include/) |
| 架构图 | [devicemanager_zh.png](../figures/devicemanager_zh.png) |
| 仓颉接口 | [interfaces/cj](../interfaces/cj/) |
| 内部实现 | [services/implementation](../services/implementation/) |

## 9. 快速开始

```javascript
// 创建设备管理器实例
let dmClass = deviceManager.createDeviceManager("ohos.samples.jshelloworld");

// 注册设备状态监听
dmClass.on('deviceStateChange', (data) => {
    console.info("deviceStateChange:" + JSON.stringify(data));
});

// 发现周边设备
dmClass.startDiscovering({
    'discoverTargetType': 1
});

// 停止发现
dmClass.stopDiscovering();

// 释放实例
deviceManager.releaseDeviceManager(dmClass);
```

> 更多示例见 [README_zh.md](../README_zh.md)

# 项目概述

## 1.1 项目定位

`drivers/peripheral` 是 OpenHarmony 操作系统的外设驱动子系统，承担着连接系统服务与硬件设备的桥梁作用。

### 核心职责
1. **定义 HDI 接口**：为上层系统服务提供统一的硬件抽象接口
2. **规范 HAL 实现**：为南向 OEM 厂商提供实现标准
3. **集成 HDF 框架**：与 OpenHarmony 驱动框架深度配合
4. **提供测试用例**：确保驱动的正确性和兼容性

### 边界与约束
- **不包含**：内核态驱动代码（位于 `drivers_adapter_khdf_linux`）
- **不包含**：JS/N-API 层（位于各子系统仓库）
- **不包含**：系统服务实现（位于对应服务子系统）

## 1.2 技术架构

### 分层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        上层服务层                                │
│  (AudioService, InputService, SensorService, etc.)              │
├─────────────────────────────────────────────────────────────────┤
│                      HDI 接口层 (本仓库)                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  interfaces/include/ - C 语言接口定义                    │   │
│  │  audio_manager.h, input_manager.h, sensor_if.h 等     │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                    HDI 服务实现层 (本仓库)                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  hdi_service/ - HDI 服务端实现                         │   │
│  │  ├── device/ - 设备相关实现                             │   │
│  │  ├── proxy/ - 客户端代理                                │   │
│  │  └── stub/ - 服务端存根                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                      HAL 层 (本仓库)                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  hal/ - 硬件抽象层实现                                  │   │
│  │  ├── include/ - 内部头文件                             │   │
│  │  └── src/ - 具体实现                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                   HDF 框架层 (drivers_framework)                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - IPC/RPC 通信                                         │   │
│  │  - 设备节点管理                                         │   │
│  │  - 服务注册发现                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│               适配层 (drivers_adapter)                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - HDI-Adapter 接口适配                                 │   │
│  │  - KHDF 内核驱动桥接                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                      内核驱动 (KHDF Linux)                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  - Linux 内核驱动                                      │   │
│  │  - 设备树配置                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 线程模型

**典型 HDI 调用时序**：
```
┌────────────┐     ┌─────────────┐     ┌──────────────┐     ┌─────────┐
│ System     │────▶│ HDF IPC     │────▶│ HDI Service   │────▶│ HAL     │
│ Service    │     │ (Binder)    │     │ (Stub)        │     │ Impl    │
└────────────┘     └─────────────┘     └──────────────┘     └─────────┘
                        │                    │
                        ▼                    ▼
                 ┌─────────────┐     ┌──────────────┐
                 │ MessageQueue │     │ WorkThread   │
                 │ (主线程)     │     │ (异步处理)   │
                 └─────────────┘     └──────────────┘
```

- **同步调用**：在调用者线程阻塞等待结果
- **异步调用**：通过回调或事件上报机制
- **事件上报**：注册回调函数，硬件事件触发时主动通知

## 1.3 目录结构规范

### 标准模块结构

```
module_name/
├── interfaces/              # 对外接口目录
│   ├── include/            # 头文件目录
│   │   └── *.h            # HDI 接口定义
│   └── supportlibs/       # 辅助库（可选）
├── hal/                     # 硬件抽象层
│   ├── include/            # 内部头文件
│   └── src/                # HAL 实现
├── hdi_service/            # HDI 服务实现
│   ├── include/            # 服务头文件
│   └── src/               # 服务实现
├── utils/                  # 工具类（可选）
├── chipset/                # 芯片相关（可选）
├── figures/                # 文档图片
├── test/                   # 测试代码
│   ├── unittest/          # 单元测试
│   ├── moduletest/        # 模块测试
│   ├── fuzztest/          # 模糊测试
│   └── performance/       # 性能测试
├── BUILD.gn               # 模块构建配置
├── *.gni                  # 模块配置片段
├── README.md              # 模块说明
└── README_zh.md           # 中文说明
```

### 关键文件说明

| 文件/目录 | 说明 | 证据位置 |
|-----------|------|----------|
| `interfaces/include/*.h` | HDI 接口定义 | `audio/interfaces/include/audio_manager.h` |
| `hal/src/*.c` | HAL 实现 | `input/hal/src/input_controller.c` |
| `hdi_service/src/*.cpp` | HDI 服务端 | `input/hdi_service/src/input_interface_driver.cpp` |
| `BUILD.gn` | GN 构建配置 | `audio/BUILD.gn` |

## 1.4 关键技术特性

### 1.4.1 接口定义风格

使用 C 语言结构体函数指针模式：

```c
// 接口定义示例 (audio_manager.h)
struct AudioManager {
    int32_t (*GetAllAdapters)(struct AudioManager *manager,
                              struct AudioAdapterDescriptor **descs,
                              int32_t *size);
    int32_t (*LoadAdapter)(struct AudioManager *manager,
                           const struct AudioAdapterDescriptor *desc,
                           struct AudioAdapter **adapter);
    void (*UnloadAdapter)(struct AudioManager *manager,
                          struct AudioAdapter *adapter);
};
```

### 1.4.2 版本化接口

支持多版本接口共存：
- `interfaces/2.0/` - 2.0 版本接口
- `interfaces/v1_0/` - 1.0 版本接口
- `interfaces/include/` - 最新版本

### 1.4.3 IPC 通信机制

基于 HDF 的 Binder 通信：
- **Stub（服务端）**：继承 `IRemoteBroker`，实现 `OnRemoteRequest`
- **Proxy（客户端）**：提供远程调用代理
- **MessageParcel**：序列化/反序列化参数

**证据**：`display/hdi_service/device/include/server/display_device_stub.h:38`

```cpp
class DisplayDeviceStub : public IRemoteBroker {
    int32_t OnRemoteRequest(int cmdId, MessageParcel *data,
                           MessageParcel *reply) override;
};
```

### 1.4.4 错误处理规范

- 返回值：`int32_t`，0 表示成功，负数表示错误
- 错误码定义：各模块独立定义
- 指针校验：`NULL_POINTER` 检查宏

**证据**：`input/input_manager.h` 中的 `INPUT_CHECK_NULL_POINTER` 宏

## 1.5 依赖关系

### 内部依赖
- `base/` - 基础公共组件（buffer_handle, hdf_trace）
- 各模块间可能的依赖（camera → sensor, audio → vibrator）

### 外部依赖
- `drivers_framework` - HDF 框架
- `drivers_adapter` - 接口适配层
- `drivers_hdf_core` - HDF 核心服务

### 被依赖
- 各系统服务子系统（multimedia, inputmethod, etc.）

## 1.6 构建与部署

### 构建系统
- **GN (Generate Ninja)**：主构建系统
- **hb (OpenHarmony Build)**：顶层构建工具

### 编译产物
- `.so` 动态库：HDI 服务实现
- `.a` 静态库：HAL 实现
- `.bin`：配置文件

### 运行时加载
- HDF 自动加载驱动模块
- 服务按需启动
- 设备节点动态创建

---

**下一节**: [模块目录](02_Module_Catalog.md) - 查看所有外设模块

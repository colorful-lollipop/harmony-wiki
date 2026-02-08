# OpenHarmony Peripheral Drivers

## 项目简介

`drivers/peripheral` 是 OpenHarmony 驱动子系统的核心仓库，存储外设硬件的 **HDI（Hardware Driver Interface）接口定义**、**HAL（Hardware Abstraction Layer）实现**、**驱动模型** 和相关测试用例。

## 核心能力

### 1. HDI 接口标准化
为上层服务提供统一的硬件驱动能力接口，包括：
- 设备管理（打开/关闭/查询）
- 数据读写（同步/异步）
- 状态监控（电源/事件）
- 配置控制（参数/模式）

### 2. HAL 实现规范
定义南向 OEM 厂商需要实现的适配层标准，确保：
- 接口一致性
- 生态兼容性
- 硬件无关性

### 3. 驱动框架集成
与 OpenHarmony HDF（Hardware Driver Framework）深度集成：
- IPC/RPC 通信
- 服务注册与发现
- 设备节点管理

## 模块分类

| 类别 | 模块 | 主要功能 |
|------|------|----------|
| **音视频** | audio, camera, display, codec, format | 音视频采集、编解码、显示输出 |
| **输入传感** | input, sensor, motion | 触摸、按键、姿态、环境感知 |
| **通信** | wlan, bluetooth, usb, nfc, ril | 无线通信、数据传输 |
| **安全认证** | fingerprint, face, user, pin auth | 生物识别、身份认证 |
| **电源功耗** | power, battery, thermal | 电源管理、温控策略 |
| **外设控制** | vibrator, light, location | 振动反馈、灯光、定位 |

## 架构位置

```
┌─────────────────────────────────────────────────────┐
│                  应用层 (Apps)                       │
├─────────────────────────────────────────────────────┤
│                系统服务 (System Services)            │
│         (InputService, AudioService, etc.)          │
├─────────────────────────────────────────────────────┤
│              驱动框架层 (drivers_framework)          │
│           HDF, Driver Host, IPC/SPI                │
├─────────────────────────────────────────────────────┤
│              外设驱动层 (drivers_peripheral)        │
│         HDI Interfaces + HAL Implementation         │
├─────────────────────────────────────────────────────┤
│              适配层 (drivers_adapter)                │
│         HDI-Adapters, Kernel Driver (KHDF)         │
├─────────────────────────────────────────────────────┤
│                 硬件 (Hardware)                     │
└─────────────────────────────────────────────────────┘
```

## 技术特点

### 接口风格
- **C 语言接口**：使用 struct 函数指针定义回调式 API
- **版本化接口**：支持 `v1_0`, `v2_0` 等多版本并存
- **稳定接口**：明确标注稳定/不稳定状态

### 通信机制
- **HDF IPC**：基于 MessageParcel 的同步/异步调用
- **Binder 框架**：跨进程通信
- **Stub/Proxy 模式**：客户端-服务端分离

### 构建系统
- **GN (Generate Ninja)**：Google 构建系统
- **模块化编译**：按需选择编译模块
- **配置化管理**：`.gni`, `BUILD.gn` 配置

## 使用指南

### 调用 HDI 接口
```c
// 1. 获取接口实例
IInputInterface *inputInterface = nullptr;
int32_t ret = GetInputInterface(&inputInterface);
if (ret != INPUT_SUCCESS) { /* 错误处理 */ }

// 2. 打开设备
ret = inputInterface->iInputManager->OpenInputDevice(devIndex);

// 3. 注册回调
inputInterface->iInputReporter->RegisterReportCallback(devIndex, &callback);

// 4. 使用完毕后关闭
inputInterface->iInputManager->CloseInputDevice(devIndex);
```

### 实现 HAL 接口
```c
// 1. 实现函数指针结构体
static struct AudioAdapter g_myAdapter = {
    .InitAllPorts = MyInitAllPorts,
    .CreateRender = MyCreateRender,
    .DestroyRender = MyDestroyRender,
    // ...
};

// 2. 注册到 HDI 框架
AudioAdapterManagerRegister(&g_myAdapter);
```

## 相关资源

### 官方文档
- [OpenHarmony 驱动开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/驱动子系统.md)
- [HDF 框架说明](https://gitee.com/openharmony/drivers_framework/blob/master/README_zh.md)

### 代码位置
- 接口定义：`*/interfaces/include/`
- HAL 实现：`*/hal/src/`
- HDI 服务：`*/hdi_service/`
- 构建配置：`*/BUILD.gn`

### 外部仓库
- [drivers_framework](https://gitee.com/openharmony/drivers_framework)
- [drivers_adapter](https://gitee.com/openharmony/drivers_adapter)
- [drivers_adapter_khdf_linux](https://gitee.com/openharmony/drivers_adapter_khdf_linux)

---

**下一步**: 阅读 [项目概述](01_Overview.md) 了解详细架构

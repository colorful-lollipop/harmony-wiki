# 项目概览

## 项目定位

**sensor_lite** 是 OpenHarmony 泛 Sensor 服务子系统的轻量级实现，专为 **LiteOS-M 内核**（mini 系统）设计。

### 核心能力

| 能力 | 描述 | API |
|-----|------|-----|
| 传感器列表查询 | 获取系统中所有可用传感器的信息 | `GetAllSensors` |
| 传感器使能 | 激活指定的传感器 | `ActivateSensor` |
| 传感器去使能 | 关闭指定的传感器 | `DeactivateSensor` |
| 数据订阅 | 订阅传感器数据流 | `SubscribeSensor` |
| 取消订阅 | 停止接收传感器数据 | `UnsubscribeSensor` |
| 批处理设置 | 配置采样间隔和上报间隔 | `SetBatch` |
| 模式设置 | 配置数据上报模式 | `SetMode` |
| 选项设置 | 配置传感器特殊选项 | `SetOption` |

## 运行环境

### 系统要求

| 要求 | 说明 |
|-----|------|
| 内核 | LiteOS-M |
| 系统类型 | mini |
| ROM 占用 | ~92KB |
| RAM 占用 | ~200KB |

### 依赖组件

| 组件 | 用途 |
|-----|------|
| `drivers_peripheral_sensor` | HDF 传感器硬件驱动抽象 |
| `ipc` | 进程间通信框架 |
| `samgr_lite` | 系统能力管理 |
| `hilog_lite` | 日志系统 |
| `utils_lite` | 基础工具库 |

## 关键概念

### 传感器类型 (SensorTypeId)

sensor_lite 支持 **30+ 种传感器**，分为两类：

#### 物理传感器 (0x00 - 0xFF)

| ID | 传感器 | 说明 |
|---|-------|------|
| 1 | ACCELEROMETER | 加速度传感器 |
| 2 | GYROSCOPE | 陀螺仪 |
| 3 | PHOTOPLETHYSMOGRAPH | 光电容积脉搏波传感器 |
| 4 | ELECTROCARDIOGRAPH | 心电图传感器 |
| 5 | AMBIENT_LIGHT | 环境光传感器 |
| 6 | MAGNETIC_FIELD | 磁场传感器 |
| 7 | CAPACITIVE | 电容传感器 |
| 8 | BAROMETER | 气压传感器 |
| 9 | TEMPERATURE | 温度传感器 |
| 10 | HALL | 霍尔传感器 |
| 11 | GESTURE | 手势传感器 |
| 12 | PROXIMITY | 接近传感器 |
| 13 | HUMIDITY | 湿度传感器 |

#### 虚拟传感器 (0x100+)

| ID | 传感器 | 说明 |
|---|-------|------|
| 256 | ORIENTATION | 方向传感器 |
| 257 | GRAVITY | 重力传感器 |
| 258 | LINEAR_ACCELERATION | 线性加速度 |
| 259 | ROTATION_VECTOR | 旋转矢量 |
| ... | ... | 更多虚拟传感器 |

**证据**: `interfaces/kits/native/include/sensor_agent_type.h:72-104`

### 数据上报模式 (SensorMode)

| 模式 | 值 | 说明 |
|-----|---|------|
| DEFAULT | 0 | 默认模式 |
| REALTIME | 1 | 实时上报模式 |
| ON_CHANGE | 2 | 变化时上报 |
| ONE_SHOT | 3 | 单次上报 |
| FIFO_MODE | 4 | FIFO 队列模式 |

**证据**: `interfaces/kits/native/include/sensor_agent_type.h:171-178`

### 错误码

| 错误码 | 值 | 说明 |
|-------|-----|------|
| SENSOR_OK | 0 | 成功 |
| SENSOR_ERROR_UNKNOWN | -1 | 未知错误 |
| SENSOR_ERROR_INVALID_ID | -2 | 无效的传感器 ID |
| SENSOR_ERROR_INVALID_PARAM | -3 | 无效参数 |

**证据**: `interfaces/kits/native/include/sensor_agent_type.h:48-51`

## 目录结构

```
sensor_lite/
├── frameworks/          # 框架代码
│   ├── include/        # 头文件
│   │   └── sensor_agent_proxy.h
│   └── src/            # 源代码
│       ├── sensor_agent.c      # API 入口
│       ├── sensor_agent_client.c  # liteos_riscv 客户端
│       └── sensor_agent_proxy.c   # liteos_a/linux 客户端
├── interfaces/         # 对外接口
│   └── kits/native/    # Native API
│       └── include/    # API 头文件
│           ├── sensor_agent.h       # 主 API
│           └── sensor_agent_type.h  # 类型定义
├── services/           # 服务实现
│   ├── include/        # 服务头文件
│   │   ├── sensor_service.h
│   │   └── sensor_service_impl.h
│   └── src/            # 服务源代码
│       ├── sensor_service.c         # IPC 分发
│       ├── sensor_service_impl.c    # 业务实现
│       └── proc.c
├── figures/            # 文档图片
├── BUILD.gn           # 构建入口
├── bundle.json        # 组件配置
└── sensor_lite.gni    # GN 配置
```

**证据**: `bundle.json:9-35`

## 版本历史

| 版本 | 日期 | 变更 |
|-----|------|------|
| 3.1 | - | 当前版本 |

---

## 相关文档

- [API 参考](02_API_Reference.md)
- [架构设计](03_Architecture.md)
- [构建配置](04_Build.md)
- [安全评审](05_Security.md)

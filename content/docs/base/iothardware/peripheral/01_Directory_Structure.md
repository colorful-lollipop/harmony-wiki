# 目录结构

> IoT Hardware Peripheral 子系统目录结构与模块职责

## 目录树

```
/base/iothardware/peripheral/
├── interfaces/                          # 接口定义层
│   └── inner_api/                        # 内部 C API 头文件
│       ├── iot_gpio.h                    # GPIO 操作接口
│       ├── iot_i2c.h                     # I2C 总线接口
│       ├── iot_uart.h                    # UART 串口接口
│       ├── iot_pwm.h                     # PWM 输出接口
│       ├── iot_watchdog.h                # 看门狗接口
│       ├── iot_flash.h                   # Flash 存储接口
│       ├── iot_errno.h                   # 错误码定义
│       ├── reset.h                       # 设备重置接口
│       └── lowpower.h                    # 低功耗管理接口
├── BUILD.gn                              # 根构建配置
├── bundle.json                           # 组件配置
├── LICENSE                               # Apache 2.0 许可证
├── README.md                             # 项目说明
└── README_zh.md                          # 中文项目说明
```

## 模块职责

### interfaces/inner_api/

**职责**：定义 IoT 硬件操作的标准 C 接口，供 Native 应用调用。

| 文件 | 职责 | 导出函数数 |
|------|------|------------|
| `iot_gpio.h` | GPIO 初始化、方向设置、电平读写、中断管理 | 10 |
| `iot_i2c.h` | I2C 初始化、去初始化、读写操作、波特率设置 | 5 |
| `iot_uart.h` | UART 初始化、去初始化、读写、流控设置 | 5 |
| `iot_pwm.h` | PWM 初始化、去初始化、启动、停止 | 4 |
| `iot_watchdog.h` | 看门狗使能、喂狗、去使能 | 3 |
| `iot_flash.h` | Flash 初始化、去初始化、读、写、擦除 | 5 |
| `iot_errno.h` | 定义返回码常量 | 2 |
| `reset.h` | 设备软复位 | 1 |
| `lowpower.h` | 低功耗初始化、模式设置 | 2 |

**总计**：9 个头文件，42 个函数/宏定义

## 头文件分类

### 按功能分类

```
硬件操作类
├── iot_gpio.h     (通用输入输出)
├── iot_i2c.h      (I2C 总线通信)
├── iot_uart.h     (UART 串口通信)
├── iot_pwm.h      (PWM 脉冲调制)
├── iot_watchdog.h (看门狗定时器)
└── iot_flash.h    (Flash 存储操作)

系统管理类
├── reset.h        (设备重置)
└── lowpower.h    (低功耗管理)

基础支撑类
└── iot_errno.h   (错误码定义)
```

### 按稳定性分类

| 稳定性 | 头文件 | 证据 |
|--------|--------|------|
| **稳定** | `iot_gpio.h`, `iot_i2c.h`, `iot_uart.h` | 完整的 Doxygen 注释，版本 2.2 |
| **稳定** | `iot_pwm.h`, `iot_watchdog.h` | 完整的 Doxygen 注释，版本 2.2 |
| **稳定** | `iot_flash.h`, `iot_errno.h` | 完整的 Doxygen 注释，版本 2.2 |
| **稳定** | `reset.h`, `lowpower.h` | 完整的 Doxygen 注释，版本 2.2 |

**注意**：所有接口均为稳定接口，无 "beta" 或 "deprecated" 标记。

## 接口统计

### 函数数量统计

| 模块 | 公共函数 | 内部函数 | 总计 |
|------|----------|----------|------|
| GPIO | 10 | 0 | 10 |
| I2C | 5 | 0 | 5 |
| UART | 5 | 0 | 5 |
| PWM | 4 | 0 | 4 |
| Watchdog | 3 | 0 | 3 |
| Flash | 5 | 0 | 5 |
| Reset | 1 | 0 | 1 |
| Lowpower | 2 | 0 | 2 |
| Errno | 2 | 0 | 2 |

### 枚举/结构体统计

| 模块 | 枚举数量 | 结构体数量 |
|------|----------|------------|
| GPIO | 4 | 0 |
| I2C | 0 | 0 |
| UART | 5 | 1 |
| PWM | 0 | 0 |
| Watchdog | 0 | 0 |
| Flash | 0 | 0 |
| Reset | 0 | 0 |
| Lowpower | 1 | 0 |

## 关键文件说明

### iot_gpio.h - GPIO 接口

**功能**：提供 GPIO 引脚的初始化、方向设置、电平读写和中断管理能力。

**关键类型**：
```c
typedef enum { IOT_GPIO_VALUE0, IOT_GPIO_VALUE1 } IotGpioValue;
typedef enum { IOT_GPIO_DIR_IN, IOT_GPIO_DIR_OUT } IotGpioDir;
typedef enum { IOT_INT_TYPE_LEVEL, IOT_INT_TYPE_EDGE } IotGpioIntType;
typedef enum { IOT_GPIO_EDGE_FALL_LEVEL_LOW, IOT_GPIO_EDGE_RISE_LEVEL_HIGH } IotGpioIntPolarity;
typedef void (*GpioIsrCallbackFunc)(char *arg);
```

**关键函数**：
- `IoTGpioInit(unsigned int id)` - 初始化 GPIO
- `IoTGpioSetDir(unsigned int id, IotGpioDir dir)` - 设置方向
- `IoTGpioSetOutputVal(unsigned int id, IotGpioValue val)` - 设置输出电平
- `IoTGpioRegisterIsrFunc(...)` - 注册中断回调

### iot_uart.h - UART 接口

**功能**：提供 UART 串口的配置、读写和流控能力。

**关键类型**：
```c
typedef struct {
    unsigned int baudRate;
    IotUartIdxDataBit dataBits;
    IotUartStopBit stopBits;
    IotUartParity parity;
    IotUartBlockState rxBlock;
    IotUartBlockState txBlock;
    unsigned char pad;
} IotUartAttribute;
```

### iot_flash.h - Flash 接口

**功能**：提供对 Flash 存储器的读写擦除操作。

**关键函数**：
- `IoTFlashRead(unsigned int flashOffset, unsigned int size, unsigned char *ramData)`
- `IoTFlashWrite(unsigned int flashOffset, unsigned int size, const unsigned char *ramData, unsigned char doErase)`
- `IoTFlashErase(unsigned int flashOffset, unsigned int size)`

## 排除的目录

根据 Wiki 生成规范，以下目录不计入文档范围：

| 目录/文件 | 排除原因 |
|-----------|----------|
| `test/` | 测试代码 |
| `tests/` | 测试代码 |
| `unittest/` | 单元测试 |
| `*_test.*` | 测试相关文件 |
| `build/` | 构建产物 |
| `.git/` | Git 版本控制 |

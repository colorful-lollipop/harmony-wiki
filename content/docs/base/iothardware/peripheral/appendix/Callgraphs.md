# 关键调用链

> IoT Hardware Peripheral 子系统关键调用链图示

## 调用链概览

### GPIO 操作调用链

```
用户应用
    │
    │ #include "iot_gpio.h"
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  io_tgpio.h (接口定义)                                      │
│  - IoTGpioInit(unsigned int id)                            │
│  - IoTGpioSetDir(unsigned int id, IotGpioDir dir)         │
│  - IoTGpioSetOutputVal(unsigned int id, IotGpioValue val) │
│  - IoTGpioRegisterIsrFunc(...)                             │
└─────────────────────────────────────────────────────────────┘
    │
    │ 链接/静态编译
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  hal_iothardware (HAL 实现)                                 │
│  - hal_gpio_init(id)                                       │
│  - hal_gpio_set_direction(id, dir)                          │
│  - hal_gpio_write(id, val)                                  │
│  - hal_gpio_register_isr(...)                               │
└─────────────────────────────────────────────────────────────┘
    │
    │ 寄存器操作
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Hi3861 GPIO 控制器 (硬件)                                  │
│  - GPIO_DATA[GPIOx] 寄存器                                  │
│  - GPIO_DIR 寄存器                                          │
│  - GPIO_IS 寄存器 (中断状态)                                 │
│  - GPIO_IE 寄存器 (中断使能)                                 │
└─────────────────────────────────────────────────────────────┘
```

### I2C 操作调用链

```
用户应用
    │
    │ #include "iot_i2c.h"
    │
    ▼
IoTI2cInit(id, baudrate)  ──►  IoTI2cWrite(id, addr, data, len)
                                  │
                                  ├──► 配置 I2C 控制器
                                  │
                                  ├──► 发送 START 信号
                                  │
                                  ├──► 发送设备地址
                                  │
                                  ├──► 发送数据
                                  │
                                  └──► 发送 STOP 信号

IoTI2cRead(id, addr, data, len)
    │
    ├──► 发送 START 信号
    ├──► 发送设备地址 (读模式)
    ├──► 读取数据
    └──► 发送 STOP 信号
```

### UART 操作调用链

```
用户应用
    │
    │ #include "iot_uart.h"
    │
    ▼
IoTUartInit(id, &attr)
    │
    ├──► 配置波特率 (UARTx_BAUD)
    ├──► 配置数据位 (UARTx_CTRL)
    ├──► 配置停止位
    └──► 配置校验位

IoTUartWrite(id, data, len)
    │
    ├──► 写入 TX FIFO
    └──► 硬件自动发送

IoTUartRead(id, data, len)
    │
    ├──► 等待 RX FIFO 有数据
    └──► 读取 RX FIFO
```

## 中断处理调用链

### GPIO 中断调用链

```
外部中断触发 (边沿/电平变化)
    │
    ▼
Hi3861 GPIO 控制器检测中断
    │
    ▼
CPU 进入中断上下文
    │
    ▼
GPIO 中断服务程序 (ISR)
    │
    ├──► 读取 GPIO_IS 寄存器确定中断源
    ├──► 调用注册的回调函数
    │       │
    │       └──► GpioIsrCallbackFunc(arg)
    │                   │
    │                   └──► 用户中断处理逻辑
    │
    └──► 清除中断标志位
    │
    ▼
中断返回，恢复主程序执行
```

## Flash 操作调用链

```
用户应用
    │
    │ #include "iot_flash.h"
    │
    ▼
IoTFlashInit()
    │
    └──► 初始化 Flash 控制器

IoTFlashRead(offset, size, buffer)
    │
    ├──► 配置读取地址
    ├──► 触发读取操作
    └──► 传输数据到 buffer

IoTFlashWrite(offset, size, data, doErase)
    │
    ├──► IF doErase == 1:
    │       │
    │       └──► IoTFlashErase(offset, size)
    │                   │
    │                   ├──► 擦除目标块
    │                   └──► 等待擦除完成
    │
    ├──► 写入数据到目标地址
    └──► 验证写入结果

IoTFlashErase(offset, size)
    │
    ├──► 确定擦除块地址
    ├──► 发送擦除命令
    └──► 等待擦除完成 (毫秒级)
```

## 看门狗操作调用链

```
用户应用
    │
    │ #include "iot_watchdog.h"
    │
    ▼
IoTWatchDogEnable()
    │
    ├──► 配置看门狗超时时间
    └──► 使能看门狗定时器
    │
    ▼
看门狗定时器开始计数
    │
    ┌────────────────────────────────────────┐
    │ 主循环                                   │
    │     │                                   │
    │     ├──► 执行任务                        │
    │     │                                   │
    │     ├──► IoTWatchDogKick()  ──► 重置计数器 │
    │     │                                   │
    │     └──► 重复                           │
    └────────────────────────────────────────┘
    │
    ▼
(异常情况) 看门狗超时
    │
    ├──► 触发系统复位
    └──► 设备重启

IoTWatchDogDisable()
    │
    └──► 禁用看门狗定时器
```

## 低功耗模式切换调用链

```
用户应用
    │
    │ #include "lowpower.h"
    │
    ▼
LpcInit()
    │
    └──► 初始化低功耗管理模块

LpcSetType(type)
    │
    ├──► type == NO_SLEEP:
    │       │
    │       └──► 恢复正常时钟
    │                   └──► 所有外设使能
    │
    ├──► type == LIGHT_SLEEP:
    │       │
    │       ├──► 降低 CPU 时钟
    │       ├──► 暂停部分外设
    │       └──► 使能唤醒源 (GPIO 中断等)
    │
    └──► type == DEEP_SLEEP:
            │
            ├──► 关闭大部分外设
            ├──► 配置 RTC 唤醒
            └──► 进入休眠状态

(唤醒事件)
    │
    ├──► RTC 定时器唤醒
    ├──► GPIO 中断唤醒
    └──► 其他唤醒源
```

## 完整系统初始化流程

```
系统启动
    │
    ├──► LiteOS-M 内核初始化
    │
    ├──► 外设驱动初始化
    │       │
    │       ├──► IoTFlashInit()
    │       ├──► IoTGpioInit() × N
    │       ├──► IoTI2cInit() × M
    │       └──► IoTUartInit() × K
    │
    ├──► 看门狗使能 (可选)
    │       │
    │       └──► IoTWatchDogEnable()
    │
    └──► 进入主循环
            │
            ├──► 读取传感器 (I2C/UART)
            ├──► 处理数据
            ├──► 控制执行器 (GPIO/PWM)
            ├──► 定期喂狗 (Watchdog Kick)
            └──► 睡眠控制 (Lowpower)
```

## 相关文档

- [API 参考](01_API_Reference.md)
- [架构说明](03_Architecture.md)
- [安全评审](05_Security_Review.md)

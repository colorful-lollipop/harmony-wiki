# 配置标志与宏定义

> IoT Hardware Peripheral 子系统关键配置项汇总

## 头文件中的宏定义

### 错误码定义 (iot_errno.h)

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `IOT_SUCCESS` | `0` | 操作成功 |
| `IOT_FAILURE` | `(-1)` | 操作失败 |

**证据**: `iot_errno.h:45-51`

### GPIO 枚举值 (iot_gpio.h)

| 枚举类型 | 枚举值 | 值 | 说明 |
|----------|--------|-----|------|
| `IotGpioValue` | `IOT_GPIO_VALUE0` | 0 | 低电平 |
| | `IOT_GPIO_VALUE1` | 1 | 高电平 |
| `IotGpioDir` | `IOT_GPIO_DIR_IN` | 0 | 输入 |
| | `IOT_GPIO_DIR_OUT` | 1 | 输出 |
| `IotGpioIntType` | `IOT_INT_TYPE_LEVEL` | 0 | 电平触发 |
| | `IOT_INT_TYPE_EDGE` | 1 | 边沿触发 |
| `IotGpioIntPolarity` | `IOT_GPIO_EDGE_FALL_LEVEL_LOW` | 0 | 下降沿/低 |
| | `IOT_GPIO_EDGE_RISE_LEVEL_HIGH` | 1 | 上升沿/高 |

**证据**: `iot_gpio.h:45-80`

### UART 枚举值 (iot_uart.h)

| 枚举类型 | 枚举值 | 值 | 说明 |
|----------|--------|-----|------|
| `IotUartIdxDataBit` | `IOT_UART_DATA_BIT_5` | 5 | 5 位数据位 |
| | `IOT_UART_DATA_BIT_6` | 6 | 6 位数据位 |
| | `IOT_UART_DATA_BIT_7` | 7 | 7 位数据位 |
| | `IOT_UART_DATA_BIT_8` | 8 | 8 位数据位 |
| `IotUartStopBit` | `IOT_UART_STOP_BIT_1` | 1 | 1 位停止位 |
| | `IOT_UART_STOP_BIT_2` | 2 | 2 位停止位 |
| `IotUartParity` | `IOT_UART_PARITY_NONE` | 0 | 无校验 |
| | `IOT_UART_PARITY_ODD` | 1 | 奇校验 |
| | `IOT_UART_PARITY_EVEN` | 2 | 偶校验 |
| `IotUartBlockState` | `IOT_UART_BLOCK_STATE_NONE_BLOCK` | 0 | 非阻塞 |
| | `IOT_UART_BLOCK_STATE_BLOCK` | 1 | 阻塞 |
| `IotFlowCtrl` | `IOT_FLOW_CTRL_NONE` | 0 | 无流控 |
| | `IOT_FLOW_CTRL_RTS_CTS` | 1 | RTS/CTS |
| | `IOT_FLOW_CTRL_RTS_ONLY` | 2 | 仅 RTS |
| | `IOT_FLOW_CTRL_CTS_ONLY` | 3 | 仅 CTS |

**证据**: `iot_uart.h:50-117`

### Lowpower 枚举值 (lowpower.h)

| 枚举类型 | 枚举值 | 值 | 说明 |
|----------|--------|-----|------|
| `LpcType` | `NO_SLEEP` | 0 | 不休眠 |
| | `LIGHT_SLEEP` | 1 | 浅睡眠 |
| | `DEEP_SLEEP` | 2 | 深睡眠 |

**证据**: `lowpower.h:44-51`

## GN 构建配置

### 构建目标定义 (BUILD.gn)

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `group("iothardware")` | - | 聚合目标 |
| `ndk_lib("iothardware_ndk")` | - | NDK 库目标 |
| `head_files` | `//base/iothardware/peripheral/interfaces/inner_api` | 头文件目录 |
| `deps` | `$ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware` | HAL 依赖 |

**证据**: `BUILD.gn:1-27`

### 条件编译

| 条件 | 定义 | 说明 |
|------|------|------|
| `ohos_kernel_type == "liteos_m"` | - | 仅 LiteOS-M 内核编译 NDK |

## 组件配置 (bundle.json)

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | `@ohos/iothardware_peripheral` | 组件名称 |
| `version` | `3.1` | 组件版本 |
| `subsystem` | `iothardware` | 所属子系统 |
| `adapted_system_type` | `["mini"]` | 适配系统类型 |
| `build.sub_component` | `["//base/iothardware/peripheral:iothardware"]` | 子组件路径 |

**证据**: `bundle.json:1-28`

## 功能开关

### 内核类型判断

| 开关 | 说明 | 使用位置 |
|------|------|----------|
| `ohos_kernel_type` | OpenHarmony 内核类型 | BUILD.gn |

**常见取值**:
- `liteos_m` - LiteOS-M 内核 (Hi3861)
- `liteos_a` - LiteOS-A 内核
- `linux` - Linux 内核

### NDK 条件编译

```gn
if (ohos_kernel_type == "liteos_m") {
  ndk_lib("iothardware_ndk") {
    deps = [
      "$ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware",
    ]
    head_files = [ "//base/iothardware/peripheral/interfaces/inner_api" ]
  }
}
```

## API 版本

| 版本信息 | 值 | 说明 |
|----------|-----|------|
| 接口规范版本 | `2.2` | 基于头文件 @version 注释 |
| 子系统版本 | `3.1` | 基于 bundle.json |

## 建议配置值

### UART 波特率配置

| 波特率 | 配置值 | 应用场景 |
|--------|--------|----------|
| 9600 | `9600` | 低速设备 |
| 19200 | `19200` | 传统设备 |
| 57600 | `57600` | 通用串口 |
| 115200 | `115200` | 高速调试 |
| 230400 | `230400` | 高速通信 |
| 460800 | `460800` | 高速日志 |

### I2C 波特率配置

| 波特率 | 配置值 | 应用场景 |
|--------|--------|----------|
| 100000 | `100000` | 标准模式 |
| 400000 | `400000` | 快速模式 |

### PWM 参数配置

| 参数 | 范围 | 说明 |
|------|------|------|
| `duty` | `1-99` | 占空比百分比 |
| `freq` | 取决于芯片 | 频率 (Hz) |

## 相关文档

- [API 参考](01_API_Reference.md)
- [构建配置](04_Build_Config.md)
- [架构说明](03_Architecture.md)

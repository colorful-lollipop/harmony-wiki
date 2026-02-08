# 常见问题与调试

## 目的

本文档汇总常见构建、运行和调试问题，提供解决方案和定位路径。

## 适用范围

- 开发者遇到构建错误
- 运行时异常问题排查
- 学习调试技巧

## 关键结论

1. **构建问题**: 大多数构建问题是缺少依赖或路径错误
2. **运行时问题**: 使用 printf 日志和串口输出调试
3. **硬件问题**: 检查 GPIO 配置和硬件连接

## 常见构建问题

### Q1: 编译错误："error: 'ohos_init.h' file not found"

**错误信息**:
```
error: 'ohos_init.h' file not found
```

**原因**: OpenHarmony 系统头文件路径未正确配置。

**解决方案**:
1. 确认已正确设置开发板：
```bash
hb set -p hispark_pegasus@wifi
```

2. 清理并重新编译：
```bash
hb clean
hb build -f
```

3. 检查环境变量：
```bash
echo $OHOS_SDK
echo $OHOS_COMPILER
```

**证据**: `app/iothardware/led_example.c:18` 等文件包含 `ohos_init.h`

### Q2: 链接错误："undefined reference to 'IoTGpioInit'"

**错误信息**:
```
undefined reference to 'IoTGpioInit'
undefined reference to 'IoTGpioSetDir'
```

**原因**: peripheral 组件未链接。

**解决方案**:
1. 检查 `BUILD.gn` 中的 `include_dirs`：
```gn
static_library("led_example") {
  sources = [ "led_example.c" ]

  include_dirs = [
    "//commonlibrary/utils_lite/include",
    "//kernel/liteos_m/kal/cmsis",
    "//base/iothardware/peripheral/interfaces/inner_api",  // 确认此路径正确
  ]
}
```

2. 确认上层构建脚本包含 peripheral 组件。

**证据**: `app/iothardware/led_example.c:20` 包含 `iot_gpio.h`

### Q3: SAMGR 相关链接错误

**错误信息**:
```
undefined reference to 'SAMGR_GetInstance'
undefined reference to 'RegisterService'
```

**原因**: samgr_lite 组件未链接。

**解决方案**:
1. 检查 `bundle.json` 中的依赖：
```json
"deps": {
  "components": [
    "samgr_lite"
  ]
}
```

2. 检查 `BUILD.gn` 中的 `include_dirs`：
```gn
include_dirs = [
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
]
```

**证据**: `app/samgr/service_example.c:26` 包含 `samgr_lite.h`

## 常见运行时问题

### Q4: LED 不闪烁

**症状**: LED 不亮或不闪烁。

**可能原因**:
1. GPIO 引脚号错误
2. GPIO 未初始化
3. 硬件连接问题
4. 任务未启动

**排查步骤**:

1. **检查日志输出**:
```bash
# 串口查看日志
# 应该看到任务创建成功的日志
```

2. **检查 GPIO 配置**:
```c
// 确认 GPIO 引脚号
#define LED_TEST_GPIO 9  // for hispark_pegasus
```

3. **检查硬件连接**:
- LED 正极连接到 GPIO 引脚
- LED 负极通过电阻连接到 GND
- 电阻值建议：220Ω - 1kΩ

4. **检查任务启动**:
```c
if (osThreadNew((osThreadFunc_t)LedTask, NULL, &attr) == NULL) {
    printf("[LedExample] Failed to create LedTask!\n");
}
```

**解决方案**:
1. 修改 GPIO 引脚号以匹配实际硬件
2. 检查硬件连接
3. 添加更多日志输出

**证据**: `app/iothardware/led_example.c:25` 定义 GPIO 引脚

### Q5: SAMGR 服务注册失败

**症状**: 看不到服务注册成功的日志。

**排查步骤**:

1. **检查日志输出**:
```
[Register Test][TaskID:...][Reg Finish S:example]Time: ...!
```

2. **检查服务名称**:
```c
#define EXAMPLE_SERVICE "example"
```

3. **检查注册顺序**:
- 服务必须在特性之前注册
- 使用正确的初始化宏

**解决方案**:
1. 确认服务名称一致
2. 检查初始化宏的使用
3. 查看错误日志（如有）

**证据**: `app/samgr/service_example.c:90-96` 注册逻辑

### Q6: 广播消息未收到

**症状**: 订阅了广播但收不到消息。

**排查步骤**:

1. **检查服务是否注册**:
```bash
# 查看日志，确认广播服务已注册
```

2. **检查主题是否添加**:
```c
subscriber->AddTopic((IUnknown *)fapi, &topic0);
```

3. **检查消费者是否订阅**:
```c
subscriber->Subscribe((IUnknown *)fapi, &topic0, &c1);
```

4. **检查消息是否发布**:
```c
provider->Publish((IUnknown *)fapi, &topic0, (uint8_t *) "==>111<==", TEST_LEN);
```

**解决方案**:
1. 确认广播服务已注册
2. 确认主题已添加和订阅
3. 添加更多调试日志

**证据**: `app/samgr/broadcast_example.c:124-158` 广播示例

## 调试技巧

### 1. 使用 printf 日志

项目使用 `printf` 输出日志，可通过串口查看。

**添加日志**:
```c
printf("[DEBUG] Function: %s, Line: %d\n", __FUNCTION__, __LINE__);
printf("[DEBUG] Variable value: %d\n", variable);
```

**查看日志**:
```bash
# 串口工具（如 minicom、screen）
minicom -D /dev/ttyUSB0 -b 115200
```

**证据**: 所有源文件都使用 `printf` 输出日志

### 2. 使用 GDB 调试

OpenHarmony 支持 GDB 调试。

**编译调试版本**:
```bash
hb build -f --build-type=debug
```

**启动 GDB**:
```bash
arm-none-eabi-gdb out/hispark_pegasus/wifi_hispark_pegasus/OHOS_Image.elf
```

**常用 GDB 命令**:
```gdb
(gdb) target remote :3333
(gdb) break LedTask
(gdb) continue
(gdb) print g_ledState
(gdb) step
(gdb) backtrace
```

### 3. 查看符号表

```bash
# 查看静态库符号
arm-none-eabi-nm out/hispark_pegasus/wifi_hispark_pegasus/libs/libled_example.a

# 查看系统镜像符号
arm-none-eabi-nm out/hispark_pegasus/wifi_hispark_pegasus/OHOS_Image.elf
```

### 4. 查看反汇编

```bash
# 查看函数反汇编
arm-none-eabi-objdump -d out/hispark_pegasus/wifi_hispark_pegasus/libs/libled_example.a
```

### 5. 使用 SAMGR 维护接口

SAMGR 提供维护接口，可用于调试。

**打印所有服务**:
```c
SAMGR_PrintServices();
```

证据：`app/samgr/maintenance_example.c:10` 使用 `SAMGR_PrintServices()`

## 性能分析

### 1. 任务优先级

查看各任务优先级：

| 任务 | 优先级 | 说明 |
|------|--------|------|
| SAMGR 服务任务 | PRI_BELOW_NORMAL (20) | 默认服务优先级 |
| LED 任务 | 25 | 硬件控制任务 |
| Demo SDK 任务 | 20 | SDK 业务逻辑 |

**证据**:
- `app/iothardware/led_example.c:24` LED_TASK_PRIO = 25
- `app/demolink/demosdk.c:22` TASK_PRIO = 20
- `app/samgr/service_example.c:67` PRI_BELOW_NORMAL

### 2. 栈大小

查看各任务栈大小：

| 任务 | 栈大小 | 说明 |
|------|--------|------|
| LED 任务 | 512 字节 | 控制任务，栈需求小 |
| Demo SDK 任务 | 1000 字节 | SDK 业务逻辑 |
| SAMGR 服务任务 | 0x800 (2048) 字节 | 默认服务栈 |

**证据**:
- `app/iothardware/led_example.c:23` LED_TASK_STACK_SIZE = 512
- `app/demolink/demosdk.c:21` TASK_STACK_SIZE = 1000
- `app/samgr/service_example.c:67` 栈大小 = 0x800

### 3. 内存占用

查看静态库大小：

```bash
ls -lh out/hispark_pegasus/wifi_hispark_pegasus/libs/
```

## 常用命令

### 构建相关

```bash
# 设置开发板
hb set -p hispark_pegasus@wifi

# 编译
hb build

# 强制重新编译
hb build -f

# 清理
hb clean

# 编译特定 target
hb build -T app/demolink:example_demolink
```

### 调试相关

```bash
# 查看静态库符号
arm-none-eabi-nm out/.../libs/libexample_demolink.a

# 查看反汇编
arm-none-eabi-objdump -d out/.../libs/libexample_demolink.a

# 启动 GDB
arm-none-eabi-gdb out/.../OHOS_Image.elf
```

### 串口相关

```bash
# 使用 minicom
minicom -D /dev/ttyUSB0 -b 115200

# 使用 screen
screen /dev/ttyUSB0 115200

# 使用 cu
cu -l /dev/ttyUSB0 -s 115200
```

## 相关跳转链接

- [项目概览](01_Project_Overview.md)
- [GN 构建系统](06_GN_Build.md)
- [编译产物](07_Build_Artifacts.md)
- [安全风险评审](08_Security_Review.md)

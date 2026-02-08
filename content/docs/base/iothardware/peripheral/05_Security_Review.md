# 安全风险评审

> IoT Hardware Peripheral 子系统安全分析与风险评估

## 评审范围

### 代码范围

| 范围 | 包含 | 排除 |
|------|------|------|
| **接口层** | `interfaces/inner_api/*.h` (9 个头文件) | ❌ 测试代码 |
| **构建配置** | `BUILD.gn`, `bundle.json` | ❌ .git 目录 |
| **文档** | 本 Wiki 文档 | ❌ 第三方依赖 |

### 证据位置

- GPIO 接口: `iot_gpio.h:1-229`
- I2C 接口: `iot_i2c.h:1-115`
- UART 接口: `iot_uart.h:1-212`
- PWM 接口: `iot_pwm.h:1-94`
- Watchdog 接口: `iot_watchdog.h:1-69`
- Flash 接口: `iot_flash.h:1-110`
- Reset 接口: `reset.h:1-55`
- Lowpower 接口: `lowpower.h:1-76`
- 错误码: `iot_errno.h:1-55`

## 威胁模型

### 攻击面分析

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          攻击面分析                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   外部输入                    内部接口                      硬件接口         │
│      │                          │                           │              │
│      ├── 应用参数 ──────────────┼───────────────────────────┼── GPIO 引脚  │
│      ├── 通信数据 ─────────────┼── I2C 总线 ────────────────┼── I2C 引脚   │
│      ├── 串口数据 ◄────────────┼── UART 串口 ───────────────┼── UART 引脚  │
│      │                        ├── PWM 输出 ────────────────┼── PWM 引脚   │
│      │                        ├── Flash 操作 ──────────────┼── Flash 存储 │
│      │                        ├── 看门狗控制 ───────────────┤             │
│      │                        └── 重置/低功耗 ──────────────┤             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 说明 | 风险等级 |
|------|------|----------|
| **用户应用 ↔ 接口层** | Native C 应用调用本模块接口 | 中 |
| **接口层 ↔ HAL** | 接口定义与 HAL 实现 | 低 |
| **HAL ↔ 硬件** | HAL 驱动与硬件交互 | 低 |

## 已识别风险

### 风险 1: GPIO 缓冲区溢出（高风险）

**证据**:
```c
// iot_gpio.h - 无参数边界检查
unsigned int IoTGpioSetDir(unsigned int id, IotGpioDir dir);
// id 参数未验证范围
```

**问题描述**:
- `id` 参数无范围验证
- 超出有效范围的 `id` 可能访问非法内存

**触发条件**:
```c
// 恶意应用调用
IoTGpioSetDir(0xFFFFFFFF, IOT_GPIO_DIR_OUT);  // 非法 ID
```

**影响**:
- 内存访问违规
- 系统崩溃
- 潜在的代码执行

**修复建议**:
```c
// HAL 层实现应添加边界检查
unsigned int IoTGpioSetDir(unsigned int id, IotGpioDir dir) {
    if (id >= GPIO_MAX_COUNT) {
        return IOT_FAILURE;  // 返回错误
    }
    // ... 正常处理
}
```

**状态**: ⚠️ 需 HAL 层实现时确认

---

### 风险 2: I2C 从设备地址注入（中风险）

**证据**:
```c
// iot_i2c.h - 无地址验证
unsigned int IoTI2cWrite(unsigned int id, unsigned short deviceAddr,
                         const unsigned char *data, unsigned int dataLen);
```

**问题描述**:
- `deviceAddr` 参数无验证
- 可能访问未授权的从设备

**触发条件**:
```c
// 访问敏感从设备地址
IoTI2cWrite(0, 0x70, data, len);  // 可能访问安全芯片
```

**影响**:
- 信息泄露
- 未授权设备访问

**修复建议**:
```c
// 在 HAL 层实现访问控制列表 (ACL)
static const unsigned short ALLOWED_I2C_ADDRS[] = {
    0x20,  // 扩展IO
    0x3C,  // OLED
    // ...
};

if (deviceAddr >= ARRAY_SIZE(ALLOWED_I2C_ADDRS) ||
    deviceAddr != ALLOWED_I2C_ADDRS[i]) {
    return IOT_FAILURE;
}
```

**状态**: ⚠️ 需 HAL 层实现时确认

---

### 风险 3: UART 缓冲区溢出（中风险）

**证据**:
```c
// iot_uart.h - 返回实际字节数
int IoTUartRead(unsigned int id, unsigned char *data, unsigned int dataLen);
// dataLen 可能超出 data 缓冲区大小
```

**问题描述**:
- 应用需确保 `data` 缓冲区足够大
- 恶意应用可能传递过大的 `dataLen`

**触发条件**:
```c
unsigned char small_buf[8];
IoTUartRead(0, small_buf, 1024);  // 缓冲区溢出
```

**影响**:
- 内存覆盖
- 数据破坏

**修复建议**:
```c
// HAL 层应限制最大读取长度
#define UART_MAX_READ_LEN 256

int IoTUartRead(unsigned int id, unsigned char *data, unsigned int dataLen) {
    if (dataLen > UART_MAX_READ_LEN) {
        dataLen = UART_MAX_READ_LEN;  // 截断或返回错误
    }
    // ... 正常处理
}
```

**状态**: ⚠️ 需 HAL 层实现时确认

---

### 风险 4: Flash 写入越界（中风险）

**证据**:
```c
// iot_flash.h - 无地址范围验证
unsigned int IoTFlashWrite(unsigned int flashOffset, unsigned int size,
                           const unsigned char *ramData, unsigned char doErase);
```

**问题描述**:
- `flashOffset` 和 `size` 可能超出 Flash 实际容量
- 越界擦写可能损坏引导程序或系统区域

**触发条件**:
```c
// 擦除引导区
IoTFlashErase(0x0, 4096);  // Flash 起始地址通常是引导程序
```

**影响**:
- 系统变砖
- 安全启动失效

**修复建议**:
```c
// HAL 层实现保护区域
#define FLASH_PROTECTED_START 0x0
#define FLASH_PROTECTED_SIZE   (4 * 1024)  // 4KB 引导区

if (flashOffset < FLASH_PROTECTED_START + FLASH_PROTECTED_SIZE &&
    flashOffset + size > FLASH_PROTECTED_START) {
    return IOT_FAILURE;  // 拒绝访问保护区域
}
```

**状态**: ⚠️ 需 HAL 层实现时确认

---

### 风险 5: 看门狗滥用（低风险）

**证据**:
```c
// iot_watchdog.h - 无参数验证
void IoTWatchDogEnable(void);
void IoTWatchDogDisable(void);
```

**问题描述**:
- 恶意应用可能禁用看门狗
- 影响系统可靠性

**触发条件**:
```c
IoTWatchDogDisable();  // 禁用看门狗
```

**影响**:
- 看门狗保护失效
- 系统卡死后无法自动恢复

**修复建议**:
```c
// 考虑添加权限检查或沙箱限制
#ifdef CONFIG_SECURITY_SANDBOX
    if (!has_watchdog_permission()) {
        return IOT_FAILURE;
    }
#endif
```

**状态**: ⚠️ 需系统层面决策

---

### 风险 6: UART 数据注入（低风险）

**证据**:
```c
// iot_uart.h - 无数据过滤
int IoTUartWrite(unsigned int id, const unsigned char *data, unsigned int dataLen);
```

**问题描述**:
- 应用可通过 UART 注入恶意数据
- 可能影响连接到 UART 的外设

**触发条件**:
```c
// 发送恶意命令到外设
IoTUartWrite(0, malicious_data, len);
```

**影响**:
- 外设行为异常
- 安全敏感数据泄露

**修复建议**:
```c
// 建议在应用层实现数据验证
void safe_uart_write(unsigned int id, const char *cmd) {
    if (!validate_command(cmd)) {
        return;  // 拒绝非法命令
    }
    IoTUartWrite(id, cmd, strlen(cmd));
}
```

**状态**: ⚠️ 建议在应用层处理

---

### 风险 7: 设备重置滥用（低风险）

**证据**:
```c
// reset.h - 无条件触发
void RebootDevice(unsigned int cause);
```

**问题描述**:
- 恶意应用可能频繁触发设备重置
- 影响系统稳定性

**触发条件**:
```c
// 恶意重置循环
while (1) {
    RebootDevice(0);
}
```

**影响**:
- 拒绝服务
- 用户体验下降

**修复建议**:
```c
// 添加重置频率限制
static unsigned long last_reboot_time = 0;
#define MIN_REBOOT_INTERVAL 5000  // 5 秒

void RebootDevice(unsigned int cause) {
    unsigned long now = get_tick_count();
    if (now - last_reboot_time < MIN_REBOOT_INTERVAL) {
        return;  // 拒绝频繁重置
    }
    last_reboot_time = now;
    // ... 正常重置
}
```

**状态**: ⚠️ 需 HAL 层实现时确认

---

### 风险 8: Flash 存储敏感数据暴露（低风险）

**证据**:
```c
// iot_flash.h - 明文读写
unsigned int IoTFlashRead(unsigned int flashOffset, unsigned int size,
                          unsigned char *ramData);
```

**问题描述**:
- Flash 存储的敏感数据可能被直接读取
- 无加密保护

**触发条件**:
```c
// 读取配置区域
IoTFlashRead(CONFIG_OFFSET, CONFIG_SIZE, buffer);
// 可能包含密码、密钥等
```

**影响**:
- 敏感信息泄露
- 密钥/凭据暴露

**修复建议**:
```c
// 建议 1: 应用层加密敏感数据
unsigned char encrypt_data[MAX_SIZE];
encrypt(plain_data, encrypt_data, key);
IoTFlashWrite(offset, size, encrypt_data, 1);

// 建议 2: 考虑 Flash 硬件加密支持
#ifdef CONFIG_FLASH_ENCRYPTION
    IoTFlashWriteEncrypted(offset, size, data, key_id);
#endif
```

**状态**: ⚠️ 建议在应用层处理

---

## 风险汇总表

| ID | 风险名称 | 严重性 | 可能性 | 风险等级 | 缓解优先级 |
|----|----------|--------|--------|----------|------------|
| R1 | GPIO 缓冲区溢出 | 高 | 中 | 🟠 中 | P1 |
| R2 | I2C 从设备地址注入 | 中 | 中 | 🟠 中 | P2 |
| R3 | UART 缓冲区溢出 | 中 | 中 | 🟠 中 | P2 |
| R4 | Flash 写入越界 | 中 | 低 | 🟡 低 | P3 |
| R5 | 看门狗滥用 | 低 | 中 | 🟡 低 | P3 |
| R6 | UART 数据注入 | 低 | 中 | 🟢 很低 | P4 |
| R7 | 设备重置滥用 | 低 | 低 | 🟢 很低 | P4 |
| R8 | Flash 敏感数据暴露 | 低 | 低 | 🟢 很低 | P4 |

### 风险等级说明

| 等级 | 说明 | 响应时间 |
|------|------|----------|
| 🔴 高 | 需立即修复 | 1 周内 |
| 🟠 中 | 需近期修复 | 1 个月内 |
| 🟡 低 | 计划修复 | 下一版本 |
| 🟢 很低 | 接受或监控 | 长期 |

### 优先级说明

| 优先级 | 说明 |
|--------|------|
| P1 | 立即实施 |
| P2 | 下一版本实施 |
| P3 | 计划中 |
| P4 | 建议/可选 |

## 缓解措施总结

### 架构层面缓解

| 缓解措施 | 适用风险 | 实施位置 |
|----------|----------|----------|
| 参数边界检查 | R1, R2, R3, R4, R7 | HAL 层 |
| 访问控制列表 | R2 | HAL 层/系统 |
| 保护区域定义 | R4 | HAL 层 |
| 频率限制 | R5, R7 | HAL 层 |
| 应用层验证 | R6, R8 | 应用层 |

### 开发建议

1. **输入验证**: 所有外部输入必须在 HAL 层进行严格验证
2. **最小权限**: 应用应仅访问其需要的外设
3. **安全编码**: 遵循 MISRA-C 等安全编码规范
4. **测试覆盖**: 增加边界条件和异常输入测试

## 检查局限性

### 本次评审未覆盖

| 范围 | 原因 |
|------|------|
| HAL 实现代码 | 不在本仓库（位于 board adapter） |
| 芯片安全特性 | 需参考 Hi3861 芯片手册 |
| 系统集成安全 | 需整体系统安全评审 |
| 网络通信安全 | 本模块不涉及网络 |

## 结论

本模块（`@ohos/iothardware_peripheral`）作为纯接口定义层，**主要安全责任在于 HAL 实现层**。接口层本身设计简洁，未包含复杂业务逻辑，降低了安全风险。

**建议优先级**:

1. **P1 (立即)**: 确保 HAL 层实现包含参数边界检查
2. **P2 (近期)**: 定义 Flash 保护区域和安全 I2C 地址白名单
3. **P3 (计划)**: 添加看门狗和重置的频率限制
4. **P4 (建议)**: 在应用层实现数据加密和验证

## 相关文档

- [架构说明](03_Architecture.md)
- [构建配置](04_Build_Config.md)
- [GPIO 接口](API_GPIO.md)
- [I2C 接口](API_I2C.md)
- [UART 接口](API_UART.md)
- [Flash 接口](API_Flash.md)

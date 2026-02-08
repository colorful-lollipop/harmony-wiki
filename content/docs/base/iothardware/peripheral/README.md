# IoT Hardware Peripheral Wiki

> OpenHarmony IoT 硬件子系统接口文档

## 文档概述

本文档为 OpenHarmony IoT Hardware Peripheral 子系统的工程 Wiki，涵盖接口规范、架构设计、构建配置和安全评审。

### 覆盖范围

- ✅ **GPIO 通用输入输出接口** (`iot_gpio.h`)
- ✅ **I2C 总线接口** (`iot_i2c.h`)
- ✅ **UART 串口接口** (`iot_uart.h`)
- ✅ **PWM 脉冲宽度调制接口** (`iot_pwm.h`)
- ✅ **Watchdog 看门狗接口** (`iot_watchdog.h`)
- ✅ **Flash 存储接口** (`iot_flash.h`)
- ✅ **Reset 重置接口** (`reset.h`)
- ✅ **Lowpower 低功耗接口** (`lowpower.h`)
- ✅ **错误码定义** (`iot_errno.h`)
- ✅ **GN 构建配置** (`BUILD.gn`)
- ✅ **安全风险评审**

### 未覆盖范围

- ❌ N-API 接口（本项目为纯 C 接口，不涉及 JavaScript/TypeScript 绑定）
- ❌ IPC/SA 通信（本项目为底层硬件抽象层，无进程间通信）
- ❌ 权限鉴权（本项目直接操作硬件，无用户态权限检查）
- ❌ 测试用例代码

### 更新方式

当接口头文件或构建配置变更时，需同步更新本文档：

```bash
# 触发文档检查
python3 scripts/wiki_check.py
```

## 版本信息

| 项目 | 版本 | 说明 |
|------|------|------|
| peripheral 子系统 | 3.1 | 当前版本 |
| 接口规范 | 2.2 | OpenHarmony 2.2 |
| 文档生成时间 | 2026-02-06 | 本次 Wiki 版本 |

## 快速链接

- [README.md](README.md) - 项目快速入门
- [SUMMARY.md](SUMMARY.md) - 全站导航
- [API_Reference.md](API_Reference.md) - 接口参考
- [Architecture.md](Architecture.md) - 架构说明
- [Build_Config.md](Build_Config.md) - 构建配置
- [Security_Review.md](Security_Review.md) - 安全评审

## 贡献指南

### 文档更新规则

1. **接口变更**：新增/修改/删除 API 必须在对应文档中同步更新
2. **构建变更**：GN target 变更需在 `Build_Config.md` 中反映
3. **安全发现**：发现新风险点需在 `Security_Review.md` 中补充

### 代码证据要求

所有关键结论必须包含可追溯的代码证据：

```markdown
**证据**：`path:line` - 符号名/代码片段
```

## 常见问题

**Q: 本项目支持哪些开发板？**
A: 目前仅支持 Hi3861 开发板（liteos_m kernel）。

**Q: 如何在应用中使用这些接口？**
A: 在 `BUILD.gn` 中添加对 `iothardware_ndk` 的依赖，然后包含对应的头文件。

**Q: 为什么没有 N-API？**
A: 本项目设计为 Native C 接口，通过 NDK 直接提供给 C/C++ 应用使用。

## 联系人

- 子系统维护：iothardware@openharmony.io
- 文档问题：请提交 Issue 到仓库

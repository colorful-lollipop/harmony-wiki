# Sensor_Lite Wiki 文档

## 项目概述

本 Wiki 文档描述 OpenHarmony **sensor_lite** 组件（泛 Sensor 服务子系统轻量版）的工程细节。

**sensor_lite** 是一个面向 LiteOS-M 内核的轻量级传感器服务框架，提供：
- 传感器列表查询
- 传感器使能/去使能
- 传感器数据订阅/取消订阅
- 采样间隔与上报间隔设置
- 数据上报模式配置

## 文档覆盖范围

### 已覆盖内容

| 文档 | 状态 | 说明 |
|-----|------|------|
| [README.md](README.md) | ✅ 完成 | 本文档 |
| [SUMMARY.md](SUMMARY.md) | ✅ 完成 | 全站导航 |
| [01_Overview.md](01_Overview.md) | 🔄 进行中 | 项目概览 |
| [02_API_Reference.md](02_API_Reference.md) | ⏳ 待完成 | Native C API 文档 |
| [03_Architecture.md](03_Architecture.md) | ⏳ 待完成 | 架构设计文档 |
| [04_Build.md](04_Build.md) | ⏳ 待完成 | 构建配置与产物 |
| [05_Security.md](05_Security.md) | ⏳ 待完成 | 安全风险评审 |

### 未覆盖内容

- **JS/N-API 接口**：本项目为纯 Native C API，无 JS 接口层
- **测试代码**：根据规范，测试相关内容不纳入文档
- **完整 HDI 实现**：仅文档化 HDI 接口调用，HDI 内部实现由 `drivers/peripheral/sensor` 仓维护

## 快速导航

### 新人阅读顺序

建议按以下顺序阅读：

1. **[01_Overview.md](01_Overview.md)** - 了解项目定位、核心能力、运行环境
2. **[02_API_Reference.md](02_API_Reference.md)** - 查看 8 个 Native C API 的使用方式
3. **[03_Architecture.md](03_Architecture.md)** - 理解客户端-服务器架构、数据流
4. **[04_Build.md](04_Build.md)** - 了解编译配置与产物

### 开发者常用链接

- API 快速查询：[02_API_Reference.md](02_API_Reference.md)
- 构建配置：[04_Build.md](04_Build.md)
- 安全注意事项：[05_Security.md](05_Security.md)

## 更新说明

### 如何随代码更新文档

当代码发生变化时，请同步更新 Wiki：

1. **API 变更**：更新 `02_API_Reference.md` 中的函数签名与描述
2. **架构变更**：更新 `03_Architecture.md` 中的架构图与模块说明
3. **构建变更**：更新 `04_Build.md` 中的 targets 与依赖
4. **安全发现**：在 `05_Security.md` 中新增风险条目

### 版本信息

| 项目 | 版本 |
|-----|------|
| sensor_lite | 3.1 |
| 适应系统 | OpenHarmony LiteOS-M (mini) |
| ROM 占用 | ~92KB |
| RAM 占用 | ~200KB |

## 贡献指南

### 文档规范

- 所有关键结论必须有代码证据（文件路径 + 符号名）
- API 文档需包含完整的参数、返回值、错误码说明
- 架构文档需包含调用链图（Mermaid）
- 安全文档需包含攻击路径与修复建议

### 证据格式

```markdown
**证据**: `path/to/file.c:function_name(line)`
```

## 联系方式

- **代码仓库**: `/base/sensors/sensor_lite`
- **相关文档**: [泛Sensor子系统总览](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E6%B3%9BSensor%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- **驱动依赖**: [drivers/peripheral/sensor](https://gitee.com/openharmony/drivers_peripheral_sensor)

---

**最后更新**: 2026-02-06
**维护者**: sensor_lite 团队

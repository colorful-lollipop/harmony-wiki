# Sensors Cangjie Wrapper Wiki

## 文档说明

本文档为 **Generic Sensor Service Cangjie Wrapper** 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、使用 API、了解构建流程并进行安全评估。

### 覆盖范围

本文档涵盖以下内容：

| 文档 | 内容 |
|------|------|
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境、目录结构 |
| [02_Architecture](02_Architecture.md) | 系统架构、组件图、数据流、调用链 |
| [03_API_Reference](03_API_Reference.md) | N-API 清单、参数、返回值、错误码 |
| [04_Build_System](04_Build_System.md) | GN 构建配置、Targets 详解 |
| [05_Build_Artifacts](05_Build_Artifacts.md) | 编译产物、安装路径、运行时加载 |
| [06_Security_Review](06_Security_Review.md) | 威胁模型、攻击面、风险清单 |
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题、调试方法 |

### 未覆盖范围

- **测试代码**: 测试目录 (`test/`) 内的代码和用法
- **FFI C++ 实现**: 底层 `cj_sensor_ffi` 的内部实现（位于 `sensors_sensor` 仓库）
- **运行时配置**: 系统级传感器服务配置

### 更新方式

当代码发生变更时，需同步更新相应章节：

1. **API 变更**: 更新 `03_API_Reference.md` 的 API 清单表
2. **构建变更**: 更新 `04_Build_System.md` 和 `05_Build_Artifacts.md`
3. **架构变更**: 更新 `02_Architecture.md` 和相关调用链
4. **安全发现**: 在 `06_Security_Review.md` 中新增风险项

### 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: 基于 `sensors_cangjie_wrapper` 仓库
- **文档版本**: 1.0.0

### 相关链接

- 项目源码: `base/sensors/sensors_cangjie_wrapper`
- 官方 API 文档: [SensorServiceKit Cangjie API Reference](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/SensorServiceKit/cj-apis-sensor.md)
- 开发指南: [Cangjie Sensor Development Guide](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/device/sensor/cj-sensor-guidelines.md)

# 分布式相机部件 Wiki

> 本 Wiki 旨在帮助开发者快速理解 OpenHarmony 分布式相机部件的架构、接口和使用方法。

## 覆盖范围

### 已覆盖

| 类别 | 内容 |
|------|------|
| **项目概述** | 组件定位、核心能力、运行环境 |
| **模块结构** | 目录结构、各模块职责 |
| **架构设计** | 整体架构、数据流、线程模型、状态机 |
| **SDK 接口** | Source/Sink API、回调、错误码 |
| **构建系统** | GN Targets、编译产物、条件编译 |
| **安全机制** | 信任边界、攻击面、风险评估 |
| **调试指南** | 常见问题、调试技巧 |

### 未覆盖

| 类别 | 说明 |
|------|------|
| **测试代码** | 单元测试、Fuzz 测试不包含在内 |
| **具体实现细节** | 仅覆盖架构层面，代码级实现在代码注释中 |
| **下游 HDF 驱动** | 依赖 `drivers/peripheral/distributed_camera` |

---

## 文档更新

### 何时更新

当发生以下变更时，应更新本文档：

1. 新增或删除模块
2. 接口签名变更
3. 构建配置变更
4. 安全机制变更
5. 发现文档错误

### 更新方式

```bash
# 1. 编辑对应 .md 文件
vim wiki/xx_xxx.md

# 2. 提交变更
git add wiki/
git commit -m "docs: update wiki for xxx"

# 3. (可选) PR 到上游
```

---

## 使用说明

### 新人入门

建议按 [SUMMARY.md](SUMMARY.md) 中的阅读顺序，依次阅读核心文档。

### 快速定位

- 查找模块代码 → [01_Directory_Structure.md](01_Directory_Structure.md)
- 查找 API 定义 → [03_Inner_API.md](03_Inner_API.md)
- 查找编译配置 → [04_Build_System.md](04_Build_System.md)
- 排查问题 → [06_Troubleshooting.md](06_Troubleshooting.md)

---

## 快速链接

### 代码仓库

- **Gitee**: https://gitee.com/openharmony/distributedhardware_distributed_camera
- **GitHub**: https://github.com/openharmony/distributedhardware_distributed_camera

### 相关仓库

| 仓库 | 说明 |
|------|------|
| [distributed_hardware_fwk](https://gitee.com/openharmony/distributedhardware_distributed_hardware_fwk) | 分布式硬件框架 |
| [device_manager](https://gitee.com/openharmony/distributedhardware_device_manager) | 设备管理 |
| [distributed_screen](https://gitee.com/openharmony/distributedhardware_distributed_screen) | 分布式屏幕 |
| [camera_framework](https://gitee.com/openharmony/multimedia_camera_framework) | 相机框架 |

### 外部资源

| 资源 | 链接 |
|------|------|
| OpenHarmony 文档 | https://docs.openharmony.cn |
| API 参考 | https://docs.openharmony.cn/application-dev/api/ |
| 构建指南 | https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-distributedhardware-build.md |

---

## 生成信息

- **生成时间**: 2026-02-06
- **适用版本**: OpenHarmony 3.1+
- **最后更新**: 2026-02-06

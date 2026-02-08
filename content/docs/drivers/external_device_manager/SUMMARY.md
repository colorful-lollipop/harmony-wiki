# 文档导航

本文档为 OpenHarmony 扩展外部设备管理模块的完整技术 Wiki，按以下顺序阅读可快速掌握项目核心内容。

## 阅读路线图

### 新手入门（推荐阅读顺序）

1. **[00_Overview.md](00_Overview.md)** - 项目定位、核心能力、运行环境
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构与模块职责
3. **[02_Architecture.md](02_Architecture.md)** - 架构设计、组件关系、数据流
4. **[03_NAPI_Reference.md](03_NAPI_Reference.md)** - JS API 接口文档

### 进阶开发

5. **[04_DDK_Reference.md](04_DDK_Reference.md)** - DDK C API 接口
6. **[05_Inner_API.md](05_Inner_API.md)** - 内部模块接口
7. **[06_Build_System.md](06_Build_System.md)** - GN 构建系统

### 安全与运维

8. **[07_Security_Review.md](07_Security_Review.md)** - 安全风险分析
9. **[08_Troubleshooting.md](08_Troubleshooting.md)** - 常见问题与调试

## 快速索引

### 按功能分类

| 功能 | 文档 | 关键文件 |
|------|------|----------|
| JS API | [03_NAPI_Reference.md](03_NAPI_Reference.md) | device_manager_middle.cpp |
| DDK 接口 | [04_DDK_Reference.md](04_DDK_Reference.md) | usb_serial_api.h, scsi_peripheral_api.h |
| SA 服务 | [02_Architecture.md](02_Architecture.md) | driver_ext_mgr.h, 5110.json |
| 构建配置 | [06_Build_System.md](06_Build_System.md) | BUILD.gn, bundle.json |
| 安全分析 | [07_Security_Review.md](07_Security_Review.md) | edm_errors.h, permission checks |

### 按代码路径分类

| 目录 | 用途 | 参考文档 |
|------|------|----------|
| frameworks/js/napi | N-API 绑定层 | [03_NAPI_Reference.md](03_NAPI_Reference.md) |
| frameworks/ddk | 驱动开发套件 | [04_DDK_Reference.md](04_DDK_Reference.md) |
| interfaces/innerkits | 内部 C++ 接口 | [05_Inner_API.md](05_Inner_API.md) |
| services/native | 系统服务实现 | [02_Architecture.md](02_Architecture.md), [05_Inner_API.md](05_Inner_API.md) |
| utils | 通用工具 | [02_Architecture.md](02_Architecture.md) |

## 版本信息

- **当前版本**：4.0
- **最后更新**：2025-02-06
- **兼容版本**：OpenHarmony Standard

# OpenMAX IL Wiki

## 库概览

| 属性 | 值 |
|------|-----|
| **库名称** | OpenMAX IL (Integration Layer) |
| **上游版本** | 1.1.2 |
| **上游组织** | Khronos Group |
| **许可证** | MIT License |
| **OH 组件名** | @ohos/openmax |
| **OH 版本** | 3.1 |
| **所属子系统** | thirdparty |
| **库类型** | 纯头文件接口库 |

## OpenHarmony 适配概述

OpenMAX IL 在 OpenHarmony 中的定位是**多媒体编解码器标准接口层**：

1. **无侵入式适配**：OH 未修改上游任何头文件，保持与 Khronos 标准 100% 兼容
2. **扩展机制**：通过独立的 `codec_omx_ext.h` 添加 OH 特有功能
3. **广泛依赖**：多媒体子系统（av_codec、media_foundation、image_framework）的核心依赖
4. **硬件抽象**：作为 Codec HDI (Hardware Device Interface) 的基础，连接芯片厂商 OMX 实现

### 为什么选择 OpenMAX IL

- **行业标准**：由 Khronos Group 制定，被 Android、Linux 等广泛采用
- **硬件加速**：支持硬件编解码器的高效集成
- **可移植性**：统一的接口使应用代码跨平台
- **生态成熟**：主流芯片厂商均提供 OMX IL 实现

### OH 特有的扩展

| 扩展文件 | 功能 |
|----------|------|
| `codec_omx_ext.h` | HEVC/VVC 扩展 Profile、Buffer 类型扩展、码控增强、ROI 编码、LPP 低功耗模式 |

## 文档导航

### 快速开始

- [01_Overview.md](01_Overview.md) - 原始库简介与 OH 定位
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 谁在依赖和使用 OpenMAX

### 开发者指南

- [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 配置详解
- [05_API_Differences.md](05_API_Differences.md) - OH 扩展 API 参考

### 维护指南

- [02_Patches.md](02_Patches.md) - Patch 分析（本库无 Patch，了解原因）
- [06_Security.md](06_Security.md) - 安全风险评估

### 内部文档

- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估报告（信息收集阶段产出）
- [_work/NOTES.md](_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](_work/PLAN.md) - 任务进度

## 关键联系

- **OH 维护者**：liufeihu@huawei.com
- **上游社区**：https://github.com/KhronosGroup/OpenMAX-IL-Registry
- **规范文档**：https://www.khronos.org/registry/omxil/

## 相关资源

- [Khronos OpenMAX IL 规范](https://www.khronos.org/registry/omxil/specs/OpenMAX_IL_1_1_2_Specification.pdf)
- [OpenHarmony 多媒体子系统架构](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/media)
- [Codec HDI 接口定义](https://gitee.com/openharmony/drivers_interface/tree/master/codec)

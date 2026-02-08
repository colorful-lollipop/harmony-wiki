# QR-Code-generator OpenHarmony 适配文档

## 库概述

QR-Code-generator 是由 Project Nayuki 开发的 QR 码生成库，完全符合 ISO/IEC 18004 QR Code Model 2 标准。在 OpenHarmony 生态系统中，该库主要为 **Ace Engine UI 框架** 提供二维码生成能力，是 ArkUI QRCode 组件的核心依赖。

## OH 适配核心要点

### 适配方式：无独立 Patch，通过条件编译实现

本库**没有独立的 `.patch` 文件**，所有 OpenHarmony 适配都通过源代码中的 **`ACE_ENGINE_QRCODE_ABLE`** 条件编译宏实现，适配代码与上游代码共存。

### 主要适配内容

1. **异常处理改为返回值处理**：避免 C++ 异常抛出，使用 `flag` 成员变量记录错误状态
2. **API 可见性调整**：部分方法从 `public` 改为 `private`，简化 AceEngine 的使用
3. **构建目标分离**：`qrcodegen_static`（通用）和 `ace_engine_qrcode`（OH 定制）

## 文档导航

### 快速开始

- **[阅读指南](SUMMARY.md)** - 推荐阅读顺序

### 核心内容

- **[01_概述](01_Overview.md)** - 原始库介绍及 OH 定位
- **[02_适配分析](02_Patches.md)** - `ACE_ENGINE_QRCODE_ABLE` 宏的详细作用
- **[03_构建适配](03_Build_Integration.md)** - BUILD.gn 配置说明
- **[04_在 OH 中的使用](04_Usage_in_OH.md)** - 依赖关系和使用场景
- **[05_API 差异](05_API_Differences.md)** - OH 与上游的 API 差异
- **[06_安全风险](06_Security.md)** - 安全评估和建议

### 工作文档

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 完整评估报告
- **[_work/NOTES.md](_work/NOTES.md)** - 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** - 任务进度追踪

## 快速参考

### 主要依赖者

| 组件 | 用途 |
|------|------|
| `ace_engine` | AceEngine NG 模式 QRCode 组件 |
| `ui_lite` | 轻量级 UI 框架二维码支持 |

### 关键文件

```
third_party/qrcodegen/
├── cpp/
│   ├── qrcodegen.hpp     # 头文件（含 OH 适配）
│   └── qrcodegen.cpp     # 实现文件（含 OH 适配）
└── BUILD.gn              # OH 构建配置
```

### 典型使用方式

```cpp
#include "qrcodegen.hpp"

// 生成二维码
auto qrCode = qrcodegen::QrCode::encodeText("Hello OH", qrcodegen::QrCode::Ecc::LOW);

// 检查生成状态
if (qrCode.getFlag()) {
    // 使用二维码数据
    bool dark = qrCode.getModule(x, y);
}
```

## 版本信息

| 项目 | 版本 |
|------|------|
| 上游版本 | 1.8.0 |
| OH 组件版本 | 3.1 |
| 许可证 | MIT |

## 相关链接

- **上游仓库**：https://www.nayuki.io/page/qr-code-generator-library
- **AceEngine 文档**：foundation/arkui/ace_engine/docs/
- **QRCode 组件**：foundation/arkui/ace_engine/docs/pattern/qrcode/

# MindSpore OpenHarmony 集成文档

**版本**: v2.7.0 (OH 版本 3.1)
**上游地址**: https://gitee.com/mindspore/mindspore-lite
**许可证**: Apache License 2.0

---

## 什么是 MindSpore

MindSpore 是华为开源的端到端深度学习框架，支持云、边、端全场景部署。OpenHarmony 集成的是 **MindSpore Lite** 轻量级推理引擎，为 HarmonyOS 设备提供高效的 AI 模型推理能力。

## 在 OpenHarmony 中的定位

MindSpore Lite 是 OpenHarmony 官方的 AI 推理框架组件，提供：

- **轻量级推理**: 针对移动设备优化的神经网络推理引擎
- **MindIR 格式**: 统一的模型序列化格式，支持模型解析和优化
- **NDK API**: 供第三方应用调用的 C 语言接口
- **NNRT 集成**: 与 Neural Network Runtime 深度集成，支持硬件加速

## 核心能力

| 能力 | 描述 |
|------|------|
| **模型推理** | 支持多种 AI 模型的端到端推理 |
| **模型解析** | MindIR 格式支持，兼容 MindSpore、ONNX、TF 等格式 |
| **硬件加速** | 支持 CPU、NNRT (Neural Network Runtime) 加速 |
| **训练支持** | 提供基础训练能力 (Lite Train) |
| **跨平台** | 兼容 HarmonyOS、Android、iOS 等平台 |

## 文档导航

| 文档 | 内容 |
|------|------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议和完整目录 |
| [01_Overview.md](01_Overview.md) | 库概览、原始功能、OH 定位 |
| [02_Patches.md](02_Patches.md) | **核心文档** - 40 个 Patch 详细分析 |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配详解 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景、集成方式 |
| [05_API_Differences.md](05_API_Differences.md) | OH 新增 API、行为变更 |
| [06_Security.md](06_Security.md) | CVE 修复状态、安全建议 |

## 快速开始

### 添加依赖

```gn
# 在 BUILD.gn 中添加依赖
external_deps = [
  "mindspore:mindir_lib",  # MindIR 模型解析
]
```

### 基本使用

```c
#include "context.h"
#include "model.h"
#include "tensor.h"

int main() {
  // 创建上下文
  MSContextHandle context = MSContextCreate();
  
  // 设置设备类型
  MSContextSetDeviceTarget(context, kMSDeviceTypeCPU);
  
  // 加载模型
  MSModelHandle model = MSModelCreate();
  MSModelLoadFromFile(model, "model.ms");
  
  // 获取输入张量
  MSTensorHandle input_tensor = MSModelGetInputByIndex(model, 0);
  
  // 设置输入数据
  // ... 设置数据 ...
  
  // 执行推理
  MSModelPredict(model);
  
  // 获取输出
  MSTensorHandle output = MSModelGetOutputByIndex(model, 0);
  
  // 清理资源
  MSTensorDestroy(&output);
  MSModelDestroy(&model);
  MSContextDestroy(&context);
  
  return 0;
}
```

## 关键文件

| 文件路径 | 用途 |
|----------|------|
| `bundle.json` | OH 组件配置 |
| `BUILD.gn` | OH 构建入口 |
| `mindspore-src/source/mindspore-lite/BUILD.gn` | 核心库构建配置 |
| `mindspore-src/source/third_party/patch/` | 第三方库 Patch |

## 相关资源

- [MindSpore Lite 官方文档](https://www.mindspore.cn/lite/docs/en/master/)
- [HarmonyOS AI 能力开发指南](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/mindspore-guidelines-based-native)
- [Neural Network Runtime 开发指南](https://gitee.com/openharmony/neural_network_runtime)

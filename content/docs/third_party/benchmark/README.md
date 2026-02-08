# Google Benchmark - OpenHarmony 集成文档

## 库概述

**Google Benchmark** 是一个由 Google 开发的 C++ 微基准测试框架，用于测量代码片段的执行性能。在 OpenHarmony 生态系统中，该库作为基准测试基础设施，被 **30+** 个系统模块用于性能验证和回归测试。

## OpenHarmony 适配特点

| 适配维度 | 状态 | 说明 |
|----------|------|------|
| **Patch 修改** | ✅ 无 | 源代码未修改，保持上游版本完整性 |
| **构建适配** | ✅ GN 构建 | 提供 OHOS 专用 BUILD.gn |
| **头文件导出** | ✅ 支持 | 通过 inner_kits 对外提供 |
| **系统类型** | ✅ 支持 | small 和 standard 双系统适配 |

## 文档导航

### 核心文档

- [01_Overview](01_Overview.md) - 库功能概述与 OH 定位
- [02_Patches](02_Patches.md) - Patch 分析（本库无 Patch）
- [03_Build_Integration](03_Build_Integration.md) - 构建系统适配
- [04_Usage_in_OH](04_Usage_in_OH.md) - OH 使用场景与依赖关系

### 快速开始

```gn
# 在 BUILD.gn 中引用 benchmark
ohos_benchmarktest("my_benchmark_test") {
    sources = [ "my_test.cpp" ]
    deps = [ "//third_party/benchmark" ]
}
```

```cpp
// 在测试源文件中使用
#include "benchmark/benchmark.h"

static void BM_MyFunction(benchmark::State& state) {
    for (auto _ : state) {
        MyFunction();
    }
}
BENCHMARK(BM_MyFunction);
BENCHMARK_MAIN();
```

## 版本信息

| 项目 | 版本 |
|------|------|
| **上游版本** | v1.9.4 |
| **OH 组件版本** | 3.1 |
| **上游地址** | [google/benchmark v1.9.4](https://github.com/google/benchmark/releases/tag/v1.9.4) |
| **许可证** | Apache License V2.0 |

## 维护建议

- **上游追踪**: 定期关注 Google Benchmark releases
- **Patch 策略**: 当前无需 Patch，未来如需 OS 特定功能可考虑
- **升级优先级**: 中等（测试工具，不影响运行时功能）

# 02_Patches - Patch 分析

## Patch 摘要

**本库在 OpenHarmony 中未使用任何 Patch**，源代码保持上游 v1.9.4 版本的完整性。

```
┌─────────────────────────────────────────────────────────────┐
│                    Patch 统计信息                            │
├─────────────────────────────────────────────────────────────┤
│  Patch 文件数量:     0                                       │
│  修改文件数量:       0                                       │
│  修改函数数量:       0                                       │
│  Patch 目录:         无                                      │
└─────────────────────────────────────────────────────────────┘
```

## 2.1 无 Patch 原因分析

### 技术层面

| 原因 | 说明 |
|------|------|
| **低 OS 耦合** | Benchmark 库专注于代码执行时间测量，不涉及操作系统核心功能 |
| **POSIX 兼容** | 核心计时功能依赖 POSIX 接口，OH POSIX 兼容层已支持 |
| **纯算法库** | 不涉及文件系统、网络、图形等 OS 特定子系统 |
| **测试工具** | 仅在构建测试时链接，不影响最终产品运行 |

### 设计层面

| 考虑因素 | 说明 |
|----------|------|
| **上游维护** | Google 积极维护，减少 OH 自定义修改有利于后续升级 |
| **标准化** | 使用上游标准版本便于社区对标和复用 |
| **最小干预** | 避免 Patch 带来的维护复杂度和潜在冲突 |

## 2.2 潜在 Patch 需求场景

虽然当前无需 Patch，但以下场景可能需要考虑:

### 场景 1: OH 特定计时器

```cpp
// 如需使用 OH 高精度计时器替代 POSIX timer
// 可能需要修改 src/timers.cc

#ifdef OHOS
#include <hiview/timer_adapter.h>
#else
#include <chrono>
#endif
```

**当前状态**: POSIX timer 已满足需求

### 场景 2: OH 内存统计

```cpp
// 如需集成 OH 内存分配追踪
// 可能需要修改 src/counter.cc

#ifdef OHOS
void TrackMemoryAllocation() {
    // OH 内存追踪接口
}
#endif
```

**当前状态**: 标准 malloc/free 统计已满足

### 场景 3: 输出重定向

```cpp
// 如需将测试结果输出到 OH 日志系统
// 可能需要修改 src/console_reporter.cc

#ifdef OHOS
void WriteToHiView(const std::string& output) {
    // OH 日志接口
}
#endif
```

**当前状态**: 标准输出/文件输出已满足

## 2.3 Patch 维护策略

### 未来如果需要 Patch

| 策略 | 说明 |
|------|------|
| **按需创建** | 仅在确实存在 OH 特定需求时创建 |
| **最小修改** | 每个 Patch 仅解决一个问题 |
| **文档同步** | 在本 Wiki 中详细记录 |
| **上游推送** | 评估通用性，积极推向上游 |

### Patch 命名规范

如果未来创建 Patch，建议遵循:

```
0001-功能描述.patch
0002-问题修复.patch
...
```

### Patch 管理位置

```
third_party/benchmark/
├── patches/                    # OH 专用 Patch 目录
│   ├── 0001-xxx.patch
│   └── 0002-xxx.patch
└── BUILD.gn                    # 构建配置
```

## 2.4 上游版本升级注意事项

### 升级检查清单

| 检查项 | 状态 |
|--------|------|
| **源代码变更** | 需对比 upstream diff |
| **API 兼容性** | 检查 breaking changes |
| **CMakeLists.txt** | 是否需要更新 BUILD.gn |
| **依赖变化** | 新增/删除的依赖 |
| **测试兼容性** | benchmarktest 模板是否需要调整 |

### 升级流程建议

```bash
# 1. 备份当前版本
git tag oh_1.9.4

# 2. 获取上游新版本
git remote add upstream https://github.com/google/benchmark.git
git fetch upstream
git checkout vX.X.X

# 3. 合并到 OH 分支
git checkout third_party/benchmark
git merge vX.X.X

# 4. 验证构建
# 执行 OH 构建验证

# 5. 验证测试
# 运行相关 benchmarktest
```

## 2.5 总结

| 维度 | 结论 |
|------|------|
| **Patch 数量** | 0 |
| **代码完整性** | 100% 保持上游版本 |
| **OH 定制化** | 仅构建系统适配 |
| **升级复杂度** | 低 |
| **长期维护建议** | 继续保持无 Patch 策略 |

### 关键建议

1. **持续监控上游** - 关注安全修复和重要功能更新
2. **按需引入 Patch** - 仅在有明确 OH 特定需求时创建
3. **保持简洁** - 避免过度定制带来的维护负担

# JSON for Modern C++ - OpenHarmony 集成文档

## 库概述

**JSON for Modern C++** 是 Niels Lohmann 开发的一个流行的 C++ JSON 库，提供直观的语法、简单的集成方式和内存效率特性。该库是一个 **header-only** 库，只需包含单个头文件 `json.hpp` 即可使用。

在 OpenHarmony 中，该库位于 `third_party/json` 目录，版本为 3.1（基于上游 3.11.3 版本）。

## OpenHarmony 适配状态

| 状态 | 描述 |
|------|------|
| ✅ **已适配** | 完成 OH 构建系统适配 |
| ✅ **无 Patch** | 无需任何 OH 特定修改 |
| ✅ **静态库** | 提供静态库目标便于依赖管理 |

## 文档导航

### 快速开始

```cpp
// 包含头文件
#include <nlohmann/json.hpp>

// 使用命名空间
using json = nlohmann::json;

// 创建 JSON 对象
json j = {
    {"name", "OpenHarmony"},
    {"version", 3.1},
    {"features", {"分布式", "原生支持"}}
};

// 序列化
std::string str = j.dump();

// 解析
json parsed = json::parse(str);
```

### 核心文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库功能介绍 |
| [02_Patches.md](02_Patches.md) | OH Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 使用情况和依赖关系 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异分析（本库无差异） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

## 关键特性

- **Header-Only**: 无需链接库文件
- **直观的语法**: 类似 Python 的 JSON 操作体验
- **完整的功能**: 解析、序列化、JSON Pointer、JSON Patch
- **高性能**: 经过广泛测试和优化
- **跨平台**: 支持多种编译器和操作系统

## OH 集成信息

| 项目 | 值 |
|------|-----|
| **OH 组件** | @ohos/json |
| **版本** | 3.1 |
| **子系统** | thirdparty |
| **构建目标** | nlohmann_json_static |
| **许可证** | MIT |

## 与上游的差异

本库在 OpenHarmony 中**没有进行任何 Patch 修改**。原因是：

1. 该库是 header-only 设计，不包含平台特定代码
2. 原生支持多种编译器和操作系统
3. 所有功能都是标准 C++ 实现

因此，在 OH 中使用的版本与上游版本完全一致。

## 版本对应关系

| OH 版本 | 上游版本 | 适配说明 |
|---------|---------|---------|
| 3.1 | 3.11.3 | 当前版本，无 Patch |

## 维护建议

1. **升级策略**: 可直接同步上游最新版本
2. **测试重点**: 验证 OH 模块的兼容性
3. **监控事项**: 关注上游的 breaking changes

## 相关资源

- **上游仓库**: https://github.com/nlohmann/json
- **上游文档**: https://json.nlohmann.me/
- **OH 仓库**: third_party/json

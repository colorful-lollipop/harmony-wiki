# 阅读路线建议

**概述**: 本文档提供 c-ares OpenHarmony 集成文档的阅读指南。

---

## 按角色阅读

### 角色一：系统开发者

如果你负责在 OpenHarmony 上开发网络相关功能：

**必读文档**（按优先级）：

1. **[01_Overview.md](01_Overview.md)** - 了解 c-ares 在 OH 中的定位和作用
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解依赖关系和使用场景
3. **[05_API_Differences.md](05_API_Differences.md)** - 掌握 OH 扩展的 API

**按需阅读**：

- **[02_Patches.md](02_Patches.md)** - 需要修改 c-ares 时参考
- **[03_Build_Integration.md](03_Build_Integration.md)** - 需要修改构建配置时参考

### 角色二：应用开发者

如果你在 OpenHarmony 上开发应用：

**必读文档**：

1. **[01_Overview.md](01_Overview.md)** - 了解 c-ares 基本功能
2. **[05_API_Differences.md](05_API_Differences.md)** - 使用 OH 特定 API（如 `ares_set_dns_netid`）

**推荐阅读**：

- **[06_Security.md](06_Security.md)** - 了解安全注意事项

### 角色三：架构师/技术负责人

如果你负责技术选型或架构设计：

**必读文档**：

1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 完整的项目评估
2. **[02_Patches.md](02_Patches.md)** - 了解 OH 特定修改的技术细节
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解依赖关系图

**按需阅读**：

- **[03_Build_Integration.md](03_Build_Integration.md)** - 了解构建配置差异

### 角色四：测试工程师

如果你负责测试或质量保障：

**必读文档**：

1. **[06_Security.md](06_Security.md)** - 了解安全测试重点
2. **[02_Patches.md](02_Patches.md)** - 回归测试的 Patch 清单

**推荐阅读**：

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 了解已识别的风险点

---

## 按任务类型阅读

### 任务类型 1：集成 c-ares

**阅读路径**：

1. [01_Overview.md](01_Overview.md) - 了解库功能
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看依赖者使用方式
3. [05_API_Differences.md](05_API_Differences.md) - 了解 OH 扩展 API
4. [03_Build_Integration.md](03_Build_Integration.md) - 配置构建依赖

### 任务类型 2：升级 c-ares

**阅读路径**：

1. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看 OH 版本和上游版本对比
2. [02_Patches.md](02_Patches.md) - 查看需要移植的 Patch
3. [06_Security.md](06_Security.md) - 检查已修复的 CVE

### 任务类型 3：故障排查

**阅读路径**：

1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看依赖关系图
2. [03_Build_Integration.md](03_Build_Integration.md) - 检查构建配置
3. [05_API_Differences.md](05_API_Differences.md) - 确认 API 使用正确性

### 任务类型 4：性能优化

**阅读路径**：

1. [02_Patches.md](02_Patches.md) - 了解缓存和指标机制
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看性能特征分析

---

## 按文档类型分类

### 概览文档

| 文档 | 目标读者 | 核心内容 |
|-----|----------|---------|
| [README.md](README.md) | 所有读者 | 文档导航和快速链接 |
| [SUMMARY.md](SUMMARY.md) | 新读者 | 阅读路线建议 |
| [01_Overview.md](01_Overview.md) | 所有读者 | c-ares 基础介绍 |

### 技术文档

| 文档 | 目标读者 | 核心内容 |
|-----|----------|---------|
| [02_Patches.md](02_Patches.md) | 开发者 | OH 修改详细分析 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建工程师 | GN 构建配置详解 |
| [05_API_Differences.md](05_API_Differences.md) | 应用开发者 | API 差异说明 |

### 生态文档

| 文档 | 目标读者 | 核心内容 |
|-----|----------|---------|
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 所有读者 | 依赖关系和使用场景 |

### 安全文档

| 文档 | 目标读者 | 核心内容 |
|-----|----------|---------|
| [06_Security.md](06_Security.md) | 安全工程师 | 安全风险和 CVE 分析 |

### 工作文档

| 文档 | 目标读者 | 核心内容 |
|-----|----------|---------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 维护者 | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 贡献者 | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 贡献者 | 任务进度跟踪 |

---

## 常见问题

### Q1: 如何判断我的应用是否使用了 c-ares？

**A**: 检查你的应用或依赖库是否链接了 `libc_ares.so` 或 `libc_ares.a`：
```bash
# 查看动态库依赖
readelf -d your_app | grep cares

# 查看静态库符号
nm your_app | grep ares_
```

### Q2: 如何启用 OpenHarmony 特定功能？

**A**: 确保编译时定义了 `OHOS_DNS_PROXY_BY_NETSYS` 宏，并链接了正确版本的库：
```c
// 使用 OH API
#include <ares.h>
ares_channel_t *channel;
ares_init(&channel);

// 设置网络 ID
ares_set_dns_netid(channel, my_netid);
```

### Q3: 如何报告 c-ares 相关问题？

**A**: 根据问题类型选择渠道：
- **功能问题**: 检查 [06_Security.md](06_Security.md) 是否为已知问题
- **OH 特定问题**: 向 OpenHarmony 社区报告
- **上游 bug**: 在 https://github.com/c-ares/c-ares/issues 提交

### Q4: 如何获取更多帮助？

**A**:
1. 查看本仓库的其他文档
2. 阅读 c-ares 上游文档：https://c-ares.org/docs.html
3. 加入 OpenHarmony 社区讨论

---

## 文档维护计划

| 任务 | 状态 | 计划日期 |
|-----|------|---------|
| 完成 Phase 0-2 | ✅ | 2026-02-08 |
| 完成 Phase 3 | ✅ | 2026-02-08 |
| 完成 Phase 4 | 🔄 进行中 | 2026-02-08 |
| 完成 Phase 5 | ⏳ 待开始 | - |
| 定期更新 | ⏳ | 按需 |

---

**最后更新**: 2026-02-08

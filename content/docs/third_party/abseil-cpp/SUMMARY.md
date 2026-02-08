# 阅读路线建议

本文档提供针对不同角色和目标的阅读路线，帮助您快速找到所需信息。

---

## 按角色分类

### 1. 新手 / 想快速了解

**阅读顺序**：
1. [README.md](./README.md) - 快速了解库的整体情况
2. [01_Overview.md](./01_Overview.md) - 了解原始库功能和 OH 中的作用
3. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看详细评估结果

**预计时间**: 15-20 分钟

---

### 2. 开发者 / 需要使用库

**阅读顺序**：
1. [README.md](./README.md) - 了解库的适配特点
2. [01_Overview.md](./01_Overview.md) - 了解可用功能模块
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 看看 OH 中如何使用
4. [03_Build_Integration.md](./03_Build_Integration.md) - 了解如何在自己的模块中依赖 abseil-cpp

**预计时间**: 30-40 分钟

---

### 3. 维护者 / 需要修改集成

**阅读顺序**：
1. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 了解当前的适配状态
2. [02_Patches.md](./02_Patches.md) - **核心文档**，理解所有 `__OHOS__` 宏使用
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建配置
4. [06_Security.md](./06_Security.md) - 了解安全加固措施
5. [05_API_Differences.md](./05_API_Differences.md) - 了解 API 差异（如有）

**预计时间**: 60-90 分钟

---

### 4. 构建工程师

**阅读顺序**：
1. [03_Build_Integration.md](./03_Build_Integration.md) - 主要参考文档
2. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 了解库目标结构
3. [02_Patches.md](./02_Patches.md) - 了解平台特定条件编译

**预计时间**: 30-45 分钟

---

### 5. 安全工程师

**阅读顺序**：
1. [06_Security.md](./06_Security.md) - 安全风险分析
2. [02_Patches.md](./02_Patches.md) - 了解哪些功能被禁用（可能影响安全）
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解 PAC-RET 等安全加固

**预计时间**: 45-60 分钟

---

### 6. 架构师 / 技术决策者

**阅读顺序**：
1. [README.md](./README.md) - 快速概览
2. [01_Overview.md](./01_Overview.md) - 理解库的定位
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 理解依赖关系和使用场景
4. [02_Patches.md](./02_Patches.md) - 理解功能限制
5. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 查看完整评估

**预计时间**: 60-90 分钟

---

## 按任务分类

### 任务 1：评估是否使用 abseil-cpp

**阅读顺序**：
1. [01_Overview.md](./01_Overview.md) - 了解库功能
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 看 OH 中已有的使用场景
3. [README.md](./README.md) - 了解适配限制

**决策点**：
- 需要的功能是否可用？（注意调试功能限制）
- 是否影响现有依赖关系？
- 是否满足许可证要求（Apache 2.0）

---

### 任务 2：集成 abseil-cpp 到新模块

**阅读顺序**：
1. [03_Build_Integration.md](./03_Build_Integration.md) - 了解如何依赖
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 参考现有使用方式
3. [01_Overview.md](./01_Overview.md) - 选择需要的组件

**关键点**：
- 使用 `external_deps` 引用 abseil-cpp 库
- 注意静态 vs 共享库的选择
- 考虑 `innerapi_tags` 设置

---

### 任务 3：升级 abseil-cpp 版本

**阅读顺序**：
1. [02_Patches.md](./02_Patches.md) - **关键**，了解所有 `__OHOS__` 使用
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建配置差异
3. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 了解当前技术债务
4. [06_Security.md](./06_Security.md) - 检查安全加固是否受影响

**步骤**：
1. 检查新版本是否兼容所有 `__OHOS__` 条件
2. 重新验证 BUILD.gn 配置
3. 测试所有依赖模块（profiler、grpc、protobuf）
4. 更新文档中的版本号

---

### 任务 4：修复上游 Bug 或添加新功能

**阅读顺序**：
1. [02_Patches.md](./02_Patches.md) - 了解现有适配模式
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建系统
3. [05_API_Differences.md](./05_API_Differences.md) - 了解 OH 特有 API（如有）

**关键点**：
- 优先实现 OH 原生版本而非禁用功能
- 添加新的 `__OHOS__` 宏使用时更新文档
- 保持与上游的兼容性

---

### 任务 5：安全审计

**阅读顺序**：
1. [06_Security.md](./06_Security.md) - 安全风险分析
2. [02_Patches.md](./02_Patches.md) - 被禁用的功能可能引入的风险
3. [03_Build_Integration.md](./03_Build_Integration.md) - 安全加固措施（PAC-RET）

**检查点**：
- PAC-RET 覆盖是否足够？
- 禁用的调试功能是否影响安全分析？
- `innerapi_tags` 设置是否合适？

---

## 快速参考

### 文档对比

| 文档 | 篇幅 | 核心内容 | 优先级 |
|------|------|---------|--------|
| README.md | 短 | 概览、导航、核心信息 | ⭐⭐⭐ |
| SUMMARY.md | 短 | 阅读路线 | ⭐⭐ |
| 01_Overview.md | 中 | 原始库简介、OH 定位 | ⭐⭐ |
| 02_Patches.md | **长** | **Patch/适配详细分析** | ⭐⭐⭐ |
| 03_Build_Integration.md | 中 | 构建系统适配 | ⭐⭐ |
| 04_Usage_in_OH.md | 中 | 依赖关系与使用场景 | ⭐⭐ |
| 05_API_Differences.md | 短 | API 差异（如有） | ⭐ |
| 06_Security.md | 中 | 安全风险分析 | ⭐⭐ |
| _work/ASSESSMENT.md | **长** | 项目评估结果 | ⭐⭐⭐ |

---

## 常见问题

### Q: 我只想知道 OH 中如何使用 abseil-cpp？

**A**: 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md)，查看现有模块的使用方式和依赖关系。

### Q: 我想了解 OH 对 abseil-cpp 做了哪些修改？

**A**: 阅读 [02_Patches.md](./02_Patches.md)，这是核心文档，详细说明了所有 `__OHOS__` 适配。

### Q: 我需要在新的 BUILD.gn 中依赖 abseil-cpp，如何做？

**A**: 阅读 [03_Build_Integration.md](./03_Build_Integration.md)，查看库目标列表和依赖方式。

### Q: 升级 abseil-cpp 版本时需要注意什么？

**A**:
1. 阅读 [02_Patches.md](./02_Patches.md)，了解所有 `__OHOS__` 使用
2. 阅读 [_work/ASSESSMENT.md](_work/ASSESSMENT.md)，了解当前技术债务
3. 重新测试所有依赖模块（profiler、grpc、protobuf）

### Q: abseil-cpp 在 OH 中有什么功能限制？

**A**: 阅读 [README.md](./README.md) 的"适配特点"部分，或查看 [_work/ASSESSMENT.md](_work/ASSESSMENT.md) 的"功能状态"表。

---

**最后更新**: 2026-02-07

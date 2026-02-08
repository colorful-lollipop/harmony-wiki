# accesscontrol_cangjie_wrapper Wiki

**生成时间**: 2025-02-06
**项目版本**: 6.1 (Beta)
**文档语言**: 中文

---

## 文档覆盖范围

本文档集涵盖 `accesscontrol_cangjie_wrapper` 项目的以下内容：

- ✅ 项目定位与边界
- ✅ 目录结构与模块职责
- ✅ 系统架构与数据流
- ✅ 对外 Cangjie API 清单
- ✅ 内部模块接口与依赖
- ✅ GN 构建目标与编译产物
- ✅ 安全风险分析与建议
- ✅ 常见问题排查指南

## 未覆盖范围

以下内容暂不包含在本文档中：

- ❌ 测试代码说明（test/ 目录）
- ❌ CI/CD 配置（如有）
- ❌ 性能基准测试结果
- ❌ 第三方库内部实现细节

## 如何随代码更新文档

当项目代码发生变更时，建议按以下步骤更新文档：

1. **代码结构变更** → 更新 `02_Directory_Structure.md`
2. **API 新增/修改** → 更新 `04_Public_API.md` 和 `05_Internal_API.md`
3. **GN 构建变更** → 更新 `06_GN_Targets.md` 和 `07_Build_Artifacts.md`
4. **安全机制调整** → 更新 `08_Security_Review.md`
5. **新增故障案例** → 更新 `09_Troubleshooting.md`

更新文档时，请确保：
- 所有关键结论都有代码证据（路径:行号）
- 术语保持一致性
- 不引用测试代码作为业务证据
- 更新本文件的"生成时间"

## 文档阅读路线

**新人推荐阅读顺序**：

1. `00_Overview.md` - 快速了解项目全貌
2. `01_Positioning_Boundaries.md` - 理解项目边界和核心能力
3. `02_Directory_Structure.md` - 熟悉代码组织
4. `03_Architecture.md` - 理解系统架构和依赖
5. `04_Public_API.md` - 学习对外 API 使用
6. `08_Security_Review.md` - 了解安全注意事项

**开发者参考顺序**：

1. `06_GN_Targets.md` - 理解构建系统
2. `05_Internal_API.md` - 深入内部实现
3. `07_Build_Artifacts.md` - 了解编译产物
4. `09_Troubleshooting.md` - 排查常见问题

## 相关资源

- **项目 README**: `README.md` / `README_zh.md`
- **工作笔记**: `wiki/_work/NOTES.md` - 代码证据索引
- **任务计划**: `wiki/_work/PLAN.md` - 任务分解与进度
- **OpenHarmony 文档**: [OpenHarmony 官方文档](https://docs.openharmony.cn/)

## 反馈与贡献

如发现文档错误或有改进建议，请通过以下方式反馈：
1. 提交 Issue 到项目仓库
2. 参与代码贡献（参见 [Code Contribution](https://gitcode.com/openharmony/docs/blob/master/en/contribute/code-contribution.md)）

---

**注意**: 本项目为 Beta 特性，API 可能发生变化。请关注官方发布说明以获取最新信息。

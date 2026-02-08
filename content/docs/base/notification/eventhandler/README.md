# EventHandler Wiki

**生成时间**: 2026-02-07
**项目**: @ohos/eventhandler
**子系统**: notification
**版本**: 3.1
**许可证**: Apache License 2.0

## 覆盖范围

本 Wiki 文档完整覆盖了 EventHandler 部件的以下内容：
- 项目定位与核心能力
- 目录结构与模块职责
- 内部架构与线程模型
- 对外 N-API 接口
- 内部 API 接口
- **攻击面分析** (N-API/ANI/FFI/C API 入口、敏感操作)
- GN 构建目标与编译产物
- 安全风险评审
- 常见构建与调试问题

## 受众与阅读路线

本文档为两类受众提供专门的阅读路线：

### 新人学习者
关注：项目定位、快速上手、架构理解、API 使用  
阅读路线：[SUMMARY.md](SUMMARY.md) → 路线 A

### 安全研究员
关注：攻击面、信任边界、输入处理、潜在漏洞  
阅读路线：[SUMMARY.md](SUMMARY.md) → 路线 B  
重点文档：[05_AttackSurface.md](05_AttackSurface.md)、[08_Security_Review.md](08_Security_Review.md)

## 更新方式

当代码库变更时，建议按以下步骤更新文档：
1. 更新 `wiki/_work/NOTES.md` 记录新发现的符号、文件和事实
2. 更新对应的主题文档，添加新的 API 或架构变更
3. 更新 `wiki/SUMMARY.md` 确保导航链接正确
4. 运行测试验证变更
5. 更新本文档的生成时间

## 未覆盖范围

- 测试代码（test/、fuzztest/ 等目录）
- 详细的 API 使用示例（请参考官方文档）
- 与其他子系统的集成细节

## 贡献指南

如需添加或修改 Wiki 内容：
1. 所有结论必须基于代码证据（包含文件路径、行号、符号名）
2. 避免引用测试代码作为业务逻辑证据
3. 使用中文撰写，保持术语一致性
4. 更新相关链接确保导航正常工作

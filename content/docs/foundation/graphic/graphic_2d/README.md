# graphic_2d Wiki 文档

## 文档说明

本 Wiki 是 OpenHarmony `graphic_2d` 子系统的工程文档，旨在帮助开发者快速理解项目架构、API 接口、构建系统和安全风险。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 定位、边界、核心能力 |
| 目录结构 | ✅ 完成 | 模块职责划分 |
| 架构说明 | ✅ 完成 | 组件图、数据流、线程模型 |
| 对外 N-API | ✅ 完成 | JS API 清单、绑定位置 |
| 内部 Inner API | ✅ 完成 | 模块接口、依赖方向 |
| GN 构建 | ✅ 完成 | Targets、依赖、产物 |
| 安全评审 | ✅ 完成 | 攻击面、风险点 |
| 常见问题 | ✅ 完成 | 构建/运行/调试问题 |
| 关键调用链 | ✅ 完成 | 附录：调用链图示 |
| 配置开关 | ✅ 完成 | 附录：Feature Flags |

### 文档更新

- **最后更新**: 2026-02-06
- **更新方式**: 手动更新，随代码变更同步
- **代码版本**: graphic_2d v3.1 (bundle.json)

### 阅读建议

1. **新人入门**: 按 `SUMMARY.md` 中的顺序阅读
2. **API 开发者**: 直接跳转到 `04_N-API.md`
3. **Native 开发者**: 查看 `05_Inner_API.md` 和 `03_Architecture.md`
4. **构建/移植**: 参考 `06_Build.md`
5. **安全评审**: 查看 `07_Security.md`

### 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [graphic_2d 源码仓库](https://gitee.com/openharmony/graphic_graphic_2d)
- [Rosen 渲染框架设计文档](./figures/graphic_rosen_architecture.jpg)

---

## 贡献指南

### 如何更新文档

1. 在 `wiki/_work/NOTES.md` 中记录发现
2. 更新对应的 Wiki Markdown 文件
3. 确保所有关键结论有代码证据（文件路径 + 符号名）
4. 更新 `SUMMARY.md` 的导航链接

### 质量要求

- 关键结论必须可追溯到代码证据
- N-API 文档必须包含 API 清单表
- GN 文档必须包含 target 列表
- 安全文档必须包含可利用点证据

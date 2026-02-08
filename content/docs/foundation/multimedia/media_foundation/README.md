# HiStreamer Wiki 文档

## 文档覆盖范围

本文档覆盖 OpenHarmony 多媒体子系统 **media_foundation** 组件（HiStreamer 媒体引擎）的完整技术文档。

### 覆盖内容

| 分类 | 内容 |
|------|------|
| **项目定位** | 模块职责、核心能力、运行环境 |
| **架构设计** | 三层架构（应用场景层、Pipeline框架层、插件层） |
| **对外接口** | C API（native_av*）、NDK 接口、Inner API |
| **内部机制** | Pipeline 框架、插件系统、Filter 工厂 |
| **构建配置** | GN Targets、Feature Flags、编译产物 |
| **安全评估** | 攻击面分析、风险点识别 |

### 未覆盖内容

- 测试相关代码（test/ tests/ unittest/ *_test.*）
- N-API 绑定层（位于 player_framework 仓库）
- 第三方依赖内部实现（FFmpeg、curl 等）

## 更新方式

当代码发生以下变更时，需要同步更新文档：

1. 新增/删除/修改 API 接口
2. 新增/删除/修改插件
3. 变更构建配置（新增 feature flag）
4. 架构重构（新增模块、变更数据流）

### 更新步骤

```bash
# 1. 更新 NOTES.md 记录变更
# 2. 修改对应的 wiki/*.md 文件
# 3. 更新 SUMMARY.md 确保链接正确
# 4. 提交 PR
```

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: media_foundation @HEAD
- **文档版本**: 1.0

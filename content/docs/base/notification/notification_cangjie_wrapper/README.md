# notification_cangjie_wrapper Wiki

## 文档覆盖范围

本 Wiki 旨在为 OpenHarmony `notification_cangjie_wrapper` 模块提供完整的工程文档，帮助开发者快速理解项目架构、使用方法和安全注意事项。

### 已覆盖内容

- **项目概述**: 模块定位、核心能力、运行环境
- **目录结构**: ohos 目录下的模块划分与职责
- **架构设计**: Cangjie FFI 绑定机制、模块依赖关系
- **API 参考**: CommonEventManager 及相关数据类 API 详解
- **构建配置**: GN 构建目标、编译产物说明
- **安全评审**: 威胁模型、攻击面分析、风险建议

### 未覆盖内容

- 底层 `common_event_service` C++ 实现细节 (位于独立仓库)
- ArkCompiler Cangjie 运行时内部机制
- 系统级 Common Event Service 架构

## 文档更新方式

### 何时更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. **API 变更**: 新增、修改或删除公共 API
2. **架构变更**: 模块拆分、合并或依赖关系变化
3. **构建变更**: 新增 BUILD.gn target、修改编译配置
4. **安全修复**: 发现并修复安全漏洞
5. **错误码变更**: 新增或修改错误码

### 更新责任

- **API 变更**: 模块所有者负责更新 API 参考章节
- **架构变更**: 架构师或模块所有者负责更新架构章节
- **安全评审**: 安全工程师定期审查并更新安全章节

## 生成信息

- **生成时间**: 2026-02-06
- **仓库**: base/notification/notification_cangjie_wrapper
- **版本**: 6.1
- **API Level**: 22+

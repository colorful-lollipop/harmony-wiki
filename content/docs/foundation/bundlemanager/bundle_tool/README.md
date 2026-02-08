# bundle_tool (bm) Wiki 文档

> OpenHarmony Bundle 管理命令行工具 Wiki

## 文档覆盖范围

本 Wiki 文档覆盖 OpenHarmony `foundation/bundlemanager/bundle_tool` 项目的完整技术细节，包括：

### 涵盖内容

- **项目概览**: 定位、边界、运行环境、核心能力
- **架构设计**: 组件图、数据流、线程模型、命令注册机制
- **命令行接口**: 所有命令的用法、参数、示例
- **内部 API**: 模块接口、依赖方向、稳定性
- **GN 构建**: targets 列表、依赖关系、编译产物
- **安全风险**: 攻击面分析、信任边界、安全风险点
- **问题定位**: 常见构建/运行/调试问题

### 未涵盖内容

- **测试相关内容**: test/ 目录下的所有代码和文档
- **N-API 接口**: 本项目为纯 Native CLI 工具，不提供 JS/N-API
- **运行时内部逻辑**: bundle_framework 子系统的内部实现（仅涉及调用关系）

---

## 文档更新方式

### 何时更新 Wiki

当发生以下变更时，需要同步更新 Wiki：

1. 新增命令行命令
2. 修改命令参数或行为
3. 新增/删除 GN targets
4. 变更模块依赖关系
5. 新增安全风险点或修复建议

### 更新流程

```bash
# 1. 克隆仓库
git clone https://gitee.com/openharmony/bundlemanager_bundle_framework.git
cd bundlemanager_bundle_framework/bundle_tool

# 2. 更新对应文档
#    - 命令变更 → 更新 02_Command_Reference.md
#    - 构建变更 → 更新 04_Build.md
#    - 安全变更 → 更新 05_Security.md

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for xxx"
```

### 版本兼容性

| Wiki 版本 | OpenHarmony 版本 | 说明 |
|-----------|-----------------|------|
| 1.0 | OpenHarmony 4.0+ | 初始版本 |

---

## 生成信息

| 属性 | 值 |
|------|------|
| 项目 | bundle_tool (bm) |
| 仓库路径 | foundation/bundlemanager/bundle_tool |
| 文档版本 | 1.0 |
| 生成时间 | 2024-XX-XX |
| 维护者 | bundlemanager 团队 |

---

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [Bundle Manager 子系统](https://gitee.com/openharmony/bundlemanager_bundle_framework)
- [Bundle Framework 接口](https://gitee.com/openharmony/bundlemanager_bundle_framework/tree/master/interfaces)
- [HDC 工具](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V3/hdc-V3)

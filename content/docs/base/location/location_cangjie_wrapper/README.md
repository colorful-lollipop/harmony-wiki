# Location Cangjie Wrapper Wiki

> OpenHarmony 位置服务 Cangjie API 封装工程文档

## 文档概述

本文档是 `location_cangjie_wrapper` 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建配置和安全特性。

### 覆盖范围

| 分类 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 已覆盖 | 项目定位、能力边界、运行环境 |
| 目录结构 | ✅ 已覆盖 | kit/LocationKit, ohos/geo_location_manager |
| 系统架构 | ✅ 已覆盖 | 接口层、框架层、依赖组件 |
| N-API/Cangjie API | ✅ 已覆盖 | 核心类、枚举、请求配置 |
| 构建配置 | ✅ 已覆盖 | BUILD.gn targets、产物映射 |
| 安全评审 | ✅ 已覆盖 | 攻击面、风险点、修复建议 |

### 未覆盖范围

- 测试用例细节（`test/` 目录）
- C/C++ FFI 实现细节（位于 `base_location` 仓库）
- 运行时动态加载机制
- 性能基准测试数据

## 更新方式

### 文档与代码同步规则

1. **新增 API**: 在 `02_API_Reference.md` 添加对应条目
2. **修改 API**: 更新参数、返回值、错误码描述
3. **新增 BUILD target**: 在 `03_Build.md` 添加 target 说明
4. **架构变更**: 更新 `01_Architecture.md` 架构图和描述

### 更新步骤

```bash
# 1. 检出代码并修改
git checkout -b feature/xxx

# 2. 更新对应 Wiki 文档
#    - 修改 wiki/ 目录下相应文件

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for feature/xxx"

# 4. 创建 PR 进行 Code Review
```

### 文档质量检查清单

- [ ] API 变更有代码证据（文件路径 + 符号名）
- [ ] N-API 文档包含 API 清单表
- [ ] GN 文档包含 target 列表和产物映射
- [ ] 安全评审有攻击面分析和风险点
- [ ] SUMMARY.md 链接正确可跳转
- [ ] 无测试相关内容引用
- [ ] 术语统一（中文为主）

## 快速导航

### 新人阅读路线

```
1. 00_Overview.md    → 项目定位与核心能力
2. 01_Architecture.md → 系统架构与数据流
3. 02_API_Reference.md → API 使用说明
4. 03_Build.md      → 构建配置与产物
5. 04_Security.md   → 安全注意事项
```

### 开发者常用链接

- [README.md](README.md) - 项目原始说明
- [README_zh.md](README_zh.md) - 中文项目说明
- [OpenHarmony Location 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/Dev_Guide/source_zh_cn/location/cj-location-guidelines.md)
- [LocationKit API 参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_zh_cn/apis/LocationKit)

## 版本信息

| 项目 | 版本 |
|------|------|
| location_cangjie_wrapper | 6.1 |
| 适配 OpenHarmony | Standard |
| API Level | 22+ |
| ROM 占用 | ~120KB |
| RAM 占用 | ~108KB |
| 文档生成时间 | 2025-02-06 |

## 贡献指南

欢迎开发者贡献 Wiki 文档：

1. 在 Git 仓库中修改 `wiki/` 目录
2. 遵循本文档的更新方式
3. 提交 Pull Request 进行审查

具体贡献流程请参考 [OpenHarmony 代码贡献指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/contribute/参与贡献.md)。

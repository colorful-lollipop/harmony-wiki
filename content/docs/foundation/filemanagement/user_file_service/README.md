# User File Service Wiki - 工程文档

## 文档说明

本 Wiki 面向 OpenHarmony `user_file_service`（公共文件访问框架）项目，提供完整的工程参考文档。

**适用范围**：OpenHarmony 标准系统 (Standard)

**最后更新**：2026-02-07

**代码版本**：基于仓库 `HEAD` 分支

---

## 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 定位、能力、约束 |
| 系统架构 | ✅ 完成 | 组件图、数据流、线程模型 |
| N-API 接口 | ✅ 完成 | JS API 清单、绑定位置 |
| 内部 API | ✅ 完成 | 模块接口、依赖方向 |
| GN 构建 | ✅ 完成 | Targets、依赖、产物 |
| 编译产物 | ✅ 完成 | .so/.hap 清单、安装路径 |
| 攻击面分析 | ✅ 完成 | 外部输入、敏感操作、信任边界 |
| 安全评审 | ✅ 完成 | 风险点、利用路径、修复建议 |
| 问题定位 | ✅ 完成 | 构建/运行时问题 |

---

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航（含双路线指引）
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md     # 架构说明
├── 02_NAPI_Reference.md   # N-API 参考
├── 03_Inner_API.md        # 内部 API
├── 04_GN_Build.md         # GN 构建系统
├── 05_Build_Artifacts.md  # 编译产物
├── 05_AttackSurface.md    # 攻击面分析（安全研究员必读）⭐ NEW
├── 06_Security_Review.md  # 安全风险评审
└── 07_Troubleshooting.md  # 问题定位
```

---

## 阅读建议

### 双路线导航

**路线一：新人入门** 👨‍💻

面向 OpenHarmony 开发者，快速理解项目、上手开发：
1. `00_Overview.md` - 了解项目定位和能力边界
2. `01_Architecture.md` - 理解整体架构和数据流
3. `02_NAPI_Reference.md` - 学习 API 使用
4. `05_Build_Artifacts.md` - 了解编译产物和部署

**路线二：安全研究** 🔒

面向安全研究员，识别攻击面、评估安全风险：
1. `00_Overview.md` - 理解项目类型和对外暴露面
2. `01_Architecture.md` - 分析信任边界和数据流
3. `05_AttackSurface.md` - **⭐ 核心：完整攻击面清单、入口点、检查点**
4. `06_Security_Review.md` - **⭐ 核心：风险点评析、利用路径、修复建议**

### 按需阅读

| 需求 | 推荐文档 |
|------|----------|
| 接口开发 | `02_NAPI_Reference.md`, `03_Inner_API.md` |
| 构建问题 | `04_GN_Build.md`, `07_Troubleshooting.md` |
| 安全审计 | `05_AttackSurface.md`, `06_Security_Review.md` |
| 架构设计 | `01_Architecture.md` |

---

## 代码证据原则

所有关键结论均基于以下证据来源：

| 类型 | 示例 |
|------|------|
| 文件路径 | `services/BUILD.gn:58-60` |
| 符号名 | `FileAccessService` 类 |
| 代码片段 | 关键函数实现 |

**硬性约束**：不引用测试代码作为业务证据

---

## 如何更新文档

当代码变更时，请同步更新 Wiki：

1. **N-API 变更**：更新 `02_NAPI_Reference.md`
2. **架构变更**：更新 `01_Architecture.md`
3. **构建变更**：更新 `04_GN_Build.md` 和 `05_Build_Artifacts.md`
4. **安全相关**：更新 `06_Security_Review.md`

**更新步骤**：
```bash
# 1. 编辑对应文档
vim wiki/xx_xxx.md

# 2. 验证文档链接
# 检查 SUMMARY.md 链接正确性

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for xxx"
```

---

## 相关链接

- **OpenHarmony 主仓库**：https://gitee.com/openharmony
- **FileAccessFramework 架构图**：见 `figures/file_access_framework.png`
- **相关组件**：
  - [媒体库服务](https://gitee.com/openharmony/multimedia_medialibrary_standard)
  - [存储管理服务](https://gitee.com/openharmony/filemanagement_storage_service)
  - [文件访问接口](https://gitee.com/openharmony/filemanagement_file_api)

---

## 贡献指南

欢迎贡献 Wiki 改进：

1. Fork 仓库
2. 创建分支 `wiki/fix-xxx`
3. 编辑 Wiki 文档
4. 提交 PR

**质量要求**：
- 关键结论需有代码证据
- 术语保持一致
- 链接有效
- 无测试代码引用

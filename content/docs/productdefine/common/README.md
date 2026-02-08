# OpenHarmony 公共产品形态配置 Wiki

## 简介

本 Wiki 面向 OpenHarmony `productdefine/common` 仓库，提供完整的工程文档体系，帮助开发者理解产品形态配置机制、配置继承规则以及各产品类型的部件组成。

## 仓库定位

`productdefine/common` 是 OpenHarmony 编译框架的核心配置仓库之一，负责定义**与芯片无关的通用系统组件形态配置**。一个完整的产品包括：
- **芯片组件部分**：在 `vendor/{company}/{product}/` 目录下定义
- **系统组件部分**：在本仓库定义

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── index.md               # 首页/概览
├── 01_Overview.md         # 项目定位与核心概念
├── 02_Structure.md        # 目录结构与模块职责
├── 03_Inheritance.md      # 配置继承与合并机制
├── 04_Products.md         # 产品配置详解
├── 05_Security.md         # 安全风险评审
├── 06_FAQ.md              # 常见问题与调试
└── appendix/
    ├── Config_Reference.md    # 配置字段参考
    └── Inheritance_Graph.md   # 继承关系图
```

## 新人阅读路线

建议按以下顺序阅读：

1. **[首页](index.md)** - 快速了解仓库概况
2. **[项目概览](01_Overview.md)** - 理解项目定位、边界和核心概念
3. **[目录结构](02_Structure.md)** - 熟悉目录组织和文件职责
4. **[继承机制](03_Inheritance.md)** - 掌握配置继承和合并规则
5. **[产品配置](04_Products.md)** - 了解具体产品形态配置
6. **[安全评审](05_Security.md)** - 了解潜在风险和缓解措施
7. **[常见问题](06_FAQ.md)** - 掌握调试和问题排查方法

## 覆盖范围

### 已覆盖内容
- ✅ 目录结构与文件职责
- ✅ 配置格式与字段说明
- ✅ 配置继承与合并机制
- ✅ 各产品形态配置详解
- ✅ 子系统与部件列表
- ✅ 安全风险分析
- ✅ 常见问题与调试方法

### 未覆盖内容（需结合其他仓库）
- ❌ 具体部件实现代码
- ❌ GN 构建脚本细节（在本仓库不涉及）
- ❌ 编译产物生成逻辑
- ❌ 芯片组件配置（位于 vendor 目录）

## 文档更新方式

### 自动更新触发条件
- 本仓库 JSON 配置文件变更
- README_zh.md 文档变更
- 新增/删除配置文件

### 手动更新步骤
1. 更新 `wiki/_work/NOTES.md` 记录变更
2. 修改相应 Wiki 文档
3. 更新 `SUMMARY.md` 导航（如新增页面）
4. 更新本文档的"覆盖范围"章节
5. 运行一致性校验

## 术语表

| 术语 | 含义 |
|------|------|
| 产品形态 | Product Profile，系统组件的配置集合 |
| 子系统 | Subsystem，功能模块的高层组织单元 |
| 部件 | Component，子系统的具体实现单元 |
| Feature | 部件的可配置特性开关 |
| 继承 | Inheritance，复用其他配置文件的部件定义 |
| 最小系统 | Base System，最基础的系统组件集合 |

## 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [编译框架文档](https://docs.openharmony.cn/pages/v4.1/zh-cn/device-dev/subsystems/subsys-build-all.md)
- [产品配置指南](https://docs.openharmony.cn/pages/v4.1/zh-cn/device-dev/subsystems/subsys-build-product.md)

## 生成信息

- **生成时间**: 2026-02-06
- **基于版本**: master 分支
- **配置文件版本**: 3.0
- **最后更新**: 2026-02-06

---

如有问题或建议，请通过 OpenHarmony 社区渠道反馈。

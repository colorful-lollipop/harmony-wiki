# contacts_data Wiki 文档

> OpenHarmony 联系人数据库子系统完整技术文档

## 文档概述

本文档为 OpenHarmony `contacts_data` 子系统提供全面的技术参考，涵盖架构设计、API 接口、编译构建和安全分析等方面。文档基于代码实际实现编写，所有关键结论均可追溯到源码证据。

**项目定位**：联系人数据库子系统，提供联系人、通话记录和语音信箱的增删改查能力。

## 覆盖范围

### 已涵盖内容

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概述 | ✅ 完成 | 定位、能力、运行环境 |
| 目录结构 | ✅ 完成 | 模块职责划分 |
| 架构设计 | ✅ 完成 | 组件图、数据流、调用链 |
| N-API 接口 | ✅ 完成 | JS API 清单、参数、调用链 |
| Inner API | ✅ 完成 | C++ 接口定义、依赖关系 |
| GN 构建 | ✅ 完成 | targets 清单、依赖、产物 |
| 编译产物 | ✅ 完成 | .so 文件、安装路径 |
| 安全评审 | ✅ 完成 | 攻击面、风险点、修复建议 |

### 未涵盖内容

| 类别 | 说明 |
|------|------|
| 测试代码 | 按照规范不引用测试代码作为业务证据 |
| 详细实现细节 | 仅包含关键架构和接口设计 |

## 阅读指南

### 新人推荐阅读顺序

1. **概览页** (`00_Overview.md`) - 快速理解项目定位
2. **目录结构** (`01_Directory_Structure.md`) - 了解模块划分
3. **架构说明** (`02_Architecture.md`) - 理解整体设计
4. **API 文档** (`03_API_Reference.md`) - 掌握接口使用
5. **构建文档** (`04_Build.md`) - 了解编译流程
6. **安全评审** (`05_Security_Review.md`) - 了解安全考量

### 按角色查找

| 角色 | 推荐文档 |
|------|----------|
| JS 开发者 | `03_API_Reference.md` (N-API 章节) |
| C++ 开发者 | `03_API_Reference.md` (Inner API 章节) |
| 构建工程师 | `04_Build.md` |
| 安全工程师 | `05_Security_Review.md` |
| 架构师 | `02_Architecture.md` |

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── 00_Overview.md              # 项目概览
├── 01_Directory_Structure.md    # 目录结构
├── 02_Architecture.md           # 架构设计
├── 03_API_Reference.md          # API 参考
├── 04_Build.md                  # 构建文档
├── 05_Security_Review.md        # 安全评审
├── appendix/
│   ├── Callgraphs.md           # 关键调用链
│   └── Config_Flags.md         # 配置参数
└── figures/                     # 文档图片
```

## 更新说明

### 如何保持文档同步

本文档基于代码分析生成，当以下情况发生时需要更新：

1. **新增 N-API 接口** - 更新 `03_API_Reference.md`
2. **修改模块结构** - 更新 `01_Directory_Structure.md` 和 `02_Architecture.md`
3. **调整构建配置** - 更新 `04_Build.md`
4. **发现安全风险** - 更新 `05_Security_Review.md`

### 更新步骤

```bash
# 1. 克隆或更新仓库
git clone <repository_url>
cd contacts_data

# 2. 执行代码分析
# - 检查新增的 N-API 注册点
# - 分析 BUILD.gn 变化
# - 识别新的模块依赖

# 3. 更新对应 Wiki 页面
# - 修改相应的 .md 文件
# - 添加代码证据引用

# 4. 更新 SUMMARY.md
# - 确保链接正确
# - 调整阅读顺序建议
```

## 代码证据规范

本文档遵循严格的代码证据规范：

- **N-API 文档**：必须包含注册文件路径、导出符号、参数校验逻辑
- **架构文档**：必须包含组件位置、调用关系、数据流向
- **构建文档**：必须包含 BUILD.gn 路径、target 名称、输出产物
- **安全文档**：必须包含攻击路径、触发条件、影响范围

无法确认的信息将标注 `TODO(需确认)` 并说明缺少的证据。

## 版本信息

| 属性 | 值 |
|------|-----|
| 文档版本 | 1.0.0 |
| 生成时间 | 2024-XX-XX |
| 适用的 contacts_data 版本 | 3.1.0 |
| 维护者 | OpenHarmony 社区 |

## 贡献指南

欢迎贡献和改进本文档：

1. 发现错误请提交 Issue
2. 改进建议请提交 Pull Request
3. 遵循本文档的证据规范
4. 保持术语一致性

## 许可证

本文档遵循与项目相同的 Apache License 2.0 许可证。

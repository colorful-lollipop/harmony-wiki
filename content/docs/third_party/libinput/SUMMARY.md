# libinput Wiki 阅读路线指南

> 本文档帮助您快速找到需要的信息，建议适合不同需求的阅读路径

---

## 🎯 阅读路径推荐

### 路径 1：快速了解 OH 定制（15 分钟）

**适合人群**：想快速了解 OpenHarmony 对 libinput 做了哪些定制

**阅读顺序**：
1. **[README.md](README.md)** - 库概览、OH 适配概述（10 分钟）
2. **[02_Patches.md](02_Patches.md)** - Patch 详细分析（5 分钟）
   - 重点关注：0.2 节 "Patch 分类" 和 "关键定制内容"
   - 跳过：详细的代码 diff

**关键收获**：
- OH 新增了哪些设备类型（Joystick、MSDP、Privacy Switch）
- OH 增强了哪些事件类型（Touchpad 专用事件）
- Patch 修改了哪些文件（37 个文件）

---

### 路径 2：深入理解 Patch（45 分钟）

**适合人群**：需要详细了解每个 Patch 的具体内容和目的

**阅读顺序**：
1. **[README.md](README.md)** - 快速概览（5 分钟）
2. **[02_Patches.md](02_Patches.md)** - Patch 详细分析（35 分钟）
   - 完整阅读：所有章节
   - 重点关注：每个 Patch 的 "OH 需求" 和 "升级建议"
3. **[ASSESSMENT.md](_work/ASSESSMENT.md)** - 项目评估结果（5 分钟）
   - 查看 0.6 节 "上游功能对比详细分析"

**关键收获**：
- 每个 Patch 的具体代码变更
- 修改的动机和 OH 特定需求
- 升级上游版本时的回归风险
- 哪些 Patch 可以推向上游

---

### 路径 3：构建和集成（30 分钟）

**适合人群**：负责构建 libinput 或集成到 OH 系统

**阅读顺序**：
1. **[README.md](README.md)** - 基础信息（5 分钟）
2. **[03_Build_Integration.md](03_Build_Integration.md)** - OH 构建适配（20 分钟）
   - 重点关注：BUILD.gn 结构、patch 应用机制
   - 理解：编译选项和依赖配置
3. **[ASSESSMENT.md](_work/ASSESSMENT.md)** - 0.4 节 "特殊适配识别"（5 分钟）

**关键收获**：
- OH 如何使用 GN 替代 Meson
- Patch 脚本的工作流程
- 安全编译选项（CFI、PAC_RET）
- 笔设备支持的编译开关

---

### 路径 4：使用和依赖（40 分钟）

**适合人群**：需要在 OH 中使用 libinput 或分析依赖关系

**阅读顺序**：
1. **[README.md](README.md)** - 在 OH 中的作用（5 分钟）
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用（30 分钟）
   - 重点关注：主要依赖者列表
   - 查看：依赖关系图（Mermaid）
3. **[ASSESSMENT.md](_work/ASSESSMENT.md)** - 0.3 节 "OH 使用情况分析"（5 分钟）

**关键收获**：
- 哪些 OH 模块使用了 libinput
- 每个模块的典型使用场景
- 如何链接和使用 libinput
- 依赖架构和事件流

---

### 路径 5：API 开发（60 分钟）

**适合人群**：需要使用 libinput API 开发或适配新设备

**阅读顺序**：
1. **[README.md](README.md)** - 快速开始（10 分钟）
2. **[02_Patches.md](02_Patches.md)** - 新增功能概览（10 分钟）
3. **[05_API_Differences.md](05_API_Differences.md)** - API 差异（35 分钟）
   - 重点关注：新增 API、新增数据结构
   - 查看：代码示例和使用场景
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 实际使用示例（5 分钟）

**关键收获**：
- OH 新增了哪些 API
- 如何使用 Joystick、MSDP、Touchpad 专用事件
- 新增数据结构的使用方法
- 实际代码示例

---

### 路径 6：安全评估（30 分钟）

**适合人群**：关注 libinput 的安全性和 CVE 状态

**阅读顺序**：
1. **[README.md](README.md)** - 基础信息（5 分钟）
2. **[06_Security.md](06_Security.md)** - 安全风险分析（20 分钟）
   - 重点关注：已知 CVE、OH 版本修复状态
   - 查看：Patch 引入的新攻击面
3. **[02_Patches.md](02_Patches.md)** - Patch 风险评估（5 分钟）

**关键收获**：
- libinput 的已知 CVE
- OH 版本的安全修复状态
- Patch 可能引入的新风险
- 安全升级建议

---

## 📊 文档速查表

### 按主题查找

| 主题 | 相关文档 | 章节 |
|------|----------|------|
| **OH 定制概述** | [README.md](README.md) | OH 适配概述 |
| **Joystick 支持** | [02_Patches.md](02_Patches.md) | 1.1 节 |
| **MSDP 设备** | [02_Patches.md](02_Patches.md) | 1.2 节 |
| **Privacy Switch** | [02_Patches.md](02_Patches.md) | 1.3 节 |
| **Touchpad 事件** | [02_Patches.md](02_Patches.md) | 1.4 节 |
| **Build 配置** | [03_Build_Integration.md](03_Build_Integration.md) | 全文 |
| **依赖关系** | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 全文 |
| **新增 API** | [05_API_Differences.md](05_API_Differences.md) | 全文 |
| **安全性** | [06_Security.md](06_Security.md) | 全文 |

### 按问题查找

| 问题 | 查看文档 | 章节 |
|------|----------|------|
| **OH 新增了哪些功能？** | [02_Patches.md](02_Patches.md) | 1.1-1.5 节 |
| **如何构建 libinput？** | [03_Build_Integration.md](03_Build_Integration.md) | 2.1 节 |
| **哪些模块使用了 libinput？** | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 1 节 |
| **如何使用 Joystick API？** | [05_API_Differences.md](05_API_Differences.md) | 3.1 节 |
| **有什么已知 CVE？** | [06_Security.md](06_Security.md) | 1 节 |
| **如何升级 libinput？** | [02_Patches.md](02_Patches.md) | 6 节 "升级建议" |
| **Patch 改了哪些文件？** | [02_Patches.md](02_Patches.md) | 2 节 |
| **如何启用笔设备支持？** | [03_Build_Integration.md](03_Build_Integration.md) | 3.3 节 |

---

## 🔧 技术深度

| 文档 | 技术深度 | 代码示例 |
|------|----------|---------|
| [README.md](README.md) | ⭐ 概览 | ✅ 基础 |
| [01_Overview.md](01_Overview.md) | ⭐⭐ 简介 | ❌ 无 |
| [02_Patches.md](02_Patches.md) | ⭐⭐⭐⭐⭐ 深入 | ✅ 大量 |
| [03_Build_Integration.md](03_Build_Integration.md) | ⭐⭐⭐ 中等 | ✅ 配置 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | ⭐⭐ 中等 | ✅ 关系图 |
| [05_API_Differences.md](05_API_Differences.md) | ⭐⭐⭐⭐ 深入 | ✅ API 示例 |
| [06_Security.md](06_Security.md) | ⭐⭐⭐ 专业 | ⚠️ CVE 列表 |

**技术深度说明**：
- ⭐ 概览：高层概念和概述
- ⭐⭐ 简介：基础概念和简单示例
- ⭐⭐⭐ 中等：具体配置和使用
- ⭐⭐⭐ 深入：详细代码实现和分析
- ⭐⭐⭐⭐ 专业：深入的技术分析

---

## 💡 常见使用场景

### 场景 1：我需要在 OH 中添加新的输入设备支持

**推荐路径**：路径 3（构建和集成）

**关键文档**：
1. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建系统
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看现有设备的集成方式

**步骤**：
1. 在 `quirks/` 目录添加设备特性文件
2. 如需 OH 特有功能，参考 [02_Patches.md](02_Patches.md)
3. 更新 BUILD.gn 添加新源文件
4. 测试新设备的事件处理

---

### 场景 2：我想了解 OH 对 libinput 的定制

**推荐路径**：路径 1（快速了解）

**关键文档**：
1. [README.md](README.md) - OH 适配概述
2. [02_Patches.md](02_Patches.md) - Patch 详细分析

**重点**：
- 1.1-1.5 节：主要定制内容
- 2 节：修改文件清单
- 3 节：上游 vs OH 对比

---

### 场景 3：我需要在应用中使用 libinput API

**推荐路径**：路径 5（API 开发）

**关键文档**：
1. [README.md](README.md) - 快速开始示例
2. [05_API_Differences.md](05_API_Differences.md) - 新增 API 详解
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用示例

**步骤**：
1. 查看需要的事件类型和 API
2. 包含 `<libinput.h>` 头文件
3. 使用 libinput API 打开设备和处理事件
4. 参考 [README.md](README.md) 的代码示例

---

### 场景 4：我需要评估 libinput 升级的风险

**推荐路径**：路径 2（深入理解 Patch）+ 路径 6（安全评估）

**关键文档**：
1. [02_Patches.md](02_Patches.md) - 每个 Patch 的升级建议
2. [06_Security.md](06_Security.md) - CVE 和风险评估
3. [ASSESSMENT.md](_work/ASSESSMENT.md) - 0.6 节上游对比

**步骤**：
1. 列出所有 OH Patch（[02_Patches.md](02_Patches.md)）
2. 检查上游版本是否包含相关功能
3. 评估 Patch 是否可以推向上游
4. 查看已知 CVE 和修复状态
5. 制定升级计划和回归测试

---

### 场景 5：我需要调试输入事件问题

**推荐路径**：路径 3（构建和集成）

**关键文档**：
1. [03_Build_Integration.md](03_Build_Integration.md) - 了解构建工具
2. [README.md](README.md) - 查看诊断工具列表

**工具**：
- `libinput-debug-mmi` - 事件调试
- `libinput-record-mmi` - 事件记录
- `libinput-analyze-mmi` - 事件分析

---

## 📚 文档版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-08 | 初始版本 |

---

**文档维护**：libinput Wiki 生成 Agent
**最后更新**：2026-02-08

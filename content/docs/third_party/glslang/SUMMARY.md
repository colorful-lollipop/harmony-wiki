# 阅读路线建议

## 快速导航

本文档帮助您根据需要快速找到相关内容。

---

## 按角色分类

### 🧑‍💻 如果你是图形开发者

**阅读顺序**:
1. [01_Overview](01_Overview.md) - 了解 glslang 功能和定位
2. [04_Usage_in_OH](04_Usage_in_OH.md) - 了解如何在项目中使用
3. [03_Build_Integration](03_Build_Integration.md) - 了解构建配置（可选）

**重点关注**:
- GLSL/HLSL → SPIR-V 编译流程
- API 使用示例
- 头文件引用方式
- 性能考虑

---

### 🏗️ 如果你是构建工程师

**阅读顺序**:
1. [03_Build_Integration](03_Build_Integration.md) - 完整的构建系统说明
2. [02_Patches](02_Patches.md) - 了解 OH 适配方式
3. [04_Usage_in_OH](04_Usage_in_OH.md) - 了解依赖关系

**重点关注**:
- 双重 BUILD.gn 结构
- 编译选项配置
- 依赖目标映射
- 白名单配置

---

### 🔒 如果你是安全工程师

**阅读顺序**:
1. [06_Security](06_Security.md) - 完整的安全分析
2. [02_Patches](02_Patches.md) - 了解 OH 特定修改

**重点关注**:
- CVE 历史
- 攻击面分析
- 安全加固建议
- 升级策略

---

### 📦 如果你要升级版本

**阅读顺序**:
1. [02_Patches](02_Patches.md) - 了解当前适配点
2. [06_Security](06_Security.md) - 检查安全更新
3. [03_Build_Integration](03_Build_Integration.md) - 验证构建配置
4. [01_Overview](01_Overview.md) - 了解版本差异

**重点关注**:
- 无 Patch 升级优势
- OH_SDK 宏位置
- API 变更检查
- 测试验证步骤

---

### 🔍 如果你是审计/评估人员

**阅读顺序**:
1. [README](README.md) - 快速概览
2. [01_Overview](01_Overview.md) - 详细功能和定位
3. [02_Patches](02_Patches.md) - OH 适配分析
4. [04_Usage_in_OH](04_Usage_in_OH.md) - 依赖关系
5. [06_Security](06_Security.md) - 安全评估

**重点关注**:
- 无 Patch 的特殊性
- 依赖者数量和类型
- 安全风险评估
- 合规性考虑

---

## 按场景分类

### 🚀 首次集成 glslang

1. [01_Overview](01_Overview.md) - 了解是否适合你的需求
2. [04_Usage_in_OH](04_Usage_in_OH.md) - 查看使用示例
3. [03_Build_Integration](03_Build_Integration.md) - 配置构建

### 🔄 从其他编译器迁移

1. [01_Overview](01_Overview.md) - 功能对比
2. [04_Usage_in_OH](04_Usage_in_OH.md) - 迁移示例
3. [02_Patches](02_Patches.md) - 了解 OH 特性

### 🧪 调试编译问题

1. [03_Build_Integration](03_Build_Integration.md) - 检查配置
2. [02_Patches](02_Patches.md) - 了解 OH 特定行为
3. [04_Usage_in_OH](04_Usage_in_OH.md) - 检查依赖

---

## 文档速查表

| 我想了解... | 阅读文档 |
|-------------|----------|
| glslang 是什么？ | [01_Overview](01_Overview.md) |
| 有哪些修改？ | [02_Patches](02_Patches.md) |
| 如何构建？ | [03_Build_Integration](03_Build_Integration.md) |
| 谁在使用？ | [04_Usage_in_OH](04_Usage_in_OH.md) |
| 安全吗？ | [06_Security](06_Security.md) |
| 如何升级？ | [02_Patches](02_Patches.md) + [06_Security](06_Security.md) |
| API 怎么用？ | [01_Overview](01_Overview.md) + [04_Usage_in_OH](04_Usage_in_OH.md) |
| 版本信息 | [01_Overview](01_Overview.md) |
| 依赖关系 | [04_Usage_in_OH](04_Usage_in_OH.md) |
| 安全建议 | [06_Security](06_Security.md) |

---

## 术语速查

| 术语 | 说明 |
|------|------|
| GLSL | OpenGL Shading Language |
| ESSL | OpenGL ES Shading Language |
| HLSL | High Level Shading Language (DirectX) |
| SPIR-V | Standard Portable Intermediate Representation for Vulkan |
| CTS | Conformance Test Suite (合规性测试套件) |
| dEQP | drawElements Quality Program (CTS 前身) |
| AST | Abstract Syntax Tree (抽象语法树) |
| RTTI | Run-Time Type Information |
| GN | Generate Ninja (构建系统) |
| OH_SDK | OpenHarmony SDK 宏定义 |

---

## 相关资源

- **上游项目**: https://github.com/KhronosGroup/glslang
- **SPIR-V 规范**: https://www.khronos.org/registry/spir-v/
- **Vulkan 规范**: https://www.khronos.org/registry/vulkan/
- **OpenHarmony 图形子系统**: `foundation/graphic/`
- **评估文档**: `_work/ASSESSMENT.md`
- **进度计划**: `_work/PLAN.md`

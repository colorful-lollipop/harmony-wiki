# glslang - OpenHarmony Wiki

**Khronos GLSL/ESSL/HLSL → SPIR-V 编译器**

---

## 快速导航

| 文档 | 内容 |
|------|------|
| [01_Overview](01_Overview.md) | 库概览、版本信息、OH 定位 |
| [02_Patches](02_Patches.md) | Patch 分析（OH_SDK 宏使用） |
| [03_Build_Integration](03_Build_Integration.md) | GN 构建系统适配 |
| [04_Usage_in_OH](04_Usage_in_OH.md) | 依赖关系、使用场景 |
| [06_Security](06_Security.md) | 安全风险分析 |

---

## 库概览

**glslang** 是 Khronos Group 官方维护的着色器语言编译器前端，支持：

- **GLSL** (OpenGL Shading Language) - 桌面和 ES 版本
- **ESSL** (OpenGL ES Shading Language)  
- **HLSL** (High Level Shading Language) - 部分支持
- **输出**: SPIR-V (Vulkan 标准中间表示)

### 关键特性

| 特性 | 状态 |
|------|------|
| GLSL/ESSL → SPIR-V | ✅ 完整支持 |
| HLSL → SPIR-V | ⚠️ 部分支持（语义非参考级） |
| SPIR-V 反汇编 | ✅ 支持 |
| 反射 API | ✅ 支持 |
| SPIRV-Tools 优化 | ❌ OH 中禁用 |

### 在 OpenHarmony 中的定位

```
┌─────────────────────────────────────────┐
│         OpenHarmony 图形栈               │
├─────────────────────────────────────────┤
│  应用层 (ArkUI, 游戏引擎)                 │
├─────────────────────────────────────────┤
│  图形框架 (Skia, Lume 3D)                │
├─────────────────────────────────────────┤
│  Vulkan/OpenGL 驱动层                    │
├─────────────────────────────────────────┤
│  vk-gl-cts (合规性测试) ← glslang       │
│  Lume Shader Compiler ← glslang         │
└─────────────────────────────────────────┘
```

**核心用途**: 主要用于 Khronos 合规性测试套件 (CTS) 和 Lume 3D 引擎的着色器编译。

---

## 版本信息

| 项目 | 内容 |
|------|------|
| **上游版本** | vulkan-sdk-1.3.275.0 |
| **OH 组件版本** | 3.2 |
| **上游许可证** | Apache-2.0 |
| **OH 声明许可证** | 3-Clause BSD |
| **维护者** | zhangleiyu1@huawei.com |

---

## OpenHarmony 适配要点

### 无 Patch 导入

⚠️ **重要**: glslang 是**无 Patch** 导入的第三方库。OH 使用原始上游代码，仅通过以下条件进行适配：

1. **编译宏控制**: `OH_SDK` 宏用于条件编译
2. **构建系统**: 从 CMake 迁移到 GN
3. **配置调整**: 禁用 SPIRV-Tools 优化以减少依赖

### 关键适配点

| 适配项 | 说明 |
|--------|------|
| `OH_SDK` 宏 | 控制 spirv-remap 工具和部分 OS 依赖代码 |
| GN 构建 | 双重 BUILD.gn（Chromium 风格 + OH 风格） |
| 禁用优化 | `ENABLE_OPT=0` 禁用 SPIRV-Tools 优化器 |
| C++17 | 使用 C++17 标准编译 |
| 禁用异常/RTTI | `-fno-exceptions`, `-fno-rtti` |

---

## 文档阅读建议

1. **如果你是图形开发者**: 阅读 [01_Overview](01_Overview.md) 了解功能，[04_Usage_in_OH](04_Usage_in_OH.md) 了解如何在你的项目中使用
2. **如果你是构建工程师**: 重点阅读 [03_Build_Integration](03_Build_Integration.md)
3. **如果你要升级版本**: 阅读 [02_Patches](02_Patches.md)（了解适配点）和 [06_Security](06_Security.md)

---

## 相关资源

- **上游仓库**: https://github.com/KhronosGroup/glslang
- **Khronos 官方文档**: https://www.khronos.org/opengles/sdk/tools/Reference-Compiler/
- **OpenHarmony 图形子系统**: `foundation/graphic/`
- **vk-gl-cts**: `third_party/vk-gl-cts/`

---

## 维护说明

本文档由 OpenHarmony Wiki Agent 自动生成于 2026-02-08。

如发现文档错误或有更新需求，请：
1. 检查 `wiki/_work/ASSESSMENT.md` 中的评估结论
2. 更新相关章节
3. 在 `wiki/_work/PLAN.md` 中记录变更

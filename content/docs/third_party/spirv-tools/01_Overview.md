# 原始库简介

## 1.1 SPIRV-Tools 概述

SPIRV-Tools 是由 **Khronos Group**（负责制定 Vulkan、OpenGL、OpenCL 等标准的行业协会）开发和维护的开源工具集，专注于处理 **SPIR-V**（Standard Portable Intermediate Representation）二进制格式。

### SPIR-V 简介

SPIR-V 是一种独立于供应商的二进制中间表示（IR）格式，主要用于：

| 应用领域 | 用途 |
|---------|------|
| **Vulkan** | 图形渲染着色器和计算着色器 |
| **OpenGL** | 通过 ARB_gl_spirv 扩展支持 SPIR-V 着色器 |
| **OpenCL** | OpenCL 2.1 及更高版本的内核代码 |
| **ML/AI** | 用于机器学习工作负载的计算着色器 |

### SPIRV-Tools 的定位

```
高级语言（GLSL, HLSL, OpenCL C）
        ↓
编译器（glslang, clspv 等）
        ↓
SPIR-V 二进制格式
        ↓
SPIRV-Tools 处理（验证、优化、汇编/反汇编）
        ↓
GPU 驱动（运行时）
```

---

## 1.2 核心功能模块

### 1.2.1 汇编器（Assembler）

**功能**: 将 SPIR-V 汇编文本（`.spvasm`）转换为二进制格式（`.spv`）

**命令**: `spirv-as`

**使用场景**:
```bash
spirv-as shader.spvasm -o shader.spv
```

**API 示例**:
```cpp
// C++ API
#include "spirv-tools/libspirv.hpp"

std::vector<uint32_t> Assemble(const std::string& text) {
    spvtools::SpirvTools tool(SPV_ENV_UNIVERSAL_1_2);
    std::vector<uint32_t> binary;
    tool.Assemble(text, &binary);
    return binary;
}
```

### 1.2.2 反汇编器（Disassembler）

**功能**: 将 SPIR-V 二进制转换为人类可读的汇编文本

**命令**: `spirv-dis`

**使用场景**:
```bash
spirv-dis shader.spv
```

**输出示例**:
```asm
; SPIR-V
; Version: 1.0
; Generator: Khronos Glslang Reference Front End
; Bound: 10
 OpSource GLSL 450
 OpEntryPoint Fragment %main "main" % fragColor
 OpExecutionMode %main OriginUpperLeft
 %float = OpTypeFloat 32
 %void = OpTypeVoid
 %void_func = OpTypeFunction %void
 %float_var = OpVariable %void_func PointerOutput
```

### 1.2.3 验证器（Validator）

**功能**: 根据 SPIR-V 规范验证二进制模块的有效性

**命令**: `spirv-val`

**使用场景**:
```bash
spirv-val --env Vulkan1.2 shader.spv
```

**验证规则示例**:
- 操作数类型的合法性检查
- 指令顺序的正确性
- 类型和常量的使用正确性
- 控制流图的结构正确性
- 资源绑定的合法性

### 1.2.4 优化器（Optimizer）

**功能**: 应用各种代码优化 pass 以改善性能或减小代码体积

**命令**: `spirv-opt`

**常用优化 pass**:

| 优化类型 | Pass 名称 | 说明 |
|---------|----------|------|
| **死代码消除** | `eliminate-dead-code` | 移除未使用的函数和指令 |
| **死代码消除（激进）** | `aggressive-dead-code-elim` | 更激进的死代码消除 |
| **常量折叠** | `fold-constants` | 折叠常量表达式 |
| **内联** | `inline` | 函数内联 |
| **循环优化** | `loop-unroll` | 循环展开 |
| | `loop-fusion` | 循环融合 |
| | `loop-peeling` | 循环剥离 |
| **代码简化** | `simplify` | 通用代码简化 |
| **冗余消除** | `redundancy-elimination` | 消除冗余计算 |

**使用场景**:
```bash
# 性能优化
spirv-opt -O shader.spv -o shader_opt.spv

# 体积优化
spirv-opt -Os shader.spv -o shader_small.spv

# 指定特定 pass
spirv-opt --eliminate-dead-code shader.spv -o shader_opt.spv
```

### 1.2.5 链接器（Linker）

**功能**: 将多个 SPIR-V 模块合并为一个模块

**命令**: `spirv-link`

**使用场景**:
```bash
spirv-link module1.spv module2.spv -o combined.spv
```

### 1.2.6 Reducer

**功能**: 根据用户定义的"有趣"函数，简化测试用例以定位问题

**命令**: `spirv-reduce`

**使用场景**: 用于调试着色器编译器问题，将失败的测试用例最小化

### 1.2.7 Fuzzer

**功能**: 通过语义-preserving 变换生成等价的 SPIR-V 模块，用于发现工具链 bug

**命令**: `spirv-fuzz`

---

## 1.3 API 接口

### C API

```c
// 汇编
SpvTextToBinary(const struct spv_context*, const char* text, size_t length,
                spv_text* pText, spv_diagnostic* pDiagnostic);

// 反汇编
SpvBinaryToText(const struct spv_context*, const uint32_t* binary,
                size_t wordCount, int options, spv_text* pText,
                spv_diagnostic* pDiagnostic);

// 验证
spv_result_t spvValidate(const struct spv_context*,
                        const uint32_t* binary, size_t wordCount,
                        spv_diagnostic* pDiagnostic);

// 获取错误信息
void spvDiagnosticPrint(spv_diagnostic diagnostic);
```

### C++ API

```cpp
namespace spvtools {

class SpirvTools {
 public:
  // 汇编
  bool Assemble(const std::string& text, std::vector<uint32_t>* binary,
                MessageConsumer consumer = nullptr);

  // 反汇编
  bool Disassemble(const std::vector<uint32_t>& binary,
                   std::string* text, uint32_t options = 0,
                   const char* env = "universal");

  // 验证
  bool Validate(const std::vector<uint32_t>& binary,
                MessageConsumer consumer = nullptr);
};

class Optimizer {
 public:
  // 注册优化 pass
  Optimizer& RegisterPass(Pass&& p);

  // 运行优化
  bool Run(const uint32_t* in, size_t in_len,
           uint32_t* out, size_t out_len);
};

}  // namespace spvtools
```

---

## 1.4 在 OpenHarmony 中的作用

### 1.4.1 主要用途

SPIRV-Tools 在 OpenHarmony 中主要用于 **Khronos 一致性测试**：

```
OpenHarmony
    ↓
vk-gl-cts（Vulkan/OpenGL Conformance Test Suite）
    ↓
SPIRV-Tools（SPIR-V 工具）
    ↓
图形驱动验证
```

### 1.4.2 使用场景

| 场景 | 描述 |
|------|------|
| **着色器测试** | CTS 测试需要汇编/反汇编 SPIR-V 着色器 |
| **验证测试** | 验证 SPIR-V 规范的各种规则 |
| **优化测试** | 测试优化 pass 的正确性 |
| **链接测试** | 测试模块链接功能 |

### 1.4.3 为什么需要它

1. **标准合规性**: 确保 OpenHarmony 图形驱动符合 Khronos 标准
2. **调试工具**: 提供着色器问题的诊断能力
3. **测试基础设施**: CTS 套件依赖 SPIRV-Tools 进行测试

---

## 1.5 版本与上游

### 当前版本

| 属性 | 值 |
|------|-----|
| **上游版本** | vulkan-sdk-1.3.275.0 |
| **上游仓库** | https://github.com/KhronosGroup/SPIRV-Tools |
| **OH 版本** | 3.2 |

### 版本说明

该库版本与 Vulkan SDK 版本对齐，确保与 Vulkan 规范的兼容性。不同版本的 SPIRV-Tools 支持不同版本的 SPIR-V 规范。

### SPIR-V 版本支持

| SPIR-V 版本 | 支持状态 |
|------------|---------|
| 1.0 | ✅ 支持 |
| 1.1 | ✅ 支持 |
| 1.2 | ✅ 支持 |
| 1.3 | ✅ 支持 |
| 1.4 | ✅ 支持 |
| 1.5 | ✅ 支持 |

---

## 1.6 许可证

### Apache-2.0 License

SPIRV-Tools 使用 Apache-2.0 许可证，这意味着：

**允许的**:
- ✅ 自由使用、修改和分发
- ✅ 商业使用
- ✅ 专利授权
- ✅ 衍生作品

**要求的**:
- 📝 保留原始版权声明
- 📝 声明重大修改
- 📝 提供许可证副本

**禁止的**:
- ❌ 使用商标名称背书

---

## 1.7 相关资源

### 上游资源

- [SPIRV-Tools GitHub](https://github.com/KhronosGroup/SPIRV-Tools)
- [SPIRV-Tools 文档](https://github.com/KhronosGroup/SPIRV-Tools/tree/master/docs)
- [SPIR-V 规范](https://www.khronos.org/registry/spir-v/)
- [Vulkan 规范](https://www.khronos.org/registry/vulkan/)

### 内部资源

- [OH 适配概述](./README.md)
- [构建适配](./03_Build_Integration.md)
- [依赖关系](./04_Usage_in_OH.md)

---

## 1.8 快速开始

### 获取源码

```bash
# 上游仓库
git clone https://github.com/KhronosGroup/SPIRV-Tools.git
```

### 构建（上游方式）

```bash
# 使用 CMake
mkdir build && cd build
cmake .. -DSPIRV_SKIP_TESTS=ON
cmake --build .
```

### 基本使用

```bash
# 汇编
./tools/spirv-as ../test.spvasm -o test.spv

# 反汇编
./tools/spirv-dis test.spv

# 验证
./tools/spirv-val --env Vulkan1.2 test.spv

# 优化
./tools/spirv-opt -O test.spv -o test_opt.spv
```

---

## 1.9 贡献者

### 维护团队

- **Khronos Group**: 主要开发和维护
- **社区贡献者**: 提交 bug 修复和新功能

### 报告问题

- **上游问题**: https://github.com/KhronosGroup/SPIRV-Tools/issues
- **OH 适配问题**: 联系组件所有者

---

*最后更新: 2026-02-07*

*相关内容:*
- *下一章: [Patch 详细分析](./02_Patches.md)*
- *构建适配: [OH 构建适配](./03_Build_Integration.md)*
- *使用方式: [依赖关系与使用](./04_Usage_in_OH.md)*

# 原始库简介

## 基础信息

| 项目 | 内容 |
|------|------|
| **库名称** | SPIRV-Headers |
| **版本** | vulkan-sdk-1.3.275.0 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/KhronosGroup/SPIRV-Headers.git |
| **维护者** | Khronos Group |

## 原始功能

SPIRV-Headers 是 Khronos Group 维护的官方 SPIR-V 规范头文件库，提供 SPIR-V（Standard Portable Intermediate Representation）指令集的标准机器可读定义。

### 核心功能

1. **SPIR-V 核心指令集头文件**
   - `spirv.h` / `spirv.hpp`：定义 SPIR-V 指令操作码、枚举值和结构体
   - 支持 SPIR-V 1.0、1.1、1.2 版本

2. **扩展指令集头文件**
   - `GLSL.std.450.h`：GLSL 标准内联函数（450 版本）
   - `OpenCL.std.h`：OpenCL 标准内联函数
   - `NonSemanticClspvReflection.h`：非语义反射扩展
   - `NonSemanticDebugPrintf.h`：非语义调试打印扩展

3. **JSON 语法文件**
   - `spirv.core.grammar.json`：SPIR-V 核心指令语法定义
   - `extinst.glsl.std.450.grammar.json`：GLSL 扩展指令语法
   - `extinst.opencl.debuginfo.100.grammar.json`：OpenCL 调试信息扩展语法

4. **XML 寄存器文件**
   - `spirv.xml`：SPIR-V 扩展和枚举值的 XML 描述

## 在 OpenHarmony 中的作用与定位

SPIRV-Headers 在 OpenHarmony 系统中定位为**底层图形/计算标准的基石库**，主要服务于以下场景：

### 图形渲染栈

- Vulkan 实现的基础依赖，提供 SPIR-V 指令规范
- 支持 Vulkan 一致性测试（vk-gl-cts）
- 为 GPU 驱动提供标准化的中间表示定义

### SPIR-V 工具链

- 作为 spirv-tools 的上游依赖，提供：
  - 汇编器/反汇编器的指令定义
  - 语法解析器的 JSON 规范
  - 验证器的枚举值检查

### 计算着色器支持

- 支持 OpenCL 标准内联函数定义
- 为后续的计算着色器运行时提供基础设施

## 架构位置

```
OpenHarmony 图形栈
├── 图形驱动层
│   └── GPU 驱动（使用 spirv-headers 定义）
├── Vulkan 实现
│   └── vk-gl-cts（测试套件依赖 spirv-headers）
├── SPIR-V 工具链
│   └── spirv-tools（汇编器/反汇编器依赖 spirv-headers）
└── 运行时层
    └── 着色器编译器（未来扩展）
```

## 版本策略

OH 使用的版本为 `vulkan-sdk-1.3.275.0`，该版本号表明其来源于 Vulkan SDK 的同步发布。建议跟随 Vulkan SDK 版本进行同步升级，以保持与上游规范的一致性。

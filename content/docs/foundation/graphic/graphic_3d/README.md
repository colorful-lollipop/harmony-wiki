# OpenHarmony graphic_3d (AGP引擎) 工程 Wiki

## 项目概述

**AGP (Ark Graphics Platform)** 是 OpenHarmony 的跨平台高性能实时3D渲染引擎。

### 核心特性
- **架构**: ECS (Entity-Component-System) 设计模式
- **后端支持**: OpenGL ES / Vulkan
- **系统能力**: `SystemCapability.ArkUi.Graphics3D`
- **主要语言**: C++17

### 编译产物
| 产物文件名 | 类型 | 说明 |
|-----------|------|------|
| `lib3dWidgetAdapter.z.so` | 共享库 | ArkUI适配层 |
| `libAGPDLL.z.so` | 共享库 | 引擎核心DLL |
| `libPluginAGP3D.z.so` | 共享库 | 3D渲染插件 |
| `libPluginAGPRender.z.so` | 共享库 | 渲染后端插件 |
| `libscene.z.so` | 共享库 | JS N-API接口模块 |
| `scene_ani.z.so` | 共享库 | ETS Taihe接口模块 |

### 源码位置
```
foundation/graphic/graphic_3d/
```

### 文档维护
- **生成时间**: 2025-02-06
- **优化时间**: 2025-02-07
- **适用版本**: OpenHarmony 3.1+
- **更新方式**: 手动更新（代码变更时需同步更新）

## Wiki 结构

| 文档 | 说明 |
|------|------|
| [首页概览](index.md) | 项目定位、核心能力、运行环境 |
| [目录结构](01_Directory_Structure.md) | 模块职责与目录组织 |
| [架构设计](02_Architecture.md) | ECS架构、组件图、数据流、线程模型 |
| [N-API接口](03_NAPI_Reference.md) | JS/ETS API清单、调用链、参数校验 |
| [内部API](04_Internal_API.md) | 模块接口、依赖关系、稳定性标注 |
| [GN构建系统](05_GN_Build.md) | Targets、依赖、产物映射 |
| [安全风险](06_Security.md) | 攻击面、风险点、修复建议 |
| [常见问题](07_FAQ.md) | 构建/运行/调试问题定位 |

## 阅读指南

### 新人入门路线
1. 先读 [首页概览](index.md) 了解项目定位
2. 阅读 [目录结构](01_Directory_Structure.md) 熟悉代码组织
3. 查看 [架构设计](02_Architecture.md) 理解核心设计
4. 根据角色选择：
   - **应用开发者** → [N-API接口](03_NAPI_Reference.md)
   - **系统开发者** → [内部API](04_Internal_API.md) + [GN构建系统](05_GN_Build.md)
   - **安全评审** → [安全风险](06_Security.md)

### 快速索引
- [关键符号索引](appendix/Symbol_Index.md)
- [调用链图谱](appendix/Callgraphs.md)
- [配置宏说明](appendix/Config_Flags.md)

## 覆盖范围与局限

### 已覆盖
- ✅ 所有非测试源码的目录结构
- ✅ N-API JS/ETS 接口清单
- ✅ 核心ECS架构与模块职责
- ✅ GN构建系统关键targets
- ✅ **8个安全风险点（含CVSS 3.1评分和攻击场景）**
- ✅ 项目评估报告 (`_work/ASSESSMENT.md`)

### 未覆盖
- ❌ 测试代码 (`test/` 目录)
- ❌ 具体Shader实现细节
- ❌ 性能优化指南
- ❌ 完整的GLTF格式支持矩阵

### 证据来源
所有关键结论均可追溯到以下代码证据：
- `bundle.json` - 组件配置与依赖
- `README.md` / `README_en.md` - 项目说明
- `*/BUILD.gn` - 构建配置
- `kits/js/src/` - N-API绑定实现
- `lume/*/api/` - 公共API头文件
- `wiki/_work/ASSESSMENT.md` - 项目评估报告

---
*本文档基于代码生成，所有路径均为相对仓库根目录的绝对路径*

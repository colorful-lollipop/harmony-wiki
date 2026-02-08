# napi-generator 项目首页

> OpenHarmony N-API 框架代码生成工具集

## 项目简介

**napi-generator** 是 OpenHarmony 标准系统配套的 N-API 框架代码生成工具集，目标是提升 OpenHarmony Native 应用开发效率。

```
         ┌─────────────────────────────────────────────────────────┐
         │                   napi-generator                        │
         │              OpenHarmony N-API 代码生成工具集              │
         └─────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
┌───────────────┐         ┌───────────────┐         ┌───────────────┐
│  dts2cpp      │         │   h2sa        │         │  h2dtscpp     │
│  TS → N-API   │         │  SA 服务生成   │         │  C++→TS+NAPI  │
└───────────────┘         └───────────────┘         └───────────────┘
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐         ┌───────────────┐         ┌───────────────┐
│  快速生成      │         │  IPC 框架     │         │  完整开发链   │
│  N-API 桥接    │         │  proxy/stub   │         │  接口+测试    │
└───────────────┘         └───────────────┘         └───────────────┘
```

## 核心能力

| 能力 | 说明 | 适用场景 |
|------|------|----------|
| **TS 接口转 N-API** | 从 TypeScript 声明文件生成完整的 C++ N-API 框架代码 | 已有 TS 接口定义，需要快速生成 Native 桥接 |
| **SA 服务框架生成** | 根据 .h 头文件生成完整的 System Ability 框架代码 | 开发系统服务，需要 IPC 通信能力 |
| **C++ 头文件转 TS** | 将 C++ 接口转换为 TypeScript 声明文件 | 已有 C++ 库，需要提供 TS 类型支持 |
| **CMake 转 GN** | 将 CMakeLists.txt 转换为 BUILD.gn | 迁移三方库到 OpenHarmony |
| **API 依赖扫描** | 扫描三方库中的非 OpenHarmony API | 兼容性分析 |

## 工具集概览

```
src/cli/
├── dts2cpp/           # ⭐ 核心工具：TypeScript → N-API
├── h2sa/              # Service Ability 框架生成
├── h2dtscpp/          # C++ → TypeScript + N-API + 测试
├── h2dts/             # C++ → TypeScript
├── cmake2gn/          # CMake → GN
└── h2hdf/             # HDF 驱动生成
```

## 快速开始

### 安装与构建

```bash
# 克隆仓库
git clone https://gitee.com/openharmony/napi_generator.git
cd napi_generator

# 安装依赖 (各工具可能有不同要求)
cd src/cli/dts2cpp/src && npm install
cd ../../../../

# 使用 dts2cpp 生成 N-API 代码
node src/cli/dts2cpp/src/gen/cmd_gen.js \
    -f @ohos.mylib.d.ts \
    -o output \
    -n uint32_t
```

### 示例项目

| 示例 | 说明 | 路径 |
|------|------|------|
| N-API 教程 | 75+ 完整示例 | `examples/napitutorials/` |
| AKI 框架示例 | 简化 N-API 开发 | `examples/akitutorials/` |
| SA 服务示例 | Service Ability 开发 | `examples/serviceCode/` |
| 完整应用 | p7zip 集成 | `examples/p7zipTest/` |

## 与 OpenHarmony 的关系

```
OpenHarmony 应用层
        │
        │ JS/ArkTS 调用
        ▼
┌───────────────────────────────────────┐
│           N-API (Native API)           │
│         OpenHarmony 运行时接口          │
└───────────────────────────────────────┘
        │
        │ 代码生成
        ▼
┌───────────────────────────────────────┐
│         napi-generator                 │
│      生成 N-API 框架代码模板            │
└───────────────────────────────────────┘
        │
        │ 开发者填充业务逻辑
        ▼
┌───────────────────────────────────────┐
│         业务 Native 代码                │
│         C/C++ 实现                     │
└───────────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────────┐
│         系统能力 (SA)                  │
│         IPC / Binder                   │
└───────────────────────────────────────┘
```

## 相关仓库

- [arkui_napi](https://gitee.com/openharmony/arkui_napi) - OpenHarmony N-API 实现
- [safwk](https://gitee.com/openharmony/systemabilitymgr_safwk) - System Ability Framework
- [samgr](https://gitee.com/openharmony/systemabilitymgr_samgr) - System Ability Manager

## 文档导航

| 主题 | 章节 |
|------|------|
| 项目概览 | [01_Overview.md](01_Overview.md) |
| 目录结构 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](03_Architecture.md) |
| API 参考 | [04_NAPI_Reference.md](04_NAPI_Reference.md) |
| 构建系统 | [06_Build_System.md](06_Build_System.md) |
| 安全评审 | [07_Security_Review.md](07_Security_Review.md) |

---

[继续阅读 → 01_Overview.md](01_Overview.md)

[返回 SUMMARY.md](SUMMARY.md)

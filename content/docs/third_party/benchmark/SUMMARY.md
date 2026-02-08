# 文档阅读指南

## 推荐阅读顺序

### 场景 1: 初次了解本库

```
README.md → 01_Overview.md → 04_Usage_in_OH.md
```

了解 Benchmark 库在 OH 中的定位和使用方式。

### 场景 2: 集成到新模块

```
03_Build_Integration.md → 04_Usage_in_OH.md
```

获取构建配置和使用模板。

### 场景 3: 升级或维护

```
02_Patches.md → 03_Build_Integration.md → 01_Overview.md
```

了解 OH 适配细节和 Patch 状态。

## 文档速查表

| 文档 | 目标读者 | 核心内容 |
|------|----------|----------|
| **README.md** | 所有开发者 | 项目概览、导航、快速示例 |
| **01_Overview.md** | 架构设计者 | 功能介绍、OH 定位、技术特性 |
| **02_Patches.md** | 维护升级人员 | Patch 清单、修改说明、升级建议 |
| **03_Build_Integration.md** | 构建系统开发者 | BUILD.gn 配置、编译选项、头文件导出 |
| **04_Usage_in_OH.md** | 模块开发者 | 依赖模块列表、使用模式、依赖关系图 |

## 关键信息速览

### 本库特点

- ✅ **无 Patch** - 源代码保持上游版本
- ✅ **30+ 依赖模块** - OH 生态广泛使用
- ✅ **纯测试工具** - 不影响运行时
- ✅ **静态链接** - 通过 benchmarktest 模板使用

### 使用前提

- C++17 编译器支持
- OHOS 构建系统 (GN)
- 目标系统类型: small / standard

## 相关资源

- [上游文档](docs/user_guide.md) - Google Benchmark 官方用户指南
- [上游仓库](https://github.com/google/benchmark) - GitHub 源码
- [OH 构建系统](../build/ohos.gni) - GN 构建配置参考

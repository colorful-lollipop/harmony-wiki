# Python 3.11.4 OpenHarmony Wiki

## 库概览

本项目是 **Python 3.11.4** (CPython) 在 OpenHarmony 中的第三方库集成，包含 OHOS 特定的适配和 MinGW 构建支持。

| 属性 | 值 |
|-----|-----|
| **库名称** | Python (CPython) |
| **版本** | 3.11.4 |
| **许可证** | Python Software Foundation License V2 |
| **上游地址** | https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tgz |
| **维护者** | anguanglin@huawei.com |
| **OH 组件** | @ohos/python (子系统: thirdparty) |

## OH 适配概述

### 主要 Patch

| Patch | 说明 | 文件数 | 行数变化 |
|-------|------|--------|---------|
| `cross_compile_support_ohos.patch` | OHOS 交叉编译支持 | 4 | +50/-10 |
| `cpython_mingw_v3.11.4.patch` | MinGW 构建支持 | 88 | +3606/-498 |

### 关键适配点

1. **目标平台识别**: 添加 `aarch64-linux-ohos` 和 `arm-linux-ohos` 支持
2. **模块禁用**: 禁用 `_uuid`, `_socket`, `zlib`, `_ctypes`, `binascii` 模块
3. **MinGW 支持**: 完整的 MinGW-w64 工具链支持
4. **路径处理**: MSYS2/MinGW 环境下的路径适配

### 在 OH 中的作用

Python 在 OpenHarmony 中主要作为**构建时工具链**，支持:

- **ArkCompiler**: ETS 前端处理
- **hiperf**: 性能分析工具
- **Protobuf**: 代码生成
- **ArkGuard**: 代码混淆

## 文档导航

### 必读文档

| 文档 | 内容 | 适用读者 |
|-----|------|---------|
| [01_Overview.md](01_Overview.md) | 库概览和 OH 定位 | 所有人 |
| [02_Patches.md](02_Patches.md) | Patch 详细分析 | 开发者、维护者 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建系统适配 | 构建工程师 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 依赖关系 | 架构师、开发者 |

### 参考文档

| 文档 | 内容 | 适用读者 |
|-----|------|---------|
| [05_API_Differences.md](05_API_Differences.md) | API 差异说明 | Python 开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

### 工作文档

| 文档 | 内容 |
|-----|------|
| [ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |

## 快速开始

### 查看 Patch

```bash
# OHOS 交叉编译 Patch
cat patches/cross_compile_support_ohos.patch

# MinGW 支持 Patch
cat patches/cpython_mingw_v3.11.4.patch
```

### 构建测试

```bash
# 标准构建
cd third_party/python
./configure
make

# 交叉编译 (OHOS)
./configure --host=aarch64-linux-ohos
make
```

## 质量保证

### Patch 分析

- [x] 所有 Patch 文件分析完成
- [x] 每个 Patch 有明确的修改目的
- [x] 提供 Patch 升级建议

### 依赖分析

- [x] 列出主要依赖者
- [x] 说明使用场景
- [x] 提供依赖关系图

### 正确性

- [x] Patch 内容摘要准确
- [x] BUILD.gn 配置有具体引用
- [x] 依赖关系基于实际引用

## 贡献和维护

### 提交 Patch

1. 修改代码
2. 更新 Patch 文件: `git diff > patches/xxx.patch`
3. 更新相关文档
4. 提交 MR 并关联 Issue

### 升级版本

1. 查看 [02_Patches.md](02_Patches.md) 升级策略章节
2. 按流程应用 Patch
3. 执行回归测试清单
4. 更新文档版本号

## 相关链接

- [Python 官方文档](https://docs.python.org/3.11/)
- [Python 3.11 新特性](https://docs.python.org/3.11/whatsnew/3.11.html)
- [OpenHarmony 构建系统](https://gitee.com/openharmony/build)
- [MinGW-w64 项目](https://www.mingw-w64.org/)

## 许可证

Python 使用 [Python Software Foundation License V2](LICENSE)。

OpenHarmony 适配 Patch 遵循原库许可证。

---

**最后更新**: 2026-02-08

# tex-hyphen - OpenHarmony 集成文档

> **tex-hyphen** 在 OpenHarmony 中的集成与适配说明

---

## 快速概览

**tex-hyphen** 是 TeX 排版系统的断字模式库，为多语言文本提供高质量的断词规则。在 OpenHarmony 中，它被集成到 Skia 文本引擎，用于提升文本排版的断词质量。

### 核心信息

| 项目 | 内容 |
|-----|------|
| **原始库名称** | TEX hyphenation patterns |
| **OH 组件名称** | @ohos/tex-hyphen |
| **OH 版本** | 5.1 |
| **上游版本** | CTAN-2024.12.31 |
| **许可证** | MIT; GPL; LGPL; LPPL; MPL 混合 |
| **子系统** | thirdparty |
| **依赖** | icu (shared_icuuc) |
| **Patch 数量** | 0（无代码修改） |

### OH 适配方式

**资源文件集成** - tex-hyphen 在 OH 中以资源文件形式集成，不包含可执行代码在 ROM 中：

```
上游 .tex 模式文件
    ↓ [OH 构建工具 hpb_transform]
OH .hpb 二进制文件
    ↓ [安装到 ROM]
/system/usr/ohos_hyphen_data/
    ↓ [Skia Hyphenator 加载]
文本排版引擎（断词功能）
```

---

## 文档导航

### 📚 核心文档

| 文档 | 描述 | 优先级 |
|-----|------|--------|
| [01_Overview.md](./01_Overview.md) | 原始库简介和在 OH 中的定位 | ⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（本库无 Patch） | - |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配详解 | ⭐⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | ⭐⭐⭐ |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异（本库不提供 API） | - |
| [06_Security.md](./06_Security.md) | 安全风险分析 | ⭐⭐ |

### 🔧 工作文档

| 文档 | 描述 |
|-----|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告（Phase 0 收集的所有信息） |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录和技术细节 |
| [_work/PLAN.md](_work/PLAN.md) | 文档编写计划和进度追踪 |

---

## 快速开始

### 了解 tex-hyphen 在 OH 中的作用
1. 阅读 [01_Overview.md](./01_Overview.md) - 了解原始库功能和 OH 集成方式
2. 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解谁在使用这个库

### 理解 OH 的构建适配
1. 阅读 [03_Build_Integration.md](./03_Build_Integration.md) - 深入了解构建流程
2. 查看 [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 获取详细的评估信息

### 维护和升级
1. 阅读 [06_Security.md](./06_Security.md) - 了解安全风险
2. 查看 [_work/PLAN.md](_work/PLAN.md) - 了解升级建议

---

## 关键技术点

### 1. .hpb 二进制格式

OH 将 TeX 格式的断词模式转换为自定义的二进制格式（.hpb），特点：

- **高效查找**: 使用优化的 Trie 树结构
- **紧凑存储**: 使用 16-bit offset 和 8-bit pattern 数组
- **快速加载**: 支持懒加载和内存映射

### 2. 构建工具链

OH 提供专用的构建工具：
- `hpb_transform`: C++17 工具，将 .tex 转换为 .hpb
- `generate_hpb.py`: GN 构建脚本，自动化转换流程
- `hyphen_pattern_processor`: 核心转换算法实现

### 3. Skia 深度集成

断词功能通过 `ENABLE_TEXT_ENHANCE` 宏控制：
- **加载位置**: `third_party/skia/m133/modules/skparagraph/`
- **加载路径**: `/system/usr/ohos_hyphen_data/`
- **语言支持**: 51 种语言的断词规则

---

## 适配复杂度评估

| 维度 | 复杂度 | 说明 |
|-----|--------|------|
| **代码修改** | 低 | 无上游代码修改 |
| **构建集成** | 中 | 需要自定义构建工具 |
| **运行时集成** | 中 | 与 Skia 文本引擎深度集成 |
| **维护成本** | 低 | 仅需同步上游资源文件 |

---

## OH 价值总结

### 1. 排版质量提升
- 多语言文本的自动断词
- 减少手动断词的工作量
- 提升文档的可读性和美观度

### 2. 小屏设备优化
- 在窄屏设备上提高内容密度
- 改善阅读体验
- 适应移动端显示需求

### 3. 国际化支持
- 支持 51 种语言的断词规则
- 覆盖主要语言和地区
- 符合国际化排版标准

---

## 维护者指南

### 日常维护
1. **同步上游**: 定期检查上游更新，获取新的语言模式
2. **验证转换**: 使用 `ohos/test/generate_report.py` 验证 .hpb 文件正确性
3. **更新配置**: 在 `tex-hyphen.gni` 和 `build-tex.json` 中添加新语言

### 升级步骤
1. 更新 `third_party/tex-hyphen/` 到上游新版本
2. 检查 .tex 文件格式是否变化
3. 运行测试验证转换工具兼容性
4. 如需要，更新 `hyphen_pattern_processor.cpp`
5. 提交测试报告

---

## 参考资源

### 官方文档
- [上游仓库](https://github.com/hyphenation/tex-hyphen)
- [TeX Hyphenation 官网](http://www.hyphenation.org/tex)
- [CTAN 包页面](https://ctan.org/pkg/hyph-utf8)

### OH 相关
- [BUILD.gn](../BUILD.gn) - OH 构建配置
- [tex-hyphen.gni](../tex-hyphen.gni) - 语言配置
- [README_zh.md](../README_zh.md) - 中文使用说明

---

**文档维护**: 本文档随 OH 版本更新而更新
**最后更新**: 2026-02-08

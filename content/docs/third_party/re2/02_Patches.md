# Patch 详细分析

## 重要声明

> **RE2 在 OpenHarmony 中没有应用任何功能性 Patch。**
> 
> 这是一个特殊的零 Patch 库，直接使用标准上游代码。以下文档记录了这一事实，并对发现的差异文件进行分析。

## 2.1 Patch 清单总览

| Patch 文件 | 类型 | 是否应用 | 说明 |
|------------|------|----------|------|
| `ucs2.diff` | 历史文档 | 否 | Google 上游历史变更记录 |

**统计**: 功能性 Patch 数量 = **0**

## 2.2 ucs2.diff 分析

### 文件性质

**这不是 OpenHarmony 的 Patch**，而是 Google 从源代码控制系统导出的历史变更记录。

- **文件路径**: `third_party/re2/ucs2.diff`
- **文件大小**: ~20KB
- **内容**: 2009 年 RE2 移除 UCS-2 编码支持的完整变更记录

### 历史背景

2009 年 9 月 16 日，RE2 作者 Russ Cox 提交了变更（Change 12780686）从 RE2 中移除了 UCS-2 编码支持。

**移除原因**:
1. **技术限制**: UCS-2 需要 2-byte lookahead，而 RE2 设计基于 1-byte lookahead
2. **功能不完整**: 在 UCS-2 模式下无法支持 `^`（行首）和 `$`（行尾）锚点
3. **实验性质**: 该功能最初为 V8 JavaScript 引擎实验添加
4. **使用有限**: 实际使用场景很少

### 变更内容摘要

```diff
# 移除的组件:
1. kEncodingUCS2 编码枚举值
2. Regexp::UCS2 解析标志
3. AddRuneRangeUCS2() 编译器方法
4. AddUCS2Pair() 辅助方法
5. BigEndian() 字节序检测函数
6. kRegexpUnsupported 错误码
7. RE2::ErrorUnsupported 错误码
8. RE2::Options::EncodingUCS2 编码选项

# 涉及文件:
- re2/bitstate.cc
- re2/compile.cc
- re2/nfa.cc
- re2/parse.cc
- re2/re2.cc
- re2/re2.h
- re2/regexp.cc
- re2/regexp.h
- re2/testing/backtrack.cc
- re2/testing/tester.cc
```

### 在 OH 中的状态

**该文件仅为参考文档，不用于构建。**

用途可能是：
- 为需要 UCS-2 支持的开发者提供历史参考
- 作为代码考古资料保留
- 文档说明 UCS-2 为何不存在于当前代码

## 2.3 为什么 RE2 没有 Patch

### 设计哲学

RE2 的设计目标之一是**最小化复杂性和依赖**，这使得它：
1. **高度可移植**: 不依赖特定平台特性
2. **易于集成**: 标准 C++ 实现
3. **行为一致**: 跨平台行为可预测

### OH 使用方式

OpenHarmony 使用 RE2 的方式：
1. **纯头文件/源文件引用**: 直接使用 RE2 的公共 API
2. **不修改内部实现**: 不需要定制功能
3. **标准编译**: 通过 BUILD.gn 进行常规编译

### 依赖模块的使用模式

**gRPC 的使用**:
```cpp
// 直接使用 RE2 公共 API
#include "re2/re2.h"
RE2::FullMatch(uri, pattern);
```

这种模式不需要修改 RE2 内部实现。

## 2.4 零 Patch 的优势

### 对维护者

| 优势 | 说明 |
|------|------|
| **升级简单** | 无需合并 Patch，直接替换上游代码 |
| **风险可控** | 不引入自定义 Bug |
| **代码清晰** | 与上游代码完全一致 |

### 对开发者

| 优势 | 说明 |
|------|------|
| **文档一致** | 参考上游文档即可 |
| **行为可预测** | 与上游行为完全一致 |
| **问题可复现** | 可在上游仓库复现问题 |

## 2.5 升级建议

### 无 Patch 升级流程

```
1. 下载新版本上游代码
2. 替换所有源文件（保留 BUILD.gn, bundle.json 等 OH 配置文件）
3. 更新 README.OpenSource 中的版本号
4. 检查 BUILD.gn 源文件列表是否需要更新
5. 检查 abseil-cpp 版本兼容性
6. 构建并测试
7. 验证 gRPC 等依赖模块功能
```

### 注意事项

1. **ABI 兼容性**: 检查新版本是否破坏 ABI，需同步更新 libre2.map
2. **API 变更**: 关注 RE2 公共 API 的变更日志
3. **依赖更新**: 新版本可能要求更新的 abseil-cpp
4. **测试覆盖**: 重点测试 gRPC 的 URI 匹配功能

## 2.6 与其他库的对比

| 库 | OH Patch 数量 | 说明 |
|----|--------------|------|
| **RE2** | 0 | 零 Patch，标准上游 |
| curl | 较多 | HTTP/3、OH 特有功能 |
| openssl | 较多 | 国密支持、平台适配 |
| abseil-cpp | 少量 | 构建适配、符号导出 |

RE2 的零 Patch 策略在 OH 第三方库中属于特例，体现了该库的高度可移植性和成熟度。

## 2.7 总结

| 项目 | 结论 |
|------|------|
| **功能性 Patch** | 无 |
| **构建配置** | BUILD.gn (OH 特有) |
| **历史文档** | ucs2.diff (非 Patch) |
| **维护难度** | 极低 |
| **升级风险** | 低 |

RE2 是 OpenHarmony 中维护最简单的第三方库之一，其零 Patch 策略大大降低了长期维护成本。

---

**上一篇**: [01_Overview.md](./01_Overview.md) | **下一篇**: [03_Build_Integration.md](./03_Build_Integration.md)

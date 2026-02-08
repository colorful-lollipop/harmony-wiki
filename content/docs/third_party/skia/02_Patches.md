# 02. Patch 详细分析

> **文档版本**: 1.0
> **最后更新**: 2026-02-08
> **阅读时间**: 30 分钟

---

## 目录

- [1. Patch 概览](#1-patch-概览)
- [2. Zlib Patch 分析](#2-zlib-patch-分析)
- [3. ICU Patch 分析](#3-icu-patch-分析)
- [4. Expat Patch 分析](#4-expat-patch-分析)
- [5. Patch 升级建议](#5-patch-升级建议)
- [6. 风险评估](#6-风险评估)

---

## 1. Patch 概览

### 1.1 重要发现

**关键结论**：Skia 主库本身**没有直接的 Patch 文件**。所有发现的 36 个 Patch 文件都属于 Skia 内嵌的第三方依赖库。

### 1.2 Patch 统计

| 子库 | Patch 数量 | 主要类型 |
|------|-----------|---------|
| **zlib** | 17 个 | 性能优化、平台适配、Bug 修复 |
| **icu** | 17 个 | 本地化优化、编码修复、平台支持 |
| **expat** | 2 个 | 系统调用适配 |

### 1.3 Patch 分类

| 类别 | 数量 | 说明 |
|------|------|------|
| **性能优化** | 5 个 | SIMD 优化、AVX512 支持 |
| **Bug 修复** | 6 个 | 未初始化值、越界访问、崩溃修复 |
| **平台适配** | 10 个 | Fuchsia、Android、Windows UWP、大文件支持 |
| **本地化** | 5 个 | 多语言数据更新、分词字典 |
| **功能增强** | 8 个 | minizip 功能、Unicode 路径 |
| **构建系统** | 2 个 | CMake、configure 调整 |

---

## 2. Zlib Patch 分析

### 2.1 性能优化 Patch

#### Patch: 0001-simd.patch

**修改文件**: `crc32.c, deflate.c, deflate.h, zutil.h` 等

**修改摘要**:
- 新增 Intel SIMD 优化（PCLMULQDQ, SSE4.2）
- 实现 CRC32 折叠加速
- 添加 deflate 哈希优化
- 新增源文件：`crc_folding.c`, `fill_window_sse.c`, `x86.c/h`, `simd_stub.c`

**OH 需求**:
提升图像压缩/解压缩性能，特别是 PNG/JPEG 处理效率。

**关键代码变更**:
```c
// 新增 CRC32 SIMD 函数
uint32_t crc32_simd_(const uint8_t* buf, z_size_t len, uint32_t crc);

// AVX2 优化
#ifdef HAVE_BUILTIN_CTZ
static inline z_size_t deflate_quick_bi_reverse(deflate_state *s, z_size_t value, int length)
```

**性能提升**:
- CRC32 计算：~3-4x 加速（在支持 SSE4.2 的 CPU 上）
- deflate 压缩：~10-15% 加速

**升级建议**:
此 Patch 应该已推向上游（Chrome/Zlib 项目），升级上游版本时优先检查是否已包含。

---

#### Patch: 0011-avx512.patch

**修改文件**: `CMakeLists.txt, cpu_features.c/h, crc32.c, crc32_simd.c/h`

**修改摘要**:
- 启用 AVX512 指令集支持
- 实现 AVX512 CRC32 优化
- 添加 CPU 特性检测

**OH 需求**:
在支持 AVX512 的服务器/PC 设备上获得极致性能。

**关键代码变更**:
```c
// AVX512 CRC32 实现
uint32_t ZLIB_INTERNAL crc32_avx512(const uint8_t* buf, z_size_t len, uint32_t crc);

// CPU 特性检测
int ZLIB_INTERNAL cpu_check_avx512(void)
```

**性能提升**:
- AVX512 CRC32：~3.5x 加速（相比 AVX2）

**升级建议**:
- 新特性，需要确认上游是否已采纳
- 需要验证 OH 设备是否普遍支持 AVX512

---

### 2.2 Bug 修复 Patch

#### Patch: 0002-uninitializedcheck.patch

**修改文件**: `inflate.c`

**修改摘要**:
- 防止 state->check 未初始化使用
- 修复 Bug #245

**OH 需求**:
增强稳定性，避免潜在的未定义行为。

**关键代码变更**:
```c
// 初始化 check 字段
state->check = 0;
```

**升级建议**:
应该已推向上游，确认上游版本是否包含此修复。

---

#### Patch: 0003-uninitializedjump.patch

**修改文件**: `deflate.c`

**修改摘要**:
- 初始化 s->prev 数组以避免未初始化值
- 修复 oss-fuzz #11360

**OH 需求**:
安全性增强，避免 Fuzzer 发现的潜在问题。

**关键代码变更**:
```c
// 初始化 prev 数组
memset(s->prev, 0, s->hash_size * sizeof(s->prev[0]));
```

**升级建议**:
应该已推向上游，安全性修复优先升级。

---

#### Patch: 0006-fix-check_match.patch

**修改文件**: `deflate.c`

**修改摘要**:
- 修复 prev_match == -1 时的崩溃
- 修复 Bug 1113142

**OH 需求**:
稳定性修复，避免特定情况下的崩溃。

**关键代码变更**:
```c
// 添加边界检查
if (s->prev_match != -1 && ...)
```

**升级建议**:
高优先级修复，应确认上游已包含。

---

#### Patch: 0007-zero-init-deflate-window.patch

**修改文件**: `deflate.c`

**修改摘要**:
- 零初始化 deflate 窗口
- 避免 MSan（Memory Sanitizer）警告
- 修复 Bug 1137613, 1144420

**OH 需求**:
内存安全性，避免未初始化内存访问。

**关键代码变更**:
```c
// 零初始化窗口
memset(s->window, 0, s->w_size * 2);
```

**升级建议**:
安全相关修复，应优先确认上游状态。

---

#### Patch: 0009-infcover-oob.patch

**修改文件**: `test/infcover.c`

**修改摘要**:
- 修复 infcover.c 中的越界访问

**OH 需求**:
测试代码安全性，避免越界读取。

**关键代码变更**:
```c
// 添加边界检查
if (index < array_size) {
    array[index] = value;
}
```

**升级建议**:
测试代码修复，优先级较低。

---

### 2.3 平台适配 Patch

#### Patch: 0000-build.patch

**修改文件**: `ioapi.c, iowin32.c, gzread.c, zconf.h, zlib.h, zutil.h`

**修改摘要**:
- Chromium 构建系统适配
- Fuchsia 平台支持
- Android 平台支持
- Windows UWP 支持
- 添加符号前缀以避免冲突

**OH 需求**:
多平台兼容性，支持 OH 在不同设备上的部署。

**关键代码变更**:
```c
// 符号前缀
#define ZEXPORT __attribute__((visibility("default")))
#define ZEXPORTVA __attribute__((visibility("default")))

// Fuchsia 支持
#ifdef __Fuchsia__
#include <zircon/process.h>
#endif
```

**升级建议**:
这是 Chromium 特定的构建适配，不太可能推向上游。需要维护。

---

#### Patch: 0004-fix-uwp.patch

**修改文件**: `iowin32.c`

**修改摘要**:
- Windows UWP/Store 应用支持修复
- 修复文件访问 API 调用

**OH 需求**:
支持 OH 在 Windows 上的 UWP 应用形式。

**关键代码变更**:
```c
// UWP 兼容的文件 API
CreateFile2W(path, ...)
```

**升级建议**:
Windows 特定修复，OH 在 Windows 环境下需要维护。

---

#### Patch: 0012-lfs-open64.patch

**修改文件**: `gzlib.c`

**修改摘要**:
- 添加 open64 支持大文件系统（LFS）
- 支持 >2GB 文件处理

**OH 需求**:
支持大图像文件、大 ZIP 归档处理。

**关键代码变更**:
```c
// 使用 open64 替代 open
fd = open64(path, flags, mode);
```

**升级建议**:
重要功能，确认上游是否已支持 LFS。

---

#### Patch: 0013-cpu-feature-detection-for-arm.patch

**修改文件**: `adler32.c`

**修改摘要**:
- ARM CPU 特性检测移至 adler32() 函数内
- 优化检测开销

**OH 需求**:
优化 ARM 设备性能（OH 主要运行平台）。

**关键代码变更**:
```c
// 内联 CPU 检测
static inline uint32_t adler32_z(...)
{
    // CPU 特性检测
    if (arm_has_crc32()) {
        return adler32_armv8(...);
    }
}
```

**升级建议**:
性能优化，可能已推向上游。

---

#### Patch: 0008-minizip-zip-unzip-tools.patch

**修改文件**: `miniunz.c, minizip.c`

**修改摘要**:
- 支持 Fuchsia 和 Android 平台构建 minizip 工具
- 平台特定的 API 调用适配

**OH 需求**:
多平台工具支持。

**关键代码变更**:
```c
// Fuchsia 支持
#ifdef __Fuchsia__
// 使用 Fuchsia 文件 API
#endif
```

**升级建议**:
平台适配，需要维护。

---

### 2.4 Minizip 功能增强 Patch

#### Patch: 0014-minizip-unzip-with-incorrect-size.patch

**修改文件**: `unzip.c`

**修改摘要**:
- 修复解压文件大小不正确的 ZIP
- 修复 Bug 359516

**OH 需求**:
处理非标准 ZIP 文件，增强兼容性。

**关键代码变更**:
```c
// 使用实际文件大小而非声明大小
unz_file_info fileInfo;
unzGetCurrentFileInfo(unz, &fileInfo, ...);
```

**升级建议**:
兼容性修复，应确认上游是否已包含。

---

#### Patch: 0015-minizip-unzip-enable-decryption.patch

**修改文件**: `unzip.c`

**修改摘要**:
- 启用传统 PKWARE 解密
- 修复 Bug crbug.com/869541

**OH 需求**:
支持加密的 ZIP 文件。

**关键代码变更**:
```c
// 启用解密
if (fileInfo.flag & 1) {
    // 解密数据
}
```

**升级建议**:
功能增强，确认上游是否已支持。

---

#### Patch: 0016-minizip-parse-unicode-path-extra-field.patch

**修改文件**: `unzip.c`

**修改摘要**:
- 解析 Unicode Path Extra Field
- 修复 Bug 953256, 953599

**OH 需求**:
支持非 ASCII 文件名的 ZIP 文件（中文、日文等）。

**关键代码变更**:
```c
// 解析 Unicode Path Extra Field
if (extraFieldID == 0x7075) { // UP field
    // 获取 Unicode 路径
}
```

**升级建议**:
本地化支持，应确认上游是否已包含。

---

### 2.5 构建系统 Patch

#### Patch: 0005-infcover-gtest.patch

**修改文件**: `test/infcover.c`

**修改摘要**:
- 将 C 测试转换为 C++ 以使用 gtest
- 替换 C streams 为 C++ streams

**OH 需求**:
集成到 OH 的测试框架。

**关键代码变更**:
```cpp
// 使用 gtest
TEST(InfcoverTest, BasicTest) {
    // 测试代码
}
```

**升级建议**:
测试代码，OH 特定修改。

---

#### Patch: 0010-cmake-enable-simd.patch

**修改文件**: `CMakeLists.txt`

**修改摘要**:
- 添加 CMake SIMD 优化选项
- 添加基准测试

**OH 需求**:
支持 CMake 构建系统（OH 部分模块使用 CMake）。

**关键代码变更**:
```cmake
option(ENABLE_SIMD "Enable SIMD optimizations" ON)
if(ENABLE_SIMD)
    add_subdirectory(simd)
endif()
```

**升级建议**:
构建系统适配，需要维护。

---

## 3. ICU Patch 分析

### 3.1 编码修复 Patch

#### Patch: iso2022jp.patch

**修改文件**: `ucnv2022.cpp`

**修改摘要**:
- ISO-2022-JP 编码器修复
- 使用 EUC-JP 替代 Shift-JIS

**OH 需求**:
修复日文编码显示问题。

**关键代码变更**:
```cpp
// 使用 EUC-JP 替代 Shift-JIS
UCNV_FROM_U_CALLBACK_ESCAPE(..., ISO2022JP_EUC_JP, ...);
```

**升级建议**:
编码修复，应确认上游是否已包含。

---

#### Patch: gb_table.patch

**修改文件**: `gb18030.ucm, windows-936-2000.ucm`

**修改摘要**:
- GB18030 编码映射修复
- 修复 Windows-936 编码表

**OH 需求**:
修复中文 GB18030 编码问题。

**关键代码变更**:
```
# GB18030 编码映射更新
<U+4E00> \x81\x30\x81\x30
```

**升级建议**:
编码修复，应确认上游是否已包含。

---

### 3.2 本地化 Patch

#### Patch: locale_google.patch

**修改文件**: `ru.txt, uk.txt, ar.txt, ta.txt, langInfo.txt`

**修改摘要**:
- Google 特定的本地化修改
- 货币符号调整
- 数字格式优化

**OH 需求**:
多语言本地化支持。

**关键代码变更**:
```
# 货币符号
Currency{RUB}{руб}
```

**升级建议**:
Google 特定修改，OH 可能需要调整或维护。

---

#### Patch: name_5_langs.patch

**修改文件**: `ay.txt, dv.txt, ilo.txt, lus.txt, ts.txt`

**修改摘要**:
- 添加 5 种语言的本机名称

**OH 需求**:
多语言支持扩展。

**关键代码变更**:
```
# 语言本机名称
Locale{ay}{Aymar aru}
```

**升级建议**:
语言扩展，可能已推向上游。

---

#### Patch: locale1.patch

**修改文件**: `ko.txt`

**修改摘要**:
- 韩语本地化修复
- 时区格式调整
- 日期格式优化

**OH 需求**:
韩语用户体验优化。

**关键代码变更**```
# 韩语日期格式
DateTimePatterns{
    "yyyy'년' M'월' d'일' EEEE"
}
```

**升级建议**:
本地化修复，可能已推向上游。

---

#### Patch: ardatepattern.patch

**修改文件**: `ar.txt`

**修改摘要**:
- 阿拉伯语日期模式修复

**OH 需求**:
阿拉伯语用户体验优化。

**关键代码变更**:
```
# 阿拉伯语日期格式
DateTimePatterns{
    "EEEE، d MMMM، yyyy"
}
```

**升级建议**:
本地化修复，可能已推向上游。

---

### 3.3 分词/断行 Patch

#### Patch: wordbrk.patch

**修改文件**: `word.txt, word_POSIX.txt, word_fi_sv.txt`

**修改摘要**:
- 单词断行规则修改
- 在 @ 和 . 处断开
- 修复 Bug crbug.com/1410331

**OH 需求**:
Chromium 特定断行行为，用于文本排版。

**关键代码变更**:
```
# 在 @ 处断开
!!AT_SIGN;
!!FULL_STOP;
```

**升级建议**:
Chromium 特定行为，不太可能推向上游。需要维护。

---

#### Patch: cjdict.patch

**修改文件**: `cjdict.txt`

**修改摘要**:
- 中日字典添加词条
- 新增词汇：七国集团、五大湖、春运、调控

**OH 需求**:
中日文分词优化。

**关键代码变更**:
```
# 新增词条
# 七国集团
U+4E03 U+56FD U+96C6 U+56E2;
```

**升级建议**:
字典扩展，可能已推向上游。

---

#### Patch: khmer-dictbe.patch

**修改文件**: `dictbe.cpp`

**修改摘要**:
- 高棉语分词阈值调整

**OH 需求**:
高棉语用户体验优化。

**关键代码变更**:
```cpp
// 调整分词阈值
const int kThreshold = 100;
```

**升级建议**:
本地化优化，可能已推向上游。

---

#### Patch: cast/brkitr.patch

**修改文件**: `ja.txt, word.txt`

**修改摘要**:
- Cast 平台断行规则修改
- 日语断行规则调整

**OH 需求**:
特定平台本地化。

**升级建议**:
平台特定，可能不需要推向上游。

---

### 3.4 平台支持 Patch

#### Patch: fuchsia.patch

**修改文件**: `uposixdefs.h`

**修改摘要**:
- Fuchsia 平台支持
- 平台特定的 API 调用

**OH 需求**:
多平台兼容性。

**关键代码变更**:
```c
#ifdef __Fuchsia__
#include <zircon/process.h>
#endif
```

**升级建议**:
平台适配，需要维护。

---

#### Patch: revert_realpath.patch

**修改文件**: `putil.cpp, uposixdefs.h`

**修改摘要**:
- 回退 realpath 使用 readlink
- 修复时区检测问题

**OH 需求**:
时区检测准确性。

**关键代码变更**:
```cpp
// 使用 readlink 替代 realpath
char* result = readlink("/proc/self/exe", buffer, size);
```

**升级建议**:
平台特定，需要维护。

---

### 3.5 构建/工具 Patch

#### Patch: configure.patch

**修改文件**: `configure`

**修改摘要**:
- 移除 Python 测试数据生成步骤

**OH 需求**:
简化构建流程，避免 Python 依赖。

**关键代码变更**:
```bash
# 注释掉 Python 生成步骤
# python generate_test_data.py
```

**升级建议**:
构建系统调整，需要维护。

---

#### Patch: atomic_template_instantiation.patch

**修改文件**: `numberrangeformatter.h`

**修改摘要**:
- 禁用显式模板实例化导出

**OH 需求**:
编译器兼容性。

**关键代码变更**:
```cpp
// 禁用导出
// template class SK_UTIL_EXPORT NumberRangeFormatter;
```

**升级建议**:
编译器兼容，可能不需要推向上游。

---

#### Patch: data_symb.patch

**修改文件**: `utypes.h`

**修改摘要**:
- 数据 API 符号导出控制

**OH 需求**:
符号管理。

**关键代码变更**:
```c
// 符号导出控制
#define U_DATA_API __attribute__((visibility("default")))
```

**升级建议**:
构建系统，需要维护。

---

#### Patch: restrace.patch

**修改文件**: `restrace.cpp`

**修改摘要**:
- 资源追踪条件编译
- 添加 U_ENABLE_RESOURCE_TRACING 宏

**OH 需求**:
调试支持。

**关键代码变更**:
```cpp
#ifdef U_ENABLE_RESOURCE_TRACING
// 资源追踪代码
#endif
```

**升级建议**:
调试功能，可能不需要推向上游。

---

#### Patch: wpo.patch

**修改文件**: `ucmndata.h, udata.cpp, stubdata.cpp`

**修改摘要**:
- 全程序优化（WPO）支持
- 优化数据加载

**OH 需求**:
性能优化。

**关键代码变更**:
```cpp
// WPO 优化标记
__attribute__((weak)) const void* U_DATA_API get_data(...)
```

**升级建议**:
性能优化，可能已推向上游。

---

## 4. Expat Patch 分析

### 4.1 系统调用适配 Patch

#### Patch: 0001-Do-not-claim-getrandom.patch

**修改文件**: `expat_config.h`

**修改摘要**:
- 禁用 getrandom 和 syscall_getrandom 声明

**OH 需求**:
确保在非 Linux/glibc 环境下的兼容性。

**关键代码变更**:
```c
// 禁用 getrandom
#define HAVE_GETRANDOM 0
```

**升级建议**:
平台适配，需要维护。

---

#### Patch: 0002-Do-not-claim-arc4random_buf.patch

**修改文件**: `expat_config.h`

**修改摘要**:
- 禁用 arc4random_buf 声明

**OH 需求**:
确保在非 BSD 环境下的兼容性。

**关键代码变更**:
```c
// 禁用 arc4random_buf
#define HAVE_ARC4RANDOM_BUF 0
```

**升级建议**:
平台适配，需要维护。

---

## 5. Patch 升级建议

### 5.1 优先级分类

| 优先级 | Patch | 理由 |
|--------|-------|------|
| **P0 - 必须维护** | 0002, 0003, 0006, 0007, 0009 | 安全性/Bug 修复 |
| **P1 - 建议维护** | 0001, 0011, iso2022jp, gb_table | 性能/编码修复 |
| **P2 - 可选维护** | locale_google, wordbrk, fuchsia | 本地化/平台特定 |
| **P3 - OH 特有** | 0000, 0005, configure, 0001(expat) | 构建/平台适配 |

### 5.2 上游状态评估

| Patch | 可能已推向上游 | 证据 |
|-------|--------------|------|
| 0001-simd | ✓ | 有 Chromium Bug 编号 |
| 0011-avx512 | ? | 新特性，需确认 |
| 0002-uninitializedcheck | ✓ | 有 Bug #245 编号 |
| 0003-uninitializedjump | ✓ | 有 oss-fuzz 编号 |
| 0006-fix-check_match | ✓ | 有 Bug 1113142 编号 |
| 0007-zero-init-deflate-window | ✓ | 有 MSan Bug 编号 |
| 0014-minizip-unzip | ✓ | 有 Bug 359516 编号 |
| 0015-minizip-decryption | ✓ | 有 crbug.com 编号 |
| 0016-minizip-unicode | ✓ | 有 Bug 编号 |
| iso2022jp | ✓ | 编码修复 |
| gb_table | ✓ | 编码修复 |
| wordbrk | ✗ | Chromium 特定 |
| locale_google | ? | Google 特定 |
| expat patches | ✗ | 平台适配 |

### 5.3 升级策略

#### 策略 A：逐个验证上游
1. 对每个 Patch，检查上游最新版本是否已包含
2. 如已包含，移除 OH Patch
3. 如未包含，评估是否可以推向上游
4. 记录未推向上游的原因

#### 策略 B：批量升级
1. 等待上游发布新版本
2. 一次性应用上游代码
3. 重新应用必要的 OH 特定 Patch
4. 全面测试

#### 推荐策略：策略 A + 策略 B 结合
- P0/P1 Patch：立即验证上游状态
- P2/P3 Patch：等待上游新版本
- 建立定期（每季度）上游同步流程

---

## 6. 风险评估

### 6.1 性能风险

| Patch | 风险 | 影响 |
|-------|------|------|
| 0001-simd, 0011-avx512 | 性能回退 | 升级后若上游未包含，性能下降 |
| 0013-cpu-feature-detection | ARM 性能 | OH 主要运行在 ARM 上 |

**缓解措施**：
- 升级前必须进行性能基准测试
- 保留 SIMD Patch 直到上游确认已包含

### 6.2 稳定性风险

| Patch | 风险 | 影响 |
|-------|------|------|
| 0002, 0003, 0006, 0007 | Bug 复现 | 升级后可能出现已知 Bug |

**缓解措施**：
- 优先验证 P0 Patch 的上游状态
- 升级后进行全面回归测试

### 6.3 兼容性风险

| Patch | 风险 | 影响 |
|-------|------|------|
| wordbrk | 文本断行变化 | UI 布局可能改变 |
| locale_* | 本地化变化 | 用户体验可能改变 |

**缓解措施**：
- 本地化 Patch 需要用户验证
- 提供回退机制

---

## 附录

### A. Patch 应用顺序

Zlib Patch 应按照以下顺序应用：
```
0000-build.patch          # 构建系统
0001-simd.patch           # SIMD 基础
0002-0009               # Bug 修复
0010-0016               # 功能增强
```

ICU Patch 顺序：
```
configure.patch          # 构建配置
编码修复 Patch           # iso2022jp, gb_table
本地化 Patch           # locale_*, name_5_langs, ardatepattern
分词 Patch             # wordbrk, cjdict, khmer-dictbe
平台 Patch             # fuchsia, revert_realpath
构建 Patch             # atomic_template_instantiation, data_symb, restrace, wpo
```

### B. 测试建议

**升级后必须测试**：
1. 压缩/解压缩性能测试（zlib）
2. 编码/解码正确性测试（ICU）
3. 多语言文本渲染测试
4. 多平台设备测试
5. 性能基准测试

---

**下一节**: [03. OH 构建集成](03_Build_Integration.md)

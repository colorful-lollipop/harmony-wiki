# 原始库简介 - bzip2

---

## 📌 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | bzip2 / libbzip2 |
| **当前版本** | 1.0.8 |
| **发布日期** | 2019-07-13 |
| **作者** | Julian Seward |
| **许可证** | bzip2 License (BSD-style) |
| **上游地址** | https://sourceware.org/bzip2/ |
| **源码仓库** | https://sourceware.org/git/?p=bzip2.git |
| **测试套件** | https://sourceware.org/git/bzip2-tests.git |

---

## 🎯 功能概述

bzip2 是一个高质量的无损数据压缩库和命令行工具，使用**块排序算法（Burrows-Wheeler Transform）**实现高压缩率。

### 核心特点

| 特性 | 说明 |
|------|------|
| **高压缩率** | 压缩率在最佳技术的 10%-15% 范围内 |
| **快速解压** | 解压速度是最佳技术的约 6 倍 |
| **快速压缩** | 压缩速度是最佳技术的约 2 倍 |
| **无专利** | 不使用任何专利算法，可自由使用 |
| **跨平台** | 支持 Unix/Linux、Windows、macOS 等 |
| **成熟稳定** | 自 1996 年发布以来被广泛使用 |

### 适用场景

bzip2 特别适合：
- ✅ **文本数据压缩**（源代码、文本文件）
- ✅ **需要高压缩率**但可牺牲速度的场景
- ✅ **长期存储**（数据归档、备份）
- ✅ **需要可逆无损压缩**的场景

不太适合：
- ❌ 已压缩过的数据（JPEG、MP3 等）
- ❌ 实时传输压缩（压缩速度相对较慢）
- ❌ 资源受限的嵌入式系统（内存占用较高）

---

## 🔧 核心算法

### 块排序（Burrows-Wheeler Transform, BWT）

bzip2 的核心是 Burrows-Wheeler 变换，这是一种**可逆**的块排序算法：

```
原始数据 → BWT → 变换后的数据 → Huffman 编码 → 最终压缩数据
        ↑                                                   ↓
        └───────────────── Huffman 解码 ← BWT 逆变换 ← 压缩数据
```

### 算法流程

1. **分块**: 将数据分成 100KB-900KB 的块
2. **BWT 变换**: 对每个块应用 Burrows-Wheeler 变换
3. **Move-to-Front (MTF)**: 将频繁字符移到前面
4. **Huffman 编码**: 对 MTF 结果进行 Huffman 编码
5. **Run-Length Encoding (RLE)**: 简单的游程编码

### 优势

- ✅ BWT 产生的数据非常适合 Huffman 编码
- ✅ 压缩率接近 PPM（预测部分匹配）等最佳技术
- ✅ 解压速度远快于压缩速度

---

## 📚 组件组成

### 上游源码结构

```
bzip2/
├── 核心库源码 (libbzip2)
│   ├── blocksort.c      # 块排序实现
│   ├── bzlib.c         # 库顶层函数
│   ├── bzlib.h         # 公共头文件
│   ├── bzlib_private.h # 私有头文件
│   ├── compress.c       # 压缩实现
│   ├── crctable.c      # CRC 表生成
│   ├── decompress.c     # 解压实现
│   ├── huffman.c       # Huffman 编码
│   └── randtable.c     # 随机表生成
│
├── 命令行工具
│   ├── bzip2.c         # bzip2 命令行工具
│   └── bzip2recover.c # 损坏文件恢复工具
│
├── 测试和辅助工具
│   ├── dlltest.c       # Windows DLL 测试
│   ├── mk251.c        # 测试程序
│   ├── spewG.c        # 测试程序
│   └── unzcrash.c     # 崩溃测试
│
└── 文档
    ├── README          # 项目说明
    ├── LICENSE         # 许可证
    ├── CHANGES        # 变更历史
    └── manual.pdf/html/xml  # 用户手册
```

### OpenHarmony 选择的组件

OH 仅选择**核心库源码**，不包含：
- ❌ 命令行工具（bzip2, bzcat, bunzip2, bzip2recover）
- ❌ 测试和辅助工具
- ❌ 文档（上游已有完整文档）

---

## 🔌 API 概览

### API 分层

bzip2 提供**三层 API**，适应不同使用场景：

#### 1. 低层 API（基于流）

适合需要细粒度控制的场景：

```c
// 压缩
BZ2_bzCompressInit(strm, blockSize100k, verbosity, workFactor);
BZ2_bzCompress(strm, action);
BZ2_bzCompressEnd(strm);

// 解压
BZ2_bzDecompressInit(strm, verbosity, small);
BZ2_bzDecompress(strm);
BZ2_bzDecompressEnd(strm);
```

#### 2. 高层 API（基于文件）

适合直接操作文件的场景：

```c
// 读取（解压）
BZFILE* b = BZ2_bzReadOpen(&bzerror, fp, verbosity, small, unused, nUnused);
BZ2_bzRead(&bzerror, b, buf, len);
BZ2_bzReadClose(&bzerror, b);

// 写入（压缩）
BZFILE* b = BZ2_bzWriteOpen(&bzerror, fp, blockSize100k, verbosity, workFactor);
BZ2_bzWrite(&bzerror, b, buf, len);
BZ2_bzWriteClose(&bzerror, b, abandon, &nbytes_in, &nbytes_out);
```

#### 3. 便捷 API（内存到内存）

适合一次性处理所有数据的场景：

```c
// 压缩
BZ2_bzBuffToBuffCompress(dest, &destLen, source, sourceLen,
                         blockSize100k, verbosity, workFactor);

// 解压
BZ2_bzBuffToBuffDecompress(dest, &destLen, source, sourceLen,
                           small, verbosity);
```

### 参数说明

| 参数 | 范围 | 说明 |
|------|------|------|
| `blockSize100k` | 1-9 | 块大小 ×100KB (1=100KB, 9=900KB)，越大压缩率越高但内存占用越大 |
| `verbosity` | 0-4 | 日志详细程度（0=无输出，4=最详细） |
| `workFactor` | 0-250 | 工作因子，控制压缩阶段的工作量（默认 30） |
| `small` | 0/1 | 是否使用低内存模式（1=内存占用少但速度慢） |

### 返回码

| 返回码 | 含义 |
|--------|------|
| `BZ_OK` | 成功 |
| `BZ_STREAM_END` | 流结束 |
| `BZ_SEQUENCE_ERROR` | 操作顺序错误 |
| `BZ_PARAM_ERROR` | 参数错误 |
| `BZ_MEM_ERROR` | 内存不足 |
| `BZ_DATA_ERROR` | 数据错误（损坏的压缩数据） |
| `BZ_DATA_ERROR_MAGIC` | 魔数错误（不是有效的 bzip2 文件） |
| `BZ_IO_ERROR` | I/O 错误 |
| `BZ_CONFIG_ERROR` | 配置错误 |
| `BZ_UNEXPECTED_EOF` | 意外的文件结束 |
| `BZ_OUTBUFF_FULL` | 输出缓冲区不足 |

---

## 📊 性能特性

### 压缩率 vs 其他算法

| 算法 | 压缩率 | 压缩速度 | 解压速度 | 内存占用 |
|------|--------|----------|----------|----------|
| **gzip** | 中等 | 快 | 快 | 低 |
| **bzip2** | 高 | 中等 | 快 | 中等 |
| **xz** | 极高 | 慢 | 中等 | 高 |
| **lzma** | 极高 | 慢 | 中等 | 高 |
| **ppmd** | 极高 | 很慢 | 快 | 高 |

### 典型数据压缩率

| 数据类型 | 原始大小 | bzip2 压缩后 | 压缩率 |
|----------|----------|-------------|--------|
| C 源代码 | 10 MB | ~1.5-2 MB | 15-20% |
| XML 文本 | 5 MB | ~0.8-1 MB | 16-20% |
| 日志文件 | 20 MB | ~2-5 MB | 10-25% |
| 二进制数据 | 10 MB | ~6-8 MB | 60-80% |

### 内存占用

| 块大小 | 压缩内存 | 解压内存 |
|--------|----------|----------|
| 100KB | ~4 MB | ~4 MB |
| 500KB | ~7 MB | ~7 MB |
| 900KB | ~11 MB | ~11 MB |

---

## 🏢 OpenHarmony 中的作用

### 在 OH 中的定位

bzip2 在 OpenHarmony 中的角色：

| 角色 | 说明 |
|------|------|
| **数据压缩库** | 为 OH 系统和上层应用提供数据压缩/解压缩功能 |
| **基础组件** | thirdparty 子系统的一部分 |
| **静态库** | 通过 `//third_party/bzip2:libbz2` 静态链接 |

### 典型使用场景（推测）

虽然未发现直接依赖者，bzip2 可能在以下场景被使用：

1. **系统更新**: 压缩/解压缩系统更新包
2. **数据归档**: 日志归档、数据备份
3. **资源压缩**: 应用资源文件的压缩存储
4. **网络传输**: 压缩网络传输数据
5. **间接使用**: 通过其他库（如某些解压缩工具）

### 为什么选择 bzip2

OpenHarmony 选择 bzip2 的原因：

| 原因 | 说明 |
|------|------|
| **成熟稳定** | 20+ 年历史，广泛验证 |
| **开源无专利** | 无法律风险 |
| **高压缩率** | 适合资源受限的移动设备 |
| **维护成本低** | 无 OH 特定修改，升级容易 |

---

## 📈 版本历史

### 主要版本里程碑

| 版本 | 发布日期 | 重大变更 |
|------|----------|----------|
| 0.9.0 | 1996-07-15 | 首次发布 |
| 1.0.0 | 2000-06-01 | 大文件支持、符号清理 |
| 1.0.5 | 2007-12-10 | 安全修复 |
| 1.0.6 | 2010-09-06 | CVE-2010-0405 修复 |
| 1.0.7 | 2019-06-27 | 多个安全修复 |
| 1.0.8 | 2019-07-13 | 当前版本，放宽 selector 限制 |

### OpenHarmony 使用的版本

**版本**: 1.0.8 (2019-07-13)

这是截至 2026 年的**最新稳定版本**，包含所有已知的安全修复。

---

## 🔗 相关资源

### 官方资源

- **项目主页**: https://sourceware.org/bzip2/
- **用户手册**: https://sourceware.org/bzip2/manual.html
- **邮件列表**: bzip2-devel@sourceware.org
- **FAQ**: https://sourceware.org/bzip2/faq.html

### 测试与验证

- **测试套件**: https://sourceware.org/git/bzip2-tests.git
- **已知测试**: 上游提供了完整的回归测试套件

### 第三方资源

- **Wikipedia**: https://en.wikipedia.org/wiki/Bzip2
- **算法介绍**: Burrows-Wheeler Transform、Huffman 编码

---

## ⚠️ 注意事项

### 使用建议

1. **块大小选择**:
   - 一般场景：推荐 `blockSize100k=9`（最高压缩率）
   - 内存受限：推荐 `blockSize100k=5`
   - 实时压缩：推荐 `blockSize100k=1`（最快）

2. **小数据优化**:
   - 小于 100KB 的数据，压缩率不明显，考虑批量压缩

3. **已压缩数据**:
   - 不要重复压缩已压缩的数据（如 JPEG、MP3）

### 已知限制

1. **单线程**: bzip2 是单线程的，不适合多核并行压缩
2. **内存占用**: 大块需要较多内存
3. **压缩速度**: 相比 gzip 较慢

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
**参考**: bzip2 1.0.8 官方文档

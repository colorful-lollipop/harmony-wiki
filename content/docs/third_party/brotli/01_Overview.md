# Brotli 原始库简介

## 基本信息

| 项目 | 内容 |
|-----|------|
| **库名称** | Brotli |
| **当前版本** | v1.1.0 |
| **许可证** | MIT License |
| **上游地址** | http://github.com/google/brotli |
| **首次发布** | 2015-08-11 |
| **维护者** | Google Chrome 团队 |

## 功能概述

Brotli 是一种通用目的的无损数据压缩算法，其核心特点包括：

### 技术特性

1. **压缩算法组合**
   - LZ77 算法的现代变体
   - Huffman 编码
   - 二阶上下文建模

2. **性能特点**
   - 压缩比：与当前最好的通用压缩方法相当
   - 压缩速度：与 deflate 相似
   - 解压速度：通常比压缩速度更快

3. **压缩格式**
   - 定义于 [RFC 7932](https://tools.ietf.org/html/rfc7932)
   - 被称为"流式"格式（stream format）
   - 不包含元信息（如校验和或未压缩数据长度）

### 应用场景

| 场景 | 说明 |
|-----|------|
| **HTTP 压缩** | Web 传输的高效压缩（比 gzip 压缩比高 15-25%） |
| **字体压缩** | WOFF2 Web 字体格式使用 Brotli |
| **文件压缩** | 通用无损压缩 |
| **图像格式** | JPEG XL 等现代图像格式采用 Brotli 作为后端 |

## API 简介

### 编码接口（Encoder）

```c
// 创建编码器实例
BrotliEncoderState* BrotliEncoderCreateInstance(
    brotli_alloc_func alloc_func,
    brotli_free_func free_func,
    void* opaque
);

// 设置压缩参数
BrotliEncoderSetParameter(BrotliEncoderState* state,
                          BROTLIEncoderParameter param,
                          uint32_t value);

// 执行压缩
BROTLI_BOOL BrotliEncoderCompressStream(
    BrotliEncoderState* state,
    BrotliEncoderOperation op,
    size_t* available_in,
    const uint8_t* buffer_in,
    size_t* available_out,
    uint8_t* buffer_out,
    size_t* total_out
);

// 销毁实例
void BrotliEncoderDestroyInstance(BrotliEncoderState* state);
```

### 解码接口（Decoder）

```c
// 创建解码器实例
BrotliDecoderState* BrotliDecoderCreateInstance(
    brotli_alloc_func alloc_func,
    brotli_free_func free_func,
    void* opaque
);

// 解码状态查询
BrotliDecoderResult BrotliDecoderDecompress(
    size_t* available_in,
    const uint8_t* buffer_in,
    size_t* available_out,
    uint8_t* buffer_out
);

// 错误信息获取
BrotliDecoderErrorCode BrotliDecoderGetErrorCode(
    const BrotliDecoderState* state
);
const char* BrotliDecoderErrorString(BrotliDecoderErrorCode error);
```

## 在 OpenHarmony 中的定位

### 核心作用

Brotli 在 OpenHarmony 系统中主要承担 **HTTP 压缩传输** 的角色：

1. **网络传输优化**
   - 为 curl 提供 Brotli 压缩支持
   - 实现比 gzip 更好的压缩率
   - 减少网络带宽占用

2. **资源压缩**
   - 支持 WebView 资源的压缩传输
   - 字体文件的压缩存储

### 集成策略

- **静态库/共享库**：提供 `brotli_shared` 共享库
- **头文件**：标准 Brotli 头文件（无 OH 特定 API）
- **依赖方式**：OH 模块通过 GN 构建系统依赖

### 与 OH 的关系

```
┌─────────────────────────────────────────┐
│           OpenHarmony                   │
├─────────────────────────────────────────┤
│  ┌───────────┐    ┌───────────┐         │
│  │   curl    │───▶│  brotli   │         │
│  └───────────┘    └───────────┘         │
│       │                  ▲              │
│       ▼                  │              │
│  HTTP 压缩          共享库              │
│  (Accept-Encoding: br)   │              │
└─────────────────────────────────────────┘
```

## 版本历史

| 版本 | 发布日期 | 主要变更 |
|-----|---------|---------|
| v1.1.0 | 2023-08-28 | 新增共享字典、CLI dictionary 选项等 |
| v1.0.9 | 2020-08-27 | 重新发布 v1.0.8 |
| v1.0.8 | 2020-08-27 | 修复 CVE-2020-8927 |
| v1.0.7 | 2018-10-23 | ARM 解码优化 |
| v1.0.0 | 2017-09-20 | 稳定 API 发布 |

## 参考资源

- **上游仓库**：http://github.com/google/brotli
- **官方文档**：http://brotli.org
- **RFC 7932**：https://tools.ietf.org/html/rfc7932
- **GitHub Actions**：https://github.com/google/brotli/actions

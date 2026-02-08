# zlib - OpenHarmony 适配概述

## 原始库简介

### 基本信息

| 属性 | 值 |
|-----|-----|
| **名称** | zlib |
| **版本** | 1.3.1 |
| **许可证** | zlib/libpng License |
| **上游维护者** | Jean-loup Gailly, Mark Adler |
| **上游地址** | https://github.com/madler/zlib |
| **发布日期** | 2024年1月 |

### 功能描述

zlib 是一个**通用的数据压缩库**，提供以下核心功能：

1. **压缩算法**
   - deflate: 标准压缩算法 (RFC 1951)
   - zlib 格式封装 (RFC 1950)
   - gzip 文件格式 (RFC 1952)

2. **核心特性**
   - 线程安全 (所有代码线程安全)
   - 内存可控 (可调压缩级别和内存使用)
   - 跨平台 (支持几乎所有主流平台)
   - 流式处理 (支持大数据流压缩/解压)

3. **提供的接口**
   - 基本压缩: `compress`, `compress2`, `uncompress`
   - deflate 流: `deflate*`, `inflate*` 系列
   - gzip 文件: `gzopen`, `gzread`, `gzwrite`, `gzclose`
   - 校验算法: `adler32`, `crc32`

### 代码结构

```
zlib/
├── 核心算法文件
│   ├── deflate.c/h    # deflate 压缩算法实现
│   ├── inflate.c/h    # inflate 解压算法实现
│   ├── trees.c/h      # Huffman 树编码
│   ├── adler32.c      # Adler-32 校验
│   ├── crc32.c/h      # CRC32 校验
│   └── zutil.c/h      # 工具函数
│
├── 高层接口
│   ├── compress.c     # 简单压缩接口
│   ├── uncompr.c      # 简单解压接口
│   ├── gzclose.c      # gzip 文件关闭
│   ├── gzlib.c        # gzip 文件核心
│   ├── gzread.c       # gzip 文件读取
│   └── gzwrite.c      # gzip 文件写入
│
├── 头文件
│   ├── zlib.h         # 主头文件 (API 声明)
│   └── zconf.h        # 配置头文件 (类型定义)
│
└── contrib/minizip/   # ZIP 文件处理 (第三方)
    ├── ioapi.c/h      # I/O 抽象层
    ├── zip.c/h        # ZIP 压缩
    └── unzip.c/h      # ZIP 解压
```

## zlib 在 OpenHarmony 中的定位

### OH 组件信息

| 属性 | 值 |
|-----|-----|
| **OH 组件名** | @ohos/zlib |
| **OH 版本** | 3.1 |
| **所属子系统** | thirdparty |
| **系统能力** | standard (标准系统) |
| **维护者** | gongjunsong@huawei.com |

### OH 中的关键作用

zlib 是 OpenHarmony 的**基础设施组件**，被多个核心子系统依赖：

#### 1. 应用生命周期管理
- **HAP 包解压**: Bundle Manager 使用 zlib + minizip 解压 HarmonyOS Ability Package
- **资源压缩**: Global Resource Manager 使用 zlib 压缩/解压资源文件

#### 2. 多媒体处理
- **PNG 图像**: Image Framework 通过 libpng 间接使用 zlib 进行 PNG 编解码
- **媒体传输**: 部分流媒体协议支持 gzip 压缩

#### 3. 网络通信
- **HTTP 压缩**: curl 使用 zlib 支持 HTTP gzip/deflate 内容编码
- **数据传输**: 减少网络传输数据量

#### 4. 数据存储
- **KV Store**: 分布式数据库使用 zlib 压缩存储数据
- **字节码归档**: ArkCompiler 使用 zlib 压缩字节码文件

### 系统架构中的位置

```
┌───────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                   │
├───────────────────────────────────────────────────────────────┤
│  ArkUI 框架    │  Ability 框架   │  媒体服务    │  网络服务   │
├───────────────────────────────────────────────────────────────┤
│  图形引擎      │  HAP 管理器     │  图像编解码  │  HTTP 客户端│
│  (libpng)     │  (minizip)      │  (libpng)   │  (curl)     │
├───────────────────────────────────────────────────────────────┤
│                        zlib (压缩层)                           │
│  ┌────────────┬────────────┬────────────┬──────────────┐     │
│  │  deflate   │  inflate   │   gzip     │   ZIP        │     │
│  └────────────┴────────────┴────────────┴──────────────┘     │
├───────────────────────────────────────────────────────────────┤
│                    内核层 (Kernel)                             │
└───────────────────────────────────────────────────────────────┘
```

### 为何选择 zlib

1. **成熟稳定**: 30+ 年开发历史，广泛用于各行业
2. **许可证友好**: zlib 许可证允许商业使用，与 OH Apache-2.0 兼容
3. **性能优秀**: 优化的 deflate 实现，支持硬件 CRC 加速
4. **生态丰富**: 大量第三方库依赖 (libpng, curl 等)
5. **维护活跃**: 上游持续更新安全修复

### OH 适配总结

| 适配维度 | 说明 |
|---------|-----|
| **源码修改** | 无 - 源码完全保留上游 |
| **Patch** | 1 个 - 仅构建系统适配 |
| **构建系统** | BUILD.gn - 完整 GN 构建支持 |
| **特殊优化** | ARM64 CRC 硬件加速 |
| **附加功能** | 启用 minizip 支持 ZIP 文件 |
| **导出方式** | NDK 完整导出 (94 API) |

---

## 相关文档

- [上游文档](https://github.com/madler/zlib/tree/develop/doc) - 原始库文档
- [02_Patches.md](./02_Patches.md) - OH Patch 详细分析
- [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 说明

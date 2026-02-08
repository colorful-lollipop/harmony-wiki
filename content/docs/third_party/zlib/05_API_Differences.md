# API/接口差异

本文档说明 zlib 在 OpenHarmony 中的 API 差异，包括 OH 新增、行为变更和废弃功能。

## 重要说明

**zlib 在 OpenHarmony 中 API 完全兼容上游**。

OpenHarmony 没有对 zlib 进行任何源码级别的修改（无 `#ifdef OHOS` 或类似宏），因此：

- ✅ API 与上游 1.3.1 完全一致
- ✅ ABI 向后兼容
- ✅ 头文件无修改
- ✅ 行为无变更

---

## API 完整列表

### NDK 导出接口 (94 个)

```json
// zlib.ndk.json 节选
[
    // 内部符号 (9 个)
    { "name": "_dist_code" },
    { "name": "_length_code" },
    { "name": "_tr_align" },
    { "name": "_tr_flush_bits" },
    { "name": "_tr_flush_block" },
    { "name": "_tr_init" },
    { "name": "_tr_stored_block" },
    { "name": "_tr_tally" },
    
    // Adler-32 校验 (4 个)
    { "name": "adler32" },
    { "name": "adler32_combine" },
    { "name": "adler32_combine64" },
    { "name": "adler32_z" },
    
    // 简单压缩 (4 个)
    { "name": "compress" },
    { "name": "compress2" },
    { "name": "compressBound" },
    
    // CRC32 校验 (8 个)
    { "name": "crc32" },
    { "name": "crc32_combine" },
    { "name": "crc32_combine64" },
    { "name": "crc32_z" },
    { "name": "get_crc_table" },
    ...
    
    // deflate (18 个)
    { "name": "deflate" },
    { "name": "deflateBound" },
    { "name": "deflateCopy" },
    { "name": "deflateEnd" },
    ...
    
    // inflate (22 个)
    { "name": "inflate" },
    { "name": "inflateBack" },
    { "name": "inflateBackEnd" },
    ...
    
    // gzip 文件操作 (28 个)
    { "name": "gzbuffer" },
    { "name": "gzclearerr" },
    { "name": "gzclose" },
    { "name": "gzclose_r" },
    { "name": "gzclose_w" },
    { "name": "gzdirect" },
    { "name": "gzdopen" },
    { "name": "gzeof" },
    { "name": "gzerror" },
    { "name": "gzflush" },
    ...
    
    // 工具函数
    { "name": "zError" },
    { "name": "zlibCompileFlags" },
    { "name": "zlibVersion" }
]
```

完整列表见: `//third_party/zlib/zlib.ndk.json`

---

## API 分类说明

### 1. 简单压缩/解压接口

**适用于**: 简单场景，小数据量

```c
// 压缩
deflateInit(strm, level);     // 初始化
deflate(strm, flush);          // 压缩数据
deflateEnd(strm);              // 清理

// 一次性压缩 (更简单)
compress(dest, destLen, source, sourceLen);
compress2(dest, destLen, source, sourceLen, level);

// 解压
inflateInit(strm);
inflate(strm, flush);
inflateEnd(strm);

// 一次性解压
uncompress(dest, destLen, source, sourceLen);
uncompress2(dest, destLen, source, sourceLen);
```

### 2. gzip 文件接口

**适用于**: 处理 .gz 文件

```c
// 打开/关闭
gzFile gzopen(const char *path, const char *mode);
gzFile gzdopen(int fd, const char *mode);
int gzclose(gzFile file);

// 读写
int gzread(gzFile file, voidp buf, unsigned len);
int gzwrite(gzFile file, voidpc buf, unsigned len);

// 定位
z_off_t gzseek(gzFile file, z_off_t offset, int whence);
z_off_t gztell(gzFile file);

// 刷新
int gzflush(gzFile file, int flush);
```

### 3. 校验接口

```c
// CRC32
uLong crc32(uLong crc, const Bytef *buf, uInt len);
uLong crc32_z(uLong crc, const Bytef *buf, z_size_t len);

// Adler-32
uLong adler32(uLong adler, const Bytef *buf, uInt len);
uLong adler32_z(uLong adler, const Bytef *buf, z_size_t len);
```

### 4. MiniZip 接口 (ZIP 文件)

**头文件**: `#include "contrib/minizip/zip.h"`

```c
// 创建 ZIP 文件
zipFile zipOpen(const char *pathname, int append);
int zipOpenNewFileInZip(zipFile file, const char *filename,
                        const zip_fileinfo *zipfi,
                        const void *extrafield_local, uInt size_extrafield_local,
                        const void *extrafield_global, uInt size_extrafield_global,
                        const char *comment, int method, int level);
int zipWriteInFileInZip(zipFile file, const void *buf, unsigned len);
int zipCloseFileInZip(zipFile file);
int zipClose(zipFile file, const char *global_comment);

// 解压 ZIP 文件
unzFile unzOpen(const char *path);
int unzGoToFirstFile(unzFile file);
int unzGoToNextFile(unzFile file);
int unzGetCurrentFileInfo(unzFile file, unz_file_info *pfile_info,
                          char *filename, uInt filename_size, ...);
int unzOpenCurrentFile(unzFile file);
int unzReadCurrentFile(unzFile file, voidp buf, unsigned len);
int unzCloseCurrentFile(unzFile file);
int unzClose(unzFile file);
```

---

## 与上游的 API 一致性

### 头文件对比

| 头文件 | OH 修改 | 说明 |
|-------|--------|-----|
| `zlib.h` | 无 | 与上游 1.3.1 完全一致 |
| `zconf.h` | 无 | 配置头文件，自动配置 |

### 宏定义

**版本宏**:
```c
#define ZLIB_VERSION "1.3.1"
#define ZLIB_VERNUM 0x1310
```

**功能宏** (用于功能检测):
```c
// 功能支持检测
#ifdef ZLIB_VERNUM
    // zlib 可用
#endif

// 特定功能检测
#if ZLIB_VERNUM >= 0x1230
    // 使用 1.2.3+ 的新特性
#endif
```

---

## OH 特定功能

### 1. ARM64 CRC32 硬件加速

**功能**: 在 ARM64 平台上使用硬件 CRC32 指令

**使用方式**: 自动启用 (通过 libz_crc 或标准 libz)

**检测方式**:
```c
// 检查编译标志
// 在 ARM64 设备上，BUILD.gn 自动添加 -march=armv8-a+crc

// 运行时检测 (如果需要)
#include <sys/auxv.h>
#include <asm/hwcap.h>

#if defined(__aarch64__)
    unsigned long hwcap = getauxval(AT_HWCAP);
    if (hwcap & HWCAP_CRC32) {
        // 支持硬件 CRC32
    }
#endif
```

### 2. minizip 启用

**上游**: 可选组件 (默认关闭)

**OH**: 强制启用 (包含在 BUILD.gn 中)

**影响**:
- 头文件路径: `contrib/minizip/zip.h`
- 无需额外链接，已包含在 libz 中

---

## 使用示例

### 示例 1: 简单压缩

```c
#include <zlib.h>
#include <stdio.h>

int compress_data(const char *input, size_t input_len,
                  char *output, size_t *output_len) {
    int ret = compress2((Bytef *)output, (uLongf *)output_len,
                        (const Bytef *)input, (uLong)input_len,
                        Z_BEST_COMPRESSION);
    return ret == Z_OK ? 0 : -1;
}
```

### 示例 2: 流式压缩

```c
#include <zlib.h>

int stream_compress(z_stream *strm, const char *input, size_t len,
                    char *output, size_t *output_len) {
    strm->avail_in = len;
    strm->next_in = (Bytef *)input;
    strm->avail_out = *output_len;
    strm->next_out = (Bytef *)output;
    
    int ret = deflate(strm, Z_FINISH);
    *output_len = *output_len - strm->avail_out;
    
    return ret == Z_STREAM_END ? 0 : -1;
}
```

### 示例 3: ZIP 文件解压

```c
#include "contrib/minizip/unzip.h"
#include <stdio.h>

int extract_zip(const char *zip_path, const char *dest_dir) {
    unzFile zf = unzOpen(zip_path);
    if (!zf) return -1;
    
    if (unzGoToFirstFile(zf) != UNZ_OK) {
        unzClose(zf);
        return -1;
    }
    
    do {
        char filename[256];
        unz_file_info file_info;
        
        if (unzGetCurrentFileInfo(zf, &file_info, filename, sizeof(filename),
                                   NULL, 0, NULL, 0) != UNZ_OK) {
            continue;
        }
        
        if (unzOpenCurrentFile(zf) != UNZ_OK) {
            continue;
        }
        
        // 读取并解压文件内容
        char buffer[8192];
        int read;
        while ((read = unzReadCurrentFile(zf, buffer, sizeof(buffer))) > 0) {
            // 写入解压后的数据
        }
        
        unzCloseCurrentFile(zf);
    } while (unzGoToNextFile(zf) == UNZ_OK);
    
    unzClose(zf);
    return 0;
}
```

---

## 性能优化建议

### 1. 选择合适的压缩级别

```c
#define Z_NO_COMPRESSION         0   // 不压缩
#define Z_BEST_SPEED             1   // 最快
#define Z_BEST_COMPRESSION       9   // 最佳压缩率
#define Z_DEFAULT_COMPRESSION   -1   // 默认 (6)
```

### 2. 内存管理

```c
// 自定义内存分配器
deflateInit2(strm, level, Z_DEFLATED, windowBits, memLevel, strategy);

// memLevel: 1-9, 默认 8
// 减小 memLevel 可减少内存使用，但降低压缩率
```

### 3. ARM64 CRC32 优化

```c
// 使用 libz_crc 库 (自动启用硬件加速)
// 在 BUILD.gn 中依赖 "//third_party/zlib:libz_crc"
```

---

## 参考资料

- [上游 zlib.h 文档](https://github.com/madler/zlib/blob/develop/zlib.h)
- [zlib Manual](https://zlib.net/manual.html)
- [RFC 1950 - ZLIB Compressed Data Format](https://tools.ietf.org/html/rfc1950)
- [RFC 1951 - DEFLATE Compressed Data Format](https://tools.ietf.org/html/rfc1951)
- [RFC 1952 - GZIP File Format](https://tools.ietf.org/html/rfc1952)

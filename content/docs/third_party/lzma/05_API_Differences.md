# 05 - API/接口差异

## 概览

**状态**: OH 未对 LZMA SDK API 进行修改

| 项目 | 状态 |
|------|------|
| OH 新增 API | 无 |
| OH 修改 API 行为 | 无 |
| OH 废弃 API | 无 |
| 与上游 API 兼容性 | 100% |

**结论**: OpenHarmony 使用的 LZMA SDK 与上游 API 完全一致，无差异。

---

## API 说明

由于 OH 未修改 API，本节主要介绍 LZMA SDK 提供的标准 API，供开发者参考。

### C 语言 API

#### 核心解压 API

**头文件**: `LzmaDec.h`

##### 1. 单步解压 (RAM → RAM)

```c
SRes LzmaDecode(Byte *dest, SizeT *destLen,
                const Byte *src, SizeT *srcLen,
                const Byte *propData, unsigned propSize,
                ELzmaFinishMode finishMode,
                ELzmaStatus *status, ISzAlloc *alloc);
```

**参数**:
| 参数 | 说明 |
|------|------|
| `dest` | 输出缓冲区 |
| `destLen` | 输出缓冲区大小（输入）/ 实际输出大小（输出） |
| `src` | 输入（压缩数据） |
| `srcLen` | 输入大小（输入）/ 实际读取大小（输出） |
| `propData` | LZMA 属性（5 字节） |
| `propSize` | 属性大小（通常为 5） |
| `finishMode` | 结束模式：`LZMA_FINISH_ANY` 或 `LZMA_FINISH_END` |
| `status` | 输出状态 |
| `alloc` | 内存分配器 |

**返回值**:
- `SZ_OK` - 成功
- `SZ_ERROR_DATA` - 数据错误
- `SZ_ERROR_MEM` - 内存分配失败
- `SZ_ERROR_UNSUPPORTED` - 不支持的属性
- `SZ_ERROR_INPUT_EOF` - 需要更多输入

**使用示例**:
```c
#include "LzmaDec.h"

void *SzAlloc(void *p, size_t size) { return malloc(size); }
void SzFree(void *p, void *address) { free(address); }

ISzAlloc g_Alloc = { SzAlloc, SzFree };

int UncompressLZMA(Byte *out, size_t *outLen, 
                   const Byte *in, size_t inLen,
                   const Byte *props) {
    ELzmaStatus status;
    return LzmaDecode(out, outLen, in, &inLen, 
                      props, LZMA_PROPS_SIZE,
                      LZMA_FINISH_ANY, &status, &g_Alloc);
}
```

##### 2. 状态机解压 (流式)

适合大文件或流式处理：

```c
// 初始化和分配
void LzmaDec_Construct(CLzmaDec *p);
SRes LzmaDec_Allocate(CLzmaDec *p, const Byte *propData, unsigned propSize, ISzAlloc *alloc);

// 解码循环
void LzmaDec_Init(CLzmaDec *p);
SRes LzmaDec_DecodeToBuf(CLzmaDec *p, Byte *dest, SizeT *destLen,
                         const Byte *src, SizeT *srcLen,
                         ELzmaFinishMode finishMode);

// 清理
void LzmaDec_Free(CLzmaDec *p, ISzAlloc *alloc);
```

**使用示例**:
```c
CLzmaDec state;
LzmaDec_Construct(&state);

// 从 LZMA 头读取属性 (5 字节)
Byte header[LZMA_PROPS_SIZE];
ReadFile(inFile, header, sizeof(header));

// 分配状态
SRes res = LzmaDec_Allocate(&state, header, LZMA_PROPS_SIZE, &g_Alloc);
if (res != SZ_OK) return res;

// 初始化
LzmaDec_Init(&state);

// 循环解码
Byte inBuf[4096], outBuf[4096];
for (;;) {
    SizeT inLen = ReadInput(inBuf);
    SizeT outLen = sizeof(outBuf);
    ELzmaFinishMode mode = (inLen == 0) ? LZMA_FINISH_END : LZMA_FINISH_ANY;
    
    res = LzmaDec_DecodeToBuf(&state, outBuf, &outLen, inBuf, &inLen, mode);
    WriteOutput(outBuf, outLen);
    
    if (res != SZ_OK || inLen == 0) break;
}

LzmaDec_Free(&state, &g_Alloc);
```

#### 7z 归档 API

**头文件**: `7z.h`

```c
// 7z 归档解析
SRes SzArEx_Open(CSzArEx *p, ILookInStream *stream, 
                 ISzAlloc *allocMain, ISzAlloc *allocTemp);

// 获取文件信息
UInt32 SzArEx_GetFileCount(const CSzArEx *p);
SRes SzArEx_GetFileNameUtf16(const CSzArEx *p, UInt32 fileIndex, 
                              UInt16 *dest);

// 解压文件
SRes SzArEx_Extract(const CSzArEx *p, ILookInStream *stream, UInt32 fileIndex,
                    UInt32 *blockIndex, Byte **outBuffer, size_t *outBufferSize,
                    size_t *offset, size_t *outSizeProcessed,
                    ISzAlloc *allocMain, ISzAlloc *allocTmp);

// 清理
void SzArEx_Free(CSzArEx *p, ISzAlloc *alloc);
```

#### XZ 格式 API

**头文件**: `Xz.h`

```c
// XZ 解码器状态
typedef struct {
    // ... 内部状态
} CXzDec;

// 初始化和解码
void XzDec_Construct(CXzDec *p);
SRes XzDec_Init(CXzDec *p, ISzAlloc *alloc);
SRes XzDec_DecodeToBuf(CXzDec *p, Byte *dest, SizeT *destLen,
                       const Byte *src, SizeT *srcLen);
void XzDec_Free(CXzDec *p, ISzAlloc *alloc);
```

#### 内存分配器接口

```c
typedef struct _ISzAlloc
{
    void *(*Alloc)(void *p, size_t size);
    void (*Free)(void *p, void *address);
} ISzAlloc;
```

**标准实现**:
```c
void *SzAlloc(void *p, size_t size) { 
    (void)p; 
    return malloc(size); 
}

void SzFree(void *p, void *address) { 
    (void)p; 
    free(address); 
}

ISzAlloc g_Alloc = { SzAlloc, SzFree };
```

---

## OH 使用模式

### faultloggerd 中的典型用法

```c
// 解压 ELF 中的压缩 unwind 信息
SRes UncompressEHFrame(Byte *dest, SizeT destLen,
                       const Byte *src, SizeT srcLen,
                       const Byte *props) {
    ELzmaStatus status;
    SizeT outLen = destLen;
    SizeT inLen = srcLen;
    
    // OH 使用标准 LZMA API，无任何修改
    SRes res = LzmaDecode(dest, &outLen,
                          src, &inLen,
                          props, LZMA_PROPS_SIZE,
                          LZMA_FINISH_ANY,
                          &status, &g_Alloc);
    
    if (res != SZ_OK) {
        DFXLOG_ERROR("LZMA decompress failed: %d", res);
        return res;
    }
    
    // 验证解压完整性
    if (status != LZMA_STATUS_FINISHED_WITH_MARK &&
        status != LZMA_STATUS_NOT_FINISHED) {
        DFXLOG_WARN("LZMA unusual status: %d", status);
    }
    
    return SZ_OK;
}
```

---

## API 版本兼容性

### LZMA SDK 版本历史

| 版本 | 日期 | 主要变化 |
|------|------|----------|
| 25.01 | 2024-11 | 当前 OH 版本 |
| 24.09 | 2024-11 | - |
| 24.05 | 2024-05 | - |
| 23.01 | 2023-07 | - |
| 22.01 | 2022-07 | 重大接口变更 |
| 19.00 | 2019-02 | - |

### 兼容性说明

**C API**: 自 SDK 4.58 (2007年) 以来保持稳定

**关键变更历史**:
- SDK 4.58: ANSI-C 接口首次稳定
- SDK 22.01: 内部优化，API 保持不变

**结论**: OH 当前使用的 API 具有良好的向前兼容性。

---

## 头文件清单

| 头文件 | 功能 | OH 使用 |
|--------|------|---------|
| `7z.h` | 7z 归档格式 | 是 |
| `7zTypes.h` | 基础类型定义 | 是 |
| `LzmaDec.h` | LZMA 解压 | 是 |
| `LzmaEnc.h` | LZMA 压缩 | 否 |
| `Lzma2Dec.h` | LZMA2 解压 | 是 |
| `Lzma2Enc.h` | LZMA2 压缩 | 否 |
| `Xz.h` | XZ 格式 | 是 |
| `Alloc.h` | 内存分配 | 是 |
| `Bcj2.h` | BCJ2 过滤器 | 是 |
| `Bra.h` | 分支重定位 | 是 |
| `CpuArch.h` | CPU 架构检测 | 是 |
| `Delta.h` | Delta 过滤器 | 是 |
| `Ppmd7.h` | PPMd 压缩 | 是 |
| `Sha256.h` | SHA-256 哈希 | 是 |

---

## 注意事项

### 线程安全

- `LzmaDecode()` (单步解压): **线程安全**
- `CLzmaDec` 状态机: 每个线程需要独立实例
- `ISzAlloc`: 需要线程安全的分配器

### 内存要求

```
LZMA 解压内存 = state_size + dictionary
state_size ≈ 16 KB (默认参数)
dictionary = 压缩时指定的字典大小
```

### 错误处理

**必须检查**:
1. `LzmaDecode()` 返回值
2. `destLen` 和 `srcLen` 输出值
3. `status` 状态码

---

## 参考文档

- [LZMA 规范](../DOC/lzma-specification.txt) (位于 `third_party/lzma/DOC/`)
- [7z 格式说明](../DOC/7zFormat.txt)
- [lzma-sdk.txt](../DOC/lzma-sdk.txt)

---

*最后更新: 2025-02-08*

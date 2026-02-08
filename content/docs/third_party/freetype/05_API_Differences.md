# API/接口差异

本文档记录 FreeType 在 OpenHarmony 集成中与上游版本的 API 差异。

---

## 1. 新增导出函数

### 1.1 Stream 操作函数

以下内部函数通过 `enable-funcs` patch 被导出为公共 API：

| 函数 | 原状态 | 现状态 | 用途 |
|------|--------|--------|------|
| `FT_Stream_New()` | FT_BASE | FT_EXPORT | 创建自定义流 |
| `FT_Stream_Free()` | FT_BASE | FT_EXPORT | 释放流 |
| `FT_Stream_Seek()` | FT_BASE | FT_EXPORT | 定位流位置 |
| `FT_Stream_Skip()` | FT_BASE | FT_EXPORT | 跳过字节 |
| `FT_Stream_Pos()` | FT_BASE | FT_EXPORT | 获取当前位置 |
| `FT_Stream_Read()` | FT_BASE | FT_EXPORT | 读取数据块 |
| `FT_Stream_ReadUShort()` | FT_BASE | FT_EXPORT | 读取 16 位 BE |
| `FT_Stream_ReadULong()` | FT_BASE | FT_EXPORT | 读取 32 位 BE |
| `FT_Stream_ReadUShortLE()` | FT_BASE | FT_EXPORT | 读取 16 位 LE |
| `FT_Stream_ReadFields()` | FT_BASE | FT_EXPORT | 读取结构化数据 |

### 1.2 Glyph 加载器函数

| 函数 | 原状态 | 现状态 | 用途 |
|------|--------|--------|------|
| `FT_GlyphLoader_Reset()` | FT_BASE | FT_EXPORT | 重置加载器 |
| `ft_glyphslot_free_bitmap()` | FT_BASE | FT_EXPORT | 释放位图 |

---

## 2. 兼容函数（桩实现）

### 2.1 内部轮廓函数

以下函数保留声明但返回错误：

```c
// 头文件声明（ftoutln.h）
FT_EXPORT( FT_Error )
FT_Outline_New_Internal( FT_Memory    memory,
                         FT_UInt      numPoints,
                         FT_Int       numContours,
                         FT_Outline  *anoutline );

FT_EXPORT( FT_Error )
FT_Outline_Done_Internal( FT_Memory    memory,
                          FT_Outline*  outline );

// 源文件实现（ftoutln.c）
FT_EXPORT_DEF( FT_Error )
FT_Outline_New_Internal( FT_Memory    memory,
                         FT_UInt      numPoints,
                         FT_Int       numContours,
                         FT_Outline  *anoutline )
{
  return FT_THROW( Unimplemented_Feature );
}

FT_EXPORT_DEF( FT_Error )
FT_Outline_Done_Internal( FT_Memory    memory,
                          FT_Outline*  outline )
{
  return FT_THROW( Unimplemented_Feature );
}
```

### 2.2 Debug Hook 函数

```c
// ftmodapi.h
typedef void
(*FT_DebugHook_Func)( void*  arg );
// 注意：返回类型为 void，非 FT_Error
```

---

## 3. 配置差异

### 3.1 启用的编译选项

| 配置 | OH 值 | 上游默认值 | 说明 |
|------|-------|-----------|------|
| `FT_CONFIG_OPTION_SUBPIXEL_RENDERING` | 启用 | 禁用 | LCD 优化渲染 |
| `FT_CONFIG_OPTION_USE_PNG` | 启用 | 可选 | PNG 位图支持 |
| `FT_CONFIG_OPTION_SYSTEM_ZLIB` | 启用 | 禁用 | 使用系统 zlib |

### 3.2 多架构配置

```c
// ftconfig.h - OH 特有多架构支持
#if __WORDSIZE == 32
# include "ftconfig-32.h"
#elif __WORDSIZE == 64
# include "ftconfig-64.h"
#endif
```

---

## 4. 使用建议

### 4.1 推荐使用新增导出函数

```c
// 推荐：使用导出的流函数
FT_Stream stream;
FT_Stream_New(library, &args, &stream);
FT_Stream_Seek(stream, offset);
FT_Stream_Read(stream, buffer, count);
FT_Stream_Free(stream, 0);

// 不推荐：尝试使用内部函数
// FT_BASE 函数可能无法链接
```

### 4.2 避免使用兼容桩函数

```c
// 避免：这些函数返回 Unimplemented_Feature
FT_Outline_New_Internal(memory, points, contours, outline);
// 返回 FT_Err_Unimplemented_Feature

// 使用：公共 API
FT_Outline_New(library, points, contours, outline);
// 正常执行
```

---

## 5. API 变更历史

| 版本 | 变更类型 | 变更描述 | 影响 |
|------|----------|----------|------|
| 2.10.0 | 废弃 | 移除 `FT_Outline_*_Internal` | 使用公共 API |
| 2.10.1 | 修复 | `FT_DebugHook_Func` 返回类型 | 兼容性 |
| 2.12.1 | 扩展 | 导出 11 个内部函数 | OH 特有 |

---

*文档版本: 1.0*
*最后更新: 2025-02-08*

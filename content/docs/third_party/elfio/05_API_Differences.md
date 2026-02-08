# API/接口差异

## 概述

**重要说明**：ELFIO 在 OpenHarmony 中**没有对上游 API 进行任何修改**。

所有上游 API 均保持原样可用，OH 仅添加了构建适配层（C 包装器）和构建系统配置。

---

## 上游 API 保持不变

### C++ 接口

所有上游 C++ 接口均未修改：

| 头文件 | API 状态 | 说明 |
|--------|----------|------|
| `elfio.hpp` | ✅ 未修改 | 主入口类 |
| `elfio_header.hpp` | ✅ 未修改 | ELF 头操作 |
| `elfio_section.hpp` | ✅ 未修改 | Section 操作 |
| `elfio_segment.hpp` | ✅ 未修改 | Segment 操作 |
| `elfio_symbols.hpp` | ✅ 未修改 | 符号表访问 |
| `elfio_relocation.hpp` | ✅ 未修改 | 重定位处理 |
| `elfio_dynamic.hpp` | ✅ 未修改 | 动态链接信息 |
| `elfio_note.hpp` | ✅ 未修改 | Note 节处理 |
| `elfio_modinfo.hpp` | ✅ 未修改 | Modinfo 操作 |
| `elfio_strings.hpp` | ✅ 未修改 | 字符串节操作 |
| `elfio_array.hpp` | ✅ 未修改 | 数组节操作 |
| `elfio_dump.hpp` | ✅ 未修改 | ELF 信息导出 |
| `elf_types.hpp` | ✅ 未修改 | ELF 类型定义 |
| `elfio_utils.hpp` | ✅ 未修改 | 工具函数 |
| `elfio_version.hpp` | ✅ 未修改 | 版本信息 |

---

## OH 新增接口

### C 包装器接口

OH 新增了完整的 C 语言接口，封装了所有 C++ 功能：

| 文件 | 新增内容 |
|------|----------|
| `c_wrapper/elfio_c_wrapper.h` | C 函数声明 |
| `c_wrapper/elfio_c_wrapper.cpp` | C 函数实现 |
| `c_wrapper/elf_types_c_wrapper.hpp` | C 类型映射 |

### C 接口与 C++ 接口映射

#### 1. 核心类操作

| C++ 接口 | C 接口 | 说明 |
|----------|--------|------|
| `elfio::load()` | `elfio_load()` | 加载 ELF 文件 |
| `elfio::save()` | `elfio_save()` | 保存 ELF 文件 |
| `elfio::create()` | `elfio_create()` | 创建 ELF 文件 |
| `elfio::validate()` | `elfio_validate()` | 验证 ELF 文件 |

#### 2. Section 操作

| C++ 接口 | C 接口 |
|----------|--------|
| `section->get_name()` | `elfio_section_get_name()` |
| `section->set_name()` | `elfio_section_set_name()` |
| `section->get_data()` | `elfio_section_get_data()` |
| `section->set_data()` | `elfio_section_set_data()` |
| `section->append_data()` | `elfio_section_append_data()` |

#### 3. Segment 操作

| C++ 接口 | C 接口 |
|----------|--------|
| `segment->get_type()` | `elfio_segment_get_type()` |
| `segment->get_virtual_address()` | `elfio_segment_get_virtual_address()` |
| `segment->add_section_index()` | `elfio_segment_add_section_index()` |

#### 4. 符号操作

| C++ 接口 | C 接口 |
|----------|--------|
| `symbol_section_accessor::get_symbols_num()` | `elfio_symbol_get_symbols_num()` |
| `symbol_section_accessor::get_symbol()` | `elfio_symbol_get_symbol()` |
| `symbol_section_accessor::add_symbol()` | `elfio_symbol_add_symbol()` |

---

## C 包装器设计

### 头文件保护

```c
#ifdef __cplusplus
typedef ELFIO::elfio*                       pelfio_t;
typedef ELFIO::section*                     psection_t;
// ... 其他 C++ 类型映射
#else
typedef void* pelfio_t;
typedef void* psection_t;
// ... C 语言 opaque 类型
#endif
```

### 宏批量生成

C 包装器使用宏批量生成访问函数：

```c
#define ELFIO_C_HEADER_ACCESS_GET_SET( TYPE, FNAME ) \
    TYPE elfio_get_##FNAME( pelfio_t pelfio );       \
    void elfio_set_##FNAME( pelfio_t pelfio, TYPE val );

#define ELFIO_C_GET_SET_ACCESS_IMPL( CLASS, TYPE, NAME ) \
    TYPE elfio_##CLASS##_get_##NAME( p##CLASS##_t p##CLASS ) \
    {                                                   \
        return p##CLASS->get_##NAME();                   \
    }                                                   \
    void elfio_##CLASS##_set_##NAME( p##CLASS##_t p##CLASS, TYPE value ) \
    {                                                                       \
        p##CLASS->set_##NAME( value );                                       \
    }
```

---

## 使用建议

### 1. C++ 项目

直接使用上游 C++ API，无需通过 C 包装器：

```cpp
#include <elfio/elfio.hpp>

ELFIO::elfio reader;
reader.load("example.elf");
// 使用 C++ 接口...
```

### 2. C 项目

必须使用 C 包装器：

```c
#include <elfio_c_wrapper.h>

pelfio_t elf = elfio_new();
elfio_load(elf, "example.elf");
// 使用 C 接口...
```

### 3. 混合项目

可以混合使用，但建议统一风格：

```cpp
// C++ 中调用 C 接口（可正常工作）
#include <elfio_c_wrapper.h>

pelfio_t elf = elfio_new();
elfio_load(elf, "example.elf");
```

---

## 版本兼容性

### API 兼容性矩阵

| OH 版本 | ELFIO 版本 | C++ API | C API | 兼容性 |
|---------|------------|---------|-------|--------|
| 4.0+ | 3.12 | ✅ 兼容 | ✅ 兼容 | 完整 |
| 3.x | 3.12 | ✅ 兼容 | ✅ 兼容 | 完整 |

### 升级注意事项

- **上游 API 变更**：需要同步更新 C 包装器
- **新增功能**：需要添加对应的 C 封装
- **废弃 API**：需要移除对应的 C 封装

---

## 相关文档

- [README](./README.md)
- [构建适配](./03_Build_Integration.md)
- [使用场景](./04_Usage_in_OH.md)

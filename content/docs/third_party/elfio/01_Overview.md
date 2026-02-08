# 原始库简介

## ELFIO 概述

### 项目背景

ELFIO 是一个轻量级的 **纯头文件（header-only）C++ 库**，专门用于读取和生成 **ELF（Executable and Linkable Format）** 二进制格式文件。

ELF 是 Unix、Linux 以及许多其他操作系统中标准的可执行文件格式，用于表示：
- 可执行程序
- 共享库（.so）
- 目标文件（.o）
- 核心转储文件

### 发展历程

| 时间 | 版本 | 主要变更 |
|------|------|----------|
| 2005 | 1.0 | 初始版本发布 |
| 2010 | 2.0 | 重构代码架构 |
| 2015 | 3.0 | 添加 C 接口支持 |
| 2022 | 3.12 | 最新稳定版本 |

### 设计理念

1. **简洁性**：最小依赖，仅使用标准 C++
2. **可移植性**：支持多种架构和编译器
3. **易用性**：直观的 API 设计
4. **完整性**：覆盖 ELF 格式的所有特性

---

## 核心架构

### 模块结构

```
elfio/
├── 核心模块
│   ├── elfio.hpp              # 主入口类
│   ├── elfio_header.hpp       # ELF 头解析
│   ├── elfio_section.hpp      # Section 管理
│   └── elfio_segment.hpp      # Segment 管理
├── 
├── 符号与重定位
│   ├── elfio_symbols.hpp      # 符号表访问
│   └── elfio_relocation.hpp   # 重定位处理
├── 
├── 特殊节区
│   ├── elfio_dynamic.hpp      # 动态链接信息
│   ├── elfio_note.hpp         # Note 节处理
│   ├── elfio_modinfo.hpp      # 模块信息
│   ├── elfio_strings.hpp     # 字符串节
│   └── elfio_array.hpp        # 数组节
├── 
├── 工具与支持
│   ├── elfio_dump.hpp         # 信息导出
│   ├── elfio_utils.hpp        # 工具函数
│   ├── elfio_version.hpp      # 版本信息
│   └── elf_types.hpp          # ELF 类型定义
└── 
└── C 包装器
    ├── elfio_c_wrapper.h       # C 接口头文件
    ├── elfio_c_wrapper.cpp     # C 接口实现
    └── elf_types_c_wrapper.hpp # C 类型映射
```

### 主要类设计

| 类名 | 职责 |
|------|------|
| `elfio` | 主入口类，管理 ELF 文件的加载、保存、创建 |
| `elf_header` | 封装 ELF 头部信息 |
| `section` | 表示 ELF 文件中的一个节区 |
| `segment` | 表示 ELF 文件中的一个程序段 |
| `symbol_section_accessor` | 提供符号表的访问接口 |
| `relocation_section_accessor` | 提供重定位表的访问接口 |
| `note_section_accessor` | 提供 Note 节区的访问接口 |
| `dynamic_section_accessor` | 提供动态链接信息的访问接口 |

---

## 功能特性

### 1. 文件操作

```cpp
// 加载现有 ELF 文件
ELFIO::elfio reader;
reader.load("example.elf");

// 创建新 ELF 文件
ELFIO::elfio writer;
writer.create(ELFCLASS64, ELFDATA2LSB);
writer.save("new_file.elf");

// 验证 ELF 格式
std::string error;
if (!reader.validate(error)) {
    std::cerr << "Invalid ELF: " << error << std::endl;
}
```

### 2. 节区遍历

```cpp
// 遍历所有节区
for (const auto& section : reader.sections) {
    std::cout << "Section: " << section->get_name() 
              << ", Size: " << section->get_size() 
              << ", Type: " << section->get_type() << std::endl;
}

// 按名称查找节区
auto* text_section = reader.sections[".text"];
if (text_section) {
    auto* data = text_section->get_data();
    // 处理节区数据...
}
```

### 3. 符号表访问

```cpp
// 获取符号表
const auto& sym_section = reader.sections[".symtab"];
ELFIO::symbol_section_accessor symbols(reader, sym_section);

// 遍历符号
for (Elf_Xword i = 0; i < symbols.get_symbols_num(); ++i) {
    std::string name;
    Elf64_Addr value;
    Elf_Xword size;
    unsigned char bind, type, other;
    Elf_Half section_index;
    
    symbols.get_symbol(i, name, value, size, bind, 
                      type, section_index, other);
    
    std::cout << "Symbol: " << name 
              << ", Value: 0x" << std::hex << value << std::endl;
}
```

### 4. 重定位处理

```cpp
// 获取重定位节区
const auto& reloc_section = reader.sections[".rela.text"];
ELFIO::relocation_section_accessor reloc(reader, reloc_section);

// 遍历重定位条目
for (Elf_Xword i = 0; i < reloc.get_entries_num(); ++i) {
    Elf64_Addr offset;
    Elf_Word symbol;
    Elf_Word type;
    Elf_Sxword addend;
    
    reloc.get_entry(i, offset, symbol, type, addend);
    // 处理重定位...
}
```

### 5. 程序段操作

```cpp
// 遍历程序段
for (const auto& segment : reader.segments) {
    std::cout << "Segment type: " << segment->get_type() 
              << ", Flags: " << segment->get_flags() << std::endl;
}

// 添加新段
auto* new_segment = writer.segments.add();
new_segment->set_type(PT_LOAD);
new_segment->set_flags(PF_R | PF_W | PF_X);
// 配置其他段属性...
```

---

## 技术规格

### 支持的格式

| 格式 | 支持状态 | 说明 |
|------|----------|------|
| ELF32 | ✅ 支持 | 32 位 ELF 格式 |
| ELF64 | ✅ 支持 | 64 位 ELF 格式 |
| Big-endian | ✅ 支持 | 大端字节序 |
| Little-endian | ✅ 支持 | 小端字节序 |

### 依赖要求

| 依赖 | 要求 | 说明 |
|------|------|------|
| C++ 标准 | C++14 或更高 | 推荐 C++17 |
| 编译器 | GCC/Clang/MSVC | 主流编译器均支持 |
| 标准库 | 仅 STL | 无其他外部依赖 |

### 性能特性

| 指标 | 描述 |
|------|------|
| 内存占用 | 极低（仅头文件） |
| 解析速度 | 高效（增量解析） |
| 二进制大小 | 无影响（header-only） |

---

## 使用场景

### 典型应用

1. **编译器工具**：处理目标文件和可执行文件
2. **链接器**：解析和操作目标文件
3. **调试器**：读取调试信息
4. **反分析工具**：解析可执行文件结构
5. **包管理器**：处理共享库依赖
6. **安全工具**：签名和验证

### 在 OpenHarmony 中的应用

| 场景 | 使用模块 |
|------|----------|
| ELF 文件签名 | hapsigner |
| 运行时编译 | arkcompiler (irtoc) |
| eBPF 程序处理 | netmanager_base (bpf) |
| BPF 功能 | libbpf |
| 代码签名 | code_signature |

---

## 许可证与版权

### 许可证

**MIT License** - 宽松的开源许可证

```
Copyright (C) 2001-present by Serge Lamikhov-Center

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

### 使用优势

| 优势 | 说明 |
|------|------|
| 商业友好 | 可用于商业产品 |
| 修改自由 | 可自由修改代码 |
| 分发自由 | 可自由分发 |
| 专利授权 | 包含专利授权条款 |

---

## 学习资源

### 官方文档

| 资源 | 链接 |
|------|------|
| 项目主页 | http://elfio.sourceforge.net/ |
| PDF 教程 | http://elfio.sourceforge.net/elfio.pdf |
| GitHub | https://github.com/serge1/ELFIO |

### 代码示例

| 示例目录 | 说明 |
|----------|------|
| `examples/` | 官方示例代码 |
| `tests/` | 测试用例 |
| `doc/` | 文档资料 |

---

## 相关文档

- [README](./README.md) - 库概览
- [构建适配](./03_Build_Integration.md) - OH 构建说明
- [使用场景](./04_Usage_in_OH.md) - OH 中的使用方式
- [Patch 分析](./02_Patches.md) - OH 特有修改

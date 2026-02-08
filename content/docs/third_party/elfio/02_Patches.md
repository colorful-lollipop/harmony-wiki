# ELFIO 库概览

## 原始库简介

### 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | ELFIO |
| **版本** | Release_3.12 |
| **许可证** | MIT License |
| **上游地址** | http://elfio.sourceforge.net/ |
| **GitHub** | https://github.com/serge1/ELFIO |

### 功能描述

ELFIO 是一个**纯头文件（header-only）**的 C++ 库，用于读取和生成 **ELF（Executable and Linkable Format）** 二进制格式文件。

ELF 是 Unix 和 Linux 系统中常用的可执行文件格式，用于：
- 可执行文件（.so, .elf）
- 共享库
- 目标文件（.o）
- 核心转储文件

### 主要特性

1. **头文件库**：无需编译，直接包含头文件即可使用
2. **跨平台**：支持多种架构（x86, ARM, MIPS 等）和编译器
3. **ISO C++**：遵循 C++ 标准，不依赖特定编译器扩展
4. **完整 ELF 支持**：支持 ELF32 和 ELF64 格式
5. **C 接口**：提供 C 语言封装层

### 核心功能

| 模块 | 功能 |
|------|------|
| `elfio.hpp` | 主入口类，创建、加载、保存 ELF 文件 |
| `elfio_header.hpp` | ELF 头解析 |
| `elfio_section.hpp` | Section 操作 |
| `elfio_segment.hpp` | Segment 操作 |
| `elfio_symbols.hpp` | 符号表访问 |
| `elfio_relocation.hpp` | 重定位处理 |
| `elfio_dynamic.hpp` | 动态链接信息 |
| `elfio_note.hpp` | Note 节处理 |
| `elfio_dump.hpp` | ELF 文件信息导出 |

---

## 在 OpenHarmony 中的定位

### 作用

ELFIO 在 OpenHarmony 中作为 **ELF 文件解析的核心基础设施库**，为多个关键模块提供 ELF 处理能力。

### 使用场景

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐    │
│  │ hapsigner    │   │ arkcompiler  │   │ libbpf      │    │
│  │ 签名工具     │   │ 运行时编译   │   │ BPF 运行时  │    │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘    │
│         │                  │                  │            │
│         └──────────────────┼──────────────────┘            │
│                            │                               │
│                    ┌──────┴──────┐                        │
│                    │   ELFIO      │                        │
│                    │   (共享库)   │                        │
│                    └─────────────┘                        │
│                            │                               │
└────────────────────────────┼────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │  ELF 格式解析   │
                    │  - 文件结构    │
                    │  - 符号信息    │
                    │  - 节区数据    │
                    │  - 重定位信息  │
                    └────────────────┘
```

### 与其他库的关系

- **上游**：ELFIO 独立于其他库
- **下游**：被签名、编译、BPF 等模块依赖
- **替代性**：无直接替代品，提供底层 ELF 解析能力

### 在 OH 中的独特价值

1. **轻量级**：纯头文件，集成成本低
2. **标准化**：符合 ELF 标准，兼容性好
3. **C 接口**：支持 C 项目使用（如 libbpf）
4. **MIT 许可证**：商业友好

---

## 版本信息

### OH 版本

| 属性 | 值 |
|------|-----|
| **OH 版本** | 3.12 |
| **发布形式** | code-segment |
| **组件名** | @ohos/elfio |
| **子系统** | thirdparty |

### 版本差异

OH 版本与上游版本同步，未进行版本定制。

---

## 快速开始

### 在 OH 项目中使用

```gn
// BUILD.gn
external_deps = [ "elfio:elfio" ]
```

### C++ 接口

```cpp
#include <elfio/elfio.hpp>

// 加载 ELF 文件
ELFIO::elfio elf_reader;
elf_reader.load("example.elf");

// 获取 section 信息
for (const auto& section : elf_reader.sections) {
    printf("Section: %s\n", section->get_name().c_str());
}

// 遍历符号
const auto& sym_section = elf_reader.sections[".symtab"];
ELFIO::symbol_section_accessor symbols(elf_reader, sym_section);
for (Elf_Xword i = 0; i < symbols.get_symbols_num(); ++i) {
    std::string name;
    Elf64_Addr value;
    symbols.get_symbol(i, name, value, ...);
}
```

### C 接口

```c
#include <elfio_c_wrapper.h>

pelfio_t elf = elfio_new();
elfio_load(elf, "example.elf");

Elf_Half num_sections = elfio_get_sections_num(elf);
// ... 使用 C 接口操作 ELF
elfio_delete(elf);
```

---

## 相关资源

- [上游官方文档](http://elfio.sourceforge.net/elfio.pdf)
- [GitHub 仓库](https://github.com/serge1/ELFIO)
- [OH 构建配置](./03_Build_Integration.md)
- [OH 使用场景](./04_Usage_in_OH.md)

# 目录结构与模块职责

## 目的与适用范围

本文档介绍 `global_resource_tool` 的代码组织结构，帮助开发者快速定位代码和理解模块职责。

**适用对象**: 新加入的开发者、代码维护人员

---

## 顶层目录结构

```
developtools/global_resource_tool/
├── include/              # 头文件目录
├── src/                  # 源代码目录
├── test/                 # 测试代码（本文档不覆盖）
├── wiki/                 # 本文档目录
├── BUILD.gn             # GN 构建脚本
├── bundle.json          # 组件配置
├── restool.gni          # GN 变量定义
├── CMakeLists.txt       # CMake 构建配置
├── win32.cmake          # Windows 交叉编译脚本
├── id_defines.json      # ID 定义 schema
├── restool_faq.json     # FAQ 配置
├── README.md            # 英文说明
├── README_zh.md         # 中文说明
└── LICENSE              # Apache 2.0 许可证
```

---

## Include 目录

### 文件列表

| 文件 | 职责 | 关键类/函数 |
|------|------|-------------|
| `append_compiler.h` | 追加编译器 | `AppendCompiler` |
| `binary_file_packer.h` | 二进制文件打包 | `BinaryFilePacker` |
| `compression_parser.h` | 压缩配置解析 | `CompressionParser` |
| `config_parser.h` | 配置解析 | `ConfigJson` |
| `file_entry.h` | 文件/目录操作 | `FileEntry` |
| `file_manager.h` | 文件管理 | `FileManager` |
| `generic_compiler.h` | 通用编译器 | `GenericCompiler` |
| `header.h` | 头文件生成 | `Header` |
| `i_resource_compiler.h` | 编译器接口 | `IResourceCompiler` |
| `id_defined_parser.h` | ID 定义解析 | `IdDefinedParser` |
| `id_worker.h` | ID 分配器 | `IdWorker` |
| `json_compiler.h` | JSON 编译器 | `JsonCompiler` |
| `key_parser.h` | 限定词解析 | `KeyParser` |
| `no_copy_able.h` | 不可拷贝基类 | `NoCopyAble` |
| `overlap_binary_file_packer.h` | 叠加打包器 | `OverlapBinaryFilePacker` |
| `overlap_compiler.h` | 叠加编译器 | `OverlapCompiler` |
| `reference_parser.h` | 引用解析 | `ReferenceParser` |
| `resconfig_parser.h` | 资源配置解析 | `ResConfigParser` |
| `resource_append.h` | 资源追加 | `ResourceAppend` |
| `resource_check.h` | 资源检查 | `ResourceCheck` |
| `resource_compiler_factory.h` | 编译器工厂 | `ResourceCompilerFactory` |
| `resource_data.h` | 数据定义 | 枚举、常量、结构体 |
| `resource_directory.h` | 资源目录 | `ResourceDirectory` |
| `resource_dumper.h` | 资源转储 | `ResourceDumper` |
| `resource_item.h` | 资源项 | `ResourceItem` |
| `resource_merge.h` | 资源合并 | `ResourceMerge` |
| `resource_module.h` | 资源模块 | `ResourceModule` |
| `resource_overlap.h` | 资源叠加 | `ResourceOverlap` |
| `resource_pack.h` | 资源打包 | `ResourcePack` |
| `resource_packer_factory.h` | 打包器工厂 | `ResourcePackerFactory` |
| `resource_table.h` | 资源表 | `ResourceTable` |
| `resource_util.h` | 工具函数 | `ResourceUtil` |
| `restool_errors.h` | 错误定义 | 错误码、错误信息 |
| `select_compile_parse.h` | 选择编译解析 | `SelectCompileParse` |
| `singleton.h` | 单例基类 | `Singleton` |
| `thread_pool.h` | 线程池 | `ThreadPool` |
| `translatable_parser.h` | 可翻译解析 | `TranslatableParser` |

### Cmd 子目录

| 文件 | 职责 | 关键类 |
|------|------|--------|
| `cmd/cmd_parser.h` | 命令解析基类 | `CmdParserBase`、`CmdParser` |
| `cmd/dump_parser.h` | Dump 命令解析 | `DumpParser` |
| `cmd/package_parser.h` | 打包命令解析 | `PackageParser` |

---

## Src 目录

### 按模块分组

#### 1. 命令解析模块 (src/cmd/)

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `cmd/cmd_parser.cpp` | `cmd/cmd_parser.h` | 命令解析基类实现 |
| `cmd/dump_parser.cpp` | `cmd/dump_parser.h` | dump 子命令实现 |
| `cmd/package_parser.cpp` | `cmd/package_parser.h` | 打包命令参数解析 |

**关键调用链**:
```
main() [restool.cpp:24]
  → CmdParser::Parse() [cmd/cmd_parser.cpp:39]
    → PackageParser::Parse() [cmd/package_parser.cpp:62]
      → PackageParser::ExecCommand() [cmd/package_parser.cpp:662]
        → ResourcePack::Package() [resource_pack.cpp:37]
```

#### 2. 资源打包模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `resource_pack.cpp` | `resource_pack.h` | 资源打包主逻辑 |
| `resource_table.cpp` | `resource_table.h` | 资源表生成与管理 |
| `resource_item.cpp` | `resource_item.h` | 资源项定义 |
| `resource_directory.cpp` | `resource_directory.h` | 资源目录处理 |
| `resource_module.cpp` | `resource_module.h` | 资源模块管理 |
| `resource_merge.cpp` | `resource_merge.h` | 资源合并 |
| `resource_overlap.cpp` | `resource_overlap.h` | 资源叠加 |
| `resource_append.cpp` | `resource_append.h` | 资源追加 |

#### 3. 编译器模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `i_resource_compiler.cpp` | `i_resource_compiler.h` | 编译器接口实现 |
| `json_compiler.cpp` | `json_compiler.h` | JSON 资源编译 |
| `generic_compiler.cpp` | `generic_compiler.h` | 通用编译器 |
| `append_compiler.cpp` | `append_compiler.h` | 追加编译器 |
| `overlap_compiler.cpp` | `overlap_compiler.h` | 叠加编译器 |
| `resource_compiler_factory.cpp` | `resource_compiler_factory.h` | 编译器工厂 |

#### 4. 文件操作模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `file_entry.cpp` | `file_entry.h` | 文件/目录操作封装 |
| `file_manager.cpp` | `file_manager.h` | 文件管理器 |
| `binary_file_packer.cpp` | `binary_file_packer.h` | 二进制文件打包 |
| `overlap_binary_file_packer.cpp` | `overlap_binary_file_packer.h` | 叠加二进制打包 |

**关键函数** (`src/file_entry.cpp`):
- `FileEntry::Exist()` - 检查文件存在
- `FileEntry::CreateDirs()` - 创建目录
- `FileEntry::RemoveAllDir()` - 递归删除目录
- `FileEntry::CopyFileInner()` - 复制文件

#### 5. 解析器模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `config_parser.cpp` | `config_parser.h` | module.json/config.json 解析 |
| `key_parser.cpp` | `key_parser.h` | 限定词解析 |
| `reference_parser.cpp` | `reference_parser.h` | 资源引用解析 |
| `resconfig_parser.cpp` | `resconfig_parser.h` | 资源配置文件解析 |
| `compression_parser.cpp` | `compression_parser.h` | 纹理压缩配置解析 |
| `id_defined_parser.cpp` | `id_defined_parser.h` | ID 定义文件解析 |
| `translatable_parser.cpp` | `translatable_parser.h` | 可翻译资源解析 |
| `select_compile_parse.cpp` | `select_compile_parse.h` | 选择编译配置解析 |

#### 6. 工具模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `thread_pool.cpp` | `thread_pool.h` | 线程池实现 |
| `id_worker.cpp` | `id_worker.h` | 资源 ID 分配 |
| `resource_util.cpp` | `resource_util.h` | 通用工具函数 |
| `restool_errors.cpp` | `restool_errors.h` | 错误处理 |
| `header.cpp` | `header.h` | 头文件生成 |
| `resource_check.cpp` | `resource_check.h` | 资源检查 |
| `resource_dumper.cpp` | `resource_dumper.h` | 资源转储 |

#### 7. 工厂模块

| 文件 | 对应头文件 | 职责 |
|------|-----------|------|
| `resource_packer_factory.cpp` | `resource_packer_factory.h` | 打包器工厂 |
| `resource_compiler_factory.cpp` | `resource_compiler_factory.h` | 编译器工厂 |

#### 8. 入口模块

| 文件 | 职责 |
|------|------|
| `restool.cpp` | 程序入口 main() |

---

## 模块依赖关系

### 高层依赖图

```
restool.cpp (入口)
    ↓
CmdParser (命令解析)
    ↓
PackageParser (参数解析)
    ↓
ResourcePack (打包主逻辑)
    ↓
    ├── ResourceTable (资源表生成)
    ├── FileManager (文件管理)
    ├── ResourceCompilerFactory (编译器工厂)
    │       ↓
    │   IResourceCompiler (编译器接口)
    │       ↓
    │   JsonCompiler / GenericCompiler / ...
    ├── ResourcePackerFactory (打包器工厂)
    ├── ThreadPool (线程池)
    └── CompressionParser (压缩配置)
```

### 关键数据流

```
输入资源目录
    ↓
ResourceDirectory (扫描目录)
    ↓
IResourceCompiler (编译资源)
    ↓
ResourceItem (资源项)
    ↓
FileManager (资源管理)
    ↓
ResourceTable (生成索引)
    ↓
输出文件 (resources.index, ResourceTable.h)
```

---

## 代码规范

### 命名空间

所有代码位于命名空间 (`src/*.cpp`):
```cpp
namespace OHOS {
namespace Global {
namespace Restool {
    // 代码实现
}
}
}
```

### 类命名规范

- 接口类: `I` 前缀，如 `IResourceCompiler`
- 工厂类: `Factory` 后缀，如 `ResourceCompilerFactory`
- 解析器类: `Parser` 后缀，如 `PackageParser`
- 单例类: 继承 `Singleton` 模板

### 文件命名规范

- 头文件: 小写下划线，如 `resource_data.h`
- 实现文件: 与头文件同名，如 `resource_data.cpp`
- 目录名: 小写，如 `cmd/`

---

## 相关文档

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [内部接口](04_Internal_API.md) - 模块接口详情

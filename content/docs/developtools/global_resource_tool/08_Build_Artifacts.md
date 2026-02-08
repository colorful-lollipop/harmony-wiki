# 编译产物

## 目的与适用范围

本文档介绍 `global_resource_tool` 的编译产物，包括输出文件、安装路径和运行时加载关系。

**适用对象**: 构建工程师、发布管理人员

---

## 产物清单

### 主要产物

| 产物 | 类型 | 说明 | 构建目标 |
|------|------|------|----------|
| `restool` | 可执行文件 | 资源编译工具主程序 | `restool` |
| `id_defined.json` | JSON 配置 | 系统资源 ID 定义 | `restool_id_defined` |
| `restool_faq.json` | JSON 配置 | 错误帮助信息 | `restool_faq` |
| `id_defines.json` | JSON Schema | ID 定义文件 schema | `restool_ids_schema` |

### 产物类型说明

#### 1. restool 可执行文件

**平台差异**:
| 平台 | 文件名 | 说明 |
|------|--------|------|
| Linux | `restool` | ELF 可执行文件 |
| macOS | `restool` | Mach-O 可执行文件 |
| Windows | `restool.exe` | PE 可执行文件 |

**构建输出路径**:
```
out/{target_cpu}-{target_os}-{target_config}/developtools/global_resource_tool/restool
```

**示例**:
```
out/linux-x64-release/developtools/global_resource_tool/restool
out/darwin-x64-release/developtools/global_resource_tool/restool
```

#### 2. 配置文件

**id_defined.json**:
- 用途: 定义系统资源的 ID 值
- 来源: `//base/global/system_resources/systemres/main/resources/base/element/id_defined.json`
- 格式: JSON 数组，包含资源类型、名称、ID 映射

**restool_faq.json**:
- 用途: 错误码帮助信息
- 来源: `//developtools/global_resource_tool/restool_faq.json`
- 格式: JSON 对象，包含错误码对应的解决方案链接

**id_defines.json**:
- 用途: ID 定义文件的 JSON Schema
- 来源: `//developtools/global_resource_tool/id_defines.json`
- 格式: JSON Schema

---

## 安装路径

### SDK 安装路径

**标准安装位置**:
```
{SDK_ROOT}/toolchains/restool
```

**完整路径示例**:
```
# Linux/macOS
~/ohos-sdk/linux/toolchains/restool

# Windows
C:\Users\{username}\ohos-sdk\windows\toolchains\restool.exe
```

### 配置文件安装路径

| 文件 | 安装路径 |
|------|----------|
| `id_defined.json` | `{SDK_ROOT}/toolchains/id_defined.json` |
| `restool_faq.json` | 不安装（仅构建使用） |
| `id_defines.json` | 不安装（仅构建使用） |

---

## 运行时加载关系

### 动态依赖

**可选动态库**:
| 库名 | 平台 | 用途 | 加载时机 |
|------|------|------|----------|
| `libimage_transcoder_shared.so` | Linux | 纹理压缩 | 启用纹理压缩时 |
| `libimage_transcoder_shared.dylib` | macOS | 纹理压缩 | 启用纹理压缩时 |
| `libimage_transcoder_shared.dll` | Windows | 纹理压缩 | 启用纹理压缩时 |

**加载代码** (`src/compression_parser.cpp`):
```cpp
#ifdef __WIN32
    handle_ = LoadLibraryW(AdaptLongPathW(extensionPath_).c_str());
#else
    handle_ = dlopen(extensionPath_.c_str(), RTLD_LAZY);
#endif
```

### 运行时文件访问

**输入文件**:
| 类型 | 路径来源 | 说明 |
|------|----------|------|
| 资源目录 | `-i` 参数 | 用户指定的资源目录 |
| 配置文件 | `-j` 参数 | module.json 或 config.json |
| ID 定义 | `--defined-ids` 参数 | 自定义 ID 定义文件 |
| 压缩配置 | `--compressed-config` 参数 | 纹理压缩配置 |

**输出文件**:
| 类型 | 输出路径 | 说明 |
|------|----------|------|
| 资源索引 | `{output}/resources.index` | 资源索引文件 |
| 资源目录 | `{output}/resources/` | 编译后的资源文件 |
| 头文件 | `-r` 参数指定 | ResourceTable.h/js/txt/ts |
| ID 定义 | `--ids` 参数指定 | 生成的 id_defined.json |

---

## 产物使用场景

### 场景 1: IDE 集成

**DevEco Studio 调用**:
```bash
{SDK}/toolchains/restool \
    -i {project}/entry/src/main \
    -j {project}/entry/src/main/module.json \
    -p com.example.app \
    -o {project}/build/intermediates/res \
    -r {project}/build/ResourceTable.h \
    -f
```

### 场景 2: 命令行使用

**手动编译资源**:
```bash
# 设置 restool 路径
export RESTOOL={SDK}/toolchains/restool

# 编译资源
$RESTOOL -i ./resources \
         -j ./module.json \
         -p com.example.app \
         -o ./out \
         -r ./out/ResourceTable.h \
         -f
```

### 场景 3: 构建系统集成

**Hvigor 集成**:
- Hvigor 自动调用 restool 编译资源
- 通过 `build-profile.json5` 配置资源编译选项
- 输出到 `build/default/intermediates/res`

---

## 产物版本信息

### 版本号

**当前版本** (`include/resource_data.h:53`):
```cpp
static const std::string RESTOOL_VERSION = { " 6.1.0.003" };
```

**版本查询**:
```bash
restool -v
# 输出: Info: Restool version = RestoolV2 6.1.0.003
```

### 兼容性

**API 版本对应关系**:
| restool 版本 | 支持 API 版本 | 主要特性 |
|--------------|---------------|----------|
| 6.1.0.003 | 18-23+ | 多线程、忽略规则、TS 头文件 |

---

## 产物验证

### 文件存在性检查

```bash
# 检查 restool 是否存在
ls -la {SDK}/toolchains/restool

# 检查配置文件
ls -la {SDK}/toolchains/id_defined.json
```

### 功能验证

```bash
# 查看版本
restool -v

# 查看帮助
restool -h

# 测试编译
restool -i ./test_resources -j ./module.json -p test -o ./test_out -r ./test_out/ResourceTable.h -f
```

---

## 产物大小估算

| 产物 | 估算大小 | 说明 |
|------|----------|------|
| `restool` 可执行文件 | 2-5 MB | 取决于平台和编译选项 |
| `id_defined.json` | ~10 KB | 系统资源 ID 定义 |
| `restool_faq.json` | ~2 KB | FAQ 配置 |

**总安装大小**: 约 5-10 MB

---

## 相关文档

- [GN 构建目标](07_GN_Targets.md) - 构建配置
- [对外接口](03_Public_API.md) - 命令行使用
- [常见问题](08_FAQ.md) - 使用问题排查

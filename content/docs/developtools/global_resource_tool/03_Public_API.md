# 对外接口

## 目的与适用范围

本文档介绍 `global_resource_tool` 的对外接口。注意：本项目为**纯命令行工具**，不提供 N-API 接口。

**适用对象**: 构建工程师、IDE 开发者、工具使用者

---

## 接口概述

### 接口类型

| 接口类型 | 支持状态 | 说明 |
|----------|----------|------|
| 命令行接口 | ✅ | 主要使用方式 |
| N-API | ❌ | 不支持 |
| IPC/SA | ❌ | 不支持 |
| 动态库导出 | ❌ | 不支持 |

### 调用方式

```bash
restool [子命令] [选项]...
```

---

## 命令行参数

### 全局选项

| 选项 | 长选项 | 是否必填 | 参数 | 说明 |
|------|--------|----------|------|------|
| `-i` | `--inputPath` | 是 | 路径 | 输入资源目录，可多次指定 |
| `-j` | `--json` | 是* | 路径 | module.json 或 config.json 路径 |
| `-o` | `--outputPath` | 是 | 路径 | 输出目录 |
| `-p` | `--packageName` | 是* | 名称 | Bundle 名称 |
| `-r` | `--resHeader` | 是* | 路径 | 资源头文件输出路径 |
| `-f` | `--forceWrite` | 否 | - | 强制覆盖输出目录 |
| `-h` | `--help` | 否 | - | 显示帮助信息 |
| `-v` | `--version` | 否 | - | 显示版本号 |

*注：某些参数在特定模式下为可选

### 高级选项

| 选项 | 长选项 | 是否必填 | 参数 | 说明 |
|------|--------|----------|------|------|
| `-e` | `--startId` | 否 | ID | 起始资源 ID（如 0x01000000） |
| `-m` | `--modules` | 否 | 列表 | 模块名列表，逗号分隔 |
| `-x` | `--append` | 否 | 路径 | 追加资源目录 |
| `-z` | `--combine` | 否 | - | 增量编译标志 |
| `-l` | `--fileList` | 否 | 路径 | 配置文件路径（resConfig.json） |
| | `--ids` | 否 | 目录 | 生成 id_defined.json 的目录 |
| | `--defined-ids` | 否 | 路径 | 指定 id_defined.json 路径 |
| | `--dependEntry` | 否 | 路径 | Entry 模块编译结果目录 |
| | `--icon-check` | 否 | - | 启用 PNG 图标校验 |
| | `--target-config` | 否 | 配置 | 选择编译配置 |
| | `--compressed-config` | 否 | 路径 | 纹理压缩配置文件 |
| | `--thread` | 否 | 数量 | 线程数（API 18+） |
| | `--ignored-file` | 否 | 规则 | 忽略文件规则（正则，冒号分隔） |
| | `--ignored-path` | 否 | 规则 | 忽略路径规则（正则，冒号分隔） |

### 代码定义

**选项定义** (`src/cmd/package_parser.cpp:33-58`):
```cpp
const struct option PackageParser::CMD_OPTS[] = {
    { "inputPath", required_argument, nullptr, Option::INPUTPATH },
    { "packageName", required_argument, nullptr, Option::PACKAGENAME },
    { "outputPath", required_argument, nullptr, Option::OUTPUTPATH },
    { "resHeader", required_argument, nullptr, Option::RESHEADER },
    { "forceWrite", no_argument, nullptr, Option::FORCEWRITE },
    { "version", no_argument, nullptr, Option::VERSION},
    // ... 更多选项
};
```

**参数解析** (`src/cmd/package_parser.cpp:60`):
```cpp
uint32_t PackageParser::Parse(int argc, char *argv[])
```

---

## 子命令

### dump 命令

**功能**: 以 JSON 格式输出 HAP 包中的资源内容

**用法**:
```bash
restool dump [-h] [config] <hap文件路径>
```

**参数**:
| 参数 | 是否必填 | 说明 |
|------|----------|------|
| `-h` | 否 | 显示帮助 |
| `config` | 否 | 只打印限定词信息 |
| `filePath` | 是 | HAP 文件路径 |

**示例**:
```bash
# 打印所有资源信息
restool dump entry.hap

# 只打印限定词信息
restool dump config entry.hap
```

**实现** (`src/cmd/dump_parser.cpp`):
```cpp
uint32_t DumpParser::ParseOption(int argc, char *argv[], int currentIndex)
```

---

## 参数校验

### 输入路径校验

**实现** (`src/cmd/package_parser.cpp:130-154`):
```cpp
uint32_t PackageParser::AddInput(const string& argValue)
```

**校验规则**:
1. 路径必须存在
2. 路径必须是 ASCII 字符（Windows）
3. 检查重复输入

### 起始 ID 校验

**实现** (`src/cmd/package_parser.cpp:290-309`):
```cpp
uint32_t PackageParser::AddStartId(const string& argValue)
```

**校验规则**:
1. 必须是有效的十六进制数
2. 范围: `[0x01000000, 0x06FFFFFF)` 或 `[0x08000000, 0xFFFFFFFF)`
3. 必须大于 0

### 线程数校验

**实现** (`src/cmd/package_parser.cpp:452-470`):
```cpp
uint32_t PackageParser::ParseThread(const std::string &argValue)
```

**校验规则**:
1. 必须是有效的整数
2. 必须大于 0

---

## 错误码

### 错误码结构

错误码格式: `112XXXXX`

| 范围 | 类型 |
|------|------|
| `11200000` | 未定义错误 |
| `11201xxx` | 依赖错误 |
| `11203xxx` | 配置错误 |
| `11204xxx` | 文件资源错误 |
| `11210xxx` | 命令解析错误 |
| `11211xxx` | 资源打包错误 |
| `11212xxx` | 资源转储错误 |

### 常见错误码

| 错误码 | 名称 | 说明 |
|--------|------|------|
| `11210001` | ERR_CODE_UNKNOWN_OPTION | 未知选项 |
| `11210002` | ERR_CODE_MISSING_ARGUMENT | 缺少参数 |
| `11210004` | ERR_CODE_INVALID_INPUT | 无效输入路径 |
| `11210007` | ERR_CODE_INVALID_OUTPUT | 无效输出路径 |
| `11210013` | ERR_CODE_INVALID_START_ID | 无效起始 ID |
| `11210026` | ERR_CODE_INVALID_THREAD_COUNT | 无效线程数 |
| `11211001` | ERR_CODE_OUTPUT_EXIST | 输出目录已存在 |
| `11211002` | ERR_CODE_CONFIG_JSON_MISSING | 缺少配置文件 |

**定义位置** (`include/restool_errors.h`):
```cpp
constexpr uint32_t ERR_CODE_UNKNOWN_OPTION = 11210001;
constexpr uint32_t ERR_CODE_INVALID_INPUT = 11210004;
// ... 更多错误码
```

### 错误信息格式

**结构** (`include/restool_errors.h:148-205`):
```cpp
class ErrorInfo {
    uint32_t code_;                    // 错误码
    std::string description_;          // 描述
    std::string cause_;                // 原因
    std::string position_;             // 位置
    std::vector<std::string> solutions_; // 解决方案
    MoreInfo moreInfo_;                // 更多信息
};
```

---

## 配置文件格式

### resConfig.json

**用途**: 通过 `-l` 选项指定，批量设置参数

**字段映射**:
| 字段 | 对应选项 | 类型 |
|------|----------|------|
| `configPath` | `-j` | string |
| `packageName` | `-p` | string |
| `output` | `-o` | string |
| `startId` | `-e` | string |
| `moduleNames` | `-m` | string |
| `ResourceTable` | `-r` | string[] |
| `applicationResource` | `-i` | string |
| `moduleResources` | `-i` | string[] |
| `dependencies` | `-i` | string[] |
| `iconCheck` | `--icon-check` | boolean |
| `thread` | `--thread` | integer |

**示例**:
```json
{
    "configPath": "entry/src/main/module.json",
    "packageName": "com.example.app",
    "output": "out",
    "ResourceTable": ["out/ResourceTable.h"],
    "moduleResources": ["entry/src/main"],
    "thread": 4
}
```

---

## 使用示例

### 基本编译

```bash
restool -i entry/src/main \
        -j entry/src/main/module.json \
        -p com.example.app \
        -o out \
        -r out/ResourceTable.h \
        -f
```

### 多模块编译

```bash
restool -i entry/src/main \
        -i feature/src/main \
        -j entry/src/main/module.json \
        -p com.example.app \
        -o out \
        -r out/ResourceTable.h \
        -m entry,feature \
        -f
```

### 增量编译

```bash
# 1. 生成资源中间件
restool -x entry/src/main/resource -o out

# 2. 编译中间件
restool -i out1 -i out2 -o out \
        -p com.example.app \
        -r out/ResourceTable.h \
        -j entry/src/main/module.json \
        -f -z
```

### 叠加编译

```bash
restool -i entry/src/main \
        -i hapResource \
        -j entry/src/main/module.json \
        -p com.example.app \
        -o out \
        -r out/ResourceTable.txt \
        -f
```

### 固定资源 ID

```bash
# 生成 id_defined.json
restool -i entry/src/main \
        -j entry/src/main/module.json \
        -p com.example.app \
        -o out \
        -r out/ResourceTable.txt \
        --ids out \
        -f

# 使用自定义 ID
restool -i entry/src/main \
        -j entry/src/main/module.json \
        -p com.example.app \
        -o out1 \
        -r out1/ResourceTable.txt \
        --defined-ids out/id_defined.json \
        -f
```

---

## 相关文档

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [常见问题](08_FAQ.md) - 使用问题排查

# 07 - 构建与产物 (Build)

## 1. GN 构建系统

### 1.1 构建配置概述

**主构建文件**: `BUILD.gn` (根目录)

**构建目标**:
- `idl` - IDL工具可执行文件
- `idl_group` - 目标组（host + target）

### 1.2 构建目标详解

**idl 可执行文件**

```gn
# BUILD.gn:377-392
ohos_executable("idl") {
  sources = [ "idl_tool_2/main.cpp" ]
  sources += common_sources  # 所有源代码
  
  configs = [ ":idl_config" ]
  use_exceptions = true
  
  external_deps = [ "bounds_checking_function:libsec_static" ]
  
  install_enable = false
  part_name = "idl_tool"
  subsystem_name = "ability"
}
```

**idl_group 目标组**

```gn
# BUILD.gn:394-399
group("idl_group") {
  deps = [
    ":idl",
    ":idl($host_toolchain)",  # host 版本
  ]
}
```

---

## 2. 源代码组织

### 2.1 源代码分类

**common_sources** 包含以下模块：

| 类别 | 文件数量 | 说明 |
|------|---------|------|
| AST 基础类型 | 17对 | bool, int, string 等基本类型 |
| AST 复合类型 | 25对 | interface, struct, array, map 等 |
| HDI C 生成器 | 10对 | C语言代码生成 |
| HDI C++ 生成器 | 8对 | C++语言代码生成 |
| HDI Java 生成器 | 3对 | Java代码生成 |
| HDI 类型发射器 | 24对 | HDI类型处理 |
| SA C++ 生成器 | 6对 | SA C++代码生成 |
| SA Rust 生成器 | 2对 | Rust代码生成 |
| SA TS 生成器 | 4对 | TypeScript代码生成 |
| SA 类型发射器 | 29对 | SA类型处理 |
| Lexer | 2对 | 词法分析 |
| Parser | 2对 | 语法分析 |
| Metadata | 9对 | 元数据处理 |
| Preprocessor | 1对 | 预处理器 |
| Hash | 1对 | Hash计算 |
| Util | 9对 | 工具类 |

**总计**: 约 150+ 个源文件

### 2.2 依赖关系

**外部依赖** (`bundle.json:22-31`):
```json
"deps": {
  "components": [
    "hilog",                    # 日志
    "ipc",                      # IPC框架
    "samgr",                    # 系统能力管理器
    "safwk",                    # 系统服务框架
    "c_utils",                  # C工具库
    "bounds_checking_function"  # 边界检查函数
  ]
}
```

**编译依赖**:
- `bounds_checking_function:libsec_static` - 安全字符串函数

---

## 3. 编译产物

### 3.1 主产物

| 产物 | 类型 | 说明 |
|------|------|------|
| `idl` | 可执行文件 | IDL工具主程序 |
| `idl($host_toolchain)` | 可执行文件 | Host版本 |

### 3.2 产物路径

```
out/标准产品目录/
├── foundation/ability/idl_tool/
│   └── idl                # 目标平台版本
└── host_tools/
    └── foundation/ability/idl_tool/
        └── idl            # Host平台版本
```

### 3.3 产物特性

- **静态链接**: 不依赖动态库，可独立运行
- **双版本**: 同时编译目标平台和Host平台版本
- **无安装**: `install_enable = false`

---

## 4. Feature 开关

### 4.1 编译选项

**配置**: `config("idl_config")`

```gn
config("idl_config") {
  include_dirs = [ "./idl_tool_2" ]
}
```

### 4.2 语言支持

通过命令行选项选择，非编译开关：
- `--gen-c` - C 语言
- `--gen-cpp` - C++ 语言
- `--gen-java` - Java 语言
- `--gen-ts` - TypeScript 语言
- `--gen-rust` - Rust 语言

### 4.3 系统级别支持

**SystemLevel 枚举** (`util/common.h:30-35`):
```cpp
enum class SystemLevel {
    INIT,   // 初始
    MINI,   // 迷你系统
    LITE,   // 轻量级系统
    FULL,   // 标准系统
};
```

### 4.4 生成模式

**GenMode 枚举** (`util/common.h:37-43`):
```cpp
enum class GenMode {
    LOW,          // 低功耗（仅MINI）
    PASSTHROUGH,  // 穿透模式
    IPC,          // IPC模式（仅FULL）
    KERNEL,       // 内核模式
};
```

---

## 5. 测试构建

### 5.1 测试目标

**测试目录**:
- `test/unittest/` - 单元测试
- `test/native/` - 原生测试
- `test/ts/` - TypeScript测试
- `test/rust/` - Rust测试
- `idl_tool_2/test/unittest/` - 新版单元测试

**测试目标配置** (`bundle.json:43-50`):
```json
"test": [
  "//foundation/ability/idl_tool/test/rust/moduletest:moduletest",
  "//foundation/ability/idl_tool/test/rust/unittest:unittest",
  "//foundation/ability/idl_tool/test/ts/moduletest:moduletest",
  "//foundation/ability/idl_tool/test/ts/unittest:unittest",
  "//foundation/ability/idl_tool/test/unittest:unittest",
  "//foundation/ability/idl_tool/idl_tool_2/test/unittest:unittest"
]
```

### 5.2 运行测试

```bash
# 编译测试
./build.sh --product <product> --build-target idl_tool_test

# 运行测试
./out/<product>/tests/idl_tool/unittest
```

---

## 6. 构建命令参考

### 6.1 完整编译

```bash
# 编译整个项目（包含IDL工具）
./build.sh --product <product_name>

# 只编译IDL工具
./build.sh --product <product_name> --build-target //foundation/ability/idl_tool:idl
```

### 6.2 清理构建

```bash
# 清理并重新编译
rm -rf out/<product>/foundation/ability/idl_tool
./build.sh --product <product_name> --build-target //foundation/ability/idl_tool:idl
```

### 6.3 验证构建

```bash
# 检查产物
ls -la out/<product>/foundation/ability/idl_tool/

# 测试运行
./out/<product>/foundation/ability/idl_tool/idl --version
```

---

## 7. 集成到产品

### 7.1 产品配置

在产品的 `config.json` 中引入:
```json
{
  "subsystem": "ability",
  "components": [
    "idl_tool"
  ]
}
```

### 7.2 自定义编译

**自定义 GN 参数**:
```gn
# 在 product.gni 中
idl_tool_enable_xxx = true
```

---

## 8. 关键证据

| 项目 | 路径 | 行号 |
|------|------|------|
| 主构建配置 | `BUILD.gn` | 1-400 |
| 可执行目标 | `BUILD.gn` | 377-392 |
| 目标组 | `BUILD.gn` | 394-399 |
| 组件配置 | `bundle.json` | 1-54 |
| 依赖定义 | `bundle.json` | 22-31 |
| 测试目标 | `bundle.json` | 43-50 |

---

**上一步**: [06_SecurityReview.md](06_SecurityReview.md)
**下一步**: [08_Internals.md](08_Internals.md)

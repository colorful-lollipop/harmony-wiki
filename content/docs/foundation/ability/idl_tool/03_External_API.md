# OpenHarmony IDL Tool - 对外 API

**目的**：说明 IDL Tool 对外提供的命令行接口和使用方式。

**注意**：IDL Tool 是代码生成器，**不提供 N-API（JavaScript API）**。本文档仅描述命令行工具接口。

---

## 适用范围

本文档适用于：
- 使用 IDL Tool 生成代码的开发者
- 需要在构建脚本中集成 IDL 工具的工程师
- 希望自动化 IDL 代码生成的 CI/CD 工程师

---

## 命令行接口

### 基本用法

```bash
# 显示帮助信息
idl -h

# 显示版本信息
idl -v
```

### 编译模式

```bash
# 编译 IDL 文件（生成 AST，不生成代码）
idl -c <idl_file>

# 编译多个 IDL 文件
idl -c file1.idl -c file2.idl

# 指定包名
idl -c <idl_file> --package <package_name>

# 指定源代码目录
idl -c <idl_file> -D <source_dir>
```

**参数说明**：
- `-c` 或 `--compile` - 编译 IDL 文件，生成 AST
- `-D <directory>` - 指定源代码根目录
- `--package <name>` - 覆盖包名

---

## 代码生成模式

### C++ 代码生成

```bash
# 生成 C++ 代码
idl -gen-cpp -d <output_dir> <idl_file>

# 指定生成模式（仅 SA 类型支持）
idl -gen-cpp -d <output_dir> --gen-cpp <idl_file>
```

**生成的文件**（SA 模式）：
- `<interface_name>.h` - 接口定义
- `<interface_name>_proxy.h` - 客户端代理
- `<interface_name>_stub.h` - 服务端桩
- `<interface_name>_proxy.cpp` - 代理实现
- `<interface_name>_stub.cpp` - 桩实现

### TypeScript 代码生成

```bash
# 生成 TypeScript 代码
idl -gen-ts -d <output_dir> <idl_file>

# 指定生成模式（仅 SA 类型支持）
idl -gen-ts -d <output_dir> --gen-ts <idl_file>
```

**生成的文件**（SA 模式）：
- `<interface_name>.d.ts` - 接口类型定义
- `<interface_name>_proxy.ts` - 客户端代理
- `<interface_name>_stub.ts` - 服务端桩

### Java 代码生成（仅 HDI）

```bash
# 生成 Java 代码（HDI 类型）
idl -gen-java -d <output_dir> <idl_file>

# 指定生成模式（仅 HDI 类型支持）
idl -gen-java -d <output_dir> --gen-java <idl_file>
```

**生成的文件**（HDI 模式）：
- `I<InterfaceName>.java` - Java 接口定义

### C 代码生成（仅 HDI）

```bash
# 生成 C 代码（HDI 类型）
idl -gen-c -d <output_dir> <idl_file>

# 指定生成模式（仅 HDI 类型支持）
idl -gen-c -d <output_dir> --gen-c <idl_file>
```

**生成的文件**（HDI 模式）：
- `<interface_name>.h` - C 语言接口定义
- `<interface_name>_service.h` - C 语言服务驱动
- `<interface_name>_driver.h` - C 语言驱动实现

### Rust 代码生成（仅 SA）

```bash
# 生成 Rust 代码（SA 类型）
idl -gen-rust -d <output_dir> <idl_file>

# 指定生成模式（仅 SA 类型支持）
idl -gen-rust -d <output_dir> --gen-rust <idl_file>
```

**生成的文件**（SA 模式）：
- `mod.rs` - Rust 模块文件

---

## 高级选项

### AST 转储

```bash
# 转储 AST 结构（用于调试）
idl -c <idl_file> --dump-ast
```

### 元数据操作

```bash
# 转储元数据（仅 SA 类型）
idl -c <idl_file> --dump-metadata

# 保存元数据到文件（仅 SA 类型）
idl -c <idl_file> -s <metadata_file>
```

**元数据用途**：
- 运行时类型信息（RTTI）
- 跨语言绑定支持
- 类型反射

### 包路径映射

```bash
# 指定包路径映射
idl -c <idl_file> -p <package_name>:<path>
```

---

## 完整示例

### 示例 1：生成 SA TypeScript 代码

**输入 IDL 文件** (`test_service.idl`):
```idl
package OHOS.System;

interface ITestService {
    int GetData([in] int type, [out] int result);
    void SetData([in] int data);
}
```

**命令**：
```bash
idl -gen-ts -d ./output test_service.idl
```

**生成的文件**：
```
output/
├── i_test_service.d.ts
├── i_test_service_proxy.ts
└── i_test_service_stub.ts
```

### 示例 2：生成 HDI C++ 代码

**输入 IDL 文件** (`camera.idl`):
```idl
package OHOS.Hardware.Camera;

interface ICameraDevice {
    int OpenCamera([in] int cameraId, [out] int fd);
    void CloseCamera([in] int fd);
}
```

**命令**：
```bash
idl -gen-cpp -d ./output camera.idl
```

**生成的文件**：
```
output/
├── i_camera_device.h
├── camera_device_proxy.h
├── camera_device_stub.h
├── camera_device_proxy.cpp
└── camera_device_stub.cpp
```

---

## 输入输出格式

### 支持的输入文件格式

| 文件扩展 | 说明 |
|---------|------|
| `.idl` | IDL 接口定义文件（主要格式） |
| `.ts` | TypeScript IDL 文件（部分支持） |

### 输出文件规范

**文件命名规则**（C++/C/TS）：
- 接口文件：`<package>_<interface_name>.h` 或 `i_<interface_name>.d.ts`
- 代理文件：`<interface_name>_proxy.h` 或 `<interface_name>_proxy.ts`
- 桩文件：`<interface_name>_stub.h` 或 `<interface_name>_stub.ts`

**目录结构**：
```
<output_dir>/
├── OHOS/                    # 包目录
│   └── System/            # 子包目录
│       └── ITestService/    # 接口目录
│           ├── i_test_service.d.ts
│           ├── test_service_proxy.ts
│           └── test_service_stub.ts
```

---

## 错误码与退出状态

### 退出码

| 退出码 | 说明 | 典型原因 |
|--------|------|---------|
| 0 | 成功 | 正常完成 |
| -1 | 命令行解析失败 | 选项错误或冲突 |
| -1 | 预处理失败 | 文件读取或宏展开失败 |
| -1 | 解析失败 | IDL 语法错误 |
| -1 | 代码生成失败 | 输出目录不存在或权限问题 |

### 错误输出格式

```
[<file_name>:<line_number>] error:<error_message>
```

**示例**：
```
[test.idl:15] error:unexpected token '}', expected ';'
[test.idl:20] error:type 'UnknownType' not found
```

---

## 与构建系统集成

### GN 构建脚本集成

**在 BUILD.gn 中使用 IDL 工具**：

```gn
# 声明 IDL 文件
idl_sources = [ "interfaces/test_service.idl" ]

# 调用 IDL 工具生成代码
action("gen_idl_code") {
  script = "$root_build_dir/out/idl -gen-ts -d {{output_gen_dir}} {{idl_sources}}"
  outputs = [
    "{{output_gen_dir}}/test_service.d.ts",
    "{{output_gen_dir}}/test_service_proxy.ts",
    "{{output_gen_dir}}/test_service_stub.ts",
  ]
}

# 依赖生成的代码
ohos_shared_library("test_service") {
  sources = get_target_outputs(":gen_idl_code")
  deps = [
    "//foundation/ability/samgr:zsamgr_client",
  ]
}
```

---

## 关键结论

1. **纯命令行工具** - IDL Tool 是命令行工具，无运行时 API
2. **多语言支持** - 支持 C/C++/Java/TS/Rust 等多种语言
3. **双模式支持** - HDI（硬件接口）和 SA（系统能力接口）
4. **元数据支持** - 仅 SA 模式支持运行时类型信息
5. **灵活的输出** - 支持目录结构、包路径映射等

---

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览与支持的后端
- [01_Directory_Structure.md](01_Directory_Structure.md) - 模块职责
- [02_Architecture.md](02_Architecture.md) - 架构设计
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建集成

---

**最后更新**: 2026-02-06

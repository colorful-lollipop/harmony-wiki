# 依赖关系与使用

本文档说明 cangjie_tools 在 OpenHarmony 中的依赖关系和使用情况。

---

## 直接依赖者

### 搜索结果

**结论：未发现其他 OpenHarmony 模块直接依赖 `cangjie_tools`**

**搜索方式**：
- 在 `../oh/` 目录下搜索包含 `"third_party/cangjie_tools"` 的 BUILD.gn 文件
- 在 `../oh/` 目录下搜索包含 `"@ohos/cangjie_tools"` 的 bundle.json 文件
- 搜索包含 `"cangjie_tools"` 的依赖声明

**搜索结果**：
```
在 ../oh/ 目录下：
- 未找到引用 "third_party/cangjie_tools" 的 BUILD.gn 文件
- 未找到引用 "@ohos/cangjie_tools" 的 bundle.json 文件
- 未找到引用 "cangjie_tools" 的依赖声明
```

### 依赖者统计

| 指标 | 值 |
|------|-----|
| **直接依赖者数量** | 0 |
| **间接依赖者** | 未统计 |

---

## 使用方式

### 作为独立开发工具

cangjie_tools 在 OpenHarmony 中**作为独立开发工具使用**，不被其他 OH 模块运行时依赖。

**使用方式**：
- **命令行工具**：开发者通过命令行直接使用 cjpm、cjfmt、cjlint 等工具
- **IDE 集成**：LSP 集成到 DevEco Studio，提供智能提示、导航、诊断等 IDE 功能
- **跨语言互操作**：HLE 工具在开发时生成 ArkTS 与 Cangjie 的互操作代码

---

## 主要使用场景

### 1. Cangjie 项目开发

**涉及工具**：cjpm, cjfmt, cjlint, LSP

**使用流程**：
1. **项目初始化**：使用 cjpm 初始化项目
   ```bash
   cjpm init my-cangjie-app
   ```

2. **依赖管理**：使用 cjpm 管理项目依赖
   ```bash
   cjpm fetch
   cjpm update
   ```

3. **代码编写**：在 DevEco Studio 中编写 Cangjie 代码
   - LSP 提供智能提示和代码补全
   - LSP 提供符号定义和引用跳转
   - LSP 提供语法和语义诊断

4. **代码格式化**：使用 cjfmt 格式化代码
   ```bash
   cjfmt -i src/main.cj
   ```

5. **代码检查**：使用 cjlint 检查代码质量
   ```bash
   cjlint src/
   ```

6. **编译构建**：使用 cjpm 编译项目
   ```bash
   cjpm build
   ```

---

### 2. DevEco Studio 集成

**涉及工具**：LSP（cangjie-language-server）

**使用场景**：
- **智能提示**：代码补全、参数提示、成员列表
- **导航功能**：定义跳转、引用查找、类型悬停
- **诊断功能**：语法错误、类型错误、警告提示
- **重构功能**：重命名符号、提取函数等
- **跨语言支持**：Cangjie ↔ ArkTS 符号跳转

**集成机制**：
1. DevEco Studio 通过 LSP 协议与 cangjie-language-server 通信
2. LSP 检测 DevEco 环境（`GetIsDeveco()`）
3. 在 DevEco 模式下，LSP 自动设置 OHOS 默认目标（`aarch64-linux-ohos`）
4. LSP 加载 OHOS 特定的 CJD 索引（`ohosCjdPath`）

**关键文件**：
- `cangjie-language-server/src/languageserver/CompilerCangjieProject.cpp`
- `cangjie-language-server/src/languageserver/LSPCompilerInstance.cpp`

---

### 3. ArkTS 互操作

**涉及工具**：HLE（HyperLang Extension）

**使用场景**：
- **ArkTS 调用 Cangjie**：生成 ArkTS 调用 Cangjie API 的桥接代码
- **Cangjie 调用 ArkTS**：生成 Cangjie 调用 ArkTS API 的桥接代码
- **类型转换**：自动处理跨语言类型转换
- **调用约定**：自动处理跨语言调用约定

**使用流程**：
1. **定义 ArkTS 接口**：编写 .d.ts 或 .d.ets 文件
   ```typescript
   // api.d.ts
   export function add(a: number, b: number): number;
   ```

2. **生成互操作代码**：使用 HLE 工具生成桥接代码
   ```bash
   hle -i api.d.ts -o output/
   ```

3. **生成的代码**：
   - Cangjie 互操作代码（.cj 文件）
   - OHOS 构建（BUILD.gn）配置
   - ArkTS 文件信息 JSON

4. **自动导入**：HLE 自动生成 OHOS 特定导入
   ```cangjie
   import ohos.ark_interop.*
   import ohos.ark_interop_helper.*
   import ohos.base.*
   ```

5. **集成到项目**：将生成的代码集成到 OpenHarmony 项目
   ```gn
   # BUILD.gn
   import("//build/ohos.gni")

   ohos_shared_library("my_interop") {
     sources = [ "output/my_interop.cj" ]
     deps = [ ":ark_interop" ]
   }
   ```

**关键文件**：
- `hyperlangExtension/src/entry/hle.cj`
- `hyperlangExtension/src/tool/create_ark_api_call_async.cj`
- `hyperlangExtension/src/tool/create_build_gn.cj`

---

### 4. 代码质量保障

**涉及工具**：cjfmt, cjlint

**使用场景**：
- **代码格式化**：使用 cjfmt 自动格式化代码，保持代码风格一致
- **代码检查**：使用 cjlint 静态分析代码，发现潜在问题和违反规范的地方
- **CI/CD 集成**：在 CI/CD 流程中集成 cjfmt 和 cjlint，自动检查代码质量

**使用示例**：
```bash
# 格式化代码
cjfmt -i src/

# 检查代码
cjlint src/ --config .cjlintrc.json

# 在 CI/CD 中使用
cjfmt --check src/
cjlint src/ --error
```

---

### 5. 测试与调试

**涉及工具**：cjcov, cjtrace-recover

**使用场景**：
- **代码覆盖率分析**：使用 cjcov 生成测试覆盖率报告，帮助开发者提高测试完整性
- **异常堆栈恢复**：使用 cjtrace-recover 恢复混淆后的异常堆栈信息，用于问题定位和根因分析

**使用示例**：
```bash
# 生成覆盖率报告
cjcov --gcov output.gcov --output coverage.html

# 恢复异常堆栈
cjtrace-recover --obfuscated-stack "a.b.c@123" --symbols-path /path/to/symbols
```

---

## 依赖图

### cangjie_tools 在 OpenHarmony 中的位置

```
OpenHarmony 系统
│
├── 开发工具链（独立使用）
│   ├── cjpm - 项目管理
│   ├── cjfmt - 代码格式化
│   ├── cjlint - 代码检查
│   ├── cjcov - 覆盖率分析
│   ├── cjtrace-recover - 堆栈恢复
│   └── hle - ArkTS 互操作
│
├── DevEco Studio（IDE 集成）
│   └── LSP（cangjie-language-server）
│       ├── 智能提示
│       ├── 导航功能
│       ├── 诊断功能
│       └── 重构功能
│
└── OpenHarmony 应用（运行时依赖）
    └── （不依赖 cangjie_tools）
```

### 工具依赖关系

```
cangjie_tools
│
├── cjpm (Cangjie)
│   └── 依赖 stdx 库
│       ├── stdx.logger
│       ├── stdx.log
│       ├── stdx.encoding.json
│       ├── stdx.encoding.url
│       └── stdx.serialization
│
├── cjfmt (C++17)
│   └── 依赖 Cangjie SDK
│
├── cjlint (C++17)
│   └── 依赖 JSON for Modern C++
│
├── cjcov (Cangjie)
│   └── 依赖 stdx 库
│
├── lsp (C++17)
│   ├── 依赖 flatbuffers
│   ├── 依赖 JSON for Modern C++
│   └── 依赖 SQLite
│
├── cjtrace-recover (C++17 + Cangjie)
│   ├── 依赖 demangler
│   └── 依赖 Cangjie SDK
│
└── hle (Cangjie)
    └── 依赖 stdx 库
```

### OpenHarmony 三方库依赖

根据 `bundle.json`：

```json
"deps": {
    "components": [
        "flatbuffers",
        "json",
        "sqlite"
    ]
}
```

**依赖说明**：

| OH 组件 | 用途 | 被 cangjie_tools 哪些工具依赖 |
|---------|------|-------------------------|
| flatbuffers | 序列化/反序列化 | LSP（索引数据序列化） |
| json | JSON 解析 | LSP, cjlint（消息解析） |
| sqlite | 数据库 | LSP（索引存储） |

---

## 使用场景汇总

| 场景 | 使用方式 | 涉及工具 | 使用者 |
|------|---------|----------|--------|
| 项目初始化和管理 | 命令行 | cjpm | Cangjie 开发者 |
| 代码编写 | IDE 集成 | LSP | Cangjie 开发者 |
| 代码格式化 | 命令行 / CI/CD | cjfmt | Cangjie 开发者 |
| 代码检查 | 命令行 / CI/CD | cjlint | Cangjie 开发者 |
| 测试覆盖率 | 命令行 | cjcov | Cangjie 开发者 |
| 异常堆栈恢复 | 命令行 | cjtrace-recover | Cangjie 开发者 |
| ArkTS 互操作 | 构建工具 | hle | Cangjie + ArkTS 开发者 |
| DevEco 集成 | IDE 插件 | LSP | DevEco Studio 用户 |

---

## 典型使用案例

### 案例 1：使用 cjpm 创建 Cangjie 项目

```bash
# 1. 初始化项目
cjpm init my-oh-app

# 2. 进入项目目录
cd my-oh-app

# 3. 获取依赖
cjpm fetch

# 4. 编译项目
cjpm build

# 5. 运行测试
cjpm test

# 6. 格式化代码
cjfmt -i src/

# 7. 检查代码
cjlint src/
```

---

### 案例 2：在 DevEco Studio 中开发 Cangjie 应用

1. **安装 DevEco Studio**：包含 cangjie-tools 和 LSP 支持
2. **创建项目**：选择 Cangjie 项目模板
3. **编写代码**：
   - LSP 提供智能提示和代码补全
   - Ctrl+点击跳转到符号定义
   - 实时显示语法和类型错误
4. **运行调试**：在 DevEco Studio 中运行和调试
5. **格式化代码**：右键菜单选择 "Format Code"（调用 cjfmt）
6. **代码检查**：右键菜单选择 "Inspect Code"（调用 cjlint）

---

### 案例 3：Cangjie 调用 ArkTS API

1. **定义 ArkTS 接口**：
   ```typescript
   // api.d.ts
   export interface HiLog {
     info(tag: string, message: string): void;
   }
   ```

2. **生成互操作代码**：
   ```bash
   hle -i api.d.ts -o output/ --target ohos-aarch64
   ```

3. **生成的 Cangjie 代码**（简化）：
   ```cangjie
   import ohos.ark_interop.*
   import ohos.ark_interop_helper.*

   public class HiLog {
       public static func info(tag: String, message: String): Unit {
           // 生成的互操作代码
       }
   }
   ```

4. **在 Cangjie 中使用**：
   ```cangjie
   import my_interop.HiLog

   public func main() {
       HiLog.info("MyTag", "Hello from Cangjie!")
   }
   ```

5. **集成到 OH 项目**：
   - 将生成的代码添加到 OpenHarmony 项目
   - 在 BUILD.gn 中引用生成的模块

---

## 常见问题

### Q1: 为什么没有其他模块依赖 cangjie_tools？

**A**：cangjie_tools 是**开发工具链**，不是运行时库。它的主要用途是：

1. 帮助开发者编写、构建、测试 Cangjie 代码
2. 为 DevEco Studio 提供 LSP 支持
3. 生成跨语言互操作代码

这些功能在开发时使用，不作为运行时依赖被其他 OH 模块链接。

### Q2: cangjie_tools 如何集成到 OpenHarmony 系统？

**A**：cangjie_tools 通过以下方式集成：

1. **包管理**：通过 OpenHarmony 包管理系统分发
2. **DevEco Studio 集成**：LSP 作为插件集成到 DevEco Studio
3. **构建工具**：HLE 作为构建工具集成到 DevEco Studio 的构建流程

### Q3: 如何使用 cangjie_tools 开发 OpenHarmony 应用？

**A**：使用流程：

1. **安装工具链**：安装 DevEco Studio（包含 cangjie-tools）
2. **创建项目**：在 DevEco Studio 中创建 Cangjie 项目
3. **开发**：使用 LSP 提供的智能提示和导航功能
4. **构建**：使用 cjpm 或 DevEco Studio 的构建按钮
5. **测试**：使用 cjpm test 或 DevEco Studio 的测试运行器
6. **调试**：使用 DevEco Studio 的调试器

### Q4: 如何在 CI/CD 中使用 cangjie_tools？

**A**：CI/CD 示例：

```yaml
# GitHub Actions 示例
name: Cangjie CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install Cangjie Tools
        run: |
          # 安装 cangjie-tools
          wget https://cangjie-lang.cn/packages/cangjie-tools-linux.tar.gz
          tar -xzf cangjie-tools-linux.tar.gz
          export PATH=$PATH:$PWD/cangjie-tools/bin
      
      - name: Install Dependencies
        run: |
          cjpm fetch
      
      - name: Check Code Format
        run: |
          cjfmt --check src/
      
      - name: Lint Code
        run: |
          cjlint src/ --error
      
      - name: Build
        run: |
          cjpm build --release
      
      - name: Test
        run: |
          cjpm test
      
      - name: Generate Coverage
        run: |
          cjcov --gcov output.gcov --output coverage.html
```

---

## 总结

### 核心要点

1. **无运行时依赖**：cangjie_tools 作为独立开发工具使用，不被其他 OH 模块依赖
2. **多场景支持**：覆盖项目开发、代码质量保障、IDE 集成、跨语言互操作等多个场景
3. **DevEco 深度集成**：LSP 深度集成 DevEco Studio，提供完整的 IDE 支持
4. **ArkTS 互操作**：HLE 工具提供完整的跨语言互操作支持

### 使用建议

1. **开发者**：使用 DevEco Studio 获得最佳开发体验
2. **CI/CD**：集成 cjfmt、cjlint、cjcov 到 CI/CD 流程
3. **跨语言开发**：使用 HLE 工具生成互操作代码
4. **调试支持**：使用 cjtrace-recover 恢复混淆后的堆栈信息

---

## 参考文档

- [README.md](./README.md) - 库概览和 OH 适配概述
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [../README.md](../README.md) - 项目主文档
- [../cjpm/doc/developer_guide.md](../cjpm/doc/developer_guide.md) - cjpm 使用指南
- [../cjfmt/doc/developer_guide.md](../cjfmt/doc/developer_guide.md) - cjfmt 使用指南

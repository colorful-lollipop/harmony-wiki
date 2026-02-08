# 常见问题解答（FAQ）

> **文档版本**: 1.0  
> **最后更新**: 2026-02-07

---

## 目录

- [构建问题](#构建问题)
- [编译问题](#编译问题)
- [运行时问题](#运行时问题)
- [调试问题](#调试问题)
- [工具使用问题](#工具使用问题)

---

## 构建问题

### Q1: 如何构建 ets_frontend？

**回答**：

使用 GN + Ninja 构建系统。在 OpenHarmony 源码根目录执行：

```bash
./build.sh --product-name <产品名> --build-target ets_frontend_build
```

示例（针对 rk3568 开发板）：

```bash
./build.sh --product-name rk3568 --build-target ets_frontend_build
```

**详细步骤**：

1. 确保已安装依赖：
   - GN 构建工具
   - Ninja 构建工具
   - C++17 兼容的编译器（GCC/Clang）

2. 设置环境变量（可选）：
   ```bash
   export ARK_ROOT=<arkcompiler目录>
   ```

3. 执行构建：
   ```bash
   cd /path/to/source/root
   python3 ./build.py --product-name rk3568 --build-target ets_frontend_build
   ```

**相关文档**：[构建系统](../04_Build_System.md)

---

### Q2: 构建失败，提示 "ninja: build stopped: subcommand failed"

**回答**：

此错误通常是底层构建工具问题。请按以下步骤排查：

1. **检查 ninja 是否可用**：
   ```bash
   ninja --version
   ```

2. **清理并重新构建**：
   ```bash
   rm -rf out/<product>/clang_x64/arkcompiler/ets_frontend/
   ./build.sh --product-name <产品名> --build-target ets_frontend_build --clean-build
   ```

3. **检查磁盘空间**：
   ```bash
   df -h
   ```

4. **检查内存**（构建大型项目需要足够内存）：
   ```bash
   free -h
   ```

5. **并行构建限制**（内存不足时尝试）：
   ```bash
   ./build.sh --product-name <产品名> --build-target ets_frontend_build -j 4
   ```

---

### Q3: 找不到 GN 或 Ninja 工具

**回答**：

1. **安装 GN**：
   ```bash
   # 从源码编译
   git clone https://gn.googlesource.com/gn
   cd gn
   python3 build/gen.py
   ninja -C out
   sudo cp out/gn /usr/local/bin/
   ```

2. **安装 Ninja**：
   ```bash
   # Linux (Ubuntu/Debian)
   sudo apt-get install ninja-build

   # macOS
   brew install ninja

   # 从源码编译
   git clone https://github.com/ninja-build/ninja.git
   cd ninja
   ./configure.py --bootstrap
   sudo cp ninja /usr/local/bin/
   ```

3. **添加到 PATH**：
   ```bash
   export PATH=$PATH:/path/to/gn:/path/to/ninja
   ```

---

### Q4: 编译选项 `is_debug` 和 `is_release` 的区别？

**回答**：

| 选项 | 调试信息 | 优化级别 | 用途 |
|------|----------|----------|------|
| `is_debug=true` | 完整 DWARF 调试信息 | -Og | 开发调试 |
| `is_debug=false` | 仅符号表 | -O2/-O3 | 发布构建 |

**配置文件**：`ark_config.gni`

```gni
# 调试构建（默认）
is_debug = true

# 发布构建
is_debug = false
```

**影响**：
- 调试构建：产物更大，运行更慢，但可调试
- 发布构建：产物更小，运行更快，不可调试

---

## 编译问题

### Q5: 编译 ETS 文件时提示 "Unknown extension"

**回答**：

此错误表示编译器无法识别输入文件的扩展名。

**错误示例**：
```
Error: Unknown extension '.txt'
```

**解决方案**：

1. **使用 `--extension` 指定文件类型**：
   ```bash
   ./es2abc --extension ets myfile.txt
   ```

2. **确保使用正确的扩展名**：
   - `.ets` - ArkTS/ETS 文件
   - `.ts` - TypeScript 文件
   - `.js` - JavaScript 文件
   - `.as` - ActionScript 文件

3. **检查文件扩展名**（排除隐藏扩展名）：
   ```bash
   ls -la myfile  # 确认不是 myfile.ets.txt
   ```

**相关文档**：[命令行接口](../02_CLI_Reference.md)

---

### Q6: 编译大文件时崩溃/内存不足

**回答**：

ets_frontend 默认读取整个源文件到内存。超大文件可能导致内存耗尽。

**解决方案**：

1. **增加系统内存限制**（临时）：
   ```bash
   ulimit -v unlimited
   ```

2. **使用 64 位系统**（确保编译器是 64 位版本）

3. **分割大文件**：
   - 将大型模块拆分为多个小文件
   - 使用模块导入（import/export）

4. **检查文件大小**：
   ```bash
   ls -lh your_large_file.ets
   # 如果超过 100MB，考虑分割
   ```

**缓解措施**：

- 编译器使用 ArenaAllocator 管理内存（`eheap.cpp`）
- OOM 时会强制终止（`OOMAction()`）
- 建议保持单个源文件 < 50MB

---

### Q7: 深度嵌套代码导致编译失败

**回答**：

过深的嵌套（如函数嵌套、类嵌套）可能触发递归深度限制。

**错误示例**：
```
Error: DEEP_NESTING - Recursion depth exceeded
```

**解决方案**：

1. **重构代码，减少嵌套深度**：
   ```typescript
   // 避免过度嵌套
   function processData(data: Data): Result {
       if (data.isValid) {  // 扁平化逻辑
           return handleValid(data);
       }
       return handleInvalid(data);
   }
   ```

2. **提取为独立函数**：
   ```typescript
   // 原始：过度嵌套
   function complex() {
       if (a) {
           if (b) {
               if (c) {
                   // 深嵌套代码
               }
           }
       }
   }

   // 改进：提前返回
   function complex() {
       if (!a) return;
       if (!b) return;
       if (!c) return;
       // 主要逻辑
   }
   ```

3. **当前限制**：
   - 最大递归深度：`MAX_RECURSION_DEPTH = 5120`（`recursiveGuard.h:21`）
   - 超过限制会报错并跳过剩余代码

---

### Q8: --opt-level 选项的有效值？

**回答**：

`--opt-level` 控制编译优化级别：

| 值 | 优化级别 | 说明 |
|----|----------|------|
| 0 | 无优化 | 最快编译速度，最大产物 |
| 1 | 基本优化 | 适中速度和产物 |
| 2 | 完全优化 | 最慢编译速度，最小产物（默认） |

**使用示例**：
```bash
# 无优化（调试用）
./es2abc --opt-level 0 input.ets -o output.abc

# 基本优化
./es2abc --opt-level 1 input.ets -o output.abc

# 完全优化
./es2abc --opt-level 2 input.ets -o output.abc
```

**注意事项**：
- 调试时建议使用 `--opt-level 0`
- 发布时建议使用 `--opt-level 2`
- AOT 编译时优化效果更明显

---

## 运行时问题

### Q9: 编译成功但运行时提示 "Failed to load module"

**回答**：

此问题通常与模块路径或依赖有关。

**排查步骤**：

1. **检查模块路径**：
   ```bash
   ./es2abc --module-path ./modules input.ets
   ```

2. **验证导入路径**：
   ```typescript
   // 错误：路径错误
   import { func } from './nonexistent';  // 文件不存在

   // 正确：相对路径
   import { func } from './localModule';

   // 正确：绝对路径（从模块根目录）
   import { func } from '@module/feature';
   ```

3. **检查模块导出**：
   ```typescript
   // moduleA.ets
   export function myFunc(): void { }  // 确保已导出

   // main.ets
   import { myFunc } from './moduleA';  // 导入
   ```

4. **查看详细错误**：
   ```bash
   ./es2abc --debug-info input.ets
   ```

---

### Q10: 字节码文件 (.abc) 无法在 ARK Runtime 执行

**回答**：

1. **确保字节码版本匹配**：
   ```bash
   # 查看字节码版本
   ./es2abc --dump-debug-info input.ets

   # 对比运行时支持的版本
   ```

2. **检查目标 API 版本**：
   ```bash
   ./es2abc --target-api-version 10 input.ets
   ```

3. **验证字节码完整性**：
   ```bash
   # 使用工具检查字节码
   ./arkcompiler_ets_runtime --verify-abc input.abc
   ```

4. **重新编译**：
   ```bash
   rm -f input.abc
   ./es2abc input.ets
   ```

---

## 调试问题

### Q11: 如何查看生成的 AST？

**回答**：

使用 `--dump-ast` 选项：

```bash
./es2abc --dump-ast input.ets

# 输出到文件
./es2abc --dump-ast input.ets -o ast_output.txt
```

**输出示例**：
```
Program:
  - FunctionDeclaration: main
    - Parameter: args
    - BlockStatement:
      - ExpressionStatement:
        - CallExpression:
          - Identifier: print
          - StringLiteral: "Hello"
```

**用途**：
- 理解代码如何被解析
- 调试语法问题
- 学习编译器工作原理

---

### Q12: 如何查看生成的字节码？

**回答**：

使用 `--dump-assembly` 选项：

```bash
./es2abc --dump-assembly input.ets

# 输出到文件
./es2abc --dump-assembly input.ets -o bytecode.txt
```

**输出示例**：
```
; Function: main
0: LDA 0
1: STARG 0
2: LDSTR "Hello"
3: PRINT
4: RETURN
```

**字节码指令说明**：
- `LDx` - 加载指令
- `STx` - 存储指令
- `CALL` - 函数调用
- `RETURN` - 返回

---

### Q13: 如何调试编译器自身？

**回答**：

1. **使用调试构建**：
   ```bash
   # 使用 is_debug=true 构建
   ./build.sh --product-name <产品> --build-target ets_frontend_build
   ```

2. **启用详细日志**：
   ```bash
   ./es2abc --debugger-evaluate-expression "verbose" input.ets
   ```

3. **GDB 调试**：
   ```bash
   gdb ./es2abc
   (gdb) set args --input input.ets
   (gdb) run
   (gdb) bt  # 崩溃时查看调用栈
   ```

4. **打印 AST/IR**：
   ```bash
   ./es2abc --dump-ast --dump-ir input.ets
   ```

---

### Q14: 编译错误信息不清晰，如何获取更多调试信息？

**回答**：

1. **启用详细模式**：
   ```bash
   ./es2abc --verbose input.ets
   ```

2. **输出调试信息**：
   ```bash
   ./es2abc --debug-info input.ets

   # 或输出到日志文件
   ./es2abc --debug-info input.ets 2>&1 | tee debug.log
   ```

3. **分步编译**：
   ```bash
   # 仅解析
   ./es2abc --parse-only input.ets

   # 解析 + 类型检查
   ./es2abc --type-check-only input.ets
   ```

4. **检查诊断输出**：
   ```bash
   ./es2abc input.ets 2>&1 | grep -i error
   ./es2abc input.ets 2>&1 | grep -i warning
   ```

---

## 工具使用问题

### Q15: es2abc 和 ets2panda 的区别？

**回答**：

| 工具 | 路径 | 用途 | 支持语言 |
|------|------|------|----------|
| `es2abc` | `es2panda/aot/` | 传统 JS/TS 编译器 | JavaScript, TypeScript, ActionScript |
| `ets2panda` | `ets2panda/aot/` | 现代 ETS 编译器 | Enhanced TypeScript (ArkTS) |

**使用建议**：
- 新项目使用 `ets2panda`
- 旧 JS/TS 项目继续使用 `es2abc`
- 两者都生成兼容的 `.abc` 字节码

**命令对照**：
```bash
# es2abc (传统)
es2abc --extension ts input.ts -o output.abc

# ets2panda (现代)
ets2panda --extension ets input.ets -o output.abc
```

---

### Q16: 如何查看所有可用的命令行选项？

**回答**：

1. **使用 `--help`**：
   ```bash
   ./es2abc --help
   ./ets2panda --help
   ```

2. **查看选项配置文件**：
   ```bash
   cat ets2panda/util/options.yaml
   ```

3. **常见选项列表**：

   | 选项 | 描述 |
   |------|------|
   | `--extension` | 指定输入文件类型 |
   | `--module` | 使用 ES 模块模式 |
   | `--opt-level` | 优化级别 (0/1/2) |
   | `--output` | 指定输出文件 |
   | `--thread` | 编译线程数 |
   | `--dump-ast` | 输出 AST |
   | `--dump-assembly` | 输出字节码汇编 |
   | `--dump-debug-info` | 输出调试信息 |
   | `--parse-only` | 仅解析，不生成字节码 |

---

### Q17: 如何指定模块搜索路径？

**回答**：

使用 `--module-path` 或 `--module` 选项：

```bash
# 指定模块根目录
./ets2panda --module-path ./src ./main.ets

# 多路径搜索
./ets2panda --module-path ./src:./lib ./main.ets

# 设置模块根
./ets2panda --base-dir ./src ./main.ets
```

**模块路径管理**（证据：`ets2panda/util/importPathManager.cpp:183`）：

- 支持相对路径（`./`, `../`）
- 支持绝对路径
- 支持符号链接（会自动规范化）

---

### Q18: 编译时如何指定目标 API 版本？

**回答**：

```bash
# 指定目标 API 版本
./es2abc --target-api-version 10 input.ets

# 查看当前版本
./es2abc --version
```

**API 版本兼容性**：
- 确保字节码版本与运行时匹配
- 旧版本字节码可在新运行时兼容执行
- 新版本字节码可能不兼容旧运行时

---

## 性能问题

### Q19: 编译速度太慢，如何优化？

**回答**：

1. **减少优化级别**（开发时）：
   ```bash
   ./es2abc --opt-level 0 input.ets
   ```

2. **增加线程数**：
   ```bash
   ./es2abc --thread 8 input.ets
   ```

3. **增量编译**：
   ```bash
   # 只重新编译修改的文件
   ./build.sh --incremental
   ```

4. **使用缓存**：
   ```bash
   # 启用构建缓存
   export CCACHE_DIR=/path/to/cache
   ```

5. **排除不需要的文件**：
   ```bash
   # 不编译测试文件
   ./es2abc --exclude-pattern '**/*.test.ets' ./src
   ```

---

### Q20: 如何分析编译性能瓶颈？

**回答**：

1. **启用性能分析**：
   ```bash
   ./es2abc --profile input.ets
   ```

2. **查看时间统计**：
   ```bash
   ./es2abc --dump-size-stat input.ets
   ```

3. **分析各阶段耗时**：
   ```
   Parsing:          100ms
   Type Checking:    500ms
   Code Generation:  200ms
   Bytecode Emit:    50ms
   Total:           850ms
   ```

4. **针对性优化**：
   - 词法/语法解析慢 → 检查源文件复杂度
   - 类型检查慢 → 简化类型定义
   - 字节码生成慢 → 减少内联代码

---

## 其他问题

### Q21: 如何报告编译器 Bug？

**回答**：

1. **收集信息**：
   ```bash
   # 编译器版本
   ./es2abc --version

   # 系统信息
   uname -a
   g++ --version

   # 完整命令和环境
   env > env.txt
   ./es2abc --verbose --debug-info input.ets 2>&1 | tee bug_report.txt
   ```

2. **准备最小复现用例**：
   ```typescript
   // minimal_test.ets
   // 尽可能小的导致问题的代码
   ```

3. **提交问题**：
   - GitHub Issues: https://gitee.com/openharmony/arkcompiler_ets_frontend/issues
   - 包含上述所有信息
   - 说明预期行为和实际行为

---

### Q22: 如何参与贡献代码？

**回答**：

1. **阅读贡献指南**：
   ```bash
   cat CONTRIBUTING.md
   ```

2. **遵循代码规范**：
   - C++17 标准
   - `-Wall -Wextra -Werror` 编译选项
   - 添加测试用例

3. **提交流程**：
   ```bash
   git checkout -b feature/my-feature
   # 修改代码
   git add .
   git commit -m "feat: add new feature"
   git push origin feature/my-feature
   # 创建 Merge Request
   ```

4. **代码审查**：
   - 响应审查意见
   - 更新代码
   - 通过 CI/CD 检查

---

## 相关文档链接

- [概览](../00_Overview.md)
- [命令行接口](../02_CLI_Reference.md)
- [构建系统](../04_Build_System.md)
- [安全评审](../05_Security_Review.md)
- [架构详解](../07_Architecture.md)

---

## 贡献者

如果您有更多问题或解决方案，欢迎贡献到本 FAQ：

1. 编辑 `appendix/FAQ.md`
2. 添加新的 Q&A
3. 提交 Pull Request

---

> **提示**：本 FAQ 会持续更新。建议在提问前先搜索本页内容。

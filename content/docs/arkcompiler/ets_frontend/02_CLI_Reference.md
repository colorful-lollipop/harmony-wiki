# es2abc 命令行接口参考

## 使用方式

```bash
./es2abc [options] input_file
```

## 选项说明

### 输入控制

| 选项 | 描述 | 值范围 | 默认值 |
|------|------|--------|--------|
| `--extension <type>` | 指定输入文件类型 | `js`, `ts`, `as` | 自动推断 |
| `--module` | 按 ECMAScript 模块编译 | - | 单文件模式 |
| `--parse-only` | 仅解析，不生成字节码 | - | false |

### 输出控制

| 选项 | 描述 | 默认值 |
|------|------|--------|
| `--output <path>` | 指定输出文件路径 | `input.abc` |
| `--dump-assembly` | 输出汇编格式 | false |
| `--dump-ast` | 打印 AST 结构 | false |
| `--dump-literal-buffer` | 打印字面量缓冲区 | false |
| `--dump-size-stat` | 显示字节码统计 | false |
| `--dump-debug-info` | 打印调试信息 | false |
| `--debug-info` | 提供调试信息 | false |

### 编译器选项

| 选项 | 描述 | 值范围 | 默认值 |
|------|------|--------|--------|
| `--opt-level <level>` | 编译优化级别 | `0`, `1`, `2` | `0` |
| `--thread <count>` | 生成字节码的线程数 | `0`-机器最大线程 | `0` |
| `--debugger-evaluate-expression` | 调试器表达式求值 | - | false |

### 帮助选项

| 选项 | 描述 |
|------|------|
| `--help` | 显示帮助信息 |

## 使用示例

### 基本编译

```bash
# 编译 JavaScript 文件
./es2abc input.js

# 指定输出路径
./es2abc --output ./dist/input.abc input.js

# 编译 TypeScript 文件
./es2abc --extension ts input.ts
```

### 模块编译

```bash
# 按 ES 模块编译
./es2abc --module --output out.abc index.js

# 指定模块格式
./es2abc --module input.ts
```

### 调试与诊断

```bash
# 输出 AST 结构
./es2abc --dump-ast input.js

# 输出字节码汇编
./es2abc --dump-assembly input.js

# 显示字节码统计
./es2abc --dump-size-stat input.js

# 输出完整调试信息
./es2abc --dump-debug-info --debug-info input.js
```

### 优化选项

```bash
# 最高优化级别
./es2abc --opt-level 2 --output optimized.abc input.js

# 多线程编译
./es2abc --thread 4 --output out.abc large_file.js
```

## 输出格式

### 字节码文件 (.abc)

默认输出格式，包含：
- 常量池
- 字节码指令
- 调试信息（如果启用）

### 汇编格式 (--dump-assembly)

可读的字节码表示：

```
FUNC @<index> <name>
  LDLEX <slot>
  ...
END
```

### AST 格式 (--dump-ast)

JSON 格式的抽象语法树：

```json
{
  "type": "Program",
  "body": [...]
}
```

## 退出码

| 退出码 | 含义 |
|--------|------|
| 0 | 成功 |
| 1 | 编译错误 |
| 2 | 无效参数 |
| 3 | 文件不存在 |
| 4 | 系统错误 |

## 相关文档

- [编译产物与运行时](06_Build_Outputs.md)
- [架构详解](07_Architecture.md)

# 附录 A：API 参考

## N-API 导出接口

### NativeModule 模块

**模块加载**：

```javascript
const koala = require('./es2panda.node');
const NativeModule = koala.NativeModule;
```

### 上下文管理 API

| JS 方法 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `_CreateConfig` | argc: number, argv: string[] | Config | 创建编译配置 |
| `_DestroyContext` | contextPtr: bigint | void | 销毁上下文 |
| `_CreateContextFromFile` | config: Config, file: string | Context | 从文件创建上下文 |
| `_ProceedToState` | context: Context, state: State | void | 执行到指定状态 |
| `_GetContextState` | context: Context | State | 获取上下文状态 |

### 诊断 API

| JS 方法 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `_ContextErrorMessage` | context: Context | string | 获取错误消息 |
| `_GetAllErrorMessages` | context: Context | string[] | 获取所有错误 |
| `_ContextClearErrors` | context: Context | void | 清除错误 |

### AST 操作 API

| JS 方法 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `_CreateCallExpression` | context: Context | Node | 创建调用表达式 |
| `_UpdateCallExpression` | node: Node, ... | void | 更新调用表达式 |
| `_AstNodeChildren` | node: Node | Node[] | 获取子节点 |
| `_GetAstNodeType` | node: Node | string | 获取节点类型 |
| `_GetAstNodeText` | node: Node | string | 获取节点文本 |

### 内存管理 API

| JS 方法 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `_MemInitialize` | allocator: Allocator | void | 初始化内存管理 |
| `_MemFinalize` | void | void | 释放内存管理 |
| `_FreeCompilerPartMemory` | ptr: Pointer | void | 释放内存 |
| `_GetMemoryUsage` | void | number | 获取内存使用量 |

## ArkUI 插件接口

### uiTransform

**签名**：

```typescript
function uiTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program
```

**参数**：

| 参数名 | 类型 | 说明 |
|-------|------|------|
| program | arkts.Program | 输入程序 |
| projectConfig | ProjectConfig | 项目配置 |

**返回值**：

| 类型 | 说明 |
|------|------|
| arkts.Program | 转换后的程序 |

### uiSyntaxLinterTransform

**签名**：

```typescript
function uiSyntaxLinterTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program
```

**配置选项**：

```typescript
interface LinterConfig {
  level: 'error' | 'warn' | 'info';
  rules: string[];
  ignorePatterns: string[];
}
```

### interopTransform

**签名**：

```typescript
function interopTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program
```

## 编译器配置接口

### ProjectConfig

```typescript
interface ProjectConfig {
  // 路径配置
  projectPath: string;
  entryObj: Record<string, string>;
  cardObj: Record<string, string>;

  // 编译模式
  compileMode: 'jsbundle' | 'module' | 'shared';
  xtsMode: boolean;
  isPreview: boolean;

  // 系统配置
  runtimeOS: string;
  sdkInfo: string;

  // 优化选项
  obfuscate: boolean;
  optimize: boolean;
}
```

### 错误码定义

| 子系统码 | 名称 | 用途 |
|---------|------|------|
| 109 | ARKUI | ArkUI 相关错误 |
| 106 | LINTER | Linter 相关错误 |
| 105 | TSC | TypeScript 编译错误 |
| 101 | ABILITY | Ability 相关错误 |

## 相关文档

- [架构说明](02_Architecture.md)
- [编译器核心](04_Compiler_Core.md)
- [ArkUI 插件系统](05_ArkUI_Plugins.md)
- [Koala 包装器](06_Koala_Wrapper.md)

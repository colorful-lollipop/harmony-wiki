# ArkUI 插件系统

## 插件概述

ArkUI 插件系统是 Ace ETS2Bundle 的核心扩展机制，提供声明式 UI 语法的解析、转换和验证能力。

### 插件类型

| 插件类型 | 入口函数 | 主要职责 | 代码位置 |
|---------|---------|---------|----------|
| UI 插件 | `uiTransform()` | 声明式 UI 语法转换 | `ui-plugins/index.ts` |
| 语法检查插件 | `uiSyntaxLinterTransform()` | 编译时语法验证 | `ui-syntax-plugins/index.ts` |
| 互操作插件 | `interopTransform()` | JS/TS 互操作支持 | `interop-plugins/index.ts` |
| Memo 优化插件 | `unmemoizeTransform()` | 编译优化 | `memo-plugins/index.ts` |

## 插件架构

### 核心接口定义

**plugin-context.ts** 定义了插件的注册和调用机制：

```typescript
// 插件接口
export interface Plugins {
  name: string;                    // 插件名称
  afterNew?: PluginHandler;        // 新建后阶段
  parsed?: PluginHandler;          // 解析后阶段
  scopeInited?: PluginHandler;      // 作用域初始化后
  checked?: PluginHandler;          // 类型检查后阶段
  lowered?: PluginHandler;          // 降级后阶段
  asmGenerated?: PluginHandler;      // 汇编生成后
  binGenerated?: PluginHandler;      // 二进制生成后
  clean?: PluginHandler;            // 清理阶段
}

// 插件处理器类型
export type PluginHandler = (context: PluginContext) => void;
```

**AbstractVisitor** - 所有转换器的基类：

```typescript
export abstract class AbstractVisitor implements VisitorOptions {
  public isExternal: boolean;           // 是否为外部源文件
  public externalSourceName?: string;   // 外部源名称
  public program?: arkts.Program;       // 当前程序

  abstract visitor(node: arkts.AstNode): arkts.AstNode;

  init(): void {}                       // 初始化钩子
  reset(): void {}                      // 重置状态
  visitEachChild(node: arkts.AstNode): arkts.AstNode;
}
```

## UI 插件 (ui-plugins)

### 入口函数

```typescript
// ui-plugins/index.ts
export function uiTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program {
  // 创建插件上下文
  const context = new PluginContext(program, projectConfig);

  // 执行组件转换
  const componentTransformer = new ComponentTransformer();
  // 执行属性转换
  const propertyTranslators = new PropertyTranslators();
  // 返回转换后的程序
  return context.getArkTSProgram();
}
```

### 组件转换器

**component-transformer.ts** - 处理组件定义和实例化：

```typescript
export class ComponentTransformer extends AbstractVisitor {
  visitor(node: arkts.AstNode): arkts.AstNode {
    if (arkts.isClassDeclaration(node)) {
      // 处理 @Component 装饰器
      return this.visitComponent(node);
    }
    if (arkts.isCallExpression(node)) {
      // 处理组件实例化
      return this.visitComponentCreation(node);
    }
    return this.visitEachChild(node);
  }
}
```

### 属性翻译器

**property-translators/** - 处理各类属性装饰器：

| 翻译器 | 处理内容 |
|-------|---------|
| `builderParam.ts` | @BuilderParam |
| `computed.ts` | @Computed |
| `consume.ts` | @Consume |
| `consumer.ts` | @Consumer |
| `event.ts` | 事件处理 |
| `factory.ts` | 工厂模式 |
| `link.ts` | @Link |
| `local.ts` | @Local |
| `localStoragePropRef.ts` | @LocalStorageProp |

### Struct 翻译器

**struct-translators/** - 处理 Struct 组件：

```typescript
// struct-translators/index.ts
export class StructTransformer extends AbstractVisitor {
  visitor(node: arkts.AstNode): arkts.AstNode {
    if (arkts.isStructDeclaration(node)) {
      // Struct 定义处理
      return this.visitStruct(node);
    }
    return this.visitEachChild(node);
  }
}
```

## 语法检查插件 (ui-syntax-plugins)

### 入口函数

```typescript
// ui-syntax-plugins/index.ts
export function uiSyntaxLinterTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program {
  // 执行语法规则检查
  // 收集诊断信息
  // 返回带诊断的程序
}
```

### 语法规则系统

**rules/** - 60+ 条语法规则，按功能分类：

| 规则分类 | 数量 | 说明 |
|---------|------|------|
| 装饰器规则 | 15+ | @Component 等正确使用 |
| 属性规则 | 10+ | 属性类型和赋值 |
| 事件规则 | 8+ | 事件处理合法性 |
| 状态规则 | 12+ | @State 等使用规范 |
| 性能规则 | 10+ | 性能最佳实践 |

### 规则基类

```typescript
// rules/ui-syntax-rule.ts
export abstract class AbstractUISyntaxRule {
  protected context: UISyntaxRuleContext;
  protected level: UISyntaxRuleLevel;  // 'error' | 'warn' | 'none'

  public beforeTransform(): void {}
  public afterTransform(): void {}

  public parsed(node: arkts.AstNode): void;   // 解析阶段检查
  public checked(node: arkts.AstNode): void;  // 检查阶段检查

  protected report(options: UISyntaxRuleReportOptions): void;
}
```

## 互操作插件 (interop-plugins)

### 入口函数

```typescript
// interop-plugins/index.ts
export function interopTransform(
  program: arkts.Program,
  projectConfig: ProjectConfig
): arkts.Program {
  // 处理 JS/TS 与 ArkTS 互操作
  // 类型转换
  // 函数签名处理
}
```

### 主要转换器

| 转换器 | 职责 |
|-------|------|
| `decl_transformer.ts` | 声明转换 |
| `emit_transformer.ts` | 代码发射 |
| `function_transformer.ts` | 函数转换 |
| `signature_transformer.ts` | 签名转换 |

### 与 Native 交互

```typescript
// interop-plugins/types.ts
export type NativePointer = number;

export interface ArktsObject {
  readonly peer: NativePointer;  // 指向原生对象
}

// 动态加载互操作模块
import * as arkts from '@koalaui/libarkts';
```

## Memo 优化插件 (memo-plugins)

### 入口函数

```typescript
// memo-plugins/index.ts
export function unmemoizeTransform(
  program: arkts.Program
): arkts.Program {
  // 移除不必要的 memoization
  // 优化函数调用
  // 提升运行时性能
}
```

## AST 遍历机制

### ProgramVisitor

**program-visitor.ts** - 协调多个 visitor 遍历 AST：

```typescript
export class ProgramVisitor {
  private visitors: AbstractVisitor[] = [];

  public addVisitor(visitor: AbstractVisitor): void {
    this.visitors.push(visitor);
  }

  public visit(program: arkts.Program): arkts.Program {
    // 遍历所有源文件
    for (const sourceFile of program.getSourceFiles()) {
      // 应用每个 visitor
      for (const visitor of this.visitors) {
        this.visitSourceFile(sourceFile, visitor);
      }
    }
    return program;
  }

  private visitSourceFile(
    sourceFile: arkts.SourceFile,
    visitor: AbstractVisitor
  ): void {
    // 遍历 AST 节点
    visitor.visit(sourceFile.getAst());
  }
}
```

### 插件执行顺序

```
1. parsed 阶段
   └── 语法解析后执行
       └── UI 插件: 结构初步转换
       └── 语法检查: 解析时检查

2. checked 阶段
   └── 类型检查后执行
       └── UI 插件: 完整转换
       └── 语法检查: 类型相关检查

3. clean 阶段
   └── 最终清理
       └── 资源释放
       └── 状态重置
```

## 插件配置

### 项目配置集成

```typescript
// plugin-context.ts
export class PluginContext {
  private projectConfig: ProjectConfig | undefined;

  getProjectConfig(): ProjectConfig | undefined {
    return this.projectConfig;
  }

  setProjectConfig(config: ProjectConfig): void {
    this.projectConfig = config;
  }
}
```

### 配置项

| 配置项 | 说明 | 默认值 |
|-------|------|--------|
| compileMode | 编译模式 | 'jsbundle' |
| isPreview | 预览模式 | false |
| xtsMode | XTS 模式 | false |
| runtimeOS | 运行时系统 | 'default' |

## 相关文档

- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
- [编译器核心](04_Compiler_Core.md)
- [Koala 包装器](06_Koala_Wrapper.md)

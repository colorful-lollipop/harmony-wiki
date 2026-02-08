# API 与接口差异

## 5.1 差异概述

### 5.1.1 差异类型

OH 适配版 TypeScript 与上游相比，API 差异主要分为以下几类：

| 差异类型 | 说明 | 数量 |
|---------|------|------|
| **新增 API** | OH 为支持 eTS 而新增的接口 | 多项 |
| **行为变更** | 语义或行为与上游不同 | 若干 |
| **禁用功能** | 上游功能在 OH 中被禁用 | 少量 |
| **内部 API** | 上游内部 API 被暴露或使用 | 若干 |

### 5.1.2 差异定位

由于 TypeScript 是 JavaScript 项目，其「API」与传统库有所不同。这里的 API 泛指：

- **TypeScript 编译器 API**：如 `ts.createProgram()`、`ts.CompilerOptions` 等
- **语言服务 API**：如 `ts.getCompletionAtPosition()`、`ts.getDefinitionAtPosition()` 等
- **内部结构**：es2panda 直接使用的 TypeScript 内部类型和函数

## 5.2 新增 API

### 5.2.1 eTS 特有节点类型

**描述**：OH 新增了支持 eTS 组件语法的 AST 节点类型。

**新增类型**：

```typescript
// src/compiler/types.ts

// 组件声明节点
interface StructDeclaration extends DeclarationStatement {
    readonly kind: SyntaxKind.StructDeclaration;
    name: Identifier;
    members: NodeArray<StructMember>;
    heritageClauses?: NodeArray<HeritageClause>;
    modifierFlags?: ModifierFlags;
}

// 组件成员类型
type StructMember =
    | PropertyDeclaration
    | MethodDeclaration
    | ConstructorDeclaration
    | AccessorDeclaration;

// 组件表达式
interface EtsComponentExpression extends PrimaryExpression {
    readonly kind: SyntaxKind.EtsComponentExpression;
    componentName: Identifier;
    typeArguments?: NodeArray<TypeNode>;
    arguments?: NodeArray<Expression>;
}
```

**使用场景**：解析 eTS 源码中的 `struct` 声明和组件调用。

### 5.2.2 装饰器相关 API

**描述**：支持 eTS 装饰器语义的类型定义和检查接口。

**新增类型**：

```typescript
// @Builder 装饰器
interface BuilderDeclaration extends Declaration {
    readonly kind: SyntaxKind.BuilderDeclaration;
    name: Identifier;
    parameters: ParameterDeclaration[];
    body: Block;
    returnType?: TypeNode;
}

// @BuilderParam 参数
interface BuilderParamDeclaration extends ParameterDeclaration {
    readonly kind: SyntaxKind.BuilderParamDeclaration;
    buildType: FunctionType;
    defaultValue?: Expression;
}

// @Styles 样式
interface StylesDeclaration extends Declaration {
    readonly kind: SyntaxKind.StylesDeclaration;
    name: Identifier;
    body: Block;
    targetType?: InterfaceDeclaration;
}

// @Extend 扩展
interface ExtendDeclaration extends Declaration {
    readonly kind: SyntaxKind.ExtendDeclaration;
    targetType: EntityName;
    body: Block;
}
```

### 5.2.3 状态样式相关 API

**描述**：支持 `stateStyles` 状态样式机制的接口。

```typescript
// 状态样式属性
interface StateStylesProperty extends ObjectLiteralElementLike {
    readonly kind: SyntaxKind.StateStylesProperty;
    name: Identifier | StringLiteral;
    value: StateStyleDeclaration;
}

// 状态样式声明
interface StateStyleDeclaration extends Declaration {
    readonly kind: SyntaxKind.StateStyleDeclaration;
    stateName: Identifier;  // normal, pressed, disabled 等
    body: Block;
}

// 可用的状态名称
enum StateName {
    Normal = "normal",
    Pressed = "pressed",
    Disabled = "disabled",
    Focused = "focused",
    Clickable = "clickable",
}
```

### 5.2.4 语法标记扩展

**描述**：新增支持 eTS 语法的标记。

```typescript
// src/compiler/types.ts

enum SyntaxKind {
    // ... 上游已有类型

    // OH 新增
    StructDeclaration = Number.MAX_SAFE_INTEGER - 1,
    EtsComponentExpression,
    BuilderDeclaration,
    BuilderParamDeclaration,
    StylesDeclaration,
    ExtendDeclaration,
    StateStylesProperty,
    StateStyleDeclaration,
}

// 支持的脚本类型
enum ScriptKind {
    TS = 0,
    TSX = 1,
    JS = 2,
    JSX = 3,
    ETS = 4,      // 新增：eTS 脚本
    D_TS = 5,     // 新增：eTS 声明文件
}
```

### 5.2.5 编译器选项扩展

**描述**：新增 OH 特定的编译器选项。

```typescript
interface CompilerOptions {
    // ... 上游已有选项

    // OH 新增选项
    ets?: {
        enableStructSyntax?: boolean;      // 启用 struct 语法
        enableDecoratorSyntax?: boolean;   // 启用装饰器语法
        strictStateStyles?: boolean;       // 严格状态样式检查
    };

    // 性能优化选项
    performance?: {
        enableIncremental?: boolean;
        parallelCheck?: boolean;
    };
}
```

## 5.3 行为变更

### 5.3.1 类型检查行为变更

**变更项**：装饰器参数类型检查

| 行为 | 上游 TypeScript | OH 适配版 |
|-----|----------------|----------|
| `@State` 位置 | 无限制 | 必须在组件内部 |
| `@Prop` 类型 | 无限制 | 必须是简单类型或可观察类型 |
| `@Link` 赋值 | 无限制 | 必须来自父组件 |

**示例**：

```typescript
// 上游 TypeScript：允许任何类型
class MyClass {
    @State items: MyComplexType[];  // 上游允许
}

// OH 适配版：可能报错
class MyComponent {
    @State items: MyComplexType[];  // OH 可能要求使用 @ObjectLink
}
```

### 5.3.2 代码生成行为变更

**变更项**：代码生成目标

| 行为 | 上游 TypeScript | OH 适配版 |
|-----|----------------|----------|
| 输出格式 | JavaScript / Declaration | Panda IR 中间产物 |
| 模块系统 | ESM / CommonJS | OH 特定模块格式 |
| 类型信息 | 编译时擦除 | 可能保留用于运行时检查 |

### 5.3.3 错误诊断行为变更

**变更项**：新增 eTS 特定错误代码

```typescript
// 新增的错误代码范围
enum DiagnosticCode {
    // 上游错误：范围 1000-9999
    // OH 新增错误：范围 100000+

    StructExpected = 100000,
    DecoratorInvalidPlacement = 100001,
    BuilderMustReturnVoid = 100002,
    StateStylesInvalidState = 100003,
    // ... 更多
}

// 错误消息示例
const diagnosticMessages: DiagnosticMessages = {
    StructExpected: {
        category: DiagnosticCategory.Error,
        code: 100000,
        message: "Expected a struct declaration",
    },
};
```

## 5.4 禁用或限制的功能

### 5.4.1 被禁用的功能

| 功能 | 上游状态 | OH 状态 | 原因 |
|-----|---------|--------|------|
| JSX 语法 | 启用 | 禁用 | 与 eTS 语法冲突 |
| 参数装饰器 | 启用 | 限制 | eTS 使用不同机制 |
| 命名空间 | 启用 | 不推荐 | 推荐使用模块 |
| export = | 启用 | 禁用 | 与 OH 模块系统冲突 |

### 5.4.2 限制使用的功能

| 功能 | 限制原因 | 替代方案 |
|-----|---------|---------|
| `any` 类型 | 失去类型安全 | 使用具体类型 |
| `eval()` | 安全风险 | 禁止 |
| 动态 `import()` | 静态分析困难 | 使用静态导入 |

### 5.4.3 配置禁用项

```json
// tsconfig.json OH 适配版
{
    "compilerOptions": {
        // 被禁用的选项
        "jsx": "react-jsx",  // 无效，JSX 不支持

        // 被重写的选项
        "module": "ets",      // OH 特定模块系统

        // 强制的选项
        "strict": true,        // 必须启用严格模式
        "noImplicitAny": true
    },
    "etsOptions": {
        "disableJsx": true,
        "restrictDecoratorTargets": true
    }
}
```

## 5.5 内部 API 使用

### 5.5.1 es2panda 使用的内部 API

es2panda 作为 C++ 编译器，直接使用了 TypeScript 的大量内部 API：

**类型系统内部结构**：

```typescript
// 类型定义（内部 API）
interface TypeSystem {
    anyType: AnyType;
    booleanType: BooleanType;
    numberType: NumberType;
    stringType: StringType;
    voidType: VoidType;
    undefinedType: UndefinedType;
    nullType: NullType;
    neverType: NeverType;
    unknownType: UnknownType;
}

// 类型检查器内部接口
interface TypeCheckerInternal {
    getTypeAtPosition(file: SourceFile, pos: number): Type;
    getTypeOfNode(node: Node): Type;
    checkTypeAssignableTo(source: Type, target: Type): void;
    getConstituentTypes(type: Type): Type[];
}
```

### 5.5.2 语言服务内部 API

IDE 工具使用了语言服务的内部 API：

```typescript
// 语言服务内部接口
interface LanguageServiceInternal {
    getCompletionsAtPosition(fileName: string, position: number): CompletionInfo;
    getDefinitionAtPosition(fileName: string, position: number): DefinitionInfo[];
    getQuickInfoAtPosition(fileName: string, position: number): QuickInfo;
    getSemanticDiagnostics(fileName: string): Diagnostic[];
}

// 增量编译 API
interface IncrementalCompiler {
    getSemanticDiagnostics(fileName: string): Diagnostic[];
    getSyntacticDiagnostics(fileName: string): Diagnostic[];
}
```

### 5.5.3 API 稳定性说明

**警告**：以下 API 为内部 API，可能在版本升级时发生变化：

- 所有以 `_` 开头的属性或方法
- `internal` 命名空间下的所有内容
- `compiler.ts` 中未在 `public.ts` 导出的内容

**建议**：依赖方应尽可能使用公开 API，内部 API 的使用需要做好版本适配准备。

## 5.6 API 差异汇总表

| 类别 | 新增数量 | 变更数量 | 禁用数量 | 说明 |
|-----|---------|---------|---------|------|
| 节点类型 | 10+ | 2 | 0 | 主要为 eTS 语法支持 |
| 装饰器 API | 4+ | 1 | 1 | @Builder、@Styles 等 |
| 编译器选项 | 3+ | 0 | 5+ | OH 特定选项 |
| 诊断代码 | 20+ | 0 | 0 | eTS 特定错误 |
| 内部 API | N/A | N/A | N/A | es2panda 大量使用 |

## 5.7 升级兼容性

### 5.7.1 API 兼容性矩阵

| 升级类型 | 兼容性影响 | 需要的适配 |
|---------|----------|----------|
| 上游 patch | 可能兼容 | 测试验证 |
| 上游 minor | 可能不兼容 | 检查 API 变更 |
| 上游 major | 可能不兼容 | 全面重构 |

### 5.7.2 升级检查清单

升级 TypeScript 上游版本时，需要检查：

- [ ] 新节点类型是否与 OH 新增冲突
- [ ] 内部 API 签名是否变化
- [ ] 诊断代码范围是否重叠
- [ ] 编译器选项是否兼容
- [ ] 语言服务 API 是否变化

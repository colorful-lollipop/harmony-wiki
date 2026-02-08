# OpenHarmony 适配修改详细分析

## 2.1 修改方式概述

### 与传统 Patch 的差异

与 curl、openssl 等使用独立 .patch 文件的第三方库不同，TypeScript 的 OH 适配采用**直接源码集成**方式。这意味着所有 OH 特定修改直接嵌入 TypeScript 源码树中，而非以补丁形式附加。

**原因分析**：TypeScript 是一个大型 JavaScript/TypeScript 项目，其编译过程本身就是代码转换。如果使用传统的 .patch 文件，会面临以下挑战：Patch 数量可能非常庞大，维护成本高；Patch 应用顺序依赖复杂；与上游版本同步困难。因此，OH 选择了直接修改源码并集成到源码树中的方式。

**影响评估**：这种方式的优缺点都很明显。优点是修改集中、易于理解和调试。缺点是升级上游版本时需要手动合并修改，冲突处理复杂；OH 特定修改分散在多个文件中，难以快速识别。

### 修改识别方法

由于没有独立的 patch 文件，识别 OH 修改需要以下方法：

**方法一：README.md 日志**。OH 在 `README.md` 中记录了 2021 年 10 月至 2022 年 2 月期间的修改日志，包括引入的功能和时间点。

**方法二：源码对比**。通过对比 Microsoft/TypeScript 上游源码，可以识别 OH 添加或修改的代码。

**方法三：特征搜索**。搜索 eTS 特有的标识符，如 StructDeclaration、EtsComponentExpression、@Builder、@Extend 等。

## 2.2 eTS 语法扩展

### 2.2.1 Struct 组件语法

**修改文件**：`src/compiler/binder.ts`、`src/compiler/parser.ts`、`src/compiler/checker.ts`

**修改目的**：为 TypeScript 添加声明自定义 UI 组件的能力。Struct 是 ArkUI 声明式 UI 的基础构建块，类似于 SwiftUI 中的 View 或 Jetpack Compose 中的 @Composable 函数。

**关键代码变更**：

```typescript
// 新增节点类型
interface StructDeclaration extends DeclarationStatement {
    kind: SyntaxKind.StructDeclaration;
    name: Identifier;
    members: NodeArray<StructMember>;
    heritageClauses?: NodeArray<HeritageClause>;
}

// 结构体成员类型
type StructMember = 
    | PropertyDeclaration
    | MethodDeclaration
    | ConstructorDeclaration;
```

**语法示例**：

```typescript
// eTS 中的组件定义
@Component
struct MyComponent {
    @Prop title: string;
    @State count: number = 0;

    build() {
        Column() {
            Text(this.title)
            Button(`Count: ${this.count}`)
                .onClick(() => this.count++)
        }
    }
}
```

**实现逻辑**：StructDeclaration 的处理涉及多个编译器阶段。Parser 阶段识别 struct 关键字并解析语法结构；Binder 阶段建立符号表，将成员注册到结构体作用域；Checker 阶段验证类型正确性，包括装饰器语义、属性类型等；Emitter 阶段生成目标代码。

### 2.2.2 @Builder 装饰器

**修改文件**：`src/compiler/checker.ts`、`src/compiler/emitter.ts`

**修改目的**：支持声明式 UI 中的构建函数语法。@Builder 允许开发者定义可复用的 UI 片段构建逻辑。

**语法示例**：

```typescript
@Builder
function headerBuilder(title: string) {
    Row() {
        Text(title)
            .fontSize(20)
    }
}

@Component
struct MyComponent {
    @BuilderParam header: () => void;

    build() {
        Column() {
            this.header()
            // 组件内容
        }
    }
}
```

**类型系统支持**：

```typescript
interface BuilderParamDeclaration extends ParameterDeclaration {
    // Builder 参数具有特殊的构建函数类型
    buildType: FunctionType;
}
```

### 2.2.3 @Styles 装饰器

**修改文件**：`src/compiler/checker.ts`、`src/compiler/emitter.ts`

**修改目的**：支持样式复用机制，允许将一组样式属性抽取为可复用的样式定义。

**语法示例**：

```typescript
@Styles
function globalStyles() {
    .backgroundColor(Color.Red)
    .width(100)
    .height(100)
}

@Component
struct MyComponent {
    @Styles
    localStyles() {
        .backgroundColor(Color.Blue)
    }

    build() {
        Row()
            .globalStyles()  // 应用全局样式
            .localStyles()   // 应用局部样式
    }
}
```

### 2.2.4 @Extend 装饰器

**修改文件**：`src/compiler/checker.ts`、`src/compiler/emitter.ts`

**修改目的**：为内置组件扩展自定义属性和方法。

**语法示例**：

```typescript
@Extend(Text)
function textStyles() {
    .fontSize(16)
    .fontWeight(FontWeight.Bold)
}

@Component
struct MyComponent {
    build() {
        Text("Hello")
            .textStyles()  // 应用扩展样式
    }
}
```

### 2.2.5 @BuilderParam 装饰器

**修改文件**：`src/compiler/checker.ts`、`src/compiler/services/completion.ts`

**修改目的**：支持组件接收构建函数作为参数，实现灵活的组件组合。

**语法示例**：

```typescript
@Component
struct CustomContainer {
    @BuilderParam content: () => void;

    build() {
        Column() {
            this.content()
        }
    }
}

// 使用时传入 lambda
CustomContainer({
    content: () => {
        Text("Child content")
    }
})
```

## 2.3 IDE 语言服务增强

### 2.3.1 gotoDefinition 增强

**修改文件**：`src/services/goToDefinition.ts`

**修改目的**：支持自定义组件名和参数的跳转定义。

**增强内容**：

```typescript
// 原始 goToDefinition 只支持 TypeScript 标识符
// OH 扩展后支持：
// 1. @Component 装饰的 Struct 名称
// 2. @Builder 函数的参数
// 3. @BuilderParam 引用的构建函数
// 4. @Extend 扩展的组件类型
```

**实现逻辑**：当用户点击 eTS 代码中的标识符时，语言服务首先判断标识符类型。如果是组件名，则跳转到对应的 StructDeclaration 定义；如果是 @BuilderParam，则跳转到声明位置。

### 2.3.2 代码补全增强

**修改文件**：`src/services/completion.ts`

**修改目的**：为 eTS 提供上下文感知的代码补全。

**增强内容**：

| 补全场景 | 增强内容 |
|---------|---------|
| 组件生命周期方法 | 自动补全 aboutToAppear、aboutToDisappear 等 |
| 装饰器参数 | 根据组件类型提供正确的参数补全 |
| 状态属性 | 补全 @State、@Prop、@Link 等装饰器 |
| 样式属性 | 提供符合 ArkUI 规范的属性列表 |

**示例**：

```typescript
@Component
struct MyComponent {
    // 输入 @ 后自动补全装饰器
    @  // 弹出: @State @Prop @Link @BuilderParam @Styles @Extend

    // 输入生命周期方法名时
    abou  // 弹出: aboutToAppear() aboutToDisappear()
}
```

### 2.3.3 类型提示增强

**修改文件**：`src/services/hover.ts`、`src/services/quickInfo.ts`

**修改目的**：提供 eTS 特定的类型信息显示。

**增强内容**：包括装饰器语义说明、组件属性类型、状态管理规则等。

## 2.4 类型检查增强

### 2.4.1 装饰器语义检查

**修改文件**：`src/compiler/checker.ts`

**修改目的**：验证装饰器的正确使用。

**检查规则**：

| 装饰器 | 检查规则 |
|-------|---------|
| @State | 只能用于组件内的成员变量 |
| @Prop | 必须有对应的父组件属性 |
| @Link | 必须是组件内的状态变量 |
| @Builder | 只能用于函数或方法 |
| @Styles | 只能用于无参数的方法 |

### 2.4.2 组件类型检查

**修改文件**：`src/compiler/checker.ts`

**修改目的**：验证组件定义的正确性。

**检查项目**：

- 结构体必须包含 build 方法
- build 方法返回值必须是合法的 UI 节点
- 组件继承关系合法性检查
- 循环依赖检测

## 2.5 性能优化

### 2.5.1 eTS 编译性能优化

**修改文件**：`src/compiler/parser.ts`、`src/compiler/checker.ts`

**优化内容**：

| 优化项 | 优化说明 |
|-------|---------|
| 语法解析优化 | 针对 struct 语法的快速解析路径 |
| 类型缓存 | 缓存常见类型，减少重复计算 |
| 并行检查 | 支持增量类型检查 |
| 内存优化 | 优化大型项目的内存占用 |

### 2.5.2 IDE 响应优化

**修改文件**：`src/services/services.ts`

**优化内容**：针对 eTS 项目的语言服务响应优化，包括快速语法分析、延迟完整检查等策略。

## 2.6 按时间线的修改记录

### 2021 年 10 月

| 修改项 | 修改文件 | 修改目的 |
|-------|---------|---------|
| Struct 语法支持 | binder.ts, parser.ts | 支持自定义组件声明 |
| EtsComponentExpression | core.ts | 支持组件表达式语法 |

### 2021 年 11 月

| 修改项 | 修改文件 | 修改目的 |
|-------|---------|---------|
| gotoDefinition 增强 | services/goToDefinition.ts | 支持组件导航 |
| 生命周期补全 | services/completion.ts | 完善 IDE 支持 |
| @Builder 装饰器 | checker.ts, emitter.ts | 支持构建函数 |

### 2022 年 1 月

| 修改项 | 修改文件 | 修改目的 |
|-------|---------|---------|
| eTS 语言优化 | 全局 | 整体性能优化 |
| @BuilderParam | checker.ts, services | 支持构建参数 |
| @Styles | checker.ts, emitter.ts | 支持样式复用 |
| @Extend | checker.ts, emitter.ts | 支持组件扩展 |
| jsDoc 增强 | services/quickInfo.ts | 完善类型提示 |
| PropertyAccessExpressionConditionCheck | checker.ts | 条件表达式检查 |

### 2022 年 2 月

| 修改项 | 修改文件 | 修改目的 |
|-------|---------|---------|
| @Styles 完整支持 | checker.ts, emitter.ts | 样式复用完整实现 |
| struct 名称检查 | checker.ts | 防止与保留名冲突 |
| stateStyles | checker.ts, emitter.ts | 状态样式支持 |
| eTS 补全性能 | services/completion.ts | 性能优化 |

## 2.7 升级上游版本注意事项

### 兼容性风险

升级 TypeScript 上游版本时，OH 需要关注以下风险：

**高风险项**：

| 风险项 | 原因 | 影响 |
|-------|------|------|
| 编译器内部结构变更 | OH 修改依赖特定内部结构 | es2panda、linter 可能失效 |
| 类型系统变更 | eTS 类型检查依赖类型系统细节 | 类型检查可能不正确 |
| 语法解析变更 | struct 语法可能与上游新语法冲突 | 解析错误 |
| API 变更 | 语言服务 API 可能变化 | IDE 功能异常 |

**中风险项**：

| 风险项 | 原因 | 影响 |
|-------|------|------|
| 性能特性变更 | 上游性能优化可能影响 OH 优化 | 编译性能下降 |
| 错误消息变更 | 错误代码可能变化 | 诊断信息不准确 |

### 建议的升级策略

**步骤一：依赖分析**

分析 es2panda、linter 等工具对 TypeScript 内部结构的依赖，列出所有直接引用的内部 API。

**步骤二：修改提取**

提取 OH 对 TypeScript 的所有修改，按模块分类整理。

**步骤三：版本预研**

在独立分支上尝试升级，验证核心功能是否正常。

**步骤四：回归测试**

运行完整的 eTS 测试用例集，验证类型检查、代码生成、IDE 功能等。

### 可推向上游的修改

以下修改具有通用性，可以考虑推向上游：

- 语法解析性能优化
- 增量编译改进
- 语言服务性能优化
- 某些通用错误检查规则

### OH 特有修改

以下修改是 OH 特有的，上游不会接受：

- eTS 组件语法和语义
- ArkUI 装饰器
- 面向 Panda VM 的代码生成
- OH 特定的 IDE 功能

## 2.8 测试相关

### 测试用例位置

OH 在 `tests/arkTSTest/` 目录下维护 eTS 相关的测试用例：

```bash
tests/
├── arkTSTest/
│   ├── README.md           # 测试说明
│   ├── README.zh-cn.md    # 中文说明
│   └── cases/              # 测试用例
│       ├── decorators/     # 装饰器测试
│       ├── components/     # 组件测试
│       └── syntax/         # 语法测试
```

### 测试覆盖

| 测试类别 | 说明 |
|---------|------|
| 语法测试 | struct 语法、装饰器语法 |
| 类型测试 | 组件类型、装饰器类型 |
| 语义测试 | 装饰器语义、组件语义 |
| 编译测试 | 代码生成、错误输出 |
| IDE 测试 | 补全、跳转、提示 |

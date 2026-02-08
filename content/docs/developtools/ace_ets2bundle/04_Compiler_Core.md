# 编译器核心

## 模块职责总览

| 模块 | 文件路径 | 核心职责 |
|------|---------|----------|
| 入口模块 | `compiler/main.js` / `compiler/interop/main.js` | 项目初始化、配置加载 |
| 类型检查 | `compiler/src/ets_checker.ts` | TypeScript 类型服务 |
| 语法转换 | `compiler/src/process_ui_syntax.ts` | UI 语法转换核心 |
| 组件处理 | `compiler/src/process_component_*.ts` | 组件各部分处理 |
| 字节码生成 | `compiler/src/gen_abc*.ts` | ABC 字节码生成 |
| 工具模块 | `compiler/src/*.ts` | 各类辅助功能 |

## 主入口模块

### compiler/main.js

**职责**：编译器的非 Interop 版本主入口，负责项目初始化和流程协调。

**关键代码位置**：`main.js:84-300`

**主要功能**：

```javascript
// 项目配置初始化
function initProjectConfig(projectConfig) {
  initProjectPathConfig(projectConfig);
  projectConfig.entryObj = {};
  projectConfig.cardObj = {};
  projectConfig.aceBuildJson = projectConfig.aceBuildJson || process.env.aceBuildJson;
  // 编译模式配置
  projectConfig.xtsMode = /ets_loader_ark$/.test(__dirname) || process.env.xtsMode === 'true';
  projectConfig.isPreview = projectConfig.isPreview || process.env.isPreview === 'true';
  projectConfig.compileMode = projectConfig.compileMode || process.env.compileMode || 'jsbundle';
  projectConfig.runtimeOS = projectConfig.runtimeOS || process.env.runtimeOS || 'default';
}
```

**环境变量支持**：

| 环境变量 | 说明 | 默认值 |
|---------|------|--------|
| `aceBuildJson` | 构建配置 JSON | 无 |
| `xtsMode` | XTS 测试模式 | false |
| `isPreview` | 预览模式 | false |
| `compileMode` | 编译模式 | 'jsbundle' |
| `runtimeOS` | 运行时系统 | 'default' |

### compiler/interop/main.js

**职责**：Interop 版本主入口，支持 ArkTS 1.1/1.2 混合编译模式。

## 类型检查模块

### ets_checker.ts

**职责**：提供 TypeScript 语言服务，执行 ArkTS 特定类型检查和 Lint 规则。

**关键代码位置**：`ets_checker.ts:380-450`

**主要功能**：

```typescript
// 创建语言服务
export function createLanguageService(rootFileNames: string[], 
                                      resolveModulePaths: string[],
                                      newLogger: Object = null,
                                      compileEnv: CompileEnv = CompileEnv.IDE): ts.LanguageService {
  // 使用 TypeScript Compiler API
  // 配置编译选项
  // 返回语言服务实例
}

// 服务模式检查
export function serviceChecker(rootFileNames: string[], 
                              newLogger: Object = null,
                              resolveModulePaths: string[] = null,
                              compileEnv: CompileEnv = CompileEnv.IDE): void {
  // 创建观察者模式
  // 监听文件变化
  // 增量类型检查
}
```

**检查类型**：

| 检查类型 | 说明 |
|---------|------|
| ArkTS 类型规则 | ArkTS 特定类型约束 |
| API 可用性 | 系统 API 版本检查 |
| 导入导出验证 | 模块解析验证 |
| 装饰器规则 | @Component 等检查 |

### do_arkTS_linter.ts

**职责**：执行 ArkTS Lint 规则检查。

## 语法转换模块

### process_ui_syntax.ts

**职责**：将声明式 UI 语法转换为运行时调用代码。

**关键代码位置**：`process_ui_syntax.ts:189-250`

**主要处理内容**：

```typescript
// 装饰器处理
export function processDecorator(node: ts.Decorator): ts.Node {
  // @Component -> 创建组件构造函数
  // @Entry -> 标记入口页面
  // @Preview -> 预览支持
  // @State/@Prop/@Link -> 状态管理
}

// build() 方法转换
export function processUISyntax(program: ts.Program, 
                                ut = false,
                                ...): ts.TransformationResult {
  // 遍历组件 build 方法
  // 转换声明式语法为函数调用
  // 生成组件树结构
}
```

### 组件处理系列模块

| 模块 | 职责 |
|------|------|
| `process_component_class.ts` | 组件类结构处理 |
| `process_component_member.ts` | 组件成员变量处理 |
| `process_component_constructor.ts` | 构造函数生成 |
| `process_component_build.ts` | build() 方法内容处理 |
| `process_custom_component.ts` | 自定义组件实例化 |

**装饰器处理流程**：

```
@State name: string = 'value'
    ↓
createState('name', this)  // 运行时调用
    ↓
状态管理框架初始化

@Prop @Link @StorageProp
    ↓
createProp/createLink  // 属性绑定
    ↓
响应式数据流建立
```

### process_import.ts

**职责**：处理模块导入语句。

**关键代码位置**：`process_import.ts:758-900`

**处理逻辑**：

```typescript
export function processImportModule(node: ts.ImportDeclaration, 
                                   pageFile: string, 
                                   log: LogInfo[],
                                   ...): void {
  // 解析 import 语句
  // 处理 OhmUrl 格式 (@kit.xxx)
  // 验证模块路径
  // 收集导入信息供后续使用
}
```

## 字节码生成模块

### gen_abc.ts

**职责**：调用 es2abc 工具生成 ABC 字节码。

**关键代码位置**：`gen_abc.ts:38-80`

**调用方式**：

```typescript
// 子进程调用 es2abc
const es2abcPath = path.join(arkDir, 'es2abc');
const args = ['--debug-info', inputFile, '-o', outputFile];
childProcess.spawnSync(es2abcPath, args);
```

### gen_abc_plugin.ts

**职责**：Webpack/Rollup 插件形式的 ABC 生成器。

**关键代码位置**：`gen_abc_plugin.ts:162-200`

**插件接口**：

```typescript
export class GenAbcPlugin {
  apply(compiler: Compiler) {
    compiler.hooks.thisCompilation.tap(
      'GenAbcPlugin',
      (compilation) => {
        // 编译开始钩子
        // 添加编译模块
        // 生成 ABC
      }
    );
  }
}
```

### gen_aot.ts

**职责**：调用 ark_aot_compiler 进行 AOT 编译。

```typescript
export function generateAot(arkDir: string, 
                            appAbc: string,
                            ...): boolean {
  // 调用 AOT 编译器
  const aotCompiler = path.join(arkDir, 'ark_aot_compiler');
  // 参数配置
  // 执行编译
}
```

### manage_workers.ts

**职责**：Worker 进程管理，支持多进程并行编译。

```typescript
// 使用 cluster 模块
import cluster from 'cluster';
import { logger } from './compile_info';

if (cluster.isMaster) {
  // 创建 Worker 进程
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  // Worker 进程处理 ABC 生成
}
```

## Fast Build 模块

### rollup-plugin-gen-abc.ts

**职责**：Rollup 插件形式的 ABC 生成。

**关键代码位置**：`fast_build/ark_compiler/rollup-plugin-gen-abc.ts`

**编译模式**：

| 模式 | 说明 | 使用场景 |
|------|------|---------|
| bundle 模式 | 所有模块合并为一个 ABC | 应用打包 |
| module 模式 | 每个模块独立 ABC | 模块化开发 |
| common 模式 | 共享库 + 业务模块 | 性能优化 |

### system_api/ 模块

**职责**：系统 API 检查和权限验证。

```typescript
// api_check_utils.ts
export function checkPermissionValue(
  apiName: string,
  permission?: string
): PermissionCheckResult {
  // 检查 API 是否需要权限
  // 验证应用是否声明权限
  // 返回检查结果
}
```

## 错误处理

### hvigor_error_code/ 模块

**职责**：统一的错误码和诊断信息管理。

```typescript
// error_code_module.ts
export enum SubsystemCode {
  ARKUI = '109',
  LINTER = '106',
  ABILITY = '101'
}

export enum ErrorCode {
  SYNTAX_ERROR = '001',
  TYPE_ERROR = '002',
  API_ERROR = '003'
}
```

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [ArkUI 插件系统](05_ArkUI_Plugins.md)
- [GN 构建目标](07_GN_Targets.md)

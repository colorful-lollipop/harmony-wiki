# 安全风险评审

## 评审范围

本评审覆盖 Ace ETS2Bundle 项目的以下模块：

- **compiler/** - TypeScript 编译器核心
- **arkui-plugins/** - ArkUI 插件系统
- **koala-wrapper/** - Koala Native 包装器
- **BUILD.gn** - 构建配置

**不包含**：
- 测试代码 (test/ tests/)
- 第三方依赖 (node_modules/)
- 构建产物 (out/ build/)

## 攻击面分析

### 输入点清单

| 输入类型 | 来源 | 处理位置 | 风险等级 |
|---------|------|---------|---------|
| ETS 源码文件 | 用户代码 | compiler/src/*.ts | 中 |
| 模块导入路径 | 用户代码 | process_import.ts | 中 |
| 组件定义 | 用户代码 | arkui-plugins/ | 中 |
| 系统 API 调用 | 用户代码 | system_api/ | 高 |
| bundleName | 配置文件 | main.js:505 | 低 |
| 编译配置 JSON | 构建系统 | gen_abc_plugin.ts | 低 |
| 命令行参数 | 构建系统 | manage_workers.ts | 中 |
| 环境变量 | 构建系统 | main.js | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         不可信区域 (用户输入)                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  ETS 源文件 (.ets)                                              │   │
│  │  组件属性值                                                     │   │
│  │  import 路径                                                    │   │
│  │  运行时数据                                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                         信任边界                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  编译器核心 (compiler/src/)                                     │   │
│  │  - 类型检查隔离                                                  │   │
│  │  - 语法转换沙箱                                                 │   │
│  │  - 输入验证                                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────┤
│                         可信区域 (系统代码)                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  系统模块 (@kit/*)                                              │   │
│  │  运行时库                                                       │   │
│  │  Native 绑定                                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## 已识别风险点

### 1. 路径遍历风险 ⚠️ MEDIUM

**证据**：`compiler/src/ark_utils.ts`

```typescript
// 路径处理可能存在遍历风险
function resolveModulePath(basePath: string, modulePath: string): string {
  // modulePath 未做规范化处理
  return path.join(basePath, modulePath);
}
```

**触发条件**：
- 构造恶意 import 路径，如 `../../../etc/passwd`

**影响**：
- 读取任意文件
- 路径遍历泄露敏感配置

**修复建议**：
```typescript
function resolveModulePath(basePath: string, modulePath: string): string {
  // 规范化路径
  const resolved = path.resolve(basePath, modulePath);
  // 验证在基础目录内
  if (!resolved.startsWith(basePath)) {
    throw new Error('Path traversal attempt detected');
  }
  return resolved;
}
```

### 2. 权限注解验证 ⚠️ MEDIUM

**证据**：`compiler/src/fast_build/system_api/api_check_utils.ts:409`

```typescript
export function checkPermissionValue(
  apiName: string,
  permission?: string
): PermissionCheckResult {
  // 权限注解检查
  if (permission && !hasPermission(permission)) {
    return { allowed: false, reason: 'Permission denied' };
  }
  return { allowed: true };
}
```

**触发条件**：
- 应用调用需要权限的 API 但未声明权限

**影响**：
- 权限绕过
- 未授权访问系统能力

**当前缓解**：
- 编译时检查
- 错误阻止构建

### 3. bundleName 处理 ⚠️ LOW

**证据**：`compiler/main.js:505`

```javascript
// 从 module.json 读取 bundleName
const bundleName = moduleJson.bundleName || 'default';
```

**触发条件**：
- 恶意 module.json 配置

**影响**：
- 有限（仅影响应用标识）

**当前缓解**：
- 默认值处理

### 4. 命令注入风险 ⚠️ LOW

**证据**：`compiler/src/gen_abc.ts`

```typescript
// 子进程调用 es2abc
const es2abcPath = path.join(arkDir, 'es2abc');
childProcess.spawnSync(es2abcPath, args);
```

**触发条件**：
- arkDir 或 args 被注入

**当前缓解**：
- 使用 `spawnSync` 而非 `exec`
- 路径来自配置，非用户输入

### 5. Native 内存访问 ⚠️ MEDIUM

**证据**：`koala-wrapper/native/src/bridges.cc`

```cpp
// 原生指针传递
KOALA_INTEROP_1(FreeMemory, void, KNativePointer) {
  void* ptr = reinterpret_cast<void*>(p0);
  free(ptr);  // 内存释放
}
```

**触发条件**：
- 传递无效指针
- 双重释放

**影响**：
- 内存破坏
- 崩溃

**当前缓解**：
- 指针验证
- Node.js 异常处理

### 6. N-API 类型转换风险 ⚠️ LOW

**证据**：`koala-wrapper/koalaui/interop/src/cpp/napi/convertors-napi.cc`

```cpp
// JS 到 Native 类型转换
template<>
struct InteropTypeConverter<KInt> {
  static KInt fromJs(napi_env env, napi_value jsValue) {
    int64_t value;
    napi_get_value_int64(env, jsValue, &value);  // 可能溢出
    return static_cast<KInt>(value);
  }
};
```

**触发条件**：
- 传递超大数值

**影响**：
- 整数溢出
- 静默数据丢失

### 7. 线程安全风险 ⚠️ MEDIUM

**证据**：`koala-wrapper/koalaui/interop/src/cpp/common-interop.cc`

```cpp
// 线程安全函数调用
napi_create_threadsafe_function(env, ...);
```

**触发条件**：
- 多线程并发访问

**影响**：
- 竞态条件
- 内存不一致

### 8. 正则表达式 DoS ⚠️ LOW

**证据**：`compiler/src/ets_checker.ts`

```typescript
// 正则表达式处理
const pattern = /^(?:[a-z]+\.)*[a-z]+$/;
// 未限制回溯
```

**触发条件**：
- 构造恶意输入触发 ReDoS

**影响**：
- 编译时间显著增加

## 安全最佳实践

### 已采用的安全措施

| 措施 | 实现位置 | 效果 |
|------|---------|------|
| 输入验证 | process_import.ts | 防止恶意路径 |
| 类型检查 | ets_checker.ts | 编译时发现错误 |
| 沙箱执行 | gen_abc.ts | 隔离 Native 调用 |
| 权限检查 | api_check_utils.ts | API 访问控制 |
| 异常处理 | 全局 | 防止信息泄露 |

### 建议改进

| 风险 | 建议 | 优先级 |
|------|------|-------|
| 路径遍历 | 实现路径白名单 | 高 |
| 内存安全 | 增加指针验证 | 高 |
| 整数溢出 | 范围检查 | 中 |
| ReDoS | 正则超时限制 | 中 |
| 竞态 | 锁机制增强 | 中 |

## 依赖安全

### 第三方依赖

| 依赖 | 版本 | 用途 | 风险评估 |
|------|------|------|---------|
| TypeScript | 4.x | 编译器 | 低 |
| webpack | 5.x | 构建工具 | 低 |
| @babel/* | 7.x | 代码转换 | 低 |
| log4js | 6.x | 日志 | 低 |

### 构建时依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| es2abc | ets_frontend | 字节码生成 |
| ark_aot_compiler | runtime_core | AOT 编译 |
| bc_obfuscator | arkcompiler | 字节码混淆 |

## 相关文档

- [架构说明](02_Architecture.md)
- [编译器核心](04_Compiler_Core.md)
- [Koala 包装器](06_Koala_Wrapper.md)
- [常见问题](10_Troubleshooting.md)

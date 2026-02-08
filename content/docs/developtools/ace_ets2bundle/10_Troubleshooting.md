# 常见问题与解决方案

## 构建问题

### Q1: npm install 失败

**问题**：安装依赖时出错

**解决方案**：

```bash
# 1. 检查 Node.js 版本
node -v  # 需要 >= 12.18.3

# 2. 配置 npm 镜像
npm config set registry https://registry.npmmirror.com
npm config set strict-ssl false

# 3. 清理缓存
npm cache clean -f

# 4. 重新安装
npm install
```

**相关文件**：`compiler/package.json`

### Q2: GN 构建报错

**问题**：GN 构建时找不到依赖

**解决方案**：

```bash
# 1. 检查环境变量
echo $ANDROID_HOME
echo $OHOS_SDK

# 2. 检查 GN 路径
which gn

# 3. 同步子模块
git submodule update --init --recursive

# 4. 清理构建缓存
rm -rf out/
```

**相关文件**：`BUILD.gn`, `bundle.json`

### Q3: es2abc 工具找不到

**问题**：ABC 字节码生成失败

**错误信息**：
```
Error: spawn es2abc ENOENT
```

**解决方案**：

```bash
# 1. 检查 ets_frontend 子系统构建
cd $OHOS_ROOT
./build.sh --product-name xxx --subsystem developtools

# 2. 检查工具路径
ls -la out/xxx/arkcompiler/ets_frontend/es2abc

# 3. 设置环境变量
export ETS_FRONTEND_HOME=$OHOS_ROOT/out/xxx/arkcompiler/ets_frontend
```

**相关文件**：`compiler/src/gen_abc.ts`

## 编译问题

### Q4: 类型检查错误

**问题**：ArkTS 类型检查失败

**常见错误**：

| 错误码 | 说明 | 解决方案 |
|--------|------|---------|
| 105001 | 未定义变量 | 检查变量声明 |
| 105002 | 类型不匹配 | 检查赋值类型 |
| 105003 | 参数类型错误 | 检查函数参数 |
| 105004 | 返回值类型错误 | 检查 return 语句 |

**相关文件**：`compiler/src/ets_checker.ts`

### Q5: 组件装饰器问题

**问题**：@Component 装饰器使用错误

**示例错误**：

```typescript
// 错误示例
@Entry  // @Entry 必须在 @Component 之前
@Component
struct MyComponent {
  build() {
    Text('Hello')
  }
}

// 正确示例
@Component
@Entry  // @Entry 在 @Component 之后
struct MyComponent {
  build() {
    Text('Hello')
  }
}
```

**相关文件**：`compiler/src/process_component_class.ts`

### Q6: 状态装饰器使用问题

**问题**：@State/@Prop/@Link 使用错误

**示例错误**：

```typescript
// 错误：@State 不能用于复杂对象未初始化
@State data: MyData;  // 未初始化

// 正确：提供默认值或初始化
@State data: MyData = new MyData();

// 正确：使用 ? 可选参数
@State data?: MyData;
```

**相关文件**：`compiler/src/process_component_member.ts`

## 运行时问题

### Q7: ABC 文件加载失败

**问题**：运行时无法加载 ABC

**诊断步骤**：

```bash
# 1. 验证 ABC 文件格式
file xxx.abc

# 2. 检查 ABC 版本兼容性
xxd xxx.abc | head -5

# 3. 查看运行时日志
hilog | grep -i abc
```

**解决方案**：

```bash
# 1. 重新编译
npm run compile

# 2. 清理构建缓存
rm -rf build/
npm run build
```

**相关文件**：`compiler/src/gen_abc_plugin.ts`

### Q8: Native 模块加载失败

**问题**：Koala Native Addon 加载失败

**错误信息**：
```
Error: Module version mismatch. Expected XX, got YY.
```

**解决方案**：

```bash
# 1. 检查 Node.js 版本匹配
node -v

# 2. 重新编译 Native 模块
cd koala-wrapper/native
npm install
npm run build

# 3. 检查系统架构匹配
uname -m
```

**相关文件**：`koala-wrapper/native/BUILD.gn`

## 性能问题

### Q9: 编译速度慢

**问题**：大型项目编译耗时过长

**优化方案**：

```bash
# 1. 使用 Fast Build 模式
npm run build -- --fast

# 2. 启用增量编译
npm run watch

# 3. 增加并行度
export MAX_WORKERS=8
```

**相关文件**：`compiler/src/fast_build/ark_compiler/`

### Q10: 内存占用过高

**问题**：编译时内存不足

**诊断**：

```bash
# 1. 查看内存使用
top -pid $(pgrep -f "node")

# 2. 设置内存限制
export NODE_OPTIONS="--max-old-space-size=4096"
```

**解决方案**：

```typescript
// 启用内存监控
// compiler/src/fast_build/meomry_monitor/
```

**相关文件**：`compiler/src/fast_build/memory_monitor/`

## 插件问题

### Q11: ArkUI 插件未生效

**问题**：UI 语法转换未执行

**诊断步骤**：

```bash
# 1. 检查插件配置
cat module.json | grep "arkui"

# 2. 验证插件路径
ls -la node_modules/@ohos/arkui-plugins/

# 3. 查看调试日志
DEBUG=arkui-plugins npm run build
```

**相关文件**：`arkui-plugins/`

### Q12: 自定义组件未识别

**问题**：自定义组件无法解析

**解决方案**：

```typescript
// 1. 检查组件声明
// 组件必须使用 @Component 装饰器

// 2. 检查文件扩展名
// 必须为 .ets 文件

// 3. 检查路径导入
import { MyComponent } from './MyComponent';

// 4. 检查导出
@Component
export struct MyComponent {
  build() {
    Text('Hello')
  }
}
```

**相关文件**：`compiler/src/process_custom_component.ts`

## 调试技巧

### 启用调试日志

```bash
# ArkTS 检查器日志
DEBUG=ets_checker npm run build

# Fast Build 日志
DEBUG=fast_build npm run build

# 插件日志
DEBUG=arkui-plugins npm run build
```

### 验证 ABC 字节码

```bash
# 使用 abc-dump 工具
abc-dump xxx.abc

# 检查字节码内容
xxd xxx.abc | head -20
```

### 语法树调试

```typescript
// 在代码中添加调试输出
import * as ts from 'typescript';

function debugAST(node: ts.Node) {
  console.log(ts.SyntaxKind[node.kind]);
  ts.forEachChild(node, debugAST);
}
```

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [编译器核心](04_Compiler_Core.md)
- [安全风险评审](09_Security_Review.md)

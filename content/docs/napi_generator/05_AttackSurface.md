# 攻击面分析

> napi-generator 工具集攻击面、信任边界与潜在风险点分析

## 分析范围

| 范围 | 说明 | 包含 |
|------|------|------|
| **代码生成工具** | `src/cli/` 下 6 个工具 | ✅ 包含 |
| **IDE 插件** | VSCode/IntelliJ 插件 | ✅ 包含 |
| **生成产物** | 工具生成的 N-API / SA 框架代码 | ✅ 包含 |
| **运行时库** | OpenHarmony N-API 运行时 | ❌ 不包含 |

---

## 威胁模型

### 1.1 数据流图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           napi-generator 威胁模型                             │
└─────────────────────────────────────────────────────────────────────────────┘

   外部输入层                      工具处理层                      生成产物层
      │                              │                              │
      ▼                              ▼                              ▼
┌───────────────┐          ┌──────────────────┐          ┌──────────────────┐
│  .d.ts 文件   │          │                  │          │  N-API C++ 代码  │
│  .h 头文件    │ ───────▶ │  生成器引擎      │ ───────▶ │  SA 框架代码     │
│  CMakeLists   │          │  (dts2cpp/h2sa)  │          │  BUILD.gn        │
│  配置文件     │          │                  │          │                  │
└───────────────┘          └──────────────────┘          └──────────────────┘
      │                              │                              │
      │ 攻击向量                      │ 处理风险                      │ 代码缺陷
      │ • 恶意输入                   │ • 解析漏洞                    │ • 缓冲区溢出
      │ • 路径遍历                   │ • 资源耗尽                    │ • 整数溢出
      │ • 代码注入                   │ • 逻辑错误                    │ • 空指针
```

### 1.2 信任边界

| 边界 | 描述 | 信任级别 | 证据 |
|------|------|----------|------|
| **输入边界** | 用户提供的 .d.ts / .h / CMakeLists.txt | ❌ 不信任 | 任何用户可构造输入 |
| **生成器内部** | 工具核心代码 (`gen/` 目录) | ✅ 信任 | 项目维护代码 |
| **模板代码** | 代码模板 (`.gen` 文件) | ✅ 信任 | 项目维护模板 |
| **生成产物** | 自动生成的 N-API / SA 代码 | ⚠️ 需验证 | 依赖输入正确性 |
| **运行时** | OpenHarmony 系统 | ✅ 信任 | 操作系统级别 |

---

## 攻击面清单

### 2.1 输入解析攻击面

#### A. TypeScript 声明文件解析 (dts2cpp)

**位置**: `src/cli/dts2cpp/src/gen/analyze.js`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| **正则表达式 DoS** | ⚠️ Medium | 复杂正则匹配导致 CPU 耗尽 |
| **深度嵌套结构** | ⚠️ Medium | 极端嵌套 namespace/class 导致栈溢出 |
| **超长标识符** | ✅ Low | 函数/变量名超长导致内存问题 |
| **类型混淆** | ⚠️ Medium | 构造特殊类型声明导致解析错误 |

**代码证据**:
```javascript
// src/cli/dts2cpp/src/gen/analyze.js:25
let data = readFile(fn);  // 读取用户输入文件

// 解析逻辑未对输入大小做限制
// TODO: 需要验证是否有文件大小和复杂度限制
```

#### B. C++ 头文件解析 (h2sa/h2dtscpp)

**位置**: 
- `src/cli/h2sa/src/gen/analyze.js:31-39`
- `src/cli/h2dtscpp/src/src/tsGen/tsMain.js:39-43`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| **外部程序调用** | ⚠️ Medium | header_parser.exe 执行风险 |
| **命令注入** | ⚠️ Medium | 头文件路径可能注入命令 |
| **解析器漏洞** | ⚠️ Medium | 恶意构造的头文件触发解析 bug |

**代码证据**:
```javascript
// src/cli/h2sa/src/gen/analyze.js:31-39
let exeFile = sysInfo === 'win32' ? 
    path.join(execPath, 'header_parser.exe') :
    path.join(execPath, 'header_parser');
// 外部可执行文件调用，路径可能受污染
```

#### C. CMakeLists.txt 解析 (cmake2gn)

**位置**: `src/cli/cmake2gn/src/src/analyze_cmake.js`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| **命令执行** | 🔴 High | CMake 配置可能包含命令执行 |
| **路径遍历** | ⚠️ Medium | 文件引用可能越界 |

### 2.2 文件系统攻击面

#### A. 输入文件读取

**位置分布**:
- `src/cli/dts2cpp/src/gen/analyze.js:25`
- `src/cli/dts2cpp/src/gen/cmd_gen.js:109`
- `src/cli/h2sa/src/gen/analyze.js:39`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| **路径遍历** | ⚠️ Medium | `../../../etc/passwd` 等路径 |
| **符号链接** | ⚠️ Medium | 符号链接指向敏感文件 |
| **文件描述符耗尽** | ✅ Low | 打开过多文件 |

#### B. 输出文件写入

**位置分布**:
- `src/cli/dts2cpp/src/gen/generate.js:282,304,315,349`
- `src/cli/dts2cpp/src/gen/tools/FileRW.js:134`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| **路径遍历写入** | ⚠️ Medium | `-o ../../etc/` 等参数 |
| **文件覆盖** | ⚠️ Medium | 覆盖系统文件或他人文件 |
| **磁盘耗尽** | ✅ Low | 生成超大文件占满磁盘 |

**代码证据**:
```javascript
// src/cli/dts2cpp/src/gen/cmd_gen.js:43
NapiLog.init(ops.loglevel, path.join('' + ops.out, 'napi_gen.log'));
// ops.out 可能包含 ../ 遍历路径

// src/cli/dts2cpp/src/gen/generate.js:282
writeFile(re.pathJoin(destDir, '%s.cpp'.format(ns0.name)), ...)
// destDir 若未校验可能导致写入任意位置
```

### 2.3 命令执行攻击面

#### A. header_parser.exe 调用

**风险等级**: 🔴 **High**

**位置**:
- `src/cli/h2dtscpp/src/src/tsGen/tsMain.js:39-43`
- `src/cli/h2dts/src/tsGen/tsMain.js:33-34`
- `src/cli/h2sa/src/gen/analyze.js:31-32`

**攻击场景**:
1. 修改 `header_parser.exe` 为恶意程序
2. 通过环境变量注入恶意路径
3. 利用路径遍历执行其他程序

#### B. clang-format 调用

**风险等级**: 🔴 **High**

**位置**: `src/cli/dts2cpp/src/gen/generate.js:200`

**代码证据**:
```javascript
let cmd = '"' + localClangFmtFile + '" -style=file -i "' + 
    path.resolve(path.join(destDir, genFileList[i])) + '"';
// 命令行字符串拼接，潜在命令注入
```

**攻击场景**:
```bash
# 若 genFileList[i] 可被控制
"; rm -rf /; echo "  # 可能导致命令注入
```

### 2.4 模板系统攻击面

#### A. 模板变量注入

**位置**: `src/cli/h2sa/src/gen/file_template.js`

**风险等级**: ✅ Low

**说明**: 模板替换使用简单的字符串替换，若模板变量可被用户控制，可能存在注入风险。

#### B. 模板文件读取

**位置**: `src/cli/h2dtscpp/src/src/napiGen/functionDirect.js:79-88`

**代码证据**:
```javascript
let funcGetParamTempletePath = path.join(__dirname, ...);
let funcGetParamTemplete = readFile(funcGetParamTempletePath);
// 模板文件读取，若模板被篡改则生成恶意代码
```

### 2.5 生成代码攻击面

#### A. 缓冲区溢出

**位置**: `src/cli/dts2cpp/src/gen/extend/tool_utility.js:1338`

**代码证据**:
```cpp
char buf[1024];
napi_status result_status = napi_get_value_string_utf8(env_, value, buf, 1024, &result);
// 固定大小缓冲区，超长字符串可能截断或溢出
```

**风险**: 生成代码中使用固定大小缓冲区，若输入字符串超过 1024 字节可能溢出。

#### B. 整数溢出

**位置**: `src/cli/dts2cpp/src/gen/generate/function_direct.js:118`

**代码证据**:
```javascript
if (type === 'number') {
    cppType = 'uint32_t';  // 默认映射
}
```

**风险**: TypeScript number 可表示 `9007199254740993`，映射为 `uint32_t` 导致截断。

#### C. 空指针解引用

**位置**: 生成的 N-API 代码中

**风险**: 若用户未实现业务函数，调用时可能导致空指针解引用。

### 2.6 IPC 攻击面 (h2sa)

#### A. MessageParcel 反序列化

**位置**: `src/cli/h2sa/src/tools/common.js`

**代码证据**:
```javascript
const DATA_R_MAP = {
    'int32_t': 'ReadInt32',
    'int': 'ReadInt32',
    // ...
};
// 生成代码中直接调用 data.ReadInt32()，无长度校验
```

**风险**: 恶意客户端发送畸形 IPC 请求，服务端读取越界或类型混淆。

#### B. SA 权限配置

**位置**: `src/cli/h2sa/src/gen/file_template.js` 生成的配置

**风险**: 生成的 SA 配置可能缺少必要的权限声明，导致未授权访问。

---

## 风险评估矩阵

| 风险 ID | 风险描述 | 攻击向量 | 影响 | 可能性 | 风险等级 |
|---------|----------|----------|------|--------|----------|
| R1 | 命令注入 | clang-format 参数 | 代码执行 | 中 | 🔴 High |
| R2 | 外部程序执行 | header_parser.exe | 代码执行 | 中 | 🔴 High |
| R3 | 路径遍历写入 | -o 参数 | 文件覆盖 | 中 | ⚠️ Medium |
| R4 | 缓冲区溢出 | 超长字符串 | 程序崩溃 | 低 | ⚠️ Medium |
| R5 | 整数溢出 | number 类型 | 逻辑错误 | 中 | ⚠️ Medium |
| R6 | IPC 越界读取 | MessageParcel | 信息泄露 | 中 | ⚠️ Medium |
| R7 | ReDoS | 正则解析 | DoS | 低 | ⚠️ Medium |
| R8 | 资源耗尽 | 嵌套结构 | DoS | 低 | ✅ Low |

---

## 修复建议

### 短期修复 (P0)

#### S1: 路径校验

**位置**: `src/cli/dts2cpp/src/gen/cmd_gen.js`

```javascript
// 添加输出路径校验
function validateOutputPath(outDir, baseDir) {
    const resolvedOut = path.resolve(outDir);
    const resolvedBase = path.resolve(baseDir);
    
    if (!resolvedOut.startsWith(resolvedBase)) {
        throw new Error('Output path outside allowed directory');
    }
}
```

#### S2: 命令参数转义

**位置**: `src/cli/dts2cpp/src/gen/generate.js:200`

```javascript
// 使用数组参数代替字符串拼接
const { execFile } = require('child_process');
execFile(localClangFmtFile, ['-style=file', '-i', filePath]);
```

#### S3: 输入大小限制

```javascript
// 添加文件大小和复杂度限制
const MAX_FILE_SIZE = 10 * 1024 * 1024;  // 10MB
const MAX_NESTING_DEPTH = 50;
```

### 中期修复 (P1)

#### M1: 沙箱执行

- 在沙箱环境中执行 header_parser.exe
- 限制文件系统访问范围
- 限制网络访问

#### M2: 类型安全改进

```javascript
// number 类型映射改进
function mapNumberType(value) {
    if (!Number.isInteger(value)) return 'double';
    if (value >= -2147483648 && value <= 2147483647) return 'int32_t';
    if (value >= 0 && value <= 4294967295) return 'uint32_t';
    return 'int64_t';
}
```

#### M3: 缓冲区安全

```cpp
// 使用动态缓冲区
std::string buf;
buf.resize(napi_get_value_string_utf8(env, value, nullptr, 0, &len));
napi_get_value_string_utf8(env, value, &buf[0], buf.size(), &len);
```

### 长期修复 (P2)

#### L1: 输入验证框架

- 建立统一的输入验证框架
- 对所有用户输入进行白名单校验
- 添加 Schema 验证

#### L2: 模糊测试

- 对解析器进行模糊测试
- 自动化发现解析漏洞
- 持续集成安全测试

#### L3: 安全审计

- 定期安全代码审计
- 第三方安全评估
- 建立安全响应流程

---

## 安全最佳实践

### 对工具使用者

1. **输入验证**
   - 仅使用可信来源的 .d.ts / .h 文件
   - 检查输入文件大小（建议 < 1MB）
   - 避免处理来自互联网的未知文件

2. **输出路径控制**
   - 使用绝对路径指定输出目录
   - 避免使用相对路径（特别是包含 `..` 的）
   - 确保输出目录有适当权限

3. **环境安全**
   - 确保 `header_parser.exe` 来源可信
   - 定期检查工具完整性
   - 在隔离环境中运行不可信输入

### 对生成代码使用者

1. **输入校验**
   - 在业务代码中添加参数校验
   - 检查字符串长度和数组大小
   - 验证数值范围

2. **边界检查**
   - 对缓冲区操作添加边界检查
   - 使用安全的字符串操作函数
   - 避免固定大小缓冲区

3. **IPC 安全** (SA 服务)
   - 在 Stub 中添加参数长度校验
   - 验证 MessageParcel 可读字节数
   - 添加权限校验

---

## 相关章节

- 架构说明: [03_Architecture.md](03_Architecture.md)
- 安全评审: [07_Security_Review.md](07_Security_Review.md)
- 接口文档: [04_NAPI_Reference.md](04_NAPI_Reference.md)

---

[返回 SUMMARY.md](SUMMARY.md)

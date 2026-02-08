# 安全评审

## 概述

本章节对 Previewer 项目进行全面的安全风险分析，识别攻击面、信任边界和潜在漏洞。

## 攻击面清单

| 攻击面 | 入口 | 类型 | 风险等级 |
|--------|------|------|----------|
| **命名管道** | 命令通道 | 任意进程可连接 | HIGH |
| **WebSocket** | 图像通道 | 任意进程可连接 | HIGH |
| **CLI 参数** | 启动参数 | 攻击者可控 | MEDIUM |
| **配置文件** | JSON 文件 | 本地攻击者 | MEDIUM |
| **HSP 文件** | 模块加载 | 恶意 HSP 包 | HIGH |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     DevEco Studio (可信)                      │
└─────────────────────┬───────────────────────────────────────┘
                      │ 命名管道 / WebSocket
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  Previewer (受信任边界)                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  命令解析 → 渲染引擎 → 预览图像                       │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────┬───────────────────────────────────────┘
                      │ 文件系统 / 系统调用
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                 操作系统 (不可信)                             │
└─────────────────────────────────────────────────────────────┘
```

**关键发现**: Previewer 接收来自 DevEco Studio 的所有命令，**没有额外的认证或验证机制**。

---

## 安全风险分析

### HIGH: 路径遍历漏洞

**证据**: `jsapp/rich/external/StageContext.cpp:438-443`

```cpp
bool StageContext::ContainsRelativePath(const std::string& path) const
{
    std::string flg1 = ".." + FileSystem::GetSeparator();
    std::string flg2 = "." + FileSystem::GetSeparator();
    return (path.find(flg1) != std::string::npos || path.find(flg2) != std::string::npos);
}
```

**问题**: 仅检测 `../` 和 `./`，遗漏：
- Null byte 注入 (`\0`)
- Unicode 范化攻击
- 双编码 (`..%2f..%2f`)
- Windows 绝对路径 (`C:\Windows\`)

**触发路径**:
```
DevEco Studio → LoadDocument 命令 → inputPath 参数 → StageContext::GetModuleBuffer()
```

**影响**: 可读取任意文件

**修复建议**: 使用 `realpath()` / `GetFullPathName()` 规范化路径后再验证

---

### HIGH: Zip Slip 漏洞

**证据**: `jsapp/rich/external/StageContext.cpp:627`

```cpp
std::string filePath = writePath + FileSystem::GetSeparator() + fileName;
FILE *outputFile = fopen(filePath.c_str(), "wb");
```

**问题**: 压缩包内文件名直接拼接到目标路径

**触发路径**:
```
DevEco Studio → Load HSP 包 → UnzipHspFile() → 恶意文件名 "../../../etc/passwd"
```

**影响**: 可覆盖系统文件

**修复建议**: 解压前验证解析后的路径是否在目标目录内

---

### HIGH: WebSocket SID 认证绕过

**证据**: `util/WebSocketServer.cpp:54-71`

```cpp
bool WebSocketServer::CheckSid(struct lws* wsi)
{
    if (WebSocketServer::GetInstance().sid.empty()) {
        return true;  // SID 为空时完全绕过!
    }
    // ... SID 比较
}
```

**问题**:
1. SID 为空时完全跳过验证
2. 使用普通字符串比较（时序攻击）
3. 无失败次数限制

**触发路径**:
```
任意进程 → 连接 WebSocket 端口 → sid 为空配置 → 认证绕过
```

**影响**: 可伪造预览图像

**修复建议**:
1. 生产环境要求必须配置 SID
2. 使用常量时间比较
3. 添加连接失败速率限制

---

### HIGH: 命名管道无认证

**证据**: `cli/CommandLineInterface.cpp:73` 及 `util/LocalSocket.cpp`

```cpp
void CommandLineInterface::ProcessCommand()
{
    std::string message;
    LocalSocket socket;
    socket.Read(message);  // 无认证直接读取
}
```

**问题**: 任意用户进程可写入命名管道

**触发路径**:
```
任意本地进程 → 写入命名管道 → ProcessCommandMessage() → 执行命令
```

**影响**: 可注入任意命令（加载恶意页面、执行 JS）

**修复建议**:
1. 使用进程身份验证 (`getpeerid()`)
2. 实现命令消息 HMAC 认证
3. 限制管道访问权限

---

### MEDIUM: JSON 解析无限制

**证据**: `util/JsonReader.cpp:639-642`

```cpp
Json2::Value JsonReader::ParseJsonData2(const std::string& jsonStr)
{
    return Json2::Value(cJSON_Parse(jsonStr.c_str()));
}
```

**问题**: 无 JSON 大小/深度限制

**触发路径**:
```
DevEco Studio → 超大/深层嵌套 JSON → 解析崩溃
```

**影响**: 拒绝服务 (DoS)

**修复建议**: 添加 JSON 大小限制 (如 1MB) 和深度限制

---

### MEDIUM: TOCTOU 竞态条件

**证据**: `util/FileSystem.cpp:34-41`

```cpp
bool FileSystem::IsFileExists(std::string path)
{
    return S_ISREG(GetFileMode(path));
}
```

**问题**: `stat()` 检查和文件使用之间存在时间窗口

**触发路径**:
```
检查文件 → 符号链接替换 → 打开恶意链接
```

**影响**: 符号链接攻击

**修复建议**: 使用文件描述符和 `O_NOFOLLOW` 标志

---

### MEDIUM: 内存分配错误处理

**证据**: `jsapp/rich/external/StageContext.cpp:242-247`

```cpp
std::vector<uint8_t> *buf = new(std::nothrow) std::vector<uint8_t>(opt.value());
if (!buf) {
    ELOG("Memory allocation failed: buf.");
}
hspBufferPtrsVec.push_back(buf);  // 可能推入 nullptr
```

**问题**: 分配失败后继续执行，推入 nullptr

**触发路径**:
```
内存不足 → new 返回 nullptr → 后续访问 nullptr 指针
```

**影响**: 崩溃 / 任意代码执行

**修复建议**: 分配失败时立即返回错误

---

## CRITICAL: 命令注入风险 (automock 模块)

**类型**: 命令注入 (Command Injection)
**证据**: `automock/mock-generate/build.js:17-33`
**风险等级**: **CRITICAL**

**问题代码**:
```javascript
const { spawnSync } = require('child_process');
const bat = spawnSync(`
${path.join(__dirname, '..', nodeDir)} ${path.join(__dirname, '..', './node_modules/typescript/bin/tsc')} &&
${path.join(__dirname, '..', nodeDir)} ${path.join(__dirname, 'dist')}/main.js ${apiInputPath} &&
${path.join(__dirname, '..', './node_modules/eslint/bin/eslint.js')} -c .eslintrc --fix ${mockJsPath}/**/*.js`, {
  cwd: __dirname,
  shell: true  // ⚠️ 直接拼接用户输入，shell 解释执行
});
```

**触发条件**: 
- `process.argv[2]` (apiInputPath) 被直接拼接到 shell 命令中
- 攻击者可通过构造特殊路径如 `; rm -rf /` 执行任意系统命令

**影响**: 
- 任意系统命令执行
- 系统文件删除/篡改
- 可能导致 RCE (Remote Code Execution)

**修复建议**:
1. 禁用 `shell: true`，使用数组形式传递命令参数
2. 对 `apiInputPath` 使用 `path.resolve()` 规范化
3. 添加路径白名单验证

**修复示例**:
```javascript
// 不安全
spawnSync(`command ${userInput}`, { shell: true });

// 安全
spawnSync('command', [path.resolve(userInput)]);
```

---

## HIGH: 路径遍历风险 - systemUtils (automock 模块)

**类型**: 路径遍历 (Path Traversal)
**证据**: `automock/mock-generate/src/common/systemUtils.ts:23-31`
**风险等级**: **HIGH**

**问题代码**:
```typescript
export function getProjectDir(): string {
  try {
    const apiInputPath = process.argv[paramIndex];  // ⚠️ 用户输入
    const apiDir = path.join('interface', 'sdk-js', 'api');
    return apiInputPath.replace(`${path.sep}${apiDir}`, '');  // ⚠️ 未验证
  } catch (error) {
    throw new Error('OpenHarmony项目路径获取失败');
  }
}
```

**触发条件**: 
- 攻击者可使用 `../../../etc/passwd` 等路径访问系统敏感文件
- 未对 `apiInputPath` 进行路径验证

**影响**: 
- 任意文件读取
- 敏感信息泄露
- 可能导致代码注入

**修复建议**:
1. 使用 `path.resolve()` 规范化路径
2. 使用 `path.isAbsolute()` 检查是否为绝对路径
3. 添加路径白名单验证

**修复示例**:
```typescript
export function getProjectDir(): string {
  const apiInputPath = path.resolve(process.argv[paramIndex]);
  
  // 白名单验证
  if (!isValidPath(apiInputPath)) {
    throw new Error('Invalid project path');
  }
  
  // 进一步处理
  // ...
}
```

---

## HIGH: 路径遍历风险 - 文件操作 (automock 模块)

**类型**: 路径遍历 (Path Traversal)
**证据**: `automock/mock-generate/src/main.ts:68-81`
**风险等级**: **HIGH**

**问题代码**:
```typescript
function collectFile(dir: string, fileList: string[], value: string, isHmsDtsFile: boolean): void {
  const fullPath = path.join(dir, value);  // ⚠️ 直接拼接路径
  const stats = fs.statSync(fullPath);
  if (stats.isDirectory()) {
    getAllDtsFile(fullPath, fileList, isHmsDtsFile);
  }
}
```

**触发条件**: 
- `value` 参数可能包含 `../` 等路径遍历字符
- 未对 `dir` 或 `value` 进行路径验证

**影响**: 
- 越权访问任意文件系统路径
- 敏感信息泄露
- 文件系统遍历攻击

**修复建议**:
1. 使用 `path.normalize()` 规范化路径
2. 验证规范化后的路径是否在预期目录内
3. 使用 `path.relative()` 检查路径是否越界

**修复示例**:
```typescript
function collectFile(dir: string, fileList: string[], value: string, isHmsDtsFile: boolean): void {
  const fullPath = path.join(dir, value);
  const normalizedPath = path.normalize(fullPath);
  
  // 验证路径是否在预期目录内
  if (!normalizedPath.startsWith(path.normalize(dir))) {
    throw new Error('Path traversal attempt detected');
  }
  
  const stats = fs.statSync(normalizedPath);
  // ...
}
```

---

## MEDIUM: 动态代码执行风险 - Function 构造 (automock 模块)

**类型**: 代码注入 (Code Injection)
**证据**: `automock/mock-generate/src/generate/generateContent.ts:708`
**风险等级**: **MEDIUM**

**问题代码**:
```typescript
function findInLibFunction(
  key: string,
  targetKeyValue: KeyValue,
  mockBuffer: MockBuffer,
  kvPath: KeyValue[],
  rootKeyValue: KeyValue
): ReferenceFindResult {
  const params = handleParams(targetKeyValue.methodParams, mockBuffer, kvPath, rootKeyValue);
  let value: string;
  if (typeof global[key].constructor === 'function') {
    value = key === 'Function' ? '() => {}' : `new ${key}(${params})`;  // ⚠️ 动态构造
  }
  // ...
}
```

**触发条件**: 
- `params` 参数可能包含用户控制的值
- 使用字符串拼接构造 `new Function()` 调用

**影响**: 
- 可能导致任意代码执行
- 如果 `params` 来自不可信源，攻击者可注入恶意代码

**修复建议**:
1. 对 `params` 进行严格的类型和格式验证
2. 使用白名单限制 `key` 的可能值
3. 避免动态构造代码，使用预定义的函数映射

**修复示例**:
```typescript
function findInLibFunction(...): ReferenceFindResult {
  const params = handleParams(...);
  
  // 白名单验证
  if (!ALLOWED_FUNCTION_KEYS.has(key)) {
    throw new Error(`Unsupported function key: ${key}`);
  }
  
  // 使用预定义映射而非动态构造
  const value = FUNCTION_MAP.get(key);
  // ...
}
```

---

## MEDIUM: 代码注入风险 - 字符串模板 (automock 模块)

**类型**: 代码注入 (Code Injection)
**证据**: `automock/mock-generate/src/generate/generateContent.ts:870-885`
**风险等级**: **MEDIUM**

**问题代码**:
```typescript
const returnInfo = isSpecial
  ? callBackParams.join(', ')
  : funType === 'single'
    ? callBackParams.join(', ')
    : returnData?.join(', ');
const data = `if (args && typeof args[args.length - 1] === 'function') {
  args[args.length - 1].call(this, ${isAsyncCallback ? callbackError : ''}${returnInfo});
}`;
```

**触发条件**: 
- `returnInfo` 和 `callBackParams` 未经过滤直接拼接到代码字符串
- 若参数来源可控，可能注入恶意代码

**影响**: 
- 任意代码执行
- 生成的 Mock 代码包含恶意逻辑

**修复建议**:
1. 对参数进行转义（如使用 JSON.stringify）
2. 使用模板引擎而非字符串拼接
3. 对所有参数来源进行验证

**修复示例**:
```typescript
// 不安全
const data = `args[${index}].call(this, ${param})`;

// 安全
const data = `args[${index}].call(this, ${JSON.stringify(param)})`;
```

---

## MEDIUM: 缺乏输入验证 (automock 模块)

**类型**: 输入验证缺失
**风险等级**: **MEDIUM**

**问题位置**: 
- `automock/mock-generate/src/main.ts` - `process.argv` 无验证
- `automock/mock-generate/src/common/commonUtils.ts:88-103` - `importPath` 无验证
- `automock/mock-generate/src/generate/generateContent.ts` - 多处动态代码生成无参数校验

**问题描述**: 
- 无输入长度验证
- 无类型严格检查
- 无范围验证

**触发条件**: 
- 超长输入可能导致内存耗尽
- 非预期类型可能导致类型混淆攻击
- 超出范围的值可能导致整数溢出

**修复建议**:
1. 对所有用户输入添加长度限制
2. 使用 TypeScript 严格模式进行类型检查
3. 添加数值范围验证

**修复示例**:
```typescript
// 不安全
const path = process.argv[2];  // 无任何验证

// 安全
const MAX_PATH_LENGTH = 256;
const path = process.argv[2];
if (path && path.length > MAX_PATH_LENGTH) {
  throw new Error('Path too long');
}
```

---

## LOW: 正则表达式注入风险 (automock 模块)

**类型**: ReDoS (Regular Expression Denial of Service)
**证据**: `automock/mock-generate/src/common/commonUtils.ts:202-203`
**风险等级**: **LOW**

**问题代码**:
```typescript
const regForStatic = /\*\s*@since\s*.*(static|staticonly)\b/ig;
const regForDynamic = /\*\s*@since\s*.*(dynamic|dynamiconly)\b/ig;

if (text.includes('@arkts 1.2') || text.match(regForStatic) && !text.match(regForDynamic)) {
  // ...
}
```

**触发条件**: 
- 使用通配符 `.*` 可能导致 ReDoS 攻击
- 若 `text` 内容可控且超长，可能消耗大量 CPU 资源

**影响**: 
- CPU 耗尽
- 服务拒绝

**修复建议**:
1. 使用更精确的正则表达式，避免过度回溯
2. 添加正则表达式执行超时
3. 限制输入文本的最大长度

**修复示例**:
```typescript
// 不安全
const regForStatic = /\*\s*@since\s*.*(static|staticonly)\b/ig;

// 安全（避免贪婪匹配）
const regForStatic = /\*\s*@since\s+[^\s]*?(static|staticonly)\b/ig;
```

---

## MEDIUM: 文件写入风险 (automock 模块)

**类型**: 任意文件写入
**证据**: `automock/mock-generate/src/main.ts:247`
**风险等级**: **MEDIUM**

**问题代码**:
```typescript
fs.writeFileSync(mockBuffer.mockedFilePath, mockedContents);
```

**触发条件**: 
- `mockedFilePath` 由用户输入的 `process.argv[2]` 派生
- 未验证目标路径是否在预期目录范围内

**影响**: 
- 可能导致任意文件覆盖
- 写入恶意文件

**修复建议**:
1. 验证目标路径是否在预期目录范围内
2. 使用 `path.resolve()` 规范化路径
3. 添加路径白名单验证

**修复示例**:
```typescript
const outputDir = path.join(__dirname, 'output');
const fullPath = path.resolve(outputDir, fileName);

// 验证路径是否在输出目录内
if (!fullPath.startsWith(path.resolve(outputDir))) {
  throw new Error('Invalid output path');
}

fs.writeFileSync(fullPath, contents);
```

---

## LOW: 原型链污染风险 (automock 模块)

**类型**: 原型链污染 (Prototype Pollution)
**证据**: `automock/mock-generate/src/common/tsNodeUtils.ts:135-142`
**风险等级**: **LOW**

**问题代码**:
```typescript
function handleClassElement(
  node: ts.ClassElement,
  mockBuffer: MockBuffer,
  members: Members,
  parent: KeyValue,
  type: KeyValueTypes
): void {
  // ...
  members[memberKey] = generateKeyValue(memberKey, type, parent);
```

**触发条件**: 
- `memberKey` 可能为 `__proto__` 或 `constructor`
- 可能导致原型链污染攻击

**影响**: 
- 原型链污染
- 可能影响后续代码执行

**修复建议**:
1. 使用 `Object.create(null)` 创建无原型对象
2. 过滤特殊键名（如 `__proto__`, `constructor`, `prototype`）

**修复示例**:
```typescript
// 不安全
const members = {};
members[key] = value;

// 安全
const members = Object.create(null);
if (['__proto__', 'constructor', 'prototype'].includes(key)) {
  throw new Error('Invalid member key');
}
members[key] = value;
```

---

## Automock 模块安全风险总结

| 风险 | 级别 | 位置 | 修复优先级 |
|------|------|------|-----------|
| R1: 命令注入 | **CRITICAL** | build.js:17-33 | **P0 - 立即** |
| R2: 路径遍历 | **HIGH** | systemUtils.ts:23-31 | **P0 - 立即** |
| R3: 路径遍历 | **HIGH** | main.ts:68-81 | **P0 - 立即** |
| R4: 动态代码执行 | **MEDIUM** | generateContent.ts:708 | **P1 - 高** |
| R5: 代码注入 | **MEDIUM** | generateContent.ts:870-885 | **P1 - 高** |
| R6: 缺乏输入验证 | **MEDIUM** | 多处 | **P1 - 高** |
| R7: ReDoS | **LOW** | commonUtils.ts:202-203 | **P2 - 中** |
| R8: 文件写入 | **MEDIUM** | main.ts:247 | **P1 - 高** |
| R9: 原型链污染 | **LOW** | tsNodeUtils.ts:135-142 | **P2 - 中** |

### Automock 修复优先级

**P0 - 立即修复 (1 周内)**:
1. 修复 build.js 中的命令注入（禁用 shell:true）
2. 修复所有路径遍历风险（添加路径白名单验证）

**P1 - 高优先级 (2 周内)**:
3. 修复动态代码执行风险（参数转义）
4. 修复代码注入风险（使用模板引擎）
5. 添加输入验证（长度、类型、范围）
6. 修复文件写入风险（路径白名单）

**P2 - 中优先级 (1 个月内)**:
7. 修复 ReDoS 风险（优化正则表达式）
8. 修复原型链污染风险（过滤特殊键名）

---

## 已验证安全机制

### ✅ 存在

| 机制 | 实现位置 | 描述 |
|------|----------|------|
| 命令参数长度检查 | `CommandParser.cpp` | `maxMainArgLength` 限制 |
| 端口范围验证 | `CommandParser.cpp:652-667` | WebSocket 端口 1024-65535 |
| 坐标范围验证 | `CommandLine.cpp` | 宽度/高度 50-3000 |
| KeyCode 范围验证 | `CommandLine.cpp` | 2000-2119 |
| 内存分配 noexcept | 多处 `new(std::nothrow)` | 防止异常 |
| 互斥锁保护 | `VirtualScreen.cpp` | WebSocket 缓冲区 |

### ❌ 缺失

| 缺失项 | 风险 |
|--------|------|
| 消息 HMAC 认证 | 命令注入 |
| SID 强制验证 | 认证绕过 |
| JSON 模式验证 | 格式错误处理 |
| 文件路径规范化 | 路径遍历 |
| 提取路径验证 | Zip Slip |

---

## 安全加固建议

### 立即执行 (P0)

1. **修复路径遍历**
   - 实施路径规范化后再验证
   - 白名单机制限制可访问目录

2. **修复 Zip Slip**
   - 验证解压后路径在目标目录内

3. **修复 WebSocket SID**
   - 删除空 SID 绕过
   - 添加速率限制

### 短期执行 (P1)

1. **添加消息认证**
   ```cpp
   // HMAC 签名验证
   if (!VerifyHmac(message, secret)) {
       return ERROR_AUTH_FAILED;
   }
   ```

2. **添加 JSON 限制**
   ```cpp
   if (jsonStr.size() > MAX_JSON_SIZE) {
       return ERROR_JSON_TOO_LARGE;
   }
   ```

3. **统一内存管理**
   - 使用 RAII 封装所有动态分配
   - 禁止混用 new/delete 和 malloc/free

### 长期执行 (P2)

1. **进程身份验证**
   - 验证管道连接方 PID/UID
   - 使用 Linux SO_PEERCRED

2. **安全编译选项**
   - `-fstack-protector-strong`
   - `-D_FORTIFY_SOURCE=2`
   - `-Wl,-z,relro,-z,now`

3. **安全测试**
   - 模糊测试 JSON 解析器
   - 模糊测试命令参数
   - 渗透测试

---

## 检查范围声明

本安全评审的检查范围：

| 范围 | 包含 |
|------|------|
| `cli/` | ✅ 完整检查 |
| `jsapp/` | ✅ 完整检查 |
| `mock/` | ✅ 完整检查 |
| `util/` | ✅ 完整检查 |
| `gn/` | ✅ 配置检查 |
| `test/` | ❌ 未检查 |
| 外部依赖仓库 | ❌ 未检查 |

---

## 相关文档

- 架构: [02_Architecture.md](./02_Architecture.md)
- 通信协议: [03_Communication_Protocol.md](./03_Communication_Protocol.md)
- 编译产物: [06_Artifacts.md](./06_Artifacts.md)

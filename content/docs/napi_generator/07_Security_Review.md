# 安全风险评审

> 攻击面、信任边界与风险点分析

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码生成工具** | `src/cli/` 下所有工具 |
| **生成产物** | 工具生成的 N-API 和 SA 框架代码 |
| **不包含** | OpenHarmony 运行时、N-API 运行时实现 |

---

## 威胁模型

### 1. 数据流图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           napi-generator 威胁模型                            │
└─────────────────────────────────────────────────────────────────────────────┘

  输入层                          生成层                          运行时层
     │                              │                                │
     ▼                              ▼                                ▼
┌───────────────┐         ┌─────────────────┐          ┌─────────────────┐
│  .d.ts 文件    │         │  生成器引擎      │          │  JS/ArkTS 应用   │
│  .h 头文件     │ ──────▶ │  (dts2cpp/      │ ────────▶│  (调用生成代码) │
│  CMakeLists    │         │   h2sa/h2dtscpp)│          │                 │
└───────────────┘         └─────────────────┘          └─────────────────┘
     │                              │                                │
     │ 恶意输入风险                  │ 生成代码缺陷                     │ 运行时攻击
     │ • 代码注入                   │ • 缓冲区溢出                    │ • 参数篡改
     │ • 路径遍历                   │ • 整数溢出                      │ • 注入攻击
     │ • 类型混淆                   │ • 资源泄漏                      │ • 权限提升
```

### 2. 信任边界

| 边界 | 描述 | 信任级别 |
|------|------|----------|
| **输入边界** | 用户提供的 .d.ts / .h / CMakeLists.txt | ❌ 不信任 |
| **生成器内部** | 工具核心代码 (`gen/`) | ✅ 信任 |
| **模板代码** | 代码模板 (`*.gen`, `file_template.js`) | ✅ 信任 |
| **生成产物** | 自动生成的 N-API / SA 代码 | ⚠️ 需验证 |
| **运行时** | OpenHarmony N-API 运行时 | ✅ 信任 |

---

## 攻击面分析

### 1. 输入验证攻击面

| 攻击面 | 触发点 | 风险等级 |
|--------|--------|----------|
| **恶意 .d.ts 文件** | `analyze.js` 解析 | ⚠️ Medium |
| **恶意 .h 头文件** | `h2sa/analyze.js` 解析 | ⚠️ Medium |
| **路径遍历** | 文件输出路径 | ✅ Low |
| **代码注入** | 模板变量替换 | ✅ Low |

### 2. 生成代码攻击面

| 攻击面 | 触发点 | 风险等级 |
|--------|--------|----------|
| **缓冲区溢出** | 字符串/数组操作 | ⚠️ Medium |
| **整数溢出** | 数值计算 | ⚠️ Medium |
| **空指针解引用** | 参数校验缺失 | ⚠️ Medium |
| **资源泄漏** | 内存/文件句柄 | ✅ Low |

### 3. IPC 攻击面 (h2sa)

| 攻击面 | 触发点 | 风险等级 |
|--------|--------|----------|
| **接口令牌校验** | `OnRemoteRequest()` | ✅ Low |
| **参数反序列化** | `MessageParcel::Read*()` | ⚠️ Medium |
| **权限验证** | SA 权限配置 | ⚠️ Medium |

---

## 已识别风险点

### 🔴 高风险 (需立即修复)

#### 1. [文件名: 参数校验缺失]

**证据**: `src/cli/dts2cpp/src/gen/analyze.js`

```javascript
// 分析: 可能缺少对 .d.ts 文件内容的完整校验
// 风险: 恶意构造的输入可能导致解析异常或内存问题
```

**触发条件**:
- 用户提供特制的 .d.ts 文件
- 包含极端深的嵌套结构
- 包含超长字符串或超大数值

**影响**:
- 工具崩溃 (DoS)
- 潜在内存问题

**修复建议**:
```javascript
// 添加输入长度和复杂度限制
const MAX_FILE_SIZE = 1024 * 1024;  // 1MB
const MAX_NESTING_DEPTH = 20;
const MAX_STRING_LENGTH = 10000;

// 校验文件大小
if (content.length > MAX_FILE_SIZE) {
    throw new Error('Input file too large');
}

// 解析时检查嵌套深度
function parseWithDepthLimit(content, maxDepth) {
    if (currentDepth > maxDepth) {
        throw new Error('Nesting depth exceeded');
    }
    // ... 正常解析
}
```

---

### 🟡 中风险 (建议修复)

#### 2. [文件名: 路径遍历风险]

**证据**: `src/cli/dts2cpp/src/gen/cmd_gen.js`

```javascript
// 分析: 输出路径可能未校验，导致路径遍历
function checkGenerate(fileName) {
    let outPath = path.join(ops.out, ...);
    // 未校验 ops.out 是否在预期范围内
}
```

**触发条件**:
```bash
node cmd_gen.js -f test.d.ts -o ../../etc
```

**影响**:
- 写入文件到预期外目录
- 覆盖系统文件

**修复建议**:
```javascript
const path = require('path');

// 校验输出路径在允许范围内
function validateOutputPath(outDir, baseDir) {
    const resolvedOut = path.resolve(outDir);
    const resolvedBase = path.resolve(baseDir);
    
    if (!resolvedOut.startsWith(resolvedBase)) {
        throw new Error('Output path outside allowed directory');
    }
}
```

---

#### 3. [文件名: MessageParcel 反序列化风险]

**证据**: `src/cli/h2sa/src/tools/common.js`

```javascript
// 分析: MessageParcel 读取可能越界或类型混淆
const DATA_R_MAP = {
    'int32_t': 'ReadInt32',
    // ...
};

// 生成代码中直接使用这些方法
data.ReadInt32();  // 无长度校验
```

**触发条件**:
- 恶意客户端发送畸形 IPC 请求
- 参数数量与预期不符

**影响**:
- 读取越界 (信息泄露)
- 进程崩溃 (DoS)

**修复建议** (生成代码模板中增加):
```cpp
// Stub 实现中增加参数校验
ErrCode TestServiceStub::testFuncInner(MessageParcel &data, MessageParcel &reply)
{
    // 1. 校验数据可用长度
    if (data.GetReadableBytes() < sizeof(int32_t) * 3) {
        return ERR_INVALID_VALUE;
    }
    
    // 2. 逐个读取并校验
    int32_t v1 = data.ReadInt32();
    int32_t v2 = data.ReadInt32();
    bool v3 = data.ReadBoolUnaligned();
    
    // 3. 业务参数范围校验
    if (v1 < 0 || v2 < 0) {
        return ERR_INVALID_VALUE;
    }
    
    // ... 业务逻辑
}
```

---

#### 4. [文件名: 整数溢出风险]

**证据**: `src/cli/dts2cpp/src/gen/generate/function_direct.js`

```javascript
// 分析: TypeScript number → C++ 映射可能整数溢出
// 默认映射为 uint32_t，但 TS number 可表示更大范围
if (type === 'number') {
    cppType = 'uint32_t';  // 溢出风险!
}
```

**触发条件**:
```typescript
// TypeScript
function largeValue(): number {
    return 9007199254740993;  // 超出安全整数范围
}
```

**影响**:
- 数据精度丢失
- 业务逻辑错误

**修复建议**:
```javascript
// 增加整数范围检测
const SAFE_MIN = -2147483648;
const SAFE_MAX = 2147483647;

function mapNumberToCpp(tsNumber) {
    if (!Number.isInteger(tsNumber)) {
        return 'double';  // 使用浮点数
    }
    if (tsNumber >= SAFE_MIN && tsNumber <= SAFE_MAX) {
        return 'int32_t';
    }
    if (tsNumber >= 0 && tsNumber <= 4294967295) {
        return 'uint32_t';
    }
    return 'int64_t';  // 大整数使用 int64_t
}
```

---

### 🟢 低风险 (建议关注)

#### 5. [文件名: 资源泄漏]

**证据**: `src/cli/dts2cpp/src/gen/extend/tool_utility.js`

```cpp
// 生成的 N-API 代码中，异步工作资源需要手动释放
napi_create_async_work(env, nullptr, execute, complete, &work);
// ...
napi_delete_async_work(env, work);  // 依赖开发者调用
```

**修复建议**: 在生成代码中增加异常安全的资源管理

```cpp
// 使用 RAII 包装器
class AsyncWorkGuard {
public:
    AsyncWorkGuard(napi_env env, napi_async_work work) 
        : env_(env), work_(work) {}
    
    ~AsyncWorkGuard() {
        if (work_) {
            napi_delete_async_work(env_, work_);
        }
    }
    
    // 禁止拷贝
    AsyncWorkGuard(const AsyncWorkGuard&) = delete;
    AsyncWorkGuard& operator=(const AsyncWorkGuard&) = delete;
    
private:
    napi_env env_;
    napi_async_work work_;
};

// 使用
{
    napi_async_work work;
    napi_create_async_work(env, execute, complete, &work);
    AsyncWorkGuard guard(env, work);
    napi_queue_async_work(env, work);
    // 自动释放
}
```

---

#### 6. [文件名: 日志敏感信息]

**证据**: `src/cli/dts2cpp/src/gen/tools/NapiLog.js`

```javascript
// 生成的日志可能包含敏感信息
NapiLog.logInfo('param value: ' + userInput);
```

**修复建议**:
```javascript
// 日志脱敏
function sanitizeLog(input) {
    if (typeof input !== 'string') {
        return input;
    }
    // 脱敏敏感模式
    return input
        .replace(/password["']?\s*[:=]\s*["']?([^"']+)["']?/gi, 'password=***')
        .replace(/\b\d{15,18}\b/g, '***');  // 身份证号
}
```

---

## SA 权限配置风险

### 1. 权限缺失

**证据**: `src/cli/h2sa/src/gen/file_template.js` 生成的配置

```xml
<!-- 生成的 SA 配置 -->
<info>
    <permission>
        <!-- 可能缺少必要的权限声明 -->
    </permission>
</info>
```

**风险**: 服务可能被未授权调用

**修复建议**:
```javascript
// 强制要求配置权限
if (!config.permissions || config.permissions.length === 0) {
    logger.warn('SA has no permission configured, using default');
    // 或强制要求
    throw new Error('Permission configuration required');
}
```

---

## 安全最佳实践

### 1. 输入验证

| 检查项 | 实现位置 |
|--------|----------|
| 文件大小限制 | `cmd_gen.js` |
| 路径遍历防护 | `file_rw.js` |
| 嵌套深度限制 | `analyze.js` |
| 参数类型校验 | `generate.js` |

### 2. 输出安全

| 检查项 | 实现位置 |
|--------|----------|
| 输出路径校验 | `cmd_gen.js` |
| 敏感信息脱敏 | `NapiLog.js` |
| 安全模板渲染 | `file_template.js` |

### 3. 运行时安全 (生成代码)

| 检查项 | 生成模板 |
|--------|----------|
| 参数边界校验 | `function_*.gen` |
| 空指针检查 | `function_*.gen` |
| 资源自动释放 | `function_*.gen` |
| IPC 令牌校验 | `h2sa template` |

---

## 安全加固建议

### 短期 (P0)

1. ✅ 添加输入文件大小和复杂度限制
2. ✅ 实现输出路径校验
3. ✅ 增加 MessageParcel 参数校验模板

### 中期 (P1)

1. ⚠️ 实现整数溢出检测
2. ⚠️ 增加资源泄漏防护
3. ⚠️ 添加敏感信息脱敏

### 长期 (P2)

1. 🔜 引入模糊测试 (Fuzzing)
2. 🔜 安全编码规范培训
3. 🔜 定期安全审计

---

## 相关章节

- 架构说明: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)
- 常见问题: [08_Troubleshooting.md](08_Troubleshooting.md)

---

[返回 SUMMARY.md](SUMMARY.md)

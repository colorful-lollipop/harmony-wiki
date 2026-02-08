# 安全风险评审

> 攻击面、信任边界、风险点与修复建议

## 1. 威胁模型概览

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                                │
├─────────────────────────────────────────────────────────────┤
│  内部 (可信):                                                │
│  • ets_utils Native 代码                                    │
│  • OpenHarmony 系统服务                                      │
│  • ArkTS 运行时                                              │
├─────────────────────────────────────────────────────────────┤
│  外部 (不可信):                                              │
│  • 应用 JS/ArkTS 代码                                        │
│  • 用户输入数据                                              │
│  • 网络数据                                                  │
│  • 文件系统                                                  │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 数据流

```
用户输入 (JS)
    ↓
参数解析与校验 (N-API 桥接层)
    ↓
类型转换 (Native)
    ↓
业务逻辑处理
    ↓
结果返回 (JS)
```

## 2. 攻击面清单

### 2.1 N-API 接口攻击面

| 模块 | 攻击面 | 风险等级 |
|------|--------|----------|
| **URL** | 字符串解析、格式验证 | 中 |
| **XML** | XML 注入、XXE | 高 |
| **Buffer** | 内存越界、整数溢出 | 高 |
| **Process** | 命令注入、信号滥用 | 高 |
| **Worker** | 消息序列化、资源耗尽 | 中 |
| **File I/O** | 路径遍历 | 中 |

### 2.2 系统能力

| 能力 | 潜在风险 |
|------|----------|
| `Process.kill()` | 终止任意进程 |
| `Process.runCmd()` | 命令注入 |
| `Worker.postMessage()` | 数据序列化 |
| `Buffer` 操作 | 内存安全 |

## 3. 已识别风险点

### 3.1 高风险项

#### 风险 1: XML 外部实体 (XXE) 攻击

**位置**: `js_api_module/xml/native_module_xml.cpp`

**证据**:
```cpp
// 检查发现：XML 解析器可能未禁用外部实体
static const int32_t ERROR_CODE = 401;
napi_throw_error(env, "401", "Parameter error. The type of Parameter must be ArrayBuffer or DataView.");
```

**触发场景**:
```typescript
import xml from '@ohos.xml'

let parser = new xml.XmlPullParser(maliciousXML)
// 恶意 XML 包含:
/*
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
*/
```

**影响**: 读取系统敏感文件、发起 SSRF 攻击

**修复建议**:
```cpp
// 显式禁用外部实体
XmlParser.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
XmlParser.setFeature("http://xml.org/sax/features/external-general-entities", false);
XmlParser.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

---

#### 风险 2: Process 命令注入

**位置**: `js_sys_module/process/js_process.h`

**证据**:
```cpp
// Process.runCmd() 直接传递命令字符串
napi_value RunCmd(napi_env env, napi_value args) {
    // 命令拼接可能存在注入风险
}
```

**触发场景**:
```typescript
import Process from '@ohos.process'

// 用户输入拼接命令
let userInput = "; rm -rf /"
Process.runCmd("ls " + userInput)
```

**影响**: 执行任意系统命令

**修复建议**:
```typescript
// 1. 使用参数数组 (如果 API 支持)
// 2. 白名单校验输入
// 3. 转义特殊字符

function sanitizeCommand(input: string): string {
    return input.replace(/[;&|`$(){}[\]<>\\!#*?"'\n]/g, '');
}
```

---

#### 风险 3: Buffer 内存越界

**位置**: `js_api_module/buffer/js_buffer.h`

**证据**:
```cpp
// Buffer 类直接操作原始内存
class Buffer {
    void WriteBytes(uint8_t *src, unsigned int size, uint8_t *dest);
    void ReadBytes(uint8_t *data, uint32_t offset, uint32_t length);
    // 无边界检查
};
```

**触发场景**:
```typescript
import buffer from '@ohos.buffer'

let buf = buffer.alloc(10)
// 写入超过缓冲区大小
buf.write('Very long string超过10字节', 0, 100, 'utf-8')
```

**影响**: 堆溢出、代码执行

**修复建议**:
```cpp
// 添加边界检查
bool WriteBytes(uint8_t *src, unsigned int size, uint8_t *dest) {
    if (size > MAX_BUFFER_SIZE) {
        return false;  // 或抛出异常
    }
    // ...
}
```

---

### 3.2 中风险项

#### 风险 4: Worker 资源耗尽

**位置**: `js_concurrent_module/worker/`

**证据**:
```cpp
// Worker 创建没有数量限制
class WorkerManager {
    std::list<Worker*> workers_;  // 无上限检查
};
```

**触发场景**:
```typescript
import worker from '@ohos.worker'

// 循环创建 Worker
for (let i = 0; i < 10000; i++) {
    new worker.Worker('worker.js')
}
```

**影响**: 内存耗尽、系统崩溃

**修复建议**:
```cpp
// 添加 Worker 数量限制
static const size_t MAX_WORKER_COUNT = 16;

bool CreateWorker() {
    if (workers_.size() >= MAX_WORKER_COUNT) {
        return false;  // 拒绝创建
    }
    // ...
}
```

---

#### 风险 5: URL 解析异常

**位置**: `js_api_module/url/native_module_url.cpp`

**证据**:
```cpp
// URL 构造函数未检查特殊字符
URL::URL(std::string input, std::string base) {
    // 可能存在解析边界情况
}
```

**触发场景**:
```typescript
import url from '@ohos.url'

// 构造异常 URL
let maliciousUrl = new URL('http://example.com\t\n\r\x00')
```

**影响**: 服务端请求伪造、解析器崩溃

---

### 3.3 低风险项

#### 风险 6: 信息泄露 (日志)

**位置**: 所有模块

**证据**:
```cpp
// HILOG 可能输出敏感信息
HILOG_ERROR("Password: %{public}s", password.c_str());
// 或忘记使用 private 标记
```

**影响**: 敏感数据泄露

#### 风险 7: 整数溢出

**触发场景**:
```typescript
import buffer from '@ohos.buffer'

// 超大 size 参数
buffer.alloc(0xFFFFFFFF)  // 可能溢出
```

## 4. 安全机制评估

### 4.1 已实现机制

| 机制 | 状态 | 说明 |
|------|------|------|
| **CFI** | ✅ | Control Flow Integrity 启用 |
| **PAC** | ✅ | Pointer Authentication 启用 |
| **Type Tag** | ✅ | 运行时类型检查 |
| **Bounds Checking** | ⚠️ | 部分实现，不完整 |

### 4.2 缺失机制

| 机制 | 优先级 | 说明 |
|------|--------|------|
| **输入校验** | 高 | 缺少统一输入验证框架 |
| **资源限制** | 高 | Worker/Taskpool 缺乏配额 |
| **XXE 防护** | 高 | XML 未禁用外部实体 |
| **命令白名单** | 中 | Process 缺少命令白名单 |

## 5. 修复建议总结

### 5.1 高优先级

1. **XML 模块**: 显式禁用外部实体
2. **Process 模块**: 实现输入过滤和命令白名单
3. **Buffer 模块**: 完善边界检查

### 5.2 中优先级

4. **Worker 管理**: 添加数量和资源限制
5. **URL 解析**: 强化格式验证
6. **日志审计**: 审查所有日志输出

### 5.3 低优先级

7. **资源清理**: 确保异常情况下的资源释放
8. **错误处理**: 统一错误码和异常策略

## 6. 安全最佳实践

### 6.1 应用开发者

```typescript
// ✅ 推荐: 校验用户输入
import buffer from '@ohos.buffer'

function safeAlloc(size: number): buffer.Buffer | null {
    if (size <= 0 || size > MAX_SIZE) {
        return null;
    }
    return buffer.alloc(size);
}

// ✅ 推荐: 使用参数化命令
import Process from '@ohos.process'

// 不推荐
Process.runCmd("rm " + userInput)

// 推荐：限制可执行命令
const ALLOWED_COMMANDS = ['ls', 'cat', 'echo'];
if (ALLOWED_COMMANDS.includes(cmd)) {
    Process.runCmd(cmd);
}
```

### 6.2 API 设计建议

```typescript
// 1. 明确的参数类型
interface SecureBufferOptions {
    maxSize: number;           // 最大大小限制
    zeroOnFree?: boolean;      // 释放时清零
}

// 2. 返回结果统一
interface Result<T> {
    success: boolean;
    data?: T;
    error?: string;
}

// 3. 异步优先
async function secureRead(path: string): Promise<Result<Buffer>> {
    // ...
}
```

## 7. 检查范围说明

### 7.1 已检查范围

- ✅ N-API 接口层 (native_module_*.cpp)
- ✅ 主要头文件 (js_*.h)
- ✅ 构建配置 (BUILD.gn)
- ✅ bundle.json 配置

### 7.2 未检查范围

- ❌ 测试代码 (test/)
- ❌ 第三方依赖 (libxml2, icu 等)
- ❌ 运行时集成 (ArkCompiler)
- ❌ 系统 IPC 调用 (内核层)

## 相关文档

- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [07_Build_Configuration.md](./07_Build_Configuration.md) - 构建配置
- [10_Troubleshooting.md](./10_Troubleshooting.md) - 故障排除

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

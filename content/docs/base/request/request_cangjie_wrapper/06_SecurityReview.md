# 安全风险评估

本文档对 `request_cangjie_wrapper` 进行安全风险评估，识别潜在漏洞并提供修复建议。

---

## 6.1 评估概述

### 评估范围

| 范围 | 说明 |
|------|------|
| 评估对象 | request_cangjie_wrapper N-API 封装层 |
| 评估方法 | 静态代码分析 |
| 依赖服务 | request 底层服务 (超出本层评估范围) |

### 风险评级标准

| 等级 | 评分 | 严重程度 |
|------|------|----------|
| 🔴 严重 | 9-10 | 可直接利用，需立即修复 |
| 🔴 高危 | 7-8 | 利用条件较简单，建议尽快修复 |
| 🟡 中危 | 4-6 | 利用条件复杂，建议修复 |
| 🟢 低危 | 1-3 | 影响有限，可接受 |

---

## 6.2 输入验证缺陷

### R1: URL 协议无限制 (中危)

**位置**: `agent.cj:596`

**证据**:
```cj
/**
 * The Universal Resource Locator for a task.
 * The maximum length is 8192 characters.
 * Using raw `url` option, even url parameters in it.
 */
public var url: String
```

**触发路径**:
```
用户代码: config.url = "ftp://attacker.com/malicious"
    → Task(config)
    → request 服务发起 FTP 请求
```

**影响**:
- 可发起任意协议请求 (FTP, SMB, etc.)
- 可访问内部网络服务
- 潜在 SSRF 攻击

**修复建议**:
```cj
public init(url: String) {
    if (!url.startsWith("http://") && !url.startsWith("https://")) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Only HTTP/HTTPS protocols are allowed")
    }
    this.url = url
}
```

---

### R2: 文件路径无规范化验证 (中危)

**位置**: `agent.cj:358`

**证据**:
```cj
/**
 * The path to save the uploaded file.
 * Currently support:
 * 1: relative path, like "./xxx/yyy/zzz.html"
 * 2: internal protocol path, starting with "internal://"
 */
public var path: String
```

**触发路径**:
```
用户代码: fileSpec.path = "../../../etc/passwd"
    → Task(config)
    → request 服务读取文件
    → 可能读取 /etc/passwd
```

**影响**:
- 路径遍历攻击
- 读取应用沙箱外文件
- 信息泄露

**修复建议**:
- 依赖 request 服务进行路径规范化
- 增加路径格式白名单检查
- 限制可访问的目录范围

---

### R3: HTTP 头注入风险 (低危)

**位置**: `agent.cj:661`

**证据**:
```cj
/**
 * The HTTP headers.
 */
public var headers: HashMap<String, String>
```

**触发路径**:
```
用户代码: headers["Content-Type"] = "text/html\r\n\r\nInjected Body"
    → HTTP 请求发送
    → 可能注入响应内容
```

**影响**:
- HTTP 响应拆分
- 缓存投毒
- 有限制的注入

**修复建议**:
```cj
// 过滤 HTTP 头部注入字符
func sanitizeHeader(value: String): String {
    return value.replace("\r\n", "").replace("\r", "").replace("\n", "")
}
```

---

## 6.3 内存安全问题

### R4: CString 转换风险 (低危)

**位置**: `ffi.cj:396-485`

**证据**:
```cj
struct CConfig {
    var url: CString
    var title: CString
    // ...
}
```

**分析**:
- FFI 层正确使用 `LibC.mallocCString()` 分配内存
- `asResource()` 确保 RAII 模式释放
- 已有异常处理保证资源释放

**风险**:
- ⚠️ 但未验证字符串长度限制
- ⚠️ 异常路径可能泄露资源

**证据来源**:
- `ffi.cj:424`: `LibC.mallocCString(config.url)`
- `ffi.cj:468`: `CTypeResource<CConfig>(this, free)`

---

### R5: 回调闭包内存管理 (低危)

**位置**: `agent.cj:1768-1771`

**证据**:
```cj
let wrapper = {
    progress: CProgress =>
    callback.invoke(None, Progress(progress))
    progress.free()  // 确保 CProgress 释放
}
```

**分析**:
- 正确释放 CProgress 资源
- 使用 `Callback1Param` 管理回调生命周期
- 未发现明显的内存泄漏

**风险**:
- ⚠️ 长期运行的回调可能导致累积

---

## 6.4 权限与鉴权

### R6: Token 安全机制 (中危)

**位置**: `agent.cj:799-804`

**证据**:
```cj
/**
 * For in-application layer isolation.
 * If given:
 *   the minimum is 8 bytes.
 *   the maximum is 2048 bytes.
 * Creates a task with token, then must provide it during normal query.
 * So saves the token carefully, it can not be retrieved by query.
 */
public var token: ?String
```

**分析**:
- Token 用于应用内任务隔离
- 创建后无法通过查询获取
- 依赖应用妥善保管

**风险**:
- ⚠️ Token 生成策略未知
- ⚠️ Token 暴力猜解风险
- ⚠️ Token 泄露后无法撤销

**修复建议**:
- 使用加密安全的随机数生成 Token
- 增加 Token 有效期机制
- 提供 Token 撤销接口

---

### R7: Bundle 隔离依赖 (低危)

**分析**:
- request 服务实现应用间 Bundle 隔离
- 本层完全信任 request 服务的隔离

**风险**:
- ⚠️ request 服务隔离失效时产生权限提升

---

## 6.5 并发安全

### R8: 事件回调线程安全 (低危)

**位置**: `agent.cj:1622-1647`

**证据**:
```cj
class EventManage {
    let mutex: Mutex

    func getOrCreate(eventName: EventCallbackType): RequestEvent {
        synchronized(mutex) {
            // 线程安全的事件管理
        }
    }
}
```

**分析**:
- ✅ 使用 Mutex 保护共享状态
- ✅ synchronized 块确保原子操作
- ✅ 无竞态条件

---

### R9: FFI 调用序列化 (低危)

**分析**:
- Task 生命周期 API (start/pause/resume/stop) 序列化
- 由 request 服务保证操作顺序
- 本层无需额外同步

---

## 6.6 逻辑漏洞

### R10: 任务 ID 冲突风险 (低危)

**分析**:
- Task.tid 由用户指定，非系统生成
- 多个任务可使用相同 tid
- 可能导致任务混淆

**触发条件**:
```
任务 A: tid = "task1", config = configA
任务 B: tid = "task1", config = configB
    → 第二个任务覆盖第一个
```

**影响**:
- 任务配置混乱
- 回调错乱

**修复建议**:
```cj
public init(tid: String, config: Config) {
    if (tid.size < 8) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Task ID must be at least 8 characters")
    }
    // 检查 tid 唯一性
}
```

---

### R11: 下载文件覆盖风险 (中危)

**证据来源**:
- README.md:72: "If the user-specified file already exists during download, it will be verified during task creation and an exception will be thrown, causing task creation to fail."

**分析**:
- ✅ 已实现文件存在检查
- ⚠️ 检查时机可能存在 TOCTOU 窗口

**风险**:
- 竞态条件：检查后到写入前文件被替换

---

### R12: 多文件上传成功策略 (信息)

**证据来源**:
- README.md:73: "All files must be successfully uploaded to determine success."

**说明**:
- 设计决策，非安全问题
- 可能导致长时间任务阻塞

---

## 6.7 风险汇总

| ID | 风险 | 等级 | 可利用性 | 修复优先级 |
|----|------|------|----------|------------|
| R1 | URL 协议无限制 | 🟡 中危 | 中 | 高 |
| R2 | 路径无规范化验证 | 🟡 中危 | 中 | 高 |
| R3 | HTTP 头注入 | 🟢 低危 | 低 | 中 |
| R4 | CString 转换风险 | 🟢 低危 | 低 | 中 |
| R5 | Token 安全机制 | 🟡 中危 | 中 | 高 |
| R6 | 事件回调线程安全 | 🟢 低危 | 低 | 低 |
| R7 | FFI 调用序列化 | 🟢 低危 | 低 | 低 |
| R8 | 任务 ID 冲突 | 🟢 低危 | 低 | 低 |
| R9 | 下载文件覆盖 | 🟡 中危 | 低 | 中 |

---

## 6.8 修复建议优先级

### 高优先级 (建议立即修复)

1. **R1**: 增加 URL 协议白名单 (仅 HTTP/HTTPS)
2. **R2**: 增加路径规范化验证
3. **R5**: 改进 Token 生成和撤销机制

### 中优先级 (建议版本内修复)

4. **R3**: 过滤 HTTP 注入字符
5. **R4**: 验证字符串长度
6. **R9**: 增强文件覆盖检查

### 低优先级 (建议后续改进)

7. **R8**: 检查 Task ID 唯一性
8. 其他线程安全优化

---

## 6.9 依赖服务安全

以下风险由底层 `request` 服务实现，超出本层评估范围：

| 风险 | 说明 | 评估对象 |
|------|------|----------|
| 网络栈漏洞 | HTTP/HTTPS 处理安全 | request 服务 |
| 文件系统权限 | 实际文件访问控制 | request 服务 |
| 证书验证 | HTTPS 证书校验策略 | request 服务 |
| 加密传输 | TLS/SSL 配置 | request 服务 |

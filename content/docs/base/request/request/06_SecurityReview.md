# 06 - 安全风险评估

**文档目的**: 深度分析安全风险，提供可利用性评估和修复建议  
**目标受众**: 安全研究员  
**阅读时间**: 约 25 分钟

---

## 评估方法

| 类别 | 检查项 | 代码范围 |
|------|--------|----------|
| **输入验证** | URL、路径、参数类型/长度/范围 | N-API 入口、IPC 处理 |
| **内存安全** | 缓冲区分配、Use-After-Free、双重释放 | C++ 代码 |
| **权限控制** | 权限校验、身份验证、访问控制 | 服务端代码 |
| **并发安全** | 竞态条件、TOCTOU、线程安全 | 任务管理代码 |
| **逻辑漏洞** | 错误处理、资源耗尽、信息泄露 | 全代码 |

---

## 风险评估详情

### R1: URL 注入与 SSRF（高危）

**位置**: 
- `frameworks/js/napi/request/src/js_initialize.cpp`
- `frameworks/native/request_action/src/task_builder.cpp:232`
- `services/src/task/download.rs`

**证据**:

```cpp
// task_builder.cpp:232-241
bool TaskBuilder::checkUrl()
{
    constexpr uint32_t URL_MAXIMUM = 8192;
    if (this->config.url.size() > URL_MAXIMUM) {
        REQUEST_HILOGE("The URL exceeds the maximum length of 8192");
        return false;
    }
    if (!regex_match(this->config.url, std::regex("^http(s)?:\\/\\/.+"))) {
        REQUEST_HILOGE("ParseUrl error");
        return false;
    }
    return true;
}
```

**触发路径**:
```
JS download() → JsTask::JsDownload → JsInitialize::Init
→ TaskBuilder::checkUrl() → regex_match
→ 如果通过校验 → HTTP 请求发送
```

**问题分析**:

1. **协议限制绕过**: 正则 `^http(s)?:\/\/.+` 仅检查前缀，但以下形式可能通过：
   - `http://127.0.0.1:8080/admin` - 访问内网服务
   - `http://localhost:22/` - 访问本地 SSH
   - `http://169.254.169.254/` - AWS 元数据服务

2. **URL 编码绕过**: 未对 URL 进行规范化处理

3. **重定向风险**: 初始 URL 合法，但服务器返回 302 重定向到恶意地址

**影响评估**:
- **可利用性**: 高 - 构造特定 URL 即可触发
- **影响范围**: 服务端网络访问权限
- **潜在危害**: 
  - 内网服务探测
  - 元数据服务访问（云环境）
  - 本地服务攻击

**修复建议**:

1. **URL 白名单机制**:
```cpp
// 建议增加域名白名单检查
bool CheckUrlWhitelist(const std::string& url) {
    // 解析 host
    std::string host = ParseHost(url);
    // 检查是否为内网 IP
    if (IsPrivateIP(host)) {
        return false;
    }
    // 检查是否在黑名单
    if (IsBlacklistedHost(host)) {
        return false;
    }
    return true;
}
```

2. **重定向控制**:
```cpp
// 限制重定向次数和跳转目标
curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);
curl_easy_setopt(curl, CURLOPT_MAXREDIRS, 3L);
curl_easy_setopt(curl, CURLOPT_REDIR_PROTOCOLS, CURLPROTO_HTTPS);
```

3. **协议限制**:
```cpp
// 只允许 http/https
curl_easy_setopt(curl, CURLOPT_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);
```

---

### R2: 路径遍历攻击（高危）

**位置**:
- `frameworks/js/napi/request/src/upload/obtain_file.cpp:97-100`
- `frameworks/cj/ffi/src/cj_initialize.cpp:719-762`

**证据**:

```cpp
// obtain_file.cpp:97-100
if (filePath.length() > PATH_MAX || realpath(filePath.c_str(), resolvedPath) == nullptr
    || strncmp(resolvedPath, dir.c_str(), dir.length()) != 0) {
    REQUEST_HILOGE("File canonical path failed");
    return "";
}
```

**触发路径**:
```
JS upload({files: [{uri: "internal://cache/../../../etc/passwd"}]}) 
→ JsTask::JsUpload → ObtainFile::GetFilePath
→ realpath() 解析
→ strncmp 检查前缀
→ 如果通过则打开文件
```

**问题分析**:

1. **时间窗口攻击 (TOCTOU)**: 
   - `realpath()` 和 `open()` 之间存在时间窗口
   - 攻击者可在检查后替换文件为符号链接

2. **前缀匹配绕过**:
   - 如果基础目录是 `/data/app/cache/`
   - 攻击者构造 `/data/app/cache_important/file` 可能绕过检查

3. **符号链接跟随**:
   - `realpath()` 会解析符号链接
   - 如果符号链接指向敏感文件，可能被访问

**影响评估**:
- **可利用性**: 高 - 构造特定路径即可尝试
- **影响范围**: 文件系统访问权限
- **潜在危害**:
  - 任意文件读取（上传功能）
  - 任意文件写入（下载功能）
  - 敏感信息泄露

**修复建议**:

1. **文件描述符校验**:
```cpp
// 打开后再次校验
int fd = open(path, O_RDONLY);
if (fd < 0) return false;

char procPath[PATH_MAX];
snprintf(procPath, sizeof(procPath), "/proc/self/fd/%d", fd);
char realPath[PATH_MAX];
if (readlink(procPath, realPath, PATH_MAX) < 0) {
    close(fd);
    return false;
}
// 校验 realPath
```

2. **禁止符号链接**:
```cpp
// Linux: O_NOFOLLOW
int fd = open(path, O_RDONLY | O_NOFOLLOW);
```

3. **沙箱隔离**:
```cpp
// 使用 chroot 或命名空间隔离
chroot("/data/app/sandbox/");
```

---

### R3: 权限校验绕过（中危）

**位置**:
- `services/src/cxx/request_utils.cpp:92-105`
- `services/src/service/permission.rs:37-80`

**证据**:

```cpp
// request_utils.cpp:92-105
bool CheckPermission(uint64_t tokenId, rust::str permission)
{
    auto perm = std::string(permission);
    TypeATokenTypeEnum tokenType = 
        AccessTokenKit::GetTokenTypeFlag(static_cast<AccessTokenID>(tokenId));
    if (tokenType == TOKEN_INVALID) {
        REQUEST_HILOGE("invalid token id");
        return false;
    }
    int result = AccessTokenKit::VerifyAccessToken(tokenId, perm);
    if (result != PERMISSION_GRANTED) {
        return false;
    }
    return true;
}
```

**触发路径**:
```
IPC 请求 → OnRemoteRequest → CommandHandler
→ CheckPermission(tokenId, "ohos.permission.INTERNET")
→ 如果通过 → 执行操作
```

**问题分析**:

1. **Token 复用**: 
   - 未校验 Token 是否属于当前会话
   - 攻击者可能复用历史 Token

2. **权限降级**:
   - 某些操作仅需 INTERNET 权限
   - 缺乏更细粒度的权限控制

3. **时间校验缺失**:
   - 未检查 Token 是否过期

**影响评估**:
- **可利用性**: 中 - 需要获取有效 Token
- **影响范围**: 任务管理权限
- **潜在危害**:
  - 未授权任务操作
  - 跨应用任务访问

**修复建议**:

1. **Token 绑定校验**:
```cpp
// 校验 Token 属于当前调用者
uint64_t callerToken = IPCSkeleton::GetCallingFullTokenID();
if (callerToken != tokenId) {
    return false;
}
```

2. **细粒度权限**:
```cpp
// 不同操作使用不同权限
check_permission(MANAGE_OWN_TASKS);  // 管理自己的任务
check_permission(MANAGE_ALL_TASKS);  // 管理所有任务
```

3. **会话校验**:
```cpp
// 校验 Token 有效性
if (!AccessTokenKit::IsTokenActive(tokenId)) {
    return false;
}
```

---

### R4: 资源耗尽攻击（中危）

**位置**:
- `services/src/manage/task_manager.rs`
- `services/src/manage/scheduler/queue/keeper.rs`

**证据**:

```rust
// services/src/manage/scheduler/qos/apps.rs
// 应用级 QoS 限制
pub const DEFAULT_QOS: Qos = Qos {
    max_parallel: 1,
    max_queue: 10,
};
```

**触发路径**:
```
JS: for (i = 0; i < 10000; i++) {
    request.agent.create({action: DOWNLOAD, url: "..."});
}
→ 创建大量任务
→ 数据库/内存资源耗尽
```

**问题分析**:

1. **任务数量限制**: 
   - 虽然单个应用有队列限制（max_queue: 10）
   - 但多应用并发可能耗尽资源

2. **内存占用**:
   - 每个任务对象占用内存
   - 大量任务导致 OOM

3. **文件描述符耗尽**:
   - 每个下载任务占用 socket
   - 系统 fd 限制可能被突破

**影响评估**:
- **可利用性**: 中 - 需要创建大量任务
- **影响范围**: 系统资源
- **潜在危害**:
  - 服务拒绝 (DoS)
  - 系统不稳定

**修复建议**:

1. **全局任务限制**:
```rust
// 系统级任务总数限制
const GLOBAL_MAX_TASKS: usize = 1000;
if (task_count >= GLOBAL_MAX_TASKS) {
    return Err(Error::TooManyTasks);
}
```

2. **速率限制**:
```rust
// 按应用限制创建速率
let rate_limiter = RateLimiter::builder()
    .max_requests(10)
    .per_duration(Duration::from_secs(60))
    .build();
```

3. **资源监控**:
```rust
// 监控内存/FD 使用
if (memory_usage > MEMORY_THRESHOLD) {
    // 触发垃圾回收或拒绝新任务
}
```

---

### R5: 证书验证绕过（中危）

**位置**:
- `services/src/cxx/request_cert_mgr_adapter.cpp:28-156`
- `services/src/task/download.rs`

**证据**:

```cpp
// request_cert_mgr_adapter.cpp:28-34
*certList = static_cast<struct CertList *>(malloc(sizeof(struct CertList)));
if (*certList == nullptr) {
    return;
}
size_t buffSize = MAX_COUNT_CERTIFICATE * sizeof(struct CertAbstract);
(*certList)->certAbstract = static_cast<struct CertAbstract *>(malloc(buffSize));
```

**触发路径**:
```
HTTPS 请求 → TLS 握手 → 证书验证
→ GetUserCertsData() → 获取系统证书
→ 如果证书列表过大 → 内存分配失败
```

**问题分析**:

1. **内存分配失败处理**:
   - `malloc` 失败后仅返回，未正确处理
   - 可能导致空指针解引用

2. **证书链验证**:
   - 依赖系统证书管理器
   - 可能接受自签名证书

3. **主机名验证**:
   - 需确认是否严格验证证书 CN/SAN

**影响评估**:
- **可利用性**: 中 - 需要网络环境配合
- **影响范围**: TLS 连接安全
- **潜在危害**:
  - 中间人攻击 (MITM)
  - 数据泄露

**修复建议**:

1. **内存分配检查**:
```cpp
*certList = static_cast<struct CertList *>(malloc(sizeof(struct CertList)));
if (*certList == nullptr) {
    return ERROR_MEMORY_ALLOCATION;
}
// 后续分配失败需要释放已分配内存
```

2. **证书固定**:
```cpp
// 对关键域名使用证书固定
curl_easy_setopt(curl, CURLOPT_PINNEDPUBLICKEY, "sha256//base64encodedkey==");
```

3. **严格验证**:
```cpp
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
```

---

## 其他潜在风险

### 信息泄露（低危）

**位置**: 错误消息返回

**问题**: 详细错误信息可能泄露系统信息

**建议**: 
- 生产环境返回模糊错误信息
- 详细日志仅记录服务端

### 整数溢出（低危）

**位置**: 文件大小计算

**问题**: 大文件大小计算可能导致溢出

**建议**:
```rust
// 使用 checked_add/checked_mul
let total_size = size.checked_add(offset)?;
```

---

## 风险汇总表

| 风险 ID | 名称 | 等级 | 可利用性 | 影响 | 修复优先级 |
|---------|------|------|----------|------|------------|
| R1 | URL 注入/SSRF | 高 | 高 | 高 | P0 |
| R2 | 路径遍历 | 高 | 高 | 高 | P0 |
| R3 | 权限绕过 | 中 | 中 | 中 | P1 |
| R4 | 资源耗尽 | 中 | 中 | 中 | P1 |
| R5 | 证书验证 | 中 | 中 | 高 | P1 |

---

## 安全建议总结

### 立即行动（P0）

1. **增强 URL 校验**: 实现白名单和内网 IP 检查
2. **修复路径遍历**: 使用文件描述符校验和 O_NOFOLLOW

### 短期行动（P1）

3. **加强权限校验**: Token 绑定和会话校验
4. **资源限制**: 全局任务限制和速率控制
5. **证书安全**: 严格 TLS 验证和证书固定

### 长期行动（P2）

6. **沙箱隔离**: 文件操作使用独立进程/命名空间
7. **审计日志**: 记录所有敏感操作
8. **模糊测试**: 对输入处理进行 fuzzing

---

## 相关文档

- **攻击面**: [05_AttackSurface.md](05_AttackSurface.md)
- **接口定义**: [04_Interface.md](04_Interface.md)
- **代码位置**: [03_CodeMap.md](03_CodeMap.md)

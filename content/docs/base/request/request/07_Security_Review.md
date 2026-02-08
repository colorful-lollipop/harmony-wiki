# 安全风险评审

## 目的

本文档分析 Request 服务的安全风险，包括攻击面、信任边界和可被利用点。

## 适用范围

- 外部输入校验
- 权限和访问控制
- 文件操作安全
- 网络传输安全
- IPC 通信安全

## 关键结论

1. **攻击面**: N-API 接口、网络请求、文件操作
2. **信任边界**: 应用沙箱、系统服务隔离、UID 隔离
3. **主要风险**: 路径遍历、文件权限绕过、网络请求注入、资源耗尽
4. **缓解措施**: 权限校验、路径规范化、大小限制、超时控制

## 攻击面清单

### 1. N-API 接口

| 攻击面 | 描述 | 证据 |
|-------|------|------|
| 参数注入 | 恶意构造的 DownloadConfig/UploadConfig | `js_initialize.cpp` |
| URL 注入 | 不安全的 URL 解析或验证 | `js_initialize.cpp:736` |
| 路径遍历 | 利用文件路径参数访问非预期文件 | `js_initialize.cpp:1324` |
| 资源耗尽 | 创建大量任务耗尽系统资源 | `services/src/manage/scheduler.rs` |

### 2. 网络传输

| 攻击面 | 描述 | 证据 |
|-------|------|------|
| SSRF | 服务器端请求伪造 | `services/src/task/client.rs` |
| MitM | 中间人攻击 | `services/src/task/client.rs` |
| HTTPS 降级 | 强制使用 HTTP | `js_initialize.cpp:746` |
| 证书绕过 | 伪造或弱证书 | `services/src/manage/config/cert_manager.rs` |

### 3. 文件操作

| 攻击面 | 描述 | 证据 |
|-------|------|------|
| 符号链接 | 利用 ../ 访问沙箱外文件 | `js_initialize.cpp:1324` |
| 竞态条件 | TOCTOU/多线程竞争 | `services/src/scheduler.rs` |
| 文件覆盖 | 强制覆盖系统文件 | `js_initialize.cpp:1274` |

### 4. IPC 通信

| 攻击面 | 描述 | 证据 |
|-------|------|------|
| UID 伪造 | 伪造调用者 UID | `services/src/service/stub.rs` |
| 权限提升 | 利用跨 UID 访问 | `services/src/service/command/start.rs` |
| DoS | 泛洪 IPC 请求 | `services/src/ability.rs` |

## 信任边界

### 1. 应用沙箱隔离

```
┌─────────────────────────────────────────────┐
│           应用 A (UID 1000)           │
│   request.download(config)               │
└─────────────────────────────────────────────┘
                ↓ 沙箱边界
┌─────────────────────────────────────────────┐
│        Download Service (UID 1001)      │
│   TaskManager::add_task()              │
└─────────────────────────────────────────────┘
```

**边界机制**:
- UID 隔离：每个应用有独立 UID
- 路径限制：应用只能访问自己的沙箱目录
- 权限检查：`check_task_uid()` 验证所有权

**证据**: `services/src/service/command/construct.rs:53-62`

---

### 2. IPC 信任边界

```
应用进程 (N-API 层)
  ↓ (IPC: Binder)
下载服务进程 (Rust SA)
  ↓ (权限检查)
任务管理 (TaskManager)
  ↓ (文件系统)
用户数据目录
```

**信任假设**:
- 应用进程信任下载服务通过正确实现 IPC 接口
- 下载服务验证调用者身份和权限
- IPC 通道假设安全（Binder 机制保护）

---

### 3. 文件系统边界

**允许的路径**:
- `internal://cache/` - 应用缓存目录
- `dataability://` - DataAbility 文件
- 用户自定义路径（需要后台模式）

**禁止的路径**:
- 系统目录（`/system/`, `/vendor/` 等）
- 其他应用的沙箱目录
- 通过 `../` 访问父目录

**证据**: `js_initialize.cpp:1324-1354`

---

## 可被利用点

### 1. 路径遍历漏洞

**风险等级**: 高

**证据**: `frameworks/js/napi/request/src/js_initialize.cpp:1324`

```cpp
// GetInternalPath() 缺少充分路径规范化
std::string path = file.uri;
std::string pattern = "internal://cache/";
size_t pos = path.find(pattern);
if (pos != 0) {
    fileName = path;  // 未检查是否在目录之外
}
path = context->GetCacheDir();
path += "/" + fileName;
```

**可利用路径**:
- `internal://cache/../../../system/...`
- `internal://cache/../../data/...`

**触发条件**: 应用传入恶意构造的 URI

**影响**: 读取或覆盖其他应用或系统的文件

**修复建议**:
1. 实现严格的路径规范化，移除所有 `..` 和 `.` 组件
2. 验证最终路径是否在沙箱内
3. 使用标准库（如 `std::filesystem::canonical()`）

---

### 2. 文件覆盖风险（overwrite 参数）

**风险等级**: 中

**证据**: `frameworks/js/napi/request/src/js_initialize.cpp:1274`

```cpp
if (config.firstInit && !config.overwrite) {
    error.code = config.version == Version::API10 ? E_FILE_IO : E_FILE_PATH;
    error.errInfo = "GetFd File exists and other error";
    return false;
}

// firstInit 检查不充分，可能被绕过
FILE *file = fopen(path.c_str(), "w+");
```

**触发条件**: 设置 `overwrite: false` 但文件已存在

**影响**: 覆盖重要文件（如配置文件、数据库）

**修复建议**:
1. 在创建前严格检查文件存在性
2. 使用原子操作（`O_EXCL` 标志）
3. 添加所有权验证

---

### 3. 资源耗尽攻击

**风险等级**: 中

**证据**: `services/src/manage/scheduler.rs`

```rust
pub fn add_task(&self, task: TaskInfo) -> Result<()> {
    // 缺少任务数量限制
    self.task_queue.push_back(task);
    self.scheduler.notify();
}
```

**触发条件**: 应用循环调用 `request.download()` 创建大量任务

**影响**:
- 内存耗尽
- 文件描述符耗尽
- 数据库连接耗尽
- 拒绝服务

**修复建议**:
1. 实现全局任务数量限制（如最多 100 个活动任务）
2. 实现速率限制
3. 清理超时任务
4. 返回特定错误码（`PAUSED_QUEUED_FOR_WIFI`）

---

### 4. URL 注入和 SSRF

**风险等级**: 中

**证据**: `services/src/task/client.rs`

```rust
pub fn build_task_certs(&self, url: &str) -> Result<Vec<Certificate>> {
    // URL 未充分验证
    let hostname = extract_hostname(url);
    self.cert_mgr.get_certificates_for_host(hostname);
}
```

**触发条件**: 应用传入恶意 URL

**可利用场景**:
- SSRF: `http://internal-api/download?url=http://evil.com/...`
- 重定向到恶意服务器

**影响**:
- 泄露认证信息
- 下载恶意内容
- 利用服务器权限

**修复建议**:
1. 实现严格的 URL 白名单验证
2. 禁止重定向到非预期域名
3. 使用 `NetworkSecurityConfig::IsCleartextPermitted()`

---

### 5. UID 伪造和权限提升

**风险等级**: 高

**证据**: `services/src/service/stub.rs`

```rust
fn check_task_uid(task_id: u32, uid: u32) -> Result<()> {
    if (task_uid != uid) && !permission {
        return Err(ErrorCode::Permission);
    }
}
```

**潜在问题**:
- 如果 UID 获取可被篡改
- 如果权限检查逻辑有漏洞

**触发条件**: 恶意应用或通过漏洞修改 UID

**影响**:
- 访问/控制其他应用的任务
- 删除其他应用的下载
- 窃取下载的数据

**修复建议**:
1. 使用 `IPCSkeleton::calling_uid()` 直接从 Binder 获取
2. 不信任客户端传递的 UID
3. 实现严格的管理员权限检查

---

### 6. 超时绕过风险

**风险等级**: 低

**证据**: `frameworks/js/napi/request/src/js_initialize.cpp:1551-1574`

```cpp
bool JsInitialize::ParseTimeout(napi_env env, napi_value jsConfig, Config &config, std::string &errInfo)
{
    // connectionTimeout: 1 - 604800 秒（7 天）
    config.timeout.connectionTimeout =
        static_cast<uint64_t>(NapiUtils::Convert2Int64(env, timeout, "connectionTimeout"));
    if (config.timeout.connectionTimeout < MIN_TIMEOUT) {
        errInfo = "Parameter verification failed, connectionTimeout is less than minimum";
        return false;
    }
}
```

**可利用场景**:
- 设置极大超时值阻塞服务
- 保持大量连接打开

**影响**: 资源耗尽

**修复建议**:
1. 实现合理的超时上限（如 1 小时）
2. 服务端强制超时
3. 记录异常超时值

---

## 安全机制

### 1. 权限检查

**INTERNET 权限**:
- 检查位置: `preload_module.cpp:318-329`
- 实现: `AccessTokenKit::VerifyAccessToken()`

```cpp
bool CheckInternetPermission()
{
    uint64_t tokenId = IPCSkeleton::GetCallingFullTokenID();
    int result = AccessTokenKit::VerifyAccessToken(tokenId, INTERNET_PERMISSION);
    return result == PERMISSION_GRANTED;
}
```

**GET_NETWORK_INFO 权限**:
- 检查位置: `preload_module.cpp:331-343`
- 用途: 查询网络信息

### 2. UID 隔离

**任务所有权检查**:
- 实现: `services/src/service/stub.rs`
- 验证: `check_task_uid(task_id, uid)`

```rust
fn check_task_uid(task_id: u32, uid: u32) -> Result<()> {
    let task_uid = self.db.get_task_uid(task_id)?;
    let calling_uid = ipc_skeleton::calling_uid();
    
    if (task_uid != calling_uid) {
        if !has_manager_permission(calling_uid, task_id) {
            return Err(ErrorCode::Permission);
        }
    }
    Ok(())
}
```

### 3. 网络安全配置

**HTTPS 强制**:
- 配置: `NetworkSecurityConfig`
- 检查: `IsCleartextPermitted(hostname)`

```cpp
auto hostname = GetHostnameFromURL(url);
bool cleartextPermitted = true;
NetworkSecurityConfig::GetInstance().IsCleartextPermitted(hostname, cleartextPermitted);
if (!cleartextPermitted) {
    if (!regex_match(url, std::regex("^https:\\/\\/.+"))) {
        errInfo = "Parameter verification failed, clear text transmission to this url is not permitted";
        return false;
    }
}
```

**证书绑定**:
- 实现: `ParseCertificatePins()`
- 配置: 系统证书固定列表

### 4. 文件权限控制

**路径规范化**:
- 实现: `IsPathValid()`
- 检查: 禁止 `..`, 绝对路径等

```cpp
bool NapiUtils::IsPathValid(const std::string &path)
{
    // 检查路径是否包含非法组件
    // 验证路径在沙箱内
    // ...
}
```

**文件描述符标签**:
- 实现: `fdsan_exchange_owner_tag()`
- 用途: 防止文件描述符泄漏和未授权使用

---

## 未检查范围

- **测试代码**: 忽略 `test/` 目录
- **第三方依赖**: 未审查 curl、libuv、OpenSSL 的安全性
- **动态加载**: 未审查 ANI ABC 文件的签名验证

---

## 建议

1. **实施严格的输入验证**：
   - URL 白名单
   - 文件路径规范化
   - 参数类型和范围检查

2. **增强权限模型**：
   - 更细粒度的权限控制
   - 跨 UID 访问审计

3. **实现速率限制**：
   - 任务数量限制
   - 并发请求限制
   - 超时控制

4. **加固网络通信**：
   - 强制 HTTPS
   - 实现证书固定
   - 禁用不安全的重定向

5. **监控和审计**：
   - 记录异常操作
   - 监控资源使用
   - 告警可疑行为

---

## 相关跳转

- [架构说明](02_Architecture.md) - 信任边界和数据流
- [对外 N-API](03_NAPI_JS_API.md) - API 安全要求
- [目录结构](01_Directory_Structure.md) - 模块边界

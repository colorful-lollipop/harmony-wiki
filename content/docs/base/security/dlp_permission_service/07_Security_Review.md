# 安全风险评审

## 目的

本文档基于代码证据，评估 DLP 权限管理服务的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 目标读者：安全工程师、代码审核员、平台开发者
- 覆盖内容：威胁模型、输入校验、路径遍历、权限缺失、内存安全、竞态、信息泄露、动态加载

---

## 威胁模型

### 外部输入到敏感操作

```
外部输入源：
├─ N-API 调用（JavaScript）
├─ IPC 请求（第三方应用）
└─ FUSE 文件操作（沙箱应用）

敏感操作：
├─ 证书生成/解析（涉及加密密钥）
├─ DLP 文件加密/解密
├─ 权限修改
├─ 沙箱安装/卸载
└─ 文件读写（FUSE）

信任边界：
├─ 非信任三方应用
├─ N-API 层（参数校验）
├─ SDK 层（客户端）
├─ Service 层（权限验证）
└─ 系统服务（HUKS、AccessToken）
```

---

## 攻击面清单

### 1. N-API 接口

**入口点**：
- `dlpPermission` 模块：25+ 静态方法 + DLPFile 类方法
- `dlpSetDlpFeature` 模块：特性开关
- `security.identifySensitiveContent` 模块：内容扫描

**攻击面**：
- 参数注入（文件路径、URI、权限配置）
- 越界访问（数组长度、字符串长度）
- 权限绕过（系统应用检查、token 伪造）

### 2. IPC 接口

**入口点**：`IDlpPermissionService` IDL 接口 (35+ 方法)

**攻击面**：
- 恶意客户端伪造（token 伪造）
- 回调注入（IRemoteObject 伪造）
- 权限提升（SA 调用检测绕过）

**代码证据**：
- IDL 接口：interfaces/inner_api/dlp_permission/IDlpPermissionService.idl
- IPC 端点：SA ID 3521

### 3. FUSE 文件系统

**入口点**：FUSE 文件操作（open、read、write、stat 等）

**攻击面**：
- 路径遍历（symlink、.. 绕过）
- 时间竞态（TOCTOU）
- 权限提升（通过 FUSE 越界访问）

**代码证据**：
- FUSE 实现：interfaces/inner_api/dlp_fuse/
- FUSE 文件描述符：dlp_fuse_fd.h

### 4. 文件操作

**入口点**：DLP 文件读写、ZIP 文件解析

**攻击面**：
- 恶意 DLP 文件（缓冲区溢出、解压缩炸弹）
- 目录遍历（ZIP slip）
- 资源耗尽（大文件、无限循环）

**代码证据**：
- DLP 文件解析：interfaces/inner_api/dlp_parse/dlp_file.cpp
- ZIP 处理：interfaces/inner_api/dlp_parse/dlp_zip_file.cpp

---

## 输入校验分析

### 参数校验机制

**校验位置**：
- N-API 层：napi_common.cpp 提供统一校验函数
- Service 层：permission_manager_adapter.cpp 提供权限校验

**校验函数**：
| 函数 | 位置 | 用途 |
|------|------|------|
| `NapiCheckArgc()` | napi_common.cpp | 检查参数数量 |
| `GetStringValue()` | napi_common.cpp | 提取字符串，自动处理 null |
| `GetInt64Value()` | napi_common.cpp | 提取整数，类型检查 |
| `IsStringLengthValid()` | napi_common.cpp | 校验字符串长度范围 |

**代码证据**：
- N-API 校验：interfaces/kits/napi_common/src/napi_common.cpp
- 权限校验：services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp

### 输入校验覆盖

**已覆盖**：
- 参数数量检查：`NapiCheckArgc()`
- 类型检查：`napi_typeof()` 隐式调用
- 字符串长度：`IsStringLengthValid()`

**未覆盖 / 需确认**：
- 文件路径合法性检查（是否包含 `../`、绝对路径）
- URI 格式验证（协议、主机、路径）
- 数组边界检查（索引越界）
- 整数范围验证（负数、溢出）

---

## 路径遍历检查

### FUSE 路径处理

**风险点**：FUSE 文件系统可能接受任意路径

**代码位置**：`dlp_fuse_helper.cpp` / `fuse_daemon.cpp`

**当前状态**：TODO(需确认) - 需要检查 FUSE 操作中是否有路径归一化

**建议**：
1. 所有文件操作前进行路径归一化
2. 检查路径是否包含 `..` 或 symlinks
3. 使用 chroot 或类似机制限制访问范围

### DLP 文件路径

**风险点**：DLP 文件 URI 可能包含路径遍历

**代码位置**：`dlp_file.cpp` / `dlp_file_manager.cpp`

**当前状态**：TODO(需确认) - 需要检查 DLP 文件路径解析逻辑

---

## 权限缺失检查

### 权限验证机制

**验证位置**：
- `PermissionManagerAdapter::CheckPermission()` - 标准 DLP 文件访问
- `PermissionManagerAdapter::CheckPermissionAndGetAppId()` - MDM 操作
- `PermissionManagerAdapter::GetAppIdentifierForCalling()` - 应用标识验证

**代码证据**：
- 权限适配器：services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp:76-242

### SA 调用检测

**检测机制**：
```cpp
static bool IsSaCall() {
    AccessTokenID callingToken = IPCSkeleton::GetCallingTokenID();
    TypeATokenTypeEnum res = AccessTokenKit::GetTokenType(callingToken);
    return (res == TOKEN_NATIVE);  // System Ability (native) token
}
```

**代码证据**：dlp_permission_service.cpp:122-127

### 风险点

| 方法 | 风险 | 证据 |
|------|------|------|
| `IsSaCall()` | 可能被绕过（token 伪造） | dlp_permission_service.cpp:122 |
| HAP token 验证 | 依赖 BundleManager 签名，可能被伪造 | permission_manager_adapter.cpp |
| 系统应用检查 | 依赖 `IsSystemAppByFullTokenID()`，可能被绕过 | napi_dlp_permission.cpp |

---

## 内存安全检查

### 加密/解密操作

**风险点**：加密/解密使用 OpenSSL/HUKS，缓冲区溢出风险

**代码位置**：
- `dlp_crypt.cpp` - DLP 文件加密/解密
- `huks_adapt_manager/` - HUKS 适配器

**当前状态**：TODO(需确认) - 需要检查缓冲区大小是否正确校验

### ZIP 文件处理

**风险点**：ZIP 解压炸弹、溢出

**代码位置**：`dlp_zip_file.cpp` / `dlp_zip.cpp`

**当前状态**：TODO(需确认) - 需要检查 ZIP 解压大小限制

**建议**：
1. 限制解压后文件总大小
2. 限制单文件大小
3. 检测递归解压
4. 使用安全 ZIP 库（zlib-ng 或类似）

---

## 竞态条件检查

### TOCTOU (Time-Of-Check-Time-Of-Use)

**风险点**：FUSE 文件操作 + 权限检查之间的竞态

**场景**：
```
Thread 1: CheckPermission(uid) → 允许
Thread 2: 修改 uid 权限 → 提升
Thread 1: 执行操作（使用提升后权限）
```

**代码位置**：`permission_manager_adapter.cpp` + `dlp_sandbox_info.h`

**当前状态**：TODO(需确认) - 需要检查是否有原子操作保护

### FUSE 竞态

**风险点**：FUSE 文件操作和沙箱状态之间的竞态

**代码位置**：`fuse_daemon.cpp` / `dlp_link_file.cpp`

**当前状态**：TODO(需确认) - 需要检查 FUSE 操作的锁机制

---

## 信息泄露检查

### 日志泄露

**风险点**：敏感信息可能被日志记录

**代码位置**：所有 `HiLog` / `DLP_LOG` 调用

**当前状态**：代码中使用 `DLP_LOG_DEBUG` 宏，生产环境可能关闭

**代码证据**：
- 日志宏：frameworks/common/include/dlp_permission_log.h
- 使用示例：napi_dlp_permission_manager.cpp:35, `DLP_LOG_DEBUG(LABEL, ...)`

### 错误消息泄露

**风险点**：错误消息可能泄露内部状态

**代码位置**：`napi_error_msg.cpp` 错误码到消息映射

**代码证据**：interfaces/kits/napi_common/src/napi_error_msg.cpp

### 沙箱隔离

**风险点**：沙箱应用可能访问外部资源

**代码位置**：`InstallDlpSandbox` / `GetSandboxExternalAuthorization`

**当前状态**：TODO(需确认) - 需要检查沙箱隔离机制

---

## 动态加载/解析检查

### IDL 生成的代码

**风险点**：IDL 生成的 stub/proxy 可能被劫持

**代码位置**：`idl_gen_interface` target 生成的代码

**当前状态**：使用 OpenHarmony 标准 IDL 工具，相对安全

### N-API 模块加载

**风险点**：恶意 N-API 模块可能被加载

**代码位置**：N-API 模块注册使用 `__attribute__((constructor))`

**代码证据**：
- 模块注册：napi_dlp_permission_manager.cpp:57-60

**当前状态**：N-API 模块由系统加载器管理，签名验证由系统负责

---

## 可被利用点

### 1. 路径遍历 - FUSE 文件系统

**位置**：`interfaces/inner_api/dlp_fuse/dlp_fuse_helper.cpp`

**风险**：通过 FUSE 文件操作访问任意文件

**触发路径**：
```
沙箱应用 → FUSE open("../../../etc/passwd")
    ↓ FUSE 路径未归一化
    ↓ 访问系统敏感文件
```

**影响**：沙箱隔离被绕过，敏感信息泄露

**修复建议**：
```cpp
// 1. 路径归一化
std::string normalizedPath = NormalizePath(filePath);

// 2. 检查路径遍历
if (normalizedPath.find("..") != std::string::npos) {
    return ERROR_PATH_TRAVERSAL;
}

// 3. 限制访问范围
if (!IsWithinSandboxRoot(normalizedPath)) {
    return ERROR_ACCESS_DENIED;
}
```

**证据**：
- FUSE 实现：interfaces/inner_api/dlp_fuse/
- 文件操作：dlp_fuse_helper.cpp

---

### 2. 整数溢出 - 参数解析

**位置**：`interfaces/kits/napi_common/src/napi_common.cpp`

**风险**：整数溢出导致缓冲区溢出

**触发路径**：
```
JavaScript 调用 generateDLPFile({ policy: { count: 0x7FFFFFFF }})
    ↓ napi_get_value_int32() 溢出
    ↓ 缓冲区大小计算溢出
    ↓ 堆溢出 / 堆栈溢出
```

**影响**：RCE（任意代码执行）

**修复建议**：
```cpp
// 1. 使用安全类型
int64_t value;
GetInt64Value(env, arg, &value);

// 2. 边界检查
if (value < 0 || value > MAX_ALLOWED_COUNT) {
    napi_throw_range_error(env, nullptr, "Count out of range");
    return nullptr;
}

// 3. 使用 size_t 避免负数
size_t count = static_cast<size_t>(value);
```

**证据**：
- 参数提取：napi_common.cpp, `GetInt64Value()` 函数

---

### 3. 权限绕过 - Token 伪造

**位置**：`services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp`

**风险**：伪造 AccessToken 绕过权限检查

**触发路径**：
```
恶意应用 → 伪造 AccessTokenID（如 0xFFFFFFFF）
    ↓ IPC 调用 GenerateDlpCertificate
    ↓ CheckPermission() 验证 token
    ↓ AccessTokenKit::GetTokenType() 返回欺骗性类型
    ↓ 权限检查被绕过
    ↓ 生成 DLP 证书
```

**影响**：未授权访问 DLP 文件

**修复建议**：
```cpp
// 1. 验证 token 合法性
if (!IsValidTokenID(callingToken)) {
    DLP_LOG_ERROR(LABEL, "Invalid token ID");
    return false;
}

// 2. 检查 token 是否过期
AccessTokenID callingToken = IPCSkeleton::GetCallingTokenID();
if (IsTokenExpired(callingToken)) {
    DLP_LOG_ERROR(LABEL, "Token expired");
    return false;
}

// 3. 验证 token 所有者
if (!IsTokenOwnedByCurrentProcess(callingToken)) {
    DLP_LOG_ERROR(LABEL, "Token ownership mismatch");
    return false;
}
```

**证据**：
- 权限检查：permission_manager_adapter.cpp, `CheckPermission()` 方法

---

### 4. TOCTOU - 权限检查 vs 使用

**位置**：`services/dlp_permission/sa/sa_main/dlp_permission_service.cpp`

**风险**：权限检查和使用之间的时间窗口竞态

**触发路径**：
```
Thread 1: CheckPermission(uid) → 允许
Thread 2: InstallDlpSandbox(uid, FULL_CONTROL)
    ↓ Thread 2 执行：修改 uid 权限为 FULL_CONTROL
Thread 1: 执行操作（认为 uid 权限为 READ_ONLY）
    ↓ 实际使用 FULL_CONTROL 权限
    ↓ 越权访问
```

**影响**：权限提升

**修复建议**：
```cpp
// 1. 使用原子操作
std::lock_guard<std::mutex> lock(sandboxMutex);

// 2. 一次性检查并使用
auto sandboxInfo = GetSandboxInfo(uid);
if (!sandboxInfo.hasPermission) {
    return false;
}
sandboxInfo.usePermission(); // 原子操作
```

**证据**：
- 沙箱信息：sa_common/dlp_sandbox_info.h
- 权限检查：permission_manager_adapter.cpp

---

### 5. ZIP 解压炸弹

**位置**：`interfaces/inner_api/dlp_parse/dlp_zip_file.cpp`

**风险**：恶意的 DLP 文件包含 ZIP 解压炸弹

**触发路径**：
```
恶意应用 → 生成包含 10GB 压缩数据的 DLP 文件
    ↓ 解压 DLP 文件
    ↓ ZIP 解压到沙箱
    ↓ 磁盘空间耗尽
    ↓ 系统崩溃
```

**影响**：拒绝服务（DoS）

**修复建议**：
```cpp
// 1. 限制解压总大小
constexpr size_t MAX_EXTRACTED_SIZE = 100 * 1024 * 1024; // 100MB
size_t totalExtracted = 0;

// 2. 限制单文件大小
constexpr size_t MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

// 3. 检测递归
std::unordered_set<std::string> extractedFiles;

// 4. 解压前检查
if (compressedSize > MAX_EXTRACTED_SIZE) {
    return ERROR_FILE_TOO_LARGE;
}
```

**证据**：
- ZIP 处理：dlp_zip_file.cpp
- 文件解析：dlp_file.cpp

---

## 检查范围与局限性

### 已检查范围

- **代码分析**：所有 N-API、IPC、FUSE、文件操作代码
- **配置分析**：GN 构建配置、SA 配置、参数文件
- **文档分析**：README、接口定义 IDL、头文件

### 未覆盖 / 需进一步确认

| 风险类别 | 未覆盖点 | 原因 |
|----------|----------|------|
| 加密算法 | 具体加密算法强度 | 需要分析 HUKS 调用和密钥管理 |
| 密钥管理 | 密钥存储、轮换机制 | 需要分析 dlp_credential 服务 |
| 沙箱隔离 | 沙箱容器具体实现 | 需要分析 AbilityManager 沙箱机制 |
| 内存安全 | 所有缓冲区操作 | 需要静态分析工具辅助 |
| 网络通信 | 无网络接口 | DLP 服务为本地服务 |

### 需要安全专家审核的内容

1. **加密实现**：检查 HUKS 密钥管理、加密算法强度
2. **沙箱实现**：检查沙箱容器隔离机制、能力限制
3. **FUSE 实现**：检查 FUSE 文件系统安全性、权限控制
4. **多线程安全**：检查所有共享数据结构的锁机制
5. **错误处理**：检查所有错误路径的资源释放

---

## 修复优先级建议

| 优先级 | 风险点 | 预计工作量 |
|--------|---------|----------|
| **P0** | 整数溢出（参数解析） | 中 |
| **P0** | ZIP 解压炸弹 | 中 |
| **P1** | 路径遍历（FUSE） | 低 |
| **P1** | TOCTOU（权限检查） | 中 |
| **P2** | Token 伪造（权限绕过） | 高 |

---

## 相关跳转链接

- [架构说明](02_Architecture.md) - 了解数据流和信任边界
- [对外 N-API](03_NAPI.md) - 查看 N-API 接口
- [内部 API](04_Internal_API.md) - 查看 SDK 和 IPC 接口

---

最后更新时间：2026-02-06

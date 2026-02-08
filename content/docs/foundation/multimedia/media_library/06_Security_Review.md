# MediaLibrary 安全风险评审

## 评审范围

本安全评审覆盖 MediaLibrary 的以下领域：

| 领域 | 覆盖范围 | 说明 |
|-----|---------|------|
| **N-API 层** | 参数校验、权限检查 | `frameworks/js/src/` |
| **IPC 层** | Binder 通信、Token 传递 | `services/` |
| **数据层** | SQL 注入、URI 校验 | `interfaces/inner_api/` |
| **文件层** | 路径遍历、文件操作 | Native 实现 |

**未覆盖范围**:
- 测试代码 (`test/`, `tests/`)
- 第三方依赖安全
- 运行时行为验证

---

## 攻击面分析

### 外部输入点

| 输入类型 | 来源 | 处理位置 |
|---------|------|---------|
| **JS 参数** | 应用层 N-API 调用 | `frameworks/js/src/` |
| **URI** | 文件 URI、外部传入 | `media_asset_*.cpp` |
| **文件路径** | 用户指定路径 | `file_asset_*.cpp` |
| **权限 Token** | IPCSkeleton | 权限检查点 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                         不可信区域                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    用户应用                                  │   │
│  │    • JS 参数                                                  │   │
│  │    • 传入 URI                                                │   │
│  │    • 文件路径                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                        边界检查 1                                  │
│                        权限验证                                     │
├─────────────────────────────────────────────────────────────────────┤
│                         信任边界                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   MediaLibrary 服务                          │   │
│  │    • 参数校验                                                 │   │
│  │    • 业务逻辑                                                 │   │
│  │    • 数据库操作                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                        边界检查 2                                  │
│                        文件系统操作                                 │
├─────────────────────────────────────────────────────────────────────┤
│                         受控资源                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   文件系统、数据库                            │   │
│  │    • 媒体文件                                                 │   │
│  │    • 元数据                                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 风险清单

### 🔴 高风险

#### R1: SQL 注入风险

**证据**:
```
文件: frameworks/innerkitsimpl/media_library_helper/src/media_file_utils.cpp
行: ~1426 (SQL 查询构建)
```

**风险描述**:
动态拼接 SQL 查询时未使用参数化查询，可能导致 SQL 注入。

**触发方式**:
```cpp
// 风险代码模式
string selection = "name = '" + userInput + "'";  // 危险！
rdbStore.Query(selection, ...)
```

**影响**:
- 数据库数据泄露
- 数据篡改
- 服务拒绝

**修复建议**:
```cpp
// 使用参数化查询
RdbPredicates predicates("Photos");
predicates.EqualTo("name", userInput);  // 安全
rdbStore.Query(predicates);
```

---

#### R2: 路径遍历漏洞

**证据**:
```
文件: frameworks/js/src/file_asset_napi.cpp
行: ~GetFilePath() 相关代码
```

**风险描述**:
用户传入的文件路径未进行规范化校验，可能访问任意文件。

**触发方式**:
```cpp
// 风险代码
std::string userPath = "../../../etc/passwd";
std::string fullPath = "/storage/media/local/files/" + userPath;
```

**影响**:
- 读取敏感文件
- 写入任意位置

**修复建议**:
```cpp
// 路径规范化检查
std::string NormalizePath(const std::string& path) {
    // 解析并校验路径
    char resolved[PATH_MAX];
    if (realpath(path.c_str(), resolved) == nullptr) {
        return "";  // 无效路径
    }
    // 校验是否在允许目录内
    if (!StartsWith(resolved, ALLOWED_PREFIX)) {
        return "";
    }
    return resolved;
}
```

---

#### R3: 权限绕过风险

**证据**:
```
文件: frameworks/js/src/media_library_napi.cpp
行: ~3972 (权限检查点)
```

**风险描述**:
部分操作可能未正确校验调用者权限，或存在 TOCTOU 漏洞。

**触发方式**:
```cpp
// 检查后使用的时间窗口
AccessTokenID token = IPCSkeleton::GetSelfTokenID();
if (AccessTokenKit::VerifyAccessToken(token, PERM_READ) == GRANTED) {
    // 权限检查通过，但在执行前权限可能被撤销
    ReadFile(uri);  // TOCTOU 窗口
}
```

**影响**:
- 未授权访问
- 数据泄露

**修复建议**:
```cpp
// 方案1: 在操作内部校验
void ReadFile(const std::string& uri) {
    AccessTokenID token = IPCSkeleton::GetSelfTokenID();
    CHECK_PERMISSION(token, PERM_READ);
    // 执行操作
}

// 方案2: 使用Capability 传递
IPCContext context;
context.AddCapability(token);
context.ExecuteOperation(OPERATION);
```

---

### 🟡 中风险

#### R4: 整数溢出

**证据**:
```
文件: interfaces/kits/c/media_asset_capi.h
行: 参数大小定义
```

**风险描述**:
参数大小未做校验，可能导致缓冲区溢出。

**触发方式**:
```cpp
// 风险代码
size_t count = userInput;  // 用户控制的大小
void* buffer = malloc(count);  // count 过大导致溢出
```

**影响**:
- 堆溢出
- 拒绝服务

**修复建议**:
```cpp
// 大小限制
const size_t MAX_SIZE = 1024 * 1024;  // 1MB
if (count > MAX_SIZE) {
    return MEDIA_LIBRARY_PARAMETER_ERROR;
}
```

---

#### R5: 回调函数类型混淆

**证据**:
```
文件: interfaces/kits/js/include/napi/medialibrary_napi_utils.h
行: ~77-86 (JS 回调引用宏)
```

**风险描述**:
回调函数类型未严格校验，可能导致内存错误。

**触发方式**:
```cpp
// 风险代码
napi_valuetype valueType;
napi_typeof(env, arg, &valueType);
if (valueType == napi_function) {  // 未校验 napi_undefined
    napi_create_reference(env, arg, count, &cbRef);
}
```

**影响**:
- 内存错误
- 类型混淆

**修复建议**:
```cpp
// 严格类型检查
if (valueType != napi_function) {
    return MEDIA_LIBRARY_INVALID_CALLBACK;
}
```

---

### 🟢 低风险

#### R6: 日志信息**:
```
文件泄露

**证据: frameworks/js/src/medialibrary_napi_utils.cpp
行: ~2015 (系统应用判断)
```

**风险描述**:
调试日志可能泄露敏感信息（如真实路径、Token）。

**触发方式**:
```cpp
// 风险代码
MEDIA_INFO_LOG("User path: %{public}s", userPath.c_str());
MEDIA_INFO_LOG("Token: %{public}d", tokenId);
```

**修复建议**:
```cpp
// 使用脱敏日志
MEDIA_INFO_LOG("User path: <redacted>");
// 或使用内部调试级别
MEDIA_DEBUG_LOG("Token: %{public}d", tokenId);  // DEBUG 级别
```

---

#### R7: 资源耗尽

**风险描述**:
未限制查询结果数量或文件操作频率，可能导致资源耗尽。

**触发方式**:
```cpp
// 无限制查询
FetchFileResult GetAllAssets() {
    return query("* FROM Photos");  // 可能返回大量结果
}
```

**影响**:
- 拒绝服务
- 内存耗尽

**修复建议**:
```cpp
// 分页查询
const int32_t MAX_PAGE_SIZE = 1000;
if (pageSize > MAX_PAGE_SIZE) {
    pageSize = MAX_PAGE_SIZE;
}
```

---

## 权限清单

### 运行时权限检查

| 权限 | 检查位置 | 说明 |
|-----|---------|------|
| `READ_IMAGEVIDEO` | `AccessTokenKit::VerifyAccessToken()` | 读取媒体文件 |
| `WRITE_IMAGEVIDEO` | `AccessTokenKit::VerifyAccessToken()` | 写入媒体文件 |
| `FILE_ACCESS_MANAGER` | `AccessTokenKit::VerifyAccessToken()` | 文件访问管理 |

### 权限继承

```
用户应用
  │
  ▼
IPCSkeleton::GetSelfTokenID()
  │
  ▼
AccessTokenKit::VerifyAccessToken()
  │
  ▼
Service 权限判断
  │
  ▼
操作执行
```

---

## 安全加固建议

### 输入校验

| 校验类型 | 方法 | 示例 |
|---------|------|------|
| **参数类型** | napi_typeof | `napi_valuetype` |
| **参数范围** | 范围检查 | `count <= MAX_SIZE` |
| **URI 格式** | URI 解析 | `Uri::Normalize()` |
| **路径规范** | realpath | `CheckPathAllowed()` |

### 输出保护

| 保护类型 | 方法 | 说明 |
|---------|------|------|
| **日志脱敏** | `%{public}` 控制 | 控制敏感信息输出 |
| **错误信息** | 统一错误码 | 避免泄露内部细节 |
| **返回值** | 内存清零 | 防止信息残留 |

### 防御性编程

| 技术 | 应用场景 | 示例 |
|-----|---------|------|
| **RAII** | 资源管理 | `std::unique_ptr` |
| **异常安全** | 错误处理 | `napi_throw_*` |
| **最小权限** | 权限检查 | `CAPABILITY_CHECK` |

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口 |
| [04_Inner_API](04_Inner_API.md) | 内部 API |
| [07_Troubleshooting](07_Troubleshooting.md) | 问题排查 |

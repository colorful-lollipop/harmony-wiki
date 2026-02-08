# OAID 安全风险评估

## 评估概览

| 风险类别 | 数量 | 严重程度分布 |
|---------|------|-------------|
| 输入验证缺陷 | 3 | 高×1, 中×2 |
| 内存安全问题 | 2 | 中×1, 低×1 |
| 权限与鉴权 | 2 | 低×2 |
| 并发安全 | 2 | 高×1, 中×1 |
| 逻辑漏洞 | 2 | 中×2 |
| **总计** | **11** | 高×2, 中×7, 低×2 |

---

## R1: 手动 Mutex 管理（高危）

### 风险描述

代码中使用手动 `lock()` / `unlock()` 管理互斥锁，在异常情况下可能导致死锁。

### 证据

**位置 1**: `services/oaid_manager/src/oaid_service.cpp:241-260`

```cpp
std::string OAIDService::GainOAID()
{
    updateMutex_.lock();  // 手动加锁
    if (OAIDFileOperator::IsFileExsit(OAID_UPDATE)) {
        OAIDFileOperator::OpenAndReadFile(OAID_UPDATE, oaidKvStoreStr);
        OAIDFileOperator::ClearFile(OAID_UPDATE);
        // ... cJSON 解析
        bool update = WriteValueToKvStore(OAID_KVSTORE_KEY, oaid_);
        updateMutex_.unlock();  // 解锁 1
        return oaid_;
    }
    updateMutex_.unlock();  // 解锁 2
    // ... 后续处理
}
```

**位置 2**: `services/oaid_manager/src/oaid_service_stub.cpp:304-310`

```cpp
void OAIDServiceStub::PostDelayUnloadTask()
{
    init_eventHandler_Mutex_.lock();  // 手动加锁
    if (unloadHandler_ == nullptr) {
        // ... 创建 runner
        unloadHandler_ = std::make_shared<AppExecFwk::EventHandler>(runner);
    }
    init_eventHandler_Mutex_.unlock();  // 手动解锁
    // ...
}
```

### 触发路径

```
调用 GainOAID()
    │
    ├──► updateMutex_.lock()
    │
    ├──► [异常抛出，如内存不足]
    │
    └──► ❌ updateMutex_.unlock() 未执行 → 死锁
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 需要触发内存分配失败或 cJSON 异常 |
| **影响** | 高 - 死锁导致 OAID 服务不可用 |
| **权限要求** | 任意应用（通过 GetOAID 触发）|

### 修复建议

使用 RAII 风格的锁管理：

```cpp
// 修复前
updateMutex_.lock();
// ... 可能抛出异常的代码
updateMutex_.unlock();

// 修复后
std::lock_guard<std::mutex> lock(updateMutex_);
// ... 代码
// 自动解锁（即使发生异常）
```

---

## R2: JSON 解析无大小限制（中危）

### 风险描述

多处使用 `cJSON_Parse` 解析外部 JSON 文件，未限制文件大小，大文件可导致内存耗尽（DoS）。

### 证据

| 位置 | 文件 | 行号 | 输入源 |
|------|------|------|--------|
| 1 | `oaid_service_stub.cpp` | 111 | 信任列表配置 |
| 2 | `oaid_service_stub.cpp` | 251 | Provider 配置 |
| 3 | `oaid_service.cpp` | 246 | 更新检查文件 |
| 4 | `connect_ads_stub.cpp` | 236 | Want 配置 |

**示例代码**:
```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:110-111
std::string fileContent((std::istreambuf_iterator<char>(inFile)), std::istreambuf_iterator<char>());
cJSON *root = cJSON_Parse(fileContent.c_str());  // 无大小检查
```

### 触发路径

```
攻击者 (需要 root 权限)
    │
    ├──► 创建超大 JSON 文件 (>100MB)
    │      写入 /etc/advertising/oaid/oaid_service_config.json
    │
    ├──► 触发 OAID 服务读取配置
    │      (如服务重启或 resetOAID)
    │
    ├──► cJSON_Parse 尝试解析
    │
    └──► 内存耗尽 (OOM) → 服务崩溃
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要 root 权限修改配置文件 |
| **影响** | 中 - 服务 DoS |
| **权限要求** | root |

### 修复建议

```cpp
// 读取前检查文件大小
struct stat st;
if (stat(filePath, &st) == 0 && st.st_size > MAX_JSON_SIZE) {
    OAID_HILOGE(OAID_MODULE_SERVICE, "Config file too large");
    return false;
}

// 或者使用流式解析替代 cJSON
```

---

## R3: 路径遍历风险（中危）

### 风险描述

使用 `realpath` 规范化路径后，未验证最终路径是否在预期目录内。

### 证据

```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:95-104
bool LoadAndCheckOaidTrustList(const std::string &bundleName)
{
    char pathBuff[PATH_MAX] = {0};
    GetOneCfgFile(OAID_TRUSTLIST_EXTENSION_CONFIG_PATH.c_str(), pathBuff, PATH_MAX);
    char realPath[PATH_MAX] = {0};
    if (realpath(pathBuff, realPath) == nullptr) {
        GetOneCfgFile(OAID_TRUSTLIST_CONFIG_PATH.c_str(), pathBuff, PATH_MAX);
        if (realpath(pathBuff, realPath) == nullptr) {
            return false;
        }
    }
    // ❌ 未检查 realPath 是否以 /etc/advertising/oaid/ 开头
    std::ifstream inFile(realPath, std::ios::in);
}
```

### 触发路径

```
条件: GetOneCfgFile 返回的路径可被操控
    │
    ├──► 攻击者利用符号链接或其他手段
    │      指向非预期的配置文件
    │
    ├──► realpath 解析为攻击者控制的文件
    │
    └──► 读取恶意配置 → 信任列表被绕过
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要控制 GetOneCfgFile 的搜索路径 |
| **影响** | 中 - 可能绕过信任列表检查 |
| **权限要求** | root |

### 修复建议

```cpp
// 验证路径前缀
const char* expectedPrefix = "/etc/advertising/oaid/";
if (strncmp(realPath, expectedPrefix, strlen(expectedPrefix)) != 0) {
    OAID_HILOGE(OAID_MODULE_SERVICE, "Invalid config path: %s", realPath);
    return false;
}
```

---

## R4: TOCTOU 竞争条件（中危）

### 风险描述

`ClearFile` 函数中 `lstat` → `access` → `remove` 存在 TOCTOU（检查时间-使用时间）竞争窗口。

### 证据

```cpp
// utils/native/src/oaid_file_operator.cpp:56-71
bool OAIDFileOperator::ClearFile(const std::string &fileName)
{
    struct stat statbuf {};
    if (lstat(fileName.c_str(), &statbuf) != 0) {  // T1: 检查
        return false;
    }
    if (S_ISREG(statbuf.st_mode)) {
        if (access(fileName.c_str(), F_OK) != 0) {  // T2: 再次检查
            return true;
        }
        remove(fileName.c_str());  // T3: 使用 (删除)
    }
    return true;
}
```

### 触发路径

```
时间线:
T1: lstat 检查文件是常规文件
    │
    ├──► [攻击者替换文件为符号链接]
    │
T2: access 检查文件存在
    │
    ├──► [攻击者再次替换]
    │
T3: remove 删除文件
    └──► 可能删除非预期的文件
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要精确的时间窗口控制 |
| **影响** | 中 - 可能删除非预期的文件 |
| **权限要求** | 与 OAID 服务相同 |

### 修复建议

使用 `unlinkat` 配合 `AT_REMOVEDIR` 或使用文件描述符操作：

```cpp
// 使用 unlinkat 直接删除，不做前置检查
int fd = open(dirPath, O_DIRECTORY);
unlinkat(fd, fileName, 0);
```

---

## R5: 内存分配未检查（低危）

### 风险描述

部分 `new` 操作未使用 `std::nothrow` 且未检查返回值。

### 证据

```cpp
// services/oaid_manager/src/oaid_service.cpp:117
instance_ = new OAIDService;  // 未使用 nothrow

// interfaces/innerkits/src/oaid_service_client.cpp:92
instance_ = new OAIDServiceClient;  // 未使用 nothrow
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 现代系统内存分配失败罕见 |
| **影响** | 低 - 抛出异常，程序终止 |
| **权限要求** | N/A |

### 修复建议

```cpp
// 统一使用 nothrow 并检查返回值
instance_ = new (std::nothrow) OAIDService;
if (instance_ == nullptr) {
    OAID_HILOGE(OAID_MODULE_SERVICE, "Failed to allocate OAIDService");
    return nullptr;
}
```

---

## R6: IPC 权限检查顺序问题（低危）

### 风险描述

`OnRemoteRequest` 中权限检查在 `InterfaceToken` 验证之前执行。

### 证据

```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:162-193
int32_t OAIDServiceStub::OnRemoteRequest(...)
{
    // 权限检查（先）
    if (code == GET_OAID && !CheckPermission(...)) {  // 行171
        return ...;
    }
    
    // InterfaceToken 验证（后）
    std::u16string remoteDescripter = data.ReadInterfaceToken();  // 行187
    if (myDescripter != remoteDescripter) {  // 行188
        return ERR_SYSYTEM_ERROR;
    }
}
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 极低 - IPC 框架已做基础验证 |
| **影响** | 低 - 可能记录无效的权限使用记录 |
| **权限要求** | N/A |

### 修复建议

将 `InterfaceToken` 验证移至权限检查之前：

```cpp
// 先验证 InterfaceToken
if (myDescripter != remoteDescripter) {
    return ERR_SYSYTEM_ERROR;
}

// 后做权限检查
if (code == GET_OAID && !CheckPermission(...)) {
    return ...;
}
```

---

## R7: 字符串操作潜在风险（中危）

### 风险描述

`Str16ToStr8` 和 `Str8ToStr16` 使用 `std::wstring_convert`，在 C++17 中已标记为弃用，可能存在异常安全问题。

### 证据

```cpp
// services/oaid_manager/src/oaid_service.cpp:424-436
std::string Str16ToStr8(const std::u16string &str)
{
    std::wstring_convert<std::codecvt_utf8_utf16<char16_t>, char16_t> convert;
    std::string result = convert.to_bytes(str);  // 可能抛出异常
    return result;
}
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 中 - 构造特定输入可能触发异常 |
| **影响** | 中 - 服务崩溃 |
| **权限要求** | 通过 IPC 传递恶意字符串 |

### 修复建议

使用现代替代方案：

```cpp
// 使用 C++11/17 安全转换
std::string Str16ToStr8(const std::u16string &str) {
    try {
        std::wstring_convert<std::codecvt_utf8_utf16<char16_t>, char16_t> convert;
        return convert.to_bytes(str);
    } catch (const std::range_error &e) {
        OAID_HILOGE(OAID_MODULE_SERVICE, "String conversion failed");
        return "";
    }
}
```

---

## R8: 沙盒禁用（中危）

### 风险描述

服务配置中禁用了沙盒 (`"sandbox": 0`)，增加了攻击面。

### 证据

```json
// etc/init/oaidservice.cfg
{
    "uid": "oaid_service",
    "gid": ["oaid_service", "shell"],
    "sandbox": 0,  // 沙盒禁用
    "permission": [...],
    "secon": "u:r:oaid_service:s0"
}
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 设计选择 |
| **影响** | 中 - 服务被攻破后影响范围更大 |
| **权限要求** | N/A |

### 修复建议

评估是否必须禁用沙盒，如非必要应启用：

```json
{
    "sandbox": 1,  // 启用沙盒
    "sandbox_policy": "restrictive"
}
```

---

## R9: 信任列表为空时默认允许（中危）

### 风险描述

当 `resetOAIDBundleName` 数组为空时，默认允许所有系统应用重置 OAID。

### 证据

```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:123-128
cJSON *oaidTrustConfig = cJSON_GetObjectItem(root, "resetOAIDBundleName");
// ...
int arraySize = cJSON_GetArraySize(oaidTrustConfig);
if (arraySize == 0) {
    OAID_HILOGI(OAID_MODULE_SERVICE, "oaidTrustConfig list is empty.");
    cJSON_Delete(root);
    return true;  // 空列表 = 允许所有
}
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 - 需要修改配置文件或配置错误 |
| **影响** | 中 - 权限提升（任意系统应用可重置） |
| **权限要求** | root 或配置错误 |

### 修复建议

```cpp
// 默认拒绝策略
if (arraySize == 0) {
    OAID_HILOGW(OAID_MODULE_SERVICE, "Trust list is empty, denying all");
    cJSON_Delete(root);
    return false;  // 空列表 = 拒绝所有
}
```

---

## R10: 未初始化变量（低危）

### 风险描述

部分变量未初始化或依赖默认构造函数。

### 证据

```cpp
// services/oaid_manager/src/oaid_service.cpp:327-375
bool OAIDService::InitKvStore(std::string storeIdStr)
{
    std::shared_ptr<DistributedKv::SingleKvStore> kvStore_;  // 未初始化
    if (kvStore_ == nullptr) {  // 依赖默认构造为 nullptr
        // ...
    }
}
```

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 |
| **影响** | 低 |
| **权限要求** | N/A |

### 修复建议

显式初始化：

```cpp
std::shared_ptr<DistributedKv::SingleKvStore> kvStore_ = nullptr;
```

---

## R11: 日志信息泄露（低危）

### 风险描述

日志中可能输出敏感信息（如 OAID 部分字段）。

### 证据

```cpp
// services/oaid_manager/src/oaid_service.cpp:287-293
std::string OAIDService::GetOAID()
{
    std::string oaid = GainOAID();
    std::string target = oaid.substr(0, 9).append(OAID_VIRTUAL_STR);  // 部分脱敏
    OAID_HILOGI(OAID_MODULE_SERVICE, "getOaid success");  // 未输出 OAID
    return oaid;
}
```

**注**: 实际代码做了部分脱敏，但仍需注意其他日志点。

### 影响评估

| 维度 | 评估 |
|------|------|
| **可利用性** | 低 |
| **影响** | 低 |
| **权限要求** | log 权限 |

---

## 风险汇总矩阵

| 风险 ID | 风险名称 | 严重程度 | 可利用性 | 修复优先级 |
|---------|---------|---------|---------|-----------|
| R1 | 手动 Mutex 管理 | 🔴 高 | 中 | P0 |
| R2 | JSON 解析无限制 | 🟠 中 | 低 | P1 |
| R3 | 路径遍历 | 🟠 中 | 低 | P1 |
| R4 | TOCTOU 竞争 | 🟠 中 | 低 | P1 |
| R5 | 内存分配未检查 | 🟡 低 | 低 | P2 |
| R6 | IPC 检查顺序 | 🟡 低 | 极低 | P2 |
| R7 | 字符串转换 | 🟠 中 | 中 | P1 |
| R8 | 沙盒禁用 | 🟠 中 | 设计选择 | P1 |
| R9 | 信任列表默认允许 | 🟠 中 | 低 | P1 |
| R10 | 未初始化变量 | 🟡 低 | 低 | P2 |
| R11 | 日志泄露 | 🟡 低 | 低 | P2 |

---

## 修复建议汇总

### 立即修复（P0）

1. **R1 - 手动 Mutex 管理**: 全部改为 `std::lock_guard` 或 `std::unique_lock`

### 短期修复（P1）

2. **R2 - JSON 限制**: 添加文件大小检查（如 1MB 限制）
3. **R3 - 路径验证**: 验证 `realpath` 结果在预期目录
4. **R4 - TOCTOU**: 使用文件描述符或 `unlinkat`
5. **R7 - 字符串转换**: 添加异常处理
6. **R8 - 沙盒**: 评估启用沙盒可行性
7. **R9 - 默认拒绝**: 空信任列表时默认拒绝

### 长期优化（P2）

8. **R5 - 内存检查**: 统一使用 `nothrow`
9. **R6 - 检查顺序**: 调整 IPC 验证顺序
10. **R10 - 初始化**: 显式初始化所有变量
11. **R11 - 日志审计**: 审查所有日志输出

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构分析](02_Architecture.md)
- [攻击面分析](05_AttackSurface.md)
- [代码地图](03_CodeMap.md)

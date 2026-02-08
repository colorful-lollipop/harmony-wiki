# 安全风险评审

## 1. 概述

本文档对 `resource_management_lite` 组件进行安全风险评审，分析攻击面、信任边界、潜在风险点及修复建议。

**评审范围**：
- 输入验证与过滤
- 内存安全
- 文件 I/O 安全
- 资源访问控制
- 路径遍历防护
- 并发安全

**评审依据**：
- 代码审计（`src/global.c`、`src/hap_parser.cpp` 等）
- 安全机制分析
- 威胁建模

---

## 2. 攻击面分析

### 2.1 外部输入点

| 输入源 | 接口 | 风险等级 | 风险类型 |
|--------|------|----------|----------|
| 资源 ID | `GLOBAL_GetValueById(id, path, value)` | 中 | ID伪造、越界访问 |
| 资源名称 | `GLOBAL_GetValueByName(name, path, value)` | 中 | 名称注入 |
| 语言配置 | `GLOBAL_ConfigLanguage(appLanguage)` | 低 | BCP47解析错误 |
| **HAP 文件路径** | `AddResource(path)` | **高** | **路径遍历** |
| **HAP 文件内容** | ZIP解压 | **高** | **ZIP Slip**、元数据篡改 |
| **资源索引文件** | `resources.index` | **高** | **二进制解析越界** |
| 配置文件 | `config.json` | 中 | 模块名注入 |

### 2.2 信任边界详情

```
┌─────────────────────────────────────────────────────────────────────┐
│                        信任边界图                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   不信任域 (UNTRUSTED)                                              │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  • 用户提供的 HAP 文件路径                                   │   │
│   │  • 应用私有目录中的资源文件                                  │   │
│   │  • 网络下载的 HAP 包（未验证签名）                          │   │
│   │  • 外部传入的资源 ID/名称                                    │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    输入验证层                                │   │
│   │  • CheckFilePath() - 路径校验                               │   │
│   │  • 空指针检查                                               │   │
│   │  • 缓冲区边界检查                                           │   │
│   │  ⚠️ 缺少: ZIP文件名验证、路径规范化                        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   信任域 (TRUSTED)                                                  │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  • 已签名的 HAP 包（系统预置）                              │   │
│   │  • 系统资源目录 (/system/data/)                             │   │
│   │  • 经过验证的 resources.index                              │   │
│   │  • 内部数据结构（ResDesc, IdItem等）                        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3 敏感操作清单

| 操作类型 | 涉及代码 | 风险 |
|----------|----------|------|
| **文件打开** | `hap_resource.cpp:88` (std::ifstream) | 路径遍历 |
| **ZIP文件解压** | `hap_parser.cpp:46` (unzOpen64) | ZIP Slip |
| **内存分配** | `hap_parser.cpp:72` (malloc) | 整数溢出导致DoS |
| **二进制解析** | `hap_parser.cpp:162` (ParseString) | 越界读取 |
| **全局状态修改** | `global.c:39` (g_locale) | 竞争条件 |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                          信任边界图                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                     信任边界内                                 │  │
│  │   • 已签名的 HAP 包                                           │  │
│  │   • 系统预置资源目录 (/system/data/)                         │  │
│  │   • 已验证的配置文件                                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│                              ▼                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                     信任边界外                                 │  │
│  │   • 用户提供的资源路径                                         │  │
│  │   • 应用私有目录资源                                           │  │
│  │   • 网络下载的资源（未验证来源）                              │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. 安全机制

### 3.1 内存安全机制

#### 安全字符串函数

组件使用 `bounds_checking_function` 库的安全字符串函数：

| 标准函数 | 安全替代 | 用途 |
|----------|----------|------|
| `strcpy` | `strcpy_s()` | 字符串拷贝（带边界检查） |
| `strncpy` | `strncpy_s()` | 有限字符串拷贝 |
| `malloc` | `malloc()` + 检查 | 内存分配（分配后检查） |

**代码证据**（`global.c:179-186`）：

```cpp
*value = (char *)malloc(idItem.valueLen);
if (*value == NULL || strcpy_s(*value, idItem.valueLen, idItem.value) != EOK) {
    close(file);
    free(idHeader.idParams);
    FreeIdItem(&idItem);
    FreeValue(value);
    return MC_FAILURE;
}
```

---

#### 内存分配检查

所有 `malloc()` 和 `new` 调用后均进行返回值检查：

**代码证据**（`hap_resource.cpp:99-104`）：

```cpp
void *buf = malloc(bufLen);
if (buf == nullptr) {
    HILOG_ERROR("Error allocating memory");
    inFile.close();
    return nullptr;
}
```

---

### 3.2 输入校验机制

#### 空指针检查

核心函数均包含空指针校验：

**代码证据**（`global.c:199-201`）：

```cpp
int32_t GLOBAL_GetValueById(uint32_t id, const char *path, char **value)
{
    if (path == NULL || path[0] == '\0' || value == NULL) {
        return MC_FAILURE;
    }
    // ...
}
```

---

#### 缓冲区边界检查

**代码证据**（`hap_parser.cpp:201-209`）：

```cpp
uint32_t readSize = offset - startOffset;
if (readSize + 1 == arrLen) {
    offset += 1;  // after arrLen, got '\0'
    break;
}
if (readSize + 1 > arrLen) {
    // size not match, cannot > arrLen
    return SYS_ERROR;
}
```

---

### 3.3 文件 I/O 安全

#### ZIP 文件安全读取

`HapParser::ReadFileFromZip()` 实现多层检查：

**代码证据**（`hap_parser.cpp:46-102`）：

```cpp
int32_t HapParser::ReadFileFromZip(const char *zipFile, const char *fileName,
    void **buffer, size_t &bufLen, std::string &errInfo)
{
    // 1. 打开 ZIP 文件检查
    unzFile uf = unzOpen64(zipFile);
    if (uf == nullptr) {
        errInfo = FormatString("Cannot open %s", zipFile);
        return UNKNOWN_ERROR;
    }
    
    // 2. 定位文件检查
    if (unzLocateFile(uf, fileName, 1)) {
        unzClose(uf);
        return UNKNOWN_ERROR;
    }
    
    // 3. 获取文件信息检查
    if (unzGetCurrentFileInfo(uf, &fileInfo, filenameInzip,
        sizeof(filenameInzip), nullptr, 0, nullptr, 0)) {
        unzClose(uf);
        return UNKNOWN_ERROR;
    }
    
    // 4. 内存分配检查
    *buffer = static_cast<void *>(malloc(fileInfo.uncompressed_size));
    if ((*buffer) == nullptr) {
        unzClose(uf);
        return UNKNOWN_ERROR;
    }
    
    // 5. 读取数据检查
    err = unzReadCurrentFile(uf, *buffer, bufLen);
    if (err < 0) {
        free(*buffer);
        *buffer = nullptr;
        return UNKNOWN_ERROR;
    }
}
```

---

#### 文件大小验证

**代码证据**（`hap_resource.cpp:92-98`）：

```cpp
inFile.seekg(0, std::ios::end);
int bufLen = inFile.tellg();
if (bufLen <= 0) {
    HILOG_ERROR("file size is zero");
    inFile.close();
    return nullptr;
}
```

---

### 3.4 资源引用安全

#### 循环引用检测

**代码证据**（`resource_manager_impl.cpp:271-307`）：

```cpp
int count = 0;
while (isRef) {
    // ... 解析引用
    if (++count > MAX_DEPTH_REF_SEARCH) {
        HILOG_ERROR("ref %s has re-ref too much", value.c_str());
        return ERROR;
    }
}
```

---

#### 父引用解析深度限制

**代码证据**（`resource_manager_impl.cpp:311-373`）：

```cpp
int count = 0;
do {
    // ... 解析父引用
    if (++count > MAX_DEPTH_REF_SEARCH) {
        HILOG_ERROR(" %u has too many parents", idItem->id_);
        return ERROR;
    }
} while (haveParent);
```

---

### 3.5 并发安全

#### 互斥锁保护

**代码证据**（`hap_manager.h` / `hap_manager.cpp:55-98`）：

```cpp
class HapManager {
private:
    Lock lock_;  // 互斥锁保护所有成员
    std::vector<HapResource*> hapResources_;
    std::vector<std::string> loadedHapPaths_;
    ResConfigImpl *resConfig_;
};

std::string HapManager::GetPluralRulesAndSelect(int quantity)
{
    AutoMutex mutex(this->lock_);  // RAII 自动加锁/解锁
    // ... 临界区操作
}
```

---

## 4. 可被利用点分析

### 4.1 高风险点

#### 风险 1：ZIP Slip 路径遍历漏洞（高风险）

**位置**：`frameworks/resmgr_lite/src/utils/hap_parser.cpp:46-102`

**证据**：
```cpp
// hap_parser.cpp:46-102
int32_t HapParser::ReadFileFromZip(const char *zipFile, const char *fileName, 
                                   void **buffer, size_t &bufLen, std::string &errInfo) {
    char filenameInzip[256];  // 固定大小缓冲区
    unz_file_info fileInfo;
    
    unzFile uf = unzOpen64(zipFile);
    // ...
    *buffer = static_cast<void *>(malloc(fileInfo.uncompressed_size)); // 基于ZIP元数据分配
    bufLen = fileInfo.uncompressed_size;
    // ...
    err = unzReadCurrentFile(uf, *buffer, bufLen);
}
```

**触发路径**：
```
恶意HAP文件 → unzOpen64() → unzGetCurrentFileInfo() → filenameInzip[256]
→ 文件名包含"../etc/passwd" → 路径遍历 → 访问系统任意文件
```

**影响评估**：
- **可利用性**: 高（构造恶意ZIP文件即可触发）
- **影响范围**: 可读取系统任意文件
- **权限要求**: 需有HAP文件写入权限

**修复建议**：
```cpp
// 添加ZIP文件名路径遍历检测
bool IsValidZipEntryName(const char* filename) {
    // 检查是否包含路径遍历序列
    if (strstr(filename, "..") != nullptr || filename[0] == '/') {
        return false;
    }
    // 检查文件名长度
    if (strlen(filename) >= 256) {
        return false;
    }
    return true;
}

// 在ReadFileFromZip中使用
if (!IsValidZipEntryName(filenameInzip)) {
    unzClose(uf);
    errInfo = "Invalid zip entry name";
    return UNKNOWN_ERROR;
}
```

---

#### 风险 2：ZIP元数据整数溢出（高风险）

**位置**：`frameworks/resmgr_lite/src/utils/hap_parser.cpp:72`

**证据**：
```cpp
// hap_parser.cpp:72
*buffer = static_cast<void *>(malloc(fileInfo.uncompressed_size));
bufLen = fileInfo.uncompressed_size;
```

**问题**：`fileInfo.uncompressed_size`来自ZIP文件元数据，可被恶意篡改。

**触发条件**：
- 构造ZIP文件，设置`uncompressed_size`为极大值（如0xFFFFFFFF）
- 导致`malloc()`分配超大内存或溢出

**影响评估**：
- **可利用性**: 中（需构造特定ZIP文件）
- **影响**: 内存耗尽(DoS)或内存损坏

**修复建议**：
```cpp
// 添加大小上限检查
const size_t MAX_ZIP_ENTRY_SIZE = 100 * 1024 * 1024; // 100MB上限

if (fileInfo.uncompressed_size > MAX_ZIP_ENTRY_SIZE) {
    unzClose(uf);
    errInfo = "Zip entry too large";
    return UNKNOWN_ERROR;
}

*buffer = static_cast<void *>(malloc(fileInfo.uncompressed_size));
if (*buffer == nullptr) {
    unzClose(uf);
    return NOT_ENOUGH_MEM;
}
```

---

### 4.2 中风险点

#### 风险 3：文件路径遍历风险（中风险）

**位置**：`frameworks/resmgr_lite/src/hap_resource.cpp:88`

**证据**：
```cpp
// hap_resource.cpp:88-107
std::ifstream inFile(path, std::ios::binary | std::ios::in);
if (!inFile.good()) {
    return nullptr;
}
```

**问题**：`path`参数直接传入文件流，未进行规范化验证。

**触发路径**：
```
AddResource("../../../etc/passwd") → HapResource::LoadFromIndex() 
→ std::ifstream 打开任意文件
```

**修复建议**：
```cpp
// 在hap_resource.cpp中添加路径验证
bool ValidateResourcePath(const char* path) {
    // 规范化路径
    char resolved[PATH_MAX];
    if (realpath(path, resolved) == nullptr) {
        return false;
    }
    
    // 检查是否在允许目录内
    const char* allowedPaths[] = {"/system/data/", "/data/app/"};
    bool valid = false;
    for (const auto& prefix : allowedPaths) {
        if (strncmp(resolved, prefix, strlen(prefix)) == 0) {
            valid = true;
            break;
        }
    }
    return valid;
}
```

---

#### 风险 4：动态路径构造风险（中风险）

**位置**：`frameworks/resmgr_lite/src/utils/hap_parser.cpp:147-151`

**证据**：
```cpp
// hap_parser.cpp:147-151
std::string indexFilePath = std::string("assets/");
indexFilePath.append(mName);  // 来自config.json的模块名
indexFilePath.append(RES_FILE_NAME);  // "/resources.index"

return ReadFileFromZip(zipFile, indexFilePath.c_str(), buffer, bufLen, errInfo);
```

**问题**：`mName`来自配置文件解析，如果包含`../`可导致路径遍历。

**修复建议**：
```cpp
// 添加模块名验证
if (mName.find("..") != std::string::npos || mName.find('/') != std::string::npos) {
    return UNKNOWN_ERROR;  // 非法模块名
}
```

---

#### 风险 5：全局状态竞争条件（中风险）

**位置**：`frameworks/resmgr_lite/src/global.c:39`

**证据**：
```c
// global.c:39
static char g_locale[MAX_LOCALE_LENGTH] = {0};
```

**问题**：全局变量`g_locale`存储语言配置，无锁保护。

**触发场景**：
```
线程1: GLOBAL_ConfigLanguage("zh") 
       写入g_locale[0] = 'z'
线程2: GLOBAL_GetValueById() 
       读取g_locale → 读到"z"（不完整）
```

**修复建议**：
```cpp
// 添加互斥锁保护
static Lock g_resmgrLock;
static char g_locale[MAX_LOCALE_LENGTH] = {0};

void GLOBAL_ConfigLanguage(const char *appLanguage)
{
    AutoMutex mutex(&g_resmgrLock);
    // ...
}

int32_t GLOBAL_GetValueById(uint32_t id, const char *path, char **value)
{
    AutoMutex mutex(&g_resmgrLock);
    // ...
}
```

---

### 4.3 低风险点

#### 风险 6：二进制缓冲区解析越界（低风险）

**位置**：`frameworks/resmgr_lite/src/utils/hap_parser.cpp:162-174`

**证据**：
```cpp
// hap_parser.cpp:162-174
int32_t ParseString(const char *buffer, uint32_t &offset, std::string &id, bool includeTemi = true) {
    uint16_t strLen;
    errno_t eret = memcpy_s(&strLen, sizeof(strLen), buffer + offset, 2);
    offset += 2;
    std::string tmp = std::string(const_cast<char *>(buffer) + offset, 
                                  includeTemi ? (strLen - 1) : strLen);
    offset += includeTemi ? strLen : (strLen + 1);
```

**问题**：`strLen`来自文件数据，如果恶意篡改可能导致越界读取。

**缓解措施**：虽然使用了`memcpy_s`，但偏移量计算仍可能溢出。

---

#### 风险 7：偏移量整数溢出（低风险）

**位置**：`frameworks/resmgr_lite/src/utils/hap_parser.cpp:170`

**证据**：
```cpp
offset += strLen;  // 可能溢出
```

**修复建议**：
```cpp
// 添加溢出检查
if (offset > UINT32_MAX - strLen) {
    return ERROR;
}
offset += strLen;
```
应用传入恶意构造的路径（如 "../etc/passwd"）
```

**影响**：可能访问超出预期目录的文件。

**证据**（`global_utils.h:106`）：

```c
typedef struct GlobalUtilsImpl {
    // ...
    int32_t (*CheckFilePath)(const char *path, char *realResourcePath, int32_t length);
} GlobalUtilsImpl;
```

**修复建议**：
1. 确认 `CheckFilePath` 的具体实现包含路径标准化
2. 添加 `realpath()` 或类似函数进行路径规范化
3. 检查路径中的 `..` 遍历序列
4. 限制访问路径在允许的目录范围内

---

#### 风险 2：资源 ID 越界访问（低风险）

**位置**：`hap_resource.cpp:212` - `uint32_t uid = id`

**说明**：资源 ID 直接使用，未进行范围验证。如果 ID 超出预期范围可能导致读取无效内存。

**触发条件**：
```
构造非预期的资源 ID 进行查询
```

**影响**：可能返回错误数据或触发未定义行为。

**证据**（`hap_resource.cpp:212`）：

```cpp
uint32_t uid = id;  // 无 ID 范围检查
auto it = idValuesMap_.find(uid);
```

**修复建议**：
```cpp
// 添加 ID 范围验证
if (id < MIN_RESOURCE_ID || id > MAX_RESOURCE_ID) {
    return NOT_FOUND;
}
uint32_t uid = id;
```

---

#### 风险 3：标准字符串函数使用（低风险）

**位置**：`utils.cpp:83-88` - `strLen` 使用标准 `strlen`

**说明**：部分工具函数使用标准 `strlen` 而非安全版本。输入已在调用前验证，风险较低。

**触发条件**：
```
传递超长字符串（但应在调用前被拒绝）
```

**影响**：潜在缓冲区溢出风险（已被输入验证缓解）。

**证据**（`utils.cpp:83-88`）：
```cpp
uint32_t strLen = strlen(input);
```

**修复建议**：确认所有字符串长度计算后立即进行边界检查，或替换为 `strnlen_s()`。

---

#### 风险 4：整数溢出风险（低风险）

**位置**：`hap_parser.cpp:170` - `offset += strLen`

**说明**：`offset` 累加操作可能发生整数溢出。

**触发条件**：
```
处理异常构造的 HAP 文件
```

**影响**：可能导致内存访问越界。

**证据**（`hap_parser.cpp:170`）：
```cpp
offset += strLen;  // 可能溢出
```

**修复建议**：
```cpp
// 添加溢出检查
if (offset > UINT32_MAX - strLen) {
    return ERROR;
}
offset += strLen;
```

---

#### 风险 5：全局状态竞争条件（中等风险）

**位置**：`global.c:39` - `static char g_locale[MAX_LOCALE_LENGTH]`

**说明**：全局变量 `g_locale` 存储当前语言配置，在多线程环境下存在竞争风险。

**触发条件**：
```
多线程同时调用 GLOBAL_ConfigLanguage 和资源查询
```

**影响**：可能导致语言配置不一致，返回错误语言的资源。

**证据**（`global.c:39`）：
```cpp
static char g_locale[MAX_LOCALE_LENGTH] = {0};
```

**修复建议**：
```cpp
// 添加互斥锁保护
static Lock g_localeLock;
static char g_locale[MAX_LOCALE_LENGTH] = {0};

void GLOBAL_ConfigLanguage(const char *appLanguage)
{
    AutoMutex mutex(&g_localeLock);
    // ...
}
```

---

### 4.2 风险汇总

| 风险ID | 风险类型 | 位置 | 等级 | 可利用性 | 影响 | 状态 |
|--------|----------|------|------|----------|------|------|
| R1 | ZIP Slip路径遍历 | `hap_parser.cpp:50` | **高** | 高 | 任意文件读取 | **需立即修复** |
| R2 | ZIP元数据整数溢出 | `hap_parser.cpp:72` | **高** | 中 | DoS/内存损坏 | **需立即修复** |
| R3 | 文件路径遍历 | `hap_resource.cpp:88` | **中** | 中 | 任意文件读取 | **建议修复** |
| R4 | 动态路径构造 | `hap_parser.cpp:147` | **中** | 中 | 路径遍历 | **建议修复** |
| R5 | 全局状态竞争 | `global.c:39` | **中** | 低 | 配置混乱 | **建议修复** |
| R6 | 缓冲区解析越界 | `hap_parser.cpp:162` | 低 | 低 | 越界读取 | 已部分缓解 |
| R7 | 偏移量整数溢出 | `hap_parser.cpp:170` | 低 | 低 | 内存损坏 | 建议添加检查 |

---

## 5. 安全建议

### 5.1 高优先级

#### 1. 完善路径遍历防护

```c
// 推荐实现
int32_t CheckFilePath(const char *path, char *realPath, int32_t len)
{
    if (path == NULL || strlen(path) >= PATH_MAX) {
        return MC_FAILURE;
    }
    
    // 路径标准化
    char resolved[PATH_MAX];
    if (realpath(path, resolved) == NULL) {
        return MC_FAILURE;
    }
    
    // 检查是否在允许目录内
    const char *allowedPrefix = "/system/data/";
    if (strncmp(resolved, allowedPrefix, strlen(allowedPrefix)) != 0) {
        return MC_FAILURE;
    }
    
    // 拷贝到输出缓冲区
    if (strcpy_s(realPath, len, resolved) != EOK) {
        return MC_FAILURE;
    }
    
    return MC_SUCCESS;
}
```

---

#### 2. 添加全局状态保护

```cpp
// 推荐实现
static Lock g_resmgrLock;
static char g_locale[MAX_LOCALE_LENGTH] = {0};

void GLOBAL_ConfigLanguage(const char *appLanguage)
{
    AutoMutex mutex(&g_resmgrLock);
    if (appLanguage == NULL) {
        return;
    }
    // ...
}

int32_t GLOBAL_GetValueById(uint32_t id, const char *path, char **value)
{
    // 读取时也加锁，确保一致性
    AutoMutex mutex(&g_resmgrLock);
    // ...
}
```

---

### 5.2 中优先级

#### 3. 整数溢出防护

```cpp
// 推荐实现
uint32_t offset = startOffset;
for (uint32_t i = 0; i < count; i++) {
    uint32_t strLen = GetStringLength(data + offset);
    
    // 溢出检查
    if (offset > UINT32_MAX - strLen) {
        return ERROR;
    }
    
    offset += strLen;
}
```

---

#### 4. 资源 ID 验证

```cpp
// 推荐实现
bool IsValidResourceId(uint32_t id)
{
    // 根据实际 ID 分配策略调整范围
    return (id >= MIN_ID && id <= MAX_ID);
}

// 在资源查询时验证
if (!IsValidResourceId(id)) {
    return ERROR;
}
```

---

## 6. 安全依赖审计

### 6.1 依赖库安全

| 组件 | 版本 | 安全审计 | 备注 |
|------|------|----------|------|
| `bounds_checking_function` | - | OpenHarmony 安全库 | 安全字符串实现 |
| `zlib` | - | 广泛审计 | ZIP 解压 |
| `minizip` | - | zlib 配套 | ZIP 文件读取 |

### 6.2 供应链安全

**建议**：
1. 定期更新依赖库版本
2. 使用锁定的依赖版本
3. 对 HAP 包进行签名验证

---

## 7. 检查局限性

### 7.1 本次评审未覆盖范围

| 范围 | 说明 |
|------|------|
| 运行时权限验证 | 未审查系统层面的权限检查 |
| 跨应用资源隔离 | 未审查应用间资源访问控制 |
| 加密存储 | 未审查资源文件加密 |
| 网络资源 | 未审查网络下载资源的安全验证 |

### 7.2 建议后续审查

1. **系统集成安全**：审查与应用框架的安全集成
2. **签名验证**：审查 HAP 包签名验证机制
3. **沙箱隔离**：审查资源访问的沙箱边界

---

## 8. 文档链接

| 主题 | 文档 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](./01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| C API | [03_C_API.md](./03_C_API.md) |
| C++ API | [04_Cpp_API.md](./04_Cpp_API.md) |
| 构建系统 | [05_Build_System.md](./05_Build_System.md) |
| 常见问题 | [07_Troubleshooting.md](./07_Troubleshooting.md) |

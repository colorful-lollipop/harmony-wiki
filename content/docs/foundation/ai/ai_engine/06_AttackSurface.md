# AI Engine 攻击面分析

> 适用读者：安全研究员、代码审计人员
> 依赖章节：了解基本架构后阅读本章节
> 更新时间：2026-02-07

---

## 概述

AI Engine 作为 OpenHarmony 系统服务，通过 IPC 接受客户端请求并动态加载插件执行推理。本章节分析所有外部输入点、特权操作和信任边界，识别潜在的安全风险。

### 攻击面总结

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    不信任域                          │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────┐  │
│  │  Client App    │───▶│   SDK Layer    │───▶│  Client Executor    │  │
│  │    (Untrusted) │    │   (KWSSdk etc)  │    │  (client_factory)   │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼ IPC (Binder/Samgr)
┌─────────────────────────────────────────────────────────────────────────┐
│                     信任域              │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────┐  │
│  │   SA Server     │───▶│ Server Executor │───▶│   Plugin Manager    │  │
│  │  (sa_server.c)  │    │ (engine_manager)│    │  (plugin_manager)   │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────┘  │
│                                    │                   │                │
│                                    ▼                   ▼                │
│                           ┌─────────────────┐    ┌─────────────────┐   │
│                           │  Engine Worker  │    │  Plugin (.so)   │   │
│                           │  (Async/Sync)   │    │ (Loaded via dl) │   │
│                           └─────────────────┘    └─────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

**信任边界**:
1. **Application → SDK**: 客户端应用代码，完全受信
2. **SDK → IPC**: 序列化数据，通过 Samgr 传输
3. **IPC → Server**: 系统服务，运行在信任域
4. **Server → Plugin**: 动态加载第三方代码

---

## 1. 外部输入点

### 1.1 IPC 数据输入

**位置**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp`

| 输入点 | 行号 | 输入类型 | 验证方式 |
|--------|------|----------|-----------|
| `ParcelDataInfo()` | 211-242 | `DataInfo` (data指针 + length) | Null 检查、长度非零验证 |
| `UnParcelDataInfo()` | 244-270 | IPC data | 长度边界检查（< 0 被拒绝） |

**验证代码**:
```cpp
// aie_ipc.cpp:221-228
if (dataInfo->data != nullptr && dataInfo->length <= 0) {
    return;  // 拒绝：数据为空但长度非零
}
if (dataInfo->data == nullptr && dataInfo->length != 0) {
    return;  // 拒绝：数据为空指针但长度非零
}
```

**风险**:
- ⚠️ **长度溢出**：`dataInfo->length` 是 `int` 类型，但在某些地方与 `size_t` 比较
- ⚠️ **共享内存绕过**：如果长度为 0 但数据指针有效，可能绕过检查

**证据**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:221-228`

---

### 1.2 客户端信息输入

**位置**: `services/common/protocol/struct_definition/aie_info_define.h`

| 输入点 | 行号 | 输入类型 | 验证方式 |
|--------|------|----------|-----------|
| `ClientInfo` | 25 | clientId, sessionId, serverUid, clientUid | 未发现客户端 UID 验证 |
| `AlgorithmInfo` | 39 | algorithmType, version, isAsync, isCloud | 未发现算法类型白名单验证 |

**结构定义**:
```cpp
// aie_info_define.h:25
typedef struct ClientInfo {
    long long clientVersion;
    int clientId;           // 由服务器生成
    int sessionId;          // 由客户端生成
    uid_t serverUid;        // 用于共享内存
    uid_t clientUid;        // 【关键】客户端 UID，未验证
    int extendLen;
    unsigned char *extendMsg;
} ClientInfo;
```

**风险**:
- 🔴 **缺失客户端身份验证**：`clientUid` 来自不信任域，但服务端不验证
- 🔴 **缺少算法白名单**：`algorithmType` 是任意 int，未验证是否为允许的类型
- 🔴 **可扩展消息**：`extendMsg` 是原始指针，长度由 `extendLen` 控制，未发现深度验证

**证据**: `services/common/protocol/struct_definition/aie_info_define.h:25-25`

---

### 1.3 插件加载输入

**位置**: `services/server/plugin_manager/source/plugin_label.cpp`

| 输入点 | 行号 | 输入类型 | 验证方式 |
|--------|------|----------|-----------|
| `GetLibPath()` | 56-80 | aid（字符串） + version（long long） | 仅硬编码路径查找 |

**路径映射**:
```cpp
// plugin_label.cpp:61-73
} else if (label == "cv_image_classification+20001001") {
    libPath = "/usr/lib/libcv_image_classification.so";
} else if (label == "asr_keyword_spotting+20001002") {
    libPath = "/usr/lib/libasr_keyword_spotting.so";
}
```

**风险**:
- 🔴 **路径注入**：如果 `label` 格式可控，可能遍历目录（但当前为硬编码，风险较低）
- ⚠️ **符号链接攻击**：插件文件可被替换为恶意库（无签名验证）

**证据**: `services/server/plugin_manager/source/plugin_label.cpp:61-73`

---

### 1.4 反序列化输入

**位置**: `services/common/utils/encdec/include/data_decoder.h`

| 输入点 | 行号 | 输入类型 | 验证方式 |
|--------|------|----------|-----------|
| `DecodeOneParameter<T>()` | 93-116 | 模板类型 T | `Ensure()` 边界检查 |

**边界检查**:
```cpp
// data_decoder.h:123-126
inline bool Ensure(const size_t size) const {
    return (size <= size_ && (pos_ + size) <= size_);
}
```

**风险**:
- 🔴 **类型混淆**：`DecodeOneParameter<T>` 依赖模板类型匹配，如果编码/解码不匹配会导致类型混淆
- 🔴 **整数溢出**：`pos_ += sizeof(T)` 未检查溢出，如果 `size_` 被破坏
- 🔴 **缓冲区溢出**：`Ensure()` 假设 `size_` 有效，但如果数据损坏可能导致 OOB

**证据**: `services/common/utils/encdec/include/data_decoder.h:93-116`

---

## 2. 特权操作

### 2.1 共享内存操作

**位置**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp`

| 操作 | 行号 | 权限 | 风险等级 |
|------|------|-----------|------------|
| `shmget(IPC_CREAT | IPC_EXCL)` | 56 | 创建共享内存段 | **高** |
| `shmat()` | 69, 131 | 附加到共享内存 | **中** |
| `shmctl(IPC_RMID)` | 38 | 删除共享内存 | **低** |
| `shmctl(IPC_SET)` 修改 UID | 97 | 更改 SHM 所有权 | **中** |

**shmKey 循环逻辑**:
```cpp
// aie_ipc.cpp:35-57
for (shmKey = SHM_KEY_START; shmKey <= SHM_KEY_END; ++shmKey) {
    shmId = shmget(shmKey, dataInfo->length, 
                SHM_READ_WRITE_PERMISSIONS | IPC_CREAT | IPC_EXCL);
    if (shmId >= 0) {
        break;
    }
}
```

**风险**:
- 🔴 **整数碰撞/溢出**：`shmKey` 在 200000-300000 范围循环，10 万次分配后会绕回
- 🔴 **竞态条件（TOCTOU）**：`shmget` 和后续 `shmat` 之间无原子性保护
- 🔴 **无大小上限**：分配共享内存前未检查最大大小限制
- 🔴 **TOCTOU in shmctl**：`IPC_STAT` 和 `IPC_SET` 序列非原子，可能被利用

**利用场景**：
1. 恶意客户端重复调用分配，消耗所有可用的 SHM key
2. 攻击者通过竞态条件，在 `shmget` 和 `shmat` 之间替换共享内存段
3. 通过构造超大的 `dataInfo->length`，导致 DoS（内存耗尽）

**证据**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:35-242`

---

### 2.2 动态代码加载

**位置**: `services/common/platform/dl_operation/source/aie_dl_operation.cpp`

| 操作 | 行号 | 权限 | 风险等级 |
|------|------|-----------|------------|
| `dlopen()` | 53 | 加载动态库 | **高** |
| `dlsym()` | 64 | 解析符号 | **高** |
| `AieDlopen()` | 32-55 | dlopen 包装（带路径验证） | **中** |

**路径验证**:
```cpp
// aie_dl_operation.cpp:35-51
const char *LIB_PATH = "/usr/";
unsigned int length = strlen(LIB_PATH);
if ((libName == nullptr) || (strlen(libName) < length)) {
    return nullptr;
}
char realLibPath[PATH_MAX + 1] = {0};
if (realpath(libName, realLibPath) == nullptr) {
    return nullptr;
}
int retCode = strncmp(realLibPath, LIB_PATH, length);
if (retCode != 0) {
    return nullptr;  // 拒绝 /usr/ 外的路径
}
```

**风险**:
- 🔴 **无签名验证**：插件 .so 文件未进行密码学签名检查
- 🔴 **无版本绑定**：版本号未与二进制签名绑定
- 🔴 **符号加载任意性**：`dlsym("PLUGIN_INTERFACE")` 加载任意符号
- 🔴 **路径依赖**：如果 `/usr/lib/` 可写，攻击者可替换合法插件

**利用场景**：
1. 如果攻击者获得对 `/usr/lib/` 的写访问（例如通过提权漏洞）
2. 攻击者将恶意库替换为合法插件名称（如 `libcv_image_classification.so`）
3. 恶意库在下次 `LoadPlugin()` 时被加载，获得代码执行权限

**证据**: `services/common/platform/dl_operation/source/aie_dl_operation.cpp:32-64`

---

### 2.3 文件系统操作

**位置**: `services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp`（示例插件）

| 操作 | 文件 | 用途 | 风险 |
|------|------|------|------|
| 读取模型文件 | `/storage/data/keyword_spotting.wk` | 加载 AI 模型 | 低（模型文件固定） |
| 读取配置文件 | `/storage/data/kws_mean.txt` | 加载 MFCC 参数 | 低（配置文件固定） |

**风险**:
- ⚠️ **路径遍历**：如果模型路径可配置（当前为硬编码），存在遍历风险
- ⚠️ **模型注入**：如果模型文件可替换，可能注入恶意模型

**证据**: 示例插件分析（实际风险取决于插件实现）

---

### 2.4 系统调用

**位置**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp`

| 系统调用 | 行号 | 参数验证 | 风险 |
|----------|------|----------|------|
| `shmget()` | 56 | shmKey（可计算）、length、权限 | **高**（整数溢出风险） |
| `shmat()` | 69, 131 | shmId（可能被操纵）、权限 | **中**（无效 shmId 导致段错误） |
| `shmctl()` | 38, 91, 97 | shmId、命令、权限 | **中**（命令参数注入） |

**证据**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:35-242`

---

## 3. 信任边界

### 3.1 域间边界

| 边界 | 信任域 | 跨越机制 | 验证 |
|------|--------|---------|------|
| Application → SDK | 不信任 | 直接 C++ 调用 | ❌ 无 |
| SDK → IPC | 受控 | Samgr 传输 | ✅ 有（Binder） |
| IPC → Server | 系统服务 | 接收 IPC | ⚠️ 部分（缺少 UID 验证） |
| Server → Plugin | 动态代码 | dlopen 加载 | ⚠️ 部分（路径验证但无签名） |

### 3.2 数据流信任级

```
[不信任] Application → [不信任] SDK → [验证] IPC → [系统服务] Server → [验证？] Plugin [动态加载]
    ↓                        ↓              ↓                        ↓
  用户输入              序列化          验证不足                签名缺失
```

**关键观察**:
- ✅ IPC 层有 Samgr 保护
- 🔴 服务端缺少客户端 UID 验证
- 🔴 插件加载无签名验证
- 🔴 反序列化依赖正确的 `size_` 值

**证据**: 架构和代码分析

---

## 4. 缺失的安全控制

### 4.1 插件签名验证

**当前状态**：❌ **未实现**

**影响**：攻击者可替换合法插件库，获得代码执行

**修复建议**：
```cpp
// 建议在 plugin_manager.cpp 添加
int PluginManager::VerifyPluginSignature(const std::string &libPath) {
    // 1. 读取插件签名文件
    std::string sigPath = libPath + ".sig";
    
    // 2. 使用系统可信密钥验证签名
    if (!CryptoVerify(libPath, sigPath, TRUSTED_PUBLIC_KEY)) {
        return RETCODE_SIGNATURE_INVALID;
    }
    
    return RETCODE_SUCCESS;
}

// 在 LoadPlugin() 中调用
int PluginManager::LoadPlugin(...) {
    // ... path validation ...
    
    // 添加签名验证
    if (VerifyPluginSignature(libPath) != RETCODE_SUCCESS) {
        return RETCODE_PLUGIN_SIGNATURE_FAILED;
    }
    
    return AieDlopen(libPath);
}
```

**证据**: `services/server/plugin_manager/source/plugin.cpp:80-108`

---

### 4.2 共享内存大小限制

**当前状态**：⚠️ **部分实现**

**影响**：恶意客户端可分配超大共享内存段，导致 DoS

**修复建议**：
```cpp
// 建议在 aie_ipc.cpp 添加
constexpr size_t MAX_SHM_SIZE = 10 * 1024 * 1024;  // 10MB

int IpcIoPushSharedMemory(...) {
    // 添加大小检查
    if (dataInfo->length > MAX_SHM_SIZE) {
        return RETCODE_SHM_SIZE_TOO_LARGE;
    }
    
    // ... existing code ...
}
```

**证据**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:211-242`

---

### 4.3 客户端 UID 验证

**当前状态**：❌ **未实现**

**影响**：任何客户端可连接并调用服务

**修复建议**：
```cpp
// 建议在 sa_server_adapter.cpp 添加
int SaServerAdapter::ValidateClient(const ClientInfo &clientInfo) {
    // 允许的 UID 白名单
    static const uid_t ALLOWED_UIDS[] = {
        SYSTEM_UID,  // 系统服务
        APP_A_UID,    // 特定应用 A
        APP_B_UID     // 特定应用 B
    };
    
    // 验证 clientUid
    for (size_t i = 0; i < sizeof(ALLOWED_UIDS) / sizeof(uid_t); ++i) {
        if (clientInfo.clientUid == ALLOWED_UIDS[i]) {
            return RETCODE_SUCCESS;
        }
    }
    
    return RETCODE_CLIENT_UID_NOT_ALLOWED;
}

// 在 SyncExecute() 中调用
int SaServerAdapter::SyncExecute(...) {
    int retCode = ValidateClient(clientInfo);
    if (retCode != RETCODE_SUCCESS) {
        return retCode;
    }
    
    // ... existing code ...
}
```

**证据**: `services/server/communication_adapter/source/sa_server_adapter.cpp:218-246`

---

### 4.4 算法类型白名单

**当前状态**：❌ **未实现**

**影响**：客户端可指定任意算法类型，触发未预期行为

**修复建议**：
```cpp
// 建议在 sa_server_adapter.cpp 添加
int SaServerAdapter::ValidateAlgorithmType(int algorithmType) {
    static const int ALLOWED_TYPES[] = {
        ALGORITHM_TYPE_KWS,
        ALGORITHM_TYPE_IC,
        ALGORITHM_TYPE_CR
    };
    
    for (size_t i = 0; i < sizeof(ALLOWED_TYPES) / sizeof(int); ++i) {
        if (algorithmType == ALLOWED_TYPES[i]) {
            return RETCODE_SUCCESS;
        }
    }
    
    return RETCODE_ALGORITHM_TYPE_NOT_ALLOWED;
}

// 在 ConvertToRequest() 中调用
void SaServerAdapter::ConvertToRequest(...) {
    int retCode = ValidateAlgorithmType(algoInfo.algorithmType);
    if (retCode != RETCODE_SUCCESS) {
        // return error
    }
    
    // ... existing code ...
}
```

**证据**: `services/common/protocol/struct_definition/aie_info_define.h:39-50`

---

### 4.5 共享内存访问控制

**当前状态**：⚠️ **部分实现**

**影响**：客户端可访问其他客户端的共享内存（如果 shmId 泄露）

**修复建议**：
```cpp
// 建议在 aie_ipc.cpp 添加
int IpcIoPushSharedMemory(IpcIo *request, const DataInfo *dataInfo, uid_t receiverUid) {
    // 当前代码已经设置 UID
    shmidDs.shm_perm.uid = receiverUid;  // give receiver privilege
    
    // 添加所有者验证
    shmidDs.shm_perm.uid = geteuid();  // owner must be server
    
    // ... existing code ...
}
```

**证据**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:97`

---

### 4.6 速率限制

**当前状态**：❌ **未实现**

**影响**：恶意客户端可高频调用，导致 DoS

**修复建议**：
```cpp
// 建议在 sa_server_adapter.cpp 添加
class RateLimiter {
private:
    std::map<uid_t, std::pair<uint64_t, uint64_t>> requestCount_;
    
public:
    bool CheckRateLimit(uid_t clientUid) {
        uint64_t now = GetCurrentTimeMs();
        constexpr uint64_t WINDOW_MS = 1000;  // 1 秒窗口
        constexpr uint64_t MAX_REQUESTS = 100;  // 每秒最多 100 次
        
        auto &entry = requestCount_[clientUid];
        if (now - entry.second < WINDOW_MS) {
            if (entry.first >= MAX_REQUESTS) {
                return false;  // 超过速率限制
            }
            entry.first++;
        } else {
            entry.first = 1;
        }
        entry.second = now;
        
        return true;
    }
};

// 在 SyncExecute() 中调用
int SaServerAdapter::SyncExecute(...) {
    if (!rateLimiter.CheckRateLimit(clientInfo.clientUid)) {
        return RETCODE_RATE_LIMIT_EXCEEDED;
    }
    
    // ... existing code ...
}
```

**证据**: `services/server/communication_adapter/source/sa_server_adapter.cpp:218-246`

---

## 5. 攻击场景

### 场景 1：共享内存耗尽

**攻击步骤**:
1. 恶意客户端重复调用 `AieClientSyncProcess()`
2. 每次调用触发共享内存分配（`IpcIoPushSharedMemory()`）
3. `shmKey` 在 200000-300000 范围循环
4. 10 万次分配后，所有 SHM key 耗尽
5. 后续合法客户端无法分配共享内存，服务不可用

**代码位置**：`services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:35-57`

**缓解措施**：
- ✅ 添加速率限制
- ✅ 添加共享内存大小上限
- ✅ 使用非循环的 shmKey 分配策略

---

### 场景 2：插件替换攻击

**攻击步骤**:
1. 攻击者获得对 `/usr/lib/` 的写权限（通过提权漏洞）
2. 攻击者将恶意库重命名为合法插件名称
3. 等待 `PluginManager` 重新加载插件（例如系统重启）
4. 恶意插件被加载，获得代码执行权限
5. 所有后续推理请求由恶意插件处理

**代码位置**：
- 加载点：`services/server/plugin_manager/source/plugin.cpp:80-108`
- 路径解析：`services/server/plugin_manager/source/plugin_label.cpp:61-73`

**缓解措施**：
- 🔴 **缺失**：添加插件签名验证
- 🔴 **缺失**：加密插件二进制或哈希绑定
- 🔴 **缺失**：使用只读文件系统存储插件（如 dm-verity）

---

### 场景 3：反序列化类型混淆

**攻击步骤**:
1. 恶意客户端构造 `AlgorithmInfo`，设置错误的 `isAsync` 标志
2. 服务端根据标志选择 `SyncMsgHandler` 或 `AsyncMsgHandler`
3. 如果插件仅实现了同步接口，但被强制以异步模式调用
4. 导致未定义行为或崩溃

**代码位置**：
- Handler 选择：`services/server/server_executor/source/engine.cpp:79-83`
- 接口定义：`services/server/plugin/i_plugin.h:47-65`

**缓解措施**：
- ✅ 当前已实现：`IsSyncMode()` 在 `engine.cpp:45-52`
- ⚠️ 建议：添加插件模式运行时验证

---

### 场景 4：整数溢出导致内存破坏

**攻击步骤**:
1. 恶意客户端发送超大的 `DataInfo.length` 值
2. 反序列化时，`pos_ += sizeof(T)` 溢出
3. `Ensure()` 检查因溢出而失效
4. 解析器读取超出缓冲区范围的数据
5. 导致堆/栈溢出，可能获得代码执行

**代码位置**：`services/common/utils/encdec/include/data_decoder.h:114`

**缓解措施**：
- 🔴 **缺失**：在 `pos_ += sizeof(T)` 前检查溢出
- ✅ 已有：`Ensure()` 边界检查（但依赖正确的 `size_`）
- 建议：使用 `__builtin_add_overflow()` 检测溢出

---

## 6. 安全最佳实践建议

### 6.1 输入验证

| 原则 | 当前状态 | 建议 |
|------|---------|------|
| **白名单** | ❌ 缺失 | 算法类型、客户端 UID 使用白名单 |
| **长度检查** | ⚠️ 部分 | 添加所有外部输入的长度上限 |
| **类型验证** | ⚠️ 部分 | 验证枚举值的有效性 |
| **NULL 检查** | ✅ 良好 | 已在多处实现 |

### 6.2 内存安全

| 原则 | 当前状态 | 建议 |
|------|---------|------|
| **RAII 守卫** | ✅ 良好 | `MallocPointerGuard`, `PointerGuard` 已实现 |
| **边界检查** | ⚠️ 部分 | `Ensure()` 存在但依赖正确 `size_` |
| **溢出检测** | ❌ 缺失 | 算术操作未检查溢出 |
| **共享内存保护** | ⚠️ 部分 | 有大小限制但不够严格 |

### 6.3 权限控制

| 原则 | 当前状态 | 建议 |
|------|---------|------|
| **最小权限** | ⚠️ 部分 | 共享内存权限可优化 |
| **身份验证** | ❌ 缺失 | 插件无签名，客户端无 UID 验证 |
| **访问控制** | ❌ 缺失 | 缺少速率限制 |
| **审计日志** | ❌ 未知 | 安全事件未记录 |

### 6.4 加密保护

| 原则 | 当前状态 | 建议 |
|------|---------|------|
| **签名验证** | ❌ 缺失 | 插件二进制未签名 |
| **加密传输** | ❌ 缺失 | IPC 数据未加密（依赖 Samgr） |
| **完整性保护** | ❌ 缺失 | 模型文件未加密/签名 |

---

## 7. 相关章节

- [07_安全风险评估](07_SecurityReview.md) — 深度风险分析和利用路径
- [02_架构与数据流](02_Architecture.md) — 理解信任边界
- [06_攻击面分析](06_AttackSurface.md) — 本章节（攻击面识别）
- [10_内部实现细节](10_Internals.md) — 查看具体实现

---

## 证据要求

本章节所有安全分析均基于实际代码审查：

- ✅ 文件路径：完整的绝对路径
- ✅ 行号范围：关键代码所在的行号
- ✅ 风险等级：标注风险等级（高/中/低）
- ✅ 利用场景：具体的攻击步骤
- ✅ 修复建议：可操作的代码级建议

**未确认项标注**: [需确认] 表示需要进一步验证

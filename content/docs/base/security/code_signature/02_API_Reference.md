# Inner API 参考

本文档列出 `code_signature` 组件对外暴露的所有 Inner API。

## 目录

- [CodeSignUtils](#codesignutils)
- [CodeSignAttrUtils](#codesignattrutils)
- [LocalCodeSignKit](#localcodesignkit)
- [JitCodeSigner](#jitcodesigner)
- [错误码](#错误码)

---

## CodeSignUtils

**头文件**：`interfaces/inner_api/code_sign_utils/include/code_sign_utils.h`

**命名空间**：`OHOS::Security::CodeSign`

### API 清单

| API | 描述 | 返回值 |
|-----|------|--------|
| `EnforceCodeSignForApp(const EntryMap &entryPath, const std::string &signatureFile)` | 对 HAP 强制执行代码签名 | `int32_t` |
| `EnforceCodeSignForApp(const std::string &path, const EntryMap &entryPathMap, FileType type, uint32_t flag = 0)` | 对 HAP 强制执行代码签名（带原生文件） | `int32_t` |
| `EnforceCodeSignForAppWithOwnerId(...)` | 带 Owner ID 的代码签名 | `int32_t` |
| `EnforceCodeSignForAppWithPluginId(...)` | 带 Plugin ID 的代码签名 | `int32_t` |
| `EnforceCodeSignForFile(const std::string &path, const ByteBuffer &signature)` | 对文件强制执行代码签名 | `int32_t` |
| `ParseOwnerIdFromSignature(const ByteBuffer &sigbuffer, std::string &ownerID)` | 从签名文件解析 Owner ID | `int32_t` |
| `EnableKeyInProfile(const std::string &bundleName, const ByteBuffer &profileBuffer)` | 在 Profile 中启用密钥 | `int32_t` |
| `RemoveKeyInProfile(const std::string &bundleName)` | 从 Profile 中移除密钥 | `int32_t` |
| `EnableKeyForEnterpriseResign(const ByteBuffer &certBuffer)` | 为企业重签名启用证书 | `int32_t` |
| `RemoveKeyForEnterpriseResign(const ByteBuffer &certBuffer)` | 为企业重签名移除证书 | `int32_t` |
| `IsSupportOHCodeSign()` | 是否支持 OH SDK 代码签名 | `bool` |
| `InPermissiveMode()` | 是否处于宽容模式 | `bool` |
| `IsSupportFsVerity(const std::string &path)` | 检查路径是否支持 FsVerity | `int32_t` |

### FileType 枚举

```cpp
typedef enum {
    FILE_ALL,           // 启用 HAP 和 SO（新旧记录）
    FILE_SELF,         // 仅启用 HAP
    FILE_ENTRY_ONLY,   // 仅启用 SO（新旧记录）
    FILE_ENTRY_ADD,    // 仅记录，不启用
    FILE_TYPE_MAX,
} FileType;
```

### CodeSignInfoFlag 枚举

```cpp
enum CodeSignInfoFlag {
    IS_UNCOMPRESSED_NATIVE_LIBS = 0x01 << 0,
};
```

---

## CodeSignAttrUtils

**头文件**：`interfaces/inner_api/code_sign_attr_utils/include/code_sign_attr_utils.h`

### C API 清单

| API | 描述 |
|-----|------|
| `InitXpm(int enableJitFort, uint32_t idType, const char *ownerId, const char *apiTargetVersionStr, const char *appSignType)` | 初始化 XPM 资源 |
| `SetXpmOwnerId(uint32_t idType, const char *ownerId)` | 设置 Owner ID |

### 常量定义

```cpp
#define MAX_OWNERID_LEN 64

#define OWNERID_SYSTEM_TAG "SYSTEM_LIB_ID"
#define OWNERID_DEBUG_TAG  "DEBUG_LIB_ID"
#define OWNERID_SHARED_TAG "SHARED_LIB_ID"
#define OWNERID_COMPAT_TAG "COMPAT_LIB_ID"
```

### FileOwneridType / ProcessOwneridType 枚举

```cpp
enum FileOwneridType {
    FILE_OWNERID_UNINT = 0,
    FILE_OWNERID_SYSTEM,          // 1
    FILE_OWNERID_APP,             // 2
    FILE_OWNERID_DEBUG,           // 3
    FILE_OWNERID_SHARED,          // 4
    FILE_OWNERID_COMPAT,          // 5
    FILE_OWNERID_EXTEND,          // 6
    FILE_OWNERID_DEBUG_PLATFORM,  // 7
    FILE_OWNERID_PLATFORM,        // 8
    FILE_OWNERID_NWEB,            // 9
    FILE_OWNERID_APP_TEMP_ALLOW,  // 10
    FILE_OWNERID_ENT_RESIGN,      // 11
    FILE_OWNERID_MAX
};
```

### XpmConfig 结构体

```cpp
struct XpmConfig {
    uint64_t regionAddr;
    uint64_t regionLength;
    uint32_t idType;
    char ownerId[MAX_OWNERID_LEN];
    uint32_t apiTargetVersion;
};
```

---

## LocalCodeSignKit

**头文件**：`interfaces/inner_api/local_code_sign/include/local_code_sign_kit.h`

**命名空间**：`OHOS::Security::CodeSign`

### API 清单

| API | 描述 | 返回值 |
|-----|------|--------|
| `InitLocalCertificate(ByteBuffer &cert)` | 初始化本地证书 | `int32_t` |
| `SignLocalCode(const std::string &filePath, ByteBuffer &signature)` | 对本地代码签名 | `int32_t` |
| `SignLocalCode(const std::string &ownerID, const std::string &filePath, ByteBuffer &signature)` | 带 Owner ID 的本地代码签名 | `int32_t` |

### 服务信息

- **SA ID**: 3507
- **服务名**: local_code_sign
- **实现类**: `LocalCodeSignService`
- **Stub 类**: `LocalCodeSignStub`
- **Proxy 类**: `LocalCodeSignProxy`

---

## JitCodeSigner

**头文件**：`interfaces/inner_api/jit_code_sign/include/jit_code_signer.h`

**命名空间**：`OHOS::Security::CodeSign`

### API 清单

| API | 描述 |
|-----|------|
| `JitCodeSigner()` | 构造函数 |
| `Reset()` | 重置签名器 |
| `SignInstruction(Instr insn)` | 签名指令 |
| `SkipNext(uint32_t n)` | 跳过下一个指令 |
| `PatchInstruction(int offset, Instr insn)` | 修补指令 |
| `ValidateCodeCopy(Instr *jitMemory, Byte *jitBuffer, int size)` | 验证代码复制 |
| `RegisterTmpBuffer(Byte *tmpBuffer)` | 注册临时缓冲区 |
| `SignData(const Byte *data, uint32_t size)` | 签名数据 |
| `PatchInstruction(Byte *jitBuffer, Instr insn)` | 修补 JIT 缓冲区指令 |
| `PatchData(int offset, const Byte *const data, uint32_t size)` | 修补数据 |
| `PatchData(Byte *buffer, const Byte *const data, uint32_t size)` | 修补缓冲区数据 |
| `FlushLog()` | 刷新日志 |

### 常量定义

```cpp
constexpr int32_t INSTRUCTION_SIZE = 4;
constexpr int32_t LOG_2_INSTRUCTION_SIZE = 2;
constexpr size_t MAX_DEFERRED_LOG_LENGTH = 150;
```

---

## 公共类型

### ByteBuffer

**头文件**：`interfaces/inner_api/common/include/byte_buffer.h`

```cpp
class ByteBuffer {
public:
    ByteBuffer();
    ByteBuffer(uint32_t bufferSize);
    ByteBuffer(const ByteBuffer &other);
    ~ByteBuffer();

    bool CopyFrom(const uint8_t *srcData, uint32_t srcSize);
    bool PutData(uint32_t pos, const uint8_t *srcData, uint32_t srcSize);
    bool Resize(uint32_t newSize);
    uint8_t *GetBuffer() const;
    uint32_t GetSize() const;
    bool Empty() const;
};
```

### EntryMap

```cpp
using EntryMap = std::unordered_map<std::string, std::string>;
```

---

## 错误码

**头文件**：`interfaces/inner_api/common/include/errcode.h`

### 通用错误码

| 错误码 | 描述 |
|--------|------|
| `CS_SUCCESS = 0` | 成功 |
| `CS_ERR_MEMORY = -0x1` | 内存错误 |
| `CS_ERR_NO_PERMISSION = -0x2` | 无权限 |
| `CS_ERR_NO_SIGNATURE = -0x3` | 无签名 |
| `CS_ERR_INVALID_SIGNATURE = -0x4` | 无效签名 |

### 文件操作错误码

| 错误码 | 描述 |
|--------|------|
| `CS_ERR_FILE_INVALID = -0x100` | 文件无效 |
| `CS_ERR_FILE_PATH = -0x101` | 文件路径错误 |
| `CS_ERR_FILE_OPEN = -0x102` | 文件打开失败 |
| `CS_ERR_FILE_READ = -0x103` | 文件读取失败 |
| `CS_ERR_EXTRACT_FILES = -0x104` | 文件提取失败 |

### 签名错误码

| 错误码 | 描述 |
|--------|------|
| `CS_ERR_PARAM_INVALID = -0x200` | 参数无效 |
| `CS_ERR_HUKS_OBTAIN_CERT = -0x201` | HUKS 获取证书失败 |
| `CS_ERR_HUKS_SIGN = -0x202` | HUKS 签名失败 |
| `CS_ERR_HUKS_INIT_KEY = -0x203` | HUKS 初始化密钥失败 |
| `CS_ERR_NO_OWNER_ID = -0x205` | 无 Owner ID |
| `CS_ERR_INIT_LOCAL_CERT = -0x206` | 初始化本地证书失败 |
| `CS_ERR_VERIFY_CERT = -0x207` | 验证证书失败 |
| `CS_ERR_NO_PLUGIN_ID = -0x208` | 无 Plugin ID |

### 验证错误码

| 错误码 | 描述 |
|--------|------|
| `CS_ERR_ENABLE = -0x300` | 启用失败 |
| `CS_ERR_FSVREITY_NOT_SUPPORTED = -0x301` | FsVerity 不支持 |
| `CS_ERR_FSVERITY_NOT_ENABLED = -0x302` | FsVerity 未启用 |
| `CS_ERR_INVALID_OWNER_ID = -0x303` | 无效 Owner ID |
| `CS_CODE_SIGN_NOT_EXISTS = -0x304` | 代码签名不存在 |
| `CS_ERR_PROFILE = -0x305` | Profile 错误 |
| `CS_ERR_ENABLE_TIMEOUT = -0x306` | 启用超时 |
| `CS_ERR_INVALID_PLUGIN_ID = -0x308` | 无效 Plugin ID |
| `CS_ERR_NOT_ENTERPRISE_DEVICE = -0x309` | 非企业设备 |
| `CS_ERR_INVALID_CERT = -0x310` | 无效证书 |
| `CS_ERR_IOCTL_ERROR = -0x311` | IOCTL 错误 |
| `CS_ERR_VERIFY_ERROR = -0x312` | 验证错误 |

### IPC 错误码

| 错误码 | 描述 |
|--------|------|
| `CS_ERR_IPC_MSG_INVALID = -0x500` | IPC 消息无效 |
| `CS_ERR_IPC_WRITE_DATA = -0x501` | IPC 写数据失败 |
| `CS_ERR_IPC_READ_DATA = -0x502` | IPC 读数据失败 |
| `CS_ERR_REMOTE_CONNECTION = -0x503` | 远程连接失败 |
| `CS_ERR_SA_GET_SAMGR = -0x504` | 获取 SAMGR 失败 |
| `CS_ERR_SA_GET_PROXY = -0x505` | 获取 SA Proxy 失败 |
| `CS_ERR_SA_LOAD_FAILED = -0x506` | SA 加载失败 |
| `CS_ERR_SA_LOAD_TIMEOUT = -0x507` | SA 加载超时 |

### JIT 签名错误码

| 错误码 | 描述 |
|--------|------|
| `CS_ERR_NO_SIGNER = -0x700` | 无签名器 |
| `CS_ERR_PATCH_INVALID = -0x701` | 修补无效 |
| `CS_ERR_JIT_SIGN_SIZE = -0x702` | JIT 签名大小错误 |
| `CS_ERR_TMP_BUFFER = -0x703` | 临时缓冲区错误 |
| `CS_ERR_VALIDATE_CODE = -0x704` | 验证代码失败 |
| `CS_ERR_JITFORT_IN = -0x705` | JITFORT 输入错误 |
| `CS_ERR_JITFORT_OUT = -0x706` | JITFORT 输出错误 |
| `CS_ERR_SIGN_OFFSET = -0x707` | 签名偏移错误 |
| `CS_ERR_INVALID_DATA = -0x708` | 无效数据 |
| `CS_ERR_JIT_MEMORY = -0x709` | JIT 内存错误 |
| `CS_ERR_OOM = -0x710` | 内存不足 |
| `CS_ERR_LOG_TOO_LONG = -0x711` | 日志过长 |
| `CS_ERR_UNSUPPORT = -0x7ff` | 不支持 |

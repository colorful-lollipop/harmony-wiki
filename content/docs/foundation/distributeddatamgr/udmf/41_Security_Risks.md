# 安全风险清单

## 风险概述

本章节详细列出 UDMF 项目在源码分析中识别的安全风险，每条风险包含：风险描述、代码证据、可利用路径、影响范围和修复建议。风险按严重程度分为高、中、低三个等级。

分析范围覆盖 N-API 接口层、NDK 接口层、InnerKit 接口层、服务层 IPC 通信和数据持久化层。未发现的风险领域在相应章节说明。

## 高风险项

### R1：参数边界检查不足

**风险描述**：多个 API 接口缺少充分的参数边界检查，可能导致整数溢出、缓冲区溢出或拒绝服务攻击。

**代码证据**：
```cpp
// interfaces/ndk/data/udmf.h:454
// count 参数未检查最大值
char** OH_UdmfData_GetTypes(OH_UdmfData* pThis, unsigned int* count);

// interfaces/ndk/data/uds.h:826
// data 和 len 参数未关联检查
int OH_UdsArrayBuffer_SetData(OH_UdsArrayBuffer* buffer, 
                             unsigned char* data, unsigned int len);
```

**可利用路径**：
```
攻击者 → 调用 API → 传入超大 count 值
                     ↓
               内存分配失败
                     ↓
               拒绝服务（DoS）
```

**影响范围**：所有使用 NDK API 的 Native 应用

**修复建议**：
```cpp
// 添加参数边界检查
char** OH_UdmfData_GetTypes(OH_UdmfData* pThis, unsigned int* count)
{
    if (pThis == nullptr || count == nullptr) {
        return nullptr;
    }
    // 限制最大返回数量
    constexpr unsigned int MAX_COUNT = 1000;
    unsigned int actualCount = 0;
    
    // ... 获取实际数量 ...
    
    if (actualCount > MAX_COUNT) {
        actualCount = MAX_COUNT;
    }
    *count = actualCount;
    
    // 分配检查
    char** result = (char**)malloc(actualCount * sizeof(char*));
    if (result == nullptr) {
        return nullptr;
    }
    
    return result;
}
```

### R2：字符串编码验证缺失

**风险描述**：API 接受字符串参数但未验证字符编码，可能导致编码混淆攻击或特殊字符处理问题。

**代码证据**：
```cpp
// interfaces/ndk/data/udmf.h:290
// type 参数直接使用，未验证编码
bool OH_UdmfData_HasType(OH_UdmfData* pThis, const char* type);

// interfaces/ndk/data/udmf.h:746
// tag 参数未验证
const char* OH_UdmfProperty_GetTag(OH_UdmfProperty* pThis);
```

**可利用路径**：
```
攻击者 → 传入特殊编码字符串
                     ↓
               服务端解码失败
                     ↓
               逻辑错误或信息泄露
```

**影响范围**：跨语言数据交互场景

**修复建议**：
```cpp
#include <string_view>

// 使用安全的字符串处理
bool OH_UdmfData_HasType(OH_UdmfData* pThis, const char* type)
{
    if (pThis == nullptr || type == nullptr) {
        return false;
    }
    
    // UTF-8 验证
    if (!IsValidUTF8(type)) {
        return false;  // 或返回错误码
    }
    
    // 长度限制
    constexpr size_t MAX_TYPE_LEN = 256;
    if (strlen(type) > MAX_TYPE_LEN) {
        return false;
    }
    
    return pThis->HasType(type);
}
```

### R3：路径遍历防护不足

**风险描述**：文件 URI 处理接口缺少路径遍历防护，可能导致任意文件访问。

**代码证据**：
```cpp
// interfaces/ndk/data/uds.h:686
// fileUri 参数未验证路径
int OH_UdsFileUri_SetFileUri(OH_UdsFileUri* pThis, const char* fileUri);
```

**可利用路径**：
```
攻击者 → 设置恶意 fileUri
         "file:///etc/passwd"
                     ↓
               服务处理 URI
                     ↓
               访问敏感文件
```

**影响范围**：文件共享功能

**修复建议**：
```cpp
int OH_UdsFileUri_SetFileUri(OH_UdsFileUri* pThis, const char* fileUri)
{
    if (pThis == nullptr || fileUri == nullptr) {
        return UDMF_E_INVALID_PARAM;
    }
    
    std::string uri(fileUri);
    
    // 解析 URI
    if (!uri.starts_with("file://")) {
        return UDMF_E_INVALID_PARAM;
    }
    
    std::string path = uri.substr(7);  // 移除 "file://"
    
    // 路径规范化
    std::string normalized = NormalizePath(path);
    
    // 检查是否在允许目录内
    std::string allowedDir = GetSandboxPath();
    if (!normalized.starts_with(allowedDir)) {
        return UDMF_E_NO_PERMISSION;
    }
    
    // 检查路径遍历
    if (ContainsPathTraversal(normalized)) {
        return UDMF_E_INVALID_PARAM;
    }
    
    pThis->SetFileUri(normalized);
    return UDMF_E_OK;
}
```

## 中风险项

### R4：回调函数验证缺失

**风险描述**：回调函数注册接口缺少调用者验证，可能导致回调劫持。

**代码证据**：
```cpp
// interfaces/ndk/data/udmf.h:368
// provider 未验证来源
int OH_UdmfRecord_SetProvider(OH_UdmfRecord* pThis, 
                              const char* const* types, 
                              unsigned int count,
                              OH_UdmfRecordProvider* provider);
```

**可利用路径**：
```
恶意应用 → 注册回调
                     ↓
               被其他应用触发
                     ↓
               敏感信息泄露
```

**影响范围**：跨应用数据共享场景

**修复建议**：
```cpp
int OH_UdmfRecord_SetProvider(OH_UdmfRecord* pThis,
                              const char* const* types,
                              unsigned int count,
                              OH_UdmfRecordProvider* provider)
{
    // 验证调用者权限
    std::string callerBundle = GetCallingBundleName();
    std::string dataOwner = GetDataOwner(pThis->GetDataId());
    
    if (callerBundle != dataOwner) {
        return UDMF_E_NO_PERMISSION;
    }
    
    // 验证回调函数签名
    if (!ValidateProviderSignature(provider)) {
        return UDMF_E_INVALID_PARAM;
    }
    
    // ... 继续处理 ...
}
```

### R5：竞态条件风险

**风险描述**：部分全局状态操作缺少同步机制，可能导致竞态条件。

**代码证据**：
```cpp
// framework/innerkitsimpl/client/udmf_client.cpp
// dataCache_ 虽然是 ConcurrentMap，但缺少事务性
ConcurrentMap<std::string, UnifiedData> dataCache_;
```

**可利用路径**：
```
攻击者 → 并发请求
         GetData(key1)
         GetData(key2)
                     ↓
               时间窗口
                     ↓
               缓存不一致
```

**影响范围**：高并发场景下的数据一致性

**修复建议**：增加更细粒度的锁或使用原子操作。

### R6：错误信息泄露

**风险描述**：API 错误返回可能泄露内部实现细节。

**代码证据**：
```cpp
// interfaces/ndk/data/udmf_err_code.h:68
// 错误码映射可能泄露路径信息
static const std::unordered_map<int32_t, std::string> ERROR_MAP {
    { Status::E_FS_ERROR, "E_FS_ERROR" },
    // ...
};
```

**可利用路径**：
```
攻击者 → 触发错误
                     ↓
               解析错误消息
                     ↓
               获取内部路径或配置信息
```

**影响范围**：调试信息暴露

**修复建议**：生产环境使用通用错误码，详细错误仅在调试日志中输出。

### R7：序列化安全

**风险描述**：TLV 序列化/反序列化可能存在安全漏洞。

**代码证据**：
```cpp
// framework/common/tlv_util.h
// 泛型序列化支持多种类型
template<typename T>
Status Writing(std::shared_ptr<TLVObject> tlvObj, const T &value);

// framework/common/tlv_object.h
// 缓冲区直接操作
std::vector<uint8_t> buffer_;
```

**可利用路径**：
```
攻击者 → 构造恶意数据
                     ↓
               TLV 反序列化
                     ↓
               缓冲区溢出或类型混淆
```

**影响范围**：IPC 通信和数据持久化

**修复建议**：
```cpp
// 增加序列化安全检查
class TLVObject {
public:
    template<typename T>
    Status Read(TLVTag expectedTag, T &value) {
        TLVTag actualTag;
        TLVLength length;
        
        // 读取头部
        if (!ReadHead(actualTag, length)) {
            return E_READ_ERROR;
        }
        
        // 验证标签
        if (actualTag != expectedTag) {
            return E_TYPE_MISMATCH;
        }
        
        // 验证长度
        if (length > MAX_TLV_LENGTH) {
            return E_LENGTH_OVERFLOW;
        }
        
        // 验证类型大小
        if (sizeof(T) < length) {
            return E_BUFFER_OVERFLOW;
        }
        
        // ... 安全读取 ...
    }
    
private:
    static constexpr TLVLength MAX_TLV_LENGTH = 64 * 1024;  // 64KB
};
```

## 低风险项

### R8：日志敏感信息

**风险描述**：调试日志可能输出敏感数据。

**代码证据**：
```cpp
// framework/common/logger.h
#define UDMF_LOG_INFO(...) \
    OHOS::Hilog::Info(...)
```

**可利用路径**：日志文件访问权限不当

**修复建议**：敏感数据脱敏后输出

### R9：资源耗尽

**风险描述**：未限制批量操作的数据量。

**代码证据**：
```cpp
// interfaces/ndk/data/udmf.h:41
// batchData 未限制数量
Status API_EXPORT GetBatchData(const QueryOption &query, 
                              std::vector<UnifiedData> &unifiedDataSet);
```

**可利用路径**：
```
攻击者 → 批量查询超大数量
                     ↓
               内存耗尽
                     ↓
               拒绝服务
```

**影响范围**：服务稳定性

**修复建议**：增加数量限制和分页支持

### R10：配置注入

**风险描述**：`uniform_data_types.json` 加载缺少完整性校验。

**可利用路径**：
```
攻击者 → 篡改配置文件
                     ↓
               服务加载恶意配置
                     ↓
               类型混淆或拒绝服务
```

**影响范围**：类型系统安全

**修复建议**：使用签名验证或 HMAC 校验配置文件完整性

## 检查范围声明

本次安全分析检查了以下范围：

**已检查**：
- N-API 接口层（`interfaces/jskits/`）
- NDK 接口层（`interfaces/ndk/`）
- InnerKit 接口层（`interfaces/innerkits/`）
- 服务层 IPC（`framework/innerkitsimpl/service/`）
- 客户端实现（`framework/innerkitsimpl/client/`）
- TLV 序列化（`framework/common/`）

**未检查**：
- 测试代码（test/、*_test.*）
- 第三方依赖的内部实现
- 运行时动态加载机制
- 设备间安全协议实现

## 风险汇总表

| ID | 风险名称 | 严重程度 | 可利用性 | 影响范围 |
|----|----------|----------|----------|----------|
| R1 | 参数边界检查不足 | 高 | 低 | 所有 API 调用者 |
| R2 | 字符串编码验证缺失 | 高 | 低 | 跨语言场景 |
| R3 | 路径遍历防护不足 | 高 | 低 | 文件共享功能 |
| R4 | 回调函数验证缺失 | 中 | 中 | 跨应用场景 |
| R5 | 竞态条件风险 | 中 | 低 | 高并发场景 |
| R6 | 错误信息泄露 | 中 | 低 | 调试场景 |
| R7 | 序列化安全 | 中 | 低 | IPC 通信 |
| R8 | 日志敏感信息 | 低 | 低 | 日志访问 |
| R9 | 资源耗尽 | 低 | 中 | 服务稳定性 |
| R10 | 配置注入 | 低 | 低 | 类型系统 |

## 修复优先级

| 优先级 | 风险 | 建议修复时间 |
|--------|------|--------------|
| P0 | R1, R2, R3 | 立即 |
| P1 | R4, R5, R6, R7 | 本版本 |
| P2 | R8, R9, R10 | 下个版本 |

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [10_NAPI_Reference.md](./10_NAPI_Reference.md)：N-API 接口
- [40_Security_Analysis.md](./40_Security_Analysis.md)：安全分析
- [31_Build_Artifacts.md](./31_Build_Artifacts.md)：编译产物

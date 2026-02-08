# 对外 API (Public API)

## 目的

说明 appverify 模块对外暴露的 C++ API，包括函数清单、参数、返回值、使用示例。

## 适用范围

- 目标读者：调用方开发者（如 BMS 开发者）
- 涵盖内容：API 清单、参数说明、返回码、使用示例
- **注意**：本模块无 JS/N-API 层，仅提供 Native C++ API

## 关键结论

1. **接口类型**：纯 Native C++ InnerKit，无 JS/N-API
2. **调用方式**：动态链接 `libhapverify.so`
3. **线程模型**：同步阻塞调用
4. **主要调用方**：Bundle Manager Service (BMS)

## API 清单表

### 核心验证 API

| 函数 | 头文件 | 功能 | 同步/异步 |
|------|--------|------|----------|
| `HapVerify()` | hap_verify.h:29 | 完整验证 HAP 包 | 同步 |
| `ParseHapProfile()` | hap_verify.h:31 | 解析 Provision 配置 | 同步 |
| `ParseHapSignatureInfo()` | hap_verify.h:33 | 解析签名信息 | 同步 |
| `ParseBundleNameAndAppIdentifier()` | hap_verify.h:34 | 提取包名和应用标识 | 同步 |

### Profile 验证 API

| 函数 | 头文件 | 功能 | 同步/异步 |
|------|--------|------|----------|
| `VerifyProfile()` | hap_verify.h:38 | 独立验证 Provision | 同步 |
| `VerifyProfileByP7bBlock()` | hap_verify.h:39 | 从 P7b 块验证 Provision | 同步 |

### 调试与开发 API

| 函数 | 头文件 | 功能 | 同步/异步 |
|------|--------|------|----------|
| `EnableDebugMode()` | hap_verify.h:27 | 启用调试模式（测试证书） | 同步 |
| `DisableDebugMode()` | hap_verify.h:28 | 禁用调试模式 | 同步 |
| `SetDevMode()` | hap_verify.h:36 | 设置开发模式 | 同步 |

### 工具 API

| 函数 | 头文件 | 功能 | 同步/异步 |
|------|--------|------|----------|
| `GenerateUuidByKey()` | hap_verify.h:37 | 生成 UUID | 同步 |

## 详细 API 说明

### 1. HapVerify

**声明**：
```cpp
DLL_EXPORT int32_t HapVerify(
    const std::string& filePath,
    HapVerifyResult& hapVerifyResult,
    bool readFile = false,
    const std::string& localCertDir = ""
);
```

**位置**：hap_verify.h:29，实现：hap_verify.cpp:91

**功能**：完整验证 HAP 包的签名、来源、完整性和配置

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | const std::string& | 是 | HAP 文件路径（.hap/.hsp/.hqf/.app） |
| hapVerifyResult | HapVerifyResult& | 是 | 输出参数，验证结果 |
| readFile | bool | 否 | 是否读取完整文件（默认 false） |
| localCertDir | const std::string& | 否 | 本地证书目录（测试用） |

**返回值**：

| 返回码 | 说明 |
|--------|------|
| VERIFY_SUCCESS (0) | 验证成功 |
| FILE_PATH_INVALID (-1) | 文件路径无效 |
| OPEN_FILE_ERROR (-2) | 打开文件失败 |
| SIGNATURE_NOT_FOUND (-3) | 未找到签名块 |
| VERIFY_APP_PKCS7_FAIL (-4) | PKCS7 解析或验证失败 |
| PROFILE_PARSE_FAIL (-5) | Profile 解析失败 |
| APP_SOURCE_NOT_TRUSTED (-6) | 应用不在可信源中 |
| VERIFY_INTEGRITY_FAIL (-8) | 完整性验证失败 |
| VERIFY_SIGNATURE_FAIL (-13) | 签名验证失败 |
| VERIFY_SOURCE_INIT_FAIL (-14) | 初始化失败 |
| DEVICE_UNAUTHORIZED (-15) | 设备未授权 |
| CERTIFICATE_EXPIRED (-16) | 证书过期 |
| VERIFY_ENTERPRISE_RESIGN_FAIL (-17) | 企业重签名验证失败 |

**HapVerifyResult 结构**：

```cpp
class HapVerifyResult {
public:
    int32_t GetVersion() const;              // 签名版本
    ProvisionInfo GetProvisionInfo() const; // Provision 配置
    std::vector<std::string> GetPublicKey() const;   // 公钥列表
    std::vector<std::string> GetSignature() const;   // 签名列表
    int32_t GetProperty(std::string& property) const; // 扩展属性
private:
    int32_t version;
    std::vector<std::string> publicKeys;
    std::vector<std::string> signatures;
    HapByteBuffer pkcs7SignBlock;
    HapByteBuffer pkcs7ProfileBlock;
    std::vector<OptionalBlock> optionalBlocks;
    ProvisionInfo provisionInfo;
};
```

**调用链**：

```mermaid
graph LR
    Client[HapVerify] --> Init[HapVerifyInit<br/>首次调用]
    Init --> V2[HapVerifyV2::Verify]
    V2 --> Path[CheckFilePath]
    Path --> Sign[FindHapSignature]
    Sign --> PKCS7[VerifyAppPkcs7]
    PKCS7 --> Source[IsTrustedSource]
    Source --> Profile[VerifyAppSourceAndParseProfile]
    Profile --> Integ[VerifyHapIntegrity]
    Integ --> Result[设置 HapVerifyResult]
    Result --> Return[返回验证码]
```

### 2. ParseHapProfile

**声明**：
```cpp
DLL_EXPORT int32_t ParseHapProfile(
    const std::string& filePath,
    HapVerifyResult& hapVerifyV1Result,
    bool readFile = false
);
```

**位置**：hap_verify.h:31，实现：hap_verify.cpp:101

**功能**：仅解析 HAP 包中的 Provision 配置，不进行签名验证

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | const std::string& | 是 | HAP 文件路径 |
| hapVerifyV1Result | HapVerifyResult& | 是 | 输出参数，Provision 信息 |
| readFile | bool | 否 | 是否读取完整文件 |

**返回值**：同 `HapVerify()`

**使用场景**：仅需读取应用配置信息，不关心签名

### 3. ParseHapSignatureInfo

**声明**：
```cpp
DLL_EXPORT int32_t ParseHapSignatureInfo(
    const std::string& filePath,
    SignatureInfo& hapSignInfo
);
```

**位置**：hap_verify.h:33，实现：hap_verify.cpp:107

**功能**：提取 HAP 包的签名信息（证书链、签名值等）

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | const std::string& | 是 | HAP 文件路径 |
| hapSignInfo | SignatureInfo& | 是 | 输出参数，签名信息 |

**SignatureInfo 结构**：

```cpp
struct SignatureInfo {
    HapByteBuffer hapSignatureBlock;  // 签名块数据
    HapByteBuffer profileBlock;       // Profile 块数据
    std::vector<HapByteBuffer> optionalBlocks;  // 可选块
};
```

**返回值**：同 `HapVerify()`

### 4. ParseBundleNameAndAppIdentifier

**声明**：
```cpp
extern "C" DLL_EXPORT int32_t ParseBundleNameAndAppIdentifier(
    const int32_t fileFd,
    std::string& bundleName,
    std::string& appIdentifier
);
```

**位置**：hap_verify.h:34，实现：hap_verify.cpp:113

**功能**：从 HAP 文件提取包名和应用标识

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fileFd | int32_t | 是 | HAP 文件描述符 |
| bundleName | std::string& | 是 | 输出参数，应用包名 |
| appIdentifier | std::string& | 是 | 输出参数，应用标识 |

**返回值**：同 `HapVerify()`

**特殊限制**：不支持 `INTERNALTESTING` 分发类型（hap_verify.cpp:133）

### 5. VerifyProfile

**声明**：
```cpp
DLL_EXPORT int32_t VerifyProfile(
    const std::string& filePath,
    ProvisionInfo& provisionInfo
);
```

**位置**：hap_verify.h:38，实现：hap_verify.cpp:147

**功能**：独立验证 Provision 文件（.p7b 格式）

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| filePath | const std::string& | 是 | Provision 文件路径（.p7b） |
| provisionInfo | ProvisionInfo& | 是 | 输出参数，解析的 Provision 信息 |

**返回值**：同 `HapVerify()`

### 6. VerifyProfileByP7bBlock

**声明**：
```cpp
DLL_EXPORT int32_t VerifyProfileByP7bBlock(
    const uint32_t p7bBlockLength,
    const unsigned char* p7bBlock,
    bool needParseProvision,
    ProvisionInfo& provisionInfo
);
```

**位置**：hap_verify.h:39，实现：hap_verify_v2.cpp:634

**功能**：从内存中的 P7b 块验证 Provision

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| p7bBlockLength | uint32_t | 是 | P7b 块长度 |
| p7bBlock | const unsigned char* | 是 | P7b 块数据 |
| needParseProvision | bool | 是 | 是否解析 Provision 内容 |
| provisionInfo | ProvisionInfo& | 是 | 输出参数，Provision 信息 |

**返回值**：同 `HapVerify()`

### 7. EnableDebugMode / DisableDebugMode

**声明**：
```cpp
DLL_EXPORT bool EnableDebugMode();
DLL_EXPORT void DisableDebugMode();
```

**位置**：hap_verify.h:27-28，实现：hap_verify.cpp:59, 73

**功能**：启用/禁用调试模式，允许使用测试证书

**效果**：
- 启用后：允许使用 `trusted_root_ca_test.json` 和 `trusted_apps_sources_test.json`
- 禁用后：仅使用正式证书

**返回值**：
- `EnableDebugMode()`：成功返回 true，失败返回 false

**线程安全**：使用互斥锁保护（hap_verify.cpp:34）

### 8. SetDevMode

**声明**：
```cpp
DLL_EXPORT void SetDevMode(DevMode devMode);
```

**位置**：hap_verify.h:36，实现：hap_verify.cpp:83

**功能**：设置开发模式（DEV/NON_DEV）

**参数**：

| 值 | 说明 |
|----|------|
| DevMode::DEFAULT | 默认模式 |
| DevMode::DEV | 开发模式 |
| DevMode::NON_DEV | 非开发模式 |

**影响**：控制设备类型检查行为

### 9. GenerateUuidByKey

**声明**：
```cpp
DLL_EXPORT std::string GenerateUuidByKey(const std::string& key);
```

**位置**：hap_verify.h:37，实现：hap_verify.cpp:142

**功能**：根据密钥生成 UUID（用于应用标识）

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| key | const std::string& | 是 | 密钥字符串 |

**返回值**：UUID 字符串

**实现**：StringHash::GenerateUuidByKey() - string_hash.cpp

## API 使用示例

### 示例 1：标准 HAP 验证

```cpp
#include "interfaces/hap_verify.h"
#include "interfaces/hap_verify_result.h"

using namespace OHOS::Security::Verify;

int32_t VerifyApp(const std::string& hapPath) {
    HapVerifyResult result;

    // 调用 HapVerify
    int32_t ret = HapVerify(hapPath, result);

    if (ret != HapVerifyResultCode::VERIFY_SUCCESS) {
        // 处理错误
        printf("Verify failed: %d\n", ret);
        return ret;
    }

    // 获取验证结果
    ProvisionInfo provisionInfo = result.GetProvisionInfo();
    printf("Bundle Name: %s\n", provisionInfo.bundleInfo.bundleName.c_str());
    printf("App Identifier: %s\n", provisionInfo.bundleInfo.appIdentifier.c_str());

    // 获取公钥
    std::vector<std::string> publicKeys = result.GetPublicKey();
    printf("Public Keys count: %zu\n", publicKeys.size());

    return VERIFY_SUCCESS;
}
```

### 示例 2：提取应用信息

```cpp
#include "interfaces/hap_verify.h"
#include "util/signature_info.h"

using namespace OHOS::Security::Verify;

void GetAppInfo(const std::string& hapPath) {
    SignatureInfo signInfo;

    int32_t ret = ParseHapSignatureInfo(hapPath, signInfo);
    if (ret == VERIFY_SUCCESS) {
        printf("Signature block size: %zu\n", signInfo.hapSignatureBlock.GetCapacity());
        printf("Profile block size: %zu\n", signInfo.profileBlock.GetCapacity());
    }
}
```

### 示例 3：调试模式

```cpp
// 启用调试模式（用于测试）
if (!EnableDebugMode()) {
    printf("Failed to enable debug mode\n");
    return -1;
}

// 验证测试应用
HapVerifyResult result;
int32_t ret = HapVerify(testHapPath, result);

// 验证完成后禁用调试模式
DisableDebugMode();
```

### 示例 4：从文件描述符验证

```cpp
#include <fcntl.h>
#include "interfaces/hap_verify.h"

using namespace OHOS::Security::Verify;

int32_t VerifyByFd(const std::string& hapPath) {
    int fd = open(hapPath.c_str(), O_RDONLY);
    if (fd < 0) {
        return OPEN_FILE_ERROR;
    }

    std::string bundleName, appIdentifier;
    int32_t ret = ParseBundleNameAndAppIdentifier(fd, bundleName, appIdentifier);

    close(fd);

    if (ret == VERIFY_SUCCESS) {
        printf("Bundle Name: %s\n", bundleName.c_str());
        printf("App Identifier: %s\n", appIdentifier.c_str());
    }

    return ret;
}
```

## 参数校验与错误码

### 输入校验

**HapVerify() 输入校验**：
- filePath 非空
- filePath 扩展名为 .hap/.hsp/.hqf/.app/.p7b
- 文件大小不超过 2GB

**错误码映射**：

| 校验失败 | 错误码 |
|----------|--------|
| 文件路径为空 | FILE_PATH_INVALID |
| 文件扩展名错误 | FILE_PATH_INVALID |
| 文件不存在 | OPEN_FILE_ERROR |
| 文件过大（>2GB） | FILE_SIZE_TOO_LARGE |
| 无读取权限 | OPEN_FILE_ERROR |

### 签名验证错误

| 阶段 | 失败原因 | 错误码 |
|------|----------|--------|
| 查找签名块 | 未找到 | SIGNATURE_NOT_FOUND |
| PKCS7 解析 | DER 解码失败 | VERIFY_APP_PKCS7_FAIL |
| 证书链验证 | 证书过期 | CERTIFICATE_EXPIRED |
| 证书链验证 | CRL 吊销 | VERIFY_APP_PKCS7_FAIL |
| 签名验证 | 签名不匹配 | VERIFY_SIGNATURE_FAIL |
| 可信源匹配 | 不在可信源中 | APP_SOURCE_NOT_TRUSTED |
| Profile 解析 | JSON 格式错误 | PROFILE_PARSE_FAIL |
| Profile 验证 | 设备未授权 | DEVICE_UNAUTHORIZED |
| 完整性验证 | 摘要不匹配 | VERIFY_INTEGRITY_FAIL |

### 权限与前置条件

**无显式权限要求**：
- 调用方无需特殊权限
- 通过文件系统权限控制 HAP 文件访问

**文件权限要求**：
- 读取权限：HAP 文件需要可读
- 执行权限：无需执行权限

## 性能特性

### 典型性能指标

| HAP 大小 | 验证耗时（估计） |
|----------|----------------|
| < 10MB | < 100ms |
| 10-50MB | 100-500ms |
| 50-200MB | 500ms-2s |
| > 200MB | 2s+ |

**影响因素**：
- 文件大小（完整性验证计算量）
- 证书链长度（证书验证）
- 并行计算支持（大文件）

### 内存占用

- 基础：约 1-2MB
- 大文件：签名块大小 + 完整性验证缓冲区（可达数十 MB）

## 线程安全

### 全局状态

```cpp
static std::mutex g_mtx;  // hap_verify.cpp:34
```

**保护范围**：
- 初始化状态（HapVerifyInit）
- 调试模式状态
- 根证书和可信源单例

### 多线程调用

- ✅ 支持多线程同时调用
- ⚠️ 全局初始化只执行一次（线程安全）
- ✅ 每次调用独立，无共享状态污染

## 相关跳转

- [架构详解](03_Architecture.md) - API 调用流程
- [内部 API](05_Internal_API.md) - 模块接口
- [常见问题](09_FAQ.md) - 排查指南

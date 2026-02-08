# 内部 API (Internal API)

## 目的

说明 appverify 模块内部各子模块之间的接口、依赖关系和稳定性。

## 适用范围

- 目标读者：模块开发者、代码审查人员
- 涵盖内容：模块接口、依赖方向、稳定级别、可替换点
- **注意**：本文档不描述对外 API（参见 04_Public_API.md）

## 关键结论

1. **依赖方向**：单向依赖，无循环依赖（interfaces → verify → provision/ticket → util/init → common）
2. **稳定级别**：common/util 稳定，init/provision/ticket 较稳定，verify 可变化
3. **可替换点**：证书验证、摘要算法、文件操作可扩展

## 模块接口总览

### Common 模块接口

#### HapByteBuffer

**头文件**：hap_byte_buffer.h

**接口**：
```cpp
class HapByteBuffer {
public:
    HapByteBuffer();
    explicit HapByteBuffer(uint32_t capacity);
    ~HapByteBuffer();

    bool PutByteArray(const unsigned char* data, uint32_t len);
    bool GetByteArray(uint32_t offset, unsigned char* data, uint32_t len) const;

    void Reset();
    uint32_t GetCapacity() const;
    uint32_t GetPosition() const;
    const unsigned char* GetBufferPtr() const;

private:
    unsigned char* buffer_;
    uint32_t capacity_;
    uint32_t position_;
};
```

**职责**：动态字节缓冲区，管理二进制数据

**稳定级别**：稳定（基础数据结构）

---

#### RandomAccessFile

**头文件**：random_access_file.h

**接口**：
```cpp
class RandomAccessFile {
public:
    RandomAccessFile();
    ~RandomAccessFile();

    bool Init(const std::string& filePath, bool readFile);
    void Close();

    int32_t ReadBuffer(uint32_t offset, unsigned char* buffer, uint32_t len);
    int32_t GetFileSize() const;

private:
    int32_t fd_;
    bool readFile_;
};
```

**职责**：支持随机访问的文件封装

**稳定级别**：稳定（文件操作基础）

---

### Init 模块接口

#### TrustedRootCa

**头文件**：trusted_root_ca.h

**接口**：
```cpp
class TrustedRootCa {
public:
    static TrustedRootCa& GetInstance();

    bool Init();
    bool EnableDebug();
    void DisableDebug();
    void SetDevMode(DevMode devMode);
    bool FindMatchedRoot(const std::string& rootId, std::string& rootCa) const;
    void Recovery();

private:
    TrustedRootCa();
    ~TrustedRootCa();
    TrustedRootCa(const TrustedRootCa&) = delete;
    TrustedRootCa& operator=(const TrustedRootCa&) = delete;
};
```

**职责**：单例管理器，加载和管理根证书列表

**依赖**：common, cJSON, hilog

**稳定级别**：较稳定（配置管理接口）

**证据**：trusted_root_ca.cpp

---

#### TrustedSourceManager

**头文件**：trusted_source_manager.h

**接口**：
```cpp
class TrustedSourceManager {
public:
    static TrustedSourceManager& GetInstance();

    bool Init();
    bool EnableDebug();
    void DisableDebug();

    MatchingStates IsTrustedSource(
        const std::string& subject,
        const std::string& issuer,
        const std::string& profileSubject,
        const std::string& profileIssuer
    ) const;
    void Recovery();

private:
    TrustedSourceManager();
    ~TrustedSourceManager();
    TrustedSourceManager(const TrustedSourceManager&) = delete;
    TrustedSourceManager& operator=(const TrustedSourceManager&) = delete;
};
```

**职责**：单例管理器，匹配应用签名证书与可信源

**依赖**：common, cJSON, hilog

**稳定级别**：较稳定（匹配逻辑稳定）

**证据**：trusted_source_manager.cpp

---

#### HapCrlManager

**头文件**：hap_crl_manager.h

**接口**：
```cpp
class HapCrlManager {
public:
    static HapCrlManager& GetInstance();

    bool Init();
    bool IsCertRevoked(const std::string& certId) const;

private:
    HapCrlManager();
    ~HapCrlManager();
    HapCrlManager(const HapCrlManager&) = delete;
    HapCrlManager& operator=(const HapCrlManager&) = delete;
};
```

**职责**：单例管理器，证书吊销列表（CRL）管理

**依赖**：common, hilog

**稳定级别**：稳定（CRL 查询逻辑）

**证据**：hap_crl_manager.cpp

---

#### DeviceTypeManager

**头文件**：device_type_manager.h

**接口**：
```cpp
class DeviceTypeManager {
public:
    static DeviceTypeManager& GetInstance();

    void GetDeviceTypeInfo();

private:
    DeviceTypeManager() = default;
    ~DeviceTypeManager() = default;
    DeviceTypeManager(const DeviceTypeManager&) = delete;
    DeviceTypeManager& operator=(const DeviceTypeManager&) = delete;
};
```

**职责**：获取设备类型和模式信息

**依赖**：common, (非标准系统) ipc, os_account

**稳定级别**：稳定

**证据**：device_type_manager.cpp

---

### Util 模块接口

#### HapVerifyOpensslUtils

**头文件**：hap_verify_openssl_utils.h

**接口**：
```cpp
class HapVerifyOpensslUtils {
public:
    static bool ParsePkcs7Package(
        const unsigned char packageData[],
        uint32_t packageLen,
        Pkcs7Context& pkcs7Context
    );

    static int32_t GetCertChains(PKCS7* p7, Pkcs7Context& pkcs7Context);
    static bool VerifyPkcs7(Pkcs7Context& pkcs7Context);
    static bool GetPublickeys(
        const CertChain& signCertChain,
        std::vector<std::string>& publicKeyVec
    );
    static bool GetSignatures(
        const CertChain& signCertChain,
        std::vector<std::string>& signatureVec
    );

    static bool DigestInit(const DigestParameter& digestParameter);
    static bool DigestUpdate(const DigestParameter& digestParameter, ...);
    static int32_t GetDigest(const DigestParameter& digestParameter, ...);

    // ... 更多辅助函数
};
```

**职责**：OpenSSL 操作封装（PKCS7、证书、签名、摘要）

**依赖**：common, openssl

**稳定级别**：稳定（工具类）

**证据**：hap_verify_openssl_utils.cpp

---

#### HapCertVerifyOpensslUtils

**头文件**：hap_cert_verify_openssl_utils.h

**接口**：
```cpp
class HapCertVerifyOpensslUtils {
public:
    static bool VerifyCertChainPeriodOfValidity(const CertChain& certsChain);
    static bool VerifyCrl(const std::string& certId, const Pkcs7Context& pkcs7Context);
    static bool GetFingerprintBase64FromPemCert(
        const std::string& pemCert,
        std::string& fingerprint
    );
    static int32_t GetCertsChain(
        const X509* signCert,
        CertChain& certsChain,
        const Pkcs7Context& pkcs7Context
    );
};
```

**职责**：证书链验证、CRL 检查、指纹计算

**依赖**：common, openssl

**稳定级别**：稳定（证书验证逻辑）

**证据**：hap_cert_verify_openssl_utils.cpp

---

#### HapSigningBlockUtils

**头文件**：hap_signing_block_utils.h

**接口**：
```cpp
class HapSigningBlockUtils {
public:
    static int32_t FindHapSignature(
        RandomAccessFile& hapFile,
        HapSignatureBlock& hapSignatureBlock
    );

    static int32_t VerifyHapIntegrity(
        RandomAccessFile& hapFile,
        const HapSignatureBlock& hapSignatureBlock
    );

    static bool GetDigestAndAlgorithm(
        const HapSignatureBlock& hapSignatureBlock,
        DigestParameter& digest
    );

    static bool HapVerifyParallelizationSupported();
};
```

**职责**：签名块查找、完整性验证、摘要计算

**依赖**：common, openssl

**稳定级别**：稳定（签名块格式稳定）

**证据**：hap_signing_block_utils.cpp

---

### Provision 模块接口

#### ProvisionVerify

**头文件**：provision_verify.h

**接口**：
```cpp
class ProvisionVerify {
public:
    ProvisionVerify() = default;
    ~ProvisionVerify() = default;

    AppProvisionVerifyResult ParseAndVerify(
        const std::string& profile,
        const Pkcs7Context& profileContext,
        ProvisionInfo& provisionInfo,
        bool profileNeedWriteCrl
    );

    bool ParseProvision(
        const std::string& profile,
        ProvisionInfo& provisionInfo
    );

    bool CheckDeviceID(const ProvisionInfo& provisionInfo);
    static bool SetRdDevice(const bool isRdDevice);
};
```

**职责**：Provision 解析与验证

**依赖**：util, init, common

**稳定级别**：较稳定（Provision 格式稳定）

**证据**：provision_verify.cpp

---

### Ticket 模块接口

#### TicketVerify

**头文件**：ticket_verify.h

**接口**：
```cpp
class TicketVerify {
public:
    static bool Verify(
        const std::string& ticket,
        const ProvisionInfo& provisionInfo
    );

private:
    static bool CheckPermissions(
        const ProvisionInfo& provisionInfo,
        const ProvisionInfo& ticketInfo
    );
    static bool CheckDevice(
        const ProvisionInfo& provisionInfo,
        const ProvisionInfo& ticketInfo
    );
    static bool CompareTicketAndProfile(
        const ProvisionInfo& provisionInfo,
        const ProvisionInfo& ticketInfo
    );
};
```

**职责**：Ticket 验证（OpenTest 应用授权）

**依赖**：util, init, common, (非标准系统) os_account

**稳定级别**：较稳定（Ticket 格式稳定）

**证据**：ticket_verify.cpp

---

### Verify 模块接口

#### HapVerifyV2

**头文件**：hap_verify_v2.h

**接口**：
```cpp
class HapVerifyV2 {
public:
    HapVerifyV2();
    ~HapVerifyV2();

    int32_t Verify(
        const std::string& filePath,
        HapVerifyResult& hapVerifyV1Result,
        bool readFile = false,
        const std::string& localCertDir = ""
    );

    int32_t Verify(
        const int32_t fileFd,
        HapVerifyResult& hapVerifyV1Result
    );

    int32_t ParseHapProfile(
        const std::string& filePath,
        HapVerifyResult& hapVerifyV1Result,
        bool readFile = false
    );

    int32_t ParseHapSignatureInfo(
        const std::string& filePath,
        SignatureInfo& hapSignInfo
    );

    int32_t VerifyProfile(
        const std::string& filePath,
        ProvisionInfo& provisionInfo
    );

    int32_t VerifyProfileByP7bBlock(
        const uint32_t p7bBlockLength,
        const unsigned char* p7bBlock,
        bool needParseProvision,
        ProvisionInfo& provisionInfo
    );

private:
    int32_t Verify(
        RandomAccessFile& hapFile,
        const std::string& localCertDir,
        HapVerifyResult& hapVerifyV1Result
    );

    int32_t VerifyAppPkcs7(
        Pkcs7Context& pkcs7Context,
        const HapByteBuffer& hapSignatureBlock
    );

    int32_t VerifyAppSourceAndParseProfile(
        Pkcs7Context& pkcs7Context,
        const HapByteBuffer& hapProfileBlock,
        HapVerifyResult& hapVerifyV1Result,
        bool& profileNeedWriteCrl
    );

    bool VerifyProfileSignature(
        const Pkcs7Context& pkcs7Context,
        Pkcs7Context& profileContext
    );

    int32_t VerifyProfileInfo(
        const Pkcs7Context& pkcs7Context,
        const Pkcs7Context& profileContext,
        HapVerifyResult& hapVerifyV1Result
    );

    bool IsAppDistributedTypeAllowInstall(
        const AppDistType& type,
        const ProvisionInfo& provisionInfo
    ) const;

    bool GenerateAppId(ProvisionInfo& provisionInfo);
    bool GenerateFingerprint(ProvisionInfo& provisionInfo);
    void SetProfileBlockData(
        const Pkcs7Context& pkcs7Context,
        const HapByteBuffer& hapProfileBlock,
        HapVerifyResult& hapVerifyV1Result
    );
};
```

**职责**：HAP 验证核心流程编排

**依赖**：util, init, provision, ticket, common

**稳定级别**：较稳定（核心验证逻辑）

**证据**：hap_verify_v2.cpp

---

#### EnterpriseResignMgr

**头文件**：enterprise_resign_mgr.h

**接口**：
```cpp
class EnterpriseResignMgr {
public:
    static bool Verify(
        const AppDistType& type,
        const ProvisionInfo& provisionInfo,
        const CertChain& localCertsChain,
        const CertChain& hapCertsChain
    );

private:
    static bool IsEnterpriseCert(const CertChain& certChain);
    static bool IsEnterpriseDevice();
    static bool VerifyLocalChainAndHapChain(
        const CertChain& localCertsChain,
        const CertChain& hapCertsChain
    );
};
```

**职责**：企业重签名应用验证

**依赖**：util, common

**稳定级别**：较稳定（企业验证逻辑）

**证据**：enterprise_resign_mgr.cpp

---

## 依赖关系矩阵

| 模块 | common | init | util | provision | ticket | verify |
|--------|--------|------|------|-----------|---------|---------|
| **common** | - | ✗ | ✗ | ✗ | ✗ |
| **init** | ✅ | - | ✗ | ✗ | ✗ |
| **util** | ✅ | ✗ | - | ✗ | ✗ |
| **provision** | ✅ | ✅ | ✅ | - | ✗ |
| **ticket** | ✅ | ✅ | ✅ | - | ✗ |
| **verify** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **interfaces** | ✅ | ✅ | ✅ | ✅ | ✅ |

✅ = 直接依赖，✗ = 无依赖

## 稳定性分析

### Stable 级别（稳定，极少变更）

**模块**：common, util

**理由**：
- 基础数据结构和工具类
- 影响范围广，变更需谨慎
- 接口设计简洁明确

**可替换点**：
- 摘要算法：DigestParameter 类
- 文件操作：RandomAccessFile 类
- 字节缓冲区：HapByteBuffer 类

---

### Fairly Stable 级别（较稳定，小幅变更）

**模块**：init, provision, ticket

**理由**：
- 配置和验证逻辑相对稳定
- 可能增加新的配置项或验证规则
- 接口向后兼容

**可替换点**：
- 证书源：TrustedRootCa（支持自定义证书存储）
- 可信源：TrustedSourceManager（支持扩展匹配规则）

---

### Volatile 级别（可变化，优化需求）

**模块**：verify

**理由**：
- 核心验证流程可能优化
- 新增验证场景（如新型签名算法）
- 性能优化需求

**可替换点**：
- 验证流程：HapVerifyV2（支持子类扩展）
- 企业验证：EnterpriseResignMgr（支持扩展验证逻辑）

## 可扩展点

### 1. 摘要算法扩展

**位置**：`DigestParameter` - digest_parameter.h

**扩展方式**：
```cpp
// 添加新的摘要算法
enum class DigestAlgorithm {
    SHA256 = 0,
    SHA384 = 1,
    SHA512 = 2,
    // 新增
    SHA3_256 = 3,
};
```

**修改点**：
- hap_verify_openssl_utils.cpp:DigestInit/Update/Get

---

### 2. 证书验证策略扩展

**位置**：`HapCertVerifyOpensslUtils` - hap_cert_verify_openssl_utils.h

**扩展方式**：
- 添加新的验证规则（如 OCSP）
- 自定义证书验证策略

---

### 3. 文件数据源扩展

**位置**：`HapByteBufferDataSource` / `HapFileDataSource`

**扩展方式**：
- 实现新的 DataSource（如网络流）
- 支持内存映射文件

---

## 线程安全分析

### 单例模式线程安全

**单例类**：
- TrustedRootCa
- TrustedSourceManager
- HapCrlManager
- DeviceTypeManager
- TrustedTicketManager

**实现方式**：
```cpp
static TrustedRootCa& GetInstance() {
    static TrustedRootCa instance;
    return instance;
}
```

**线程安全**：
- ✅ C++11 保证静态局部变量线程安全初始化
- ⚠️ 单例内部状态需自行保护

### 全局状态保护

**互斥锁**：
```cpp
static std::mutex g_mtx;  // hap_verify.cpp:34
```

**保护范围**：
- 初始化状态
- 调试模式切换
- 根证书/可信源加载

**无锁操作**：
- HapVerifyV2 实例方法
- OpenSSL 内部操作（OpenSSL 自身线程安全）

---

## 内部调用链

### HapVerify 内部调用

```
HapVerify (hap_verify.cpp:91)
  ├─ HapVerifyInit() [首次]
  │   ├─ TrustedRootCa::Init()
  │   ├─ TrustedSourceManager::Init()
  │   ├─ TrustedTicketManager::Init()
  │   ├─ HapCrlManager::Init()
  │   └─ DeviceTypeManager::GetDeviceTypeInfo()
  │
  └─ HapVerifyV2::Verify()
      ├─ CheckFilePath()
      ├─ FindHapSignature()
      ├─ VerifyAppPkcs7()
      │   ├─ ParsePkcs7Package()
      │   ├─ GetCertChains()
      │   │   ├─ GetCertsChain()
      │   │   ├─ VerifyCertChainPeriodOfValidity()
      │   │   └─ VerifyCrl()
      │   └─ VerifyPkcs7()
      ├─ VerifyAppSourceAndParseProfile()
      │   ├─ IsTrustedSource()
      │   ├─ ParseAndVerifyProfile()
      │   ├─ VerifyProfileSignature()
      │   ├─ VerifyProfileInfo()
      │   ├─ VerifyTicket() [APP_GALLERY]
      │   └─ EnterpriseResignMgr::Verify() [ENTERPRISE]
      ├─ GetDigestAndAlgorithm()
      ├─ GetPublickeys()
      ├─ GetSignatures()
      └─ VerifyHapIntegrity()
```

---

## 相关跳转

- [架构详解](03_Architecture.md) - 模块协作流程
- [对外 API](04_Public_API.md) - 公共接口
- [GN 构建目标](06_GN_Targets.md) - 构建配置

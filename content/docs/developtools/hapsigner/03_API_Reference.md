# API 参考

## 目的

本文档介绍 hapsigner 提供的 API 接口，包括命令行接口、Java API 和 C++ API。

## 适用范围

- 需要调用签名功能的开发者
- 需要集成签名工具的 CI/CD 系统

---

## 命令行接口

### Java 命令行工具

**入口**: `java -jar hap-sign-tool.jar`

#### 1. generate-keypair - 生成密钥对

**语法**:
```bash
java -jar hap-sign-tool.jar generate-keypair \
    -keyAlias "alias" \
    -keyAlg "ECC" \
    -keySize "NIST-P-256" \
    -keystoreFile "keystore.p12" \
    [-keyPwd "password"] \
    [-keystorePwd "password"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-keyAlias` | 是 | 密钥别名 |
| `-keyAlg` | 是 | 密钥算法：RSA/ECC |
| `-keySize` | 是 | RSA: 2048/3072/4096; ECC: NIST-P-256/NIST-P-384 |
| `-keystoreFile` | 是 | 密钥库文件路径 (.p12/.jks) |
| `-keyPwd` | 否 | 密钥密码 |
| `-keystorePwd` | 否 | 密钥库密码 |

**代码证据**: `hapsigntool/hap_sign_tool/src/main/java/com/ohos/hapsigntool/HapSignTool.java:304-317`

#### 2. generate-csr - 生成证书签名请求

**语法**:
```bash
java -jar hap-sign-tool.jar generate-csr \
    -keyAlias "alias" \
    -subject "C=CN,O=Huawei,OU=OpenHarmony,CN=App" \
    -signAlg "SHA256withECDSA" \
    -keystoreFile "keystore.p12" \
    [-outFile "request.csr"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-keyAlias` | 是 | 密钥别名 |
| `-subject` | 是 | 证书主题，X.500 格式 |
| `-signAlg` | 是 | 签名算法：SHA256withRSA/SHA384withRSA/SHA256withECDSA/SHA384withECDSA |
| `-keystoreFile` | 是 | 密钥库文件路径 |
| `-outFile` | 否 | 输出 CSR 文件路径 |

#### 3. generate-ca - 生成 CA 证书

**语法**:
```bash
java -jar hap-sign-tool.jar generate-ca \
    -keyAlias "caKey" \
    -keyAlg "ECC" \
    -keySize "NIST-P-256" \
    -subject "C=CN,O=Huawei,OU=OpenHarmony,CN=RootCA" \
    -signAlg "SHA256withECDSA" \
    -keystoreFile "ca.p12" \
    [-validity "3650"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-keyAlias` | 是 | 密钥别名 |
| `-keyAlg` | 是 | 密钥算法 |
| `-keySize` | 是 | 密钥长度 |
| `-subject` | 是 | 证书主题 |
| `-signAlg` | 是 | 签名算法 |
| `-keystoreFile` | 是 | 密钥库文件 |
| `-validity` | 否 | 有效期天数，默认 3650 |
| `-issuer` | 否 | 颁发者，为空表示根 CA |
| `-issuerKeyAlias` | 否 | 颁发者密钥别名 |

#### 4. generate-app-cert - 生成应用证书

**语法**:
```bash
java -jar hap-sign-tool.jar generate-app-cert \
    -keyAlias "appKey" \
    -issuer "C=CN,O=Huawei,CN=SubCA" \
    -issuerKeyAlias "subCaKey" \
    -subject "C=CN,O=Huawei,CN=App" \
    -signAlg "SHA256withECDSA" \
    -keystoreFile "app.p12" \
    -rootCaCertFile "root.cer" \
    -subCaCertFile "sub.cer" \
    [-outFile "app.cer"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-keyAlias` | 是 | 密钥别名 |
| `-issuer` | 是 | 颁发者主题 |
| `-issuerKeyAlias` | 是 | 颁发者密钥别名 |
| `-subject` | 是 | 证书主题 |
| `-signAlg` | 是 | 签名算法 |
| `-keystoreFile` | 是 | 密钥库文件 |
| `-rootCaCertFile` | 是 | 根 CA 证书（输出证书链时） |
| `-subCaCertFile` | 是 | 中间 CA 证书（输出证书链时） |
| `-outFile` | 否 | 输出文件路径 |
| `-outForm` | 否 | 输出格式：cert/certChain，默认 certChain |

#### 5. sign-profile - 签名 Profile

**语法**:
```bash
java -jar hap-sign-tool.jar sign-profile \
    -mode "localSign" \
    -keyAlias "profileKey" \
    -profileCertFile "profile.cer" \
    -inFile "profile.json" \
    -signAlg "SHA256withECDSA" \
    -keystoreFile "keystore.p12" \
    -outFile "profile.p7b"
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-mode` | 是 | 签名模式：localSign/remoteSign |
| `-keyAlias` | 是（localSign） | 密钥别名 |
| `-profileCertFile` | 是（localSign） | Profile 签名证书 |
| `-inFile` | 是 | 输入 Profile JSON 文件 |
| `-signAlg` | 是 | 签名算法 |
| `-keystoreFile` | 是（localSign） | 密钥库文件 |
| `-outFile` | 是 | 输出 p7b 文件 |
| `-keyPwd` | 否 | 密钥密码 |
| `-keystorePwd` | 否 | 密钥库密码 |

**代码证据**: `hapsigntool/hap_sign_tool/src/main/java/com/ohos/hapsigntool/HapSignTool.java:376-397`

#### 6. sign-app - 签名应用

**语法**:
```bash
java -jar hap-sign-tool.jar sign-app \
    -mode "localSign" \
    -keyAlias "appKey" \
    -appCertFile "app.cer" \
    -profileFile "profile.p7b" \
    -inFile "app.zip" \
    -signAlg "SHA256withECDSA" \
    -keystoreFile "keystore.p12" \
    -outFile "app.hap" \
    [-signCode "1"] \
    [-inForm "zip"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-mode` | 是 | 签名模式：localSign/remoteSign/remoteResign |
| `-keyAlias` | 是（localSign） | 密钥别名 |
| `-appCertFile` | 是 | 应用签名证书 |
| `-profileFile` | 是（HAP） | Profile 文件（p7b 格式） |
| `-inFile` | 是 | 输入文件 |
| `-signAlg` | 是 | 签名算法 |
| `-keystoreFile` | 是（localSign） | 密钥库文件 |
| `-outFile` | 是 | 输出文件 |
| `-signCode` | 否 | 启用代码签名：1/0，默认 1 |
| `-inForm` | 否 | 输入格式：zip/elf/bin，默认 zip |
| `-profileSigned` | 否 | Profile 是否已签名：1/0，默认 1 |

**代码证据**: `hapsigntool/hap_sign_tool/src/main/java/com/ohos/hapsigntool/HapSignTool.java:328-355`

#### 7. verify-profile - 验证 Profile

**语法**:
```bash
java -jar hap-sign-tool.jar verify-profile \
    -inFile "profile.p7b" \
    [-outFile "result.json"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-inFile` | 是 | 输入 p7b 文件 |
| `-outFile` | 否 | 输出验证结果 JSON |

#### 8. verify-app - 验证应用

**语法**:
```bash
java -jar hap-sign-tool.jar verify-app \
    -inFile "app.hap" \
    -outCertchain "certchain.cer" \
    -outProfile "profile.p7b" \
    [-inForm "zip"]
```

**参数说明**:

| 参数 | 必填 | 说明 |
|------|------|------|
| `-inFile` | 是 | 输入文件 |
| `-outCertchain` | 是 | 输出证书链文件 |
| `-outProfile` | 是 | 输出 Profile 文件 |
| `-inForm` | 否 | 输入格式：zip/elf/bin，默认 zip |

---

## Java Service API

### 接口定义

**代码位置**: `hapsigntool/hap_sign_tool_lib/src/main/java/com/ohos/hapsigntool/api/ServiceApi.java`

```java
public interface ServiceApi {
    // 密钥和证书生成
    boolean generateKeyStore(Options options);
    boolean generateCsr(Options options);
    boolean generateCert(Options options);
    boolean generateCA(Options options);
    boolean generateAppCert(Options options);
    boolean generateProfileCert(Options options);
    
    // 签名
    boolean signProfile(Options options);
    boolean signHap(Options options);
    
    // 验证
    boolean verifyProfile(Options options);
    boolean verifyHap(Options options);
}
```

### 使用示例

```java
import com.ohos.hapsigntool.api.ServiceApi;
import com.ohos.hapsigntool.api.SignToolServiceImpl;
import com.ohos.hapsigntool.entity.Options;

public class SignExample {
    public static void main(String[] args) {
        // 创建 API 实例
        ServiceApi api = new SignToolServiceImpl();
        
        // 构建参数
        Options options = new Options();
        options.put(Options.KEY_ALIAS, "myKey");
        options.put(Options.KEY_ALG, "ECC");
        options.put(Options.KEY_SIZE, "NIST-P-256");
        options.put(Options.KEY_STORE_FILE, "keystore.p12");
        options.put(Options.SUBJECT, "C=CN,O=Test,CN=App");
        options.put(Options.SIGN_ALG, "SHA256withECDSA");
        
        // 生成密钥对
        boolean success = api.generateKeyStore(options);
        
        if (success) {
            System.out.println("密钥对生成成功");
        } else {
            System.err.println("密钥对生成失败");
        }
    }
}
```

### 参数常量

**代码位置**: `hapsigntool/hap_sign_tool_lib/src/main/java/com/ohos/hapsigntool/entity/ParamConstants.java`

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `KEY_ALIAS` | "keyAlias" | 密钥别名 |
| `KEY_PWD` | "keyPwd" | 密钥密码 |
| `KEY_ALG` | "keyAlg" | 密钥算法 |
| `KEY_SIZE` | "keySize" | 密钥长度 |
| `KEY_STORE_FILE` | "keystoreFile" | 密钥库文件 |
| `KEY_STORE_PWD` | "keystorePwd" | 密钥库密码 |
| `SUBJECT` | "subject" | 证书主题 |
| `ISSUER` | "issuer" | 颁发者 |
| `SIGN_ALG` | "signAlg" | 签名算法 |
| `IN_FILE` | "inFile" | 输入文件 |
| `OUT_FILE` | "outFile" | 输出文件 |
| `MODE` | "mode" | 签名模式 |
| `PROFILE_FILE` | "profileFile" | Profile 文件 |
| `APP_CERT_FILE` | "appCertFile" | 应用证书文件 |
| `SIGN_CODE` | "signCode" | 启用代码签名 |

---

## C++ Service API

### hapsigntool_cpp 完整版

**代码位置**: `hapsigntool_cpp/api/include/service_api.h`

```cpp
namespace OHOS {
namespace SignatureTools {

class ServiceApi {
public:
    ServiceApi() = default;
    virtual ~ServiceApi() = default;

    virtual bool GenerateKeyStore(Options* params) = 0;
    virtual bool GenerateCsr(Options* params) = 0;
    virtual bool GenerateCert(Options* params) = 0;
    virtual bool GenerateCA(Options* params) = 0;
    virtual bool GenerateAppCert(Options* params) = 0;
    virtual bool GenerateProfileCert(Options* params) = 0;
    virtual bool SignProfile(Options* params) = 0;
    virtual bool VerifyProfile(Options* params) = 0;
    virtual bool SignHap(Options* params) = 0;
    virtual bool VerifyHapSigner(Options* params) = 0;
};

} // namespace SignatureTools
} // namespace OHOS
```

### binary_sign_tool 精简版

**代码位置**: `binary_sign_tool/api/include/service_api.h`

```cpp
namespace OHOS {
namespace SignatureTools {

class ServiceApi {
public:
    ServiceApi() = default;
    virtual ~ServiceApi() = default;

    virtual bool Sign(Options* params) = 0;
    virtual bool Verify(Options* option) = 0;
};

} // namespace SignatureTools
} // namespace OHOS
```

### 使用示例

```cpp
#include "service_api.h"
#include "sign_tool_service_impl.h"
#include "options.h"

using namespace OHOS::SignatureTools;

int main() {
    // 创建服务实例
    std::unique_ptr<ServiceApi> api = std::make_unique<SignToolServiceImpl>();
    
    // 创建参数对象
    std::unique_ptr<Options> options = std::make_unique<Options>();
    options->Put(Options::KEY_ALIAS, "myKey");
    options->Put(Options::KEY_ALG, "ECC");
    options->Put(Options::KEY_SIZE, "NIST-P-256");
    options->Put(Options::KEY_STORE_FILE, "keystore.p12");
    options->Put(Options::SUBJECT, "C=CN,O=Test,CN=App");
    options->Put(Options::SIGN_ALG, "SHA256withECDSA");
    
    // 生成密钥对
    bool success = api->GenerateKeyStore(options.get());
    
    if (success) {
        std::cout << "密钥对生成成功" << std::endl;
    } else {
        std::cerr << "密钥对生成失败" << std::endl;
    }
    
    return success ? 0 : 1;
}
```

---

## 错误码

### Java 错误码

**代码位置**: `hapsigntool/hap_sign_tool_lib/src/main/java/com/ohos/hapsigntool/error/ERROR.java`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS_CODE` | 0 | 成功 |
| `COMMAND_ERROR` | 10001 | 命令错误 |
| `COMMAND_PARAM_ERROR` | 10002 | 参数错误 |
| `SIGN_ERROR` | 10003 | 签名错误 |
| `VERIFY_ERROR` | 10004 | 验证错误 |
| `IO_ERROR` | 10005 | IO 错误 |
| `READ_FILE_ERROR` | 10006 | 文件读取错误 |
| `WRITE_FILE_ERROR` | 10007 | 文件写入错误 |
| `UNKNOWN_ERROR` | 10099 | 未知错误 |

### C++ 错误码

**代码位置**: `hapsigntool_cpp/error/signature_tools_errno.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `SUCCESS` | 0 | 成功 |
| `COMMAND_ERROR` | 10001 | 命令错误 |
| `COMMAND_PARAM_ERROR` | 10002 | 参数错误 |
| `SIGN_ERROR` | 10003 | 签名错误 |
| `VERIFY_ERROR` | 10004 | 验证错误 |

---

## 相关链接

- [架构说明](02_Architecture.md) - 了解系统设计
- [构建系统](04_Build_System.md) - 查看构建配置
- [安全风险分析](05_Security_Analysis.md) - 了解安全注意事项

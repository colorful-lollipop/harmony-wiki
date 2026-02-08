# N-API 参考

本文档详细描述 `security_cangjie_wrapper` 导出的所有 Cangjie API。

## Kit 导出清单

| Kit 名称 | 导出路径 | 依赖模块 |
|----------|----------|----------|
| CryptoArchitectureKit | `kit.CryptoArchitectureKit` | `ohos.security.crypto_framework` |
| UniversalKeystoreKit | `kit.UniversalKeystoreKit` | `ohos.security.huks` |

**证据来源**：`kit/CryptoArchitectureKit/index.cj:18-20`、`kit/UniversalKeystoreKit/index.cj:18-20`

---

## Crypto Framework

### 模块概述

提供基础加密算法能力，包括对称加密、消息摘要、随机数、MAC 等。

**包路径**：`ohos.security.crypto_framework`

**SysCap**：`SystemCapability.Security.CryptoFramework.*`

### 1. DataBlob

```cj
// 证据来源：cj_crypto_common.cj:31-53
public class DataBlob {
    public var data: Array<UInt8>
    
    public init(data: Array<UInt8>) {
        this.data = data
    }
}
```

### 2. Key 接口

```cj
// 证据来源：cj_crypto_interface.cj:27-72
public interface Key {
    prop format: String
    prop algName: String
    func getEncoded(): DataBlob  // throws BusinessException
}
```

### 3. SymKey（对称密钥）

```cj
// 证据来源：sym_key.cj:43-143
public class SymKey <: RemoteDataLite & Key {
    public prop algName: String  // 例如 "AES256"
    public prop format: String   // 例如 "PKCS#12"
    
    // 获取密钥编码
    public func getEncoded(): DataBlob  // throws 17620001, 17630001
    
    // 清除内存中的密钥
    public func clearMem(): Unit
}
```

### 4. ParamsSpec（加密参数）

```cj
// 证据来源：cj_crypto_interface.cj:82-99, cj_crypto_common.cj:63-201
public class ParamsSpec {
    public var algName: String
}

// IvParamsSpec - CBC/CTR/OFB/CFB 模式
public class IvParamsSpec <: ParamsSpec {
    public var iv: DataBlob  // AES: 16字节, 3DES: 8字节, SM4: 16字节
}

// GcmParamsSpec - GCM 模式
public class GcmParamsSpec <: ParamsSpec {
    public var iv: DataBlob
    public var aad: DataBlob      // 附加认证数据
    public var authTag: DataBlob  // 认证标签
}

// CcmParamsSpec - CCM 模式
public class CcmParamsSpec <: ParamsSpec {
    public var iv: DataBlob
    public var aad: DataBlob
    public var authTag: DataBlob
}
```

### 5. Cipher（加解密）

```cj
// 证据来源：cipher.cj:64-253
public func createCipher(transformation: String): Cipher
// throws 801, 17620001

public class Cipher <: RemoteDataLite {
    public prop algName: String
    
    public func initialize(opMode: CryptoMode, key: Key, params: ?ParamsSpec): Unit
    // throws 17620001, 17620002, 17630001
    
    public func update(data: DataBlob): DataBlob
    // workerthread: true
    // throws 17620001, 17620002, 17630001
    
    public func doFinal(data: ?DataBlob): DataBlob
    // workerthread: true
    // throws 17620001, 17620002, 17630001
}

// 加密模式
public enum CryptoMode {
    EncryptMode  // 值: 0
    DecryptMode  // 值: 1
}

// 使用示例
let cipher = createCipher("AES256|GCM|NoPadding")
cipher.initialize(CryptoMode.EncryptMode, key, GcmParamsSpec(...))
let encrypted = cipher.doFinal(DataBlob(plainText))
```

### 6. SymKeyGenerator（对称密钥生成）

```cj
// 证据来源：sym_key_generator.cj:45-143
public func createSymKeyGenerator(algName: String): SymKeyGenerator
// throws 801

public class SymKeyGenerator <: RemoteDataLite {
    public prop algName: String
    
    // 随机生成密钥
    public func generateSymKey(): SymKey
    // workerthread: true
    // throws 17620001
    
    // 从二进制数据转换
    public func convertKey(key: DataBlob): SymKey
    // workerthread: true
    // throws 17620001
}

// 使用示例
let generator = createSymKeyGenerator("AES256")
let symKey = generator.generateSymKey()
// 或
let keyData = DataBlob(binaryKeyData)
let symKey = generator.convertKey(keyData)
```

### 7. Md（消息摘要）

```cj
// 证据来源：md.cj:45-165
public func createMd(algName: String): Md
// throws 17620001

public class Md <: RemoteDataLite {
    public prop algName: String
    
    public func update(input: DataBlob): Unit
    // throws 17620001, 17630001
    
    public func digest(): DataBlob
    // workerthread: true
    // throws 17620001, 17620002, 17630001
    
    public func getMdLength(): UInt32
    // throws 17630001
}

// 使用示例
let md = createMd("SHA256")
md.update(DataBlob(data1))
md.update(DataBlob(data2))
let result = md.digest()
```

### 8. Random（随机数）

```cj
// 证据来源：random.cj:44-143
public func createRandom(): Random
// throws 17620001

public class Random <: RemoteDataLite {
    public prop algName: String  // 当前仅支持 "CTR_DRBG"
    
    public func generateRandom(len: Int32): DataBlob
    // throws 17620001, 17630001
    
    public func setSeed(seed: DataBlob): Unit
    // throws 17620001
}

// 使用示例
let random = createRandom()
let randomData = random.generateRandom(32)
```

### 9. Mac（消息认证码）

```cj
// 证据来源：mac.cj:47-186
public func createMac(algName: String): Mac
// throws 17620001

public class Mac <: RemoteDataLite {
    public prop algName: String
    
    public func initialize(key: SymKey): Unit
    // throws 17620001, 17630001
    
    public func update(input: DataBlob): Unit
    // workerthread: true
    // throws 17620001, 17630001
    
    public func doFinal(): DataBlob
    // workerthread: true
    // throws 17620001, 17620002, 17630001
    
    public func getMacLength(): UInt32
    // throws 17630001
}

// 使用示例
let mac = createMac("HMAC|SHA256")
mac.initialize(symKey)
mac.update(DataBlob(message))
let macResult = mac.doFinal()
```

### 10. CryptoMode（加密模式枚举）

```cj
// 证据来源：cj_crypto_enum.cj:131-158
public enum CryptoMode {
    EncryptMode  // 加密操作
    DecryptMode  // 解密操作
}
```

### 11. Result（错误结果枚举）

```cj
// 证据来源：cj_crypto_enum.cj:29-93
enum Result {
    InvalidParams           // 401
    | NotSupport            // 801
    | ErrOutOfMemory        // 17620001
    | ErrRuntimeError       // 17620002
    | ErrCryptoOperation    // 17630001
    // ...
}
```

---

## HUKS（密钥管理）

### 模块概述

提供完整的密钥生命周期管理，包括密钥生成、存储、使用、销毁和证明。

**包路径**：`ohos.security.huks`

**SysCap**：`SystemCapability.Security.Huks.*`

### 1. HuksParam（参数项）

```cj
// 证据来源：huks_struct.cj:29-61
public class HuksParam {
    public var tag: UInt32
    public var value: HuksParamValue
}
```

### 2. HuksParamValue（参数值联合类型）

```cj
// 证据来源：huks_struct.cj:76-168
public enum HuksParamValue {
    BooleanValue(Bool)
    | Int32Value(Int32)
    | Uint32Value(UInt32)
    | Uint64Value(UInt64)
    | BytesValue(Bytes)
    // ...
}
```

### 3. HuksOptions（参数集）

```cj
// 证据来源：huks_struct.cj:173-209
public class HuksOptions {
    public var properties: Array<HuksParam>
    public var inData: Bytes
    
    public init(properties!: Array<HuksParam> = [], inData!: Bytes = Bytes())
}
```

### 4. HuksSessionHandle（会话句柄）

```cj
// 证据来源：huks_struct.cj:214-243
public class HuksSessionHandle {
    public var handle: HuksHandleId
    public var challenge: Bytes
}
```

### 5. HuksHandleId（句柄 ID）

```cj
// 证据来源：huks_struct.cj:248-257
public class HuksHandleId {
    var data: Bytes
}
```

### 6. initSession（初始化会话）

```cj
// 证据来源：huks_session.cj:49-98
public func initSession(keyAlias: String, options: HuksOptions): HuksSessionHandle
// workerthread: true
// throws: 801, 12000001-12000014, 12000018

// 前置条件检查
if (keyAlias.isEmpty()) {
    throw BusinessException(401, "key alias is empty")
}
if (options.properties.isEmpty()) {
    throw BusinessException(401, "properties is None")
}
```

### 7. updateSession（更新会话）

```cj
// 证据来源：huks_session.cj:137-139
public func updateSession(
    handle: HuksHandleId,
    options: HuksOptions,
    token!: Bytes = Bytes()
): Option<Bytes>
// workerthread: true
// throws: 801, 12000001-12000011, 12000014
```

### 8. finishSession（完成会话）

```cj
// 证据来源：huks_session.cj:170-172
public func finishSession(
    handle: HuksHandleId,
    options: HuksOptions,
    token!: Bytes = Bytes()
): Option<Bytes>
// workerthread: true
// throws: 801, 12000001-12000011, 12000014
```

### 9. abortSession（中止会话）

```cj
// 证据来源：huks_session.cj:293-316
public func abortSession(handle: HuksHandleId, options: HuksOptions): Unit
// workerthread: true
// throws: 801, 12000004-12000006, 12000012, 12000014
```

### 10. hasKeyItem（检查密钥是否存在）

```cj
// 证据来源：huks_key_item.cj:46-79
public func hasKeyItem(keyAlias: String, options: HuksOptions): Bool
// workerthread: true
// throws: 801, 12000004-12000006, 12000011-12000014
```

### 11. getKeyItemProperties（获取密钥属性）

```cj
// 证据来源：huks_key_item.cj:103-129
public func getKeyItemProperties(keyAlias: String, _: HuksOptions): Array<HuksParam>
// workerthread: true
// throws: 801, 12000001, 12000004-12000006, 12000011-12000014
```

### 12. generateKeyItem（生成密钥）

```cj
// 证据来源：huks_key_item.cj:364-391
public func generateKeyItem(keyAlias: String, options: HuksOptions): Unit
// workerthread: true
// throws: 801, 12000001-12000006, 12000012-12000017
```

### 13. importKeyItem（导入密钥）

```cj
// 证据来源：huks_key_item.cj:471-512
public func importKeyItem(keyAlias: String, options: HuksOptions): Unit
// workerthread: true
// throws: 801, 12000001-12000006, 12000011-12000017
```

### 14. importWrappedKeyItem（导入包装密钥）

```cj
// 证据来源：huks_key_item.cj:297-335
public func importWrappedKeyItem(
    keyAlias: String,
    wrappingKeyAlias: String,
    options: HuksOptions
): Unit
// workerthread: true
// throws: 201, 801, 12000001-12000006, 12000011-12000017
```

### 15. exportKeyItem（导出密钥）

```cj
// 证据来源：huks_key_item.cj:235-267
public func exportKeyItem(keyAlias: String, _: HuksOptions): Bytes
// workerthread: true
// throws: 801, 12000001, 12000004-12000006, 12000011-12000014
```

### 16. deleteKeyItem（删除密钥）

```cj
// 证据来源：huks_key_item.cj:414-443
public func deleteKeyItem(keyAlias: String, options: HuksOptions): Unit
// workerthread: true
// throws: 801, 12000004-12000006, 12000011-12000012, 12000014
```

### 17. anonAttestKeyItem（匿名证明）

```cj
// 证据来源：huks_key_item.cj:157-211
public func anonAttestKeyItem(keyAlias: String, options: HuksOptions): Array<String>
// workerthread: true
// throws: 201, 801, 12000001, 12000004-12000006, 12000011-12000014
```

---

## HUKS 常量定义

### HuksExceptionErrCode（错误码）

```cj
// 证据来源：huks_enum.cj:27-167
enum HuksExceptionErrCode {
    HuksErrCodePermissionFail         // 201
    | HuksErrCodeIllegalArgument     // 401
    | HuksErrCodeNotSupportedApi     // 801
    | HuksErrCodeFeatureNotSupported // 12000001
    | HuksErrCodeMissingCryptoAlgArgument  // 12000002
    | HuksErrCodeInvalidCryptoAlgArgument  // 12000003
    | HuksErrCodeFileOperationFail   // 12000004
    | HuksErrCodeCommunicationFail   // 12000005
    | HuksErrCodeCryptoFail          // 12000006
    | HuksErrCodeKeyAuthPermanentlyInvalidated  // 12000007
    | HuksErrCodeKeyAuthVerifyFailed // 12000008
    | HuksErrCodeKeyAuthTimeOut      // 12000009
    | HuksErrCodeSessionLimit         // 12000010
    | HuksErrCodeItemNotExist        // 12000011
    | HuksErrCodeExternalError       // 12000012
    | HuksErrCodeCredentialNotExist  // 12000013
    | HuksErrCodeInsufficientMemory  // 12000014
    | HuksErrCodeCallServiceFailed   // 12000015
}
```

### HuksKeyPurpose（密钥用途）

```cj
// 证据来源：huks_enum.cj:176-249
class HuksKeyPurpose {
    public static const HUKS_KEY_PURPOSE_ENCRYPT: UInt32 = 1
    public static const HUKS_KEY_PURPOSE_DECRYPT: UInt32 = 2
    public static const HUKS_KEY_PURPOSE_SIGN: UInt32 = 4
    public static const HUKS_KEY_PURPOSE_VERIFY: UInt32 = 8
    public static const HUKS_KEY_PURPOSE_DERIVE: UInt32 = 16
    public static const HUKS_KEY_PURPOSE_WRAP: UInt32 = 32
    public static const HUKS_KEY_PURPOSE_UNWRAP: UInt32 = 64
    public static const HUKS_KEY_PURPOSE_MAC: UInt32 = 128
    public static const HUKS_KEY_PURPOSE_AGREE: UInt32 = 256
}
```

### HuksKeyAlg（密钥算法）

```cj
// 证据来源：huks_enum.cj:611-724
class HuksKeyAlg {
    public static const HUKS_ALG_RSA: UInt32 = 1
    public static const HUKS_ALG_ECC: UInt32 = 2
    public static const HUKS_ALG_AES: UInt32 = 20
    public static const HUKS_ALG_HMAC: UInt32 = 50
    public static const HUKS_ALG_SM2: UInt32 = 150
    public static const HUKS_ALG_SM4: UInt32 = 152
    // ...
}
```

### HuksKeySize（密钥长度）

```cj
// 证据来源：huks_enum.cj:448-602
class HuksKeySize {
    // RSA
    public static const HUKS_RSA_KEY_SIZE_512: UInt32 = 512
    public static const HUKS_RSA_KEY_SIZE_2048: UInt32 = 2048
    // ...
    
    // ECC
    public static const HUKS_ECC_KEY_SIZE_256: UInt32 = 256
    // ...
    
    // AES
    public static const HUKS_AES_KEY_SIZE_128: UInt32 = 128
    public static const HUKS_AES_KEY_SIZE_256: UInt32 = 256
    // ...
}
```

### HuksTag（参数标签）

```cj
// 证据来源：huks_enum.cj:1172-1590
class HuksTag {
    public static const HUKS_TAG_ALGORITHM: UInt32 = ...
    public static const HUKS_TAG_PURPOSE: UInt32 = ...
    public static const HUKS_TAG_KEY_SIZE: UInt32 = ...
    public static const HUKS_TAG_DIGEST: UInt32 = ...
    public static const HUKS_TAG_PADDING: UInt32 = ...
    public static const HUKS_TAG_BLOCK_MODE: UInt32 = ...
    // ... 80+ 标签定义
}
```

---

## API 使用模式

### 同步 vs 异步

| 模式 | 说明 | 示例 |
|------|------|------|
| 同步 | API 直接返回结果 | `createCipher()`、`generateSymKey()` |
| Worker Thread | 标注 `workerthread: true` | `updateSession()`、`digest()` |

### 参数校验

所有 API 均进行参数校验：

```cj
// 空值检查
if (keyAlias.isEmpty()) {
    throw BusinessException(401, "key alias is empty")
}

// 数组非空检查
if (options.properties.isEmpty()) {
    throw BusinessException(401, "properties is None")
}
```

### 异常处理

```cj
try {
    let cipher = createCipher("AES256|GCM|NoPadding")
    cipher.initialize(CryptoMode.EncryptMode, key, params)
    let result = cipher.doFinal(data)
} catch (e: BusinessException) {
    // 处理异常
    let errorCode = e.code
    let message = e.message
}
```

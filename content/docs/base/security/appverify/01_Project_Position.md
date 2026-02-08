# 项目定位与边界

## 目的

说明 appverify 模块的项目定位、核心能力、运行环境和边界条件。

## 适用范围

- 目标读者：产品经理、架构师、新人开发者
- 涵盖内容：定位、能力、环境、约束、边界

## 关键结论

1. **定位**：安全子系统的核心签名验证模块，提供 HAP 包完整性校验能力
2. **接口方式**：纯 InnerKit C++ API，无 JS API 层，无 Service Ability
3. **适用场景**：仅限应用安装时的签名验证，不涉及运行时校验
4. **边界条件**：不支持应用更新时的增量验证、不支持运行时完整性监控

## 项目定位

### 在 OpenHarmony 安全子系统中的位置

```
OpenHarmony 安全子系统
│
├── 应用安全
│   ├── appverify ← 本模块 (HAP 签名验证)
│   ├── accesstoken (权限管理)
│   └── datahuks (密钥管理)
│
├── 系统安全
│   ├── selinux (访问控制)
│   └── security_guard (安全框架)
│
└── 数据安全
    └── file_management (文件加密)
```

### 核心价值

1. **完整性保障**：通过签名验证确保 HAP 包在分发过程中未被篡改
2. **来源可信**：通过证书链与可信源匹配，识别应用来源
3. **权限管控**：解析 Provision 配置，确保应用权限符合配置声明
4. **防恶意应用**：证书吊销列表（CRL）检查，防止已吊销证书签名应用安装

## 核心能力

### 1. HAP 包签名验证

**功能**：验证 HAP 包的 PKCS7 签名块

**输入**：
- HAP 文件路径（.hap/.hsp/.hqf/.app）
- 可选：本地证书目录（用于测试）

**输出**：
- `HapVerifyResult`：包含验证结果、证书链、签名信息、Provision 配置
- 返回码：`HapVerifyResultCode` 枚举

**关键能力**：
- 支持多算法：RSA-PSS/PKCS1v1.5、ECDSA、DSA
- 支持多摘要：SHA256/384/512
- 证书链验证：有效期、签名、CRL 检查
- 完整性验证：文件内容摘要比对

**证据**：
- 入口：`HapVerify()` - hap_verify.cpp:91
- 实现：`HapVerifyV2::Verify()` - hap_verify_v2.cpp:51

### 2. 应用来源识别

**功能**：通过匹配签名证书与可信源配置，识别应用来源

**可信源类型**：
- APP_GALLERY：华为应用市场签名
- APP_SYSTEM：系统签名
- APP_THIRD_PARTY_PRELOAD：第三方预装签名

**匹配规则** (trusted_source_manager.cpp):
- MATCH_WITH_SIGN：签名块签名证书直接匹配可信源
- MATCH_WITH_PROFILE：Profile 签名证书匹配可信源
- MATCH_WITH_PROFILE_DEBUG：Debug Profile 签名证书匹配测试源
- DO_NOT_MATCH：不在可信源中

**证据**：
- 配置：`trusted_apps_sources.json` - config/trusted_apps_sources.json
- 实现：`TrustedSourceManager::IsTrustedSource()` - trusted_source_manager.cpp

### 3. Provision 配置解析与验证

**功能**：解析应用配置文件，验证权限和分发类型

**ProvisionInfo 包含** (provision_info.h):
```cpp
struct BundleInfo {
    std::string bundleName;
    std::string appIdentifier;
    int32_t apiReleaseType;
    std::string versionCode;
    std::string versionName;
};

struct Permissions {
    std::vector<std::string> restrictedPermissions;
    std::vector<std::string> restrictedCapabilities;
};

struct ProvisionInfo {
    BundleInfo bundleInfo;
    Permissions permissions;
    std::vector<std::string> allowedAcls;
    ProvisionType type;  // DEBUG/RELEASE
    AppDistType distributionType;  // APP_GALLERY/ENTERPRISE/...
};
```

**验证逻辑** (hap_verify_v2.cpp:361):
- Debug 模式：验证开发证书与签名证书一致
- Release 模式：验证分发类型允许安装
- 检查设备授权：设备 ID 是否在 Provision 设备列表中

### 4. 企业应用验证

**功能**：支持企业重签名应用的特殊验证逻辑

**验证流程** (enterprise_resign_mgr.cpp):
1. 检查应用分发类型为 `ENTERPRISE/ENTERPRISE_NORMAL/ENTERPRISE_MDM`
2. 验证证书包含企业 OID (1.3.6.1.4.1.2011.2.376.1.9)
3. 验证设备为企业设备 (`const.edm.is_enterprise_device`)
4. 比对本地证书链与 HAP 证书链
5. 验证叶子证书颁发者

**证据**：
- 实现：`EnterpriseResignMgr::Verify()` - enterprise_resign_mgr.cpp

### 5. Ticket 验证（OpenTest 应用）

**功能**：验证 OpenTest 应用的授权 Ticket

**验证内容** (ticket_verify.cpp):
- Ticket 签名可信度
- Ticket 与 Profile 的 bundleName 一致性
- Ticket 与 Profile 的证书匹配
- 权限比对：Ticket 权限是否覆盖 Profile 权限
- 设备授权：设备 ID 是否在 Ticket 中

**证据**：
- 实现：`TicketVerify::Verify()` - ticket_verify.cpp

## 运行环境

### 支持的系统类型

| 系统类型 | 库文件 | 完整功能 | 支持的设备 |
|----------|--------|----------|-----------|
| Standard | libhapverify.so | ✅ 完整 | 平板、手机、IoT 设备 |
| Small | libverify.so | ⚠️ 简化 | 小型 IoT 设备 |
| Mini | libverify.so | ⚠️ 简化 | 极小型设备 |

**Lite 版本差异** (appverify_lite):
- 使用 mbedtls 替代 OpenSSL
- 简化的证书验证
- 不支持部分高级特性（如 Ticket 验证）

### 资源占用

- **ROM**：约 5000kb（来源：bundle.json:22）
- **RAM**：约 500kb（来源：bundle.json:23）

### 编译选项

#### 标准系统特性 (STANDARD_SYSTEM)

- 使用 `hilog:libhilog` 记录日志
- 使用 `init:libbegetutil` 获取系统参数
- 不依赖 IPC

#### 非标准系统特性

- 依赖 `ipc:ipc_core` 进行 IPC 通信
- 依赖 `os_account:libaccountkits` 获取账号信息
- 支持 `SUPPORT_GET_DEVICE_TYPES` 获取设备类型

#### 其他编译宏

| 宏 | 默认值 | 说明 |
|----|--------|------|
| HILOG_ENABLE | 是 | 启用 Hilog 日志 |
| OPENSSL_SUPPRESS_DEPRECATED | 是 | 抑制 OpenSSL 过期 API 警告 |
| X86_EMULATOR_MODE | 否 (is_emulator) | x86 模拟器模式 |
| PARSE_PEM_FORMAT_SIGNED_DATA | 是 (Lite) | 解析 PEM 格式签名数据 |

## 约束与限制

### 1. 调用场景限制

**支持的调用**：
- ✅ 应用安装时验证
- ✅ Profile 独立验证
- ✅ 签名信息解析

**不支持的调用**：
- ❌ 运行时完整性监控
- ❌ 增量更新验证
- ❌ 应用卸载时验证

### 2. 输入文件限制

**支持的文件扩展名** (hap_verify_v2.cpp:44-48):
- `.hap` - 标准应用包
- `.hsp` - 共享包
- `.hqf` - Quick 应用包
- `.app` - Legacy 应用包
- `.p7b` - Provision 文件

**文件大小限制**：
- 单个文件不超过 2GB (hap_verify_v2.cpp:926)

### 3. 签名算法限制

**支持的算法** (hap_verify_openssl_utils.h:35-48):

| 算法类型 | ID | 摘要算法 | 备注 |
|----------|----|----------|------|
| RSA-PSS | 0x00000101 | SHA256/384/512 | 推荐 |
| RSA-PKCS1v1.5 | 0x00000201 | SHA256/384/512 | 兼容性 |
| ECDSA | 0x00000201 | SHA256/384/512 | 现代标准 |
| DSA | 0x00000301 | SHA256/384/512 | 兼容性 |

**不支持的算法**：
- ❌ EdDSA
- ❌ SM2/SM3（国产算法）

### 4. 证书限制

**支持的证书格式**：
- X.509 证书（DER 编码）

**支持的密钥类型**：
- RSA（2048 位及以上）
- EC（P-256, P-384, P-521）
- DSA（2048 位及以上）

**证书链限制**：
- 最长链：暂无硬性限制（依赖系统内存）
- 根证书：必须在 `trusted_root_ca.json` 中配置

### 5. 性能约束

**文件读取**：
- 支持并行计算摘要（HapVerifyParallelizationSupported）
- 大文件分块读取（默认 4MB 块）

**内存使用**：
- 大文件验证可能占用较多内存（读取完整签名块）
- 不支持流式验证（需要加载完整签名块）

## 边界条件

### 1. 证书有效期边界

- 证书**过期**：返回 `CERTIFICATE_EXPIRED` (hap_verify_result.h:51)
- 证书**未生效**：在有效期前，验证失败
- 证书**吊销**：CRL 检查失败

### 2. 可信源匹配边界

- **完全匹配**：主题和颁发者都匹配 → `MATCH_WITH_SIGN`
- **仅 Profile 匹配**：只有 Profile 签名匹配 → `MATCH_WITH_PROFILE`
- **不匹配**：不在任何可信源中 → `DO_NOT_MATCH`
- **不匹配后果**：返回 `APP_SOURCE_NOT_TRUSTED` (hap_verify_result.h:41)

### 3. 分发类型边界

**允许安装** (hap_verify_v2.cpp:392):
- `APP_GALLERY`：需要 Ticket 验证
- `ENTERPRISE`：需要企业验证
- `ENTERPRISE_NORMAL`：需要企业验证
- `ENTERPRISE_MDM`：需要 MDM 企业验证
- `OS_INTEGRATION`：允许
- `CROWDTESTING`：允许
- `INTERNALTESTING`：不允许（设备鉴权场景）

**拒绝安装**：
- `INTERNALTESTING`（ParseBundleNameAndAppIdentifier 场景）
- `NONE_TYPE`：未知类型

### 4. 设备授权边界

**开发模式** (provision_verify.cpp):
- 设备 ID 在 `debugInfo.deviceIds` 列表中 → 允许
- 设备 ID 不在列表中 → 拒绝
- 空 `deviceIds` → 允许任意设备（某些场景）

**企业模式** (enterprise_resign_mgr.cpp):
- `const.edm.is_enterprise_device` = true → 允许
- `const.edm.is_enterprise_device` = false → 拒绝

## 相关跳转

- [架构详解](03_Architecture.md) - 完整验证流程
- [对外 API](04_Public_API.md) - API 使用方式
- [安全评审](08_Security_Review.md) - 安全边界与风险
- [GN 构建目标](06_GN_Targets.md) - 构建配置

# 目录结构与模块职责

## 目的

说明 appverify 模块的目录组织、各子模块的职责和依赖关系。

## 适用范围

- 目标读者：开发者、代码审查人员
- 涵盖内容：目录树、模块职责、依赖关系、代码规范

## 关键结论

1. **目录结构**：按功能分层组织（interfaces/init/util/verify/provision/ticket/common）
2. **模块划分**：22 个源文件，6 个功能子模块
3. **依赖方向**：单向依赖，无循环依赖
4. **测试隔离**：test/ 目录与生产代码分离

## 目录结构

### 顶层目录

```
/base/security/appverify/
├── interfaces/innerkits/appverify/    # 主模块（标准系统）
│   ├── include/                      # 头文件
│   ├── src/                         # 源代码
│   ├── config/                       # 配置文件（预构建）
│   └── test/                        # 单元测试（忽略）
├── interfaces/innerkits/appverify_lite/  # 精简版（Lite 系统）
│   ├── include/
│   ├── src/
│   └── unittest/                    # 单元测试（忽略）
├── test/resource/                    # 测试资源（忽略）
├── figures/                         # 架构图
├── BUILD.gn                        # 根构建文件
├── bundle.json                      # 组件元数据
├── README.md                       # 项目说明（英文）
├── README_zh.md                    # 项目说明（中文）
└── OAT.xml                        # 开源审计信息
```

### 主模块结构（标准系统）

```
interfaces/innerkits/appverify/
├── include/                        # 对外暴露的头文件
│   ├── common/                     # 通用类
│   │   ├── export_define.h        # DLL_EXPORT 宏定义
│   │   ├── hap_byte_buffer.h      # 字节缓冲区
│   │   ├── hap_byte_buffer_data_source.h  # 数据源抽象
│   │   ├── hap_file_data_source.h        # 文件数据源
│   │   ├── hap_verify_log.h            # 日志封装
│   │   └── random_access_file.h         # 随机访问文件
│   ├── init/                       # 初始化管理器
│   │   ├── device_type_manager.h        # 设备类型管理
│   │   ├── hap_crl_manager.h           # CRL 管理
│   │   ├── json_parser_utils.h         # JSON 解析
│   │   ├── matching_result.h            # 匹配结果枚举
│   │   ├── trusted_root_ca.h           # 根证书管理
│   │   ├── trusted_source_manager.h     # 可信源管理
│   │   └── trusted_ticket_manager.h    # Ticket 管理
│   ├── interfaces/                 # 对外 API 接口
│   │   ├── hap_verify.h              # HapVerify 主接口
│   │   └── hap_verify_result.h       # 验证结果结构
│   ├── provision/                  # Provision 解析与验证
│   │   ├── provision_info.h            # ProvisionInfo 结构
│   │   └── provision_verify.h         # Provision 验证接口
│   ├── ticket/                     # Ticket 验证
│   │   └── ticket_verify.h            # Ticket 验证接口
│   ├── util/                       # 工具类
│   │   ├── digest_parameter.h          # 摘要参数
│   │   ├── hap_cert_verify_openssl_utils.h   # 证书验证工具
│   │   ├── hap_profile_verify_utils.h        # Profile 验证工具
│   │   ├── hap_signing_block_utils.h         # 签名块处理
│   │   ├── hap_verify_openssl_utils.h        # OpenSSL 工具
│   │   ├── pkcs7_context.h                  # PKCS7 上下文
│   │   └── signature_info.h                 # 签名信息
│   └── verify/                     # 核心验证逻辑
│       ├── enterprise_resign_mgr.h       # 企业重签名管理
│       └── hap_verify_v2.h             # HapVerifyV2 主类
│
├── src/                            # 源代码实现（与 include/ 对应）
│   ├── common/
│   ├── init/
│   ├── interfaces/
│   ├── provision/
│   ├── ticket/
│   ├── util/
│   └── verify/
│
└── config/                         # 预构建配置文件
    ├── OpenHarmony/                 # 公开版本配置
    │   ├── trusted_root_ca.json
    │   └── trusted_apps_sources.json
    ├── trusted_root_ca.json         # 内部版本
    ├── trusted_apps_sources.json
    ├── trusted_tickets_sources.json
    ├── trusted_root_ca_test.json    # 测试配置
    └── trusted_apps_sources_test.json
```

## 模块职责

### 1. Common 模块（通用基础）

**目录**：`common/`

**职责**：
- 提供通用数据结构和工具类
- 封装文件操作和数据源抽象
- 日志输出接口

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `HapByteBuffer` | hap_byte_buffer.h | 字节缓冲区，管理二进制数据 |
| `HapByteBufferDataSource` | hap_byte_buffer_data_source.h | 从缓冲区读取数据的源抽象 |
| `HapFileDataSource` | hap_file_data_source.h | 从文件读取数据的源抽象 |
| `RandomAccessFile` | random_access_file.h | 支持随机访问的文件封装 |
| `HapVerifyLog` | hap_verify_log.h | Hilog 日志封装 |

**文件**：
- `export_define.h` - DLL_EXPORT 宏（跨平台符号导出）

**依赖**：
- 依赖：`hilog`、`c_utils`
- 被依赖：所有其他模块

**稳定级别**：稳定（基础公共类）

### 2. Init 模块（初始化管理）

**目录**：`init/`

**职责**：
- 加载和管理系统配置（根证书、可信源）
- 单例模式管理器
- 提供配置查询接口

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `TrustedRootCa` | trusted_root_ca.h | 加载和管理根证书列表 |
| `TrustedSourceManager` | trusted_source_manager.h | 匹配应用签名证书与可信源 |
| `HapCrlManager` | hap_crl_manager.h | 证书吊销列表管理 |
| `DeviceTypeManager` | device_type_manager.h | 获取设备类型和模式 |
| `TrustedTicketManager` | trusted_ticket_manager.h | 加载和管理 Ticket 可信源 |
| `JsonParserUtils` | json_parser_utils.h | JSON 配置解析工具 |

**依赖**：
- 依赖：`common`、`cJSON`、`hilog`
- (非标准系统) `ipc`、`os_account`

**稳定级别**：稳定（配置管理接口不常变化）

### 3. Interfaces 模块（对外 API）

**目录**：`interfaces/`

**职责**：
- 定义对外暴露的 C++ API
- 提供同步验证接口
- 初始化和调试模式控制

**核心函数**：

| 函数 | 头文件 | 功能 |
|----|--------|------|
| `HapVerify()` | hap_verify.h | 主验证入口 |
| `ParseHapProfile()` | hap_verify.h | 解析 Provision |
| `ParseHapSignatureInfo()` | hap_verify.h | 解析签名信息 |
| `VerifyProfile()` | hap_verify.h | 独立验证 Provision |
| `VerifyProfileByP7bBlock()` | hap_verify.h | 从 P7b 块验证 |
| `EnableDebugMode()` | hap_verify.h | 启用调试模式 |
| `DisableDebugMode()` | hap_verify.h | 禁用调试模式 |
| `SetDevMode()` | hap_verify.h | 设置开发模式 |

**类**：
- `HapVerifyResult` - 验证结果结构（hap_verify_result.h）

**依赖**：
- 依赖：`verify`、`init`、`provision`、`util`、`common`

**稳定级别**：稳定（对外 API，需谨慎变更）

### 4. Provision 模块（配置解析与验证）

**目录**：`provision/`

**职责**：
- 解析 Provision PKCS7 签名
- 提取和验证 ProvisionInfo
- 设备授权检查

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `ProvisionVerify` | provision_verify.h | Provision 解析与验证 |

**核心函数** (ProvisionVerify)：
- `ParseAndVerify()` - 解析并验证 Provision
- `ParseProvision()` - 仅解析不验证
- `CheckDeviceID()` - 检查设备 ID 授权

**依赖**：
- 依赖：`util`、`init`、`common`

**稳定级别**：较稳定（Provision 格式稳定）

### 5. Ticket 模块（Ticket 验证）

**目录**：`ticket/`

**职责**：
- 验证 OpenTest 应用的授权 Ticket
- 比对 Ticket 与 Profile 信息
- 设备和权限检查

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `TicketVerify` | ticket_verify.h | Ticket 解析与验证 |

**核心函数** (TicketVerify)：
- `Verify()` - 验证 Ticket
- `CheckPermissions()` - 权限比对
- `CheckDevice()` - 设备检查
- `CompareTicketAndProfile()` - Ticket/Profile 一致性

**依赖**：
- 依赖：`util`、`init`、`common`
- (非标准系统) `os_account`

**稳定级别**：较稳定（Ticket 格式稳定）

### 6. Util 模块（工具类）

**目录**：`util/`

**职责**：
- OpenSSL 操作封装（PKCS7、证书、签名）
- HAP 签名块解析和完整性验证
- 摘要计算和算法工具

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `HapVerifyOpensslUtils` | hap_verify_openssl_utils.h | PKCS7 解析、签名验证、证书操作 |
| `HapCertVerifyOpensslUtils` | hap_cert_verify_openssl_utils.h | 证书链验证、CRL 检查、有效期验证 |
| `HapSigningBlockUtils` | hap_signing_block_utils.h | 签名块查找、完整性验证、摘要计算 |
| `HapProfileVerifyUtils` | hap_profile_verify_utils.h | Profile 解析工具 |
| `DigestParameter` | digest_parameter.h | 摘要参数封装 |
| `Pkcs7Context` | pkcs7_context.h | PKCS7 上下文结构 |
| `SignatureInfo` | signature_info.h | 签名信息结构 |

**依赖**：
- 依赖：`common`、`openssl`、`bounds_checking_function`

**稳定级别**：稳定（工具类）

### 7. Verify 模块（核心验证逻辑）

**目录**：`verify/`

**职责**：
- 编排验证流程
- 调用各子模块完成验证
- 应用来源和分发类型检查

**核心类**：

| 类 | 头文件 | 职责 |
|----|--------|------|
| `HapVerifyV2` | hap_verify_v2.h | HAP 验证核心流程 |
| `EnterpriseResignMgr` | enterprise_resign_mgr.h | 企业重签名验证 |

**核心函数** (HapVerifyV2)：
- `Verify(filePath)` - 主验证入口
- `Verify(fileFd)` - 文件描述符版本
- `ParseHapProfile()` - 解析 Profile
- `ParseHapSignatureInfo()` - 解析签名信息
- `VerifyAppPkcs7()` - 验证应用签名
- `VerifyAppSourceAndParseProfile()` - 来源验证与 Profile 解析
- `VerifyHapIntegrity()` - 完整性验证

**依赖**：
- 依赖：`util`、`init`、`provision`、`ticket`、`common`

**稳定级别**：较稳定（核心验证逻辑）

### 8. Config 模块（配置文件）

**目录**：`config/`

**职责**：
- 预构建系统配置文件
- 提供根证书和可信源配置

**配置文件**：

| 文件 | 说明 | 安装路径 |
|------|------|----------|
| `trusted_root_ca.json` | 根证书列表 | /system/etc/security/ |
| `trusted_apps_sources.json` | 可信应用源配置 | /system/etc/security/ |
| `trusted_tickets_sources.json` | Ticket 可信源配置 | /system/etc/security/ |
| `trusted_root_ca_test.json` | 测试根证书 | /system/etc/security/ |
| `trusted_apps_sources_test.json` | 测试可信源 | /system/etc/security/ |

**依赖**：无（静态配置）

**稳定级别**：稳定（配置文件格式固定）

## 依赖关系图

### 模块依赖层次

```
interfaces (对外 API)
    ↓
verify (核心验证)
    ↓
provision (配置解析) ← ticket (Ticket 验证)
    ↓                ↓
util (工具类) ──────→ init (初始化)
    ↓
common (基础)
```

### 模块间依赖矩阵

| 依赖者 | common | init | util | provision | ticket | verify | interfaces |
|--------|--------|------|------|-----------|---------|---------|-------------|
| **common** | - | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **init** | ✅ | - | ✗ | ✗ | ✗ | ✗ | ✗ |
| **util** | ✅ | ✗ | - | ✗ | ✗ | ✗ | ✗ |
| **provision** | ✅ | ✅ | ✅ | - | ✗ | ✗ | ✗ |
| **ticket** | ✅ | ✅ | ✅ | ✗ | - | ✗ | ✗ |
| **verify** | ✅ | ✅ | ✅ | ✅ | ✅ | - | ✗ |
| **interfaces** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |

**说明**：✅ = 直接依赖，✗ = 无依赖

**特点**：
- 无循环依赖
- 清晰的分层结构
- interfaces 层最上层

### 外部依赖

| 模块 | 外部依赖 |
|------|----------|
| common | hilog, c_utils |
| init | cJSON, hilog, c_utils, (ipc), (os_account) |
| util | openssl, bounds_checking_function |
| provision | util, init, common |
| ticket | util, init, common, (os_account) |
| verify | util, init, provision, ticket, common |
| interfaces | verify, init, provision, util, common |

**说明**：(ipc) 和 (os_account) 仅在非标准系统使用

## 代码规范

### 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 类名 | PascalCase | `HapVerifyV2`, `TrustedRootCa` |
| 函数名 | PascalCase | `HapVerify()`, `ParseHapProfile()` |
| 成员变量 | camelCase | `publicKeys`, `signatures` |
| 静态常量 | UPPER_SNAKE_CASE | `VERIFY_SUCCESS`, `MAX_OID_LENGTH` |
| 宏定义 | UPPER_SNAKE_CASE | `DLL_EXPORT`, `HILOG_ENABLE` |
| 命名空间 | lowercase | `OHOS::Security::Verify` |

### 文件组织

- 每个类一个头文件和实现文件
- 头文件放在 `include/`，实现放在 `src/`
- 按功能模块分组目录

### 注释规范

- 文件头注释：版权 + 许可证 + 描述
- 类注释：简要说明职责
- 公共函数注释：参数、返回值、用途
- 使用 Doxygen 风格（可选）

### 编码风格

- 遵循 OpenHarmony C++ 编码规范
- 使用 4 空格缩进
- 大括号不换行（K&R 风格）
- 使用 `HAPVERIFY_LOG_*` 宏记录日志

## 测试组织

### 测试目录结构

```
interfaces/innerkits/appverify/test/
└── unittest/
    ├── src/                      # 测试用例源码
    │   ├── hap_verify_test.cpp
    │   ├── hap_cert_verify_openssl_utils_test.cpp
    │   └── ...
    └── packets/                  # 测试数据包
        ├── business_packet.cpp
        ├── modified_packet.cpp
        └── ...
```

**注意**：测试代码不在本文档覆盖范围内

## 相关跳转

- [架构详解](03_Architecture.md) - 模块协作流程
- [内部 API](05_Internal_API.md) - 模块接口详情
- [GN 构建目标](06_GN_Targets.md) - 构建配置

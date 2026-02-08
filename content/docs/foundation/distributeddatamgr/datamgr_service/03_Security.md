# 安全风险评审

分析分布式数据管理服务的攻击面、信任边界和安全风险。

---

## 1. 攻击面分析

### 1.1 输入点清单

| 攻击面 | 入口点 | 数据类型 | 风险等级 |
|-------|--------|---------|---------|
| **IPC 接口** | `OnRemoteRequest()` | MessageParcel | 高 |
| **Feature 接口** | `Feature::OnRemoteRequest()` | MessageParcel | 高 |
| **数据库操作** | `GeneralStore` | KV/RDB 数据 | 中 |
| **云同步** | `CloudService` | 云端数据 | 中 |
| **备份/恢复** | `BackupManager` | 文件系统 | 中 |
| **配置加载** | `Bootstrap::LoadConfigs()` | JSON/XML | 低 |
| **事件监听** | `EventCenter` | 事件数据 | 低 |

### 1.2 IPC 接口详情

**Stub 文件**: 各模块均实现 `OnRemoteRequest()`:

| 模块 | Stub 文件 | 接口数量 |
|-----|----------|---------|
| **KVDB** | `service/kvdb/kvdb_service_stub.h` | ~30+ |
| **RDB** | `service/rdb/rdb_service_stub.h` | ~10+ |
| **Cloud** | `service/cloud/cloud_service_stub.h` | ~15+ |
| **UDMF** | `service/udmf/udmf_service_stub.h` | ~15+ |
| **DataShare** | `service/data_share/data_share_service_stub.h` | ~25+ |

**证据**: `service/kvdb/kvdb_service_stub.h` 定义了 `OnRemoteRequest()` 作为 IPC 入口

---

## 2. 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    服务端 (trusted)                       │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │  KvStoreData │  │   Feature    │  │   AutoCache │   │  │
│  │  │   Service    │  │   System     │  │              │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │ PermitDelegate│  │ MetaDataMgr │  │   CryptoMgr  │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                      边界 (IPC Layer)                           │
├─────────────────────────────────────────────────────────────────┤
│                     客户端 (untrusted)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   JS API    │  │  Native App  │  │  System App │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 现有安全机制

### 3.1 权限检查

| 组件 | 文件 | 检查点 |
|-----|------|-------|
| **PermissionValidator** | `service/permission/include/permission_validator.h` | `CheckSyncPermission()` |
| **PermitDelegate** | `service/permission/include/permit_delegate.h` | `VerifyPermission()` |
| **CheckerManager** | `framework/include/checker/checker_manager.h` | 签名/权限检查 |

**权限定义**:

```cpp
// permission_validator.h
static constexpr const char *DISTRIBUTED_DATASYNC = "ohos.permission.DISTRIBUTED_DATASYNC";
static constexpr const char *CLOUD_DATA_CONFIG = "ohos.permission.CLOUDDATA_CONFIG";
```

### 3.2 接口令牌校验

**证据**: 多个 Stub 实现 `CheckInterfaceToken()`:

| 模块 | 文件 |
|-----|------|
| RDB | `service/rdb/rdb_result_set_stub.h` |
| DataShare | `service/data_share/data_share_service_stub.h` |
| Object | `service/object/include/object_service_stub.h` |

### 3.3 密钥管理

| 组件 | 文件 | 机制 |
|-----|------|-----|
| **CryptoManager** | `framework/crypto/crypto_manager.cpp` | Huks 集成 |
| **SecretKeyMetaData** | `framework/include/metadata/secret_key_meta_data.h` | 密钥元数据 |

### 3.4 数据脱敏

**证据**: `framework/include/utils/anonymous.h` 提供数据匿名化功能

---

## 4. 风险点清单

### 4.1 IPC 参数校验不足

**风险等级**: 🟠 中

| 项目 | 说明 |
|-----|------|
| **描述** | IPC 接口参数校验可能不完整 |
| **证据** | `service/udmf/udmf_service_impl.h` 有 `CheckDragParams()` 方法，但未发现所有参数校验 |
| **触发** | 构造恶意 IPC 请求，传入异常参数 |
| **影响** | 可能的崩溃、拒绝服务 |
| **缓解措施** | 增加参数完整性校验，检查参数边界 |

### 4.2 路径遍历风险

**风险等级**: 🟠 中

| 项目 | 说明 |
|-----|------|
| **描述** | 文件路径处理可能存在遍历风险 |
| **证据** | `service/data_share/data_share_service_impl.cpp` 处理 URI 和路径 |
| **触发** | 构造包含 `../` 的恶意路径 |
| **影响** | 未授权文件访问 |
| **缓解措施** | 使用路径规范化 API，限制可访问目录 |

### 4.3 序列化安全

**风险等级**: 🟡 低

| 项目 | 说明 |
|-----|------|
| **描述** | `MessageParcel` 反序列化安全 |
| **证据** | 大量 `Unmarshalling()` 实现 (`service/cloud/cloud_types_util.h`) |
| **触发** | 恶意构造的序列化数据 |
| **影响** | 潜在的内存安全问题 |
| **缓解措施** | 检查数据长度，使用安全序列化函数 |

### 4.4 内存安全 (C++)

**风险等级**: 🟡 低至🟠 中

| 项目 | 说明 |
|-----|------|
| **描述** | 使用 `malloc`, `strcpy_s`, `sprintf_s` 等 |
| **证据** | `adapter/communicator/test/mock/` 中使用 `strcpy_s` |
| **触发** | 缓冲区溢出 |
| **影响** | 内存损坏、代码执行 |
| **缓解措施** | 使用现代 C++ 智能指针，避免原始内存操作 |

**已使用安全函数**:
- `strcpy_s` (安全字符串拷贝)
- `sprintf_s` (安全格式化)
- `malloc` 配合错误检查

### 4.5 动态库加载

**风险等级**: 🟡 低

| 项目 | 说明 |
|-----|------|
| **描述** | `Bootstrap::LoadComponents()` 使用 `dlopen` |
| **证据** | `service/bootstrap/src/bootstrap.cpp:68` |
| **触发** | 加载恶意动态库 |
| **影响** | 代码执行风险 |
| **缓解措施** | 动态库来自受信任路径，由系统签名保证 |

### 4.6 Fuzz 测试覆盖

**正面发现**: 项目有 fuzz 测试:

| 测试文件 | 路径 |
|---------|------|
| `udmfservicecheckpermission_fuzzer` | `service/test/fuzztest/` |
| `udmfservicecheckkeyandintention_fuzzer` | `service/test/fuzztest/` |
| `udmfservicechecktransferparams_fuzzer` | `service/test/fuzztest/` |

---

## 5. 安全建议

### 5.1 高优先级

| 建议 | 优先级 | 难度 |
|-----|-------|------|
| 增加 IPC 参数校验 | 高 | 中 |
| 实施路径规范化 | 高 | 低 |
| 增强序列化安全 | 高 | 中 |

### 5.2 中优先级

| 建议 | 优先级 | 难度 |
|-----|-------|------|
| 增加输入边界检查 | 中 | 低 |
| 增加安全日志记录 | 中 | 低 |
| 定期安全审计 | 中 | 高 |

### 5.3 低优先级

| 建议 | 优先级 | 难度 |
|-----|-------|------|
| 考虑使用 Rust 组件 | 低 | 高 |
| 增加 ASan/TSan 测试 | 低 | 低 |

---

## 6. 检查范围说明

### 6.1 已检查范围

| 类别 | 路径/文件 |
|-----|----------|
| 权限检查 | `service/permission/` |
| IPC 接口 | `service/*/*_stub.h` |
| 配置加载 | `service/bootstrap/` |
| 密钥管理 | `framework/crypto/` |
| 数据校验 | `service/udmf/permission/` |

### 6.2 未检查范围

| 类别 | 原因 |
|-----|------|
| 底层存储 (kv_store) | 独立仓库 |
| 关系数据库 (relational_store) | 独立仓库 |
| 云端服务 | 超出本仓库范围 |
| 客户端 N-API | 无 N-API 层 |

---

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| 概览 | [00_Overview.md](00_Overview.md) |
| 架构 | [01_Architecture.md](01_Architecture.md) |
| 构建 | [02_Build.md](02_Build.md) |

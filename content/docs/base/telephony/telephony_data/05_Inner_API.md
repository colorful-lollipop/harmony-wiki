# 内部 API 与模块接口

## 5.1 模块接口概览

### 5.1.1 公共头文件层级

| 层级 | 目录 | 稳定性 | 描述 |
|------|------|--------|------|
| **对外接口** | `interfaces/innerkits/include/` | 稳定 | 供其他子系统使用的公开 API |
| **模块头文件** | `*/include/` | 较稳定 | 本模块内部使用的接口 |
| **实现头文件** | `common/include/` | 稳定 | 公共基础设施接口 |

### 5.1.2 接口稳定性标注

| 符号 | 含义 | 使用场景 |
|------|------|----------|
| `⭐⭐⭐` | 高度稳定 | 对外 API、结构体定义 |
| `⭐⭐` | 较稳定 | 内部模块接口 |
| `⭐` | 可能变化 | 私有实现细节 |

---

## 5.2 对外 Inner API

### 5.2.1 数据结构定义 (interfaces/innerkits/include/)

| 头文件 | 稳定性 | 导出符号 |
|--------|--------|----------|
| `sim_data.h` | ⭐⭐⭐ | `SimData` 结构体字段常量 |
| `sms_mms_data.h` | ⭐⭐⭐ | `SmsMmsInfo` 结构体字段常量 |
| `pdp_profile_data.h` | ⭐⭐⭐ | `PdpProfileData` 结构体字段常量 |
| `opkey_data.h` | ⭐⭐⭐ | `OpKeyData` 结构体字段常量 |
| `global_params_data.h` | ⭐⭐⭐ | `GlobalParamsData` 结构体字段常量 |

**代码证据**: `interfaces/innerkits/include/sim_data.h`

```cpp
// SIM 数据结构定义
struct SimData {
    static constexpr const char *SIM_ID = "sim_id";
    static constexpr const char *ICC_ID = "icc_id";
    static constexpr const char *CARD_ID = "card_id";
    static constexpr const char *SLOT_INDEX = "slot_index";
    static constexpr const char *SHOW_NAME = "show_name";
    static constexpr const char *PHONE_NUMBER = "phone_number";
    // ...
};
```

---

## 5.3 模块内部接口

### 5.3.1 common 模块接口

| 接口类 | 文件 | 稳定性 | 职责 |
|--------|------|--------|------|
| `RdbBaseHelper` | `rdb_base_helper.h` | ⭐⭐⭐ | RDB 数据库基础操作 |
| `RdbBaseCallback` | `rdb_base_callback.h` | ⭐⭐ | RDB 操作回调基类 |
| `PermissionUtil` | `permission_util.h` | ⭐⭐⭐ | 权限检查封装 |
| `TelephonyDataShareStubImpl` | `telephony_datashare_stub_impl.h` | ⭐⭐⭐ | IPC 存根实现 |
| `ParserUtil` | `parser_util.h` | ⭐⭐ | 数据解析工具 |
| `TimeUtil` | `time_util.h` | ⭐⭐ | 时间格式化工具 |
| `DataStorageLogWrapper` | `data_storage_log_wrapper.h` | ⭐⭐ | 日志封装 |

### 5.3.2 RdbBaseHelper 接口

**文件**: `common/include/rdb_base_helper.h`

| 方法 | 描述 | 稳定性 |
|------|------|--------|
| `Init()` | 初始化 RDB 数据库 | ⭐⭐⭐ |
| `ExecuteInsert()` | 执行插入操作 | ⭐⭐⭐ |
| `ExecuteUpdate()` | 执行更新操作 | ⭐⭐⭐ |
| `ExecuteDelete()` | 执行删除操作 | ⭐⭐⭐ |
| `ExecuteQuery()` | 执行查询操作 | ⭐⭐⭐ |
| `ExecuteBatchInsert()` | 执行批量插入 | ⭐⭐⭐ |

### 5.3.3 TelephonyDataShareStubImpl 接口

**文件**: `common/include/telephony_datashare_stub_impl.h`

| 方法 | 描述 | 稳定性 |
|------|------|--------|
| `Insert(uri, value)` | 路由 Insert 请求 | ⭐⭐⭐ |
| `Update(uri, predicates, value)` | 路由 Update 请求 | ⭐⭐⭐ |
| `Delete(uri, predicates)` | 路由 Delete 请求 | ⭐⭐⭐ |
| `Query(uri, predicates, columns)` | 路由 Query 请求 | ⭐⭐⭐ |
| `BatchInsert(uri, values)` | 路由 BatchInsert 请求 | ⭐⭐⭐ |
| `GetOwner(uri)` | 根据 URI 获取对应 Ability | ⭐⭐⭐ |
| `SetXxxAbility(ability)` | 注入各模块 Ability 实例 | ⭐⭐⭐ |

---

## 5.4 模块依赖方向

### 5.4.1 依赖关系图

```
                    ┌─────────────────────┐
                    │  TelephonyDataShare │
                    │      StubImpl       │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   SimAbility    │   │ SmsMmsAbility   │   │ PdpProfileAbility│
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                      │                     │
         └──────────────────────┼─────────────────────┘
                                │
                                ▼
                    ┌─────────────────────┐
                    │    RdbBaseHelper    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    PermissionUtil   │
                    └─────────────────────┘
```

### 5.4.2 依赖说明

| 模块 | 依赖 | 依赖类型 |
|------|------|----------|
| 所有 Ability | `RdbBaseHelper` | 功能依赖 |
| 所有 Ability | `PermissionUtil` | 安全依赖 |
| Stub | 各 Ability | 组合关系 |
| PDP Profile | `ApnEncryptionUtil` | 安全功能 |

---

## 5.5 可替换点

### 5.5.1 数据库层可替换点

| 替换点 | 当前实现 | 可替换方案 |
|--------|----------|------------|
| **RdbStore 创建** | `RdbHelper::GetRdbStore()` | 可注入自定义 RdbStore |
| **数据库路径** | 默认路径 | 可配置自定义路径 |
| **表结构** | 硬编码 Schema | 可动态 Schema |

**代码证据**: `common/include/rdb_base_helper.h`

```cpp
class RdbBaseHelper {
public:
    // 可替换点: 获取 RdbStore 实例
    static std::shared_ptr<NativeRdb::RdbStore> GetRdbStore(
        const RdbStoreConfig &config);
};
```

### 5.5.2 加密层可替换点

| 替换点 | 当前实现 | 可替换方案 |
|--------|----------|------------|
| **APN 加密** | `ApnEncryptionUtil` | 可替换加密算法 |
| **密钥管理** | 硬编码密钥 | 可集成系统 KeyStore |

### 5.5.3 权限层可替换点

| 替换点 | 当前实现 | 可替换方案 |
|--------|----------|------------|
| **权限检查** | `PermissionUtil::CheckPermission()` | 可自定义权限策略 |
| **Token 验证** | `AccessTokenKit` | 可集成其他认证系统 |

---

## 5.6 线程安全标注

### 5.6.1 线程安全类

| 类 | 线程安全级别 | 实现机制 |
|----|--------------|----------|
| `RdbBaseHelper` | 线程安全 | 互斥锁 + 原子操作 |
| `PermissionUtil` | 线程安全 | 无状态 + 只读方法 |
| `DataStorageLogWrapper` | 线程安全 | 线程局部存储 |
| `TelephonyDataShareStubImpl` | 线程安全 | 互斥锁保护 |

### 5.6.2 非线程安全类

| 类 | 使用限制 |
|----|----------|
| 各 Ability 实例 | 应在单线程创建，多线程访问需加锁 |
| RdbPredicates | 应在线程内创建和使用 |

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure](02_Directory_Structure.md) |
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |
| 构建系统 | [06_Build](06_Build.md) |

---

*最后更新: 2024-02-06*

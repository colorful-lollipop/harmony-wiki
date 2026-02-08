# Data Share N-API 参考

## 目的

本文档提供 Data Share 所有 N-API 的完整参考，包括 JS API 名称、参数、返回值、错误码和调用链。

## 适用范围

- 使用 Data Share 进行应用开发的 JS/ArkTS 开发者
- 需要理解 JS 到 Native 调用链的开发者

## 模块概览

| 模块名 | 模块标识 | 说明 |
|--------|----------|------|
| data.dataShare | `data.dataShare` | 主模块，提供 DataShareHelper |
| data.dataSharePredicates | `data.dataSharePredicates` | 谓词构造器 |
| application.DataShareExtensionAbility | `application.DataShareExtensionAbility` | 扩展能力（JS 嵌入） |
| application.DataShareExtensionAbilityContext | `application.DataShareExtensionAbilityContext` | 扩展能力上下文（JS 嵌入） |

**证据**:
- `frameworks/js/napi/dataShare/src/native_datashare_module.cpp:54` - 模块名 `data.dataShare`
- `frameworks/js/napi/dataShare/src/native_datashare_predicates_module.cpp:45` - 模块名 `data.dataSharePredicates`

## data.dataShare 模块

### 模块注册

```cpp
// frameworks/js/napi/dataShare/src/native_datashare_module.cpp:49-65
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "data.dataShare",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

### 全局方法

| JS API | C++ 实现 | 行号 | 说明 |
|--------|----------|------|------|
| `createDataShareHelper` | `NapiDataShareHelper::Napi_CreateDataShareHelper` | 34 | 创建 Helper 实例 |
| `enableSilentProxy` | `NapiDataShareHelper::EnableSilentProxy` | 35 | 启用静默代理 |
| `disableSilentProxy` | `NapiDataShareHelper::DisableSilentProxy` | 36 | 禁用静默代理 |
| `createDataProxyHandle` | `NapiDataProxyHandle::Napi_CreateDataProxyHandle` | 37 | 创建代理句柄 |

### DataShareHelper 类方法

**构造函数**: `NapiDataShareHelper::GetConstructor()` (line 238-273)

| JS 方法 | C++ 实现 | 行号 | 参数 | 返回值 | 同步/异步 |
|---------|----------|------|------|--------|-----------|
| `on` | `Napi_On` | 249 | (eventType, callback) | void | 同步 |
| `off` | `Napi_Off` | 250 | (eventType, callback?) | void | 同步 |
| `insert` | `Napi_Insert` | 251 | (uri, valuesBucket) | number | 异步 |
| `delete` | `Napi_Delete` | 252 | (uri, predicates) | number | 异步 |
| `query` | `Napi_Query` | 253 | (uri, predicates, columns) | ResultSet | 异步 |
| `update` | `Napi_Update` | 254 | (uri, predicates, valuesBucket) | number | 异步 |
| `batchInsert` | `Napi_BatchInsert` | 255 | (uri, valuesBuckets) | number | 异步 |
| `batchUpdate` | `Napi_BatchUpdate` | 256 | (operations) | number | 异步 |
| `normalizeUri` | `Napi_NormalizeUri` | 257 | (uri) | string | 异步 |
| `denormalizeUri` | `Napi_DenormalizeUri` | 258 | (uri) | string | 异步 |
| `notifyChange` | `Napi_NotifyChange` | 259 | (uri) | void | 异步 |
| `addTemplate` | `Napi_AddTemplate` | 260 | (uri, subscriberId, template) | number | 异步 |
| `delTemplate` | `Napi_DelTemplate` | 261 | (uri, subscriberId) | number | 异步 |
| `publish` | `Napi_Publish` | 262 | (data, bundleName) | OperationResult[] | 异步 |
| `getPublishedData` | `Napi_GetPublishedData` | 263 | (bundleName) | Data | 异步 |
| `close` | `Napi_Close` | 264 | () | void | 同步 |

### 参数详情

#### createDataShareHelper

```typescript
function createDataShareHelper(
    context: Context,
    uri: string,
    options?: CreateOptions
): Promise<DataShareHelper>;

interface CreateOptions {
    isProxy?: boolean;    // 仅当 uri scheme 为 datashareproxy 时有效
    waitTime?: number;    // 连接等待时间（秒），默认 2
}
```

**参数校验** (`napi_datashare_helper.cpp:159-184`):
1. 检查系统应用: `IsSystemApp()` (line 41-45)
2. 检查参数个数: 2-4 个参数
3. 检查 context: `GetStageModeContext()` 获取，非空检查
4. 检查 uri: 必须为字符串
5. 检查 options: 必须为对象（可选）

**错误码**:
- `202` - 非系统应用 (`EXCEPTION_SYSTEMAPP_CHECK`)
- `401` - 参数错误 (`EXCEPTION_PARAMETER_CHECK`)
- `15700010` - Helper 未初始化

#### insert

```typescript
function insert(uri: string, valuesBucket: ValuesBucket): Promise<number>;
```

**参数**:
- `uri`: string - 数据路径
- `valuesBucket`: object - 键值对数据

**C++ 实现**: `Napi_Insert()` (line 301-347)

**调用链**:
```
Napi_Insert
  → GetValueBucketObject() 解析 valuesBucket
  → dataShareHelper->Insert() 
    → DataShareHelperImpl::Insert()
      → GeneralController::Insert()
        → DataShareProxy::Insert() [IPC]
          → DataShareStub::CmdInsert()
            → DataShareStubImpl::Insert()
              → JsDataShareExtAbility::Insert()
```

#### query

```typescript
function query(
    uri: string, 
    predicates: DataSharePredicates, 
    columns: string[]
): Promise<ResultSet>;
```

**参数**:
- `uri`: string - 数据路径
- `predicates`: DataSharePredicates - 查询条件
- `columns`: string[] - 查询列名数组

**C++ 实现**: `Napi_Query()` (line 394-442)

#### registerObserver / unregisterObserver

```typescript
function on(type: 'dataChange', uri: string, callback: AsyncCallback<void>): void;
function off(type: 'dataChange', uri: string, callback?: AsyncCallback<void>): void;
```

**订阅类型**:
- `rdbDataChange` - RDB 数据变化
- `publishedDataChange` - 发布数据变化
- `dataChange` - 通用数据变化

**C++ 实现**: `Napi_On()` (line 877-902), `Napi_Off()` (line 904-940)

## data.dataSharePredicates 模块

### DataSharePredicates 类方法

**构造函数**: `DataSharePredicatesProxy::CreateConstructor()` (line 40-87)

| JS 方法 | C++ 实现 | 行号 | 说明 |
|---------|----------|------|------|
| `equalTo` | `EqualTo` | 44 | 等于 |
| `notEqualTo` | `NotEqualTo` | 45 | 不等于 |
| `beginWrap` | `BeginWrap` | 46 | 开始括号 |
| `endWrap` | `EndWrap` | 47 | 结束括号 |
| `or` | `Or` | 48 | 或 |
| `and` | `And` | 49 | 与 |
| `contains` | `Contains` | 50 | 包含 |
| `beginsWith` | `BeginsWith` | 51 | 以...开头 |
| `endsWith` | `EndsWith` | 52 | 以...结尾 |
| `isNull` | `IsNull` | 53 | 为空 |
| `isNotNull` | `IsNotNull` | 54 | 不为空 |
| `like` | `Like` | 55 | SQL LIKE |
| `unlike` | `Unlike` | 56 | SQL NOT LIKE |
| `glob` | `Glob` | 57 | SQL GLOB |
| `between` | `Between` | 58 | 在...之间 |
| `notBetween` | `NotBetween` | 59 | 不在...之间 |
| `greaterThan` | `GreaterThan` | 60 | 大于 |
| `lessThan` | `LessThan` | 61 | 小于 |
| `greaterThanOrEqualTo` | `GreaterThanOrEqualTo` | 62 | 大于等于 |
| `lessThanOrEqualTo` | `LessThanOrEqualTo` | 63 | 小于等于 |
| `orderByAsc` | `OrderByAsc` | 64 | 升序 |
| `orderByDesc` | `OrderByDesc` | 65 | 降序 |
| `distinct` | `Distinct` | 66 | 去重 |
| `limit` | `Limit` | 67 | 限制数量 |
| `groupBy` | `GroupBy` | 68 | 分组 |
| `indexedBy` | `IndexedBy` | 69 | 索引 |
| `in` | `In` | 70 | 在集合中 |
| `notIn` | `NotIn` | 71 | 不在集合中 |
| `prefixKey` | `PrefixKey` | 72 | 前缀键 |
| `inKeys` | `InKeys` | 73 | 键集合 |

### 参数详情

#### equalTo / notEqualTo

```typescript
equalTo(field: string, value: string | number | boolean): DataSharePredicates;
notEqualTo(field: string, value: string | number | boolean): DataSharePredicates;
```

**C++ 实现**: `EqualTo()` / `NotEqualTo()` (line 214-302)

**支持类型**: `napi_number`, `napi_boolean`, `napi_string`

#### between / notBetween

```typescript
between(field: string, low: string, high: string): DataSharePredicates;
notBetween(field: string, low: string, high: string): DataSharePredicates;
```

**C++ 实现**: `Between()` / `NotBetween()` (line 499-537)

**参数**: 3 个参数 (field, low, high)

#### limit

```typescript
limit(total: number, offset?: number): DataSharePredicates;
```

**C++ 实现**: `Limit()` (line 740-758)

**参数**: 
- `total`: number - 限制数量
- `offset`: number - 偏移量（可选）

## 错误码参考

### JS 层错误码

**文件**: `frameworks/js/napi/common/include/datashare_error.h`

| 错误码 | 值 | 说明 | 触发场景 |
|--------|-----|------|----------|
| `EXCEPTION_PARAMETER_CHECK` | 401 | 参数检查错误 | 参数类型/个数错误 |
| `EXCEPTION_INNER` | 15700000 | 内部错误 | 内部异常 |
| `EXCEPTION_HELPER_UNINITIALIZED` | 15700010 | Helper 未初始化 | 未调用 createDataShareHelper |
| `EXCEPTION_URI_NOT_EXIST` | 15700011 | URI 不存在 | 无效的 URI |
| `EXCEPTION_DATA_AREA_NOT_EXIST` | 15700012 | 数据区域不存在 | 数据区域无效 |
| `EXCEPTION_HELPER_CLOSED` | 15700013 | Helper 已关闭 | Helper 已释放 |
| `EXCEPTION_PROXY_PARAMETER_CHECK` | 15700014 | 代理参数检查错误 | 代理参数错误 |

### Native 层错误码

**文件**: `interfaces/inner_api/common/include/datashare_errno.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_OK` | 0 | 成功 |
| `E_ERROR` | 1001 | 通用异常 |
| `E_DATASHARE_PERMISSION_DENIED` | 1080 | 权限被拒绝 |
| `E_EMPTY_URI` | 1081 | URI 为空 |
| `E_NOT_IN_TRUSTS` | 1082 | 不在信任列表 |
| `E_VERIFY_FAILED` | 1079 | 验证失败 |

完整错误码列表请参考 [错误码附录](appendix/Error_Codes.md)。

## 调用链示例

### Insert 操作完整调用链

```
[JS Layer]
  helper.insert(uri, valuesBucket)
    │
    ▼
[NAPI Layer]
  NapiDataShareHelper::Napi_Insert()
    → DataShareJSUtils::GetValueBucketObject()  // 解析 ValuesBucket
    │
    ▼
[Native Consumer Layer]
  DataShareHelperImpl::Insert()
    → GeneralControllerProviderImpl::Insert()  // 或 ServiceImpl
      → DataShareProxy::Insert()
        │
        ▼
[IPC Layer]
        SendRequest(CMD_INSERT)
        │
        ▼
[Native Provider Layer]
  DataShareStub::OnRemoteRequest()
    → DataShareStub::CmdInsert()
      → DataShareStubImpl::Insert()
        → DataShareUvQueue::JsSyncCall()
          │
          ▼
[JS Extension Layer]
          JsDataShareExtAbility::Insert()
```

## 权限要求

### createDataShareHelper

- **系统应用**: 无特殊权限要求
- **普通应用**: 需要目标 URI 对应的读取/写入权限

### 各操作权限

| 操作 | 所需权限 | 检查位置 |
|------|----------|----------|
| insert | writePermission | `DataShareStubImpl::CheckCallingPermission()` |
| update | writePermission | `DataShareStubImpl::CheckCallingPermission()` |
| delete | writePermission | `DataShareStubImpl::CheckCallingPermission()` |
| query | readPermission | `DataShareStubImpl::CheckCallingPermission()` |
| registerObserver | readPermission | `DataShareStubImpl::CheckCallingPermission()` |

**证据**: `frameworks/native/provider/src/datashare_stub_impl.cpp:55-64`

## 关键结论

1. **4 个 N-API 模块**: data.dataShare（主模块）、data.dataSharePredicates（谓词）、以及 2 个 Extension 模块
2. **DataShareHelper 提供 16 个实例方法**: 覆盖 CRUD、批量操作、URI 处理、观察者、模板等
3. **Predicates 提供 30 个方法**: 支持复杂的查询条件构造
4. **异步为主**: 除 on/off/close 外，大多数操作都是异步的（返回 Promise）
5. **系统应用限制**: createDataShareHelper 要求系统应用（检查 fullTokenId）
6. **完整权限检查**: 所有操作都有权限校验，使用 AccessTokenKit

## 相关文档

- [架构设计](05_Architecture.md) - N-API 层架构详解
- [错误码附录](appendix/Error_Codes.md) - 完整错误码列表
- [内部 API](06_Inner_API.md) - 底层 C++ 接口

# 附录：错误码速查

## JS 层错误码

**文件**: `frameworks/js/napi/common/include/datashare_error.h`

| 错误码 | 值 | 说明 | 常见触发场景 |
|--------|-----|------|-------------|
| `E_OK` | 0 | 成功 | 操作成功完成 |
| `EXCEPTION_PARAMETER_CHECK` | 401 | 参数检查错误 | 参数类型/个数错误 |
| `EXCEPTION_INNER` | 15700000 | 内部错误 | 内部异常 |
| `EXCEPTION_HELPER_UNINITIALIZED` | 15700010 | Helper 未初始化 | 未调用 createDataShareHelper |
| `EXCEPTION_URI_NOT_EXIST` | 15700011 | URI 不存在 | 无效的 URI |
| `EXCEPTION_DATA_AREA_NOT_EXIST` | 15700012 | 数据区域不存在 | 数据区域无效 |
| `EXCEPTION_HELPER_CLOSED` | 15700013 | Helper 已关闭 | Helper 已释放 |
| `EXCEPTION_PROXY_PARAMETER_CHECK` | 15700014 | 代理参数检查错误 | 代理参数错误 |

## Native 层错误码

**文件**: `interfaces/inner_api/common/include/datashare_errno.h`

### 基础错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `DATA_SHARE_ERROR` | -1 | 通用错误 |
| `E_OK` | 0 | 成功 |
| `E_ERROR` | 1001 | 通用异常 |

### 注册相关

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_REGISTERED_REPEATED` | 1002 | 重复注册 |
| `E_UNREGISTERED_EMPTY` | 1003 | 未注册 |

### 数据操作错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_INVALID_STATEMENT` | 1007 | 无效语句 |
| `E_INVALID_COLUMN_INDEX` | 1008 | 无效列索引 |
| `E_INVALID_OBJECT_TYPE` | 1020 | 无效对象类型 |
| `E_INVALID_PARCEL` | 1042 | 无效 Parcel |
| `E_VERSION_NOT_NEWER` | 1045 | 版本不是最新 |
| `E_TEMPLATE_NOT_EXIST` | 1046 | 模板不存在 |
| `E_SUBSCRIBER_NOT_EXIST` | 1047 | 订阅者不存在 |
| `E_URI_NOT_EXIST` | 1048 | URI 不存在 |
| `E_BUNDLE_NAME_NOT_EXIST` | 1049 | Bundle 名不存在 |

### 系统状态错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_BMS_NOT_READY` | 1050 | BMS 未就绪 |
| `E_METADATA_NOT_EXISTS` | 1051 | 元数据不存在 |
| `E_SILENT_PROXY_DISABLE` | 1052 | 静默代理已禁用 |
| `E_TOKEN_EMPTY` | 1053 | Token 为空 |
| `E_EXT_URI_INVALID` | 1054 | 扩展 URI 无效 |
| `E_DATA_SHARE_NOT_READY` | 1055 | DataShare 未就绪 |
| `E_DB_ERROR` | 1056 | 数据库错误 |
| `E_DATA_SUPPLIER_ERROR` | 1057 | 数据提供者错误 |

### 序列化错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_MARSHAL_ERROR` | 1058 | 序列化错误 |
| `E_UNMARSHAL_ERROR` | 1059 | 反序列化错误 |
| `E_WRITE_TO_PARCE_ERROR` | 1060 | 写入 Parcel 错误 |

### 资源错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_RESULTSET_BUSY` | 1061 | ResultSet 忙 |
| `E_APPINDEX_INVALID` | 1062 | AppIndex 无效 |
| `E_NULL_OBSERVER` | 1063 | Observer 为空 |
| `E_HELPER_DIED` | 1064 | Helper 已销毁 |
| `E_DATA_OBS_NOT_READY` | 1065 | DataObs 未就绪 |
| `E_PROVIDER_NOT_CONNECTED` | 1066 | Provider 未连接 |
| `E_PROVIDER_CONN_NULL` | 1067 | Provider 连接为空 |

### 权限与验证错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_INVALID_USER_ID` | 1068 | 无效用户 ID |
| `E_NOT_SYSTEM_APP` | 1069 | 非系统应用 |
| `E_NOT_DATASHARE_EXTENSION` | 1077 | 非 DataShare 扩展 |
| `E_DATASHARE_INVALID_URI` | 1078 | 无效 URI |
| `E_VERIFY_FAILED` | 1079 | 验证失败 |
| `E_DATASHARE_PERMISSION_DENIED` | 1080 | 权限拒绝 |
| `E_EMPTY_URI` | 1081 | URI 为空 |
| `E_NOT_IN_TRUSTS` | 1082 | 不在信任列表 |
| `E_GET_CALLER_NAME_FAILED` | 1083 | 获取调用者名失败 |
| `E_NULL_OBSERVER_CLIENT` | 1084 | Observer 客户端为空 |

### 运行时错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_REGISTER_ERROR` | 1070 | 注册错误 |
| `E_NOTIFYCHANGE_ERROR` | 1071 | 通知变更错误 |
| `E_TIMEOUT_ERROR` | 1072 | 超时错误 |
| `E_ERROR_OVER_LIMIT_TASK` | 1073 | 任务超限 |
| `E_EXECUTOR_POOL_IS_NULL` | 1074 | 执行器池为空 |
| `E_NOT_HAP` | 1075 | 非 HAP |
| `E_GET_BUNDLEINFO_FAILED` | 1076 | 获取 BundleInfo 失败 |

### 类型与字段错误

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_UNIMPLEMENT` | 1085 | 未实现 |
| `E_DATASHARE_TYPE` | 1086 | DataShare 类型错误 |
| `E_FIELD_ILLEGAL` | 1087 | 字段非法 |
| `E_FIELD_INVALID` | 1088 | 字段无效 |

## 错误码映射

### JS 错误 → Native 错误

| JS 错误码 | 对应的 Native 错误 |
|-----------|-------------------|
| EXCEPTION_PARAMETER_CHECK (401) | E_DATASHARE_INVALID_URI, E_EMPTY_URI |
| EXCEPTION_HELPER_UNINITIALIZED (15700010) | E_PROVIDER_NOT_CONNECTED |
| EXCEPTION_URI_NOT_EXIST (15700011) | E_URI_NOT_EXIST, E_BUNDLE_NAME_NOT_EXIST |
| EXCEPTION_DATA_AREA_NOT_EXIST (15700012) | E_METADATA_NOT_EXISTS |

## 错误处理建议

### 常见错误处理

```javascript
// 处理权限错误
try {
    await helper.insert(uri, values);
} catch (error) {
    if (error.code === 15700014) {
        // 代理参数错误，检查 URI 格式
        console.error("Proxy parameter error, check URI format");
    } else if (error.code === 15700011) {
        // URI 不存在，检查 Provider 是否正确配置
        console.error("URI not exist, check Provider configuration");
    }
}
```

```cpp
// Native 层错误处理
auto [errCode, helper] = OHOS::DataShare::DataShareHelper::Create(token, uri, "", 2);
if (errCode == OHOS::DataShare::E_NOT_SYSTEM_APP) {
    // 非系统应用，需要申请权限
    HandlePermissionDenied();
} else if (errCode == OHOS::DataShare::E_URI_NOT_EXIST) {
    // URI 不存在
    HandleUriNotExist();
}
```

## 错误码查询表

| 场景 | 可能的错误码 |
|------|-------------|
| 创建 Helper 失败 | E_NOT_SYSTEM_APP, E_URI_NOT_EXIST, E_BUNDLE_NAME_NOT_EXIST |
| 插入失败 | E_DATASHARE_PERMISSION_DENIED, E_DB_ERROR, E_DATA_SUPPLIER_ERROR |
| 查询失败 | E_DATASHARE_PERMISSION_DENIED, E_RESULTSET_BUSY, E_INVALID_COLUMN_INDEX |
| 注册观察者失败 | E_DATASHARE_PERMISSION_DENIED, E_REGISTERED_REPEATED, E_NULL_OBSERVER |
| 文件操作失败 | E_DATASHARE_PERMISSION_DENIED, E_VERIFY_FAILED |
| 超时 | E_TIMEOUT_ERROR, E_ERROR_OVER_LIMIT_TASK |
| IPC 错误 | E_INVALID_PARCEL, E_MARSHAL_ERROR, E_UNMARSHAL_ERROR |

## 相关文档

- [N-API 参考](../04_NAPI_Reference.md) - JS API 错误码说明
- [内部 API](../06_Inner_API.md) - Native API 错误码说明
- [安全分析](../08_Security_Analysis.md) - 权限相关错误

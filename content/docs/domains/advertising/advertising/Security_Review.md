# 安全风险评审

## 威胁模型

### 外部输入

| 输入源 | 数据类型 | 信任边界 |
|--------|----------|----------|
| JS 应用调用 | N-API 参数 | 受限 |
| 配置文件 | JSON 文件 | 系统可控 |
| IPC 数据 | Parcel 序列化 | SA 内部 |
| CommonEvent | 事件回调 | 受限 |

### 敏感操作

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| IPC 通信 | 中 | 跨进程数据传输 |
| 广告展示 | 中 | UI 渲染与用户交互 |
| 数据收集 | 低 | 用户行为追踪 |
| 配置文件加载 | 低 | 系统路径读取 |

## 攻击面清单

| 攻击面 | 类型 | 位置 |
|--------|------|------|
| N-API 参数 | 输入验证 | `advertising.cpp` |
| 配置文件 | 路径遍历 | `cj_advertising_impl.cpp:87` |
| IPC 数据 | 序列化 | `ad_load_proxy.cpp` |
| OAID 暴露 | 信息泄露 | `advertising.cpp:619` |
| 日志输出 | 信息泄露 | 全模块 |

## 输入验证机制

### 字符串长度限制

**文件**: `advertising.cpp:41-56`

| 字段 | 限制值 | 说明 |
|------|--------|------|
| `MAX_STRING_LENGTH` | 65536 | 最大字符串长度 |
| `STR_MAX_SIZE` | 256 | 短字符串限制 |
| `CUSTOM_DATA_MAX_SIZE` | 1MB | 自定义数据限制 |

**代码证据**:
```cpp
// advertising.cpp:41-56
static const int MAX_STRING_LENGTH = 65536;
static const int32_t STR_MAX_SIZE = 256;
static const int32_t CUSTOM_DATA_MAX_SIZE = 1024 * 1024;

// 行 169-173: 自定义数据超长检查
if (strLen > CUSTOM_DATA_MAX_SIZE) {
    return nullptr;
}
```

### 参数类型校验

**文件**: `advertising.cpp:203-240`

- `napi_has_named_property` 检查属性是否存在
- `napi_typeof` 验证类型
- `napi_get_value_string_utf8` 安全字符串读取

**代码证据**:
```cpp
// advertising.cpp:189-198: 字符串类型检查
NAPI_CALL(env, napi_typeof(env, result, &valuetype));
if (valuetype != napi_string) {
    return nullptr;  // 类型不匹配时返回 null
}
```

### 路径遍历防护

**文件**: `cj_advertising_impl.cpp:87`

```cpp
char realPath[PATH_MAX] = {0};
if (strlen(pathBuff) >= PATH_MAX || realpath(pathBuff, realPath) == nullptr) {
    // 路径验证失败处理
}
```

## 可被利用点

### 1. OAID 敏感信息泄露风险

**风险等级**: 低

**证据位置**: `advertising.cpp:619-622`

```cpp
std::string requestRootString = AdJsonUtil::ToString(requestRoot);
cJSON_ReplaceItemInObject(requestRoot, "oaid", cJSON_CreateString("********-****-****-************"));
std::string requestParam = AdJsonUtil::ToString(requestRoot);
ADS_HILOGD(OHOS::Cloud::ADS_MODULE_JS_NAPI, "requestParam is: %{public}s", requestParam.c_str());
```

**分析**: 
- ✅ 已实施脱敏处理
- ⚠️ 日志仍输出 requestParam，但 OAID 已被替换

**建议**: 
- 审查其他可能的敏感字段
- 考虑增加完整日志脱敏层

### 2. 回调引用管理

**风险等级**: 中

**证据位置**: `advertising.cpp:568-577`

```cpp
bool GetCallbackProperty(napi_env env, napi_value obj, napi_ref &property, int argc)
{
    napi_valuetype valueType = napi_undefined;
    NAPI_CALL_BASE(env, napi_typeof(env, obj, &valueType), false);
    if (valueType != napi_function) {
        return false;  // 仅检查类型，不抛异常
    }
    NAPI_CALL_BASE(env, napi_create_reference(env, obj, argc, &property), false);
    return true;
}
```

**分析**:
- ✅ 类型检查防止错误回调注册
- ⚠️ 未验证回调是否已被释放 (dangling reference)

**建议**: 
- 增加回调生命周期追踪
- 在回调触发前验证 napi_ref 有效性

### 3. 配置文件注入

**风险等级**: 低

**证据位置**: `advertising.cpp:99-123`

```cpp
// 配置文件 JSON 解析后检查
cJSON *cloudServiceBundleName = cJSON_GetObjectItem(root, "providerBundleName");
if (cloudServiceBundleName == nullptr || ... {
    return;  // 字段缺失时静默返回
}
```

**分析**:
- ✅ JSON 解析后进行有效性检查
- ⚠️ 配置缺失时静默失败，可能导致意外行为

**建议**:
- 增加配置文件验证日志
- 考虑使用白名单校验 providerBundleName

### 4. IPC 数据序列化

**风险等级**: 中

**证据位置**: `ad_load_proxy.cpp:43-63`

```cpp
// 序列化时仅写入数据，未验证内容
if (!data.WriteString16(Str8ToStr16(requestData->adRequest))) {
    return ERR_AD_COMMON_AD_WRITE_PARCEL_ERROR;
}
```

**分析**:
- ✅ 使用 MessageParcel 安全序列化
- ⚠️ 未验证 adRequest JSON 内容格式

**建议**:
- 在序列化前增加 JSON 格式验证
- 考虑对敏感字段加密

### 5. CommonEvent 广播

**风险等级**: 低

**证据位置**: `adsservice_extension.js`

**分析**:
- 广告平台通过 CommonEvent 发送状态变化
- ⚠️ 未验证接收者身份

**建议**:
- 使用受保护的 CommonEvent
- 考虑使用目标明确的 IPC 替代

## 安全亮点

| 特性 | 说明 | 证据 |
|------|------|------|
| OAID 脱敏 | 日志输出前脱敏 | `advertising.cpp:620` |
| 路径规范 | 使用 realpath() 防止遍历 | `cj_advertising_impl.cpp:87` |
| CFI 保护 | 编译时启用控制流完整 | BUILD.gn |
| PAC-RET | 分支保护 | BUILD.gn |
| JSON 验证 | cJSON_IsValid() 检查 | `ad_json_util.cpp` |

## 权限要求

### 应用权限

| 权限 | 用途 | 必须 |
|------|------|------|
| `ohos.permission.APP_TRACKING_CONSENT` | 广告跟踪 | 是 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取其他应用信息 | SA 实现需 |

### SA 权限

广告服务 SA (ID: 6104) 需要系统权限配置。

## 安全建议

### 高优先级

1. **回调生命周期管理**
   - 增加 napi_ref 有效性验证
   - 实现回调超时清理机制

2. **配置白名单**
   - providerBundleName 加入白名单校验
   - 防止恶意配置注入

### 中优先级

3. **IPC 数据验证**
   - 序列化前验证 JSON 格式
   - 敏感数据考虑加密传输

4. **日志审计**
   - 增加安全相关日志
   - 脱敏策略统一管理

### 低优先级

5. **CommonEvent 保护**
   - 使用受保护的广播机制
   - 验证事件接收者

## 相关文档

- [架构说明](Architecture.md) - 数据流图
- [N-API 参考](NAPI_Reference.md) - API 安全考量
- [构建配置](Build_Configuration.md) - 编译安全选项

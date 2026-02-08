# N-API 参考

## 概述

font_manager 的 N-API 层负责将 C++ 实现暴露给 ArkTS 调用，使用 Node-API (napi) 标准接口实现 JS 到 C++ 的绑定。

**注册入口**: `interfaces/js/kits/src/font_manager_napi.cpp:25-38`

```cpp
// 命名空间: OHOS::Global::FontManager
static napi_module g_FontResourceModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = FontManagerAddonInit,
    .nm_modname = "fontmanager",  // JS 模块名
    .nm_priv = nullptr,
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void AbilityRegister()
{
    napi_module_register(&g_FontResourceModule);
}
```

## API 清单

### 模块信息

| 属性 | 值 |
|------|-----|
| JS 模块名 | `fontmanager` |
| 命名空间 | 无（全局函数） |
| C++ 实现 | `FontManagerAddon` |

### 导出方法

| JS 方法 | C++ 实现 | 同步/异步 | 返回类型 |
|---------|----------|----------|----------|
| `installFont(fontPath: string)` | `FontManagerAddon::InstallFont` | Promise | `Promise<number>` |
| `uninstallFont(fontName: string)` | `FontManagerAddon::UninstallFont` | Promise | `Promise<number>` |
| `dataMigration(options: DataMigrationListener)` | `FontManagerAddon::DataMigration` | 异步回调 | `number` |

## 详细 API

### installFont

安装指定路径的字体文件。

#### 签名

```typescript
function installFont(fontPath: string): Promise<number>
```

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| fontPath | string | 是 | 字体文件路径（沙箱路径或物理路径） |

#### 返回值

| 类型 | 说明 |
|------|------|
| Promise\<number\> | Promise 回调，返回错误码（0=成功，非0=失败） |

#### 错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 0 | ERR_OK | 成功 |
| 201 | ERR_NO_PERMISSION | 无权限（需 ohos.permission.UPDATE_FONT） |
| 202 | ERR_NOT_SYSTEM_APP | 非系统应用 |
| 401 | ERR_INVALID_PARAM | 参数无效 |
| 31100101 | ERR_FILE_NOT_EXISTS | 字体文件不存在 |
| 31100102 | ERR_FILE_VERIFY_FAIL | 字体文件格式不支持 |
| 31100103 | ERR_COPY_FAIL | 文件复制失败 |
| 31100104 | ERR_INSTALLED_ALRADY | 字体已安装 |
| 31100105 | ERR_MAX_FILE_COUNT | 超过最大安装数量 (200) |
| 31100106 | ERR_INSTALL_FAIL | 安装失败 |

#### 示例

```typescript
import fontmanager from '@ohos.fontmanager';

try {
    const result = await fontmanager.installFont('/data/storage/el2/base/files/myfont.ttf');
    if (result === 0) {
        console.log('字体安装成功');
    } else {
        console.log(`字体安装失败，错误码: ${result}`);
    }
} catch (error) {
    console.error('安装异常:', error);
}
```

#### 实现路径

```
JS → FontManagerAddon::InstallFont()
  → interfaces/js/kits/src/font_manager_addon.cpp:196
  → installFontFunc (lambda)
  → FontManagerKits::GetInstance().InstallFont()
  → service/client/src/font_manager_client.cpp:30
```

### uninstallFont

卸载指定名称的字体。

#### 签名

```typescript
function uninstallFont(fontName: string): Promise<number>
```

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| fontName | string | 是 | 字体全名（full name） |

#### 返回值

| 类型 | 说明 |
|------|------|
| Promise\<number\> | Promise 回调，返回错误码 |

#### 错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 0 | ERR_OK | 成功 |
| 201 | ERR_NO_PERMISSION | 无权限 |
| 202 | ERR_NOT_SYSTEM_APP | 非系统应用 |
| 401 | ERR_INVALID_PARAM | 参数无效 |
| 31100107 | ERR_UNINSTALL_FILE_NOT_EXISTS | 字体文件不存在 |
| 31100108 | ERR_UNINSTALL_REMOVE_FAIL | 文件删除失败 |
| 31100109 | ERR_UNINSTALL_FAIL | 卸载失败 |

#### 示例

```typescript
import fontmanager from '@ohos.fontmanager';

try {
    const result = await fontmanager.uninstallFont('MyCustomFont');
    if (result === 0) {
        console.log('字体卸载成功');
    } else {
        console.log(`字体卸载失败，错误码: ${result}`);
    }
} catch (error) {
    console.error('卸载异常:', error);
}
```

#### 实现路径

```
JS → FontManagerAddon::UninstallFont()
  → interfaces/js/kits/src/font_manager_addon.cpp:238
  → uninstallFontFunc (lambda)
  → FontManagerKits::GetInstance().UninstallFont()
  → service/client/src/font_manager_client.cpp:63
```

### dataMigration

执行字体数据迁移。

#### 签名

```typescript
function dataMigration(options: DataMigrationListener): number
```

#### DataMigrationListener

| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| onHeartBeat | function | 否 | 心跳回调 |
| onProgress | function | 否 | 进度回调 |
| onResult | function | 否 | 结果回调 |

#### 返回值

| 类型 | 说明 |
|------|------|
| number | 立即返回错误码（0=启动成功） |

#### 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 启动成功 |
| 201 | 无权限 |
| 401 | 参数无效 |
| 31100110 | 系统错误 |
| 31100111 | 正在迁移中 |

#### 示例

```typescript
import fontmanager from '@ohos.fontmanager';

const result = fontmanager.dataMigration({
    onHeartBeat: () => {
        console.log('迁移心跳');
    },
    onProgress: (progress) => {
        console.log(`迁移进度: ${progress}%`);
    },
    onResult: (result) => {
        console.log(`迁移完成，结果: ${result}`);
    }
});

if (result !== 0) {
    console.log(`启动迁移失败，错误码: ${result}`);
}
```

#### 实现路径

```
JS → FontManagerAddon::DataMigration()
  → interfaces/js/kits/src/font_manager_addon.cpp:286
  → DataMigrationInner()
  → FontManagerKits::GetInstance().DataMigration()
  → service/client/src/font_manager_client.cpp:74
```

## 内部实现细节

### 参数解析

```cpp
// interfaces/js/kits/src/font_manager_addon.cpp:258-284
std::string FontManagerAddon::GetResNameOrPath(napi_env env, size_t argc, napi_value *argv)
{
    // 1. 检查参数数量
    if (argc == 0 || argv == nullptr) {
        return "";
    }
    // 2. 检查类型是否为字符串
    napi_valuetype valuetype;
    napi_typeof(env, argv[0], &valuetype);
    if (valuetype != napi_string) {
        FONT_LOGE("Invalid param, not string");
        return "";
    }
    // 3. 获取字符串长度
    size_t len = 0;
    napi_get_value_string_utf8(env, argv[0], nullptr, 0, &len);
    // 4. 获取字符串内容
    std::vector<char> buf(len + 1);
    napi_get_value_string_utf8(env, argv[0], buf.data(), len + 1, &len);
    return buf.data();
}
```

### 异步执行模型

```cpp
// 使用 napi_create_async_work 实现 Promise 异步执行
napi_value FontManagerAddon::GetResult(napi_env env, std::unique_ptr<FontNapiCallback> &callback,
    const std::string &name, napi_async_execute_callback execute)
{
    napi_create_promise(env, &callback->deferred_, &result);
    napi_create_async_work(env, nullptr, resource, execute, Complete, 
        static_cast<void*>(callback.get()), &callback->work_);
    napi_queue_async_work_with_qos(env, callback->work_, napi_qos_user_initiated);
    callback.release();
    return result;
}
```

### 错误处理

```cpp
// interfaces/js/kits/src/font_manager_addon.cpp:67-78
napi_value GetCallbackErrorCode(napi_env env, const int32_t errCode, const std::string &errMsg)
{
    napi_value error = nullptr;
    napi_create_object(env, &error);
    napi_value eCode = nullptr;
    napi_value eMsg = nullptr;
    napi_create_int32(env, errCode, &eCode);
    napi_create_string_utf8(env, errMsg.c_str(), NAPI_AUTO_LENGTH, &eMsg);
    napi_set_named_property(env, error, "code", eCode);
    napi_set_named_property(env, error, "message", eMsg);
    return error;
}
```

## 相关文档

- [项目概览](00_Overview.md)
- [架构说明](01_Architecture.md)
- [内部 API](03_Inner_API.md)
- [安全评审](06_Security_Review.md)

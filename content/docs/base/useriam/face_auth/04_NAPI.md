# 对外 N-API

## API 清单

### FaceAuthManager 类

| JS API | 类型 | C++ 实现 | 同步/异步 | 说明 |
|--------|------|----------|----------|------|
| `FaceAuthManager` | 构造函数 | `GetFaceAuthManagerConstructor()` | 同步 | 人脸认证管理器 |
| `setSurfaceId(surfaceId)` | 实例方法 | `SetSurfaceId()` | 同步 | 设置预览画面 |

### ResultCode 常量

| 常量 | 值 | 说明 |
|------|------|------|
| `ResultCode.FAIL` | `12700001` | 服务不可用 |

## 错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| `201` | `OHOS_CHECK_PERMISSION_FAILED` | 权限校验失败 |
| `202` | `OHOS_CHECK_SYSTEM_PERMISSION_FAILED` | 非系统应用 |
| `12700001` | `RESULT_CODE_FAIL` | 服务不可用 |

## API 详情

### FaceAuthManager

**注册位置**: `frameworks/js/napi/src/face_auth_napi.cpp:155-164`

**TypeScript 定义**:

```typescript
class FaceAuthManager {
    constructor();

    // 设置人脸录入时的预览画面
    setSurfaceId(surfaceId: string): void;
}

// 错误码
class ResultCode {
    static readonly FAIL: number = 12700001;
}
```

**使用示例**:

```javascript
import faceAuth from '@ohos.faceAuth';

const manager = new faceAuth.FaceAuthManager();
manager.setSurfaceId('1234567890');
```

### setSurfaceId(surfaceId)

**C++ 实现**: `frameworks/js/napi/src/face_auth_napi.cpp:114-153`

**参数**:

| 参数 | 类型 | 必填 | 校验规则 |
|------|------|------|----------|
| `surfaceId` | `string` | 是 | 长度≤25，非空，数字字符串 |

**参数校验逻辑**:

```cpp
// 1. 检查参数个数
size_t argc = argsOne;  // 必须恰好 1 个参数
if (ret != napi_ok || argc != argsOne) {
    napi_throw(env, GenerateBusinessError(env, RESULT_CODE_FAIL));
    return nullptr;
}

// 2. 检查字符串长度
static constexpr int maxLen = 25;
char buf[maxLen] = { '\0' };
ret = napi_get_value_string_utf8(env, argv, buf, maxLen, &len);
if (ret != napi_ok) {
    napi_throw(env, GenerateBusinessError(env, RESULT_CODE_FAIL));
    return nullptr;
}

// 3. 解析为 uint64_t
std::istringstream surfaceIdStream(strSurfaceId);
uint64_t surfaceId;
surfaceIdStream >> surfaceId;

// 4. 获取 BufferProducer
GetBufferProducerBySurfaceId(surfaceId, bufferProducer)
```

**异常处理**:

| 异常 | 触发条件 |
|------|----------|
| `{ code: 201 }` | 权限校验失败 |
| `{ code: 202 }` | 非系统应用 |
| `{ code: 12700001 }` | 其他失败 |

**调用链**:

```
JS: manager.setSurfaceId('12345')
    │
    ▼
C++: SetSurfaceId(env, info)
    │ 参数校验: 个数、类型、长度
    │
    ▼
C++: GetBufferProducerBySurfaceId(surfaceId)
    │ SurfaceUtils::GetInstance()
    │ Surface::GetProducer()
    │
    ▼
C++: FaceAuthClient::GetInstance().SetBufferProducer(bufferProducer)
    │
    ▼
IPC: FaceAuthProxy::SetBufferProducer()
    │ WriteInterfaceToken()
    │ WriteRemoteObject()
    │ SendRequest(FACE_AUTH_SET_BUFFER_PRODUCER)
    │
    ▼
SA: FaceAuthService::SetBufferProducer()
    │ 权限检查
    │
    ▼
HDI: FaceAuthDriverHdi::SetBufferProducer()
```

## N-API 模块注册

**入口**: `frameworks/js/napi/src/face_auth_napi.cpp:191-201`

```cpp
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module module = {
        .nm_version = 1,
        .nm_filename = nullptr,
        .nm_register_func = ModuleInit,
        .nm_modname = "userIAM.faceAuth",
        .nm_priv = nullptr,
    };
    napi_module_register(&module);
}
```

**模块名**: `@ohos/faceAuth` (JS 中 import faceAuth from '@ohos/faceAuth')

## 头文件依赖

| 文件 | 用途 |
|------|------|
| `napi/native_common.h` | N-API 通用宏 |
| `node_api.h` | N-API 核心 |
| `surface.h` | Surface 引用 |
| `face_auth_client.h` | FaceAuth 客户端 |
| `face_auth_defines.h` | 定义常量 |

## 权限要求

| API | 权限 | 调用方要求 |
|-----|------|-----------|
| `FaceAuthManager` | - | 无 |
| `setSurfaceId` | `ohos.permission.MANAGE_USER_IDM` | 系统应用 |

# 附录B: 错误码参考

## 通用错误码

| 错误码 | 定义位置 | 说明 |
|--------|----------|------|
| `ERR_OK` (0) | `ans_inner_errors.h` | 成功 |
| `ERR_PERMISSION_DENIED` | `ans_inner_errors.h` | 权限拒绝 |
| `ERR_INVALID_PARAM` | `ans_inner_errors.h` | 参数无效 |
| `ERR_DEVICE_NOT_SUPPORTED` | `ans_inner_errors.h` | 设备不支持 |
| `ERR_NOTIFICATION_SEND_FAILED` | `ans_inner_errors.h` | 发送失败 |

## 错误码范围

| 范围 | 说明 |
|------|------|
| 0x0000 - 0x00FF | 通用错误 |
| 0x0100 - 0x01FF | 发布相关 |
| 0x0200 - 0x02FF | 取消相关 |
| 0x0300 - 0x03FF | 订阅相关 |
| 0x0400 - 0x04FF | 通道管理 |
| 0x0500 - 0x05FF | 设置相关 |

**证据**: `frameworks/core/common/include/ans_inner_errors.h`

## 常见错误处理

### 发布失败

```cpp
// frameworks/js/napi/src/publish.cpp
ErrCode Publish(const napi_env &env, const NotificationRequest &request) {
    ErrCode ret = NotificationHelper::PublishNotification(request);
    if (ret != ERR_OK) {
        // 处理错误
        return ret;
    }
    return ERR_OK;
}
```

### 权限检查失败

```cpp
// services/ans/src/access_token_helper.cpp
bool AccessTokenHelper::CheckPermission(const std::string &permission) {
    if (!HasPermission()) {
        return false;
    }
    return true;
}
```

## 错误码定义示例

```cpp
// 错误码定义格式
enum {
    ERR_COMMON_START = 0x00000000,
    ERR_OK = 0x00000000,
    ERR_PERMISSION_DENIED = 0x00000001,
    ERR_INVALID_PARAM = 0x00000002,
    // ...
};
```

**证据**: `frameworks/core/common/src/ans_inner_errors.cpp`

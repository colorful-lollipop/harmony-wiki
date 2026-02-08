# 常见问题

## 构建问题

### Q1: 编译报错 "napi_module_register 未定义"

**问题描述**:
```
error: undefined reference to 'napi_module_register'
```

**原因**: 缺少 napi 依赖

**解决方案**: 确保 `frameworks/js/napi/BUILD.gn` 包含以下依赖：

```gn
external_deps = [
  "napi:ace_napi",
  // ... 其他依赖
]
```

---

### Q2: 编译报错 "undefined reference to 'iface_auth'" 

**问题描述**:
```
error: undefined reference to 'IFaceAuth'
```

**原因**: 缺少 IPC framework 依赖

**解决方案**: 检查调用方的 `BUILD.gn`：

```gn
deps = [
  "//base/useriam/face_auth/frameworks/ipc:faceauth_framework",
]
```

---

### Q3: CFI 检查失败

**问题描述**:
```
CFIShadow: control flow integrity check failed
```

**原因**: 代码跳转到了非法地址

**解决方案**:
1. 检查 `cfi_blocklist.txt` 是否需要更新
2. 确认没有使用函数指针类型转换
3. 查看日志中的 PC 地址定位问题

---

## 运行问题

### Q4: SA 启动失败 (Error 942)

**问题描述**:
```
Failed to get FaceAuthService
SA 942 not found
```

**原因**: SA 未注册或未启动

**排查步骤**:

```bash
# 1. 检查 SA 配置
cat /system/sa_profile/942.json

# 2. 检查 SA 库是否存在
ls -l /system/lib/libfaceauthservice.so

# 3. 查看日志
hilog | grep -E "FaceAuth|FACE_AUTH"
```

**解决方案**:
1. 确保 `faceauthservice` 构建成功
2. 检查 `bundle.json` 中 `fwk_group` 包含 SA target

---

### Q5: setSurfaceId 返回权限错误

**问题描述**:
```
{ code: 202, message: "The caller is not a system application." }
```

**原因**: 调用方不是系统应用

**解决方案**:
1. 确认调用方持有 `ohos.permission.MANAGE_USER_IDM` 权限
2. 确认调用方是系统应用（privileged app）

**权限声明** (`module.json5`):
```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.MANAGE_USER_IDM",
        "usedScene": {
          "abilities": ["EntryAbility"]
        }
      }
    ]
  }
}
```

---

### Q6: surfaceId 无效错误

**问题描述**:
```
{ code: 12700001, message: "The service is unavailable." }
```

**原因**: surfaceId 格式错误或不存在

**排查步骤**:

```cpp
// 1. 检查 surfaceId 格式
// 必须是数字字符串，长度 ≤ 25

// 2. 检查 Surface 是否存在
SurfaceUtils::GetInstance()->GetSurface(surfaceId)
```

**解决方案**:
```javascript
// JS 调用示例
const surfaceId = '123456789';  // 必须是有效的 surface ID
manager.setSurfaceId(surfaceId);
```

---

## 调试问题

### Q7: 如何查看 face_auth 日志

**日志标签**:
- `FACE_AUTH_NAPI` - N-API 层
- `FACE_AUTH_SDK` - 客户端 SDK
- `FACE_AUTH_SA` - 服务端 SA

**日志命令**:

```bash
# 查看所有 face_auth 日志
hilog | grep -E "FACE_AUTH|FaceAuth|faceauth"

# 过滤错误日志
hilog | grep -E "E\s+FACE_AUTH"
```

---

### Q8: 如何调试 IPC 通信

**方法 1: 查看 IPC 序列化日志**

```cpp
// 在 face_auth_proxy.cpp 添加调试日志
IAM_LOGI("SendRequest code=%{public}d", code);
```

**方法 2: 使用 hdc 调试**

```bash
# 抓取 IPC 调用
hdc shell "param get faceauth.*"
```

---

### Q9: 如何验证 HDI 连接

**检查 HDI 服务**:
```bash
# 查看 HDI 驱动加载
ls /dev/hdf/

# 检查 face auth HDI
ls /dev/hdf/face_auth_*
```

---

## 性能问题

### Q10: setSurfaceId 调用延迟高

**可能原因**:
1. SurfaceId 对应的 Surface 不存在
2. IPC 跨进程开销
3. 权限校验耗时

**优化建议**:
1. 缓存 SurfaceId 解析结果
2. 批量设置 SurfaceId
3. 检查权限预校验

---

## 移植问题

### Q11: 如何适配新的设备厂商 HDI

**步骤**:

1. **实现 HDI 接口**
   
   实现 `drivers/interface/face_auth V2_0` 定义的接口

2. **注册 HDI 服务**
   
   在设备厂商的 `drivers_peripheral` 中注册

3. **配置依赖**
   
   确保 `face_auth` 构建时链接到正确的 HDI stub

---

### Q12: 如何禁用 face_auth 模块

**全局开关**: `face_auth.gni`

```gn
declare_args() {
  face_auth_enabled = false
}
```

**注意**: 禁用后相关 SA 和 N-API 将不可用

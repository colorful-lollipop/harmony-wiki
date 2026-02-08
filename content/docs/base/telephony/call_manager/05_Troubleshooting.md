# 常见问题

**目的**: 收集构建、运行和调试中的常见问题及解决方案

---

## 构建问题

### 问题 1: 编译报错 "napi_module_register 未定义"

**错误信息**:
```
undefined reference to `napi_module_register'
```

**原因**: N-API 头文件未正确包含

**解决方案**:
```bash
# 确保包含正确的头文件路径
hb build -p call_manager --clean
```

**证据**: `frameworks/js/napi/src/native_module.cpp:16`

### 问题 2: 缺少依赖库

**错误信息**:
```
cannot find -lphonenumber_standard
```

**原因**: libphonenumber 依赖未安装

**解决方案**:
```bash
# 检查依赖配置
cat bundle.json | grep libphonenumber

# 安装依赖
hb set
hb build
```

**证据**: `BUILD.gn:35`

---

## 运行问题

### 问题 1: SA 启动失败 (4005)

**错误日志**:
```
Failed to start call manager service
```

**排查步骤**:
1. 检查 SA 配置
   ```bash
   cat sa_profile/4005.json
   ```
2. 检查库文件是否存在
   ```bash
   ls -l out/.../libtel_call_manager.z.so
   ```
3. 检查权限
   ```bash
   hidumper -sa 4005
   ```

### 问题 2: 权限被拒绝 (Error 201)

**错误信息**:
```
BusinessError 201: Permission denied
```

**原因**: 应用未声明所需权限

**解决方案**:
1. 在 `module.json5` 中声明权限
   ```json
   {
     "requestPermissions": [
       {
         "name": "ohos.permission.PLACE_CALL"
       }
     ]
   }
   ```
2. 申请运行时权限

**证据**: `services/call_manager_service.cpp:62-68`

### 问题 3: 参数错误 (Error 401/8300001)

**错误信息**:
```
BusinessError 401: Parameter error
BusinessError 8300001: Invalid parameter value
```

**常见原因**:
- phoneNumber 为空
- callId 不存在
- slotId 超出范围

**解决方案**:
```typescript
// 检查参数有效性
if (!phoneNumber || phoneNumber.length === 0) {
    console.error("phoneNumber cannot be empty");
    return;
}
```

---

## 调试方法

### 日志查看

```bash
# 开启完整日志
hilog | grep -E "CallManager|Telephony"

# 过滤错误日志
hilog | grep -E "ERROR|LOGE"
```

### 调试技巧

#### 1. 权限调试

```cpp
// 在代码中添加权限检查日志
if (!TelephonyPermission::CheckPermission(OHOS_PERMISSION_PLACE_CALL)) {
    TELEPHONY_LOGE("PLACE_CALL permission check failed!");
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

#### 2. IPC 调试

```bash
# 查看 IPC 调用
ipcperf dump

# 检查 SA 状态
hidumper -sa 4005 -a
```

#### 3. 状态检查

```typescript
// 检查通话状态
call.getCallState((err, state) => {
    console.log(`Call state: ${state}`);
});
```

---

## 常见错误码速查

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 201 | 权限被拒 | 声明并申请权限 |
| 202 | 非系统应用 | 使用系统 API |
| 401 | 参数错误 | 检查参数有效性 |
| 8300001 | 无效参数值 | 检查参数范围 |
| 8300002 | 服务连接失败 | 检查 SA 状态 |
| 8300003 | 系统内部错误 | 查看日志 |
| 8300005 | 飞行模式 | 关闭飞行模式 |
| 8300006 | 网络不可用 | 检查网络连接 |

**证据**: `interfaces/kits/js/@ohos.telephony.call.d.ts:97-106`

---

## 相关文档

- [API 参考](02_API_Reference.md)
- [安全评审](04_Security_Review.md)

# 06_Troubleshooting - 问题排查

> 常见问题、调试方法与日志分析

---

## 1. 日志系统

### 1.1 日志标签

| 标签 | 文件 | 说明 |
|------|------|------|
| CDA_NAPI | `frameworks/js/napi/src/*.cpp` | N-API层日志 |
| CDA_SA | `services/service_entry/src/*.cpp` | SystemAbility日志 |
| CDA_CLIENT | `frameworks/native/client/src/*.cpp` | 客户端日志 |
| COMPANION_DEVICE_AUTH_IPC | `frameworks/native/ipc/` | IPC日志 |

### 1.2 日志级别

```cpp
IAM_LOGE()  // 错误
IAM_LOGW()  // 警告
IAM_LOGI()  // 信息
IAM_LOGD()  // 调试
```

### 1.3 查看日志

```bash
# 查看N-API日志
hilog | grep CDA_NAPI

# 查看服务日志
hilog | grep CDA_SA

# 查看所有伴随设备认证日志
hilog | grep -E "(CDA_|COMPANION)"
```

---

## 2. 错误码速查

### 2.1 内部错误码

从 `common/inc/common_defines.h:28-54`:

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 0 | SUCCESS | 成功 |
| 1 | FAIL | 失败 |
| 2 | GENERAL_ERROR | 一般错误 |
| 3 | CANCELED | 取消 |
| 4 | TIMEOUT | 超时 |
| 5 | TYPE_NOT_SUPPORT | 类型不支持 |
| 6 | TRUST_LEVEL_NOT_SUPPORT | 信任级别不支持 |
| 7 | BUSY | 忙碌 |
| 8 | INVALID_PARAMETERS | 参数无效 |
| 9 | LOCKED | 锁定 |
| 10 | NOT_ENROLLED | 未注册 |
| 20001 | CHECK_PERMISSION_FAILED | 权限检查失败 |
| 20002 | CHECK_SYSTEM_PERMISSION_FAILED | 非系统应用 |
| 20003 | INVALID_BUSINESS_ID | 业务ID无效 |
| 20004 | USER_ID_NOT_FOUND | 用户不存在 |

### 2.2 JS错误码映射

从 `frameworks/js/napi/src/companion_device_auth_napi_helper.cpp:45-53`:

| JS错误码 | 含义 | 内部错误 |
|----------|------|----------|
| 201 | 权限检查失败 | CHECK_PERMISSION_FAILED |
| 202 | 非系统应用 | CHECK_SYSTEM_PERMISSION_FAILED |
| 32600001 | 一般错误 | GENERAL_ERROR |
| 32600002 | 资源不存在 | NOT_ENROLLED / USER_ID_NOT_FOUND |
| 32600003 | 参数无效 | INVALID_BUSINESS_ID |

---

## 3. 常见问题

### 3.1 权限问题

**现象**: API调用返回错误码201

**原因**: 未声明USE_USER_IDM权限

**解决**:
1. 在module.json5中声明权限:
```json
"requestPermissions": [
    {
        "name": "ohos.permission.USE_USER_IDM"
    }
]
```

2. 确保应用为系统应用

**验证**:
```bash
# 检查权限
bm dump -n <bundle_name> | grep USE_USER_IDM
```

### 3.2 非系统应用错误

**现象**: API调用返回错误码202

**原因**: 应用未标记为系统应用

**解决**:
1. 应用必须具有系统签名
2. 在module.json5中声明:
```json
"installationFree": false,
"deliveryWithInstall": true,
```

### 3.3 服务未启动

**现象**: IPC调用超时或失败

**原因**: SA 945未启动

**验证**:
```bash
# 检查服务状态
ps -ef | grep useriam

# 检查SA是否注册
samgr -l | grep 945
```

**解决**:
```bash
# 手动启动服务
sactl start 945
```

### 3.4 回调不触发

**现象**: on/off回调不生效

**排查步骤**:

1. 检查回调是否正确注册:
```javascript
// 正确示例
statusMonitor.onTemplateChange((templates) => {
    console.info('Template changed:', templates);
});
```

2. 检查日志是否有DeathRecipient相关错误:
```bash
hilog | grep DeathRecipient
```

3. 确保StatusMonitor实例未被垃圾回收

### 3.5 跨设备连接失败

**现象**: 无法发现或连接伴随设备

**排查步骤**:

1. 检查SoftBus是否正常工作:
```bash
# 查看设备列表
softbus_tool -l
```

2. 检查网络连接:
```bash
# 检查网络状态
ifconfig
```

3. 检查日志:
```bash
hilog | grep -E "(SoftBus|cross_device)"
```

---

## 4. 调试方法

### 4.1 启用调试日志

```cpp
// 在代码中添加
IAM_LOGD("Debug: value=%{public}d", value);
```

### 4.2 服务Dump

```bash
# 获取服务状态信息
sactl dump 945
```

### 4.3 抓包分析

跨设备通信抓包:
```bash
# 在SoftBus层抓包
tcpdump -i any -w companion_device_auth.pcap
```

---

## 5. 构建问题

### 5.1 编译失败

**错误**: `undefined reference to ...`

**解决**:
```bash
# 清理并重新构建
hb clean
hb build //base/useriam/companion_device_auth/...
```

### 5.2 Rust编译错误

**错误**: `cargo build failed`

**解决**:
```bash
# 更新Rust工具链
rustup update

# 重新构建Rust部分
hb build //base/useriam/companion_device_auth/services/external_adapters/security_command_adapter:companiondeviceauthservice_rs
```

### 5.3 IDL生成失败

**错误**: IDL文件编译错误

**解决**:
```bash
# 单独生成IDL
hb build //base/useriam/companion_device_auth/frameworks/native/ipc:companion_device_auth_ipc_interface
```

---

## 6. 性能问题

### 6.1 内存泄漏

**检查方法**:
```bash
# 查看服务内存使用
dumpsys meminfo useriam
```

**常见泄漏点**:
- 回调未正确注销
- JsRefHolder引用未释放
- IPC对象未清理

### 6.2 响应缓慢

**可能原因**:
- 跨设备网络延迟
- 安全操作计算量大
- 主线程阻塞

**优化建议**:
- 使用异步API
- 添加超时处理
- 检查XCollie超时日志

---

## 7. 测试验证

### 7.1 单元测试

```bash
# 运行所有单元测试
hb build //base/useriam/companion_device_auth/test/unittest:companion_device_auth_unittest

# 运行客户端测试
hb build //base/useriam/companion_device_auth/test/unittest/client:companion_device_auth_client_test
```

### 7.2 模块测试

```bash
# 运行模块测试
hb build //base/useriam/companion_device_auth/test/moduletest:companion_device_auth_moduletest
```

### 7.3 Fuzz测试

```bash
# 运行Fuzz测试
hb build //base/useriam/companion_device_auth/test/fuzztest:companion_device_auth_fuzztest
```

---

## 8. 联系与支持

### 8.1 相关仓库

- [useriam_user_auth_framework](https://gitcode.com/openharmony/useriam_user_auth_framework)
- [useriam_pin_auth](https://gitcode.com/openharmony/useriam_pin_auth)

### 8.2 文档资源

- [OpenHarmony官方文档](https://gitee.com/openharmony/docs)
- [N-API开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/napi)

---

*文档生成时间: 2025-02-06*

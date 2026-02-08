# 常见问题 (FAQ)

## 1. 构建问题

### Q1: 编译报错 "undefined reference to xxx"

**问题**: 链接阶段找不到符号

**可能原因**:
- 依赖的子系统未编译
- GN 依赖配置缺失

**解决步骤**:
```bash
# 1. 检查依赖是否完整
hb build -f

# 2. 检查 BUILD.gn deps 配置
# 文件: frameworks/js/napi/user_auth/BUILD.gn
deps = [
    "../../../native/client:userauth_client",
    "../../../common:user_auth_common"
]
```

**相关文件**:
- `frameworks/js/napi/user_auth/BUILD.gn`
- `bundle.json` dependencies

---

### Q2: 找不到 SA 服务 (901/921/931)

**问题**: IPC 调用返回空对象

**可能原因**:
- SA 服务未启动
- SA profile 配置错误

**解决步骤**:
```bash
# 1. 检查 SA 服务状态
hdc shell sa ps | grep useriam

# 2. 检查 SA profile
cat /system/etc/sa/901.json

# 3. 检查服务注册日志
hdc shell hidumper -a | grep UserAuth
```

**相关文件**:
- `sa_profile/default/{901,921,931}.json`
- `services/ipc/src/user_auth_service.cpp`

---

## 2. 运行问题

### Q3: 认证 API 返回错误码 2 (GENERAL_ERROR)

**问题**: auth() 调用返回 GENERAL_ERROR

**排查步骤**:
```typescript
// 1. 检查认证类型是否支持
let status = userIAM.getAvailableStatus(authType, atl);
if (status !== 0) {
    console.log(`认证不可用，状态码: ${status}`);
}

// 2. 检查是否已注册凭证
let state = userIAM.getEnrolledState(authType);
console.log(`注册状态: ${state.isEnrolled}`);
```

**常见原因**:
- 未注册生物特征
- 认证类型不支持当前设备
- ATL 等级不匹配

---

### Q4: 认证超时 (错误码 4)

**问题**: auth() 调用超时

**可能原因**:
- 用户未完成认证操作
- 认证执行器无响应
- 系统负载过高

**解决建议**:
```typescript
// 设置合理的超时时间
authInstance.on('result', (code) => {
    if (code === 4 /* TIMEOUT */) {
        // 处理超时，可重试
        authInstance.start();
    }
});
```

---

### Q5: 凭证操作失败 (UserIdm)

**问题**: AddCredential/DelCredential 返回错误

**可能原因**:
- 未调用 OpenSession
- Session 已过期
- 权限不足

**解决步骤**:
```typescript
// 正确的凭证操作流程
let client = userIDM.getUserIdmClient();

// 1. 打开会话
await client.openSession(userId);

// 2. 添加凭证
client.addCredential({
    authType: userIAM.AuthType.FACE,
    credType: 1,
    // ...
}, (result) => {
    console.log(`Credential added: ${result}`);
});

// 3. 关闭会话
client.closeSession(userId);
```

---

## 3. 调试问题

### Q6: 如何打印认证调试日志

**问题**: 需要查看详细的认证流程日志

**解决步骤**:
```bash
# 开启 HiTrace 日志
hdc shell hidumper -a | grep -i iam

# 查看用户态日志
hdc shell cat /var/log/iam/iam.log

# 使用 hilog 查看实时日志
hdc shell hilog | grep -E "USERIAM|UserAuth"
```

**日志级别控制**:
- `IAM_LOGI` - 信息
- `IAM_LOGW` - 警告
- `IAM_LOGE` - 错误

---

### Q7: 如何验证 HDI 连接

**问题**: 无法与 TEE 通信

**解决步骤**:
```bash
# 1. 检查 HDI 服务状态
hdc shell hdcd status

# 2. 查看 HDI 日志
hdc shell hdcdump

# 3. 检查驱动加载
hdc shell lsmod | grep user_auth
```

---

### Q8: 远程认证问题

**问题**: 跨设备认证失败

**排查步骤**:
```bash
# 1. 检查设备发现
hdc shell devicemanager discover

# 2. 检查认证策略
cat /data/iam/remote_auth_policy.json

# 3. 查看远程认证日志
hdc shell hilog | grep RemoteAuth
```

---

## 4. 安全问题

### Q9: 如何检查 Token 有效性

**问题**: verifyAuthToken() 返回错误

**代码示例**:
```typescript
import userAccessCtrl from '@ohos.userIAM.userAccessCtrl';

let result = userAccessCtrl.verifyAuthToken(token);
if (result !== 0) {
    console.log(`Token 无效，错误码: ${result}`);
    // 错误码:
    // 1 - 无效 Token
    // 2 - Token 已过期
    // 3 - 权限不足
}
```

---

### Q10: 如何限制应用的认证权限

**问题**: 需要控制第三方应用的认证能力

**解决方案**:
1. 使用 `AuthTrustLevel` 限制最低信任等级
2. 在系统策略中配置应用白名单
3. 使用 `BundleName` 验证调用者身份

```typescript
// 系统内部使用
let authInstance = userIAM.getAuthInstance(
    userIAM.AuthType.FACE,
    userIAM.AuthTrustLevel.ATL3,  // 要求较高信任等级
    bundleName  // 可选：限制调用方
);
```

---

## 5. 性能问题

### Q11: 认证响应慢

**问题**: auth() 调用延迟高

**排查建议**:
```bash
# 1. 检查 CPU 负载
hdc shell top -n 1

# 2. 检查内存使用
hdc shell cat /proc/meminfo

# 3. 检查 I/O
hdc shell iostat
```

**优化建议**:
- 减少同时进行的认证操作
- 使用可复用认证结果 (queryReusableAuthResult)
- 合理设置认证超时时间

---

### Q12: 内存占用过高

**问题**: 认证模块内存占用大

**排查步骤**:
```bash
# 查看进程内存
hdc shell cat /proc/$(pidof useriam)/status | grep VmRSS

# 查看内存映射
hdc shell cat /proc/$(pidof useriam)/maps
```

**常见原因**:
- 未释放认证实例
- 回调中创建了大对象
- 缓存未清理

---

## 6. 问题定位路径

### 日志文件

| 路径 | 说明 |
|------|------|
| `/var/log/iam/` | IAM 模块日志 |
| `/data/iam/` | 运行时数据 |
| `/system/etc/sa/` | SA 配置 |

### 调试命令

| 命令 | 用途 |
|------|------|
| `hdc shell sa ps` | 查看 SA 进程 |
| `hdc shell hidumper -a` | 系统转储 |
| `hdc shell hilog` | 日志查看 |

---

## 7. 相关文档链接

- [N-API 接口](02_NAPI.md)
- [Inner API](03_InnerAPI.md)
- [架构说明](01_Architecture.md)
- [安全评审](05_Security.md)

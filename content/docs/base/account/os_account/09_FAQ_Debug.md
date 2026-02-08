# 09_FAQ_Debug.md

# OpenHarmony os_account 常见问题与调试指南

> 本文档收集 os_account 子系统的常见问题和调试方法。

---

## 1. 构建问题

### Q1: 编译失败 - 找不到头文件

**问题**:
```
fatal error: 'account_log_wrapper.h' file not found
```

**原因**: 头文件搜索路径未正确配置。

**解决方案**:
```bash
# 清理并重新构建
cd /path/to/openharmony
rm -rf out/product_name/accountmgr
./build.sh --product-name product_name --build-target os_account
```

**排查步骤**:
1. 检查 `os_account.gni` 中 `common_path` 是否正确
2. 检查 `include_dirs` 是否包含 `common/log/include`
3. 检查 GN 依赖是否正确

---

### Q2: 链接失败 - 找不到符号

**问题**:
```
undefined reference to 'AccountPermissionManager::VerifyPermission'
```

**原因**: 依赖的库未正确链接。

**解决方案**:
1. 检查 `deps` 或 `public_deps` 是否包含 `//base/account/os_account/frameworks/common:account_common`
2. 检查是否正确引用了 `libaccount_common.z.so`

---

### Q3: Feature Flag 未生效

**问题**: 启用了 `os_account_distributed_feature` 但分布式功能不可用。

**解决方案**:
```bash
# 清理构建缓存
rm -rf out/product_name/accountmgr
rm -rf build/build_tmp/product_name

# 重新构建
./build.sh --product-name product_name --build-target os_account \
  --gn-args os_account_distributed_feature=true
```

---

## 2. 运行问题

### Q4: SA 200 启动失败

**问题**:
```
AccountMgrService failed to start
```

**排查步骤**:
```bash
# 1. 检查 SA 注册状态
hisysevent -l | grep accountmgr

# 2. 检查日志
hilog | grep -E "ACCOUNT|accountmgr"

# 3. 检查依赖服务
sm -l | grep -E "200|bundle|ability"
```

**常见原因**:
1. SAMgr 未启动
2. 依赖的库文件不存在
3. 配置文件格式错误

---

### Q5: 账号创建失败

**问题**: 调用 `createOsAccount()` 返回错误。

**排查步骤**:
```javascript
// JS 调试代码
try {
  let accountManager = account.osAccount.getAccountManager();
  let osAccountInfo = await accountManager.createOsAccount("testUser", 1);
  console.log("Account created:", osAccountInfo);
} catch (error) {
  console.error("Error code:", error.code);
  console.error("Error message:", error.message);
}
```

**错误码排查**:

| 错误码 | 说明 | 可能原因 |
|--------|------|----------|
| 401 | 权限拒绝 | 非系统应用调用 |
| 401 | 无效参数 | 参数为空或格式错误 |
| 460 | 服务异常 | 服务未启动或 IPC 失败 |

---

### Q6: 权限校验失败

**问题**: 权限校验返回 `ERR_ACCOUNT_COMMON_PERMISSION_DENIED`。

**排查步骤**:
```bash
# 1. 检查调用者 Token
hidumper -sa 200 -a

# 2. 检查权限配置
cat /system/etc/(accountmgr.cfg | grep permission

# 3. 检查应用权限
aafmt getperm <bundle-name>
```

**常见原因**:
1. 应用未声明所需权限
2. 权限未授予应用
3. Token 验证失败

---

## 3. 调试方法

### 3.1 日志调试

**日志 TAG**: `ACCOUNT`  
**日志域**: `0xD001B00`

```cpp
// 启用调试日志
ACCOUNT_LOGD("Debug message: %{public}d", value);

// 启用信息日志
ACCOUNT_LOGI("Info message: %{public}s", str.c_str());

// 启用错误日志
ACCOUNT_LOGE("Error occurred: %{public}d", errorCode);
```

**日志过滤**:
```bash
hilog | grep -E "0xD001B00|ACCOUNT"
```

---

### 3.2 HiSysEvent 调试

**事件组件**: `ACCOUNT`

```bash
# 监听账号事件
hisysevent -l | grep ACCOUNT

# 查看事件详情
hisysevent -d ACCOUNT
```

---

### 3.3 HiTrace 调试

```cpp
// 开始追踪
auto traceId = HiTraceBegin("CreateOsAccount", HITRACE_FLAG_DEFAULT);

// 结束追踪
HiTraceEnd(traceId);
```

```bash
# 查看追踪
hitrace --trace ACCOUNT
```

---

### 3.4 HiDumper 调试

```bash
# 导出账号服务状态
hidumper -sa 200

# 导出详细信息
hidumper -sa 200 -a

# 导出指定能力
hidumper -sa 200 -dump all
```

---

### 3.5 IPC 调试

```cpp
// 在 IPC 调用前后添加日志
ACCOUNT_LOGI("SendRequest: cmd=%{public}d", cmd);
auto startTime = std::chrono::steady_clock::now();

int32_t result = proxy->SendRequest(cmd, data, reply, option);

auto endTime = std::chrono::steady_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::ms>(endTime - startTime);
ACCOUNT_LOGI("SendRequest: cmd=%{public}d, result=%{public}d, duration=%{public}ld",
    cmd, result, duration.count());
```

---

## 4. 常见错误码

### 4.1 权限相关错误码

| 错误码 | 定义 | 说明 |
|--------|------|------|
| 401 | `ERR_ACCOUNT_COMMON_PERMISSION_DENIED` | 权限拒绝 |
| 401 | `ERR_ACCOUNT_COMMON_NOT_SYSTEM_APP_ERROR` | 非系统应用 |

### 4.2 参数相关错误码

| 错误码 | 定义 | 说明 |
|--------|------|------|
| 401 | `ERR_ACCOUNT_COMMON_INVALID_PARAM` | 无效参数 |
| 401 | `ERR_ACCOUNT_COMMON_INVALID_NAME` | 无效名称 |

### 4.3 账号相关错误码

| 错误码 | 定义 | 说明 |
|--------|------|------|
| 401 | `ERR_ACCOUNT_COMMON_ACCOUNT_NOT_EXIST` | 账号不存在 |
| 401 | `ERR_ACCOUNT_COMMON_ACCOUNT_EXIST` | 账号已存在 |
| 460 | `ERR_ACCOUNT_COMMON_SERVICE_EXCEPTION` | 服务异常 |

---

## 5. 性能调试

### 5.1 性能指标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| `CreateOsAccount` | < 100ms | 账号创建耗时 |
| `QueryOsAccountById` | < 10ms | 账号查询耗时 |
| `ActivateOsAccount` | < 200ms | 账号切换耗时 |

### 5.2 性能调试方法

```cpp
// 使用 PerfStat 统计
#include "perf_stat/include/perf_stat.h"

PerfStat::TimeStat stat("CreateOsAccount");
stat.Start();

// 执行操作
CreateOsAccount(...);

stat.Stop();
ACCOUNT_LOGI("Operation took %{public}lf ms", stat.GetTimeMs());
```

---

## 6. 相关文档

- [概述](./01_Overview.md)
- [N-API 接口](./03_NAPI_Interfaces.md)
- [服务与 IPC](./05_Service_IPC.md)
- [安全机制](./07_Security.md)
- [GN 构建系统](./06_Build_System.md)

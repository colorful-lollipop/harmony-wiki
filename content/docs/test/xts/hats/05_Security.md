# 安全风险评审

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          不可信区域                                      │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  测试用例代码 (GoogleTest)                                        │  │
│  │  - 直接调用 HDI 接口                                              │  │
│  │  - 模拟恶意输入 (Fuzzing)                                         │  │
│  │  - 验证权限检查                                                   │  │
│  └─────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────┤
│                          信任边界                                        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  HDI 服务接口                                                    │  │
│  │  - 接口版本验证                                                   │  │
│  │  - 参数校验                                                       │  │
│  │  - 权限检查                                                       │  │
│  └─────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────┤
│                          敏感区域                                        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  用户认证模块 (UserIAM)                                           │  │
│  │  - 生物特征模板存储                                               │  │
│  │  - 认证令牌管理                                                   │  │
│  │  - 用户数据删除                                                   │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 攻击面清单

| 攻击面 | 类型 | 涉及模块 | 风险等级 |
|--------|------|----------|----------|
| **HDI 接口** | IPC/Cross-Process | 所有模块 | 高 |
| **系统调用** | Syscall | kernel 模块 | 高 |
| **设备节点** | /dev/* | accesstokenid | 中 |
| **Binder IPC** | IPC | hdf/manager | 高 |
| **回调注入** | IPC/Callback | useriam/telephony | 中 |
| **配置文件** | JSON | powermgr | 低 |

---

## 已识别风险点

### 风险 1：AccessToken ID 设备暴露

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/accesstokenid/accesstokenid_test.cpp:33-62`

```cpp
// 设备节点路径暴露
const char dev_accesstokenid[] = "/dev/access_token_id";

// IOCTL 命令暴露
constexpr unsigned char ACCESS_TOKEN_ID_IOCTL_BASE = 'A';
#define ACCESS_TOKENID_GET_TOKENID \
    _IOR(ACCESS_TOKEN_ID_IOCTL_BASE, GET_TOKEN_ID, unsigned long long)
#define ACCESS_TOKENID_SET_TOKENID \
    _IOW(ACCESS_TOKEN_ID_IOCTL_BASE, SET_TOKEN_ID, unsigned long long)
```

| 项目 | 说明 |
|------|------|
| **影响** | 攻击者可读取/修改访问令牌 ID |
| **触发条件** | 打开 `/dev/access_token_id` 并发送 IOCTL |
| **风险等级** | 中高 |
| **修复建议** | 仅限特权进程访问，添加 SELinux 策略 |

---

### 风险 2：UID/GID 特权提升测试暴露

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/syscalls/user/UserApiTest.cpp:223-230`

```cpp
// 权限提升测试用例
HWTEST_F(UserApiTest, SetuidRootChangeUserIDSuccess_0011, Function | MediumTest | Level1)
{
    uid_t uid = getuid();
    int32_t ret = setuid(0);  // 尝试设置 root UID
    EXPECT_EQ(ret, 0);        // 验证成功（测试期望成功）
    uid_t newUid = getuid();
    EXPECT_EQ(newUid, 0);
}
```

| 项目 | 说明 |
|------|------|
| **影响** | 若此代码在生产环境执行导致权限提升 |
| **触发条件** | 以非 root 用户执行测试 |
| **风险等级** | 低（仅测试环境） |
| **修复建议** | 确认测试仅在特权环境中运行 |

---

### 风险 3：Bundle Name 验证绕过测试

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/external_device_manager/drivers_pkg_manager_test/pkg_db_helper_test.cpp:61-249`

```cpp
// 测试使用硬编码的包名
static std::string g_bundleName = "testBundleName";

// 验证流程
int32_t ret = helper->AddOrUpdateRightRecord(g_bundleName, g_Ability, driverInfo);
string bundleName = helper->QueryBundleInfoNames(driverInfo);
EXPECT_EQ("testAbility", bundleName);
```

| 项目 | 说明 |
|------|------|
| **影响** | 验证逻辑可能存在边界条件绕过 |
| **触发条件** | 异常包名格式 |
| **风险等级** | 中 |
| **修复建议** | 增加包名格式白名单校验 |

---

### 风险 4：HDI 回调注入

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/manager/managerServiceTest/service_manager_hdi_test.cpp:173-186`

```cpp
// 传递远程对象给服务端
sptr<IRemoteObject> callback = new IPCObjectStubTest();
OHOS::MessageParcel data;
data.WriteRemoteObject(callback);  // 注入远程对象

int status = sampleService->SendRequest(SAMPLE_SERVICE_CALLBACK, data, reply, option);
```

| 项目 | 说明 |
|------|------|
| **影响** | 恶意回调可注入到服务进程 |
| **触发条件** | 服务未验证回调对象来源 |
| **风险等级** | 高 |
| **修复建议** | 验证远程对象 UID/权限 |

---

### 风险 5：服务死亡回调竞态

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/display/buffer/death/death_test.cpp:47-57`

```cpp
// 使用 system() 杀进程触发死亡回调
system("killall allocator_host");  // 竞态窗口存在

// 在回调中验证
HWTEST_F(DeathTest, SUB_Driver_Display_Buffer_Death_0100, TestSize.Level1)
{
    sptr<IRemoteObject::DeathRecipient> recipient = new BufferDiedRecipient();
    auto ret = displayBuffer_->AddDeathRecipient(recipient);
    EXPECT_EQ(ret, true);
    g_isServiceDead = true;
    system("killall allocator_host");  // 触发死亡通知
}
```

| 项目 | 说明 |
|------|------|
| **影响** | 死亡回调处理不当导致竞态条件 |
| **触发条件** | 服务崩溃与回调注销并发 |
| **风险等级** | 中 |
| **修复建议** | 使用同步原语保护回调生命周期 |

---

### 风险 6：内存分配溢出

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/ai/nnrt/hdi/v2_0/nnrtFunctionTest/src/hdi_device_test.cpp`

```cpp
// 设备 Buffer 分配
int32_t AllocateBuffer(uint32_t size, std::vector<nnabuffer_buffer>& buffer) {
    // size 未验证，可能导致整数溢出
    buffer.resize(size);  // 潜在 DoS
}
```

| 项目 | 说明 |
|------|------|
| **影响** | 恶意 size 导致内存耗尽 |
| **触发条件** | 传入超大 size 值 |
| **风险等级** | 中 |
| **修复建议** | 添加 size 上限检查 |

---

### 风险 7：Parcel 数据溢出

**证据文件**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/useriam/common/src/iam_hat_test.cpp:35-101`

```cpp
// Parcel 数据填充
void FillTestBuffer(Parcel &parcel, void *p, uint32_t len) {
    // len 未验证
    parcel.WriteBuffer(p, len);  // 可能越界
}
```

| 项目 | 说明 |
|------|------|
| **影响** | 恶意 len 导致内存越界 |
| **触发条件** | 传入超大 len 值 |
| **风险等级** | 高 |
| **修复建议** | 验证 len 在合理范围内 |

---

### 风险 8：Fuzzing 导致的异常输入

**证据文件**：所有 useriam 测试文件使用 Parcel Fuzzing

```cpp
// 使用 Parcel 进行模糊测试
void FillTestUint8Vector(Parcel &parcel, std::vector<uint8_t> &data) {
    // 生成随机数据，可能包含特殊值
    for (int i = 0; i < RANDOM_SIZE; i++) {
        data.push_back(rand() % 256);
    }
}
```

| 项目 | 说明 |
|------|------|
| **影响** | 未处理的异常输入导致服务崩溃 |
| **触发条件** | HDI 服务未做完整输入验证 |
| **风险等级** | 中 |
| **修复建议** | HDI 接口层增加参数校验 |

---

## 检查范围声明

### 已检查范围

| 范围 | 深度 | 发现风险数 |
|------|------|-----------|
| kernel 模块 | 完整 | 2 |
| useriam 模块 | 完整 | 3 |
| hdf/manager | 完整 | 2 |
| powermgr 模块 | 完整 | 1 |
| ai/nnrt 模块 | 完整 | 1 |
| telephony 模块 | 完整 | 0 |

### 未检查范围

| 范围 | 原因 | 建议 |
|------|------|------|
| 第三方依赖 (googletest) | 外部代码 | 参考上游安全公告 |
| HDF 框架核心 | 代码不在此仓库 | 参考 HDF 安全文档 |
| 内核驱动 | 涉及内核代码 | 参考内核安全指南 |

---

## 安全最佳实践建议

### 1. 输入验证

```cpp
// 推荐：添加参数校验
int32_t HdiMethod(SomeParam param) {
    if (param.size > MAX_SIZE) {
        return HDF_ERR_INVALID_PARAM;
    }
    // ...
}
```

### 2. 权限检查

```cpp
// 推荐：验证调用者权限
int32_t SensitiveOperation(Request& req) {
    if (!CheckPermission(req.callerUid)) {
        return PERMISSION_DENIED;
    }
    // ...
}
```

### 3. 回调安全

```cpp
// 推荐：验证回调对象
int32_t RegisterCallback(const sptr<IRemoteObject>& callback) {
    if (callback == nullptr) {
        return HDF_ERR_INVALID_PARAM;
    }
    // 验证 callback 权限
    if (!VerifyCallerPermission(callback)) {
        return PERMISSION_DENIED;
    }
    // ...
}
```

### 4. 竞态保护

```cpp
// 推荐：使用锁保护回调生命周期
std::mutex callbackMutex;
std::vector<DeathRecipient*> recipients;

void AddRecipient(DeathRecipient* r) {
    std::lock_guard<std::mutex> lock(callbackMutex);
    recipients.push_back(r);
}

void RemoveRecipient(DeathRecipient* r) {
    std::lock_guard<std::mutex> lock(callbackMutex);
    // 安全的删除操作
}
```

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 概览 | [00_Overview.md](./00_Overview.md) | 项目定位 |
| 架构 | [01_Architecture.md](./01_Architecture.md) | 系统架构 |
| 模块 | [02_Modules.md](./02_Modules.md) | 子系统详解 |
| N-API | [03_N-API.md](./03_N-API.md) | HDI 接口清单 |
| 构建 | [04_Build.md](./04_Build.md) | GN 构建配置 |
| 附录 | [06_Appendix.md](./06_Appendix.md) | 调用链与配置 |

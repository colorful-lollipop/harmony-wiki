# 常见问题（FAQ）

> **目的**：提供构建、运行、调试过程中常见问题的解决方案
> **适用范围**：所有开发者
> **关键结论**：大多数问题可以通过检查日志、权限和配置解决
> **相关文档**：[GN Targets](06_GN_Targets.md) | [编译产物](07_Build_Artifacts.md) | [安全风险评审](08_Security_Review.md)

---

## 构建问题

### 1. 编译失败：找不到 HDI 接口

**症状**：
```
error: no such file or directory: 'drivers_interface_pin_auth'
```

**原因**：
- `drivers_interface_pin_auth` 组件未包含在编译配置中
- 依赖路径配置错误

**解决方案**：
1. 检查 `bundle.json` 中的依赖：
   ```json
   "deps": {
     "components": [
         "drivers_interface_pin_auth"  // 确保此行存在
     ]
   }
   ```

2. 确保 `drivers_interface` 仓库已拉取：
   ```bash
   # 在 OpenHarmony 根目录
   repo sync drivers_interface_pin_auth
   ```

3. 检查 `.repo/manifest.xml` 中的路径配置

**代码证据**：
- `bundle.json:39` - 依赖声明
- `services/BUILD.gn:86` - 外部依赖引用

---

### 2. 编译失败：权限检查相关

**症状**：
```
error: 'AccessTokenKit' was not declared in this scope
```

**原因**：
- `access_token` 组件未包含在编译配置中
- 头文件路径不正确

**解决方案**：
1. 检查 `bundle.json` 中的依赖：
   ```json
   "deps": {
     "components": [
         "access_token"  // 确保此行存在
     ]
   }
   ```

2. 清理构建缓存：
   ```bash
   hb clean
   hb build -f
   ```

**代码证据**：
- `services/sa/src/pin_auth_service.cpp:107` - AccessTokenKit 使用
- `bundle.json:34` - 依赖声明

---

### 3. 编译警告：未使用的变量

**症状**：
```
warning: unused variable 'tokenId' [-Wunused-variable]
```

**原因**：
- 编译器优化级别设置
- 某些变量仅在调试构建中使用

**解决方案**：
- 通常可以忽略，不影响功能
- 如需消除，检查变量是否实际使用

---

### 4. 链接错误：符号未定义

**症状**：
```
error: undefined reference to 'PinAuthService::GetInstance()'
```

**原因**：
- 缺少 `pinauthservice` 目标的链接
- 版本符号映射文件错误

**解决方案**：
1. 检查 BUILD.gn 中的依赖关系：
   ```gn
   deps = [ ":pinauthservice" ]  // 确保添加此依赖
   ```

2. 检查版本符号映射文件（use_musl 时）：
   - `services/pin_auth_service_map`
   - 确保导出的符号正确

**代码证据**：
- `services/BUILD.gn:111-134` - pinauthservice 定义

---

## 运行问题

### 1. SA 启动失败：找不到服务

**症状**：
```
get system ability failed: 941
```

**原因**：
- SA profile 配置错误
- 库文件未正确安装
- 权限问题

**定位步骤**：
1. 检查日志：
   ```bash
   hilog -T PinAuthService
   ```

2. 检查 SA profile 是否存在：
   ```bash
   ls -l /system/etc/sa_profile/941.json
   ```

3. 检查库文件是否安装：
   ```bash
   ls -l /system/lib64/libpinauthservice.z.so
   ```

4. 检查进程是否运行：
   ```bash
   ps -A | grep pinauth
   ```

**解决方案**：
- 如果 SA profile 缺失：重新编译并安装
- 如果库文件缺失：重新编译并推送库文件
- 如果权限问题：检查 SELinux 上下文

**代码证据**：
- `sa_profile/default/941.json:1-13` - SA 配置
- `services/sa/src/pin_auth_service.cpp:40` - SA 注册

---

### 2. 权限拒绝：RegisterInputer 失败

**症状**：
```
RegisterInputer returned false
```

**原因**：
- 应用未持有 `ohos.permission.ACCESS_PIN_AUTH` 权限
- 应用未使用系统签名
- Token ID 获取失败

**定位步骤**：
1. 检查应用权限声明：
   ```json
   // module.json
   "requestPermissions": [
       {
           "name": "ohos.permission.ACCESS_PIN_AUTH",
           "reason": "$string:permission_reason",
           "usedScene": {
               "abilities": ["EntryAbility"]
           }
       }
   ]
   ```

2. 检查日志中的权限检查结果：
   ```bash
   hilog -T PinAuthService | grep -i permission
   ```

3. 检查应用签名：
   ```bash
   # 使用 hdc 工具
   hdc shell
   pm dump <package_name>
   ```

**解决方案**：
- 添加权限到应用配置
- 使用系统签名重新签名应用
- 确保应用签名与 SA profile 中的 APL 匹配

**代码证据**：
- `services/sa/src/pin_auth_service.cpp:107-113` - 权限检查实现
- `sa_profile/dynamic_load/pinauth_sa_profile.cfg` - APL 配置

---

### 3. 认证超时：OnGetData 无响应

**症状**：
```
authentication timeout
```

**原因**：
- Inputer 回调未及时返回
- 死锁或阻塞
- 线程模型问题

**定位步骤**：
1. 检查 Inputer 实现的 `OnGetData` 方法：
   ```cpp
   void OnGetData(int32_t authSubType, 
                 std::vector<uint8_t> challenge, 
                 std::shared_ptr<IInputerData> inputerData) override {
       // 检查是否有阻塞操作
       // 检查是否有死锁
   }
   ```

2. 检查线程日志：
   ```bash
   hilog -T PinAuthService | grep -i thread
   ```

3. 使用 Dump 工具：
   ```bash
   hdc shell hidumper -s 941 -a -z
   ```

**解决方案**：
- 确保 `OnGetData` 快速返回（不阻塞 SA 线程）
- 避免在回调中执行耗时操作
- 使用异步机制处理 UI 更新

**代码证据**：
- `services/modules/executors/src/pin_auth_executor_callback_hdi.cpp:44-66` - HDI 回调
- `interfaces/inner_api/i_inputer.h:42-43` - Inputer 接口

---

### 4. PIN 验证始终失败

**症状**：
```
authentication result: FAIL
```

**原因**：
- HDI 驱动未正确初始化
- TEE/安全芯片通信失败
- PIN 数据格式错误

**定位步骤**：
1. 检查 HDI 驱动日志：
   ```bash
   hilog -T PinAuthDriverHdi
   ```

2. 检查 HDI 服务是否可用：
   ```bash
   hdc shell
   hdc shell hdi_device -l
   ```

3. 检查 scrypt 加密结果：
   ```bash
   hilog -T Scrypt
   ```

**解决方案**：
- 确保 HDI 驱动正确启动（`PinAuthDriverHdi::Start()`）
- 检查 `libpin_auth_proxy_3.0.z.so` 是否安装
- 联系南向厂商确认 TEE 状态

**代码证据**：
- `services/modules/driver/src/pin_auth_driver_hdi.cpp` - HDI 驱动
- `services/sa/src/pin_auth_service.cpp:StartDriverManager()` - 驱动启动

---

## 调试技巧

### 1. 启用详细日志

**方法 1：使用 hilog**

```bash
# 设置日志级别
hdc shell hilog -b D -T PinAuthService

# 查看日志
hilog -T PinAuthService
```

**方法 2：使用 Dump 工具**

```bash
# 获取 SA 的 Dump 信息
hdc shell hidumper -s 941 -a -z

# 获取特定模块的 Dump
hdc shell hidumper -s 941 -a PinAuthManager
```

**代码证据**：
- `common/logs/iam_logger.h` - 日志宏定义

---

### 2. 调试 SA 启动流程

**关键日志点**：

1. SA 注册：
   ```
   [INFO] MakeAndRegisterAbility result: 1
   ```

2. OnStart：
   ```
   [INFO] PinAuthService OnStart
   [INFO] Publish result: 0
   ```

3. 驱动启动：
   ```
   [INFO] StartDriverManager
   [INFO] IDriverManager::Start result: 0
   ```

**文件位置**：
- `services/sa/src/pin_auth_service.cpp:OnStart()` - 启动日志

---

### 3. 调试 IPC 通信

**关键日志点**：

1. RegisterInputer 调用：
   ```
   [INFO] RegisterInputer called, tokenId: 12345678
   ```

2. 权限检查：
   ```
   [INFO] CheckPermission: ohos.permission.ACCESS_PIN_AUTH, result: 0
   ```

3. Inputer 注册：
   ```
   [INFO] PinAuthManager::RegisterInputer, tokenId: 12345678
   ```

**文件位置**：
- `frameworks/ipc/src/pin_auth_stub.cpp:37-57` - IPC Stub 日志

---

### 4. 调试 HDI 回调

**关键日志点**：

1. OnGetData 回调：
   ```
   [INFO] IExecutorCallbackHdi::OnGetData, scheduleId: 100, tokenId: 12345678
   ```

2. Inputer 查找：
   ```
   [INFO] PinAuthManager::GetInputer, tokenId: 12345678, result: success
   ```

3. OnSetData 调用：
   ```
   [INFO] IInputerDataImpl::OnSetData, scheduleId: 100, dataSize: 32
   ```

**文件位置**：
- `services/modules/executors/src/pin_auth_executor_callback_hdi.cpp:44-66` - 回调日志

---

### 5. 使用 GDB 调试

**附加到 useriam 进程**：

```bash
hdc shell
gdb -p $(pidof useriam)
(gdb) break PinAuthService::OnStart
(gdb) continue
```

**附加到 pinauth 进程（动态加载）**：

```bash
hdc shell
gdb -p $(pidof pinauth)
(gdb) break PinAuthService::OnStart
(gdb) continue
```

---

### 6. 使用 addr2line 分析 Crash

**Crash 日志示例**：

```
backtrace:
    #00 pc 00000000001234567  /system/lib64/libpinauthservice.z.so (PinAuthService::RegisterInputer+124)
    #01 pc 00000000002345678  /system/lib64/libpinauthservice.z.so (PinAuthManager::RegisterInputer+56)
```

**分析方法**：

```bash
# 使用 addr2line 查找对应源代码行号
addr2line -e /path/to/libpinauthservice.z.so -f 00000000001234567

# 或使用 objdump
objdump -d /path/to/libpinauthservice.z.so | grep -A 20 1234567
```

---

## 常见错误码

### PinAuth 模块错误码

| 错误码 | 值 | 说明 | 解决方法 |
|---------|-----|------|---------|
| SUCCESS | 0 | 操作成功 | - |
| GENERAL_ERROR | 1 | 通用错误 | 检查日志 |
| INVALID_PARAM | 2 | 参数无效 | 检查输入参数 |
| PERMISSION_DENIED | 3 | 权限拒绝 | 检查权限声明 |
| INPUTER_NOT_REGISTERED | 4 | Inputer 未注册 | 先注册 Inputer |
| HDI_ERROR | 5 | HDI 错误 | 检查 HDI 驱动状态 |

**代码证据**：
- `services/modules/executors/inc/pin_auth_executor_hdi_common.h` - 错误码定义（TODO）

---

## 性能优化建议

### 1. 减少 IPC 调用

**问题**：频繁的 IPC 调用导致性能下降

**建议**：
- 缓存 SA Proxy（不要重复调用 `GetSystemAbility()`）
- 批量处理请求（如适用）

**代码示例**：
```cpp
// 好的做法：缓存 Proxy
static sptr<PinAuthProxy> s_proxy = nullptr;

sptr<PinAuthProxy> GetProxy() {
    if (s_proxy == nullptr) {
        s_proxy = new (std::nothrow) PinAuthProxy();
    }
    return s_proxy;
}

// 不好的做法：每次都调用 GetSystemAbility
sptr<PinAuthProxy> GetProxy() {
    auto saMgr = SystemAbilityManagerClient::GetInstance();
    return saMgr->GetSystemAbility<PinAuthProxy>(941);
}
```

---

### 2. 优化 Death Recipient 处理

**问题**：Death Recipient 触发后未及时清理

**建议**：
- 使用智能指针自动管理生命周期
- 添加重连机制

**代码示例**：
```cpp
class PinAuthDeathRecipient : public IRemoteObject::DeathRecipient {
public:
    void OnRemoteDied(const wptr<IRemoteObject> &object) override {
        // 自动清理
        sptr<IRemoteObject> remote = object.promote();
        if (remote != nullptr) {
            // 清理逻辑
        }
        
        // 自动重连
        ReconnectService();
    }
};
```

---

### 3. 避免 UI 线程阻塞

**问题**：在 Inputer 回调中执行耗时操作阻塞 SA 线程

**建议**：
- 将耗时操作移到异步线程
- 仅在回调中设置标志或触发事件

**代码示例**：
```cpp
void MyInputer::OnGetData(int32_t authSubType, 
                         std::vector<uint8_t> challenge, 
                         std::shared_ptr<IInputerData> inputerData) override {
    // 好的做法：异步显示对话框
    std::thread([authSubType, challenge, inputerData]() {
        // 在 UI 线程中显示对话框
        ShowDialog(authSubType, challenge, inputerData);
    }).detach();
    
    // 不好的做法：在回调中同步等待
    std::string pin = ShowDialogBlocking(authSubType, challenge);
    inputerData->OnSetData(authSubType, std::vector<uint8_t>(pin.begin(), pin.end()));
}
```

---

## 相关资源

### 日志工具

- `hilog` - OpenHarmony 日志系统
- `hidumper` - 系统信息 Dump 工具

### 调试工具

- `gdb` - GNU 调试器
- `addr2line` - 地址转行号工具
- `objdump` - 目标文件反汇编工具

### 文档资源

- [OpenHarmony 调试指南](https://docs.openharmony.cn/docs/application-dev/quick-start/start-overview)
- [System Ability 开发指南](https://docs.openharmony.cn/docs/application-dev/ability/sa-overview)

---

## 代码证据索引

| 问题类型 | 相关文件 |
|---------|----------|
| 编译配置 | `bundle.json`, `pin_auth.gni` |
| SA 注册 | `services/sa/src/pin_auth_service.cpp:40` |
| 权限检查 | `services/sa/src/pin_auth_service.cpp:107-113` |
| Inputer 管理 | `services/modules/inputters/src/pin_auth_manager.cpp` |
| HDI 交互 | `services/modules/driver/src/pin_auth_driver_hdi.cpp` |
| 日志宏 | `common/logs/iam_logger.h` |

---

## 下一步

- 返回文档导航 → [全站导航](SUMMARY.md)

# SAFWK 常见问题

## 构建问题

### Q1: 编译失败，提示找不到依赖

**问题**: 构建时提示 `ninja: error: unknown target 'xxx'` 或找不到依赖

**解决方案**:
```bash
# 1. 检查 bundle.json 中的 target 定义
cat bundle.json | grep -A5 '"build"'

# 2. 使用正确的 target 名称
hb build -p systemabilitymgr/safwk -T //foundation/systemabilitymgr/safwk/services/safwk:sa_main

# 3. 全量同步依赖
hb set
hb build
```

**参考**: [03_Build.md](03_Build.md) - 构建命令

---

### Q2: 特征开关未生效

**问题**: 设置了 `safwk_feature_support_saspawn = true` 但 `libsa_start.z.so` 未构建

**解决方案**:
```bash
# 1. 在产品配置文件中设置
# 修改 <product>/config.gni
safwk_feature_support_saspawn = true

# 2. 清理并重新构建
hb clean
hb build
```

**注意**: `sa_start` 目标仅在 `safwk_feature_support_saspawn = true` 时构建

**参考**: [03_Build.md](03_Build.md) - 特征开关

---

### Q3: Rust 绑定编译失败

**问题**: `libsystem_ability_fwk.so` (Rust) 编译失败

**解决方案**:
```bash
# 1. 检查 Rust 工具链
rustc --version

# 2. 检查 cxx crate
cat interfaces/innerkits/safwk/rust/Cargo.toml | grep cxx

# 3. 清理 Rust 构建缓存
cargo clean
hb build -p systemabilitymgr/safwk --clean
```

**参考**: [03_Build.md](03_Build.md) - Rust 绑定构建

---

## 运行问题

### Q4: SA 启动失败

**问题**: SA 进程启动后立即退出，日志显示注册失败

**日志示例**:
```
E/SAMGR: Register system ability failed: 0x1234, libpath: libxxx.z.so
```

**排查步骤**:
```bash
# 1. 检查 profile 配置
cat /system/profile/xxx.json

# 2. 检查 SA ID 是否冲突
# 查看 foundation_trust.json 中的已注册 SA

# 3. 检查权限
dumpsys bundle --ohos-idl
```

**常见原因**:
- SA ID 已注册
- libpath 路径错误
- 权限不足

**参考**: [00_Overview.md](00_Overview.md) - 开发流程

---

### Q5: GetSystemAbility 返回空

**问题**: `GetSystemAbility()` 返回 `nullptr`

**解决方案**:
```cpp
// 1. 检查 SA 是否已注册
sptr<IRemoteObject> sa = GetSystemAbility(SA_ID);
if (sa == nullptr) {
    // 2. 检查 SA 状态
    auto state = GetAbilityState();
    // 3. 检查权限
    CheckPermission(GET_ABILITY_TRANSACTION);
}
```

**常见原因**:
- SA 未注册
- SA 未启动 (按需启动模式)
- 权限不足

---

### Q6: IPC 调用超时

**问题**: IPC 调用长时间阻塞或超时

**解决方案**:
```bash
# 1. 检查目标 SA 状态
dumpsys ability --<sa_id>

# 2. 检查是否有死锁
# 查看线程栈

# 3. 增加超时时间 (如适用)
MessageOption option(MessageOption::TF_WAIT_TIME, 5000);  // 5s
```

**参考**: [01_Architecture.md](01_Architecture.md) - IPC 框架

---

## 调试问题

### Q7: 如何查看 SA 状态

**解决方案**:
```bash
# 1. 使用 svc 工具
svc list
svc dump <sa_id>

# 2. 使用 hdc 命令
hdc shell
dumpsys ability

# 3. 查看日志
hilog | grep -E "SA|SAFWK|SystemAbility"
```

**参考**: [03_Build.md](03_Build.md) - svc 工具

---

### Q8: 如何调试 SA

**解决方案**:

**1. 增加日志**:
```cpp
// 在关键路径添加日志
HiLog::Info(LABEL, "OnStart called, saId=%{public}d", saId_);
HiLog::Info(LABEL, "Publish result: %{public}d", result);
```

**2. 使用 GDB/LLDB**:
```bash
# 附加到进程
hdc gdb
target remote :1234

# 或启动调试
hdc shell
gdbserver :1234 /system/bin/sa_main
```

**3. 启用 Dump**:
```cpp
void OnDump() override
{
    // 输出调试信息
    HiLog::Info(LABEL, "SA state: %{public}d", abilityState_);
}
```

---

### Q9: 如何排查权限问题

**问题**: 收到权限拒绝错误

**排查步骤**:
```bash
# 1. 检查调用者 token
hdc shell
app getappinfo <calling_pid>

# 2. 检查权限定义
cat etc/profile/foundation_permission_desc.json | grep <permission>

# 3. 检查信任配置
cat etc/profile/foundation_trust.json

# 4. 验证 AccessToken
# 在代码中添加调试日志
uint32_t token = IPCSkeleton::GetCallingTokenID();
HiLog::Info(LABEL, "Calling token: %{public}u", token);
```

**参考**: [04_Security.md](04_Security.md) - 权限控制

---

### Q10: 按需启动不生效

**问题**: 配置了 `run-on-create: false` 但请求时 SA 不启动

**解决方案**:
```bash
# 1. 检查按需启动配置
cat /system/profile/xxx.json | grep run-on-create

# 2. 检查依赖配置
cat /system/profile/xxx.json | grep depend

# 3. 检查 FFRT 任务队列
# 查看 ffrt_handler 相关日志

# 4. 验证触发原因
# 检查 SystemAbilityOnDemandReason
```

**参考**: [01_Architecture.md](01_Architecture.md) - 按需启动机制

---

## 性能问题

### Q11: IPC 开销过大

**问题**: 高频 IPC 调用导致性能问题

**解决方案**:
```cpp
// 1. 使用 ApiCacheManager 缓存结果
ApiCacheManager::GetInstance().AddCacheApi(
    descriptor, apiCode, 60000);  // 60s 过期

// 2. 批量请求
// 合并多个小请求为一个大请求

// 3. 使用异步 IPC
MessageOption option(MessageOption::TF_ASYNC);
```

**参考**: [02_APIs.md](02_APIs.md) - ApiCacheManager

---

### Q12: 内存占用过高

**问题**: SAFWK 或 SA 内存占用持续增长

**排查步骤**:
```bash
# 1. 查看内存使用
hdc shell
meminfo <pid>

# 2. 检查 LRU 缓存
# 查看 api_cache_manager 配置

# 3. 检查资源泄漏
# 重点关注 OnStop 中的清理
```

**解决方案**:
```cpp
// 实现 OnIdle 回调，释放资源
int32_t OnIdle(const SystemAbilityOnDemandReason& idleReason) override
{
    // 清理非必要资源
    ClearCache();
    return 10000;  // 延迟 10s 后卸载
}
```

---

## 移植问题

### Q13: 从旧版本迁移

**问题**: 代码从 OpenHarmony 3.0 迁移到 3.1+

**注意事项**:
```cpp
// 旧版 (3.0)
class MyAbility : public SystemAbility {
public:
    MyAbility() : SystemAbility(SA_ID) {}  // 旧构造
};

// 新版 (3.1+)
class MyAbility : public SystemAbility {
public:
    MyAbility(int32_t saId, bool runOnCreate) 
        : SystemAbility(saId, runOnCreate) {}  // 新构造
};
```

**参考**: `interfaces/innerkits/safwk/system_ability.h` - 构造函数

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 架构设计 | [01_Architecture.md](01_Architecture.md) |
| API 接口 | [02_APIs.md](02_APIs.md) |
| 构建系统 | [03_Build.md](03_Build.md) |
| 安全评审 | [04_Security.md](04_Security.md) |

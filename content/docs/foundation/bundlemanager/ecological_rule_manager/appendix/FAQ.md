# 常见问题（FAQ）

## 构建相关问题

### Q: 如何编译 ecological_rule_manager？

**A**: 在 OpenHarmony 源码根目录执行：

```bash
./build.sh --product-name rk3568 --ccache --build-target ecological_rule_manager
```

**参数说明**:
- `--product-name`: 产品名称，如 rk3568 或 Hi3516D V300
- `--ccache`: 启用编译缓存
- `--build-target`: 指定编译的组件名称

**相关文档**: [04_GN_Build.md](../04_GN_Build.md)

### Q: 编译产物有哪些？安装在哪里？

**A**: 编译产物包括：

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libecologicalrulemgr_service.z.so` | 共享库 | `/usr/lib/` | 主服务共享库 |
| `liberms_client.z.so` | 共享库 | `/usr/lib/` | 客户端 SDK |
| `6105.json` | 配置文件 | `/etc/profile/` | SA 注册配置 |
| `ecological_rule_manager/clientTest` | 可执行文件 | `/usr/bin/` | 单元测试 |

**相关文档**: [04_GN_Build.md](../04_GN_Build.md)

### Q: 编译依赖哪些组件？

**A**: 根据 `bundle.json`，依赖以下组件：

**内部依赖**:
- ability_base
- ability_runtime
- bundle_framework
- access_token
- c_utils
- eventhandler
- hilog
- ipc
- safwk
- samgr

**说明**: 这些组件需要在编译前就已就绪。

**相关文档**: [04_GN_Build.md](../04_GN_Build.md)

### Q: 编译时有哪些 Feature 开关？

**A**: 当前**无编译时 Feature 开关**。`bundle.json` 中 `features` 字段为空：

```json
"features": []
```

如有需要，可后续扩展自定义编译选项。

---

## 运行相关问题

### Q: 服务什么时候启动？

**A**: 服务在**系统启动时自动启动**，原因如下：

1. **SA 配置** (`profile/6105.json`) 中设置了 `"run-on-create": true`
2. 服务运行在 `foundation` 进程（系统核心进程）
3. SA ID 为 6105，由 SystemAbility Framework 管理

**启动流程**:
```
系统启动 → foundation 进程启动 → 加载 6105 SA profile
  → 加载 libecologicalrulemgr_service.z.so
  → 调用 EcologicalRuleMgrService::OnStart()
  → 发布服务到 SA 管理器
  → 服务就绪
```

**相关文档**: [02_Architecture.md](../02_Architecture.md)

### Q: 如何查看服务状态？

**A**: 有以下几种方式：

#### 1. 查看 SA 状态
```bash
# 查看 SA 是否注册
hdc shell hidumper -s 6105

# 或使用 dump 命令（需服务支持）
hdc shell "hidumper -s -a EcologicalRuleManagerService"
```

#### 2. 查看进程
```bash
# 检查 foundation 进程是否运行
ps -A | grep foundation
```

#### 3. 查看日志
```bash
# 实时查看服务日志
hdc shell "hilog -T ERMS_MAIN"
```

**相关文档**: [02_Architecture.md](../02_Architecture.md)

### Q: SA 死亡后如何恢复？

**A**: 服务实现了 **DeathRecipient** 机制，自动处理 SA 死亡：

**实现位置**: `interfaces/innerkits/src/ecological_rule_mgr_service_client.cpp:77-78`

```cpp
class ServiceDeathRecipient : public IRemoteObject::DeathRecipient {
    void OnRemoteDied(const wptr<IRemoteObject> &object) override {
        // 清理服务代理并标记为未连接
        // 下次调用时自动重连
    }
};
```

**恢复流程**:
```
SA 进程崩溃 → DeathRecipient 触发
  → 清理服务代理
  → 标记连接状态为断开
  → 下次调用时检测到断开
  → 自动重新连接并获取新代理
```

**注意**: 如果 SA 反复崩溃，需要检查崩溃日志。

---

## 调试相关问题

### Q: 如何启用详细日志？

**A**: 服务使用 HiLog 模块，日志标签为 `ERMS_MAIN`。

#### 查看日志
```bash
# 实时查看服务日志
hdc shell "hilog -T ERMS_MAIN"

# 查看所有日志
hdc shell "hilog -T ERMS_MAIN | grep EcologicalRuleMgr"
```

#### 日志级别说明

服务代码中使用以下日志宏：
- `LOG_INFO()`: 信息级别（默认显示）
- `LOG_DEBUG()`: 调试级别（需开启详细日志）
- `LOG_WARN()`: 警告级别
- `LOG_ERROR()`: 错误级别

**开启调试日志**:
```bash
# 设置 HiLog 为 DEBUG 级别
hdc shell "hilog -b D"
```

**相关文档**: [02_Architecture.md](../02_Architecture.md)

### Q: 如何查看 IPC 调用？

**A**: 有以下几种方式跟踪 IPC 调用：

#### 1. 使用 HiLog 跟踪
```bash
# 查看 IPC 请求日志
hdc shell "hilog -T ERMS_MAIN | grep 'QueryStartExperience\|QueryFreeInstallExperience'"
```

#### 2. 使用 hidumper 跟踪
```bash
# 查看 IPC 调用统计
hdc shell "hidumper -s 6105 -a -i"
```

#### 3. 使用 BPF 或 strace（高级）
```bash
# 跟踪 binder 调用（需要 root 权限）
hdc shell "strace -p <foundation_pid> -e trace=binder_ioctl"
```

**说明**: `QueryStartExperience` 等接口调用会在日志中打印调用方信息：

```
want bundle name = xxx, uri = yyy
callerInfo bundle name = zzz, callerAppType = 1, targetAppType = 2
```

**相关文档**: [02_Architecture.md](../02_Architecture.md), [05_Security_Review.md](../05_Security_Review.md)

### Q: 如何跟踪接口调用？

**A**: 可以在以下位置添加断点或日志：

#### 1. 客户端入口（Proxy）
**文件**: `interfaces/innerkits/src/ecological_rule_mgr_service_proxy.cpp`

```cpp
// 示例: QueryStartExperience
int32_t EcologicalRuleMgrServiceProxy::QueryStartExperience(...)
{
    // 添加日志
    LOG_INFO("Client: QueryStartExperience called");
    // ...
}
```

#### 2. 服务端入口（Stub）
**文件**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp`

```cpp
// 示例: OnQueryStartExperience
int32_t EcologicalRuleMgrServiceStub::OnQueryStartExperience(...)
{
    // 添加日志
    LOG_INFO("Stub: OnQueryStartExperience called");
    // ...
}
```

#### 3. 服务实现
**文件**: `services/manager/src/ecologic_rule_mgr_service.cpp`

```cpp
// 示例: QueryStartExperience
int32_t EcologicalRuleMgrService::QueryStartExperience(...)
{
    // 当前实现已有日志
    LOG_INFO("want bundle name = %{public}s", bundleName.c_str());
    // ...
}
```

**调用链**:
```
Client → Proxy::QueryStartExperience() 
    → IPC 
    → Stub::OnQueryStartExperience()
    → Service::QueryStartExperience()
```

**相关文档**: [appendix/Callgraphs.md](../appendix/Callgraphs.md)

---

## 安全相关问题

### Q: 谁可以调用这个服务？

**A**: 只有**系统服务**可以调用，普通应用无法访问。

**原因**:
1. 服务运行在 `foundation` 进程（高权限）
2. 没有暴露 N-API JS 接口
3. `VerifySystemApp()` 校验调用方身份

**可调用方**:
- `AbilityManagerService` (系统服务)
- `BundleManagerService` (系统服务)
- `FormManagerService` (系统服务)
- 其他系统服务（需要 Native Token）

**权限检查**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp:233-257`

**相关文档**: [05_Security_Review.md](../05_Security_Review.md)

### Q: 如果权限校验失败会怎样？

**A**: 权限校验失败会返回错误码 `ERR_PERMISSION_DENIED`。

**错误码定义** (`interfaces/innerkits/include/ecological_rule_mgr_service_interface.h`):
```cpp
enum ErrCode {
    ERR_BASE = (-99),
    ERR_FAILED = (-1),
    ERR_PERMISSION_DENIED = (-2),  // 权限拒绝
    ERR_OK = 0,
};
```

**触发场景**:
- 调用方不是系统应用
- 调用方不是 Native Token / Shell Token
- 调用方 UID 不在允许列表

**相关文档**: [05_Security_Review.md](../05_Security_Review.md)

---

## 接口使用相关问题

### Q: 如何在代码中调用这个服务？

**A**: 需要通过 `EcologicalRuleMgrServiceClient` 客户端。

**示例代码**:

```cpp
#include "ecological_rule_mgr_service_client.h"

using namespace OHOS::EcologicalRuleMgrService;

// 1. 获取客户端实例
auto client = EcologicalRuleMgrServiceClient::GetInstance();

// 2. 构造参数
Want want;
want.SetBundle("com.example.atom");
CallerInfo callerInfo;
callerInfo.packageName = "com.example.caller";

// 3. 调用接口
ExperienceRule rule;
int32_t ret = client->QueryStartExperience(want, callerInfo, rule);

// 4. 检查结果
if (ret == ERR_OK) {
    if (rule.isAllow) {
        // 允许，使用原始或替代 Want
    } else {
        // 不允许，处理拒绝逻辑
    }
}
```

**相关文档**: [03_Inner_API.md](../03_Inner_API.md)

### Q: 为什么接口都返回 SUCCESS？

**A**: 当前实现为**空壳（stub）**，实际业务逻辑待实现。

**现状** (`services/manager/src/ecologic_rule_mgr_service.cpp`):
```cpp
int32_t EcologicalRuleMgrService::QueryStartExperience(...)
{
    LOG_INFO("want bundle name = %{public}s", bundleName.c_str());
    return SUCCESS;  // TODO: 实际业务逻辑待实现
}
```

**说明**: 服务框架已完成，规则引擎和管控逻辑需要后续开发实现。

**相关文档**: [05_Security_Review.md](../05_Security_Review.md) - 风险 5

---

## 其他问题

### Q: 如何提交 Bug 或建议？

**A**: 可以通过以下途径：

1. **OpenHarmony Gitee**: 提交 Issue 到 `bundlemanager_ecological_rule_manager` 仓库
2. **社区论坛**: OpenHarmony 开发者社区
3. **内部渠道**: 如你是内部开发人员，请使用内部 Bug 跟踪系统

### Q: 如何参与开发？

**A**: 参考以下步骤：

1. 阅读 [项目概览](../00_Overview.md) 了解项目定位
2. 阅读 [架构设计](../02_Architecture.md) 理解架构
3. 阅读 [Inner API](../03_Inner_API.md) 了解接口规范
4. 查阅 [安全评审](../05_Security_Review.md) 了解安全要求
5. Fork 仓库并提交 Pull Request

---

## 相关文档

- [项目概览](../00_Overview.md)
- [架构设计](../02_Architecture.md)
- [Inner API 文档](../03_Inner_API.md)
- [安全风险评审](../05_Security_Review.md)
- [构建与产物](../04_GN_Build.md)

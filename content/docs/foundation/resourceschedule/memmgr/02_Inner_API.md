# Inner API 文档 (Inner API)

> MemMgr 组件对内 C++ 接口说明

## ⚠️ 重要说明

**本组件不提供 N-API (JS API)**，仅提供 Inner C++ API。

> 这意味着：
> - 没有 JS/TS 接口暴露给应用层
> - 只能由系统级服务 (如 bundlems, abilityms) 调用
> - 需通过 IPC 机制 (`IMemMgr`) 使用

---

## 1. API 清单总表

### 1.1 基础信息

| 属性 | 值 |
|------|-----|
| 接口命名空间 | `OHOS::Memory` |
| 服务端接口 | `IMemMgr` (继承 `IRemoteBroker`) |
| 客户端入口 | `MemMgrClient` (单例模式) |
| SA ID | `1909` |
| IPC 描述符 | `ohos.memory.MemMgr` |

**证据**: `interface/innerkits/include/i_mem_mgr.h:36`

### 1.2 API 列表

| 方法 | 同步/异步 | 条件编译 | 实现位置 | 描述 |
|------|-----------|----------|----------|------|
| `GetBundlePriorityList` | 同步 | - | `mem_mgr_service.cpp` | 获取 Bundle 优先级列表 |
| `NotifyDistDevStatus` | 同步 | - | `mem_mgr_service.cpp` | 通知分布式设备状态 |
| `GetKillLevelOfLmkd` | 同步 | - | `mem_mgr_service.cpp` | 获取 lmkd 查杀级别 |
| `OnWindowVisibilityChanged` | 同步 | - | `mem_mgr_service.cpp` | 窗口可见性变化通知 |
| `GetReclaimPriorityByPid` | 同步 | - | `mem_mgr_service.cpp` | 按 PID 获取回收优先级 |
| `NotifyProcessStateChangedSync` | 同步 | - | `mem_mgr_service.cpp` | 同步通知进程状态变化 |
| `NotifyProcessStateChangedAsync` | 同步 | - | `mem_mgr_service.cpp` | 异步通知进程状态变化 |
| `NotifyProcessStatus` | 同步 | - | `mem_mgr_service.cpp` | 通知进程状态 |
| `SetCritical` | 同步 | - | `mem_mgr_service.cpp` | 设置进程为关键进程 |
| `RegisterActiveApps` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 注册活跃应用 |
| `DeregisterActiveApps` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 注销活跃应用 |
| `SubscribeAppState` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 订阅应用状态 |
| `UnsubscribeAppState` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 取消订阅 |
| `GetAvailableMemory` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 获取可用内存 |
| `GetTotalMemory` | 同步 | `USE_PURGEABLE_MEMORY` | `mem_mgr_service.cpp` | 获取总内存 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:38-63`

---

## 2. MemMgrClient 详解

### 2.1 类定义

```cpp
class MemMgrClient {
    DECLARE_SINGLE_INSTANCE(MemMgrClient);  // 单例模式
};
```

**证据**: `interface/innerkits/include/mem_mgr_client.h:52-53`

### 2.2 方法列表

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `GetBundlePriorityList` | `BundlePriorityList &` | `int32_t` | 获取优先级列表 |
| `NotifyDistDevStatus` | `pid, uid, name, connected` | `int32_t` | 通知分布式设备 |
| `GetKillLevelOfLmkd` | `int32_t &` | `int32_t` | 获取查杀级别 |
| `RegisterActiveApps` | `pid, uid` | `int32_t` | 注册活跃应用 |
| `DeregisterActiveApps` | `pid, uid` | `int32_t` | 注销活跃应用 |
| `SubscribeAppState` | `const AppStateSubscriber &` | `int32_t` | 订阅状态 |
| `UnsubscribeAppState` | `const AppStateSubscriber &` | `int32_t` | 取消订阅 |
| `GetAvailableMemory` | `int32_t &` / 无 | `int32_t` | 获取可用内存 |
| `GetTotalMemory` | `int32_t &` / 无 | `int32_t` | 获取总内存 |
| `OnWindowVisibilityChanged` | `std::vector<sptr<MemMgrWindowInfo>>` | `int32_t` | 窗口可见性变化 |
| `GetReclaimPriorityByPid` | `pid, priority` | `int32_t` | 按 PID 查优先级 |
| `NotifyProcessStateChangedSync` | `MemMgrProcessStateInfo` | `int32_t` | 同步通知状态变化 |
| `NotifyProcessStateChangedAsync` | `MemMgrProcessStateInfo` | `int32_t` | 异步通知状态变化 |
| `NotifyProcessStatus` | `pid, type, status, saId` | `int32_t` | 通知进程状态 |
| `SetCritical` | `pid, critical, saId` | `int32_t` | 设置关键进程 |
| `MemoryStatusChanged` | `pid, type, status` | `int32_t` | 内存状态变化 |
| `SetDmabufUsage` | `fd, usage` | `int32_t` | 设置 dmabuf 用法 |
| `Reclaim` | `pid, fd` | `int32_t` | 清理 (purgeable) |
| `Resume` | `pid, fd` | `int32_t` | 恢复 (purgeable) |
| `SetDmabufInfo` | `fd, info` | `int32_t` | 设置 dmabuf 信息 |

**证据**: `interface/innerkits/include/mem_mgr_client.h:56-77`

### 2.3 内部实现

```cpp
class MemMgrClient {
private:
    sptr<IMemMgr> GetMemMgrService();  // 获取 SA 代理
    std::mutex mutex_;
    sptr<IMemMgr> dpProxy_;
};
```

**证据**: `interface/innerkits/include/mem_mgr_client.h:79-82`

**调用链**:

```
MemMgrClient::XXX()
        │
        └──▶ GetMemMgrService()
                │
                ├──▶ SamgrProxy::GetSystemAbility(1909)
                │           │
                │           └──▶ IPC Framework
                │
                └──▶ IMemMgr (代理调用)
```

---

## 3. IMemMgr 服务端接口

### 3.1 接口定义

```cpp
class IMemMgr : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.memory.MemMgr");
    // ... 虚方法声明
};
```

**证据**: `interface/innerkits/include/i_mem_mgr.h:34-36`

### 3.2 接口方法详情

#### 3.2.1 GetBundlePriorityList

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t GetBundlePriorityList(BundlePriorityList &bundlePrioList)` |
| 参数 | `bundlePrioList`: [输出] Bundle 优先级列表 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 实现文件 | `mem_mgr_service.cpp` |

**证据**: `interface/innerkits/include/i_mem_mgr.h:38`

#### 3.2.2 NotifyDistDevStatus

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t NotifyDistDevStatus(int32_t pid, int32_t uid, const std::string &name, bool connected)` |
| 参数 | `pid`: 进程 ID; `uid`: 用户 ID; `name`: 设备名; `connected`: 是否连接 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 通知分布式设备的连接状态变化 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:40`

#### 3.2.3 GetKillLevelOfLmkd

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t GetKillLevelOfLmkd(int32_t &killLevel)` |
| 参数 | `killLevel`: [输出] 当前查杀级别 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 获取当前 lmkd 的查杀级别 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:42`

#### 3.2.4 OnWindowVisibilityChanged

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t OnWindowVisibilityChanged(const std::vector<sptr<MemMgrWindowInfo>> &)` |
| 参数 | `MemMgrWindowInfo`: 窗口信息结构 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 通知窗口可见性变化，更新进程优先级 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:58`

#### 3.2.5 GetReclaimPriorityByPid

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t GetReclaimPriorityByPid(int32_t pid, int32_t &priority)` |
| 参数 | `pid`: 进程 ID; `priority`: [输出] 回收优先级 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 根据 PID 查询进程的回收优先级 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:59`

#### 3.2.6 NotifyProcessStateChanged (Sync/Async)

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t NotifyProcessStateChangedSync/Async(const MemMgrProcessStateInfo &)` |
| 参数 | `processStateInfo`: 进程状态信息 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 同步/异步通知进程状态变化 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:60-61`

#### 3.2.7 NotifyProcessStatus

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t NotifyProcessStatus(int32_t pid, int32_t type, int32_t status, int saId = -1)` |
| 参数 | `type`: 状态类型; `status`: 状态值; `saId`: SA ID |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 通用进程状态通知 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:62`

#### 3.2.8 SetCritical

| 属性 | 值 |
|------|-----|
| 方法签名 | `int32_t SetCritical(int32_t pid, bool critical, int32_t saId = -1)` |
| 参数 | `critical`: 是否关键进程 |
| 返回值 | `0`: 成功; `<0`: 错误码 |
| 描述 | 设置进程为关键进程（不可被杀） |

**证据**: `interface/innerkits/include/i_mem_mgr.h:63`

---

## 4. 条件编译 API (Purgeable Memory)

以下 API 需要启用 `USE_PURGEABLE_MEMORY` 编译选项：

| 方法 | 描述 |
|------|------|
| `RegisterActiveApps(pid, uid)` | 注册活跃应用 |
| `DeregisterActiveApps(pid, uid)` | 注销活跃应用 |
| `SubscribeAppState(subscriber)` | 订阅应用状态变化 |
| `UnsubscribeAppState(subscriber)` | 取消订阅 |
| `GetAvailableMemory(memSize)` | 获取可用内存大小 |
| `GetTotalMemory(memSize)` | 获取系统总内存 |

**证据**: `interface/innerkits/include/i_mem_mgr.h:44-56`

---

## 5. 数据结构

### 5.1 BundlePriorityList

```cpp
class BundlePriorityList {
    std::vector<BundlePriority> bundlePriorities_;
};
```

**证据**: `interface/innerkits/include/bundle_priority_list.h`

### 5.2 MemMgrWindowInfo

```cpp
class MemMgrWindowInfo {
    int32_t pid_;
    int32_t uid_;
    bool isVisible_;        // 是否可见
    int32_t priority_;     // 窗口优先级
};
```

**证据**: `interface/innerkits/include/mem_mgr_window_info.h`

### 5.3 MemMgrProcessStateInfo

```cpp
class MemMgrProcessStateInfo {
    int32_t pid_;
    int32_t uid_;
    int32_t state_;         // 进程状态
    // ... 其他状态信息
};
```

**证据**: `interface/innerkits/include/mem_mgr_process_state_info.h`

---

## 6. 调用链示例

### 6.1 客户端调用示例

```cpp
#include "mem_mgr_client.h"

using namespace OHOS::Memory;

void Example() {
    // 获取单例
    auto &client = MemMgrClient::GetInstance();

    // 获取优先级列表
    BundlePriorityList list;
    client.GetBundlePriorityList(list);

    // 通知窗口变化
    std::vector<sptr<MemMgrWindowInfo>> windowInfos;
    // ... 填充 windowInfos
    client.OnWindowVisibilityChanged(windowInfos);

    // 获取进程优先级
    int32_t priority = 0;
    client.GetReclaimPriorityByPid(getpid(), priority);
}
```

### 6.2 调用链图

```
调用方 (如 AbilityMS)
        │
        ▼
MemMgrClient::GetInstance()
        │
        ▼
MemMgrClient::XXX()
        │
        ▼
GetMemMgrService()
        │
        ├──▶ SamgrProxy::GetSystemAbility(1909)
        │           │
        │           └──▶ IPC Binder
        │
        └──▶ IMemMgr Proxy
                    │
                    ▼
            MemMgrService::OnRemoteRequest()
                    │
                    ├──▶ IMemMgr 接口码路由
                    │
                    └──▶ 对应方法实现
```

---

## 7. 常量定义

### 7.1 内存类型码

```cpp
enum class MemoryTypeCode {
    DMABUF = 0,
};
```

### 7.2 内存状态码

```cpp
enum class MemoryStatusCode {
    USED = 0,
    UNUSED = 1,
};
```

**证据**: `interface/innerkits/include/mem_mgr_client.h:30-37`

---

## 8. C 导出接口

```cpp
extern "C" {
    int32_t notify_process_status(int32_t pid, int32_t type, int32_t status, int saId = -1);
    int32_t set_critical(int32_t pid, bool critical, int32_t saId = -1);
}
```

**证据**: `interface/innerkits/include/mem_mgr_client.h:23-26`

---

## 9. 相关跳转

- **概览**: [00_Overview.md](./00_Overview.md)
- **架构**: [01_Architecture.md](./01_Architecture.md)
- **构建配置**: [03_Build.md](./03_Build.md)
- **安全评审**: [04_Security.md](./04_Security.md)
- **导航**: [SUMMARY.md](./SUMMARY.md)

---

*文档版本: 3.1.0 | 最后更新: 2026-02-06*

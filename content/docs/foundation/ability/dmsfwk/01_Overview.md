# 01_Overview - 项目概览

## 一句话定义

**dmsfwk (Distributed Ability Manager Service Framework)** 是 OpenHarmony 的分布式组件管理服务框架，负责跨设备的 Ability 启动、续接、绑定和调用，实现应用在多设备间的无缝协作。

---

## 项目定位

### 在 OpenHarmony 中的位置

```
应用层 (Applications)
    ↓
应用框架层 (Ability Framework)
    ↓
系统服务层 ← dmsfwk 在此层级
    ├── 分布式调度 (dtbschedmgr - SA 1401)
    ├── 能力管理 (dtbabilitymgr - SA 1404)
    └── 协作管理 (dtbcollabmgr)
    ↓
基础设施层 (SoftBus, IPC, 权限)
```

### 核心职责

| 职责 | 说明 |
|-----|------|
| **跨设备发现** | 发现并管理周边可信设备 |
| **远程启动** | 在远程设备启动 Ability |
| **状态续接** | 迁移 Ability 运行状态到另一设备 |
| **远程绑定** | 跨设备绑定 Service Ability |
| **数据同步** | 分布式数据与事件同步 |

---

## 核心能力

### 1. 远程 Ability 启动

在远程设备上启动 Ability，支持数据返回。

**使用场景**: 手机端点击视频，自动在智慧屏上播放

**关键代码**: `services/dtbschedmgr/src/distributed_sched_service.cpp:START_REMOTE_ABILITY`

### 2. Ability 续接

将正在运行的 Ability 迁移到另一设备，保持状态连续。

**使用场景**: 手机上编辑文档，迁移到平板继续编辑

**关键代码**: `services/dtbschedmgr/src/continue/dsched_continue.cpp`

### 3. 远程 Ability 绑定

跨设备绑定 Service Ability，建立持久连接。

**使用场景**: 手机绑定车载导航服务，实时同步导航状态

**关键代码**: `services/dtbcollabmgr/src/ability_connection_manager/`

### 4. 远程调用

跨设备同步调用 Ability 的方法。

**使用场景**: 手表调用手机的支付能力

**关键代码**: `services/dtbschedmgr/src/distributed_sched_service.cpp:START_REMOTE_ABILITY_BY_CALL`

---

## 运行环境

### 依赖组件

来自 `bundle.json` 的关键依赖：

| 组件 | 用途 |
|-----|------|
| `dsoftbus` | 分布式软总线，底层通信 |
| `ability_runtime` | Ability 运行时 |
| `device_auth` | 设备认证 |
| `access_token` | 权限管理 |
| `ipc` | 进程间通信 |
| `samgr` | SystemAbility 管理 |

### 权限要求

关键权限定义：

```cpp
// services/dtbabilitymgr/src/distributed_ability_manager_service.cpp:47
const std::string PERMISSION_DISTRIBUTED_DATASYNC = "ohos.permission.DISTRIBUTED_DATASYNC";

// services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:44-47
"ohos.permission.INTERNET",
"ohos.permission.GET_NETWORK_INFO", 
"ohos.permission.DISTRIBUTED_DATASYNC"
```

---

## 快速开始

### N-API 使用示例

```javascript
// 1. 导入模块
import continuationManager from '@ohos.continuation.continuationManager';

// 2. 注册 Ability 续接能力
continuationManager.register(
  {
    bundleName: "com.example.myapp",
    abilityName: "MainAbility"
  },
  (err, token) => {
    if (err) {
      console.error("注册失败:", err);
      return;
    }
    console.log("续接令牌:", token);
  }
);

// 3. 启动设备选择器
continuationManager.startDeviceManager(token, (err, deviceList) => {
  if (err) {
    console.error("启动失败:", err);
    return;
  }
  console.log("可选设备:", deviceList);
});

// 4. 更新连接状态
continuationManager.updateConnectStatus(token, deviceId, "CONNECTED");
```

### C++ SDK 使用示例

```cpp
#include "distributed_ability_manager_client.h"

using namespace OHOS::DistributedSchedule;

// 获取客户端实例
auto& client = DistributedAbilityManagerClient::GetInstance();

// 注册续接回调
client.Register(callback, extraParams);

// 开始续接
client.StartDeviceManager(token, filterParams);
```

---

## 系统能力 (SysCap)

根据 `bundle.json:17-18`：

- `SystemCapability.DistributedSched.AppCollaboration` - 应用协作能力
- `SystemCapability.Ability.DistributedAbilityManager` - 分布式 Ability 管理能力

---

## 版本信息

| 属性 | 值 |
|-----|-----|
| 组件名 | @ohos/dmsfwk |
| 版本 | 3.1 |
| 子系统 | ability |
| 许可证 | Apache License 2.0 |
| 仓库 | https://gitee.com/openharmony/ability_dmsfwk |

---

## 相关链接

- 下一章: [02_Architecture.md](02_Architecture.md) - 系统架构
- 接口文档: [04_Interface.md](04_Interface.md) - 完整接口清单
- 安全分析: [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析

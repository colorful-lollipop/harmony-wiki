# Device Standby 概览

## 项目定位

`device_standby` 是 OpenHarmony 资源调度子系统的核心部件，负责**管理设备待机时的功耗优化**。当设备进入待机空闲状态时，系统会限制后台应用使用资源，开发者可以为自己的应用申请纳入待机资源管控或暂时不被待机资源管控。

## 核心能力

| 能力 | 说明 |
|------|------|
| **待机状态管理** | 设备工作/睡眠/午睡/暗色/维护状态切换 |
| **资源豁免** | 应用可申请临时豁免待机管控 |
| **策略执行** | 网络/定时器/RunningLock 等资源限制 |
| **状态监控** | 充电状态/运动传感器/输入事件监听 |
| **事件通知** | 订阅待机状态变化事件 |

## 运行环境

### 系统要求

| 要求 | 版本 |
|------|------|
| OpenHarmony | 4.0+ |
| 系统类型 | standard |

### 依赖组件

| 组件 | 用途 |
|------|------|
| `ability_runtime` | 能力运行时 |
| `access_token` | 权限管理 |
| `ipc` | 进程间通信 |
| `napi` | Node.js API |
| `power_manager` | 电源管理 |
| `samgr` | 系统能力框架 |
| `sensor` | 传感器 |
| `work_scheduler` | 工作调度 |

### 完整依赖列表

```
ability_base, ability_runtime, access_token, background_task_mgr,
battery_manager, bundle_framework, call_manager, common_event_service,
config_policy, c_utils, eventhandler, hiccollie, hilog, hitrace,
idl_tool, init, ipc, input, napi, netmanager_base, power_manager,
runtime_core, safwk, samgr, sensor, time_service, work_scheduler,
json, resource_schedule_service
```

## 关键概念

### 资源类型 (ResourceType)

应用可申请豁免的资源类型：

| 名称 | 值 | 说明 |
|------|-----|------|
| `NETWORK` | 1 | 网络访问资源 |
| `RUNNING_LOCK` | 2 | CPU RunningLock 资源 |
| `TIMER` | 4 | 定时器任务资源 |
| `WORK_SCHEDULER` | 8 | Work 任务资源 |
| `AUTO_SYNC` | 16 | 自动同步资源 |
| `PUSH` | 32 | PushKit 资源 |
| `FREEZE` | 64 | 冻结应用资源 |

### 设备状态

| 状态 | 说明 |
|------|------|
| `working` | 设备正常工作状态 |
| `sleep` | 设备睡眠状态 |
| `nap` | 设备午睡状态（短时间休息） |
| `dark` | 设备暗色状态（常亮但低功耗） |
| `maintenance` | 设备维护状态 |

### 豁免机制

- **临时豁免**：应用通过 API 申请，指定时长
- **系统应用**：系统应用可自动获得豁免
- **特权操作**：特定系统操作可自动豁免

## 使用示例

```typescript
// 申请豁免资源
import deviceStandby from '@ohos.deviceStandby';

let request = {
  resourceTypes: deviceStandby.ResourceType.NETWORK | deviceStandby.ResourceType.TIMER,
  uid: 1000,
  name: "com.example.myapp",
  duration: 300,  // 秒
  reason: "music_playback"
};

deviceStandby.requestExemptionResource(request);

// 获取豁免应用列表
deviceStandby.getExemptedApps(deviceStandby.ResourceType.NETWORK)
  .then((apps) => {
    console.log("豁免应用列表:", apps);
  });
```

## 相关仓库

- [resourceschedule_device_standby](https://gitee.com/openharmony/resourceschedule_device_standby)
- [resourceschedule_work_scheduler](https://gitee.com/openharmony/resourceschedule_work_scheduler)
- [notification_ces_standard](https://gitee.com/openharmony/notification_ces_standard)
- [appexecfwk_standard](https://gitee.com/openharmony/appexecfwk_standard)
- [powermgr_battery_manager](https://gitee.com/openharmony/powermgr_battery_manager)
- [resourceschedule_background_task_mgr](https://gitee.com/openharmony/resourceschedule_background_task_mgr)

## 权限要求

使用设备待机 API 需要申请以下权限：

| 权限 | 说明 |
|------|------|
| `ohos.permission.DEVICE_STANDBY_EXEMPTION` | 应用豁免权限（敏感权限） |

> **注意**：第三方应用需要通过权限弹窗获取用户授权。

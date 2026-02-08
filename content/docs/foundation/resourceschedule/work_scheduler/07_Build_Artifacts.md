# 编译产物文档

> **目的**: 列出 Work Scheduler 模块的所有编译产物和安装路径
> **适用范围**: ROM/RAM 规划者、系统集成者、设备制造商

---

## 核心编译产物

### System Ability 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libworkschedservice.z.so` | `workschedservice` | ohos_shared_library (sa) | `/system/lib64/` 或 `/vendor/lib64/` | ~1.5 MB |

**用途**:
- Work Scheduler System Ability (SA 1904) 的主库
- 由 SAMGR 在系统启动时加载到 `resource_schedule_service` 进程
- 实现所有任务调度、条件监听、策略过滤逻辑

**依赖的库**:
- `libworkschedclient.so` (客户端代理)
- `libwork_sched_utils.so` (工具库)
- 其他系统服务库（Ability Runtime、Bundle Manager 等）

**证据**: `services/BUILD.gn:27-152` + `sa_profile/1904.json`

---

### N-API 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libworkscheduler.so` | `workscheduler` | ohos_shared_library | `/system/module/resourceschedule/` | ~200 KB |

**用途**:
- 对外暴露 `resourceschedule.workScheduler` 模块
- 提供 6 个核心 API 方法
- 导出 4 个枚举类型（NetworkType、ChargingType、BatteryStatus、StorageRequest）

**导出符号**:
```
napi_value InitApi()
NetworkType, ChargingType, BatteryStatus, StorageRequest (类)
startWork, stopWork, getWorkStatus, obtainAllWorks, stopAndClearWorks, isLastWorkTimeOut
```

**证据**: `interfaces/kits/js/BUILD.gn:22-68`

---

### Extension 框架库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libworkschedextension.so` | `workschedextension` | ohos_shared_library | `/system/module/extensionability/` | ~100 KB |

**用途**:
- WorkSchedulerExtension 框架实现
- 支持 ExtensionAbility 生命周期
- 支持任务回调机制

**证据**: `frameworks/extension/BUILD.gn:27-90`

---

### ExtensionAbility N-API 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libworkschedulerextensionability_napi.so` | `workschedulerextensionability_napi` | ohos_shared_library | `/system/module/` | ~50 KB |

**用途**:
- `WorkSchedulerExtensionAbility` N-API 绑定
- 对外暴露 ExtensionAbility 类
- 支持 `onWorkStart(workInfo)` 和 `onWorkStop(workInfo)` 回调

**证据**: `interfaces/kits/js/napi/work_scheduler_extension/BUILD.gn`

---

### ExtensionContext N-API 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libworkschedulerextensioncontext_napi.so` | `workschedulerextensioncontext_napi` | ohos_shared_library | `/system/module/application/` | ~50 KB |

**用途**:
- `WorkSchedulerExtensionContext` N-API 绑定
- 对外暴露 ExtensionContext 类
- 支持 `startServiceExtensionAbility` 和 `stopServiceExtensionAbility` 方法

**证据**: `interfaces/kits/js/napi/work_scheduler_extension_context/BUILD.gn`

---

### Cangjie FFI 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libcj_work_scheduler_ffi.so` | `cj_work_scheduler_ffi` | ohos_shared_library | `/system/module/` (未指定) | ~150 KB |

**用途**:
- Cangjie 语言 FFI 接口
- 提供 10 个 FFI 函数（V1 + V2 版本）
- 与主 N-API 功能一致

**导出函数**:
```
CJ_StartWork, CJ_StopWork, CJ_GetWorkStatus, CJ_ObtainAllWorks, CJ_IsLastWorkTimeOut, CJ_StopAndClearWorks
CJ_StartWorkV2, CJ_StopWorkV2, CJ_GetWorkStatusV2, CJ_ObtainAllWorksV2
```

**证据**: `interfaces/kits/cj/BUILD.gn:18-76`

---

### ArkTS/Taihe 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libwork_scheduler.so` | `work_scheduler_taihe` | taihe_shared_library | `/system/module/` | ~300 KB |

**用途**:
- ArkTS 原生接口（Taihe 框架）
- 提供高性能的 ArkTS 接口
- 替代 N-API（未来方向）

**证据**: `interfaces/kits/ets/taihe/work_scheduler/BUILD.gn:36-75`

### ANI Extension 库

| 产物 | Target | 类型 | 安装路径 | 大小估算 |
|--------|--------|------|-----------|----------|
| `libwork_scheduler_extension_ani.so` | `work_scheduler_extension_ani` | taihe_shared_library | `/system/module/` | ~200 KB |

**用途**:
- WorkSchedulerExtensionAbility 的 ANI 实现
- 支持 ArkTS Extension 回调
- 提供高性能的原生桥接

**证据**: `interfaces/kits/ets/taihe/work_scheduler_extension/BUILD.gn:18-77`

---

## 预编译文件 (ABC)

| 文件 | 类型 | 位置 | 大小估算 |
|------|------|--------|----------|
| `work_scheduler_abc.abc` | ohos_prebuilt_etc | `/system/framework/work_scheduler_abc.abc` | ~50 KB |
| `workscheduler_extension_context_abc.abc` | ohos_prebuilt_etc | `/system/framework/workscheduler_extension_context_abc.abc` | ~30 KB |
| `workscheduler_extension_ability_abc.abc` | ohos_prebuilt_etc | `/system/framework/workscheduler_extension_ability_abc.abc` | ~30 KB |

**用途**:
- ArkTS/ETS 类型定义文件（ABC 格式）
- 用于运行时类型检查和加载
- 预编译到 `/system/framework/` 目录

**证据**: `interfaces/kits/ets/taihe/work_scheduler/BUILD.gn:77-84`

---

## SA 配置文件

| 文件 | 位置 | 说明 |
|------|------|------|
| `1904.json` | `/system/profile/` | WorkScheduler SA 配置 |

**配置内容**:
```json
{
  "process": "resource_schedule_service",
  "systemability": [{
    "name": 1904,
    "libpath": "libworkschedservice.z.so",
    "run-on-create": true,
    "distributed": false
  }]
}
```

**加载时机**:
- SAMGR 在系统启动时加载配置
- 创建 `resource_schedule_service` 进程
- 加载 `libworkschedservice.z.so` 并注册 SA 1904

**证据**: `sa_profile/1904.json` + `services/BUILD.gn`

---

## HiSysEvent 配置

| 文件 | 位置 | 说明 |
|------|------|------|
| `hisysevent.yaml` | `/system/etc/` 或内置 | 事件定义 |

**事件域**: `WORK_SCHEDULER`
**事件列表**:
- `WORK_SCHEDULER_START`
- `WORK_SCHEDULER_STOP`
- `WORK_SCHEDULER_TIMEOUT`
- `WORK_SCHEDULER_CONDITION_CHANGE`
- `WORK_SCHEDULER_POLICY_CHECK`

**证据**: `hisysevent.yaml`

---

## 运行时加载关系

### 应用启动流程

```
应用进程启动
  ↓
加载 N-API: libworkscheduler.so
  ↓
执行 RegisterModule() -> InitApi()
  ↓
注册 JS 模块: resourceschedule.workScheduler
  ↓
首次 API 调用时建立 IPC 连接
  ↓
通过 SAMGR 获取 SA 1904 代理
  ↓
后续 API 调用直接通过 IPC
```

### 系统启动流程

```
init 进程
  ↓
SAMGR (System Ability Manager)
  ↓
加载 SA 1904 配置 (sa_profile/1904.json)
  ↓
创建/启动 resource_schedule_service 进程
  ↓
加载 libworkschedservice.z.so
  ↓
注册 IWorkSchedService 接口 (SA ID 1904)
  ↓
OnStart(): 初始化 EventRunner 和 Handler
  ↓
AddSystemAbilityListener(): 监听依赖 SA
  ↓
SA 进入 Ready 状态
```

---

## ROM/RAM 占用

### ROM 占用（来自 bundle.json）

| 组件 | ROM | 说明 |
|--------|-----|------|
| **Work Scheduler Service** | 2048 KB | 包含所有服务逻辑 |
| **N-API 库** | ~200 KB | JS 接口库 |
| **Extension 框架** | ~100 KB | Extension 支持 |
| **其他支持** | ~200 KB | 策略、监听器等 |
| **总计** | ~2550 KB | 不包含依赖库 |

### RAM 占用（运行时）

| 组件 | RAM | 说明 |
|--------|-----|------|
| **SA Service (常驻）** | 10-15 MB | 包含所有条件监听器和策略 |
| **N-API 库（应用）** | 1-2 MB | 应用进程加载 |
| **Extension 框架** | 500 KB - 1 MB | Extension Ability 加载 |
| **单个任务** | 50-100 KB | WorkInfo + 状态管理 |
| **条件监听器** | 100-200 KB | 每个监听器实例 |

---

## 动态库依赖

### 运行时依赖库

**SA Service 依赖**:
```
libworkschedservice.z.so
├── libworkschedclient.so (内部静态链接或动态加载)
├── libffrt.so (FFRT)
├── libhilog.so (日志)
├── libipc_single.so (IPC)
├── libz.so (压缩)
├── libnapi.z.so (N-API，仅初始化)
└── [系统服务库]
    ├── libability_runtime.z.so
    ├── libbundle_framework.z.so
    ├── libcesfwk.z.so
    └── ...
```

**N-API 库依赖**:
```
libworkscheduler.so
├── libipc_single.so (IPC)
├── libnapi.z.so (N-API)
├── libhilog.so (日志)
└── libc.so / libm.so (系统库)
```

---

## 产品差异

### 标准版 vs 精简版

| 功能 | 标准版 | 精简版 | 说明 |
|------|---------|---------|------|
| **应用活跃度分组** | ✅ 支持 | ❌ 可能不支持 |
| **所有条件监听** | ✅ 支持 | 🟡 部分支持 |
| **所有策略过滤器** | ✅ 支持 | 🟡 部分支持 |
| **HiCollie 集成** | ✅ 支持 | ❌ 可能不支持 |
| **Extension 回调** | ✅ 支持 | 🟡 部分支持 |

### Feature Flags 影响

根据 `workscheduler.gni` 中的 Feature Flags，最终产物可能不同：

| Feature | 禁用时影响 |
|--------|-------------|
| `bundle_active_enable` | 禁用频率控制 |
| `device_standby_enable` | 禁用待机协同 |
| `resourceschedule_bgtaskmgr_enable` | 禁用后台任务订阅 |
| `powermgr_battery_manager_enable` | 禁用电池条件 |
| `powermgr_thermal_manager_enable` | 禁用温度策略 |
| `workscheduler_with_communication_netmanager_base_enable` | 禁用网络条件 |

---

## 验证方法

### 检查编译产物

```bash
# 1. 查看编译输出目录
ls -lh out/[产品名]/lib.unstripped/

# 2. 检查 SA 库
ls -lh out/[产品名]/sa_profile/

# 3. 检查 N-API 库
ls -lh out/[产品名]/module/resourceschedule/

# 4. 检查符号表
readelf -s out/[产品名]/lib.unstripped/libworkscheduler.z.so | grep "Symbol table"
```

### 检查安装位置

```bash
# 在设备上查看
hdc shell hdc shell "ls -l /system/module/resourceschedule/libworkscheduler.so"

hdc shell hdc shell "ls -l /system/lib64/libworkschedservice.z.so"

# 查看 SA 配置
hdc shell hdc shell "cat /system/profile/1904.json"
```

### 运行时依赖检查

```bash
# 查看 N-API 库依赖
hdc shell hdc shell "ldd /system/module/resourceschedule/libworkscheduler.so"

# 查看 SA 库依赖
hdc shell hdc shell "ldd /system/lib64/libworkschedservice.z.so"
```

---

## 相关跳转

- [06_GN_Targets.md](06_GN_Targets.md) - 构建系统详解
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [08_Security.md](08_Security.md) - 安全注意事项

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| ROM 占用 | `bundle.json:20-21` |
| RAM 占用 | `bundle.json:21-22` |
| SA 配置 | `sa_profile/1904.json` |
| N-API 库路径 | `interfaces/kits/js/BUILD.gn:65` |
| SA 库路径 | `services/BUILD.gn:27` |

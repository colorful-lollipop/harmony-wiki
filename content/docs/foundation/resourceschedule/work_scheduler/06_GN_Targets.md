# GN Targets 与编译产物

> **目的**: 列出 Work Scheduler 模块的所有 GN 构建目标和编译产物
> **适用范围**: 构建系统开发者、ROM/RAM 规划者、系统集成者

---

## 顶级 Group Targets

### 根 BUILD.gn

**文件**: `/BUILD.gn`

| Target | 类型 | 说明 |
|--------|------|------|
| `fwk_group_work_scheduler_all` | group | 框架层所有目标（对外 SDK） |
| `service_group_work_scheduler_all` | group | 服务层所有目标（System Ability） |
| `test_work_scheduler_all` | group | 测试目标（已忽略） |

---

## Frameworks 层 Targets

### frameworks/BUILD.gn

| Target | 类型 | 输出名 | 依赖 |
|--------|------|--------|------|
| `work_sched_service_interface` | idl_gen_interface | - |
| `workschedclient` | ohos_shared_library | `work_sched_service_interface`, `workschedutils` |
| `work_sched_service_proxy` | ohos_source_set | `work_sched_service_interface`, `workschedutils` |
| `work_sched_service_stub` | ohos_source_set | `work_sched_service_interface`, `workschedutils` |

### frameworks/extension/BUILD.gn

| Target | 类型 | 输出名 | 依赖 |
|--------|------|--------|------|
| `workschedextension` | ohos_shared_library | `workschedclient`, `workschedulerextension_ani`, `workschedutils`, `workschedservice_zidl_stub` |

---

## Interfaces 层 Targets

### interfaces/kits/js/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `workscheduler` | ohos_shared_library | libworkscheduler.so | module/resourceschedule/ |

### interfaces/kits/js/napi/work_scheduler_extension/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `workschedulerextensionability_napi` | ohos_shared_library | libworkschedulerextensionability_napi.so | module/ |
| `work_scheduler_extension_ability_js` | gen_js_obj | - (内部编译） |
| `work_scheduler_extension_ability_abc` | gen_js_obj | - (内部编译） |

### interfaces/kits/js/napi/work_scheduler_extension_context/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `workschedulerextensioncontext_napi` | ohos_shared_library | libworkschedulerextensioncontext_napi.so | module/application/ |
| `work_scheduler_extension_context_js` | gen_js_obj | - (内部编译） |
| `work_scheduler_extension_context_abc` | gen_js_obj | - (内部编译） |

### interfaces/kits/cj/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `cj_work_scheduler_ffi` | ohos_shared_library | libcj_work_scheduler_ffi.so | (未指定，默认 module/） |

### interfaces/kits/ets/taihe/work_scheduler/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `work_scheduler_ani` | taihe_shared_library | libwork_scheduler.so | module/ |
| `work_scheduler_abc` | generate_static_abc | work_scheduler_abc.abc | system/framework/ (预编译） |
| `work_scheduler_etc` | ohos_prebuilt_etc | work_scheduler_abc.abc | system/framework/ |

### interfaces/kits/ets/taihe/work_scheduler_extension/BUILD.gn

| Target | 类型 | 输出名 | 安装路径 |
|--------|------|--------|--------|
| `work_scheduler_extension_ani` | taihe_shared_library | libwork_scheduler_extension_ani.so | module/ |
| `workscheduler_extension_context_abc` | generate_static_abc | workscheduler_extension_context_abc.abc | system/framework/ |
| `workscheduler_extension_context_etc` | ohos_prebuilt_etc | workscheduler_extension_context_abc.abc | system/framework/ |
| `workscheduler_extension_ability_abc` | generate_static_abc | workscheduler_extension_ability_abc.abc | system/framework/ |
| `workscheduler_extension_ability_etc` | ohos_prebuilt_etc | workscheduler_extension_ability_abc.abc | system/framework/ |

---

## Services 层 Targets

### services/BUILD.gn

| Target | 类型 | 输出名 | SA 类型 |
|--------|------|--------|--------|
| `workschedservice` | ohos_shared_library | libworkschedservice.z.so | sa |
| `workschedservice_static` | ohos_static_library | libworkschedservice_static.a | - (测试用） |

### services/zidl/BUILD.gn

| Target | 类型 | 输出名 |
|--------|------|--------|
| `work_scheduler_interface` | idl_gen_interface | - |
| `workschedservice_zidl_stub` | ohos_source_set | - |
| `workschedservice_zidl_proxy` | ohos_source_set | - |

---

## Utils 层 Targets

### utils/native/BUILD.gn

| Target | 类型 | 输出名 |
|--------|------|--------|
| `workschedutils` | ohos_source_set | - |

---

## sa_profile Targets

### sa_profile/BUILD.gn

| Target | 类型 | 输出名 | 说明 |
|--------|------|--------|------|
| `worksched_sa_profile` | ohos_sa_profile | 1904.json (SA 配置） |

---

## Feature Flags

### workscheduler.gni 定义

| Flag | 默认值 | 控制的功能 | 影响的 Target |
|------|---------|---------|---------------|
| `work_scheduler_device_enable` | true | 全局开关 | 所有 targets |
| `bundle_active_enable` | true (depends) | 应用活跃度频率控制 | `workschedservice` |
| `device_standby_enable` | true (depends) | 待机状态监听 | `workschedservice` |
| `resourceschedule_bgtaskmgr_enable` | true (depends) | 后台任务订阅 | `workschedservice` |
| `powermgr_battery_manager_enable` | true (depends) | 电池状态监听 | `workschedservice` |
| `powermgr_thermal_manager_enable` | true (depends) | 温度策略过滤 | `workschedservice` |
| `powermgr_power_manager_enable` | true (depends) | 功耗模式策略 | `workschedservice` |
| `workscheduler_with_communication_netmanager_base_enable` | true (depends) | 网络状态监听 | `workschedservice` |
| `workscheduler_hicollie_enable` | true (depends) | HiCollie 故障收集 | `workschedservice` |

### Feature Flags 实现

**在 `services/BUILD.gn` 中使用**:
```gn
if (bundle_active_enable) {
  external_deps += [ "device_usage_statistics:usagestatsinner" ]
  defines += [ "DEVICE_USAGE_STATISTICS_ENABLE" ]
}

if (device_standby_enable) {
  external_deps += [ "device_standby:standby_innerkits" ]
  defines += [ "DEVICE_STANDBY_ENABLE" ]
}

if (powermgr_power_manager_enable) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "POWERMGR_POWER_MANAGER_ENABLE" ]
  sources += [ "native/src/policy/power_mode_policy.cpp" ]
}
```

---

## 编译产物清单

### 核心库文件

| 库名 | 类型 | 大小估算 | 用途 |
|------|------|----------|------|
| `libworkschedservice.z.so` | Shared Library (SA) | ~1.5 MB | SA 服务主库 |
| `libworkscheduler.so` | Shared Library (NAPI) | ~200 KB | 主 N-API 接口库 |
| `libcj_work_scheduler_ffi.so` | Shared Library (FFI) | ~150 KB | Cangjie FFI 库 |
| `libwork_scheduler.so` | Taihe Shared Library | ~300 KB | ArkTS 接口库 |
| `libwork_scheduler_extension_ani.so` | Taihe Shared Library | ~200 KB | Extension ANI 库 |
| `libworkschedextension.so` | Shared Library | ~100 KB | Extension 框架库 |
| `libworkschedulerextensionability_napi.so` | Shared Library | ~50 KB | ExtensionAbility N-API |
| `libworkschedulerextensioncontext_napi.so` | Shared Library | ~50 KB | ExtensionContext N-API |

### IDL 生成代码

| 产物 | 说明 | 位置 |
|------|------|------|
| `IWorkSchedServiceStub.cpp` | 服务端 IPC Stub | 编译输出目录（由 IDL 编译器生成） |
| `IWorkSchedServiceProxy.cpp` | 客户端 IPC Proxy | 编译输出目录 |
| `WorkSchedulerStub.cpp` | Extension 回调 Stub | 编译输出目录 |
| `WorkSchedulerProxy.cpp` | Extension 回调 Proxy | 编译输出目录 |
| `IWorkSchedulerStub.cpp` | Extension 回调 Stub (ANI) | 编译输出目录 |
| `IWorkSchedulerProxy.cpp` | Extension 回调 Proxy (ANI) | 编译输出目录 |

### ABC 预编译文件

| 文件 | 类型 | 位置 |
|------|------|------|
| `work_scheduler_abc.abc` | ArkTS ABI 文件 | `/system/framework/work_scheduler_abc.abc` |
| `workscheduler_extension_context_abc.abc` | ArkTS ABI 文件 | `/system/framework/workscheduler_extension_context_abc.abc` |
| `workscheduler_extension_ability_abc.abc` | ArkTS ABI 文件 | `/system/framework/workscheduler_extension_ability_abc.abc` |

---

## 安装路径映射

| 库/文件 | 安装路径 | 说明 |
|-----------|-----------|------|
| `libworkschedservice.z.so` | `/system/lib64/` 或 `/vendor/lib64/` | System Ability 库（SAMGR 加载） |
| `libworkscheduler.so` | `/system/module/resourceschedule/` | N-API 库（应用加载） |
| `libcj_work_scheduler_ffi.so` | `/system/module/` (未指定) | Cangjie FFI 库 |
| `libwork_scheduler.so` | `/system/module/` | ArkTS 库（Taihe 加载） |
| `libwork_scheduler_extension_ani.so` | `/system/module/` | Extension ANI 库 |
| `libworkschedextension.so` | `/system/module/extensionability/` | Extension 框架库 |
| `libworkschedulerextensionability_napi.so` | `/system/module/` | ExtensionAbility N-API |
| `libworkschedulerextensioncontext_napi.so` | `/system/module/application/` | ExtensionContext N-API |
| `1904.json` | `/system/profile/` | SA 配置文件（SAMGR 读取） |

---

## 运行时加载关系

### 系统启动流程

```
init (init 进程)
  ↓
SAMGR (System Ability Manager)
  ↓
加载 SA 1904 配置 (sa_profile/1904.json)
  ↓
启动 resource_schedule_service 进程
  ↓
加载 libworkschedservice.z.so
  ↓
注册 IWorkSchedService 接口 (SA ID 1904)
  ↓
监听系统事件（网络、电池、屏幕等）
  ↓
应用启动时加载 N-API
  ↓
应用调用 workScheduler.startWork()
  ↓
通过 IPC 调用 SA 1904 的 StartWork() 方法
  ↓
Work Scheduler 添加到任务队列
  ↓
条件满足时启动 Ability
  ↓
Ability 的 onWorkStart() 回调
  ↓
任务执行完成
  ↓
Ability 的 onWorkStop() 回调
  ↓
通过 IWorkScheduler 回调通知服务
```

### 应用进程加载

```
应用进程启动
  ↓
加载 N-API: libworkscheduler.so
  ↓
执行 RegisterModule() -> InitApi()
  ↓
注册 JS 模块: resourceschedule.workScheduler
  ↓
应用代码调用 import workScheduler
  ↓
首次调用时建立 IPC 连接
  ↓
通过 SAMGR 获取 SA 1904 代理
  ↓
后续调用直接通过 IPC
```

### SA 依赖关系

```
Work Scheduler (SA 1904)
  ↓ 依赖
Bundle Manager (SA 2001)
  ↓ 依赖
Common Event Service (SA 2002)
  ↓ 依赖
Background Task Manager (SA 2701)
  ↓ 依赖
Device Usage Statistics (SA 2704)
  ↓ 依赖
Device Standby (SA 2802)
  ↓ 依赖
Battery Manager (SA 2301)
  ↓ 依赖
Thermal Manager (SA 2302)
  ↓ 依赖
Power Manager (SA 2304)
```

---

## IDL 编译流程

### frameworks/IWorkSchedService.idl

**输入**: `IWorkSchedService.idl`
**IDL 编译器**: `idl_gen_interface`
**输出**:
- `IWorkSchedService.h` - 接口头文件（在编译输出目录）
- `IWorkSchedServiceProxy.cpp` - 客户端代理实现
- `IWorkSchedServiceStub.cpp` - 服务端桩实现

### services/zidl/IWorkScheduler.idl

**输入**: `IWorkScheduler.idl`
**IDL 编译器**: `idl_gen_interface`
**输出**:
- `IWorkScheduler.h` - 接口头文件（在编译输出目录）
- `IWorkSchedulerProxy.cpp` - 客户端代理实现
- `IWorkSchedulerStub.cpp` - 服务端桩实现

---

## 构建配置详解

### Sanitize 配置

所有主要 Target 启用 CFI (Control Flow Integrity):

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

**作用**: 防止代码注入攻击，增强运行时安全性

### 分支保护

```gn
branch_protector_ret = "pac_ret"
```

**作用**: 使用 PAC (Pointer Authentication Codes) 保护返回地址

### 优化选项

```gn
cflags_cc = [
  "-fdata-sections",
  "-ffunction-sections",
  "-fvisibility=hidden",
  "-fstack-protector-strong",
  "-Os",
]
```

**说明**:
- `-fdata-sections` / `-ffunction-sections`: 数据和函数分段，减小二进制大小
- `-fvisibility=hidden`: 默认符号隐藏，仅导出必需符号
- `-fstack-protector-strong`: 堆栈保护
- `-Os`: 优化代码大小

---

## 依赖关系图

```
workscheduler (N-API 主库)
  ↓ depends
work_sched_service_interface (IDL 生成)
  ↓
workschedutils (工具库)
  ↓
[IPC 层]
  ├── work_sched_service_proxy (客户端代理)
  └── work_sched_service_stub (服务端桩)

workschedservice (SA 服务)
  ↓ depends
workschedclient (客户端库)
  ↓
work_sched_service_stub (服务端桩)
  ↓
workschedutils (工具库)
  ↓
workschedservice_zidl_proxy (Extension 代理)

workschedextension (Extension 框架)
  ↓ depends
workschedclient (客户端库)
  ↓
workschedutils (工具库)
  ↓
workschedservice_zidl_stub (Extension 桩)
  ↓
workschedulerextension_ani (Taihe 库)
```

---

## 运行时内存占用估算

| 组件 | 运行时内存 | 说明 |
|--------|-----------|------|
| **SA Service** | ~5-8 MB | 包含所有条件监听器、策略过滤器和队列 |
| **N-API 库** | ~500 KB | 应用进程加载 |
| **Extension 框架** | ~300 KB | Extension Ability 加载 |
| **单个任务** | ~50-100 KB | WorkInfo + 状态管理 |
| **条件监听器** | ~100-200 KB | 每个监听器实例 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 模块概览
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物详情
- [03_Architecture.md](03_Architecture.md) - 架构设计

---

**证据索引**:

| 结论 | 证据 |
|------|------|
| Group Targets | `BUILD.gn:17-39` |
| Framework Targets | `frameworks/BUILD.gn:34-74` |
| Extension Targets | `frameworks/extension/BUILD.gn:27-90` |
| Services Targets | `services/BUILD.gn:27-152` |
| NAPI Targets | `interfaces/kits/js/BUILD.gn:22-68` |
| CJ Targets | `interfaces/kits/cj/BUILD.gn:18-76` |
| Taihe Targets | `interfaces/kits/ets/taihe/work_scheduler/BUILD.gn:36-75` |
| Feature Flags | `workscheduler.gni:42-91` |
| SA 配置 | `sa_profile/1904.json:2-12` |

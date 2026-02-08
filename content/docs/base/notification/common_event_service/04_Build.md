# 04_构建编译

## 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口 |
| `bundle.json` | 组件配置 |
| `event.gni` | 全局变量与配置 |

## GN 关键 Targets

### 服务层 Targets

**文件**: `services/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cesfwk_services` | shared_library | libcesfwk_services.z.so | SA 服务实现 |
| `cesfwk_services_static` | static_library | libcesfwk_services_static.a | 测试用静态库 |
| `ces.para` | prebuilt_etc | etc/param/ces.para | 参数配置 |
| `ces.para.dac` | prebuilt_etc | etc/param/ces.para.dac | DAC 配置 |

### 核心框架 Targets

**文件**: `frameworks/core/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cesfwk_core` | shared_library | libcesfwk_core.z.so | 核心框架 |
| `common_event_interface` | idl_gen | ICommonEvent.*.cpp | IPC 接口生成 |
| `event_receive_interface` | idl_gen | IEventReceive.*.cpp | 回调接口生成 |
| `common_event_proxy` | source_set | - | IPC 代理 |
| `common_event_stub` | source_set | - | IPC 存根 |

### Native 套件 Targets

**文件**: `frameworks/native/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cesfwk_innerkits` | shared_library | libcesfwk_innerkits.z.so | Native 套件 |

### N-API Targets

**文件**: `interfaces/kits/napi/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `napi_packages` | - | - | N-API 包聚合 |
| `napi_commoneventmanager` | shared_library | libnapi_commoneventmanager.z.so | N-API 核心 |
| `commoneventmanager` | shared_library | libcommoneventmanager.z.so | 包装库 |

### 扩展框架 Targets

**文件**: `frameworks/extension/BUILD.gn`

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `cesfwk_extension` | shared_library | libcesfwk_extension.z.so | 扩展框架 |
| `static_subscriber_ipc` | shared_library | libstatic_subscriber_ipc.z.so | 静态订阅者 IPC |

## 组件配置

**文件**: `bundle.json`

```json
{
  "component": {
    "name": "common_event_service",
    "subsystem": "notification",
    "syscap": [
      "SystemCapability.Notification.CommonEvent"
    ],
    "features": [
      "common_event_service_with_graphics",
      "common_event_service_tool_cem_enable",
      "common_event_service_limit_screen_event",
      "common_event_service_boot_complete_delay"
    ]
  }
}
```

### 特性开关

| 特性 | 默认值 | 说明 |
|------|--------|------|
| `common_event_service_with_graphics` | true | 图形支持 |
| `common_event_service_tool_cem_enable` | true | CEM 工具 |
| `common_event_service_limit_screen_event` | false | 限制屏幕事件 |
| `common_event_service_boot_complete_delay` | false | 启动完成延迟 |

## 编译产物

### 运行时产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libcesfwk_services.z.so | /system/lib64/ | SA 服务 |
| libcesfwk_core.z.so | /system/lib64/ | 核心框架 |
| libcesfwk_innerkits.z.so | /system/lib64/ | Native 套件 |
| libcesfwk_extension.z.so | /system/lib64/ | 扩展框架 |
| libnapi_commoneventmanager.z.so | /system/lib64/ | N-API |
| libcommoneventmanager.z.so | /system/lib64/ | N-API 包装 |
| ces.para | /system/etc/param/ | 参数配置 |

### 安装路径

| 产物 | 安装目录 |
|------|----------|
| .so 文件 | /system/lib64/ (64位) 或 /system/lib/ (32位) |
| 参数文件 | /system/etc/param/ |

## 依赖关系

### 内部依赖

```
cesfwk_services
├── cesfwk_core
├── cesfwk_innerkits
└── static_subscriber_ipc

napi_commoneventmanager
└── cesfwk_innerkits

commoneventmanager
├── napi_commoneventmanager
└── cesfwk_innerkits
```

### 外部依赖

| 组件 | 用途 |
|------|------|
| bundle_framework | 包管理 |
| ipc | IPC 通信 |
| access_token | 权限 |
| safwk | SA 框架 |
| samgr | 服务管理 |
| ability_base | Ability 基础 |
| ability_runtime | Ability 运行 |
| eventhandler | 事件处理 |
| hilog | 日志 |
| ffrt | 任务调度 |
| libuv | 异步 I/O |
| napi | Node API |

## 构建配置

### 编译器标志

```gn
cflags = [
  "-fno-math-errno",
  "-fno-unroll-loops",
  "-fmerge-all-constants",
  "-fno-ident",
  "-Oz",
  "-flto",
  "-ffunction-sections",
  "-fdata-sections",
]
```

### 安全编译选项

| 选项 | 说明 |
|------|------|
| `integer_overflow` | 整数溢出检测 |
| `ubsan` | 未定义行为检测 |
| `boundary_sanitize` | 边界检查 |
| `cfi` | 控制流完整性 |
| `branch_protector_ret` | PACRET 返回地址保护 |

### 条件编译

```gn
// 用户版本
if (build_variant == "root") {
  defines += [ "BUILD_VARIANT_USER" ]
}

// 调试支持
if (build_variant == "root") {
  defines += [ "CEM_SUPPORT_DUMP" ]
}

// HiSysEvent
if (has_hisysevent_part) {
  cflags_cc += [ "-DHAS_HISYSEVENT_PART" ]
}

// 追踪
if (ces_hitrace_usage) {
  defines += [ "HITRACE_METER_ENABLE" ]
}
```

## 构建命令

### 完整构建

```bash
# 构建整个子系统
./build.sh --subsystem notification common_event_service

# 或使用 hb
hb build -p notification_common_event_service
```

### 模块构建

```bash
# 构建单个 part
hb build -p common_event_service

# 构建指定 target
hb build -T //base/notification/common_event_service/services:cesfwk_services
```

### 构建产物验证

```bash
# 查看构建产物
ls -la out/standard/xxx/system/lib64/ | grep ces
ls -la out/standard/xxx/system/lib64/ | grep commonevent
```

## 构建产物与 Target 映射

| Target | 产物 | 安装路径 |
|--------|------|----------|
| `cesfwk_services` | libcesfwk_services.z.so | /system/lib64/ |
| `cesfwk_core` | libcesfwk_core.z.so | /system/lib64/ |
| `cesfwk_innerkits` | libcesfwk_innerkits.z.so | /system/lib64/ |
| `cesfwk_extension` | libcesfwk_extension.z.so | /system/lib64/ |
| `napi_commoneventmanager` | libnapi_commoneventmanager.z.so | /system/lib64/ |
| `commoneventmanager` | libcommoneventmanager.z.so | /system/lib64/ |
| `ces.para` | ces.para | /system/etc/param/ |

## 相关文档

- [概览](00_Overview.md)
- [架构](01_Architecture.md)
- [内部 API](03_Inner_API.md)
- [安全评审](05_Security.md)

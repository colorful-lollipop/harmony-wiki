# 构建系统

**目的**: 描述 GN 构建配置、Targets 和编译产物

---

## 构建配置

### 根构建文件

| 文件 | 用途 | 证据 |
|------|------|------|
| `BUILD.gn` | 主构建配置 | 行 17-77 |
| `callmanager.gni` | 构建参数和源文件列表 | 行 1-376 |
| `bundle.json` | 模块配置 | 行 1-125 |

### 构建入口

```gn
# BUILD.gn:17-77
ohos_shared_library("tel_call_manager") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
    cfi_vcall_icall_only = true
  }
  branch_protector_ret = "pac_ret"
  sources = call_manager_sources
  include_dirs = call_manager_include_dirs
  deps = [
    "frameworks/native:tel_call_manager_api",
  ]
  # ... 更多配置
}
```

---

## 主要 Targets

### 编译单元列表

| Target 名称 | 类型 | 输出 | 依赖 |
|-------------|------|------|------|
| `tel_call_manager` | shared_library | `libtel_call_manager.z.so` | frameworks/native, 外部依赖 |
| `tel_call_manager_api` | static_library | `libtel_call_manager_api.a` | innerkits headers |

**证据**: `BUILD.gn:17`, `bundle.json:109`

### 源文件统计

- **核心源文件**: ~146 个 `.cpp` 文件
- **头文件目录**: 17 个 include 路径
- **总代码量**: ~376 行构建配置

**证据**: `callmanager.gni:28-147`, `callmanager.gni:149-172`

---

## 编译产物

### 产物清单

| 产物 | 路径 | 用途 |
|------|------|------|
| `libtel_call_manager.z.so` | system/lib64/ | 主共享库 |
| `libtel_call_manager_api.a` | out/ | 静态库（内部 API） |
| JS Bundle | - | 打包到 `.hap` |

### SA 配置

```json
// sa_profile/4005.json
{
    "process": "telecom",
    "systemability": [{
        "name": 4005,
        "libpath": "libtel_call_manager.z.so",
        "run-on-create": true
    }]
}
```

**证据**: `sa_profile/4005.json:5-10`

---

## 编译开关

### Feature Flags

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `call_manager_feature_hfp_async_enable` | false | HFP 异步启用 |
| `call_manager_feature_not_support_multicall` | false | 不支持多方通话 |
| `call_manager_feature_support_dsoftbus` | true | 支持分布式软总线 |
| `call_manager_feature_support_rtt` | false | 支持 RTT |
| `call_manager_feature_support_hearing_aid` | true | 支持助听器 |
| `call_manager_sos_no_ringback_tone` | false | SOS 无回铃音 |
| `call_manager_watch_call_blocking` | false | 手表呼叫阻断 |

**证据**: `callmanager.gni:15-21`

### 条件编译

| 宏定义 | 条件 | 作用 |
|--------|------|------|
| `SUPPORT_DSOFTBUS` | `call_manager_feature_support_dsoftbus=true` | 启用分布式通信 |
| `HFP_ASYNC_ENABLE` | `call_manager_feature_hfp_async_enable=true` | HFP 异步 |
| `NOT_SUPPORT_MULTICALL` | `call_manager_feature_not_support_multicall=true` | 禁用多方通话 |
| `SUPPORT_RTT_CALL` | `call_manager_feature_support_rtt=true` | RTT 支持 |
| `SUPPORT_HEARING_AID` | `call_manager_feature_support_hearing_aid=true` | 助听器支持 |

**证据**: `callmanager.gni:174-376`

---

## 外部依赖

### 核心依赖

| 依赖项 | 用途 | 证据 |
|--------|------|------|
| `ability_runtime` | 能力框架 | `callmanager.gni:217-223` |
| `access_token` | 权限管理 | `callmanager.gni:224-225` |
| `audio_framework` | 音频框架 | `callmanager.gni:226-230` |
| `ipc` | 进程间通信 | `callmanager.gni:246` |
| `safwk` | System Ability 框架 | `callmanager.gni:253` |
| `samgr` | 服务管理 | `callmanager.gni:254` |

### 可选依赖

| 依赖项 | 条件 | 用途 |
|--------|------|------|
| `bluetooth` | global_parts_info.communication_bluetooth | 蓝牙功能 |
| `dsoftbus` | SUPPORT_DSOFTBUS | 分布式通信 |
| `camera_framework` | multimedia_camera_framework | 视频通话 |
| `power_manager` | powermgr_power_manager | 电源管理 |

**证据**: `callmanager.gni:257-339`

---

## 编译命令

### 全量编译

```bash
# 编译 call_manager 模块
hb build call_manager

# 或使用 ninja
ninja -C out/... tel_call_manager
```

### 增量编译

```bash
# 编译指定模块
hb build -p call_manager
```

---

## 相关文档

- [API 参考](02_API_Reference.md)
- [安全评审](04_Security_Review.md)

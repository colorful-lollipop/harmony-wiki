# GN 构建

## 目的

本文档详细说明 CalendarData 组件的 GN 构建系统，包括 targets 列表、类型、依赖关系、配置选项和编译开关。

## 适用范围

- 目标读者：构建工程师、平台开发者
- 知识级别：中级到高级
- 前置知识：了解 GN 构建系统

## 构建文件结构

### 主构建文件

| 文件 | 说明 |
|------|------|
| `calendarmanager/BUILD.gn` | 主要 GN 构建文件 |
| `calendarmanager/cj/BUILD.gn` | CJ FFI 绑定构建 |
| `calendarmanager/calendardata.gni` | 配置变量 |
| `build-profile.json5` | 应用级构建配置 |
| `entry/build-profile.json5` | Entry 模块配置 |

## Targets 清单

### calendarmanager/BUILD.gn

#### 公共配置

```gn
config("calendarmanager_public_config") {
  include_dirs = [
    "./common",
    "./native/include",
    "./napi/include",
  ]
  configs = [ "//build/config/compiler:exceptions" ]
}
```

#### calendarmanager

**类型**: `ohos_shared_library`

**说明**: 主要的日历管理共享库，包含 N-API 绑定和 C++ 实现

**Sources**:

| 源文件 | 说明 |
|---------|------|
| `napi/src/calendar_enum_napi.cpp` | 枚举类型 N-API |
| `napi/src/calendar_manager_napi.cpp` | CalendarManager N-API |
| `napi/src/calendar_napi.cpp` | Calendar N-API |
| `napi/src/event_filter_napi.cpp` | EventFilter N-API |
| `napi/src/module_init.cpp` | 模块初始化 |
| `napi/src/module_register.cpp` | 模块注册 |
| `napi/src/napi_env.cpp` | N-API 环境 |
| `napi/src/napi_queue.cpp` | N-API 队列 |
| `napi/src/napi_util.cpp` | N-API 工具 |
| `native/src/calendar_env.cpp` | 日历环境 |
| `native/src/data_share_helper_manager.cpp` | DataShare 管理器 |
| `native/src/event_filter.cpp` | 事件筛选 |
| `native/src/native_calendar.cpp` | Native Calendar |
| `native/src/native_calendar_manager.cpp` | Native CalendarManager |
| `native/src/native_util.cpp` | Native 工具 |
| `native/src/report_hievent_manager.cpp` | HiEvent 报告 |

**Deps**:
- `:editor_abc`
- `:editor_js`

**External Deps**:
- `ability_base:want`
- `ability_base:zuri`
- `ability_runtime:ability_context_native`
- `ability_runtime:ability_manager`
- `ability_runtime:abilitykit_native`
- `ability_runtime:data_ability_helper`
- `ability_runtime:napi_base_context`
- `ability_runtime:napi_common`
- `access_token:libaccesstoken_sdk`
- `ace_engine:ace_uicontent`
- `c_utils:utils`
- `data_share:datashare_common`
- `data_share:datashare_consumer`
- `hilog:libhilog`
- `ipc:ipc_single`
- `napi:ace_napi`
- `hiappevent:hiappevent_innerapi` (条件依赖)

**Defines**:
- `DEVICE_USAGE_HIAPPEVENT_ENABLE` (条件)

**Config**:
- `:calendarmanager_public_config`

**输出**: `libcalendarmanager.z.so`

**安装路径**: `module/`

#### calendarmanager_static

**类型**: `ohos_static_library`

**说明**: 静态库版本，用于链接到其他组件

**Sources**: 同 calendarmanager（不包含 N-API 源文件）

**Deps**: 无

**External Deps**: 同 calendarmanager

**输出**: `libcalendarmanager_static.a`

#### editor_abc

**类型**: `es2abc_gen_abc`

**说明**: 将 JavaScript 编译为 ABC 字节码

**Input**: `js/editor.js`

**Output**: `editor.abc`

#### editor_js

**类型**: `gen_js_obj`

**说明**: 编译 JavaScript 对象文件

**Input**: `js/editor.js`

**Output**: `editor.o`

### calendarmanager/cj/BUILD.gn

#### cj_calendar_manager_ffi

**类型**: `ohos_shared_library`

**说明**: CJ FFI 绑定库

**Sources**:

| 源文件 | 说明 |
|---------|------|
| `../native/src/calendar_env.cpp` | 日历环境 |
| `src/cj_calendar_env.cpp` | CJ 日历环境 |
| `src/cj_data_share_helper_manager.cpp` | CJ DataShare 管理器 |
| `../native/src/event_filter.cpp` | 事件筛选 |
| `src/cj_native_calendar.cpp` | CJ Native Calendar |
| `src/cj_native_calendar_manager.cpp` | CJ Native CalendarManager |
| `src/cj_native_util.cpp` | CJ Native 工具 |
| `src/calendar_manager_ffi.cpp` | CalendarManager FFI |
| `src/cj_calendar.cpp` | CJ Calendar |
| `src/cj_calendar_manager.cpp` | CJ CalendarManager |
| `src/cj_event_filter.cpp` | CJ EventFilter |

**Include Dirs**:
- `../common`
- `../native/include`
- `include`

**External Deps**:
- `ability_base:want`
- `ability_base:zuri`
- `ability_runtime:ability_connect_callback_stub`
- `ability_runtime:ability_context_native`
- `ability_runtime:ability_manager`
- `ability_runtime:abilitykit_native`
- `ability_runtime:data_ability_helper`
- `ability_runtime:napi_base_context`
- `ability_runtime:napi_common`
- `access_token:libaccesstoken_sdk`
- `access_token:libprivacy_sdk`
- `ace_engine:ace_uicontent`
- `c_utils:utils`
- `data_share:datashare_common`
- `data_share:datashare_consumer`
- `hilog:libhilog`
- `ipc:ipc_single`
- `napi:ace_napi`
- `napi:cj_bind_ffi`
- `napi:cj_bind_native`

**输出**: `libcj_calendar_manager_ffi.z.so`

**安装路径**: `platformsdk/`

**Inner API Tags**: `["platformsdk"]`

## 配置变量

### device_usage_hiappevent_enabled

**位置**: calendarmanager/calendardata.gni

**说明**: 是否启用 hiappevent 支持

**类型**: boolean

**默认值**: `false`

**自动检测**:
```gn
declare_args() {
    device_usage_hiappevent_enabled = false
    if (defined(global_parts_info) &&
        defined(global_parts_info.hiviewdfx_hiappevent)) {
        device_usage_hiappevent_enabled = true
    }
}
```

**影响**:
- 当启用时，添加 `hiappevent:hiappevent_innerapi` 依赖
- 定义 `DEVICE_USAGE_HIAPPEVENT_ENABLE` 宏

## 依赖关系图

### calendarmanager 依赖

```
calendarmanager (ohos_shared_library)
    ├── editor_abc (es2abc_gen_abc)
    └── editor_js (gen_js_obj)

外部依赖:
    ├── ability_base:want
    ├── ability_base:zuri
    ├── ability_runtime:ability_context_native
    ├── ability_runtime:ability_manager
    ├── ability_runtime:abilitykit_native
    ├── ability_runtime:data_ability_helper
    ├── ability_runtime:napi_base_context
    ├── ability_runtime:napi_common
    ├── access_token:libaccesstoken_sdk
    ├── ace_engine:ace_uicontent
    ├── c_utils:utils
    ├── data_share:datashare_common
    ├── data_share:datashare_consumer
    ├── hilog:libhilog
    ├── ipc:ipc_single
    ├── napi:ace_napi
    └── hiappevent:hiappevent_innerapi [条件]
```

### cj_calendar_manager_ffi 依赖

```
cj_calendar_manager_ffi (ohos_shared_library)
    ├── ability_base:want
    ├── ability_base:zuri
    ├── ability_runtime:ability_connect_callback_stub
    ├── ability_runtime:ability_context_native
    ├── ability_runtime:ability_manager
    ├── ability_runtime:abilitykit_native
    ├── ability_runtime:data_ability_helper
    ├── ability_runtime:napi_base_context
    ├── ability_runtime:napi_common
    ├── access_token:libaccesstoken_sdk
    ├── access_token:libprivacy_sdk
    ├── ace_engine:ace_uicontent
    ├── c_utils:utils
    ├── data_share:datashare_common
    ├── data_share:datashare_consumer
    ├── hilog:libhilog
    ├── ipc:ipc_single
    ├── napi:ace_napi
    ├── napi:cj_bind_ffi
    └── napi:cj_bind_native
```

## 构建目标

### OpenHarmony 系统构建

```bash
./build.sh --product-name rk3568 --ccache --build-target calendar_data
```

**参数说明**:
- `--product-name`: 产品名称（如 rk3568）
- `--ccache`: 启用编译缓存
- `--build-target`: 构建目标（calendar_data）

### 组件定义

**位置**: bundle.json:12-62

**Service Group**:
```json
"service_group": [
  "//applications/standard/calendardata/calendarmanager:calendarmanager",
  "//applications/standard/calendardata/calendarmanager/cj:cj_calendar_manager_ffi"
]
```

**Inner Kits**:
```json
"inner_kits": [
  {
    "header": {
      "header_base": "//applications/standard/calendardata/calendarmanager/cj/include",
      "header_files": []
    },
    "name": "//applications/standard/calendardata/calendarmanager/cj:cj_calendar_manager_ffi"
  }
]
```

## 编译配置

### C++ 编译选项

**异常支持**:
```gn
configs = [ "//build/config/compiler:exceptions" ]
```

### 分支保护

**静态库**（calendarmanager_static）:
```gn
branch_protector_ret = "pac_ret"
```

### CFI（控制流完整性）

**静态库**:
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

## 常见构建问题

### 缺少权限错误

**症状**: 编译时报缺少权限错误

**解决**: 确保声明了正确的权限：
```json
"ohos.permission.READ_CALENDAR"
"ohos.permission.WRITE_CALENDAR"
"ohos.permission.READ_WHOLE_CALENDAR"
"ohos.permission.WRITE_WHOLE_CALENDAR"
```

### hiappevent 依赖错误

**症状**: 找不到 hiappevent 组件

**解决**:
1. 确保 `device_usage_hiappevent_enabled` 正确设置
2. 检查 `global_parts_info.hiviewdfx_hiappevent` 是否定义

### N-API 头文件未找到

**症状**: 找不到 `napi/native_api.h` 等头文件

**解决**:
1. 确保 `napi:ace_napi` 在 external_deps 中
2. 确保 include_dirs 包含正确的路径

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [编译产物](06_Build_Artifacts.md) - 产物清单和安装路径
- [架构说明](02_Architecture.md) - 详细架构和数据流

---

返回 [目录](SUMMARY.md) | [首页](README.md)

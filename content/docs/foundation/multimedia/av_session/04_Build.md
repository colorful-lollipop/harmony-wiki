# 构建配置 (GN)

## 构建入口

### 根配置文件

| 文件 | 说明 |
|------|------|
| `config.gni` | 全局编译开关 |
| `av_session_ohos_sources.gni` | 源码文件配置 |
| `bundle.json` | 组件配置 |
| `bluetooth_part.gni` | 蓝牙部件配置 |
| `castplus_cast_engine_part.gni` | Cast Engine 部件配置 |
| `efficiency_manager_part.gni` | 效率管理部件配置 |

### 编译开关 (config.gni)

```gn
declare_args() {
  # 默认关闭
  av_session_enable_start_stop_on_demand = false
  av_session_enable_input_redistribute = false

  # 可选功能
  multimedia_av_session_enable_background_audio_control = false
  multimedia_av_session_enable_trace_control = true
  multimedia_av_session_enable_sysevent_control = true
  multimedia_av_session_enable_hicollie = true

  # 条件编译
  multimedia_av_session_enable_data_object = false
  multimedia_av_session_enable_device_manager = false
  multimedia_av_session_enable_dsoftbus = false

  # 根据全局部件信息自动配置
  if (defined(global_parts_info) &&
      !defined(global_parts_info.hiviewdfx_hicollie)) {
    multimedia_av_session_enable_hicollie = false
  }
}
```

---

## 主要 Targets

### Framework 层 Targets

| Target | GN 路径 | 类型 | 输出 |
|--------|---------|------|------|
| `avsession_common` | `//frameworks/common` | static_library | `libavsession_common.z.a` |
| `ohavsession` | `//frameworks/native/ohavsession` | shared_library | `libohavsession.z.so` |
| `avsession_client` | `//frameworks/native/session` | shared_library | `libavsession_client.z.so` |
| `avsession_client_lite` | `//frameworks/native/session` | shared_library | `libavsession_client_lite.z.so` |
| `avsession_napi` | `//frameworks/js/napi/session` | shared_library | `libavsession_napi.z.so` |
| `cj_multimedia_avsession_ffi` | `//frameworks/cj` | shared_library | `libcj_multimedia_avsession_ffi.z.so` |
| `av_session_taihe` | `//frameworks/taihe` | shared_library | `libav_session_taihe.z.so` |

### UI 组件 Targets

| Target | GN 路径 | 类型 | 输出 |
|--------|---------|------|------|
| `avcastpicker` | `//avpicker` | shared_library | `libavcastpicker.z.so` |
| `avcastpickerparam` | `//avpicker` | shared_library | `libavcastpickerparam.z.so` |
| `avvolumepanel` | `//avvolumepanel` | shared_library | `libavvolumepanel.z.so` |
| `avinputcastpicker` | `//avinputcastpicker` | shared_library | `libavinputcastpicker.z.so` |

### 服务层 Targets

| Target | GN 路径 | 类型 | 输出 |
|--------|---------|------|------|
| `avsession_item` | `//services/session` | static_library | `libavsession_item.z.a` |
| `avsession_server` | `//services/session` | shared_library | `libavsession_server.z.so` |
| `remote_session_source` | `//services/session/server/remote` | shared_library | `libremote_session_source.z.so` |
| `remote_session_sink` | `//services/session/server/remote` | shared_library | `libremote_session_sink.z.so` |

### 工具库 Targets

| Target | GN 路径 | 类型 | 输出 |
|--------|---------|------|------|
| `avsession_utils` | `//utils` | static_library | `libavsession_utils.z.a` |

### 静态库变体

| Target | GN 路径 | 类型 | 用途 |
|--------|---------|------|------|
| `avpicker_static` | `//avpicker_static` | static_library | 静态链接变体 |
| `avinputcastpicker_static` | `//avinputcastpicker_static` | static_library | 静态链接变体 |
| `avvolumepanel_static` | `//avvolumepanel_static` | static_library | 静态链接变体 |

---

## Target 详细配置

### frameworks/common/BUILD.gn

```gn
static_library("avsession_common") {
  sources = [
    "src/avcall_meta_data.cpp",
    "src/avmeta_data.cpp",
    "src/avplayback_state.cpp",
    "src/avcontrol_command.cpp",
    # ... 更多文件
  ]

  include_dirs = [
    "include",
    "$ohos_root_path/interfaces/inner_api/native/session/include",
    "$ohos_root_path/utils/include",
  ]

  deps = [
    "$ohos_root_path/utils/native/avsession:avsession_utils",
    "$ohos_root_path/interfaces/inner_api/native/session:avsession_info",
  ]

  public_deps = [
    "//utils/native/avsession:avsession_utils",
    "//interfaces/inner_api/native/session:avsession_info",
  ]
}
```

### frameworks/native/session/BUILD.gn

```gn
shared_library("avsession_client") {
  sources = [
    "src/avsession_manager.cpp",
    "src/avsession_manager_impl.cpp",
    "src/avsession_callback_client.cpp",
    # ... 更多文件
  ]

  include_dirs = [
    "include",
    "$ohos_root_path/interfaces/inner_api/native/session/include",
  ]

  deps = [
    ":avsession_client_lite",
    "//frameworks/common:avsession_common",
    "//utils/native/avsession:avsession_utils",
  ]
}

shared_library("avsession_client_lite") {
  # 轻量版实现
  sources = [
    "src/avsession_manager_lite.cpp",
    # ... 更多文件
  ]

  exclude_sources = [
    "src/avsession_callback_client.cpp",
  ]
}
```

### services/session/BUILD.gn

```gn
shared_library("avsession_server") {
  sources = [
    "server/avsession_service.cpp",
    "server/avsession_item.cpp",
    "server/avcontroller_item.cpp",
    "server/avcast_controller_item.cpp",
    "ipc/stub/*.cpp",
    "ipc/proxy/*.cpp",
    # ... 更多文件
  ]

  include_dirs = [
    "server/",
    "ipc/base/",
    "$ohos_root_path/interfaces/inner_api/native/session/include",
  ]

  deps = [
    "//frameworks/common:avsession_common",
    "//utils/native/avsession:avsession_utils",
    "//services/session:avsession_item",
  ]

  # SA 服务配置
  subsystem_name = "multimedia"
  part_name = "av_session"
}
```

---

## 依赖关系

### 框架层依赖

```
avsession_napi
├── napi (系统依赖)
├── avsession_client
│   ├── avsession_client_lite
│   ├── avsession_common
│   │   ├── avsession_utils
│   │   └── interfaces (inner_api)
│   └── utils
├── ohavsession (可选)
└── av_session_taihe (可选)

cj_multimedia_avsession_ffi
├── avsession_client
└── FFI 框架
```

### 服务层依赖

```
avsession_server
├── avsession_item
│   ├── avsession_common
│   └── utils
├── avsession_client
├── remote_session_source (可选)
│   ├── cast_engine (可选)
│   └── dsoftbus (可选)
└── remote_session_sink (可选)
```

---

## 编译产物

### 动态库 (.so)

| 产物 | 预计路径 | 说明 |
|------|----------|------|
| `libavsession_napi.z.so` | `out/.../libs/` | N-API 接口 |
| `libavsession_client.z.so` | `out/.../libs/` | Native 客户端 |
| `libohavsession.z.so` | `out/.../libs/` | OH AVSession C API |
| `libavsession_server.z.so` | `system/lib/` | SA 服务 |
| `libavsession_common.z.so` | `system/lib/` | 公共框架 |
| `libavsession_utils.z.so` | `system/lib/` | 工具库 |

### 静态库 (.a)

| 产物 | 预计路径 | 说明 |
|------|----------|------|
| `libavsession_common.z.a` | `out/.../obj/` | 静态链接变体 |
| `libavsession_utils.z.a` | `out/.../obj/` | 工具静态库 |
| `libavsession_item.z.a` | `out/.../obj/` | 服务项 |

### SA 服务产物

| 产物 | 预计路径 | 说明 |
|------|----------|------|
| `libavsession_service.z.so` | `system/lib/` | SA 服务实现 |
| `av_session.json` | `etc/sa_profile/` | SA 配置文件 |
| `avsession_service.cfg` | `etc/` | 权限配置 |

---

## 安装路径

### 系统库

```
/system/lib/
├── libavsession_server.z.so      # SA 服务
├── libavsession_common.z.so       # 公共框架
├── libavsession_utils.z.so        # 工具库
└── modules/
    └── libavsession_napi.z.so     # N-API 模块
```

### SA 配置

```
/system/sa_profile/
└── av_session.json                # SA ID 3010

/etc/
└── avsession_service.cfg          # 服务权限配置
```

### 设备端产物

```
/data/service/el2/101/
└── avsession/                     # 服务运行时数据
```

---

## 构建命令

### 全量构建

```bash
# 编译整个 multimedia 子系统
./build.sh --product_name <product> --subsystem multimedia

# 仅编译 av_session 部件
./build.sh --product_name <product> --parts av_session
```

### 单模块构建

```bash
# 编译 N-API 模块
hb build -p av_session -f frameworks/js/napi/session

# 编译 SA 服务
hb build -p av_session -f services/session

# 编译工具库
hb build -p av_session -f utils
```

### 查看依赖

```bash
# 查看 target 依赖
gn deps out/.../all_dependent_modules
```

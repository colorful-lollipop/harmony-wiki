# GN 构建配置

## 构建入口

**文件**: `BUILD.gn`

## Target 概览

| Target | 类型 | 输出 | 描述 |
|--------|------|------|------|
| `safwk_lite` | lite_component | - | 组件封装 |
| `foundation` | executable | `foundation` | 主可执行文件 |

## 根构建配置

**代码证据**: `BUILD.gn:16-21`

```gn
declare_args() {
  enable_timertask = false
  safwk_lite_feature_enable_abilityms = true
  safwk_lite_feature_enable_bundlems = true
  safwk_lite_feature_enable_dtbschedmgr = true
}
```

### 全局参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enable_timertask` | bool | false | 启用定时任务 |
| `safwk_lite_feature_enable_abilityms` | bool | true | 启用 AbilityMS |
| `safwk_lite_feature_enable_bundlems` | bool | true | 启用 BundleMS |
| `safwk_lite_feature_enable_dtbschedmgr` | bool | true | 启用 DMSchedMgr |

## 组件定义

**代码证据**: `BUILD.gn:23-26`

```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  lite_component("safwk_lite") {
    features = [ ":foundation" ]
  }
}
```

**编译条件**: 仅在 LiteOS-A 或 Linux 内核上编译

## foundation Target

### 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | executable |
| 输出名 | `foundation` |

### 源文件

**代码证据**: `BUILD.gn:33`

```gn
sources = [ "src/main.c" ]
```

### 编译标志

**代码证据**: `BUILD.gn:30-31`

```gn
cflags = [ "-Wall" ]
cflags_cc = cflags
```

### 包含目录

**代码证据**: `BUILD.gn:35-38`

```gn
include_dirs = [
  "//commonlibrary/utils_lite/include",
  "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
]
```

### 链接标志

**代码证据**: `BUILD.gn:40-43`

```gn
ldflags = [
  "-lstdc++",
  "-Wl,-Map=foundation.map",
]
```

### 依赖配置

#### 基础依赖

**代码证据**: `BUILD.gn:45-49`

```gn
deps = [
  "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
  "//base/security/permission_lite/services/ipc_auth:ipc_auth_target",
  "//base/security/permission_lite/services/pms:pms_target",
  "//foundation/systemabilitymgr/samgr_lite/samgr_server:server",
]
```

| 依赖 | 类型 | 作用 |
|------|------|------|
| `hilog_shared` | external | 日志输出 |
| `ipc_auth_target` | external | IPC 权限认证 |
| `pms_target` | external | 权限管理服务 |
| `samgr_server` | external | Samgr 服务端 |

#### 条件依赖 - Timer Task

**代码证据**: `BUILD.gn:51-53`

```gn
if (enable_timertask == true) {
  deps += [ "//base/update/dupdate/services/timertask_lite:timertask" ]
}
```

#### 条件依赖 - AbilityMS

**代码证据**: `BUILD.gn:54-56`

```gn
if (safwk_lite_feature_enable_abilityms == true) {
  deps += [ "${aafwk_lite_path}/services/abilitymgr_lite:abilityms" ]
}
```

#### 条件依赖 - BundleMS

**代码证据**: `BUILD.gn:57-59`

```gn
if (safwk_lite_feature_enable_bundlems == true) {
  deps += [ "${appexecfwk_lite_path}/services/bundlemgr_lite:bundlems" ]
}
```

#### 条件依赖 - DMSchedMgr

**代码证据**: `BUILD.gn:60-64`

```gn
if (board_name != "hispark_aries") {
  if (safwk_lite_feature_enable_dtbschedmgr == true) {
    deps += [ "//foundation/ability/dmsfwk_lite:dtbschedmgr" ]
  }
}
```

**注意**: `dtbschedmgr` 在 `hispark_aries` 开发板上被排除。

## 依赖图

```
safwk_lite
└── foundation
    ├── hilog_shared
    ├── ipc_auth_target
    ├── pms_target
    ├── samgr_server
    ├── abilityms (条件)
    ├── bundlems (条件)
    ├── dtbschedmgr (条件，非 hispark_aries)
    └── timertask (条件)
```

## 编译命令示例

### 全量编译

```bash
./build.sh --product-name xxx --build-target safwk_lite
```

### 仅编译 safwk_lite

```bash
hb build safwk_lite
```

### 带调试日志编译

```bash
hb build safwk_lite -D DEBUG_SERVICES_SAFWK_LITE=true
```

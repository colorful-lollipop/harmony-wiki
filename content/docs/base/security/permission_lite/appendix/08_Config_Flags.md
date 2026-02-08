# 配置参数与宏定义

## 概述

本文档记录 Permission Lite 编译构建相关的宏定义、Feature Flags 和配置参数。

---

## 编译宏定义

### 系统功能宏

| 宏定义 | 定义位置 | 作用 | 默认值 |
|--------|----------|------|--------|
| `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` | `services/ipc_auth/BUILD.gn:52` | 启用 Bundle Manager 集成 | 未定义 |
| `OHOS_APPFWK_ENABLE` | `services/ipc_auth/BUILD.gn:53` | 启用应用框架集成 | 未定义 |

**说明**：这两个宏仅在 `ohos_kernel_type == "liteos_a"` 时定义

---

### 权限长度限制

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `PERM_NAME_LEN` | 64 | 权限名称最大长度 |
| `PERM_DESC_LEN` | 256 | 权限描述最大长度 |
| `PKG_NAME_LEN` | 64 | 包名最大长度 |

**代码证据**：`services/pms/include/perm_define.h`

```c
#define PKG_NAME_LEN 64
```

---

### 其他常量

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `CAP_NOT_BINDED` | -1 | 能力未绑定 |
| `FIXED_UID_MAX` | 8 | 固定 UID 列表最大数量 |

---

## Feature Flags

### 构建类型

| Flag | 位置 | 说明 |
|------|------|------|
| `os_level` | `services/BUILD.gn` | 系统级别（mini/small） |
| `ohos_kernel_type` | `services/ipc_auth/BUILD.gn` | 内核类型（liteos_a 等） |
| `ohos_build_type` | `services/BUILD.gn` | 构建类型（debug/release） |

---

### Debug 构建

```gn
if (ohos_build_type == "debug") {
  features += [ "unittest:unittest" ]
}
```

**说明**：Debug 构建包含单元测试模块

---

## 构建配置变量

### os_level（系统级别）

| 值 | 系统 | 产物类型 |
|----|------|----------|
| `"mini"` | Mini System | 静态库 (.a) |
| 非 `"mini"` | Small System | 动态库 (.so) |

**代码证据**：`services/BUILD.gn:17`

---

### ohos_kernel_type（内核类型）

| 值 | 说明 |
|-----|------|
| `"liteos_a"` | 轻量级内核 A |
| 其他 | 标准内核 |

**条件编译**：liteos_a 时启用额外依赖

```gn
if (ohos_kernel_type == "liteos_a") {
  include_dirs += [ ... ]
  deps += [ ... ]
  defines += [ ... ]
}
```

---

### 路径变量

| 变量 | 用途 |
|------|------|
| `permission_lite_path` | Permission Lite 根路径 |
| `permission_lite_pms_hal_path` | PMS HAL 层路径 |
| `ohos_product_adapter_dir` | 产品适配目录 |
| `ipc_path` | IPC 框架路径 |
| `samgr_lite_path` | SAMGR 路径 |
| `hilog_lite_path` | HiLog 路径 |

**代码证据**：`build/config.gni`

---

## 组件配置

### NDK 配置

```gn
ndk_lib("permission_notes") {
  lib_extension = ".so"
  deps = [ "${permission_lite_path}/services/pms_client:pms_client" ]
  head_files = [ "${permission_lite_path}/interfaces/kits" ]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `lib_extension` | `.so` | 库文件扩展名 |
| `deps` | pms_client | 依赖组件 |
| `head_files` | interfaces/kits | 头文件目录 |

---

### 静态库配置（Mini）

```gn
ohos_static_library("pms_target_static") {
  subsystem_name = "permission_lite"
  part_name = "permission_lite"
  sources = [...]
  include_dirs = [...]
  deps = [...]
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `subsystem_name` | `"permission_lite"` | 子系统名 |
| `part_name` | `"permission_lite"` | 组件名 |

---

## HAL 层配置

### HAL 路径

| 组件 | 变量 |
|------|------|
| PMS HAL | `permission_lite_pms_hal_path:hal_pms` (Small) |
| PMS HAL | `permission_lite_pms_hal_path:hal_pms_static` (Mini) |

---

## 服务配置

### SAMGR Feature 名称

| 服务 | 常量定义 |
|------|----------|
| Permission Service | `PERMISSION_SERVICE = "permissionms"` |
| Inner Feature | `PERM_INNER = "PermInnerFeature"` |

**代码证据**：`services/pms_base/include/permission_service.h`

---

## 日志配置

### 模块名

| 模块 | 宏定义 |
|------|--------|
| APP | `HILOG_MODULE_APP` |

**代码来源**：参考 `README.md` 中的日志示例

---

## 权限配置

### 系统权限列表

**代码证据**：`services/unittest/pms/src/acts_pms_test.h`

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.INSTALL_BUNDLE` | 安装应用 |
| `ohos.permission.LISTEN_BUNDLE_CHANGE` | 监听 Bundle 变化 |
| `ohos.permission.GET_BUNDLE_INFO` | 获取 Bundle 信息 |

---

### 测试用权限

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.TEST` | 单元测试 |

---

## 相关文档

- 构建配置 → `04_Build.md`
- 安全评审 → `05_Security.md`
- 内部 API → `appendix/06_Inner_APIs.md`

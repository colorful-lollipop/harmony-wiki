# 附录：配置开关

> utils_lite 的关键宏定义与 Feature Flags。

## Feature Flags

### 构建时 Feature

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `utils_lite_feature_file` | bool | false | 启用文件操作模块 |
| `utils_lite_feature_kal_timer` | bool | false | 启用 KAL 定时器模块 |
| `utils_lite_feature_timer_task` | bool | false | 启用定时器任务模块 |
| `utils_lite_feature_js_builtin` | bool | false | 启用 JS 内置 API 模块 |

**定义位置**：`BUILD.gn:16-21`

**使用方法**：
```gn
# 在 GN args 中启用
utils_lite_feature_file = true
utils_lite_feature_js_builtin = true
```

---

### SDK 构建

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `build_ohos_sdk` | bool | false | 启用 SDK 构建（包含模拟器） |

**定义位置**：`js/builtin/simulator/BUILD.gn:19`

**影响**：构建 SDK 模拟器库供预览器使用

---

## 平台条件

### 内核类型

| 配置项 | 适用值 | 说明 |
|--------|--------|------|
| `ohos_kernel_type` | "liteos_m" / "liteos_a" | 目标内核类型 |

**使用示例**：
```gn
if (ohos_kernel_type == "liteos_m") {
  target_type = "static_library"
} else {
  target_type = "shared_library"
}
```

---

## KV Store 配置

### 功能开关

| 宏 | 说明 |
|-----|------|
| `FEATURE_KV_CACHE` | 启用 KV 缓存功能 |

**定义位置**：`include/kv_store.h:95`

**使用条件编译**：
```c
#ifdef FEATURE_KV_CACHE
int ClearKVCache(void);
#endif
```

---

### 限制配置

| 配置 | 默认值 | 说明 |
|------|--------|------|
| `MAX_CACHE_SIZE` | 10 | 缓存条目数最大值 |
| `MAX_KV_SUM` | 50 | 每应用最大 KV 对数 |

**证据来源**：`include/kv_store.h:24-26`

```
If {@link FEATURE_KV_CACHE} is enabled, key-value pairs can be stored in the cache.
For details about cache specifications, see {@link MAX_CACHE_SIZE}.
For details about the number of key-value pairs that can be stored in an application, see {@link MAX_KV_SUM}.
```

---

## 文件系统限制

### SPIFFS 限制（liteos_m）

| 限制 | 值 | 说明 |
|------|------|------|
| 文件名最大长度 | 32 字节 | 含结束符 |
| 最大打开文件数 | 32 | 同时打开 |
| 多级目录 | 不支持 | 仅单级目录 |

**证据来源**：`include/utils_file.h:25-31`

---

## SDK 模拟器配置

### 编译器配置

```gn
# js/builtin/simulator/BUILD.gn:61-72
config("storage_config") {
  cflags = [
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECTMP=//",
    "-D_STDIO_S_DEFINED",
    "-Wno-error",
  ]
}
```

### 可见性配置

```gn
# js/builtin/simulator/BUILD.gn:21-24
visibility = [
  ":*",
  "//ide/tools/previewer/mock/*",
]
```

---

## NDK 头文件

| 头文件 | 包含条件 | 说明 |
|--------|----------|------|
| `utils_config.h` | always | 配置宏定义 |
| `utils_file.h` | liteos_m | 文件操作 API |
| `kv_store.h` | - | KV 存储 API |
| `utils_list.h` | - | 链表 API |
| `ohos_types.h` | - | 类型定义 |
| `ohos_errno.h` | - | 错误码 |
| `ohos_init.h` | - | 初始化框架 |

**证据来源**：`BUILD.gn:35-39`

---

## 外部依赖

| 依赖组件 | 用途 | 模块 |
|----------|------|------|
| `bounds_checking_function:libsec_shared` | 安全函数 | filekit, kvstorekit |
| `init:libbegetutil` | 工具库 | deviceinfokit |
| `hmos_spiffs` | SPIFFS 文件系统 | file (liteos_m) |

**证据来源**：
- `js/builtin/filekit/BUILD.gn:38`
- `js/builtin/kvstorekit/BUILD.gn:38`
- `js/builtin/deviceinfokit/BUILD.gn:40`
- `file/BUILD.gn:26-28`

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [GN 构建](05_GN_Build.md) - 构建配置
- [编译产物](06_Build_Artifacts.md) - 产物清单
- [N-API 参考](03_NAPI_Reference.md) - JS API
- [安全评审](07_Security_Review.md) - 安全配置

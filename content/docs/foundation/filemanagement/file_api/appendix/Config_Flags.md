# 附录 B: 配置标志

## Feature Flags

### file_api_read_optimize

**描述**: 穿戴设备文件读优化

**类型**: Boolean

**默认值**: `false`

**配置位置**: `file_api.gni:23`

```gn
declare_args() {
    file_api_read_optimize = false
}
```

**影响代码**:
- `interfaces/kits/js/BUILD.gn:387-389`

```gn
if (file_api_read_optimize) {
    defines = [ "WEARABLE_PRODUCT" ]
}
```

**影响模块**: `file` (ohos_shared_library)

**说明**: 启用后定义 `WEARABLE_PRODUCT` 宏，用于针对穿戴设备的特殊优化。

### file_api_feature_hyperaio

**描述**: 启用 HyperAIO 高性能异步 IO (基于 io_uring)

**类型**: Boolean

**默认值**: `false`

**配置位置**: `file_api.gni:24`

```gn
declare_args() {
    file_api_feature_hyperaio = false
}
```

**影响代码**:
- `interfaces/kits/hyperaio/BUILD.gn`

```gn
group("group_hyperaio") {
    deps = []
    if (file_api_feature_hyperaio) {
        deps += [ ":HyperAio" ]
    }
}

ohos_shared_library("HyperAio") {
    if (file_api_feature_hyperaio) {
        external_deps += [ "liburing:liburing" ]
        defines = [ "HYPERAIO_USE_LIBURING" ]
    }
}
```

**影响模块**: `HyperAio` (ohos_shared_library)

**说明**: 启用后：
1. 构建 `HyperAio` 目标
2. 链接 `liburing` 库
3. 定义 `HYPERAIO_USE_LIBURING` 宏

**使用要求**:
- 需要 `ohos.permission.ALLOW_IOURING` 权限
- 仅支持 Linux 内核 5.1+ (io_uring 支持)

## 编译宏定义

### 平台相关宏

| 宏 | 定义条件 | 说明 | 使用位置 |
|----|----------|------|----------|
| `WIN_PLATFORM` | `use_mingw_win` | Windows 平台 | 多处 |
| `IOS_PLATFORM` | `use_mac` | iOS/macOS 平台 | 多处 |
| `FILE_API_TRACE` | `!use_mingw_win && !use_mac` | 启用追踪 | `mod_fs/module.cpp` |

**代码示例** (`interfaces/kits/js/src/mod_fs/module.cpp:23-36`):
```cpp
#if !defined(WIN_PLATFORM) && !defined(IOS_PLATFORM)
#include "class_atomicfile/atomicfile_n_exporter.h"
#include "class_tasksignal/task_signal_n_exporter.h"
// ... 其他仅 OHOS 支持的类
#endif
```

### 功能相关宏

| 宏 | 定义条件 | 说明 | 使用位置 |
|----|----------|------|----------|
| `WEARABLE_PRODUCT` | `file_api_read_optimize` | 穿戴设备优化 | `mod_file/module.cpp` |
| `HYPERAIO_USE_LIBURING` | `file_api_feature_hyperaio` | 启用 io_uring | `hyperaio.cpp` |
| `OPENSSL_SUPPRESS_DEPRECATED` | 始终 | OpenSSL 兼容性 | `mod_fileio`, `mod_hash` |

### 调试相关宏

| 宏 | 默认值 | 说明 | 设置方式 |
|----|--------|------|----------|
| `FILE_API_DEBUG` | undefined | 调试日志 | 编译时定义 |
| `FILE_API_TRACE` | 见上 | 性能追踪 | 平台自动设置 |

**代码示例** (`interfaces/kits/js/src/mod_fs/module.cpp:85-87`):
```cpp
FileApiDebug::isLogEnabled = GetParaBool("param.key.fileapi.debug.log");
FileApiDebug::isTraceEnhanced = GetParaBool("param.key.fileapi.debug.trace");
```

## 运行时参数

### 系统参数

| 参数名 | 类型 | 说明 | 默认值 |
|--------|------|------|--------|
| `param.key.fileapi.debug.log` | bool | 启用调试日志 | `false` |
| `param.key.fileapi.debug.trace` | bool | 启用增强追踪 | `false` |

**设置方式**:
```bash
# 设置参数
param set param.key.fileapi.debug.log true
param set param.key.fileapi.debug.trace true

# 查询参数
param get param.key.fileapi.debug.log
```

**代码实现** (`interfaces/kits/js/src/mod_fs/module.cpp:44-52`):
```cpp
static bool GetParaBool(const char* key) {
    char value[] = "false";
    int ret = GetParameter(key, "false", value, sizeof(value));
    return (ret > 0 && !std::strcmp(value, "true"));
}
```

## 编译选项

### 安全加固选项

| 选项 | 值 | 说明 |
|------|-----|------|
| `branch_protector_ret` | `"pac_ret"` | ARM64 指针认证 |
| `integer_overflow` | `true` | 整数溢出检测 |
| `ubsan` | `true` | 未定义行为检测 |
| `boundary_sanitize` | `true` | 边界检测 |
| `cfi` | `true` | 控制流完整性 |
| `cfi_cross_dso` | `true` | 跨 DSO CFI |

**配置位置** (`interfaces/kits/js/BUILD.gn:50-57`):
```gn
sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
}
```

### 优化选项

| 选项 | 值 | 说明 |
|------|-----|------|
| `-fvisibility=hidden` | cflags | 隐藏符号 |
| `-fdata-sections` | cflags | 数据段分离 |
| `-ffunction-sections` | cflags | 函数段分离 |
| `-Oz` | cflags | 最小化代码大小 |
| `-std=c++17` | cflags_cc | C++17 标准 |

## 权限配置

### 需要申请的权限

| 功能 | 权限 | 类型 | 说明 |
|------|------|------|------|
| HyperAIO | `ohos.permission.ALLOW_IOURING` | system_grant | 使用 io_uring |
| 系统目录 | `ohos.permission.FILE_ACCESS_MANAGER` | system_grant | 访问系统目录 |
| 安全标签 | `ohos.permission.FILE_ACCESS_MANAGER` | system_grant | 修改安全标签 |

**权限检查代码**:
```cpp
// interfaces/kits/hyperaio/src/hyperaio.cpp:40-51
bool HasAccessIouringPermission() {
    Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
    int res = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenCaller, "ohos.permission.ALLOW_IOURING");
    return res == Security::AccessToken::PermissionState::PERMISSION_GRANTED;
}
```

## Syscap 配置

### bundle.json 中的 syscap

**文件**: `bundle.json:15-21`

```json
"syscap": [
    "SystemCapability.FileManagement.File.FileIO",
    "SystemCapability.FileManagement.File.FileIO.Lite",
    "SystemCapability.FileManagement.File.Environment",
    "SystemCapability.FileManagement.File.DistributedFile",
    "SystemCapability.FileManagement.File.Environment.FolderObtain"
]
```

### syscap 对应功能

| Syscap | 说明 | 使用场景 |
|--------|------|----------|
| `FileIO` | 文件 IO 基础能力 | fs, fileio, hash, statvfs |
| `FileIO.Lite` | 轻量文件 IO | 小型设备 |
| `Environment` | 环境目录 | environment 模块 |
| `DistributedFile` | 分布式文件 | 分布式场景 |
| `Environment.FolderObtain` | 目录获取 | 扩展目录接口 |

## 配置检查清单

### 添加新 Feature Flag

1. 在 `file_api.gni` 中声明
   ```gn
   declare_args() {
       my_new_feature = false
   }
   ```

2. 在 BUILD.gn 中使用
   ```gn
   if (my_new_feature) {
       defines = [ "MY_NEW_FEATURE" ]
   }
   ```

3. 在代码中条件编译
   ```cpp
   #ifdef MY_NEW_FEATURE
       // 新功能代码
   #endif
   ```

4. 在 `bundle.json` 中注册 feature
   ```json
   "features": [
       "my_new_feature"
   ]
   ```

5. 更新本文档

### 配置验证

```bash
# 查看所有参数
gn args out --list

# 查看特定参数
gn args out --list | grep file_api

# 生成时设置参数
gn gen out --args='file_api_feature_hyperaio=true file_api_read_optimize=true'
```

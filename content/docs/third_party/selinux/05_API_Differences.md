# API/接口差异

## 概述

由于本库没有传统 Patch 文件修改现有 API，而是通过条件编译和新增模块实现定制，因此 API 差异分为两类：

1. **行为差异 API** - 相同 API 在 OH 下行为不同
2. **OH 新增 API** - 完全新增的 OH 特有接口

---

## 行为差异 API

### selabel_open()

**函数签名**（无变化）：
```c
struct selabel_handle *selabel_open(unsigned int backend,
                                    const struct selinux_opt *opts,
                                    unsigned nopts);
```

**行为差异**：

| 场景 | 上游行为 | OH 行为（启用 OHOS_FC_INIT） |
|-----|---------|---------------------------|
| SELABEL_OPT_PATH | 只处理第一个 PATH 选项 | 处理所有 PATH 选项 |
| 内存分配 | 分配单个字符串 | 分配字符串数组 |

**使用示例**：

```c
// 上游用法（单文件）
struct selinux_opt opts[] = {
    { SELABEL_OPT_PATH, "/system/etc/file_contexts" }
};
handle = selabel_open(SELABEL_CTX_FILE, opts, 1);

// OH 用法（多文件）
struct selinux_opt opts[] = {
    { SELABEL_OPT_PATH, "/system/etc/file_contexts" },
    { SELABEL_OPT_PATH, "/vendor/etc/file_contexts" },
    { SELABEL_OPT_PATH, "/product/etc/file_contexts" }
};
handle = selabel_open(SELABEL_CTX_FILE, opts, 3);
```

### selabel_close()

**行为差异**：

| 场景 | 上游行为 | OH 行为 |
|-----|---------|--------|
| 内存释放 | 释放单个字符串 | 遍历释放字符串数组 |

**实现差异**（详见 02_Patches.md）：

```c
// 上游
free(rec->spec_file);

// OH
for (int i = 0; i < rec->spec_file_nums; i++) {
    free(rec->spec_file[i]);
}
free(rec->spec_file);
```

---

## OH 新增 API

### app_allow_config 模块

#### load_app_allow_config()

**功能**：从配置文件加载应用白名单

**函数签名**：
```c
void load_app_allow_config();
```

**配置文件**：`/system/etc/selinux/app_allow_cfg`

**文件格式**：
```
# 每行一个路径
/data/app/com.example.app
/data/app/com.vendor.app
/storage/emulated/0/Android
```

**使用时机**：通常在 `selinux_restorecon()` 之前调用一次

**示例**：
```c
#include "app_allow_config.h"

// 初始化时加载
void init_selinux() {
    load_app_allow_config();
    // ...
}
```

#### is_in_app_allow_config()

**功能**：检查路径是否在白名单中

**函数签名**：
```c
bool is_in_app_allow_config(const char *pathname);
```

**参数**：
- `pathname` - 要检查的文件路径

**返回值**：
- `true` - 路径在白名单中（应跳过 restorecon）
- `false` - 路径不在白名单中

**匹配规则**：前缀匹配
```c
// 如果白名单包含 "/data/app/com.example.app"
// 则以下路径返回 true:
//   /data/app/com.example.app
//   /data/app/com.example.app/files
//   /data/app/com.example.app/cache/data.txt
```

**示例**：
```c
void process_path(const char *path) {
    if (is_in_app_allow_config(path)) {
        // 跳过 restorecon
        return;
    }
    // 执行 restorecon
    selinux_restorecon(path, ...);
}
```

#### insert_line_to_app_allow_config()

**功能**：向白名单插入路径（内部使用）

**函数签名**：
```c
static bool insert_line_to_app_allow_config(const char *line);
```

**说明**：
- 内部函数，通常由 `load_app_allow_config()` 调用
- 动态分配内存扩容数组
- 自动去除行尾换行符和斜杠

---

## 数据结构差异

### selabel_handle

**结构体差异**（由 `OHOS_FC_INIT` 宏控制）：

```c
#ifndef OHOS_FC_INIT
    // 上游版本
    char *spec_file;
#else
    // OH 版本
    char **spec_file;
    size_t spec_file_nums;
#endif
```

**影响**：
- 外部代码不应直接访问 `rec->spec_file`
- 应使用 `selabel_open()` / `selabel_close()` 管理生命周期

### 新增常量

**文件路径常量**：
```c
// app_allow_config.c
#define SYSTEM_APP_ALLOW_CONFIG_PATH "/system/etc/selinux/app_allow_cfg"
```

---

## 使用建议

### 对于 OH 系统开发者

1. **优先使用标准 API**
   - `selabel_open()` / `selabel_close()`
   - `selabel_lookup()`
   - `setfilecon()` / `getfilecon()`

2. **如需白名单功能**
   ```c
   #include "app_allow_config.h"
   
   // 初始化时加载配置
   load_app_allow_config();
   
   // 使用时检查
   if (!is_in_app_allow_config(path)) {
       // 执行操作
   }
   ```

3. **多文件 contexts 使用**
   ```c
   struct selinux_opt opts[] = {
       { SELABEL_OPT_PATH, path1 },
       { SELABEL_OPT_PATH, path2 },
       // ... 更多路径
   };
   handle = selabel_open(SELABEL_CTX_FILE, opts, n);
   ```

### 对于从上游移植的代码

1. **检查 `OHOS_FC_INIT` 宏**
   - 如果代码直接访问 `rec->spec_file`，需要适配
   - 建议改为使用标准 API

2. **白名单功能可选**
   - 如果不需要白名单，可不调用相关 API
   - 不影响标准 SELinux 功能

---

## 兼容性说明

### 向前兼容（上游 → OH）

✅ **兼容**：所有上游代码无需修改即可在 OH 编译

原因：
- `OHOS_FC_INIT` 是新增宏，未定义时保持上游行为
- 新增 API 是独立模块，不影响现有代码

### 向后兼容（OH → 上游）

⚠️ **需注意**：使用了 OH 特有功能的代码无法直接移植到上游

需要修改：
- 移除 `app_allow_config` 相关调用
- 修改多文件 contexts 逻辑

---

## API 对照表

| API | 上游 | OH | 差异类型 |
|-----|-----|-----|---------|
| selabel_open() | ✅ | ✅ | 行为差异（多文件支持） |
| selabel_close() | ✅ | ✅ | 行为差异（内存释放） |
| selabel_lookup() | ✅ | ✅ | 无差异 |
| setfilecon() | ✅ | ✅ | 无差异 |
| getfilecon() | ✅ | ✅ | 无差异 |
| load_app_allow_config() | ❌ | ✅ | OH 新增 |
| is_in_app_allow_config() | ❌ | ✅ | OH 新增 |

---

## 示例代码

### 示例 1：标准文件标签查询

```c
#include <selinux/label.h>
#include <selinux/selinux.h>

void label_example() {
    struct selabel_handle *handle;
    char *context;
    
    // 打开 handle（OH 支持多文件）
    struct selinux_opt opts[] = {
        { SELABEL_OPT_PATH, "/system/etc/file_contexts" },
        { SELABEL_OPT_PATH, "/vendor/etc/file_contexts" },
    };
    handle = selabel_open(SELABEL_CTX_FILE, opts, 2);
    if (!handle) {
        // 错误处理
        return;
    }
    
    // 查询标签
    if (selabel_lookup(handle, &context, "/data/app/test", 0) == 0) {
        printf("Context: %s\n", context);
        freecon(context);
    }
    
    // 关闭 handle
    selabel_close(handle);
}
```

### 示例 2：使用白名单的 restorecon

```c
#include "app_allow_config.h"
#include <selinux/restorecon.h>

void restorecon_with_whitelist() {
    // 加载白名单
    load_app_allow_config();
    
    // 遍历目录
    DIR *dir = opendir("/data/app");
    struct dirent *entry;
    
    while ((entry = readdir(dir)) != NULL) {
        char path[PATH_MAX];
        snprintf(path, sizeof(path), "/data/app/%s", entry->d_name);
        
        // 检查白名单
        if (is_in_app_allow_config(path)) {
            printf("Skipping whitelisted path: %s\n", path);
            continue;
        }
        
        // 执行 restorecon
        selinux_restorecon(path, SELINUX_RESTORECON_RECURSE);
    }
    
    closedir(dir);
}
```


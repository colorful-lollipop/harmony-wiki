# Patch 详细分析

## Patch 清单表

> **重要说明**：本库**没有独立的 Patch 文件**（`.patch` 格式）。所有 OpenHarmony 特有修改通过以下方式实现：
> 1. 条件编译宏（`OHOS_FC_INIT`）
> 2. 新增源文件（`app_allow_config.c/h`）
> 3. BUILD.gn 构建配置

| 修改类型 | 涉及文件 | 修改目的 | 关联的 OH 需求 |
|---------|---------|---------|---------------|
| 条件编译 | `libselinux/src/label_internal.h` | 支持多文件 file_contexts | OHOS_FC_INIT |
| 条件编译 | `libselinux/src/label.c` | 多文件内存管理 | OHOS_FC_INIT |
| 条件编译 | `libselinux/src/label_file.c` | 多文件加载逻辑 | OHOS_FC_INIT |
| 新增模块 | `libselinux/src/app_allow_config.c/h` | 应用白名单配置 | 应用数据保护 |

---

## OHOS_FC_INIT 宏详细分析

### 概述

`OHOS_FC_INIT` 是 OpenHarmony 引入的条件编译宏，用于启用**多文件 file_contexts 支持**。

### 原始问题

上游 SELinux 的 `selabel_handle` 结构体只支持**单个** file_contexts 配置文件：

```c
// 原始代码（无 OHOS_FC_INIT）
struct selabel_handle {
    // ...
    char *spec_file;    // 单个配置文件路径
    // ...
};
```

这在复杂系统中存在局限：
- 无法模块化策略配置
- 所有规则必须放在单一文件
- 难以实现分层策略管理

### OH 解决方案

通过 `OHOS_FC_INIT` 宏，修改为支持**文件数组**：

```c
// OH 代码（启用 OHOS_FC_INIT）
struct selabel_handle {
    // ...
    char **spec_file;       // 配置文件路径数组
    size_t spec_file_nums;  // 文件数量
    // ...
};
```

### 修改内容分析

#### 1. label_internal.h (第 111-116 行)

```c
/*
 * The main spec file used. Note for file contexts the local and/or
 * homedirs could also have been used to resolve a context.
 */
#ifdef OHOS_FC_INIT
    char **spec_file;
    size_t spec_file_nums;
#else
    char *spec_file;
#endif
```

**修改目的**：根据宏定义选择数据结构
- 启用 `OHOS_FC_INIT`：使用数组支持多文件
- 未启用：保持原始单文件兼容

#### 2. label.c (第 146-156 行)

```c
static int selabel_fini(const struct selabel_handle *rec,
                        struct selabel_lookup_rec *lr,
                        bool translating)
{
#ifdef OHOS_FC_INIT
    char *path = NULL;
    if (rec->spec_file != NULL) {
        path = rec->spec_file[0];    // 取第一个文件路径用于日志
    }
    if (compat_validate(rec, lr, path, lr->lineno))
        return -1;
#else
    if (compat_validate(rec, lr, rec->spec_file, lr->lineno))
        return -1;
#endif
    // ...
}
```

**修改目的**：适配多文件场景下的路径获取

#### 3. label.c (第 207-218 行)

```c
#ifdef OHOS_FC_INIT
static void free_spec_files(struct selabel_handle *rec)
{
    if (rec->spec_file != NULL) {
        for (int path_index = 0; path_index < rec->spec_file_nums; path_index++) {
            if (rec->spec_file[path_index] != NULL) {
                free(rec->spec_file[path_index]);
            }
        }
        free(rec->spec_file);
    }
}
#endif
```

**修改目的**：提供多文件内存释放函数

#### 4. label.c (第 388-391 行)

```c
void selabel_close(struct selabel_handle *rec)
{
    // ...
#ifdef OHOS_FC_INIT
    free_spec_files(rec);
#else
    free(rec->spec_file);
#endif
    free(rec);
}
```

**修改目的**：关闭 handle 时正确释放多文件内存

#### 5. label_file.c (第 795-860 行)

```c
#ifdef OHOS_FC_INIT
static int init(struct selabel_handle *rec, const struct selinux_opt *opts, unsigned n)
{
    struct saved_data *data = (struct saved_data *)rec->data;
    const char *prefix = NULL;
    int status = -1;
    size_t path_nums = 0;
    size_t opt_nums = n;

    // 1. 统计 SELABEL_OPT_PATH 选项数量
    while (n--) {
        switch (opts[n].type) {
            case SELABEL_OPT_PATH:
                path_nums++;    // 计算需要加载的文件数
                break;
            default:
                break;
        }
    }

    if (path_nums == 0) {
        selinux_log(SELINUX_ERROR, "No specific file_contexts provided\n");
        goto finish;
    }

    // 2. 分配数组内存
    rec->spec_file = (char **)calloc(path_nums, sizeof(char *));
    if (rec->spec_file == NULL) {
        goto finish;
    }
    rec->spec_file_nums = path_nums;

    // 3. 复制所有文件路径
    size_t i = 0;
    n = opt_nums;
    while (n--) {
        if (opts[n].type == SELABEL_OPT_PATH) {
            rec->spec_file[i] = strdup(opts[n].value);
            if (rec->spec_file[i] == NULL) {
                goto finish;
            }
            i++;
        }
    }

    // 4. 依次处理每个文件
    for (int path_index = 0; path_index < rec->spec_file_nums; path_index++) {
        status = process_file(rec->spec_file[path_index], NULL, rec, prefix, rec->digest);
        if (status) {
            goto finish;
        }

        if (rec->validating) {
            status = nodups_specs(data, rec->spec_file[path_index]);
            if (status) {
                goto finish;
            }
        }
    }

    digest_gen_hash(rec->digest);
    status = sort_specs(data);

finish:
    if (status)
        closef(rec);
    return status;
}
#else
// 原始单文件 init 函数...
#endif
```

**修改目的**：
1. 统计需要加载的文件数量
2. 为文件路径数组分配内存
3. 复制所有配置文件路径
4. 遍历处理每个配置文件
5. 统一排序和验证

### OH 价值

1. **模块化策略管理**
   - 系统策略和应用策略分离
   - 不同厂商可添加独立配置文件
   - 避免单个文件过大难以维护

2. **动态配置能力**
   - 运行时决定加载哪些配置文件
   - 支持条件加载（如不同设备类型）

3. **向后兼容**
   - 未定义 `OHOS_FC_INIT` 时完全保持上游行为
   - 可在其他平台复用

### 回归风险

**风险等级：中**

| 风险项 | 说明 | 缓解措施 |
|-------|------|---------|
| API 变更 | `selabel_handle` 结构体布局变化 | 仅在 OH 编译，不影响上游 |
| 内存管理 | 单文件 vs 多文件释放逻辑不同 | 代码已通过条件编译隔离 |
| 升级冲突 | 新版本可能有结构体改动 | 升级时需检查 label_file.c 的 init 函数 |

**升级建议**：
1. 升级上游版本时，保留 `#ifdef OHOS_FC_INIT` 条件编译块
2. 检查新增的结构体成员是否需要加入多文件支持
3. 验证 `init()` 函数逻辑是否有重大变更

---

## app_allow_config 模块分析

### 概述

`app_allow_config` 是 OpenHarmony 新增的模块，用于实现**应用白名单配置**。

### 原始问题

SELinux 的 `restorecon` 操作会递归遍历目录并恢复默认标签。这在某些场景下会导致问题：
- 应用数据目录标签被系统策略覆盖
- 应用沙箱内的文件权限被重置
- 需要一种机制跳过特定路径

### OH 解决方案

新增白名单机制，允许配置不需要 restorecon 的路径。

### 代码实现

#### 1. app_allow_config.h

```c
#ifndef APP_ALLOW_CONFIG_H_
#define APP_ALLOW_CONFIG_H_

#include <stdbool.h>

static bool insert_line_to_app_allow_config(const char *line);
void load_app_allow_config();
bool is_in_app_allow_config(const char *pathname);

#endif
```

#### 2. app_allow_config.c

**配置路径**：
```c
#define SYSTEM_APP_ALLOW_CONFIG_PATH "/system/etc/selinux/app_allow_cfg"
```

**数据结构**：
```c
static char **g_app_allow_config = NULL;    // 白名单路径数组
static size_t g_line_count = 0;              // 路径数量
```

**关键函数**：

```c
// 加载配置文件
void load_app_allow_config()
{
    FILE *file = fopen(SYSTEM_APP_ALLOW_CONFIG_PATH, "r");
    if (file == NULL) {
        selinux_log(SELINUX_ERROR, "Failed to open file, %s\n", 
                     SYSTEM_APP_ALLOW_CONFIG_PATH);
        return;
    }

    char *line = NULL;
    size_t len = 0;
    while (getline(&line, &len, file) != -1) {
        len = trim_newline(line);
        // 去除末尾的 '/'
        if (len > 0 && line[len -1] == '/') {
            line[len - 1] = '\0';
        }

        if (!insert_line_to_app_allow_config(line)) {
            selinux_log(SELINUX_ERROR, "Failed to insert line: %s\n", line);
            continue;
        }
        g_line_count++;
    }

    free(line);
    fclose(file);
}

// 检查路径是否在白名单中
bool is_in_app_allow_config(const char *pathname)
{
    if (pathname == NULL || g_app_allow_config == NULL) {
        return false;
    }

    for (size_t i = 0; i < g_line_count; i++) {
        const char *allow_path = g_app_allow_config[i];
        if (allow_path == NULL) {
            continue;
        }
        // 前缀匹配：pathname 以 allow_path 开头
        if (strncmp(pathname, allow_path, strlen(allow_path)) == 0) {
            return true;
        }
    }
    return false;
}
```

### 使用位置

在 `selinux_restorecon.c` 第 776 行使用：

```c
if ((!is_in_app_allow_config(pathname))) {
    // 执行 restorecon
} else {
    // 跳过白名单路径
}
```

### OH 价值

1. **应用数据保护**
   - 应用沙箱目录不被系统策略覆盖
   - 应用自定义标签得以保留

2. **灵活配置**
   - 配置文件可独立更新
   - 无需修改代码即可调整白名单

3. **性能优化**
   - 减少不必要的标签检查
   - 提升 restorecon 执行效率

### 回归风险

**风险等级：低**

| 风险项 | 说明 | 缓解措施 |
|-------|------|---------|
| 新增文件 | 升级可能误删 | 确保文件在 BUILD.gn 的 sources 列表中 |
| 集成点 | restorecon 的调用点 | 升级时需检查 selinux_restorecon.c 的变化 |
| 配置文件 | 格式兼容性 | 保持简单的每行一个路径格式 |

**升级建议**：
1. 新增文件无需修改，直接保留
2. 检查 selinux_restorecon.c 的变更，确保集成点正确
3. 配置文件格式保持简单，避免复杂解析

---

## Patch 升级建议汇总

### 可直接推向上游的修改

无。当前所有修改都是 OH 特有需求。

### OH 特有需保留的修改

| 修改 | 保留原因 | 升级注意事项 |
|-----|---------|------------|
| OHOS_FC_INIT | OH 特有需求 | 检查结构体变更 |
| app_allow_config | OH 特有功能 | 检查集成点变更 |

### 升级检查清单

- [ ] 确认 label_internal.h 结构体定义
- [ ] 确认 label.c 内存释放逻辑
- [ ] 确认 label_file.c init() 函数逻辑
- [ ] 确认 app_allow_config 集成点
- [ ] 确认 BUILD.gn sources 列表
- [ ] 运行 selinux 测试套件


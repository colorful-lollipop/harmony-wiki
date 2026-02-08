# 02 - Patch 详细分析

本文档详细分析 SANE-backends 在 OpenHarmony 中的所有 Patch，包括修改目的、技术细节和升级建议。

---

## Patch 清单总览

| 序号 | Patch 文件 | 功能分类 | OH 特有 | 修改文件 |
|------|-----------|----------|---------|----------|
| 1 | `add_thread_poll.patch` | 性能优化 | **是** | `backend/dll.c` |
| 2 | `hilog_debug.patch` | 日志适配 | **是** | `include/sane/sanei_debug.h` |
| 3 | `modifying_driver_search_path.patch` | 路径适配 | **是** | `backend/dll.c` |
| 4 | `modify_load_function.patch` | 错误处理 | **是** | `backend/dll.c` |
| 5 | `Rules-quot.patch` | 构建修复 | 否 | `po/Rules-quot` |
| 6 | `ltmain.sh.patch` | 库名处理 | 否 | `ltmain.sh` |
| 7 | `ax_create_stdint_h.19-20.m4.patch` | 许可证更新 | 否 | `m4/ax_create_stdint_h.m4` |
| 8 | `ax_create_stdint_h.20-21.m4.patch` | 构建修复 | 否 | `m4/ax_create_stdint_h.m4` |

---

## OH 特有 Patch 详解

### Patch 1: add_thread_poll.patch

**修改文件**: `backend/dll.c`

**修改目的**: 将串行的扫描设备发现改为并行执行，提升设备发现性能。在拥有多个后端（如 escl, pixma, epsonds 等）的场景下，串行发现会导致显著的延迟。

**关键技术变更**:

1. **新增数据结构**
```c
typedef struct backend_task {
    struct backend *be;
    SANE_Bool local_only;
    const SANE_Device **be_list;
    SANE_Status status;
    int num_devs;
    SANE_Device **devices;
    int device_count;
} backend_task_t;
```

2. **任务处理函数**
```c
static void process_backend_task(void *arg) {
    backend_task_t *task = (backend_task_t *)arg;
    // 初始化后端（如果未初始化）
    if (!be->inited) {
        if (init(be) != SANE_STATUS_GOOD) {
            task->status = SANE_STATUS_INVAL;
            return;
        }
    }
    // 并发执行 sane_get_devices
    status = (*(op_get_devs_t)be->op[OP_GET_DEVS])(&task->be_list, task->local_only);
    // ... 处理设备列表
}
```

3. **并发控制**
```c
static threadpool_handle_t* pool = NULL;
static const int max_thread_number = 20;

// 根据后端数量动态确定线程数
int discoverThreadNum = backend_count < max_thread_number ? backend_count : max_thread_number;
pool = threadpool_create(discoverThreadNum);
```

**OH 需求背景**:
- OH 扫描服务需要在启动时快速发现所有可用扫描仪
- 传统串行发现在多后端场景下耗时过长（可能超过 10 秒）
- 并行发现可将时间缩短至原来的 1/N（N 为后端数量）

**代码变更摘要**:
- 新增约 215 行代码
- 保留原有的串行逻辑（通过 `HAVE_THREAD_POLL` 宏控制）
- 新增线程池初始化和清理逻辑
- 任务结果合并和内存管理

**回归风险评估**:
- **高**: 线程池并发可能引入竞态条件
- **中**: 后端初始化顺序不再确定
- **低**: 内存使用增加（任务结构体分配）

**升级建议**:
- 此 Patch 为 OH 特有，升级上游版本时需重新适配
- 需验证新后端是否正确支持并发初始化
- 建议将此功能推向上游（需要充分的测试和社区审查）

---

### Patch 2: hilog_debug.patch

**修改文件**: `include/sane/sanei_debug.h`

**修改目的**: 将 SANE 原有调试日志接入 OpenHarmony HiLog 日志系统，实现日志统一管理。

**关键技术变更**:

1. **条件编译控制**
```c
#ifdef ENABLE_HILOG
#include "hilog/log.h"
#endif // ENABLE_HILOG
```

2. **宏重定向**
```c
#ifndef ENABLE_HILOG
# define DBG        sanei_debug_ndebug
#else
# define DBG(level, ...)    ((void)HiLogPrint(LOG_APP, LOG_INFO, 0, "sanekit", __VA_ARGS__))
#endif // ENABLE_HILOG
```

**OH 需求背景**:
- OH 要求所有系统组件使用统一的日志系统（HiLog）
- 便于日志收集、过滤和分析
- 支持按域（Domain）和标签（Tag）分类

**代码变更摘要**:
- 新增 8 行代码
- 在 `NDEBUG` 和非 `NDEBUG` 模式下均支持 HiLog
- 使用 `LOG_APP` 域和 `LOG_INFO` 级别
- 标签固定为 `"sanekit"`

**升级注意事项**:
- HiLog API 可能随 OH 版本升级而变化
- 日志格式与原 SANE 略有不同（HiLog 有自己的格式）
- 调试级别信息可能需要额外处理

**升级建议**:
- 此 Patch 为 OH 特有，无法推向上游
- 升级时需检查 HiLog API 兼容性
- 考虑使用更灵活的日志抽象层

---

### Patch 3: modifying_driver_search_path.patch

**修改文件**: `backend/dll.c`

**修改目的**: 适配 OpenHarmony 的文件系统结构和驱动加载方式，支持 OH 沙箱目录和扫描服务架构。

**关键技术变更**:

1. **驱动库搜索函数**
```c
static void find_libname_by_drivername(char* libname, char* dir, char* drivername)
{
    // 支持 libsane-xxx.z.so (压缩格式)
    snprintf(driver_name_with_z_so, sizeof(driver_name_with_z_so), 
             "libsane-%s.z.so", drivername);
    // 支持 libsane-xxx.so (标准格式)
    snprintf(driver_name_with_so, sizeof(driver_name_with_so), 
             "libsane-%s.so", drivername);
    
    // 遍历目录查找匹配的库文件
    DIR *backends_dir = opendir(dir);
    while ((entry = readdir(backends_dir)) != NULL) {
        if (has_suffix(full_path, driver_name_with_z_so) || 
            has_suffix(full_path, driver_name_with_so)) {
            strncpy(libname, full_path, PATH_MAX);
            break;
        }
    }
}
```

2. **配置文件扫描**
```c
static void read_configs(const char *conffile)
{
    const char* dir = "/data/service/el1/public/print_service/sane/config";
    DIR *config_dir = opendir(dir);
    while ((entry = readdir(config_dir)) != NULL) {
        if (has_suffix(entry->d_name, conffile)) {
            read_config(entry->d_name);
        }
    }
}
```

3. **加载路径选择**
```c
#elif defined (HAVE_SCAN_SERVICE)
    find_libname_by_drivername(libname, dir, be->name);
#else
    snprintf(libname, sizeof(libname), "%s/" PREFIX "%s" POSTFIX,
             dir, be->name, V_MAJOR);
#endif
```

**OH 需求背景**:
- OH 使用沙箱目录结构，而非传统 Linux FHS
- 驱动库可能使用压缩格式（*.z.so）以节省空间
- 扫描服务需要动态加载配置

**OH 沙箱路径**:
```
/data/service/el1/public/print_service/sane/
├── backend/     # 后端驱动库 (*.so, *.z.so)
├── config/      # 配置文件 (*.conf)
├── data/        # 数据文件
└── lock/        # 锁文件
```

**代码变更摘要**:
- 新增约 70 行代码
- 新增 `find_libname_by_drivername()` 函数
- 新增 `read_configs()` 函数
- 新增 `has_suffix()` 辅助函数

**安全考虑**:
- 目录遍历需确保路径合法性（防止路径遍历攻击）
- 库文件验证（签名检查由系统负责）

**升级建议**:
- 此 Patch 为 OH 特有，升级时需重新适配
- 路径可能随 OH 版本变化，需要同步更新
- 建议将路径定义为配置项，减少硬编码

---

### Patch 4: modify_load_function.patch

**修改文件**: `backend/dll.c`

**修改目的**: 优化后端加载失败时的错误处理，提供更准确的错误码。

**关键技术变更**:

1. **错误码优化**
```c
// 修改前
return SANE_STATUS_INVAL;

// 修改后  
return SANE_STATUS_UNSUPPORTED;
```

2. **避免重复加载检查**
```c
// 修改前
if (!be->loaded)

// 修改后
if (!be->loaded || be->op[OP_INIT] == op_unsupported)
```

**OH 需求背景**:
- 扫描服务需要根据错误码做不同处理
- `SANE_STATUS_INVAL`（无效参数）不够准确
- `SANE_STATUS_UNSUPPORTED`（不支持）更准确地表达了"后端不存在"的语义

**代码变更摘要**:
- 修改 2 处代码
- 错误码变更提升错误处理精确度
- 避免对已标记为不支持的后端重复尝试加载

**升级建议**:
- 此改进具有通用性，可考虑推向上游
- 升级时需验证错误码变化是否影响现有逻辑

---

## 上游移植 Patch 详解

### Patch 5: Rules-quot.patch

**修改文件**: `po/Rules-quot`

**修改目的**: 修复 .po 文件生成时的词包装不一致问题。

**技术变更**:
- 添加 `$(MSGCONV_OPTIONS)` 和 `$(MSGFILTER_OPTIONS)` 选项支持
- 允许自定义宽度参数

**来源**: 上游社区修复（Olaf Meeuwissen）

**OH 相关性**: 低（仅影响国际化构建）

---

### Patch 6: ltmain.sh.patch

**修改文件**: `ltmain.sh`

**修改目的**: 统一所有后端库的 soname 为 `libsane`，而非 `libsane-backendname`。

**技术变更**:
```bash
# 将所有 libsane-xxx 重命名为 libsane
soname=`echo $soname | sed -e "s/libsane-[A-Za-z_0-9]*/libsane/g"`
```

**背景**: SANE 后端设计中，内部库名统一为 `libsane`，便于动态链接。

**OH 相关性**: 中（影响构建，但 OH 使用 GN 构建，此 Patch 可能不需要）

---

### Patch 7 & 8: ax_create_stdint_h 系列 Patch

**修改文件**: `m4/ax_create_stdint_h.m4`

**修改目的**:
1. **Patch 7**: 将许可证从 GPL 改为 all-permissive
2. **Patch 8**: 修复 autoconf 宏兼容性（`AC_TRY_COMPILE` → `AC_COMPILE_IFELSE`）

**OH 相关性**: 低（OH 使用 GN 构建，不使用 autoconf）

---

## Patch 应用顺序

```
install.py 执行顺序：

1. modifying_driver_search_path.patch
   └─> 修改驱动搜索路径和配置加载

2. add_thread_poll.patch
   └─> 添加线程池支持（依赖路径修改）

3. hilog_debug.patch
   └─> 添加 HiLog 支持（依赖线程池中的 DBG 调用）

4. modify_load_function.patch
   └─> 优化错误处理（最后应用，避免冲突）
```

---

## Patch 维护建议

### 推向上游评估

| Patch | 可推向上游 | 难度 | 备注 |
|-------|-----------|------|------|
| `add_thread_poll.patch` | 是 | 高 | 需要充分测试和社区讨论 |
| `hilog_debug.patch` | 否 | - | OH 特有 |
| `modifying_driver_search_path.patch` | 部分 | 中 | 抽象为配置选项 |
| `modify_load_function.patch` | 是 | 低 | 简单改进 |

### 升级检查清单

升级上游版本时，按以下顺序检查：

1. [ ] Patch 是否还能干净应用
2. [ ] 新版本中是否有相似功能的官方实现
3. [ ] 修改的文件是否有重大重构
4. [ ] 测试用例是否通过
5. [ ] 性能测试是否有回归

### 版本兼容性矩阵

| 上游版本 | 当前 Patch 适用性 | 备注 |
|----------|------------------|------|
| 1.4.0 | ✓ 完全适用 | 当前目标版本 |
| 1.3.x | △ 部分适用 | 需验证 |
| 1.2.x | ✗ 不适用 | 代码结构差异大 |
| 1.5.0+ | ? 待验证 | 需评估 |

---

## 总结

SANE-backends 的 8 个 Patch 可分为三类：

1. **OH 核心适配** (4 个): 线程池、HiLog、路径适配、错误处理 - **升级需重新适配**
2. **构建修复** (2 个): Autotools 相关 - **OH 使用 GN，可逐步移除**
3. **通用改进** (2 个): 国际化、库名处理 - **可考虑推向上游**

维护工作的重点是保持 4 个 OH 特有 Patch 与上游版本的兼容性，同时关注上游安全更新。

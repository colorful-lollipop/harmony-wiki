# 内部 API 文档

> OpenHarmony musl 内部接口说明

---

## 目的与适用范围

**目的**: 说明 musl 内部模块之间的接口，帮助理解模块依赖和扩展机制。

**适用范围**: musl 开发者、系统移植人员、安全分析人员。

---

## 内部接口概览

musl 内部接口主要分布在：

| 位置 | 内容 | 稳定性 |
|------|------|--------|
| `src/internal/*.h` | 内部头文件 | 不稳定 |
| `src/*/internal.h` | 模块内部头文件 | 不稳定 |
| `ldso/*.h` | 动态链接器头文件 | 较稳定 |
| `arch/*/bits/*.h` | 架构特定定义 | 稳定 |

---

## src/internal/ 核心头文件

### 1. pthread_impl.h

线程实现的核心头文件。

```c
// 关键结构
struct pthread {
    struct pthread *self;
    uintptr_t *dtv;
    struct pthread *prev, *next;
    // ... 更多字段
};

// 关键函数
hidden void __pthread_tsd_run_dtors();
hidden void __do_cleanup_push(struct __ptcb *cb);
hidden void __do_cleanup_pop(struct __ptcb *cb);
```

### 2. stdio_impl.h

标准 I/O 实现头文件。

```c
// FILE 结构定义
struct _IO_FILE {
    unsigned flags;
    unsigned char *rpos, *rend;
    int (*close)(FILE *);
    // ... 更多字段
};

// 关键函数
hidden int __lockfile(FILE *);
hidden void __unlockfile(FILE *);
```

### 3. locale_impl.h

locale 实现头文件。

```c
// locale 结构
struct __locale_map {
    const void *map;
    size_t map_size;
    const char *name;
    struct __locale_map *next;
};

// ICU 集成
#ifdef FEATURE_ICU_LOCALE
// ICU 相关定义
#endif
```

---

## 动态链接器内部接口

### 1. dynlink.h

动态链接器核心头文件。

```c
// DSO (动态共享对象) 结构
struct dso {
    unsigned char *base;
    char *name;
    size_t *dynv;
    struct dso *next, *prev;
    // ... 更多字段
};

// 关键函数
hidden void __dls3(void);
hidden struct dso *load_library(const char *name, struct dso *needed_by);
hidden void *dlopen_preload(const char *file, int mode);
```

### 2. namespace.h

Namespace 机制头文件。

```c
// Namespace 结构 (ldso/linux/namespace.h:37-52)
typedef struct _namespace_t_ {
    char *ns_name;            // namespace 名称
    char *env_paths;          // LD_LIBRARY_PATH
    char *lib_paths;          // 库搜索路径
    strlist *permitted_paths; // 允许的路径
    dsolist *ns_dsos;         // DSO 列表
    struct _ns_inherit_list_ *ns_inherits; // 继承列表
    // ... 更多字段
} ns_t;

// 关键函数
hidden ns_t *get_default_ns();
hidden bool is_accessible(ns_t *ns, const char *lib_pathname, bool is_asan, bool check_inherited);
hidden ns_t *find_ns_by_name(const char *ns_name);
```

### 3. dynlink_rand.h

地址随机化头文件。

```c
// 加载任务结构 (ldso/linux/dynlink_rand.h:36-82)
struct loadtask {
    const char *name;
    const char *fullname;
    struct dso *needed_by;
    ns_t *namespace;
    // ... 更多字段
};

// 关键函数
hidden void shuffle_loadtasks(struct loadtasks *tasks);
hidden struct loadtask *create_loadtask(const char *name, struct dso *needed_by, ns_t *ns, bool check_inherited);
```

---

## 内存分配器内部接口

### 1. mallocng/meta.h

mallocng 元数据头文件。

```c
// 关键结构 (src/malloc/mallocng/meta.h:17-58)
struct group {
    struct meta *meta;
    unsigned char active_idx:5;
    char pad[UNIT - sizeof(struct meta *) - 1];
    unsigned char storage[];
};

struct meta {
    struct meta *prev, *next;
    struct group *mem;
    volatile int avail_mask, freed_mask;
    uintptr_t last_idx:5;
    uintptr_t freeable:1;
    uintptr_t sizeclass:6;
    uintptr_t maplen:8*sizeof(uintptr_t)-12;
};

struct malloc_context {
    uint64_t secret;
    int init_done;
    struct meta *active[48];
    // ... 更多字段
};

// 关键函数
hidden struct meta *alloc_meta(void);
static inline struct meta *get_meta(const unsigned char *p);
static inline void *encode_ptr(void *ptr, uint64_t key);
```

### 2. mallocng/glue.h

mallocng 胶水层头文件。

```c
// 命名空间宏
#define size_classes __malloc_size_classes
#define ctx __malloc_context
#define malloc __libc_malloc_impl
#define free __libc_free

// 锁操作
static inline void rdlock();
static inline void wrlock();
static inline void unlock();
```

---

## Hook 机制内部接口

### 1. musl_preinit_common.h

Hook 框架核心头文件。

```c
// Hook 函数枚举 (src/hook/linux/musl_preinit_common.h:40-50)
enum EnumFunc {
    INITIALIZE_FUNCTION,
    FINALIZE_FUNCTION,
    GET_HOOK_FLAG_FUNCTION,
    SET_HOOK_FLAG_FUNCTION,
    ON_START_FUNCTION,
    ON_END_FUNCTION,
    SEND_HOOK_MISC_DATA,
    GET_HOOK_CONFIG,
    LAST_FUNCTION,
};

// Hook 模式
enum EnumHookMode {
    STARTUP_HOOK_MODE,
    DIRECT_HOOK_MODE,
    STEP_HOOK_MODE,
};

// 关键函数
inline bool __get_global_hook_flag();
inline bool __get_hook_flag();
inline volatile const struct MallocDispatchType* get_current_dispatch_table();
```

### 2. musl_malloc_dispatch.h

内存分配调度头文件。

```c
// 分配函数指针类型
typedef void* (*MallocType)(size_t);
typedef void (*FreeType)(void*);
typedef void* (*ReallocType)(void*, size_t);
typedef void* (*CallocType)(size_t, size_t);

// 调度表结构
struct MallocDispatchType {
    MallocType malloc;
    FreeType free;
    ReallocType realloc;
    CallocType calloc;
    // ... 更多字段
};
```

---

## 系统调用 Hook 接口

### syscall_hooks.h

系统调用 Hook 头文件。

```c
// Hook 表和入口 (src/internal/linux/syscall_hooks.h:22-23)
extern hidden volatile const char *g_syscall_hooks_table;
extern hidden volatile void *g_syscall_hooks_entry;

// Hook 检查
static inline int is_syscall_hooked(long n) {
    return g_syscall_hooks_table != NULL && g_syscall_hooks_table[n] != 0;
}

// Hook 入口函数
static inline long __syscall_hooks_entry0(long n);
static inline long __syscall_hooks_entry1(long n, long a);
// ... 最多6个参数
```

---

## 模块依赖关系

### 依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层                                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                   src/internal                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │pthread  │  │stdio    │  │malloc   │  │locale   │        │
│  │_impl.h  │  │_impl.h  │  │_config.h│  │_impl.h  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └─────────┘        │
└───────┼────────────┼────────────┼──────────────────────────┘
        │            │            │
        └────────────┼────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    arch/                                     │
│              架构特定实现                                     │
│         系统调用封装 (syscall.h)                              │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Linux Kernel                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 接口稳定性说明

| 接口类别 | 稳定性 | 说明 |
|----------|--------|------|
| 标准 C/POSIX | 稳定 | 遵循标准规范 |
| 架构特定接口 | 稳定 | 架构抽象层 |
| 动态链接器接口 | 较稳定 | 可能随功能扩展变化 |
| 内部头文件 | 不稳定 | 可能随时变更 |
| Hook 接口 | 较稳定 | 向后兼容 |

---

## 扩展点

### 1. 添加新的 Hook 类型

```c
// 1. 在 EnumFunc 中添加新函数类型
enum EnumFunc {
    // ... 现有函数
    NEW_HOOK_FUNCTION,  // 新增
    LAST_FUNCTION,
};

// 2. 实现 Hook 函数
void* new_hook_function(args) {
    // 实现
}
```

### 2. 添加新的 Namespace 配置

```c
// 在配置文件中添加
[new_namespace]
    namespace.new.lib.paths = /path/to/libs
    namespace.new.inherits = default
```

---

## 相关跳转

- [对外 API](03_Public_API.md) - 对外接口说明
- [架构设计](02_Architecture.md) - 组件架构
- [安全分析](07_Security_Analysis.md) - 接口安全风险


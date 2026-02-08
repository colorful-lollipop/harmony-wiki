# 对外 API 文档

> OpenHarmony musl 对外 API 说明

---

## 目的与适用范围

**目的**: 说明 musl 提供的对外 API 接口，包括标准 C/POSIX 接口和 OpenHarmony 扩展接口。

**适用范围**: 应用开发者、系统服务开发者。

---

## API 概览

musl 提供以下类别的 API：

| 类别 | 标准 | 头文件 | 说明 |
|------|------|--------|------|
| 标准 C | ISO C99 | `stdio.h`, `stdlib.h`, `string.h` | 标准 C 库函数 |
| POSIX | POSIX.1-2008 | `pthread.h`, `unistd.h`, `fcntl.h` | POSIX 接口 |
| 网络 | BSD/Linux | `socket.h`, `netdb.h` | 网络编程接口 |
| 动态链接 | GNU/Linux | `dlfcn.h` | 动态库操作 |
| OHOS 扩展 | OpenHarmony | `info/*.h`, `trace/*.h` | 系统特定扩展 |

---

## 标准 C API

### 1. 标准 I/O (stdio.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `fopen()` | 打开文件 | `src/stdio/fopen.c` |
| `fread()` | 读取数据 | `src/stdio/fread.c` |
| `fwrite()` | 写入数据 | `src/stdio/fwrite.c` |
| `fclose()` | 关闭文件 | `src/stdio/fclose.c` |
| `printf()` | 格式化输出 | `src/stdio/printf.c` |
| `scanf()` | 格式化输入 | `src/stdio/scanf.c` |

### 2. 标准库 (stdlib.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `malloc()` | 分配内存 | `src/malloc/mallocng/malloc.c` |
| `free()` | 释放内存 | `src/malloc/mallocng/free.c` |
| `realloc()` | 重新分配 | `src/malloc/mallocng/realloc.c` |
| `atoi()` | 字符串转整数 | `src/stdlib/atoi.c` |
| `exit()` | 退出程序 | `src/exit/exit.c` |
| `qsort()` | 快速排序 | `src/stdlib/qsort.c` |

### 3. 字符串操作 (string.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `memcpy()` | 内存拷贝 | `src/string/memcpy.c` |
| `memset()` | 内存设置 | `src/string/memset.c` |
| `strlen()` | 字符串长度 | `src/string/strlen.c` |
| `strcpy()` | 字符串拷贝 | `src/string/strcpy.c` |
| `strcmp()` | 字符串比较 | `src/string/strcmp.c` |

---

## POSIX API

### 1. 线程 (pthread.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `pthread_create()` | 创建线程 | `src/thread/pthread_create.c` |
| `pthread_join()` | 等待线程 | `src/thread/pthread_join.c` |
| `pthread_mutex_lock()` | 加锁 | `src/thread/pthread_mutex_lock.c` |
| `pthread_cond_wait()` | 条件等待 | `src/thread/pthread_cond_wait.c` |

### 2. 文件操作 (unistd.h, fcntl.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `open()` | 打开文件 | `src/fcntl/open.c` |
| `read()` | 读取 | `src/unistd/read.c` |
| `write()` | 写入 | `src/unistd/write.c` |
| `close()` | 关闭 | `src/unistd/close.c` |
| `lseek()` | 定位 | `src/unistd/lseek.c` |

### 3. 进程控制 (unistd.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `fork()` | 创建进程 | `src/process/fork.c` |
| `execve()` | 执行程序 | `src/process/execve.c` |
| `waitpid()` | 等待子进程 | `src/process/waitpid.c` |

---

## 动态链接 API (dlfcn.h)

### 标准接口

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `dlopen()` | 加载动态库 | `src/ldso/dlopen.c` |
| `dlsym()` | 查找符号 | `src/ldso/dlsym.c` |
| `dlclose()` | 关闭动态库 | `src/ldso/dlclose.c` |
| `dlerror()` | 获取错误信息 | `src/ldso/dlerror.c` |

### OpenHarmony 扩展

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `dlns_create()` | 创建 namespace | `ldso/linux/namespace.c` |
| `dlns_set_search_paths()` | 设置搜索路径 | `ldso/linux/namespace.c` |
| `dlns_get_error()` | 获取 namespace 错误 | `ldso/linux/namespace.c` |

---

## OpenHarmony 扩展 API

### 1. 信息接口 (include/info/*.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `get_application_target_sdk_version()` | 获取应用目标 SDK 版本 | `src/info/application_target_sdk_version.c` |
| `get_device_api_version()` | 获取设备 API 版本 | `src/info/device_api_version.c` |

### 2. 追踪接口 (include/trace/*.h)

| 函数 | 说明 | 代码位置 |
|------|------|----------|
| `trace_marker_begin()` | 开始追踪标记 | `src/trace/trace_marker.c` |
| `trace_marker_end()` | 结束追踪标记 | `src/trace/trace_marker.c` |

---

## 内存分配 API 详解

### mallocng 安全分配器

OpenHarmony musl 使用 mallocng 作为默认内存分配器，提供以下安全特性：

```c
// 标准接口
void *malloc(size_t size);
void free(void *ptr);
void *realloc(void *ptr, size_t size);
void *calloc(size_t nmemb, size_t size);
int posix_memalign(void **memptr, size_t alignment, size_t size);

// musl 扩展
size_t malloc_usable_size(void *ptr);
```

### 安全级别控制

| 宏 | 说明 | 代码位置 |
|----|------|----------|
| `MALLOC_FREELIST_HARDENED` | 空闲列表加固 | `src/malloc/mallocng/meta.h:77-82` |
| `MALLOC_FREELIST_QUARANTINE` | 隔离区机制 | 延迟释放 |
| `MALLOC_RED_ZONE` | 红区保护 | 检测溢出 |
| `MALLOC_SECURE_ALL` | 全部安全特性 | 最高安全级别 |

---

## 错误处理

### errno 定义

```c
// 标准错误码
#define EPERM        1  // 操作不允许
#define ENOENT       2  // 没有该文件或目录
#define ESRCH        3  // 没有该进程
#define EINTR        4  // 系统调用被中断
#define EIO          5  // I/O 错误
#define ENXIO        6  // 没有该设备或地址
#define E2BIG        7  // 参数列表太长
#define ENOEXEC      8  // 执行格式错误
#define EBADF        9  // 错误的文件描述符
#define ECHILD      10  // 没有子进程
// ... 更多错误码
```

### 错误处理模式

```c
// 典型错误处理模式
FILE *fp = fopen("file.txt", "r");
if (fp == NULL) {
    // 错误处理
    perror("fopen failed");  // 输出错误信息
    // 或
    fprintf(stderr, "Error: %s\n", strerror(errno));
    return -1;
}
```

---

## API 调用链示例

### malloc 调用链

```
malloc() [src/malloc/mallocng/malloc.c]
    └──► __libc_malloc_impl()
        └──► alloc_slot()
            └──► get_meta() [src/malloc/mallocng/meta.h:144]
                └──► encode_ptr() [安全校验]
```

### dlopen 调用链

```
dlopen() [src/ldso/dlopen.c]
    └──► dlopen_preload()
        └──► load_library() [ldso/linux/dynlink.c]
            └──► load_library_ns() [namespace 检查]
                └──► map_library()
                    └──► reloc_library()
```

---

## 相关跳转

- [内部 API](04_Internal_API.md) - 内部接口说明
- [架构设计](02_Architecture.md) - 组件架构
- [安全分析](07_Security_Analysis.md) - API 安全风险


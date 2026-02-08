# 关键调用链

> OpenHarmony musl 关键调用链附录

---

## 1. 程序启动调用链

```
_start (arch/{arch}/crt_arch.h)
    └──► __libc_start_main (src/env/__libc_start_main.c)
        ├──► __init_tls
        ├──► __libc_init
        ├──► 执行构造函数
        └──► main()
```

## 2. 动态链接调用链

```
dlopen()
    └──► dlopen_preload() (src/ldso/dlopen.c)
        └──► load_library() (ldso/linux/dynlink.c)
            ├──► load_library_ns() (namespace 检查)
            ├──► map_library()
            │   ├──► mmap 加载 ELF
            │   └──► 解析 ELF 头
            └──► reloc_library()
                ├──► 符号解析
                └──► 重定位处理
```

## 3. 内存分配调用链

```
malloc() (src/malloc/mallocng/malloc.c)
    └──► __libc_malloc_impl()
        ├──► size_overflows() (大小检查)
        ├──► alloc_slot()
        │   ├──► get_meta() (获取元数据)
        │   ├──► enqueue/dequeue (管理空闲块)
        │   └──► enframe() (分配帧)
        └──► 返回用户指针

free() (src/malloc/mallocng/free.c)
    └──► get_meta() (验证并获取元数据)
        ├──► 安全检查
        ├──► 释放 slot
        └──► 更新空闲列表
```

## 4. 线程创建调用链

```
pthread_create() (src/thread/pthread_create.c)
    └──► __clone() (系统调用)
        └──► 新线程执行 start_thread()
            ├──► 初始化 TLS
            ├──► 设置线程属性
            └──► 调用用户函数
```

## 5. 文件打开调用链

```
fopen() (src/stdio/fopen.c)
    └──► __fopen_rb_ca()
        ├──► open() (src/fcntl/open.c)
        │   └──► __syscall(SYS_open)
        └──► 初始化 FILE 结构
```

## 6. Namespace 加载调用链

```
ldso 启动
    └──► nslist_init() (ldso/linux/namespace.c)
        ├──► 解析配置文件
        ├──► ns_alloc() 创建默认 namespace
        └──► 加载程序依赖
            └──► is_accessible() 路径检查
```

## 7. Hook 机制调用链

```
malloc() [Hook 版本]
    └──► get_current_dispatch_table()
        ├──► 检查 Hook 标志
        ├──► 如果启用: 调用 Hook 实现
        └──► 否则: 调用原始实现
```


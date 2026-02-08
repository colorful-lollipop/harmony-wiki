# 编译产物

> .so/.abc 产物、安装路径与运行时加载关系

## 1. 产物概览

### 1.1 产物类型

| 类型 | 扩展名 | 说明 |
|------|--------|------|
| **动态库** | `.so` | Native 共享库 |
| **字节码** | `.abc` | ArkTS 字节码 |
| **目标文件** | `.o` / `.c` | 编译中间产物 |

### 1.2 产物清单

#### js_api_module 产物

| 产物 | 路径 | 大小估算 |
|------|------|----------|
| liburl.so | `module/` | ~50KB |
| liburi.so | `module/` | ~30KB |
| libxml.so | `module/` | ~80KB |
| libbuffer.so | `module/` | ~60KB |
| libconvertxml.so | `module/` | ~40KB |
| libfastbuffer.so | `module/` | ~50KB |

#### js_util_module 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libarraylist.so | `module/util/` | 动态数组 |
| libdeque.so | `module/util/` | 双端队列 |
| libqueue.so | `module/util/` | 队列 |
| libvector.so | `module/util/` | 向量 |
| liblinkedlist.so | `module/util/` | 双向链表 |
| liblist.so | `module/util/` | 单向链表 |
| libstack.so | `module/util/` | 栈 |
| libtreemap.so | `module/util/` | 有序 Map |
| libtreeset.so | `module/util/` | 有序 Set |
| libhashmap.so | `module/util/` | 哈希 Map |
| libhashset.so | `module/util/` | 哈希 Set |
| liblightweightmap.so | `module/util/` | 轻量级 Map |
| liblightweightset.so | `module/util/` | 轻量级 Set |
| libplainarray.so | `module/util/` | 数值数组 |
| libstruct.so | `module/util/` | 结构化数据 |
| libutil.so | `module/` | 工具类 (TextEncoder 等) |
| libjson.so | `module/` | JSON 处理 |
| libcollections.so | `module/` | 集合工具 |
| libstream.so | `module/` | 流处理 |

#### js_sys_module 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libprocess.so | `module/` | 进程管理 |
| libtimer.so | `module/` | 定时器 |
| libconsole.so | `module/` | 控制台 |
| libdfx.so | `module/` | 调试工具 |

#### js_concurrent_module 产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libworker.so | `module/` | Worker 线程 |
| libtaskpool.so | `module/` | 任务池 |
| libconcurrentutils.so | `module/` | 并发工具 |

## 2. 运行时加载关系

### 2.1 模块加载层次

```
ArkTS 应用 (.hap)
    │
    ├─ import @ohos.buffer
    │       │
    │       ▼
    │   libbuffer.so (延迟加载)
    │       │
    │       ▼
    │   libnapi.so (N-API 运行时)
    │       │
    │       ▼
    │   libc++.so (C++ 运行时)
    │       │
    │       ▼
    │   libc.so (系统 C 库)
    │
├─ import @ohos.util
│       │
│       ▼
│   libutil.so
│   libcontainer_*.so (按需加载)
│
├─ import @ohos.worker
│       │
│       ▼
│   libworker.so
│       │
│       ▼
│   libffrt.so (FFRT 线程池)
```

### 2.2 依赖图

```
libbuffer.so
├── libnapi.so
├── libhilog.so
└── libc++.so

libworker.so
├── libnapi.so
├── libffrt.so
├── libhilog.so
└── libc++.so

libcontainer_hashmap.so
├── libnapi.so
├── libhilog.so
└── libc++.so
```

## 3. 字节码文件

### 3.1 .abc 字节码

ArkTS 编译生成的字节码文件。

| 字节码 | 对应模块 |
|--------|----------|
| `url.abc` | URL |
| `buffer.abc` | Buffer |
| `arraylist.abc` | ArrayList |
| `hashmap.abc` | HashMap |
| ... | ... |

### 3.2 字节码加载

```
ArkTS 运行时
    │
    ▼
加载 .abc 字节码文件
    │
    ▼
解释执行 / JIT 编译
    │
    ▼
调用 Native 方法 (通过 N-API)
```

## 4. 安装路径

### 4.1 系统路径

```
/system/lib/module/           # 系统 Native 库
    ├── liburl.so
    ├── liburi.so
    ├── libbuffer.so
    └── ...

/system/lib/module/util/       # 容器库
    ├── libarraylist.so
    ├── libdeque.so
    └── ...

/system/etc/                   # 配置文件
```

### 4.2 应用路径

```
/data/app/<bundle-name>/  # 应用私有目录
    ├── libs/             # Native 库
    ├── module/           # 模块
    └── cache/            # 缓存
```

## 5. 加载时配置

### 5.1 dlopen 加载

Native 模块通过 `dlopen()` 动态加载：

```cpp
// 伪代码示例
void* handle = dlopen("libbuffer.so", RTLD_LAZY);
if (handle) {
    auto init = (napi_init*)dlsym(handle, "napi_module_register");
    if (init) {
        init(&bufferModule);
    }
}
```

### 5.2 符号导出

```
nm libbuffer.so | grep " T "
0000000000001234 T Buffer_init
0000000000001456 T Buffer_alloc
0000000000001678 T Buffer_write
```

## 6. 调试产物

### 6.1 Debug 符号

调试版本包含 `.dbg` 或调试符号：

```
libbuffer.so.dbg          # 调试符号文件
libbuffer.so.unstripped    # 未剥离符号的库
```

### 6.2 Source Map

```
buffer.map                # 源文件映射
buffer.js.map             # JS 源码映射
```

## 相关文档

- [07_Build_Configuration.md](./07_Build_Configuration.md) - 构建配置
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [10_Troubleshooting.md](./10_Troubleshooting.md) - 故障排除

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

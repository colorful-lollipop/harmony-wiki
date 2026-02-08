# NDK 接口文档

> **重要说明**: 本项目**不包含 N-API**（JavaScript/TypeScript 绑定），仅提供 **NDK (Native Development Kit)** 纯 C 接口。
>
> NDK 接口供 Native 应用（C/C++）直接调用，不涉及 JavaScript/TypeScript 层。

## 模块信息

| 属性 | 值 |
|------|-----|
| **库文件** | libpurgeablemem.z.so / purgeable_memory_ndk.z.so |
| **头文件** | purgeable_memory.h |
| **Syscap** | SystemCapability.Kernel.Memory |
| **版本** | 1.0 |
| **起始版本** | 10 |

**证据来源**: `libpurgeablemem/interfaces/kits/c/purgeable_memory.h:16-40`

---

## 包含文件

```c
#include <purgeable_memory.h>
```

---

## 类型定义

### OH_PurgeableMemory

```c
// purgeable_memory.h:58
typedef struct PurgMem OH_PurgeableMemory;
```

**说明**: 可回收内存对象的不透明句柄。开发者无需了解内部结构，通过 API 操作。

### OH_PurgeableMemory_ModifyFunc

```c
// purgeable_memory.h:72
typedef bool (*OH_PurgeableMemory_ModifyFunc)(void *, size_t, void *);
```

**说明**: 重建回调函数指针类型。

| 参数 | 类型 | 说明 |
|------|------|------|
| void * | void* | 数据指针，指向可回收内存对象的起始地址 |
| size_t | size_t | 数据大小 |
| void * | void* | 其他私有参数（创建时传入） |
| 返回值 | bool | 重建结果，true 成功，false 失败 |

---

## API 清单

### OH_PurgeableMemory_Create

```c
OH_PurgeableMemory *OH_PurgeableMemory_Create(
    size_t size,
    OH_PurgeableMemory_ModifyFunc func,
    void *funcPara
);
```

**功能**: 创建一个可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| size | size_t | 可回收内存对象的数据大小 |
| func | OH_PurgeableMemory_ModifyFunc | 函数指针，用于在内存被回收后重建数据 |
| funcPara | void* | 传递给 func 的参数 |
| **返回值** | OH_PurgeableMemory* | 创建成功返回对象指针，失败返回 NULL |

**错误码**: 无（返回 NULL 表示失败）

**C++ 实现位置**: `libpurgeablemem/cpp/src/purgeable_mem.cpp`

---

### OH_PurgeableMemory_Destroy

```c
bool OH_PurgeableMemory_Destroy(OH_PurgeableMemory *purgObj);
```

**功能**: 销毁一个可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 要销毁的可回收内存对象 |
| **返回值** | bool | true 成功或 purgObj 为 NULL；false 失败 |

**错误处理**:
- 如果 `purgObj` 为 NULL，返回 true
- 调用 munmap 释放映射的内存
- 销毁对象后，将指针置为 NULL 以避免 Use-After-Free

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:100`

---

### OH_PurgeableMemory_BeginRead

```c
bool OH_PurgeableMemory_BeginRead(OH_PurgeableMemory *purgObj);
```

**功能**: 开始读取可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |
| **返回值** | bool | true - 内容存在或重建成功；false - 内容被回收且重建失败 |

**语义**:
1. 增加页表引用计数（Pin 内存）
2. 如果内存已被回收，触发重建
3. 返回 true 表示可以安全访问数据

**使用要求**:
```c
// 必须检查返回值并正确处理
if (OH_PurgeableMemory_BeginRead(purgObj)) {
    void *data = OH_PurgeableMemory_GetContent(purgObj);
    // 访问数据...
    OH_PurgeableMemory_EndRead(purgObj);
} else {
    // 处理重建失败情况
}
```

**线程安全**: 线程安全，可在多线程中调用

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:126`

---

### OH_PurgeableMemory_EndRead

```c
void OH_PurgeableMemory_EndRead(OH_PurgeableMemory *purgObj);
```

**功能**: 结束读取可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |

**语义**:
1. 减少页表引用计数（Unpin 内存）
2. 函数返回后，系统可能在任意时间回收内存

**注意**: 调用此函数后，不应再访问数据指针

**线程安全**: 线程安全，可在多线程中调用

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:139`

---

### OH_PurgeableMemory_BeginWrite

```c
bool OH_PurgeableMemory_BeginWrite(OH_PurgeableMemory *purgObj);
```

**功能**: 开始写入可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |
| **返回值** | bool | true - 内容存在或重建成功；false - 内容被回收且重建失败 |

**语义**:
1. 增加页表引用计数（Pin 内存）
2. 如果内存已被回收，触发重建
3. 返回 true 表示可以安全修改数据

**使用要求**:
```c
if (OH_PurgeableMemory_BeginWrite(purgObj)) {
    void *data = OH_PurgeableMemory_GetContent(purgObj);
    // 修改数据...
    OH_PurgeableMemory_EndWrite(purgObj);
} else {
    // 处理重建失败情况
}
```

**线程安全**: 线程安全，可在多线程中调用

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:165`

---

### OH_PurgeableMemory_EndWrite

```c
void OH_PurgeableMemory_EndWrite(OH_PurgeableMemory *purgObj);
```

**功能**: 结束写入可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |

**语义**:
1. 减少页表引用计数（Unpin 内存）
2. 函数返回后，系统可能在任意时间回收内存

**注意**: 调用此函数后，不应再访问数据指针

**线程安全**: 线程安全，可在多线程中调用

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:178`

---

### OH_PurgeableMemory_GetContent

```c
void *OH_PurgeableMemory_GetContent(OH_PurgeableMemory *purgObj);
```

**功能**: 获取可回收内存对象的数据指针。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |
| **返回值** | void* | 数据起始地址；purgObj 为 NULL 时返回 NULL |

**使用要求**:
- 必须在 `BeginRead/EndRead` 或 `BeginWrite/EndWrite` 之间调用
- 返回的指针在 EndRead/EndWrite 调用后可能失效

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:193`

---

### OH_PurgeableMemory_ContentSize

```c
size_t OH_PurgeableMemory_ContentSize(OH_PurgeableMemory *purgObj);
```

**功能**: 获取可回收内存对象的数据大小。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |
| **返回值** | size_t | 数据大小；purgObj 为 NULL 时返回 0 |

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:206`

---

### OH_PurgeableMemory_AppendModify

```c
bool OH_PurgeableMemory_AppendModify(
    OH_PurgeableMemory *purgObj,
    OH_PurgeableMemory_ModifyFunc func,
    void *funcPara
);
```

**功能**: 追加一个修改操作到可回收内存对象。

| 参数 | 类型 | 说明 |
|------|------|------|
| purgObj | OH_PurgeableMemory* | 可回收内存对象 |
| func | OH_PurgeableMemory_ModifyFunc | 修改函数指针 |
| funcPara | void* | 传递给 func 的参数 |
| **返回值** | bool | true 成功；false 失败 |

**语义**: 将修改操作追加到构建器链中，在数据重建时按顺序执行。

**C 实现位置**: `libpurgeablemem/c/src/purgeable_memory.c:220`

---

## API 总结表

| API | 同步/异步 | 参数 | 返回值 | 线程安全 |
|-----|-----------|------|--------|----------|
| Create | 同步 | size_t, ModifyFunc, void* | OH_PurgeableMemory* | ✅ |
| Destroy | 同步 | OH_PurgeableMemory* | bool | ✅ |
| BeginRead | 同步 | OH_PurgeableMemory* | bool | ✅ |
| EndRead | 同步 | OH_PurgeableMemory* | void | ✅ |
| BeginWrite | 同步 | OH_PurgeableMemory* | bool | ✅ |
| EndWrite | 同步 | OH_PurgeableMemory* | void | ✅ |
| GetContent | 同步 | OH_PurgeableMemory* | void* | ✅ |
| ContentSize | 同步 | OH_PurgeableMemory* | size_t | ✅ |
| AppendModify | 同步 | OH_PurgeableMemory*, ModifyFunc, void* | bool | ✅ |

---

## 使用示例

### 基本使用流程

```c
#include <purgeable_memory.h>
#include <stdio.h>
#include <string.h>

// 重建回调函数
bool RebuildCallback(void *data, size_t size, void *para) {
    printf("Rebuilding data...\n");
    // 重建数据，例如从文件读取
    memset(data, 0, size);
    return true;
}

int main() {
    size_t dataSize = 4096;  // 4KB
    
    // 1. 创建可回收内存对象
    OH_PurgeableMemory *purgObj = OH_PurgeableMemory_Create(
        dataSize,
        RebuildCallback,
        NULL
    );
    
    if (purgObj == NULL) {
        printf("Failed to create purgeable memory\n");
        return -1;
    }
    
    // 2. 写入数据
    if (OH_PurgeableMemory_BeginWrite(purgObj)) {
        void *data = OH_PurgeableMemory_GetContent(purgObj);
        if (data != NULL) {
            memcpy(data, "Hello", 5);
            printf("Data written: %s\n", (char *)data);
        }
        OH_PurgeableMemory_EndWrite(purgObj);
    }
    
    // 3. 读取数据
    if (OH_PurgeableMemory_BeginRead(purgObj)) {
        void *data = OH_PurgeableMemory_GetContent(purgObj);
        if (data != NULL) {
            printf("Data read: %s\n", (char *)data);
        }
        OH_PurgeableMemory_EndRead(purgObj);
    }
    
    // 4. 销毁对象
    OH_PurgeableMemory_Destroy(purgObj);
    
    return 0;
}
```

### 追加修改操作

```c
// 修改操作函数
bool ModifyFunc1(void *data, size_t size, void *para) {
    const char *msg = "Modified1 ";
    memcpy(data, msg, strlen(msg));
    return true;
}

bool ModifyFunc2(void *data, size_t size, void *para) {
    const char *msg = "Modified2 ";
    memcpy(data + 10, msg, strlen(msg));
    return true;
}

// 在创建后追加修改操作
OH_PurgeableMemory_AppendModify(purgObj, ModifyFunc1, NULL);
OH_PurgeableMemory_AppendModify(purgObj, ModifyFunc2, NULL);
```

---

## 错误处理指南

### 错误码

NDK API 不使用错误码，直接返回错误状态：

| API | 错误情况 | 处理方式 |
|-----|----------|----------|
| Create | 返回 NULL | 检查返回值，使用默认内存分配回退 |
| BeginRead | 返回 false | 重建失败，应用应优雅处理 |
| BeginWrite | 返回 false | 重建失败，应用应优雅处理 |
| Destroy | 返回 false | 销毁失败，记录日志 |
| AppendModify | 返回 false | 追加失败，记录日志 |

### 重建失败处理

```c
if (OH_PurgeableMemory_BeginRead(purgObj)) {
    // 安全访问
    void *data = OH_PurgeableMemory_GetContent(purgObj);
    // ...
    OH_PurgeableMemory_EndRead(purgObj);
} else {
    // 重建失败的处理
    printf("Failed to recover purgeable memory\n");
    // 可能需要：
    // 1. 重新创建对象
    // 2. 使用备用内存
    // 3. 向用户报告错误
}
```

---

## 线程安全说明

所有 NDK API 都是**线程安全**的：

- 使用 `pthread_rwlock_t` 读写锁保护内部状态
- 多个线程可以并发调用 BeginRead/EndRead
- 写入操作需要获取写锁，串行执行

**注意**: 同一个对象在同一时刻只能有一个写入者。

---

## 编译与链接

### CMake

```cmake
target_link_libraries(your_target PRIVATE
    libpurgeable_memory_ndk.z.so
)
```

### BUILD.gn

```gn
external_deps = [ "purgeablemem:purgeable_memory_ndk" ]
```

---

## 版本历史

| NDK API 版本 | 变更说明 |
|--------------|----------|
| 1.0 (API 10) | 初始版本 |

---

## 相关文档

- [内部 API (C++)](03_Inner_API.md)
- [架构说明](01_Architecture.md)
- [GN 构建配置](04_GN_Build.md)

---

**最后更新**: 2026-02-06

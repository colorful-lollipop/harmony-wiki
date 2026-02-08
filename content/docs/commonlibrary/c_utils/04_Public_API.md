# 对外 API

## 目的

本文档描述 c_utils 对外暴露的 C++ API 接口，供其他模块集成使用。

## 适用范围

- 需要集成 c_utils 的开发者
- 接口使用者查阅参考

## 重要说明

**c_utils 是纯 C++ 基础库，不提供 N-API 接口。** 所有接口均为 C++ 头文件形式，供其他 native 模块通过 `#include` 使用。

---

## API 清单

### 1. 内存管理

#### RefBase - 引用计数基类

**头文件**: `base/include/refbase.h`

| 类/模板 | 功能 | 关键方法 |
|---------|------|----------|
| `RefBase` | 引用计数基类 | `IncStrongRef()`, `DecStrongRef()`, `OnLastStrongRef()` |
| `sptr<T>` | 强引用智能指针 | `operator->()`, `operator*()`, `get()` |
| `wptr<T>` | 弱引用智能指针 | `AttemptIncStrong()`, `promote()` |

**使用示例**:
```cpp
#include "refbase.h"

class MyClass : public OHOS::RefBase {
public:
    void DoSomething() {}
};

// 强引用
OHOS::sptr<MyClass> obj = new MyClass();
obj->DoSomething();

// 弱引用
OHOS::wptr<MyClass> weak = obj;
OHOS::sptr<MyClass> strong = weak.promote();
if (strong != nullptr) {
    strong->DoSomething();
}
```

#### unique_fd - RAII 资源管理

**头文件**: `base/include/unique_fd.h`

| 类 | 功能 | 关键方法 |
|----|------|----------|
| `unique_fd` | 文件描述符自动管理 | `release()`, `reset()`, `get()` |
| `unique_file` | FILE* 自动管理 | 同 unique_fd |
| `unique_dir` | DIR* 自动管理 | 同 unique_fd |
| `unique_map` | mmap 自动管理 | 同 unique_fd |

**使用示例**:
```cpp
#include "unique_fd.h"
#include <fcntl.h>

{
    OHOS::unique_fd fd(open("/path/to/file", O_RDONLY));
    if (fd >= 0) {
        // 使用 fd...
    }
    // 自动关闭 fd
}
```

---

### 2. 序列化

#### Parcel - 数据序列化容器

**头文件**: `base/include/parcel.h`

| 类 | 功能 | 关键方法 |
|----|------|----------|
| `Parcelable` | 可序列化接口 | `Marshalling()`, `Unmarshalling()` (静态) |
| `Parcel` | 数据容器 | 读写各类数据类型 |

**Parcel 数据写入方法**:

| 方法 | 参数 | 说明 |
|------|------|------|
| `WriteInt32(int32_t)` | int32_t | 写入32位整数 |
| `WriteInt64(int64_t)` | int64_t | 写入64位整数 |
| `WriteUint32(uint32_t)` | uint32_t | 写入无符号32位整数 |
| `WriteUint64(uint64_t)` | uint64_t | 写入无符号64位整数 |
| `WriteFloat(float)` | float | 写入浮点数 |
| `WriteDouble(double)` | double | 写入双精度浮点数 |
| `WriteBool(bool)` | bool | 写入布尔值 |
| `WriteString(const std::string&)` | string | 写入字符串 |
| `WriteBuffer(const uint8_t*, size_t)` | buffer, size | 写入二进制数据 |
| `WriteParcelable(const Parcelable*)` | parcelable | 写入可序列化对象 |

**使用示例**:
```cpp
#include "parcel.h"

class MyData : public OHOS::Parcelable {
public:
    int value;
    std::string name;
    
    bool Marshalling(OHOS::Parcel& parcel) const override {
        return parcel.WriteInt32(value) && parcel.WriteString(name);
    }
    
    static MyData* Unmarshalling(OHOS::Parcel& parcel) {
        MyData* data = new MyData();
        if (!parcel.ReadInt32(data->value)) return nullptr;
        if (!parcel.ReadString(data->name)) return nullptr;
        return data;
    }
};
```

---

### 3. 并发工具

#### ThreadPool - 线程池

**头文件**: `base/include/thread_pool.h`

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Start(int numThreads)` | 线程数 | uint32_t | 启动线程池 |
| `Stop()` | - | void | 停止线程池 |
| `AddTask(Task)` | 任务函数 | void | 添加任务 |
| `SetMaxTaskNum(size_t)` | 最大任务数 | void | 设置任务队列上限 |
| `GetCurTaskNum()` | - | size_t | 获取当前任务数 |

**使用示例**:
```cpp
#include "thread_pool.h"

OHOS::ThreadPool pool("MyPool");
pool.Start(4);  // 4个工作线程

pool.AddTask([]() {
    // 执行任务
});

pool.Stop();
```

#### RWLock - 读写锁

**头文件**: `base/include/rwlock.h`

| 类 | 功能 |
|----|------|
| `RWLock` | 读写锁原语 |
| `ReadLockGuard` | 读锁 RAII 包装 |
| `WriteLockGuard` | 写锁 RAII 包装 |

**使用示例**:
```cpp
#include "rwlock.h"

OHOS::RWLock rwlock;

// 读操作
{
    OHOS::ReadLockGuard guard(rwlock);
    // 读取数据
}

// 写操作
{
    OHOS::WriteLockGuard guard(rwlock);
    // 修改数据
}
```

#### Semaphore - 信号量

**头文件**: `base/include/semaphore_ex.h`

| 方法 | 说明 |
|------|------|
| `Semaphore(int value)` | 构造函数，指定初始值 |
| `Wait()` | P 操作（等待）|
| `Post()` | V 操作（释放）|
| `TryWait()` | 非阻塞等待 |

---

### 4. 线程安全容器

#### SafeMap - 线程安全 Map

**头文件**: `base/include/safe_map.h`

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Insert(K, V)` | key, value | bool | 插入键值对 |
| `Find(K, V&)` | key, value | bool | 查找键值对 |
| `Erase(K)` | key | void | 删除键值对 |
| `ReadVal(K)` | key | V | 读取值（不存在会默认构造）|
| `Size()` | - | int | 获取大小 |
| `IsEmpty()` | - | bool | 判断是否为空 |
| `Clear()` | - | void | 清空 |
| `Iterate(callback)` | callback | void | 遍历 |

**使用示例**:
```cpp
#include "safe_map.h"

OHOS::SafeMap<int, std::string> map;
map.Insert(1, "value1");

std::string value;
if (map.Find(1, value)) {
    // value == "value1"
}
```

#### SafeQueue - 线程安全队列

**头文件**: `base/include/safe_queue.h`

| 方法 | 说明 |
|------|------|
| `Push(T)` | 入队 |
| `Pop(T&)` | 出队 |
| `Front(T&)` | 获取队首 |
| `IsEmpty()` | 判断是否为空 |
| `Size()` | 获取大小 |
| `Clear()` | 清空 |

#### SafeBlockQueue - 线程安全阻塞队列

**头文件**: `base/include/safe_block_queue.h`

在 SafeQueue 基础上增加阻塞等待功能：

| 方法 | 说明 |
|------|------|
| `Push(T)` | 入队（非阻塞）|
| `Pop(T&)` | 出队（阻塞等待）|
| `Pop(T&, timeout)` | 出队（带超时）|
| `TryPop(T&)` | 尝试出队（非阻塞）|

---

### 5. 文件系统

#### 文件操作

**头文件**: `base/include/file_ex.h`

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `LoadStringFromFile(path, content)` | 路径, 输出字符串 | bool | 从文件读取字符串（最大32MB）|
| `SaveStringToFile(path, content, truncated)` | 路径, 内容, 是否截断 | bool | 保存字符串到文件 |
| `LoadStringFromFd(fd, content)` | fd, 输出字符串 | bool | 从fd读取 |
| `SaveStringToFd(fd, content)` | fd, 内容 | bool | 写入fd |
| `LoadBufferFromFile(path, buffer)` | 路径, vector | bool | 读取二进制 |
| `SaveBufferToFile(path, buffer, truncated)` | 路径, vector, 截断 | bool | 保存二进制 |
| `FileExists(path)` | 路径 | bool | 检查文件存在 |
| `StringExistsInFile(path, str, caseSensitive)` | 路径, 子串, 大小写敏感 | bool | 检查文件包含字符串 |
| `CountStrInFile(path, str, caseSensitive)` | 路径, 子串, 大小写敏感 | int | 统计字符串出现次数 |

**使用示例**:
```cpp
#include "file_ex.h"

std::string content;
if (OHOS::LoadStringFromFile("/data/config.txt", content)) {
    // 处理内容
}

OHOS::SaveStringToFile("/data/output.txt", "Hello World");
```

#### 目录操作

**头文件**: `base/include/directory_ex.h`

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `GetCurrentProcFullFileName()` | - | string | 获取当前程序完整路径 |
| `GetCurrentProcPath()` | - | string | 获取当前程序目录 |
| `ExtractFilePath(fullPath)` | 完整路径 | string | 提取目录路径 |
| `ExtractFileName(fullPath)` | 完整路径 | string | 提取文件名 |
| `ExtractFileExt(fileName)` | 文件名 | string | 提取扩展名 |
| `GetDirFiles(path, files)` | 路径, vector | void | 递归获取所有文件 |
| `IsEmptyFolder(path)` | 路径 | bool | 检查空目录 |
| `ForceCreateDirectory(path)` | 路径 | bool | 递归创建目录 |
| `ForceRemoveDirectory(path)` | 路径 | bool | 递归删除目录 |
| `RemoveFile(fileName)` | 文件名 | bool | 删除文件 |
| `GetFolderSize(path)` | 路径 | uint64_t | 获取目录大小 |
| `ChangeModeFile(file, mode)` | 文件, 权限 | bool | 修改文件权限 |
| `ChangeModeDirectory(path, mode)` | 路径, 权限 | bool | 修改目录权限 |
| `PathToRealPath(path, realPath)` | 路径, 输出 | bool | 转换相对路径为绝对路径 |

---

### 6. 内存映射

#### MappedFile - 文件内存映射

**头文件**: `base/include/mapped_file.h`

| 方法 | 说明 |
|------|------|
| `Open(const std::string& fileName, int flags)` | 打开文件 |
| `Close()` | 关闭映射 |
| `GetData()` | 获取映射内存指针 |
| `GetSize()` | 获取映射大小 |
| `IsOpened()` | 检查是否已打开 |

**使用示例**:
```cpp
#include "mapped_file.h"

OHOS::MappedFile mappedFile;
if (mappedFile.Open("/data/large_file.bin", O_RDONLY)) {
    void* data = mappedFile.GetData();
    size_t size = mappedFile.GetSize();
    // 访问内存...
    mappedFile.Close();
}
```

#### Ashmem - 匿名共享内存

**头文件**: `base/include/ashmem.h`

| 方法 | 说明 |
|------|------|
| `Create(const std::string& name, size_t size)` | 创建共享内存 |
| `MapReadWrite()` | 映射为读写 |
| `MapReadOnly()` | 映射为只读 |
| `Unmap()` | 解除映射 |
| `GetName()` | 获取名称 |
| `GetSize()` | 获取大小 |
| `GetFd()` | 获取文件描述符 |

---

### 7. 字符串处理

#### 字符串增强

**头文件**: `base/include/string_ex.h`

| 函数 | 说明 |
|------|------|
| `StrToInt(str, value)` | 字符串转整数 |
| `StrToUint(str, value)` | 字符串转无符号整数 |
| `StrToLong(str, value)` | 字符串转长整数 |
| `StrToDouble(str, value)` | 字符串转双精度浮点 |
| `Trim(str)` | 去除两端空白 |
| `ToUpper(str)` | 转大写 |
| `ToLower(str)` | 转小写 |
| `ReplaceStr(str, from, to)` | 替换子串 |
| `SplitStr(str, delim, results)` | 分割字符串 |
| `IsNumeric(str)` | 检查是否为数字 |
| `IsAlpha(str)` | 检查是否为字母 |
| `IsUpper(str)` | 检查是否为大写 |
| `IsLower(str)` | 检查是否为小写 |

---

### 8. 定时器

#### Timer - 定时器

**头文件**: `base/include/timer.h`

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Setup()` | - | uint32_t | 启动定时器线程 |
| `Shutdown(useJoin)` | 是否join | void | 停止定时器 |
| `Register(callback, interval, once)` | 回调, 间隔(ms), 是否单次 | uint32_t | 注册定时器 |
| `Unregister(timerId)` | 定时器ID | void | 注销定时器 |

**使用示例**:
```cpp
#include "timer.h"

OHOS::Utils::Timer timer("MyTimer");
timer.Setup();

uint32_t timerId = timer.Register([]() {
    // 定时执行
}, 1000, false);  // 每1000ms执行一次

// ...

timer.Unregister(timerId);
timer.Shutdown();
```

---

### 9. 设计模式

#### Singleton - 单例模式

**头文件**: `base/include/singleton.h`

```cpp
#include "singleton.h"

class MySingleton : public OHOS::Singleton<MySingleton> {
    DECLARE_SINGLETON(MySingleton);
public:
    void DoSomething() {}
};

// 使用
MySingleton::GetInstance().DoSomething();
```

#### Observer - 观察者模式

**头文件**: `base/include/observer.h`

| 类 | 说明 |
|----|------|
| `Observer` | 观察者接口 |
| `Observable` | 被观察者基类 |
| `ObservableHandler` | 观察者管理器 |

---

### 10. 错误处理

#### 错误码

**头文件**: `base/include/errors.h`

```cpp
enum class ErrCode : int32_t {
    ERR_OK = 0,
    ERR_NO_MEMORY = -1,
    ERR_INVALID_DATA = -2,
    ERR_INVALID_PARAM = -3,
    ERR_NAME_NOT_FOUND = -4,
    ERR_PERMISSION_DENIED = -5,
    ERR_NOT_SUPPORTED = -6,
    ERR_UNKNOWN_OBJECT = -7,
    // ...
};
```

---

## 接口稳定性

| 接口类别 | 稳定性 | 说明 |
|----------|--------|------|
| RefBase, sptr, wptr | **稳定** | 广泛使用，不会轻易变更 |
| Parcel, Parcelable | **稳定** | IPC 基础，保持兼容 |
| file_ex, directory_ex | **稳定** | 基础文件操作 |
| safe_map, safe_queue | **稳定** | 容器接口 |
| thread_pool | **稳定** | 线程池接口 |
| timer | **稳定** | 定时器接口 |
| string_ex | **稳定** | 字符串处理 |
| unique_fd | **稳定** | RAII 资源管理 |

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 头文件位置
- [内部 API](05_Inner_API.md) - 内部模块接口
- [架构说明](03_Architecture.md) - 架构设计

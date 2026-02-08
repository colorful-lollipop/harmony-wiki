# 安全风险评审

## 目的

本文档基于代码证据对 c_utils 进行安全风险评审，识别攻击面和潜在可被利用点。

## 适用范围

- 安全审计
- 风险评估
- 安全加固参考

---

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           c_utils 信任边界                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   外部不可信输入                    信任边界              内部处理           │
│   ┌──────────────┐                ┌─────────┐          ┌──────────────┐    │
│   │              │                │         │          │              │    │
│   │ 文件路径      │───────────────►│  校验   │─────────►│ 文件操作      │    │
│   │ 用户输入      │                │  过滤   │          │ 内存分配      │    │
│   │ 网络数据      │                │         │          │ 序列化        │    │
│   │ IPC数据       │                │         │          │ 线程同步      │    │
│   │              │                │         │          │              │    │
│   └──────────────┘                └─────────┘          └──────────────┘    │
│                                                                             │
│   风险等级: 高                      风险等级: 中         风险等级: 低        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 入口点 | 风险等级 | 说明 |
|--------|--------|----------|------|
| 文件系统 | `file_ex.h`, `directory_ex.h` | **高** | 路径遍历、符号链接攻击 |
| 内存映射 | `mapped_file.h` | **中** | 资源耗尽、越界访问 |
| 共享内存 | `ashmem.h` | **中** | 权限设置、信息泄露 |
| 序列化 | `parcel.h` | **中** | 数据伪造、对象注入 |
| 线程池 | `thread_pool.h` | **低** | 资源耗尽、任务注入 |
| 定时器 | `timer.h` | **低** | 资源耗尽 |
| 字符串处理 | `string_ex.h` | **低** | 缓冲区溢出（已防护）|

---

## 可被利用点分析

### 1. 路径遍历风险

**证据**: `base/src/directory_ex.cpp:90-100`

```cpp
string GetCurrentProcFullFileName() {
    char procFile[PATH_MAX + 1] = {0};
    int ret = readlink("/proc/self/exe", procFile, PATH_MAX);
    if (ret < 0 || ret > PATH_MAX) {
        return string();
    }
    procFile[ret] = '\0';
    return string(procFile);
}
```

**风险**: 
- 使用 `readlink` 读取符号链接，可能被劫持
- 路径长度检查存在边界问题（`ret > PATH_MAX` 应为 `ret >= PATH_MAX`）

**触发路径**:
```
GetCurrentProcFullFileName()
  -> readlink("/proc/self/exe", ...)
  -> 如果 /proc/self/exe 被篡改，可能返回恶意路径
```

**影响**: 信息泄露、路径欺骗

**修复建议**:
```cpp
// 修复边界检查
if (ret < 0 || ret >= PATH_MAX) {  // 使用 >= 而非 >
    return string();
}
```

---

### 2. 文件操作路径遍历

**证据**: `base/src/file_ex.cpp:100-150`

```cpp
bool LoadStringFromFile(const std::string& filePath, std::string& content) {
    unique_fd fd(open(filePath.c_str(), O_RDONLY | O_CLOEXEC));
    // ...
}
```

**风险**:
- 未对 `filePath` 进行路径规范化检查
- 可能存在 `../` 路径遍历
- 符号链接攻击

**触发路径**:
```
LoadStringFromFile("../../../etc/passwd", content)
  -> open("../../../etc/passwd", ...)
  -> 读取敏感文件
```

**影响**: 任意文件读取

**修复建议**:
```cpp
bool LoadStringFromFile(const std::string& filePath, std::string& content) {
    // 1. 路径规范化
    std::string realPath;
    if (!PathToRealPath(filePath, realPath)) {
        return false;
    }
    
    // 2. 检查路径是否在允许范围内
    if (!IsPathAllowed(realPath)) {
        return false;
    }
    
    unique_fd fd(open(realPath.c_str(), O_RDONLY | O_CLOEXEC | O_NOFOLLOW));
    // ...
}
```

---

### 3. 递归目录遍历风险

**证据**: `base/src/directory_ex.cpp:200-250`

```cpp
void GetDirFiles(const std::string& path, std::vector<std::string>& files) {
    DIR* dir = opendir(path.c_str());
    // ...
    while ((entry = readdir(dir)) != nullptr) {
        std::string fullPath = path + "/" + entry->d_name;
        if (entry->d_type == DT_DIR) {
            GetDirFiles(fullPath, files);  // 递归
        }
    }
}
```

**风险**:
- 符号链接循环导致无限递归
- 目录嵌套过深导致栈溢出
- 无递归深度限制

**触发路径**:
```
GetDirFiles("/path/with/symlink/loop", files)
  -> 跟随符号链接
  -> 形成循环
  -> 无限递归 / 栈溢出
```

**影响**: 拒绝服务（DoS）

**修复建议**:
```cpp
void GetDirFiles(const std::string& path, std::vector<std::string>& files, 
                 int depth = 0) {
    const int MAX_DEPTH = 100;
    if (depth > MAX_DEPTH) {
        return;  // 限制递归深度
    }
    
    // 使用 O_NOFOLLOW 避免跟随符号链接
    int fd = open(path.c_str(), O_RDONLY | O_DIRECTORY | O_NOFOLLOW);
    // ...
}
```

---

### 4. 内存映射文件风险

**证据**: `base/src/mapped_file.cpp`

```cpp
bool MappedFile::Open(const std::string& fileName, int flags) {
    fd_ = open(fileName.c_str(), flags);
    // ...
    data_ = mmap(nullptr, size_, prot, MAP_SHARED, fd_, 0);
    // ...
}
```

**风险**:
- 映射超大文件导致内存耗尽
- 无文件大小限制检查
- MAP_SHARED 修改会影响其他进程

**触发路径**:
```
MappedFile::Open("/path/to/huge/file", O_RDWR)
  -> 文件大小 10GB
  -> mmap(0, 10GB, ...)
  -> 内存耗尽
```

**影响**: 资源耗尽（DoS）

**修复建议**:
```cpp
const size_t MAX_MMAP_SIZE = 512 * 1024 * 1024;  // 512MB限制

bool MappedFile::Open(const std::string& fileName, int flags) {
    // ...
    if (size_ > MAX_MMAP_SIZE) {
        return false;  // 拒绝超大文件
    }
    // ...
}
```

---

### 5. Parcel 容量限制绕过

**证据**: `base/src/parcel.cpp:108-142`

```cpp
size_t CalcNewCapacity(size_t minNewCapacity) {
    // ...
    if ((maxDataCapacity_ > 0) && (newCapacity > maxDataCapacity_)) {
        newCapacity = maxDataCapacity_;
    }
    return newCapacity;
}
```

**风险**:
- `maxDataCapacity_` 默认为 200KB，但可被修改
- 大量小对象写入可能导致对象偏移表溢出

**触发路径**:
```
parcel.SetMaxCapacity(VERY_LARGE_SIZE)
  -> 写入大量数据
  -> 内存分配失败或性能下降
```

**影响**: 资源耗尽

**修复建议**:
- 限制 `SetMaxCapacity` 的最大值
- 对对象数量也进行限制

---

### 6. 线程池任务队列溢出

**证据**: `base/src/thread_pool.cpp:71-84`

```cpp
void ThreadPool::AddTask(const Task& f) {
    if (threads_.empty()) {
        f();  // 直接执行
    } else {
        std::unique_lock<std::mutex> lock(mutex_);
        while (Overloaded()) {
            acceptNewTask_.wait(lock);  // 阻塞等待
        }
        tasks_.push_back(f);
    }
}
```

**风险**:
- 默认无任务数量限制（`maxTaskNum_ = 0`）
- 快速添加任务可能导致内存无限增长

**触发路径**:
```
for (i = 0; i < 1000000; i++) {
    threadPool.AddTask(largeTask);  // 快速添加大量任务
}
  -> 任务队列无限增长
  -> 内存耗尽
```

**影响**: 资源耗尽（DoS）

**修复建议**:
```cpp
ThreadPool::ThreadPool(const std::string& name)
    : myName_(name), maxTaskNum_(DEFAULT_MAX_TASKS),  // 设置默认值
      running_(false) {}
```

---

### 7. 定时器FD泄漏

**证据**: `base/src/timer.cpp:80-111`

```cpp
uint32_t Timer::Register(const TimerCallback& callback, uint32_t interval, bool once) {
    int timerFd = once ? INVALID_TIMER_FD : GetTimerFd(interval);
    if (timerFd == INVALID_TIMER_FD) {
        uint32_t ret = DoRegister(..., timerFd);
        // ...
    }
    // ...
}
```

**风险**:
- 异常情况下 timerfd 可能未关闭
- 高频注册/注销可能导致 fd 耗尽

**触发路径**:
```
while (true) {
    id = timer.Register(callback, 1000, false);
    timer.Unregister(id);  // 异常情况下fd未关闭
}
  -> fd 泄漏
  -> EMFILE (Too many open files)
```

**影响**: 资源耗尽

---

### 8. 字符串转换溢出

**证据**: `base/src/string_ex.cpp`

```cpp
bool StrToInt(const std::string& str, int& value) {
    // 使用 stoi 或 sscanf
}
```

**风险**:
- 超长数字字符串可能导致未定义行为
- 边界值处理不当

**影响**: 数值溢出、未定义行为

**现状**: 使用 bounds_checking_function 进行防护，风险较低

---

## 安全检查总结

### 已实施的安全措施

| 措施 | 位置 | 说明 |
|------|------|------|
| 整数溢出检测 | BUILD.gn | `c_utils_feature_intsan` 默认开启 |
| 安全C库 | 全模块 | 使用 bounds_checking_function |
| ARM32对齐检查 | parcel.cpp | 防止未对齐访问 |
| O_CLOEXEC | file_ex.cpp | 防止fd泄漏到子进程 |
| 文件大小限制 | file_ex.cpp | 32MB读取限制 |

### 建议添加的安全措施

| 优先级 | 措施 | 目标模块 |
|--------|------|----------|
| 高 | 路径规范化检查 | directory_ex, file_ex |
| 高 | 符号链接防护 | directory_ex |
| 中 | 递归深度限制 | directory_ex |
| 中 | 内存映射大小限制 | mapped_file |
| 中 | 任务队列默认上限 | thread_pool |
| 低 | Parcel对象数量限制 | parcel |

---

## 局限性说明

本次安全评审的局限性：

1. **代码覆盖范围**: 仅分析了主要模块，未覆盖所有边界情况
2. **运行时分析**: 缺乏动态分析，无法发现运行时特定的问题
3. **依赖分析**: 未深入分析外部依赖（如 bounds_checking_function）的实现
4. **模糊测试**: 未进行模糊测试，可能存在未发现的问题

---

## 相关跳转

- [对外 API](04_Public_API.md) - 接口安全使用指南
- [内部 API](05_Inner_API.md) - 实现细节
- [问题排查](09_Troubleshooting.md) - 安全相关故障排查

# 攻击面分析

> utils_lite 的完整攻击面识别与安全边界分析。
>
> **适用对象**: 安全研究员、渗透测试人员
> **前置知识**: OpenHarmony 基础架构、C/C++、JavaScript

---

## 目的与范围

### 目的

本文档旨在：
1. 识别 utils_lite 的所有外部输入入口
2. 定位敏感操作和特权接口
3. 分析信任边界和隔离机制
4. 为安全审计提供完整的攻击面清单

### 范围

覆盖模块：
- 文件操作 API (`js/builtin/filekit/`, `file/`)
- KV 存储 API (`js/builtin/kvstorekit/`, `include/kv_store.h`)
- 设备信息 API (`js/builtin/deviceinfokit/`)
- 定时器 API (`timer_task/`, `kal/timer/`)
- 内存管理 API (`memory/`)

**证据来源**: `wiki/_work/NOTES.md`

---

## 外部输入清单

### 1. JavaScript 层输入

#### 1.1 FileKit 输入

| 输入类型 | JS API | 参数 | 证据位置 |
|---------|---------|------|----------|
| 文件路径 | `move()` | src, dest | `nativeapi_fs.cpp:96-97` |
| 文件路径 | `copy()` | src, dest | `nativeapi_fs.cpp:96-97` |
| 文件路径 | `delete()` | path | `nativeapi_fs.cpp:145` |
| 文件路径 | `getFileInfo()` | path | `nativeapi_fs.cpp` |
| 文件路径 | `readTextFile()` | path | `nativeapi_fs.cpp` |
| 文件路径 + 内容 | `writeTextFile()` | path, text | `nativeapi_fs.cpp` |
| 文件路径 | `access()` | path | `nativeapi_fs.cpp` |
| 目录路径 | `createDir()` | dir | `nativeapi_fs.cpp` |
| 目录路径 | `removeDir()` | dir | `nativeapi_fs.cpp` |

**输入验证位置**: `nativeapi_fs.cpp:30-50` - `IsValidPath()` 函数

**验证机制**:
```cpp
// 证据：nativeapi_fs.cpp:30-50
bool IsValidPath(const char* path)
{
    if (path == nullptr) {
        return false;
    }

    size_t pathLen = strnlen(path, URI_NAME_MAX_LEN + 1);
    if (pathLen > URI_NAME_MAX_LEN) {
        return false;
    }
    if ((pathLen < PREFIX_LEN) || (strncmp(path, FILE_PREFIX, PREFIX_LEN) != 0)) {
        return false;
    }
    if ((strstr(path, "/./") != nullptr) || (strstr(path, "/../") != nullptr)) {
        return false;
    }
    if (strpbrk(path + PREFIX_LEN, "\"*+,:;<=>\?[]|\x7F")) {
        return false;
    }
    return true;
}
```

#### 1.2 KVStoreKit 输入

| 输入类型 | JS API | 参数 | 证据位置 |
|---------|---------|------|----------|
| Key | `get()` | key | `nativeapi_kv.cpp` |
| Key + Value | `set()` | key, value | `nativeapi_kv.cpp` |
| Key | `delete()` | key | `nativeapi_kv.cpp` |

**输入验证位置**: `nativeapi_kv.cpp:30-43` - `IsValidKey()` 函数

**验证机制**:
```cpp
// 证据：nativeapi_kv.cpp:30-43
bool IsValidKey(const char* key)
{
    if (key == nullptr) {
        return false;
    }
    size_t keyLen = strnlen(key, KEY_MAX_LEN + 1);
    if ((keyLen == 0) || (keyLen > KEY_MAX_LEN)) {
        return false;
    }
    if (strpbrk(key, "/\\\"*+,:;<=>\?[]|\x7F")) {
        return false;
    }
    return true;
}
```

#### 1.3 DeviceInfoKit 输入

| 输入类型 | JS API | 参数 | 证据位置 |
|---------|---------|------|----------|
| 无参数 | `getInfo()` | 无 | `nativeapi_deviceinfo.h:28` |

**特点**: DeviceInfoKit 不接受外部输入，仅提供只读的设备信息

---

### 2. C 层输入

#### 2.1 文件操作 C API 输入

| 函数 | 参数 | 证据位置 |
|------|------|----------|
| `UtilsFileOpen` | path, oflag, mode | `utils_file.h:162` |
| `UtilsFileRead` | fd, buf, len | `utils_file.h:185` |
| `UtilsFileWrite` | fd, buf, len | `utils_file.h:197` |
| `UtilsFileDelete` | path | `utils_file.h:209` |
| `UtilsFileStat` | path, fileSize | `utils_file.h:220` |
| `UtilsFileSeek` | fd, offset, whence | `utils_file.h:241` |
| `UtilsFileCopy` | src, dest | `utils_file.h:254` |
| `UtilsFileMove` | src, dest | `utils_file.h:267` |

**实现位置**: `file/src/file_impl_hal/file.c:26-100`

**输入验证**: 无直接验证，依赖上层（JSI 层）验证

#### 2.2 定时器任务 C API 输入

| 函数 | 参数 | 证据位置 |
|------|------|----------|
| `StartTimerTask` | isPeriodic, delay, userCallback, userContext, timerHandle | `nativeapi_timer_task.h:30` |
| `StopTimerTask` | timerHandle | `nativeapi_timer_task.h:32` |

**实现位置**: `timer_task/src/nativeapi_timer_task.c:26-46`

**输入验证**:
```cpp
// 证据：nativeapi_timer_task.c:29-31
if (userCallback == NULL || timerHandle == NULL) {
    return EC_FAILURE;
}
```

#### 2.3 定时器 KAL API 输入

| 函数 | 参数 | 证据位置 |
|------|------|----------|
| `KalTimerCreate` | func, type, arg, millisec | `kal.h:39` |
| `KalTimerStart` | timerId | `kal.h:40` |
| `KalTimerChange` | timerId, millisec | `kal.h:41` |
| `KalTimerStop` | timerId | `kal.h:42` |
| `KalTimerDelete` | timerId | `kal.h:43` |

**实现位置**: `kal/timer/src/kal.c`

**输入验证**: 无直接验证，依赖上层验证

---

### 3. HAL/KAL 层输入

#### 3.1 文件 HAL 输入

| 函数 | 参数 | 证据位置 |
|------|------|----------|
| `HalFileOpen` | path, oflag, mode | `hal_file.h` |
| `HalFileRead` | fd, buf, len | `hal_file.h` |
| `HalFileWrite` | fd, buf, len | `hal_file.h` |
| `HalFileDelete` | path | `hal_file.h` |
| `HalFileStat` | path, fileSize | `hal_file.h` |
| `HalFileSeek` | fd, offset, whence | `hal_file.h` |

**实现位置**: `hals/file/hal_file.c`

**输入验证**: 无直接验证，依赖上层验证

---

## 敏感操作清单

### 1. 文件系统操作

| 操作 | API | 风险等级 | 证据位置 |
|------|------|----------|----------|
| 文件读取 | `UtilsFileRead()` | 中 | `utils_file.h:185` |
| 文件写入 | `UtilsFileWrite()` | 高 | `utils_file.h:197` |
| 文件删除 | `UtilsFileDelete()` | 高 | `utils_file.h:209` |
| 文件复制 | `UtilsFileCopy()` | 中 | `utils_file.h:254` |
| 文件移动 | `UtilsFileMove()` | 高 | `utils_file.h:267` |
| 目录创建 | `createDir()` | 中 | `nativeapi_fs.h:37` |
| 目录删除 | `removeDir()` | 高 | `nativeapi_fs.h:38` |

**风险点**:
- 路径遍历攻击
- 符号链接攻击
- 资源耗尽（文件描述符限制）
- TOCTOU（Time-of-check to time-of-use）

### 2. KV 存储操作

| 操作 | API | 风险等级 | 证据位置 |
|------|------|----------|----------|
| 读取键值 | `Get()` | 低 | `nativeapi_kv.h:28` |
| 写入键值 | `Set()` | 中 | `nativeapi_kv.h:29` |
| 删除键值 | `Delete()` | 中 | `nativeapi_kv.h:30` |
| 清空所有键值 | `Clear()` | 高 | `nativeapi_kv.h:31` |

**风险点**:
- Key 注入
- 数据泄露
- 存储空间耗尽
- 权限绕过

### 3. 设备信息读取

| 操作 | API | 风险等级 | 证据位置 |
|------|------|----------|----------|
| 读取设备信息 | `GetDeviceInfo()` | 中 | `nativeapi_deviceinfo.h:28` |
| 读取设备 UDID | JS 属性 `udid` | 高 | `nativeapi_ohos_deviceinfo.cpp:351-362` |

**风险点**:
- 敏感信息泄露（UDID、序列号）
- 设备追踪
- 用户隐私泄露

### 4. 定时器操作

| 操作 | API | 风险等级 | 证据位置 |
|------|------|----------|----------|
| 创建定时器 | `KalTimerCreate()` | 中 | `kal.h:39` |
| 启动定时器 | `KalTimerStart()` | 低 | `kal.h:40` |
| 修改定时器 | `KalTimerChange()` | 中 | `kal.h:41` |
| 停止定时器 | `KalTimerStop()` | 低 | `kal.h:42` |
| 删除定时器 | `KalTimerDelete()` | 低 | `kal.h:43` |

**风险点**:
- 回调函数注入
- 定时器资源耗尽
- 信号处理上下文中的不安全操作

### 5. 内存操作

| 操作 | API | 风险等级 | 证据位置 |
|------|------|----------|----------|
| 内存分配 | `malloc()` | 低 | `file/src/file_impl_hal/file.c:77` |
| 内存释放 | `free()` | 低 | `file/src/file_impl_hal/file.c:91` |
| C++ new | `new(std::nothrow)` | 低 | `nativeapi_fs.cpp:78` |
| C++ delete | `delete` | 低 | `nativeapi_fs.cpp:134` |

**风险点**:
- 缓冲区溢出
- Use-After-Free
- 双重释放
- 内存泄漏

---

## 信任边界

### 完整信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        🌐 外部世界（不信任）                          │
│                                                                       │
│  ┌──────────────────────┐                                            │
│  │  JS Application   │  ← 恶意应用可能存在                           │
│  │  (用户代码)        │                                            │
│  └────────┬───────────┘                                            │
│           │                                                         │
│           │ ⚠️ 不可信输入（文件路径、KV 键值、定时器参数）             │
│           │                                                         │
│  ┌────────▼────────────┐ 🔒 边界 1：JSI 参数解析与验证          │
│  │  JSI Framework    │  ← IsValidPath(), IsValidKey()              │
│  │  (信任边界)       │                                            │
│  └────────┬────────────┘                                            │
│           │                                                         │
│           │ ✅ 已验证输入                                             │
│           │                                                         │
│  ┌────────▼──────────────────────────────────────────────────────────┐   │
│  │  Native Modules (信任区域)                                      │   │
│  │                                                                 │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │   │
│  │  │  FileKit     │  │ KVStoreKit  │  │DeviceInfoKit │      │   │
│  │  │  (文件操作)   │  │ (KV 存储)   │  │ (设备信息)   │      │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │   │
│  │         │                 │                   │                │   │
│  └─────────┼─────────────────┼───────────────────┼────────────────┘   │
│            │                 │                   │                    │
│  ┌─────────▼─────────────────▼───────────────────▼────────────┐    │
│  │  C/C++ API (信任区域)                                   │    │
│  │                                                           │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │    │
│  │  │UtilsFile │  │UtilsKV   │  │TimerTask │           │    │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘           │    │
│  │       │               │               │               │    │
│  └───────┼───────────────┼───────────────┼───────────────┘    │
│          │               │               │                       │
│  ┌─────────▼─────────────────────────────────────────────────────┐   │
│  │  HAL/KAL Layer (信任区域)                              │   │
│  │                                                          │   │
│  │  ┌──────────┐  ┌──────────┐                           │   │
│  │  │ HAL File │  │ KAL Timer│                           │   │
│  │  │ (POSIX)  │  │ (POSIX)  │                           │   │
│  │  └────┬─────┘  └────┬─────┘                           │   │
│  │       │               │                                   │   │
│  └───────┼───────────────┼───────────────────────────────────┘   │
│          │               │                                       │
│  ┌─────────▼─────────────────────────────────────────────────────┐   │
│  │  🖥️  Kernel / Hardware (信任区域)                         │   │
│  │                                                           │   │
│  │  • 文件系统                                                  │   │
│  │  • 内核定时器                                                 │   │
│  │  • 硬件资源                                                  │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                   │
└───────────────────────────────────────────────────────────────────────────┘
```

### 边界点分析

| 边界 | 位置 | 验证机制 | 信任度 |
|------|------|----------|--------|
| 边界 1：JS → C | `js/builtin/*/src/*.cpp` | `IsValidPath()`, `IsValidKey()` | 部分信任 |
| 边界 2：C → HAL | `file/src/file_impl_hal/file.c` | 无直接验证 | 信任 |
| 边界 3：HAL → Kernel | `hals/file/hal_file.c` | POSIX 调用 | 信任 |

---

## 攻击路径示例

### 路径 1：路径遍历攻击

**入口**: JavaScript `file.move()` API

**攻击路径**:
```
JavaScript: file.move("internal://app/../../../etc/passwd", "test")
    ↓
JSI: IsValidPath("internal://app/../../../etc/passwd")
    ← ⚠️ 仅检查 `/../` 序列，可能被 URL 编码绕过
    ↓
C: GetFullPath() → UtilsFileMove() → HalFileMove()
    ↓
HAL: rename("/data/app/../../../etc/passwd", "/data/app/test")
    ↓
Kernel: 访问 /etc/passwd
    ← 🔓 成功访问沙箱外文件
```

**潜在绕过**:
- URL 编码：`%2e%2e`
- Unicode 变体：全角 `。。`
- 大小写混合

**详细分析**: 参见 [Security Review - 风险 1](07_Security_Review.md#风险-1-路径遍历漏洞)

---

### 路径 2：KV Key 注入攻击

**入口**: JavaScript `kvstore.set()` API

**攻击路径**:
```
JavaScript: kvstore.set("key/../../../", "value")
    ↓
JSI: IsValidKey("key/../../../")
    ← ⚠️ Key 验证可能不完整，未检查特殊字符组合
    ↓
C: GetFullPath() → SetValue()
    ↓
文件系统: 写入到 /data/app/key/../../../value
    ← 🔓 可能访问/覆盖其他应用的 KV 数据
```

**潜在绕过**:
- 特殊字符组合：`key/../../../`, `key\x00`, `key..`
- 编码绕过

**详细分析**: 参见 [Security Review - 风险 2](07_Security_Review.md#风险-2-kv-store-key-注入)

---

### 路径 3：设备信息泄露

**入口**: JavaScript `deviceInfo.udid` 属性

**攻击路径**:
```
JavaScript: const udid = deviceInfo.udid;
    ↓
JSI: GetDeviceInfo() → JSGetDevUdid()
    ← ⚠️ 无权限检查，任意应用可读取
    ↓
C: 读取系统 UDID
    ↓
JavaScript: 返回 UDID
    ← 🔓 敏感设备标识符泄露
```

**影响**:
- 设备追踪
- 用户隐私泄露
- 行为分析

**详细分析**: 参见 [Security Review - 风险 3](07_Security_Review.md#风险-3-设备信息泄露)

---

### 路径 4：文件描述符耗尽攻击

**入口**: JavaScript `file.open()` API

**攻击路径**:
```
JavaScript: for (let i = 0; i < 100; i++) {
               file.open("/test" + i);  // 未正确关闭
           }
    ↓
JSI: 重复调用 ExecuteAsyncWork()
    ↓
C: 多次调用 UtilsFileOpen()
    ← ⚠️ 未检查全局 fd 限制
    ↓
HAL: 重复调用 open()
    ↓
Kernel: fd 资源耗尽
    ← 🔓 拒绝服务（DoS）
```

**限制**: SPIFFS 最多 32 个同时打开的文件

**详细分析**: 参见 [Security Review - 风险 4](07_Security_Review.md#风险-4-文件描述符泄漏)

---

## 权限模型分析

### 当前权限模型

| 资源类型 | 访问控制 | 证据位置 |
|---------|---------|----------|
| 文件系统 | 路径前缀检查（`internal://app/`） | `nativeapi_fs.cpp:40-42` |
| KV 存储 | 应用隔离（基于数据路径） | `nativeapi_kv.cpp:53` |
| 设备信息 | 无权限检查，任意应用可读取 | `nativeapi_deviceinfo.cpp:351-362` |
| 定时器 | 无权限检查，任意应用可创建 | `nativeapi_timer_task.c:21-24` |

### 权限控制缺陷

| 缺陷 | 严重性 | 证据位置 |
|------|--------|----------|
| 设备信息无权限检查 | 中 | `nativeapi_deviceinfo.cpp:351-362` |
| 文件路径验证不完整 | 高 | `nativeapi_fs.cpp:30-50` |
| KV Key 验证不完整 | 中 | `nativeapi_kv.cpp:30-43` |
| 无全局 fd 限制 | 中 | `file/src/file_impl_hal/file.c:61-100` |

---

## 安全加固建议

### 1. 输入验证加固

**优先级**: 高

**建议**:
1. 使用 `realpath()` 规范化路径，验证规范化后的路径
2. 拒绝所有特殊字符，而非仅检查已知危险序列
3. 对输入进行白名单验证，而非黑名单
4. 添加长度限制的硬编码检查

**参考代码**:
```cpp
// ✅ 推荐：使用 realpath() 规范化路径
char resolvedPath[PATH_MAX];
if (realpath(userPath, resolvedPath) == NULL) {
    return ERROR_CODE_PARAM;
}

// 验证规范化后的路径是否在沙箱内
if (strncmp(resolvedPath, SANDBOX_PATH, strlen(SANDBOX_PATH)) != 0) {
    return ERROR_CODE_PARAM;
}
```

---

### 2. 权限检查加固

**优先级**: 高

**建议**:
1. 为敏感设备信息（如 `udid`, `serial`）添加权限检查
2. 实现 UID/PID 隔离，防止跨应用访问
3. 添加 Access Token 验证机制
4. 实现基于沙箱的访问控制列表

---

### 3. 资源限制加固

**优先级**: 中

**建议**:
1. 实现全局文件描述符限制检查
2. 添加超时自动关闭机制
3. 实现内存使用配额
4. 添加磁盘空间配额

**参考代码**:
```cpp
// ✅ 推荐：全局 fd 限制检查
static int g_openFdCount = 0;
#define MAX_OPEN_FDS 32

int UtilsFileOpen(const char* path, int oflag, int mode) {
    if (g_openFdCount >= MAX_OPEN_FDS) {
        return EC_FAILURE;  // 拒绝打开更多文件
    }

    int fd = HalFileOpen(path, oflag, mode);
    if (fd >= 0) {
        g_openFdCount++;
    }
    return fd;
}

int UtilsFileClose(int fd) {
    int ret = HalFileClose(fd);
    if (ret == 0) {
        g_openFdCount--;
    }
    return ret;
}
```

---

### 4. 错误处理加固

**优先级**: 中

**建议**:
1. 避免信息泄露（不在错误消息中泄露敏感信息）
2. 统一错误码，避免枚举攻击
3. 添加详细的审计日志
4. 实现错误率限制，防止枚举攻击

---

## 相关文档

- [Security Review](07_Security_Review.md) - 详细安全风险评估
- [Architecture](02_Architecture.md) - 架构设计与信任边界
- [NAPI Reference](03_NAPI_Reference.md) - 完整 API 参考
- [Inner API](04_Inner_API.md) - 内部 C API

---

## 附录

### A. 输入验证函数汇总

| 函数 | 位置 | 验证项 | 局限性 |
|------|------|---------|--------|
| `IsValidPath()` | `nativeapi_fs.cpp:30-50` | NULL, 长度, 前缀, `..`, 特殊字符 | 仅检查 `/../`, 可被绕过 |
| `IsValidKey()` | `nativeapi_kv.cpp:30-43` | NULL, 长度, 特殊字符 | 未检查特殊字符组合 |

### B. SPIFFS 限制

| 限制 | 值 | 证据位置 |
|------|-----|----------|
| 多级目录 | 不支持 | `utils_file.h:28` |
| 文件名最大长度 | 32 字节 | `utils_file.h:29` |
| 同时打开文件数 | 32 | `utils_file.h:30` |

### C. 常量定义

| 常量 | 值 | 位置 |
|-------|-----|------|
| `BUFFER_SIZE` | 128 | `file/src/file_impl_hal/file.c:24` |
| `FILE_PREFIX` | `internal://app/` | `nativeapi_config.h` |
| `PREFIX_LEN` | strlen(FILE_PREFIX) | `nativeapi_fs.cpp:28` |
| `URI_NAME_MAX_LEN` | (配置) | `nativeapi_config.h` |
| `KEY_MAX_LEN` | (配置) | `nativeapi_config.h` |

---

**最后更新**: 2026-02-06
**证据来源**: `wiki/_work/NOTES.md`
**维护者**: Sisyphus Agent

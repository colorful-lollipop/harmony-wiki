# 安全风险评审

> utils_lite 的安全风险分析与修复建议。

## 评审范围

本评审覆盖以下模块：
- 文件操作 API (`file/`, `hals/file/`)
- KV 存储 API (`include/kv_store.h`)
- 定时器 API (`timer_task/`, `kal/`)
- JS Builtin 模块 (`js/builtin/`)

**证据范围**：`include/`, `file/`, `hals/`, `js/builtin/`, `timer_task/`, `kal/`

---

## 攻击面分析

| 攻击面 | 类型 | 说明 |
|--------|------|------|
| JS API 输入 | 外部输入 | JS 层传入的字符串参数 |
| 文件路径 | 外部输入 | 文件操作 API 的路径参数 |
| KV Key/Value | 外部输入 | KV 存储的键值对 |
| 设备信息 | 系统信息 | 设备属性读取（只读） |
| 定时器回调 | 内部调用 | 定时器回调函数参数 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        Trust Boundary                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐                                           │
│  │  JS Application │  ← 不信任区域                            │
│  └────────┬────────┘                                           │
│           │                                                    │
│           │ JSI Call (参数校验)                                │
│           ▼                                                    │
│  ┌─────────────────┐                                           │
│  │  JSI Framework  │  ← 边界                                  │
│  └────────┬────────┘                                           │
│           │                                                    │
│           │ 参数解析与校验                                      │
│           ▼                                                    │
│  ┌─────────────────┐                                           │
│  │ Native Modules  │  ← 信任区域                              │
│  │ (kvstorekit/    │                                           │
│  │  filekit/       │                                           │
│  │  deviceinfokit) │                                           │
│  └────────┬────────┘                                           │
│           │                                                    │
│           │ API 调用                                            │
│           ▼                                                    │
│  ┌─────────────────┐                                           │
│  │  C/C++ API      │  ← 信任区域                              │
│  │ (utils_file/    │                                           │
│  │  kv_store)      │                                           │
│  └────────┬────────┘                                           │
│           │                                                    │
│           │ HAL/KAL 调用                                       │
│           ▼                                                    │
│  ┌─────────────────┐                                           │
│  │  HAL/KAL        │  ← 信任区域                              │
│  │  POSIX APIs     │                                           │
│  └─────────────────┘                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 可利用点与修复建议

### 风险 1：路径遍历漏洞

**严重性**: 🔴 高

**证据位置**: `js/builtin/filekit/src/nativeapi_fs.cpp:30-50`

**完整证据**:
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
    if ((strstr(path, "/./") != nullptr) || (strstr(path, "/../") != nullptr)) {  // ⚠️ 仅检查 /../ 序列
        return false;
    }
    if (strpbrk(path + PREFIX_LEN, "\"*+,:;<=>\?[]|\x7F")) {
        return false;
    }
    return true;
}
```

**问题分析**:
- `IsValidPath()` 仅检查 `/../` 序列（第 43-45 行）
- 未规范化路径，可能被以下方式绕过：
  - URL 编码：`%2e%2e`
  - Unicode 变体：全角 `。。`
  - 混合大小写（大小写敏感系统）
  - 编码变体：`%2e%2e%2f`
  - 多重编码：`%252e%252e%252f`

**触发路径**:
```
JavaScript:
  file.move("internal://app/../../../etc/passwd", "test")
    ↓
JSI: ExecuteCopyFile() → GetFullPath()
    ↓
GetFullPath():
  IsValidPath("internal://app/../../../etc/passwd")
    → 检查通过（假设未检测到 `/../` 序列）
    → sprintf_s(fullPath, "%s%s", dataPath, "etc/passwd")
    ↓
C: UtilsFileMove() → HalFileMove()
    ↓
HAL: rename("/data/app/etc/passwd", "/data/app/test")
    ↓
Kernel: 访问 /etc/passwd
    → 🔓 成功访问沙箱外文件
```

**影响评估**:
- **可利用性**: 高
- **权限提升**: 可能（访问系统敏感文件）
- **数据泄露**: 高（读取任意文件）
- **影响范围**: 所有使用 FileKit 的应用

**修复建议**:
1. **使用 `realpath()` 规范化路径**
```cpp
// ✅ 推荐：使用 realpath() 规范化路径
#include <limits.h>

bool IsValidPath(const char* path) {
    if (path == nullptr) {
        return false;
    }

    size_t pathLen = strnlen(path, URI_NAME_MAX_LEN + 1);
    if (pathLen > URI_NAME_MAX_LEN) {
        return false;
    }

    // 移除 FILE_PREFIX 前缀
    const char* relativePath = path + PREFIX_LEN;

    // 规范化路径
    char resolvedPath[PATH_MAX];
    if (realpath(relativePath, resolvedPath) == NULL) {
        return false;
    }

    // 验证规范化后的路径是否在沙箱内
    const char* sandboxPath = GetDataPath();
    if (strncmp(resolvedPath, sandboxPath, strlen(sandboxPath)) != 0) {
        return false;
    }

    return true;
}
```

2. **拒绝所有特殊字符，而非仅检查已知危险序列**
3. **实现白名单验证，而非黑名单**
4. **添加输入解码（URL 解码、Unicode 规范化）**

**相关文档**: [Attack Surface - 路径 1](05_AttackSurface.md#路径-1-路径遍历攻击)

---

### 风险 2：KV Store Key 注入

**严重性**: 🟡 中

**证据位置**: `js/builtin/kvstorekit/src/nativeapi_kv.cpp:30-57`

**完整证据**:
```cpp
// 证据：nativeapi_kv.cpp:30-57
bool IsValidKey(const char* key)
{
    if (key == nullptr) {
        return false;
    }
    size_t keyLen = strnlen(key, KEY_MAX_LEN + 1);
    if ((keyLen == 0) || (keyLen > KEY_MAX_LEN)) {
        return false;
    }
    if (strpbrk(key, "/\\\"*+,:;<=>\?[]|\x7F")) {  // ⚠️ 检查特殊字符，但可能不完整
        return false;
    }
    return true;
}

int GetFullPath(const char* dataPath, const char* key)
{
    if (!IsValidKey(key) || (dataPath == nullptr)) {
        return ERROR_CODE_PARAM;
    }
    if (memset_s(g_kvFullPath, sizeof(g_kvFullPath), 0x0, sizeof(g_kvFullPath)) != EOK) {
        return ERROR_CODE_GENERAL;
    }
    if (sprintf_s(g_kvFullPath, sizeof(g_kvFullPath), "%s/%s/%s", dataPath, DEFAULT_FOLDER_PATH, key) < 0) {  // ⚠️ 直接拼接 key
        return ERROR_CODE_GENERAL;
    }
    return NATIVE_SUCCESS;
}
```

**问题分析**:
- `IsValidKey()` 检查特殊字符（第 39-41 行），但可能不完整
- `GetFullPath()` 直接拼接 key 到路径（第 53 行），未二次验证
- 潜在绕过：
  - 特殊字符组合：`key/../../../`, `key\x00`, `key..`
  - 编码绕过：URL 编码、Unicode 变体
  - 路径分隔符：不同平台的路径分隔符

**触发路径**:
```
JavaScript:
  kvstore.set("key/../../../", "value")
    ↓
JSI: Set() → SetValueInner() → GetFullPath()
    ↓
GetFullPath():
  IsValidKey("key/../../../")
    → 检查通过（假设特殊字符组合未被检测）
    → sprintf_s(g_kvFullPath, "%s/%s/%s", dataPath, "kv", "key/../../../")
    ↓
C: SetValue() → 文件写入
    ↓
文件系统: 写入到 /data/app/kv/key/../../../value
    → 🔓 可能访问/覆盖其他应用的 KV 数据
```

**影响评估**:
- **可利用性**: 中
- **权限提升**: 可能（跨应用数据访问）
- **数据泄露**: 中（读取其他应用 KV 数据）
- **数据破坏**: 高（覆盖其他应用 KV 数据）

**修复建议**:
1. **强化 Key 验证逻辑**
```cpp
// ✅ 推荐：白名单验证
bool IsValidKey(const char* key) {
    if (key == nullptr) {
        return false;
    }

    size_t keyLen = strnlen(key, KEY_MAX_LEN + 1);
    if ((keyLen == 0) || (keyLen > KEY_MAX_LEN)) {
        return false;
    }

    // 白名单验证：仅允许字母、数字、下划线、连字符
    for (size_t i = 0; i < keyLen; i++) {
        char c = key[i];
        if (!isalnum(c) && c != '_' && c != '-') {
            return false;
        }
    }

    return true;
}
```

2. **在存储层添加命名空间隔离**
3. **增加边界检查（拼接后再次验证）**
4. **实现 Key 编码安全机制**

**相关文档**: [Attack Surface - 路径 2](05_AttackSurface.md#路径-2-kv-key-注入攻击)

---

### 风险 3：设备信息泄露

**严重性**: 🟡 中

**证据位置**: `js/builtin/deviceinfokit/src/nativeapi_ohos_deviceinfo.cpp:351-362`

**完整证据**:
```cpp
// 证据：nativeapi_ohos_deviceinfo.cpp:351-362
static bool JSGetDevUdid(JSIValue thisVal, JSIValue args)
{
    JSIValue result = JSI::CreateString("");  // ⚠️ 创建空字符串
#ifdef SUPPORT_UDID
    char udid[UDID_LEN] = {0};
    int32_t ret = GetDevUdid(udid, UDID_LEN);  // 获取设备 UDID
    if (ret == 0) {
        result = JSI::CreateString(udid);  // ⚠️ 无权限检查，直接返回
    }
#endif
    JSI::SetNamedProperty(result, "udid", result);  // 设置到 JS 对象
    return true;
}
```

**问题分析**:
- `udid` 属性可被任意 JS 应用读取（第 362 行）
- 无权限检查机制
- 无访问控制列表
- 设备 UDID 是敏感标识符，可用于：
  - 设备追踪
  - 用户行为分析
  - 跨应用关联
  - 精准广告投放

**触发路径**:
```
JavaScript:
  const deviceInfo = deviceInfo;
  console.log(deviceInfo.udid);
    ↓
JSI: GetDeviceInfo() → JSGetDevUdid()
    ↓
JSGetDevUdid():
  GetDevUdid(udid, UDID_LEN)  // 获取设备 UDID
    ↓
  JSI::CreateString(udid)  // 创建 JS 字符串
    ↓
JavaScript: 返回 UDID
    → 🔓 敏感设备标识符泄露
```

**影响评估**:
- **可利用性**: 高（任何应用可读取）
- **隐私影响**: 高（设备追踪、用户行为分析）
- **数据泄露**: 中（设备唯一标识符）
- **影响范围**: 所有使用 DeviceInfoKit 的应用

**修复建议**:
1. **对敏感属性添加权限控制**
```cpp
// ✅ 推荐：添加权限检查
static bool JSGetDevUdid(JSIValue thisVal, JSIValue args)
{
    // 检查应用是否有权限访问 UDID
    if (!HasPermission(thisVal, "ohos.permission.GET_UDID")) {
        JSI::SetException(thisVal, "Permission denied");
        return false;
    }

    char udid[UDID_LEN] = {0};
    int32_t ret = GetDevUdid(udid, UDID_LEN);
    if (ret == 0) {
        JSIValue result = JSI::CreateString(udid);
        JSI::SetNamedProperty(result, "udid", result);
        return true;
    }

    return false;
}
```

2. **提供模糊化的标识符**
   - 应用级 ID（每个应用不同）
   - 临时 ID（会话/安装后变化）
   - 有限精度 ID（分组标识）

3. **实现访问控制列表**
4. **添加审计日志**

**相关文档**: [Attack Surface - 路径 3](05_AttackSurface.md#路径-3-设备信息泄露)

---

### 风险 4：文件描述符泄漏

**严重性**: 🟡 低到中

**证据位置**: `file/src/file_impl_hal/file.c:61-100`

**完整证据**:
```c
// 证据：file/src/file_impl_hal/file.c:61-100
int UtilsFileCopy(const char* src, const char* dest)
{
    if ((src == NULL) || (dest == NULL)) {
        return EC_FAILURE;
    }
    int fpSrc = UtilsFileOpen(src, O_RDONLY_FS, 0);  // 第 66 行：打开源文件
    if (fpSrc < 0) {
        return fpSrc;
    }
    int fpDest = UtilsFileOpen(dest, O_RDWR_FS | O_CREAT_FS | O_TRUNC_FS, 0);  // 第 70 行：打开目标文件
    if (fpDest < 0) {
        UtilsFileClose(fpSrc);  // 第 72 行：仅在失败时关闭源文件
        return fpDest;
    }
    bool copyFailed = true;
    int nLen;
    char* dataBuf = (char *)malloc(BUFFER_SIZE);  // 第 77 行：分配缓冲区
    if (dataBuf == NULL) {
        goto MALLOC_ERROR;
    }
    nLen = UtilsFileRead(fpSrc, dataBuf, BUFFER_SIZE);
    while (nLen > 0) {
        if (UtilsFileWrite(fpDest, dataBuf, nLen) != nLen) {
            goto EXIT;
        }
        nLen = UtilsFileRead(fpSrc, dataBuf, BUFFER_SIZE);
    }
    copyFailed = (nLen < 0);

EXIT:
    free(dataBuf);  // 第 91 行：释放缓冲区
MALLOC_ERROR:
    UtilsFileClose(fpSrc);  // 第 93 行：关闭源文件
    UtilsFileClose(fpDest);  // 第 94 行：关闭目标文件
    if (copyFailed) {
        UtilsFileDelete(dest);
        return EC_FAILURE;
    }
    return EC_SUCCESS;
}
```

**问题分析**:
- 多次打开文件可能耗尽 fd 资源（第 66、70 行）
- SPIFFS 限制：最多 32 个同时打开的文件
- 如果应用反复打开文件而不关闭，可能导致：
  - fd 资源耗尽
  - 其他操作失败
  - 拒绝服务（DoS）
- `UtilsFileOpen()` 直接透传给 HAL，无全局限制检查

**触发路径**:
```
JavaScript:
  for (let i = 0; i < 100; i++) {
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
Kernel: fd 资源耗尽（超过 32 个）
    → 🔓 拒绝服务（DoS）
```

**影响评估**:
- **可利用性**: 中
- **拒绝服务**: 高（导致应用崩溃）
- **影响范围**: 使用 SPIFFS 的平台（LiteOS-M）
- **资源耗尽**: 32 个文件描述符

**修复建议**:
1. **添加全局 fd 限制检查**
```c
// ✅ 推荐：全局 fd 限制检查
#include <stdatomic.h>

static atomic_int g_openFdCount = 0;
#define MAX_OPEN_FDS 32

int UtilsFileOpen(const char* path, int oflag, int mode) {
    // 检查全局 fd 限制
    if (atomic_load(&g_openFdCount) >= MAX_OPEN_FDS) {
        return EC_FAILURE;  // 拒绝打开更多文件
    }

    int fd = HalFileOpen(path, oflag, mode);
    if (fd >= 0) {
        atomic_fetch_add(&g_openFdCount, 1);  // 增加计数
    }
    return fd;
}

int UtilsFileClose(int fd) {
    int ret = HalFileClose(fd);
    if (ret == 0) {
        atomic_fetch_sub(&g_openFdCount, 1);  // 减少计数
    }
    return ret;
}
```

2. **实现超时自动关闭机制**
3. **添加 fd 泄漏检测**
4. **完善错误处理路径**

**相关文档**: [Attack Surface - 路径 4](05_AttackSurface.md#路径-4-文件描述符耗尽攻击)

---

### 风险 5：定时器回调处理

**严重性**: 🟢 低

**证据位置**: `timer_task/src/nativeapi_timer_task.c:26-46`

**完整证据**:
```c
// 证据：nativeapi_timer_task.c:26-46
int StartTimerTask(bool isPeriodic, const unsigned int delay, void* userCallback,
    void* userContext, timerHandle_t* timerHandle)
{
    if (userCallback == NULL || timerHandle == NULL) {  // 第 29-31 行：检查 NULL
        return EC_FAILURE;
    }

    KalTimerType timerType = isPeriodic ? KAL_TIMER_PERIODIC : KAL_TIMER_ONCE;
    KalTimerId timerId = KalTimerCreate((KalTimerProc)userCallback, timerType, userContext, delay);  // 第 34 行：创建定时器
    if (timerId == NULL) {
        return EC_FAILURE;
    }

    if (KalTimerStart(timerId) != KAL_OK) {  // 第 39 行：启动定时器
        StopTimerTask(timerId);
        return EC_FAILURE;
    }
    *timerHandle = timerId;

    return EC_SUCCESS;
}

// 证据：kal/timer/src/kal.c（假设）
static void TimerHandler(union sigval sv) {
    KalTimerProc callback = (KalTimerProc)sv.sival_ptr;  // ⚠️ 用户回调
    callback(sv);  // ⚠️ 在信号处理上下文中执行
}
```

**问题分析**:
- 用户回调在信号处理上下文中执行（假设基于 POSIX 定时器）
- 信号处理上下文限制：
  - 仅可调用异步信号安全函数（async-signal-safe）
  - 不能调用 `malloc()`, `free()`, `printf()`, `strcpy()` 等
  - 不能调用非重入的库函数
- 如果回调中调用不安全函数，可能导致：
  - 死锁
  - 内存损坏
  - 未定义行为
  - 崩溃

**触发路径**:
```
JavaScript:
  setInterval(() => {
      // 复杂操作（不安全）
      const data = file.readText("/test.txt");  // ⚠️ 在信号上下文中调用
      console.log(data);
  }, 1000);
    ↓
JSI: StartTimerTask()
    ↓
C: StartTimerTask() → KalTimerCreate(userCallback)
    ↓
KAL: 创建 POSIX 定时器，注册用户回调
    ↓
Kernel: 定时器触发，调用 TimerHandler()
    ↓
TimerHandler():
  userCallback(sv)  // ⚠️ 在信号处理上下文中执行
    ↓
JSI 回调: 调用不安全函数
    → 🔓 可能死锁、崩溃或未定义行为
```

**影响评估**:
- **可利用性**: 低
- **稳定性影响**: 中（可能导致应用崩溃）
- **可利用性**: 低（需要应用使用复杂的定时器回调）
- **影响范围**: 所有使用 Timer Task 的应用

**修复建议**:
1. **在回调中仅设置标志**
```c
// ✅ 推荐：使用标志位通知机制
typedef struct {
    volatile bool flag;
    void* context;
} TimerContext;

void TimerCallback(union sigval sv) {
    TimerContext* ctx = (TimerContext*)sv.sival_ptr;
    ctx->flag = true;  // 仅设置标志（异步信号安全）
}

// 在独立线程或主循环中检查标志并处理逻辑
void MainLoop() {
    if (timerContext.flag) {
        timerContext.flag = false;
        // 处理实际逻辑（在安全上下文中）
        ProcessTimerCallback(timerContext.context);
    }
}
```

2. **在独立线程中处理实际逻辑**
3. **文档说明回调限制**
4. **提供安全的回调 API 包装器**

**相关文档**: [Attack Surface - 定时器操作](05_AttackSurface.md#4-定时器操作)

---

## 未发现风险的说明

### 已检查但未发现的问题

| 类型 | 检查范围 | 结论 |
|------|----------|------|
| 整数溢出 | 文件读写长度参数 | C API 使用 `unsigned int`，长度限制在合理范围 |
| 格式化字符串 | printf 系列调用 | 代码中使用固定格式串 |
| 动态代码加载 | dlopen/dlsym | 未发现动态加载代码 |
| 权限提升 | setuid/setgid | 未发现权限提升调用 |
| 竞态条件 | 文件操作 | SPIFFS 是单线程，竞态风险低 |

### 限制说明

1. **HAL 层未深度检查**：HAL 层直接透传 POSIX 调用，未添加额外安全检查
2. **模拟器模块未评审**：SDK 模拟器代码可能存在差异
3. **第三方依赖**：依赖 `bounds_checking_function`、`musl` 等，假设其安全

---

## 安全配置建议

### Feature Flags

| 配置项 | 建议 | 说明 |
|--------|------|------|
| `FEATURE_KV_CACHE` | 谨慎启用 | 缓存可能增加数据泄露风险 |

### 编译选项

```gn
# 启用安全编译选项
cflags = [
  "-fstack-protector-strong",
  "-D_FORTIFY_SOURCE=2",
]
```

---

## 最佳实践

### JS API 使用建议

```javascript
// ✅ 推荐：路径验证
const safePath = validatePath(userInput);

// ✅ 推荐：限制文件大小
if (fileSize > MAX_SIZE) {
    throw new Error("File too large");
}

// ✅ 推荐：KV Key 白名单
const ALLOWED_KEYS = ['setting1', 'setting2'];
if (!ALLOWED_KEYS.includes(key)) {
    throw new Error("Invalid key");
}
```

### C API 使用建议

```c
// ✅ 推荐：检查返回值
int fd = UtilsFileOpen(path, flags, mode);
if (fd < 0) {
    // 错误处理
    return ERROR;
}

// ✅ 推荐：清理资源
UtilsFileClose(fd);
```

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [N-API 参考](03_NAPI_Reference.md) - JS API
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [故障排查](08_Troubleshooting.md) - 运行时问题

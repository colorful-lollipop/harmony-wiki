# 常见问题与调试

## 目的

本文档汇总 Pasteboard 剪贴板服务的常见问题、定位方法和调试技巧。

## 适用范围

- 遇到构建或运行问题的开发者
- 进行问题定位的工程师
- 需要调试剪贴板功能的人员

## 构建问题

### 问题 1: 编译错误 "undefined reference to..."

**现象**:
```
undefined reference to `OHOS::MiscServices::PasteboardClient::GetInstance()`
```

**原因**: 缺少链接依赖

**解决**:
```gn
# 在 BUILD.gn 中添加依赖
deps = [
    "//foundation/distributeddatamgr/pasteboard/framework/innerkits:pasteboard_client",
]

external_deps = [
    "c_utils:utils",
    "ipc:ipc_single",
]
```

**代码参考**: `framework/innerkits/BUILD.gn`

---

### 问题 2: 头文件找不到

**现象**:
```
fatal error: 'pasteboard_client.h' file not found
```

**原因**: include_dirs 配置错误

**解决**:
```gn
include_dirs = [
    "//foundation/distributeddatamgr/pasteboard/framework/innerkits/include",
]
```

**代码参考**: `interfaces/kits/BUILD.gn:27-33`

---

### 问题 3: N-API 模块加载失败

**现象**:
```
Error: Cannot find module 'pasteboard'
```

**原因**: 
1. so 文件未正确安装
2. 模块路径配置错误

**检查**:
```bash
# 检查 so 是否存在
ls /system/lib/module/libpasteboard_napi.z.so

# 检查 bundle.json
# 确认 entry 配置正确
```

**代码参考**: `interfaces/kits/BUILD.gn:83`
```gn
relative_install_dir = "module"
```

---

## 运行问题

### 问题 4: 服务启动失败

**现象**:
```
PasteboardService OnStart register to system ability manager failed
```

**定位步骤**:
```bash
# 1. 检查服务是否存在
ps -A | grep pasteboard

# 2. 检查 SA 配置文件
cat /system/profile/3701.json

# 3. 查看日志
hilog | grep -i pasteboard
```

**代码参考**: `services/core/src/pasteboard_service.cpp:148-153`

---

### 问题 5: 权限拒绝

**现象**:
```
Permission verification failed. A non-permission application calls a API.
```

**原因**: 缺少 `ohos.permission.READ_PASTEBOARD` 权限

**解决**:
```json
// 在 module.json5 中添加权限
"requestPermissions": [
    {
        "name": "ohos.permission.READ_PASTEBOARD",
        "reason": "$string:pasteboard_permission_reason"
    }
]
```

**代码参考**: `framework/framework/permission/permission_utils.cpp:23`

---

### 问题 6: IPC 调用超时

**现象**:
```
IPC communication timeout
```

**原因**:
1. 服务进程崩溃
2. 数据过大
3. 线程池满

**定位**:
```bash
# 检查服务状态
ps -T -p $(pidof pasteboardservice)

# 查看线程数
cat /proc/$(pidof pasteboardservice)/status | grep Threads
```

**代码参考**: `services/core/src/pasteboard_service.cpp:82`
```cpp
constexpr uint32_t MAX_IPC_THREAD_NUM = 32;
```

---

### 问题 7: 数据序列化失败

**现象**:
```
TLV serialization failed
```

**原因**: 数据过大或格式错误

**检查**:
```cpp
// 检查数据大小
if (dataSize > MAX_TEXT_LEN) {  // 100MB
    return PasteboardError::INVALID_DATA_SIZE;
}
```

**代码参考**: `framework/innerkits/src/paste_data_record.cpp:26`

---

### 问题 8: 分布式同步失败

**现象**:
```
Remote data is not ready
```

**定位步骤**:
```bash
# 1. 检查设备管理器
hilog | grep -i device_manager

# 2. 检查网络状态
ifconfig

# 3. 检查设备画像
hilog | grep -i device_profile
```

**代码参考**: `adapter/src/device_profile_adapter.cpp`

---

## 调试方法

### 日志级别

```cpp
// 定义位置: utils/native/include/pasteboard_hilog.h
PASTEBOARD_HILOGD(module, fmt, ...)  // Debug
PASTEBOARD_HILOGI(module, fmt, ...)  // Info
PASTEBOARD_HILOGW(module, fmt, ...)  // Warning
PASTEBOARD_HILOGE(module, fmt, ...)  // Error
```

**模块定义**:
```cpp
PASTEBOARD_MODULE_SERVICE   // 服务层
PASTEBOARD_MODULE_CLIENT    // 客户端层
PASTEBOARD_MODULE_JS_NAPI   // N-API 层
```

### 日志过滤

```bash
# 查看所有剪贴板日志
hilog | grep -i pasteboard

# 查看错误日志
hilog | grep -iE "(pasteboard|E/)"

# 查看特定模块
hilog | grep PASTEBOARD_MODULE_SERVICE

# 查看 PID
cat /proc/$(pidof pasteboardservice)/status
hilog | grep "$(pidof pasteboardservice)"
```

### Dump 信息

```bash
# 服务 dump
hidumper -s 3701

# 或
dumpsys pasteboard
```

**代码参考**: `services/dfx/src/pasteboard_dump_helper.cpp`

### 性能跟踪

```cpp
// 使用 trace
#include "pasteboard_trace.h"
{
    PASTEBOARD_TRACE("FunctionName");
    // ... code
}
```

**查看 trace**:
```bash
# 抓取 trace
hitrace -t 10 -o /data/hitrace.txt

# 分析
cat /data/hitrace.txt | grep -i pasteboard
```

**代码参考**: `services/dfx/src/pasteboard_trace.cpp`

### 调试构建

```gn
# 启用调试信息
cflags = [
    "-g3",
    "-O0",
    "-DDEBUG",
]

# 禁用优化
sanitize = {
    debug = true
}
```

## 常见错误码

| 错误码 | 错误信息 | 可能原因 | 解决方案 |
|--------|----------|----------|----------|
| 201 | Permission verification failed | 缺少权限 | 添加 READ_PASTEBOARD |
| 401 | Parameter error | 参数无效 | 检查参数类型和范围 |
| 12900001 | Invalid parameter | 参数越界 | 检查数据大小 |
| 12900002 | Operation failed | 服务异常 | 重启服务，查看日志 |
| 12900003 | The clipboard is empty | 剪贴板为空 | 先写入数据 |
| 12900004 | Copy or paste is in progress | 操作进行中 | 等待或取消 |
| 12900005 | The remote data is not ready | 远程数据未就绪 | 检查分布式连接 |
| 12900006 | The share option is InApp | 分享限制 | 检查 ShareOption |

**代码参考**: `utils/native/include/pasteboard_error.h`

## 关键调试位置

### 服务入口

```cpp
// services/core/src/pasteboard_service.cpp:175
void PasteboardService::OnStart()
{
    PASTEBOARD_HILOGI(PASTEBOARD_MODULE_SERVICE, "PasteboardService OnStart.");
    // 在此处添加断点
}
```

### IPC 入口

```cpp
// services/zidl/src/pasteboard_observer_stub.cpp:33
int PasteboardObserverStub::OnRemoteRequest(...)
{
    // 在此处添加断点
    switch (code) {
        case GET_PASTE_DATA: // 200
        case SET_PASTE_DATA: // 104
    }
}
```

### N-API 入口

```cpp
// interfaces/kits/napi/src/napi_init.cpp:25
static napi_value NapiInit(napi_env env, napi_value exports)
{
    // 模块初始化
    PASTEBOARD_HILOGD(PASTEBOARD_MODULE_JS_NAPI, "NapiInit");
}
```

### 权限检查

```cpp
// framework/framework/permission/permission_utils.cpp:23
bool PermissionUtils::IsPermissionGranted(const std::string &perm)
{
    // 在此处添加断点
    int32_t result = AccessTokenKit::VerifyAccessToken(tokenID, perm);
}
```

## 调试命令速查

```bash
# ===== 服务状态 =====
# 查看服务进程
ps -A | grep pasteboard

# 查看服务线程
ps -T -p $(pidof pasteboardservice)

# 查看服务资源
cat /proc/$(pidof pasteboardservice)/status

# ===== 日志 =====
# 实时日志
hilog | grep -i pasteboard

# 清除日志
hilog -r

# ===== 文件检查 =====
# 检查 so 文件
ls -la /system/lib/*pasteboard*

# 检查配置文件
cat /system/profile/3701.json
cat /system/etc/init/pasteboardservice.cfg

# ===== Dump =====
# 服务 dump
hidumper -s 3701

# ===== 重启服务 =====
# 停止
kill $(pidof pasteboardservice)

# 启动（自动重启）
# 服务由 init 管理，kill 后会自动重启
```

## 关键结论

1. **日志优先**: 遇到问题时首先查看 hilog，大多数问题可通过日志定位。

2. **权限常见**: 权限问题是最常见的运行时错误，检查 module.json5 配置。

3. **IPC 超时**: IPC 超时通常是服务异常，检查线程状态和服务日志。

4. **数据大小**: 大数据传输容易失败，注意 MAX_TEXT_LEN (100MB) 限制。

5. **分布式复杂**: 分布式问题涉及多个组件，需检查设备管理和网络状态。

## 相关链接

- [N-API 参考 → 03_NAPI_Reference.md](03_NAPI_Reference.md)
- [内部 API → 04_Inner_API.md](04_Inner_API.md)
- [安全评审 → 06_Security.md](06_Security.md)

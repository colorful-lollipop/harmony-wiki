# 附录：调用链图谱

> utils_lite 关键调用链（入口→核心逻辑）

## 文件操作调用链

### UtilsFileOpen

```
JS: file.move(src, dest)
    │
    ▼
JSI: nativeapi_fs.cpp:532-535
    │
    ▼
NativeapiFs::MoveFile(args)
    │
    ▼
ExecuteAsyncWork() → JsAsyncWork::DispatchAsyncWork()
    │
    ▼
ExecuteCopyFile(data) [nativeapi_fs.cpp:88-135]
    │
    ├── UtilsFileOpen(src) [file.c:26]
    │       │
    │       └── HalFileOpen(path, oflag, mode) [hal_file.c:21-25]
    │               │
    │               └── POSIX open(path, flags, mode)
    │
    ├── UtilsFileRead(fd, buf, len) [file.c:36-44]
    │       └── HalFileRead(fd, buf, len) → POSIX read()
    │
    ├── UtilsFileOpen(dest, O_CREAT|O_WRONLY)
    │       └── HalFileOpen() → POSIX open()
    │
    ├── UtilsFileWrite(fd, buf, len) → POSIX write()
    │
    └── UtilsFileClose(src_fd), UtilsFileClose(dest_fd)
```

**证据来源**：
- JS 注册：`js/builtin/filekit/src/nativeapi_fs.cpp:513-530`
- 实现：`js/builtin/filekit/src/nativeapi_fs.cpp:88-135`
- UtilsFile：`file/src/file_impl_hal/file.c:26-44`
- HAL：`hals/file/hal_file.c:21-40`

---

### UtilsFileCopy

```
UtilsFileCopy(src, dest) [utils_file.h:254]
    │
    ▼
file.c:61-100
    │
    ├── UtilsFileOpen(src, O_RDONLY) [line 63-65]
    │       └── HalFileOpen() → POSIX open()
    │
    ├── UtilsFileOpen(dest, O_CREAT|O_WRONLY|O_TRUNC) [line 67-69]
    │
    ├── Loop: UtilsFileRead() → UtilsFileWrite() [line 72-88]
    │       128-byte buffer
    │
    └── UtilsFileClose(src_fd), UtilsFileClose(dest_fd) [line 92-96]
```

**证据来源**：`file/src/file_impl_hal/file.c:61-100`

---

## KV 存储调用链

### UtilsSetValue

```
JS: kvstore.set(key, value)
    │
    ▼
JSI: nativeapi_kv.cpp:268-271
    │
    ▼
NativeapiKv::Set(args)
    │
    ▼
ExecuteAsyncWork() → JsAsyncWork::DispatchAsyncWork()
    │
    ▼
ExecuteSet(data) [nativeapi_kv.cpp:69-116]
    │
    ├── IsValidKey(key) [line 30-43]
    │       └── 检查长度 1-32，字符集
    │
    ├── IsValidValue(value) [nativeapi_kv_impl.c:33-43]
    │       └── 检查长度 1-128
    │
    ├── UtilsSetValue(key, value) → 文件系统 [line 107]
    │
    └── SuccessCallBack() / FailCallBack()
```

**证据来源**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp`

---

### UtilsGetValue

```
JS: kvstore.get(key)
    │
    ▼
JSI: nativeapi_kv.cpp:263-266
    │
    ▼
NativeapiKv::Get(args)
    │
    ▼
ExecuteAsyncWork() → JsAsyncWork::DispatchAsyncWork()
    │
    ▼
ExecuteGet(data) [nativeapi_kv.cpp:118-156]
    │
    ├── IsValidKey(key)
    │
    ├── UtilsGetValue(key, value, len) [line 126-130]
    │       └── 从文件读取 KV 数据
    │
    └── SuccessCallBack(value)
```

**证据来源**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp:118-156`

---

## 定时器调用链

### StartTimerTask

```
JS: setInterval(callback, delay)
    │
    ▼
JSI: timer_task 相关
    │
    ▼
StartTimerTask(isPeriodic, delay, callback, context, handle)
    │
    ▼
nativeapi_timer_task.c:26-46
    │
    ├── KalTimerCreate(callback, type, arg, millisec) [line 33]
    │       │
    │       └── kal.c:50-76
    │               │
    │               └── timer_create(CLOCK_REALTIME, &sevp, &timerId)
    │                       │
    │                       └── POSIX timer_create()
    │
    └── KalTimerStart(timerId) [line 39]
            │
            └── kal.c:78-97
                    │
                    └── timer_settime(timerId, 0, &its, NULL)
                            │
                            └── POSIX timer_settime()
```

**证据来源**：
- Timer Task：`timer_task/src/nativeapi_timer_task.c:26-46`
- KAL：`kal/timer/src/kal.c:50-97`

---

### StopTimerTask

```
JS: clearInterval(timerId)
    │
    ▼
StopTimerTask(handle)
    │
    ▼
nativeapi_timer_task.c:48-54
    │
    └── KalTimerDelete(timerId)
            │
            └── kal.c:139-148
                    │
                    └── timer_delete(timerId)
                            │
                            └── POSIX timer_delete()
```

---

## 设备信息调用链

### getInfo()

```
JS: deviceInfo.getInfo()
    │
    ▼
JSI: nativeapi_deviceinfo.cpp:82-85
    │
    ▼
NativeapiDeviceInfo::GetDeviceInfo(args)
    │
    ▼
ExecuteAsyncWork() → JsAsyncWork::DispatchAsyncWork()
    │
    ▼
NativeapiDeviceInfo::ExecuteGetDeviceInfo(data)
    │
    ├── 获取 40+ 设备属性
    │   ├── JSGetDeviceType() → 系统属性
    │   ├── JSGetManufacture() → 系统属性
    │   ├── JSGetUdid() → 设备标识
    │   └── ... 更多属性
    │
    └── 返回 DeviceInfo 对象
```

**证据来源**：`js/builtin/deviceinfokit/src/nativeapi_ohos_deviceinfo.cpp`

---

## 回调处理调用链

### SuccessCallBack

```
NativeapiCommon::SuccessCallBack(thisVal, retVal)
    │
    ▼
nativeapi_common.cpp:57-75
    │
    ├── 检查 IsValidJSIValue(retVal)
    │
    ├── 获取回调函数: JSI::GetNamedProperty(thisVal, "success")
    │
    └── JSI::CallFunction(cb, thisVal, retVal)
            │
            └── 返回 JS Promise.resolve()
```

**证据来源**：`js/builtin/common/src/nativeapi_common.cpp:57-75`

---

### FailCallBack

```
NativeapiCommon::FailCallBack(thisVal, errCode, errMsg)
    │
    ▼
nativeapi_common.cpp:22-55
    │
    ├── 创建错误对象
    │   ├── code: 错误码
    │   └── message: 错误信息
    │
    ├── 获取回调函数: JSI::GetNamedProperty(thisVal, "fail")
    │
    └── JSI::CallFunction(cb, thisVal, errorObj)
            │
            └── 返回 JS Promise.reject()
```

**证据来源**：`js/builtin/common/src/nativeapi_common.cpp:22-55`

---

## 完整调用层次

```
┌─────────────────────────────────────────────────────────────────────┐
│                         JavaScript Layer                            │
│   kvstore.set(), file.move(), deviceInfo.getInfo(), setInterval()  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      JSI Framework Layer                            │
│   JSI::SetModuleAPI(), JsAsyncWork::DispatchAsyncWork()             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     Native Module Layer                              │
│   NativeapiKv::Set(), NativeapiFs::MoveFile(),                     │
│   NativeapiDeviceInfo::GetDeviceInfo()                              │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        C/C++ API Layer                              │
│   UtilsSetValue(), UtilsFileOpen(), UtilsGetValue(),               │
│   StartTimerTask()                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │   HAL API   │ │  KAL API    │ │   KV API    │
            │ hal_file.*  │ │  kal.*      │ │   文件存储  │
            └─────────────┘ └─────────────┘ └─────────────┘
                    │               │               │
                    ▼               ▼               ▼
            ┌─────────────────────────────────────────────────────┐
            │              POSIX / System API                     │
            │   open(), read(), write(), close(), timer_create() │
            └─────────────────────────────────────────────────────┘
```

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [架构说明](02_Architecture.md) - 组件关系
- [N-API 参考](03_NAPI_Reference.md) - JS API
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [故障排查](08_Troubleshooting.md) - 问题定位

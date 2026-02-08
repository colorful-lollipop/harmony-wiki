# 常见问题与调试

> 本文档整理 multimedia_cangjie_wrapper 的常见问题、排查路径与调试技巧

---

## 目的与适用范围

**目的**：帮助开发者快速定位和解决问题

**适用范围**：应用开发者、框架开发者

---

## 常见问题

### Q1: Camera 创建失败，返回错误码 7400101

**现象**：
```
BusinessException: 7400101 - Parameter missing or parameter type incorrect.
```

**可能原因**：
1. 传入的 CameraDevice 对象为 null
2. CameraDevice 参数已被释放
3. 参数类型不匹配

**排查路径**：
1. 检查 `getSupportedCameras()` 返回的设备是否有效
2. 确认 `createCameraInput()` 前设备对象未被释放
3. 检查权限 `ohos.permission.CAMERA` 是否已申请

**证据**：`ohos/multimedia/camera/camera_manager.cj:149-163`
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.CAMERA",
    // ...
]
public func createCameraInput(camera: CameraDevice): CameraInput
```

---

### Q2: 图像解码失败，错误码 62980006

**现象**：
```
BusinessException: 62980006 - Image decoding failed
```

**可能原因**：
1. 图像文件损坏
2. 图像格式不支持
3. 解码参数错误

**排查路径**：
1. 检查图像文件是否完整
2. 确认图像格式在支持列表（JPEG、PNG、GIF、WebP、HEIC、RAW）
3. 检查 `DecodingOptions` 参数是否有效

**证据**：`ohos/multimedia/image/cj_image_common.cj:430-438`
```cangjie
func parseDecodeOptions() {
    if (rotate < 0 || rotate > 360) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid rotate ${rotate}")
    }
}
```

---

### Q3: 相册访问返回空列表

**现象**：
```
getAssets() 返回 PhotoAssetResult，但 getCount() 为 0
```

**可能原因**：
1. 权限未申请
2. FetchOptions 条件过于严格
3. 相册为空

**排查路径**：
1. 检查是否申请了 `ohos.permission.READ_IMAGEVIDEO` 权限
2. 检查 `FetchOptions.predicates` 条件
3. 使用系统相册应用确认是否有照片

**证据**：`ohos/file/photo_access_helper/photo_accesshelper.cj:83-99`
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.READ_IMAGEVIDEO",
    // ...
]
public func getAssets(options: FetchOptions): PhotoAssetResult
```

---

### Q4: 回调不触发

**现象**：
```
注册了 on(CameraEvents.CameraStatus) 回调，但状态变化时未触发
```

**可能原因**：
1. 回调被提前注销
2. 回调对象被垃圾回收
3. 注册失败但未处理错误

**排查路径**：
1. 检查回调对象的生命周期，确保在需要时存活
2. 检查 `on()` 方法是否抛出异常
3. 使用日志确认注册成功

**证据**：`ohos/multimedia/camera/camera_manager.cj:381-399`
```cangjie
synchronized(cameraStatusMutex) {
    let callbackList = cameraStatusList
    if (findCallbackObject(callbackList, callback)) {
        CAMERA_LOG.error("CameraManager on failed: callback already registered")
        return
    }
    // ...
}
```

---

### Q5: FFI 调用返回错误码

**现象**：
```
successOrThrow throws BusinessException with error code from FFI
```

**可能原因**：
1. 底层服务未启动
2. 资源已释放
3. 参数在底层校验失败

**排查路径**：
1. 检查系统服务状态（`Camera Service`、`Media Library` 等）
2. 确认对象未被提前释放
3. 检查底层框架日志

**证据**：`ohos/multimedia/camera/camera_common.cj:62-66`
```cangjie
func successOrThrow(errCode: Int32): Unit {
    if (errCode != SUCCESS_CODE) {
        throw BusinessException(errCode, getErrorMsg(errCode))
    }
}
```

---

## 错误码速查

### Camera 错误码

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 7400101 | 参数错误 | 参数缺失、类型错误、已释放 |
| 7400102 | 操作不允许 | 会话状态不正确 |
| 7400103 | 会话未配置 | 未调用 beginConfig |
| 7400104 | 会话未运行 | 未调用 start |
| 7400107 | 相机冲突 | 其他应用占用相机 |
| 7400108 | 相机被禁用 | 安全策略限制 |
| 7400201 | 相机服务错误 | 服务异常、IPC 失败 |

### Image 错误码

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 62980002 | 参数错误 | 参数范围超出 |
| 62980004 | 内存不足 | 图片太大 |
| 62980006 | 解码失败 | 文件损坏、格式不支持 |
| 62980011 | 格式不支持 | 图片格式无法识别 |
| 62980015 | 旋转角度无效 | rotate 不在 0-360 |
| 62980104 | 内部对象失败 | 资源初始化失败 |

### MediaLibrary 错误码

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 201 | 权限被拒绝 | 未申请权限或用户拒绝 |
| 13900020 | 参数无效 | URI 格式错误 |
| 14000011 | 系统内部错误 | 服务异常 |

---

## 调试技巧

### 1. 开启日志

**日志域**：`0xD002B00`

**标签**：
- `CJ-Camera` - 相机模块
- `CJ-Image` - 图像模块
- `CJ-PhotoAccessHelper` - 相册模块

**日志级别**：
```cangjie
CAMERA_LOG.info("message")    // 信息
CAMERA_LOG.error("message")   // 错误
```

**证据**：`ohos/multimedia/camera/camera_common.cj:29-30`
```cangjie
const LOG_DOMAIN: UInt32 = 0xD002B00
let CAMERA_LOG = HilogChannel(0, LOG_DOMAIN, "CJ-Camera")
```

### 2. 检查 FFI 调用

在关键 FFI 调用前后添加日志：

```cangjie
// 调用前
CAMERA_LOG.info("Before FFI call: FfiCameraManagerCreateCameraInput")
let cameraInputId = FfiCameraManagerCreateCameraInputWithCameraDevice(getID(), CCameraDevice(camera), inout errCode)
CAMERA_LOG.info("After FFI call: errCode=${errCode}")
successOrThrow(errCode)
```

### 3. 验证权限

检查权限是否已申请：

```cangjie
// 在应用代码中检查
import ohos.security.accesstoken.AccessToken

let tokenId = AccessToken.getNativeTokenId()
let result = AccessToken.verifyAccessToken(tokenId, "ohos.permission.CAMERA")
```

### 4. 检查对象状态

确保对象未被提前释放：

```cangjie
// 检查 CameraManager ID 是否有效
if (cameraManager.getID() <= 0) {
    CAMERA_LOG.error("Invalid CameraManager ID")
}
```

### 5. 使用 Mock 调试

在 Windows/Mac 上使用 Mock 进行编译调试：

```gn
# 使用 Mock 源文件编译
if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.camera.cj"]
}
```

---

## 性能调优

### 1. Worker 线程使用

耗时操作自动在 Worker 线程执行：

```cangjie
@!APILevel[
    since: "22",
    workerthread: true  // 标记为 Worker 线程执行
]
public func getAssets(options: FetchOptions): PhotoAssetResult
```

**注意**：Worker 线程中不能更新 UI

### 2. 资源及时释放

使用 `try-with-resource` 或显式释放：

```cangjie
// 方式1：try-with-resource
try (cSurfaceId = LibC.mallocCString(surfaceId).asResource()) {
    // 使用 cSurfaceId
} // 自动释放

// 方式2：显式释放
pixelMap.release()
```

### 3. 批量操作

使用批量接口减少 IPC 调用：

```cangjie
// 获取所有对象
let allAssets = fetchResult.getAllObjects()

// 而不是逐个获取
while (let asset = fetchResult.getNextObject()) {
    // ...
}
```

---

## 调试工具

### 1. HiLog 日志工具

```bash
# 查看相机模块日志
hilog -g 0xD002B00

# 实时查看
hilog | grep "CJ-Camera"
```

### 2. 内存分析

检查 FFI 资源是否正确释放：

```cangjie
// 在关键位置打印对象 ID
CAMERA_LOG.info("CameraManager ID: ${getID()}")
```

### 3. 调用链跟踪

使用系统提供的调用链工具：

```bash
# TODO(需确认)：OpenHarmony 是否提供类似的 systrace 工具
```

---

## 关键结论

1. **常见错误**：权限问题、参数错误、资源释放问题
2. **调试重点**：日志输出、FFI 错误码、对象生命周期
3. **性能优化**：Worker 线程、批量操作、及时释放资源

---

## 相关跳转

- [N-API 接口](03_NAPI_Interfaces.md) - 错误码详情
- [架构说明](01_Architecture.md) - 数据流分析
- [内部 API](04_Inner_API.md) - 资源管理

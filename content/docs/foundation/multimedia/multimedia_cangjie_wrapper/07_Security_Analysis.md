# 安全风险分析

> 本文档基于代码证据对 multimedia_cangjie_wrapper 进行安全风险评审

---

## 目的与适用范围

**目的**：识别系统中的安全风险，提供修复建议

**适用范围**：安全审计人员、框架开发者

**分析范围**：
- 代码位置：`ohos/**/*.cj`（不含 test）
- 分析维度：输入校验、权限、内存安全、IPC、文件操作、日志

---

## 威胁模型

### 攻击面识别

```
┌─────────────────────────────────────────────────────────────┐
│                      外部攻击面                              │
├─────────────────────────────────────────────────────────────┤
│  1. N-API 接口输入（应用传入的参数）                          │
│  2. 文件路径/URI（相册访问）                                  │
│  3. 回调函数（用户提供的回调）                                │
│  4. 底层框架返回数据                                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    multimedia_cangjie_wrapper                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │  输入校验   │ │  权限检查   │ │  资源管理   │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      底层系统服务                            │
└─────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 描述 | 防护措施 |
|------|------|----------|
| 应用 ↔ Wrapper | 应用调用 Kit API | 参数校验、权限声明 |
| Wrapper ↔ 底层 | FFI 调用 C++ 框架 | 错误码检查、资源释放 |
| 系统服务边界 | IPC 调用 | 系统服务权限控制 |

---

## 安全检查结果

### 1. 输入校验

#### 1.1 枚举值解析校验

**位置**：`ohos/multimedia/camera/camera_common.cj:182-189`

```cangjie
static func parse(val: Int32) {
    match (val) {
        case 0 => Off
        case 1 => On
        case 2 => Auto
        case _ => throw BusinessException(7400101, "Parameter error.")
    }
}
```

**评估**：✅ **安全** - 对无效枚举值抛出异常

#### 1.2 图像解码参数校验

**位置**：`ohos/multimedia/image/cj_image_common.cj:430-438`

```cangjie
func parseDecodeOptions() {
    if (rotate < 0 || rotate > 360) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid rotate ${rotate}")
    }
    if (desiredPixelFormat.getValue() > 9) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid desiredPixelFormat")
    }
}
```

**评估**：✅ **安全** - 对旋转角度和像素格式进行范围校验

#### 1.3 URI 字符串处理

**位置**：`ohos/file/photo_access_helper/photo_accesshelper.cj:119-125`

```cangjie
public func getBurstAssets(burstKey: String, options: FetchOptions): PhotoAssetResult {
    try (cBurstKey = LibC.mallocCString(burstKey).asResource()) {
        // ...
    }
}
```

**评估**：✅ **安全** - 使用资源管理器（asResource）确保内存释放

### 2. 权限检查

#### 2.1 相机权限

**位置**：`ohos/multimedia/camera/camera_manager.cj:149-163`

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.CAMERA",
    syscap: "SystemCapability.Multimedia.Camera.Core",
    throwexception: true
]
public func createCameraInput(camera: CameraDevice): CameraInput
```

**评估**：✅ **安全** - 敏感操作声明了 CAMERA 权限

#### 2.2 相册读写权限

**位置**：`ohos/file/photo_access_helper/photo_accesshelper.cj:83-99`

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.READ_IMAGEVIDEO",
    syscap: "SystemCapability.FileManagement.PhotoAccessHelper.Core",
    throwexception: true,
    workerthread: true
]
public func getAssets(options: FetchOptions): PhotoAssetResult
```

**评估**：✅ **安全** - 读取相册声明了 READ_IMAGEVIDEO 权限

#### 2.3 相册写入权限

**位置**：`ohos/file/photo_access_helper/photo_accesshelper.cj:342-355`

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.WRITE_IMAGEVIDEO",
    syscap: "SystemCapability.FileManagement.PhotoAccessHelper.Core",
    throwexception: true,
    workerthread: true
]
public func applyChanges(mediaChangeRequest: MediaChangeRequest): Unit
```

**评估**：✅ **安全** - 修改相册声明了 WRITE_IMAGEVIDEO 权限

### 3. 内存安全

#### 3.1 FFI 资源管理

**位置**：`ohos/multimedia/camera/camera_manager.cj:50-52`

```cangjie
~init() {
    releaseFFIData(myDataId)
}
```

**评估**：✅ **安全** - 使用析构函数确保 FFI 资源释放

#### 3.2 C 字符串内存管理

**位置**：`ohos/multimedia/camera/camera_manager.cj:212-219`

```cangjie
public func createPreviewOutput(profile: Profile, surfaceId: String): PreviewOutput {
    try (cSurfaceId = LibC.mallocCString(surfaceId).asResource()) {
        var errCode: Int32 = 0
        id = FfiCameraManagerCreatePreviewOutput(CProfile(profile), cSurfaceId.value, inout errCode)
        successOrThrow(errCode)
    }
    return PreviewOutput(id)
}
```

**评估**：✅ **安全** - 使用 `asResource()` 确保 C 字符串内存释放

#### 3.3 数组资源释放

**位置**：`ohos/file/photo_access_helper/photo_accesshelper_ffi.cj:264-294`

```cangjie
struct CPhotoCreationConfigs {
    init(param: Array<PhotoCreationConfig>) {
        unsafe {
            var res: CPointer<CPhotoCreationConfig> = safeMalloc<CPhotoCreationConfig>(count: param.size)
            for (i in 0..param.size) {
                try {
                    let cPhotoCreationConfig = param[i].toCPhotoCreationConfig()
                    res.write(i, cPhotoCreationConfig)
                } catch (e: Exception) {
                    for (j in 0..i) {
                        res.read(j).free()
                    }
                    LibC.free(res)
                    throw BusinessException(13900011, "Out of memory.")
                }
            }
        }
    }
}
```

**评估**：✅ **安全** - 异常时清理已分配的内存

### 4. 并发安全

#### 4.1 回调列表互斥保护

**位置**：`ohos/multimedia/camera/camera_manager.cj:36-40`

```cangjie
public class CameraManager <: RemoteDataLite {
    let cameraStatusList = ArrayList<(CallbackObject, Int64)>()
    let cameraStatusMutex = Mutex()
    // ...
}
```

**评估**：✅ **安全** - 使用 Mutex 保护共享状态

#### 4.2 回调注册互斥

**位置**：`ohos/multimedia/camera/camera_manager.cj:386-399`

```cangjie
synchronized(cameraStatusMutex) {
    let callbackList = cameraStatusList
    if (findCallbackObject(callbackList, callback)) {
        CAMERA_LOG.error("CameraManager on failed: ${paramError("callback object", "different")}")
        return
    }
    // ...
    callbackList.add((callback, lambdaData.getID()))
}
```

**评估**：✅ **安全** - 使用 synchronized 块保护回调列表

### 5. 错误处理

#### 5.1 FFI 错误码检查

**位置**：`ohos/multimedia/camera/camera_common.cj:62-66`

```cangjie
func successOrThrow(errCode: Int32): Unit {
    if (errCode != SUCCESS_CODE) {
        throw BusinessException(errCode, getErrorMsg(errCode))
    }
}
```

**评估**：✅ **安全** - 统一检查 FFI 错误码并抛出异常

---

## 潜在风险点

### 风险 1：日志敏感信息泄露

**位置**：多处 `CAMERA_LOG`, `IMAGE_LOG`, `PHOTO_ACCESS_HELPER_LOG`

**分析**：
- 日志级别包括 error、info、debug
- 可能记录文件路径、URI 等敏感信息

**示例**：
```cangjie
// camera_manager.cj:389
CAMERA_LOG.error("CameraManager on failed: ${paramError("callback object", "different")}")
```

**影响**：低 - 可能泄露内部实现细节

**修复建议**：
1. 审查日志输出，确保不记录敏感信息
2. 生产环境关闭 debug 日志

### 风险 2：字符串转换失败处理

**位置**：`ohos/file/photo_access_helper/photo_accesshelper_ffi.cj:189-197`

```cangjie
func toMemberType(): MemberType {
    match (memberType) {
        case 0 => Int64Value(intValue)
        case 1 =>
            let str = stringValue.toString()
            StringValue(str)
        case 2 => BoolValue(boolValue)
        case _ => throw BusinessException(13900020, "Parameter error.")
    }
}
```

**分析**：
- `stringValue.toString()` 转换 C 字符串可能失败
- 未处理空指针情况

**影响**：中 - 可能导致崩溃

**修复建议**：
```cangjie
case 1 =>
    if (stringValue.isNull()) {
        throw BusinessException(13900020, "Invalid string value")
    }
    let str = stringValue.toString()
    StringValue(str)
```

### 风险 3：数组索引越界

**位置**：`ohos/multimedia/image/cj_image_common.cj:674-677`

```cangjie
init(component: CRetComponent) {
    // ...
    for (i in 0..component.bufSize) {
        this.byteBuffer[i] = unsafe { component.byteBuffer.read(i) }
    }
}
```

**分析**：
- `component.bufSize` 来自底层框架
- 未校验 `bufSize` 与 `byteBuffer` 实际大小是否一致

**影响**：中 - 可能导致越界访问

**修复建议**：
```cangjie
init(component: CRetComponent) {
    if (component.bufSize < 0 || component.bufSize > MAX_BUFFER_SIZE) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Invalid buffer size")
    }
    // ...
}
```

### 风险 4：时间戳解析

**位置**：`ohos/multimedia/camera/camera_ffi.cj:35`

```cangjie
func FfiImageImageImplGetTimestamp(id: Int64): Int64
```

**分析**：
- 时间戳值直接返回，未校验范围
- 异常值可能导致业务逻辑错误

**影响**：低 - 业务逻辑问题

**修复建议**：在业务层增加时间戳有效性校验

### 风险 5：Bundle 信息获取

**位置**：`ohos/file/photo_access_helper/photo_accesshelper.cj:261-275`

```cangjie
func getSelfBundleInfo(): FfiBundleInfo {
    let bundleFlags = BundleFlag.GET_BUNDLE_INFO_WITH_ABILITY |
        BundleFlag.GET_BUNDLE_INFO_WITH_HAP_MODULE |
        BundleFlag.GET_BUNDLE_INFO_WITH_SIGNATURE_INFO |
        BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    let bundleInfo = BundleManager.getBundleInfoForSelf(bundleFlags)
    // ...
}
```

**分析**：
- 获取了详细的 Bundle 信息，包括签名信息
- 信息传递给底层进行权限校验

**影响**：低 - 信息泄露风险可控

---

## 可利用点清单（基于代码证据）

| # | 可利用点 | 证据位置 | 触发条件 | 影响 | 修复建议 |
|---|----------|----------|----------|------|----------|
| 1 | 日志信息泄露 | 多处 LOG 调用 | 日志开启 | 信息泄露 | 审查日志内容，关闭 debug 日志 |
| 2 | 字符串转换失败 | `photo_accesshelper_ffi.cj:193` | 底层返回无效指针 | 崩溃 | 增加空指针检查 |
| 3 | 数组越界访问 | `cj_image_common.cj:676` | 底层返回异常 size | 越界访问 | 增加 size 校验 |
| 4 | 时间戳异常值 | `image.cj:110-113` | 底层返回异常值 | 业务错误 | 业务层校验 |
| 5 | 回调重复注册 | `camera_manager.cj:388-391` | 应用重复注册 | 资源泄漏 | 增加重复注册限制 |

---

## 安全最佳实践

### 已实施的安全措施

1. ✅ **参数校验** - 枚举值、数值范围校验
2. ✅ **权限声明** - 敏感操作声明权限
3. ✅ **资源管理** - RAII 模式，确保资源释放
4. ✅ **并发保护** - Mutex 保护共享状态
5. ✅ **错误处理** - 统一错误码处理

### 建议加强的安全措施

1. 🔧 **输入白名单** - 对字符串参数增加白名单校验
2. 🔧 **资源使用限制** - 限制最大资源使用（内存、句柄）
3. 🔧 **防重放保护** - 敏感操作增加防重放机制
4. 🔧 **日志脱敏** - 日志中敏感信息脱敏

---

## 关键结论

1. **整体安全**：代码整体安全设计良好，使用了参数校验、权限声明、资源管理等措施
2. **主要风险**：潜在风险主要集中在边界情况处理和日志安全
3. **修复建议**：加强输入校验的完整性，完善日志脱敏

---

## 相关跳转

- [架构说明](01_Architecture.md)
- [N-API 接口](03_NAPI_Interfaces.md)
- [内部 API](04_Inner_API.md)

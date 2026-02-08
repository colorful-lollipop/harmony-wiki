# 安全风险评估 (Security Review)

> OpenHarmony Camera Framework - 详细安全风险分析

---

## 评估说明

**评估范围**: camera_framework 核心代码（排除 test/ fuzztest/）  
**评估方法**: 静态代码分析 + 攻击面建模  
**风险等级定义**:
- 🔴 **高危 (High)**: 可导致权限提升、代码执行、数据泄露
- 🟡 **中危 (Medium)**: 可导致拒绝服务、功能绕过
- 🟢 **低危 (Low)**: 轻微安全问题、防御性编程缺陷

---

## R1: 内存安全问题 🔴

### R1.1 内存分配后未检查空指针

**风险等级**: 🔴 **高危** (可导致崩溃/未定义行为)

**位置 1**: `frameworks/cj/camera_picker/src/camera_picker_impl.cpp:80`

**证据**:
```cpp
// frameworks/cj/camera_picker/src/camera_picker_impl.cpp:80
char *res = static_cast<char *>(malloc(sizeof(char) * len));
// ❌ 未检查 res 是否为 nullptr 即使用
strcpy(res, src);  // 如果 malloc 失败，导致崩溃
```

**触发路径**:
```
Cangjie 应用调用相机选择器
  ↓
CameraPicker FFI 接口
  ↓
malloc(len) 在内存不足时返回 nullptr
  ↓
strcpy/res 解引用空指针
  ↓
[空指针解引用 → 进程崩溃]
```

**影响评估**:
- **可利用性**: 中 - 需要内存压力条件
- **影响**: 拒绝服务 (Camera Service 或应用崩溃)
- **权限提升**: 否，但可导致服务不稳定

**修复建议**:
```cpp
char *res = static_cast<char *>(malloc(sizeof(char) * len));
if (res == nullptr) {
    MEDIA_ERR_LOG("malloc failed for size %{public}zu", len);
    return nullptr;  // 或返回错误码
}
strcpy(res, src);
```

---

**位置 2**: `frameworks/cj/camera/src/video_output_impl.cpp:164`

**证据**:
```cpp
// frameworks/cj/camera/src/video_output_impl.cpp:164
(FrameRateRange *)malloc(sizeof(FrameRateRange) * supportedFrameRatesRange.size());
// ❌ 缺少 null 检查
```

**位置 3**: `frameworks/native/ndk/impl/camera_manager_impl.cpp:261-265`

**证据**:
```cpp
// frameworks/native/ndk/impl/camera_manager_impl.cpp:261-265
Camera_Device* outCameras = new Camera_Device[cameraSize];  // ❌ 无 null 检查
char* dst = new char[dstSize];  // ❌ 无 null 检查
memcpy_s(dst, dstSize, src.c_str(), src.length());  // 如果 dst 为 null，memcpy_s 可能崩溃
```

**修复建议**:
```cpp
Camera_Device* outCameras = new (std::nothrow) Camera_Device[cameraSize];
if (outCameras == nullptr) {
    return CAMERA_SERVICE_ERROR;
}
```

---

**位置 4**: `frameworks/native/ndk/impl/camera_manager_impl.cpp:813`

**证据**:
```cpp
// frameworks/native/ndk/impl/camera_manager_impl.cpp:813
char* src = new char[infoSize];
// ❌ 未检查 src 是否为 nullptr 即使用
memcpy_s(src, infoSize, dst.c_str(), dst.length());
```

**修复优先级**: 🔴 **立即修复** (多处高危缺陷)

---

### R1.2 缓冲区拷贝未充分验证

**风险等级**: 🟡 **中危**

**位置**: `services/camera_service/src/hstream_metadata.cpp:397-406`

**证据**:
```cpp
// services/camera_service/src/hstream_metadata.cpp:397-406
uint32_t itemSize = 0;
memcpy_s(&itemSize, sizeof(itemSize), entry.data.u8, sizeof(uint32_t));
// itemSize 来自外部输入，可能过大
void* buffer = malloc(itemSize);  // 可能分配过大内存
memcpy_s(buffer, itemSize, entry.data.u8 + sizeof(uint32_t), itemSize);
```

**风险**: itemSize 可能由攻击者控制，导致：
1. 超大内存分配 → 拒绝服务
2. 缓冲区溢出（如果 itemSize 与实际数据大小不符）

**修复建议**:
```cpp
// 添加最大值限制
const uint32_t MAX_METADATA_SIZE = 10 * 1024 * 1024;  // 10MB
if (itemSize > MAX_METADATA_SIZE || itemSize == 0) {
    MEDIA_ERR_LOG("Invalid metadata size: %{public}u", itemSize);
    return;
}
```

---

## R2: 输入验证缺陷 🔴

### R2.1 IPC 反序列化缺乏充分验证

**风险等级**: 🔴 **高危**

**位置**: `services/camera_service/binder/server/src/hstream_capture_photo_callback_stub.cpp:25-58`

**证据**:
```cpp
// services/camera_service/binder/server/src/hstream_capture_photo_callback_stub.cpp:25-58
int HStreamCapturePhotoCallbackStub::OnRemoteRequest(...) {
    MessageParcel data;
    data.ReadFromParcel(data);
    sptr<SurfaceBuffer> surfaceBuffer = SurfaceBuffer::Create();
    surfaceBuffer->ReadFromMessageParcel(data);  // 反序列化
    // ❌ 缺少对 surfaceBuffer 内容/大小的充分验证
}
```

**风险**: 攻击者可能构造恶意 IPC 数据包，导致：
1. 无效 Buffer 句柄 → use-after-free
2. 超大 buffer size → 内存耗尽
3. 格式错误数据 → 未定义行为

**触发路径**:
```
恶意应用获取 IPC 接口
  ↓
构造伪造的 MessageParcel
  ↓
包含无效 SurfaceBuffer 句柄
  ↓
回调处理时解引用
  ↓
[Use-After-Free 或崩溃]
```

**修复建议**:
```cpp
// 1. 添加 buffer 有效性检查
if (!surfaceBuffer || !surfaceBuffer->GetHandle()) {
    MEDIA_ERR_LOG("Invalid surface buffer received");
    return ERR_INVALID_DATA;
}

// 2. 限制 buffer 大小
size_t bufferSize = surfaceBuffer->GetSize();
if (bufferSize > MAX_PHOTO_SIZE) {
    MEDIA_ERR_LOG("Photo size too large: %{public}zu", bufferSize);
    return ERR_INVALID_DATA;
}
```

---

### R2.2 路径长度检查不足

**风险等级**: 🟡 **中危**

**位置**: `services/camera_service/src/hcamera_service.cpp:2173`

**证据**:
```cpp
// services/camera_service/src/hcamera_service.cpp:2173
CHECK_RETURN_RET(cameraId.length() > PATH_MAX, CAMERA_INVALID_ARG);
// ✅ 长度检查存在，但仅此一项
```

**分析**: 虽有长度检查，但缺少：
1. 字符集验证（是否包含非法字符）
2. 路径规范化验证
3. 路径穿越检测

**修复建议**:
```cpp
// 添加路径字符集验证
if (!IsValidCameraId(cameraId)) {
    // cameraId 只允许 [a-zA-Z0-9_-]
    return CAMERA_INVALID_ARG;
}
```

---

### R2.3 字符串处理潜在风险

**风险等级**: 🟢 **低危**

**位置**: 多处使用 `strcpy`

**证据**:
```cpp
// frameworks/cj/camera_picker/src/camera_picker_impl.cpp
strcpy(res, src);  // 如果 src 长度超过 res 分配大小，缓冲区溢出
```

**修复建议**: 使用 `strcpy_s` 或确保缓冲区大小足够

---

## R3: 并发安全 🟡

### R3.1 共享状态竞争条件

**风险等级**: 🟡 **中危**

**位置**: `services/camera_service/src/hcamera_session_manager.h`

**证据**:
```cpp
// services/camera_service/src/hcamera_session_manager.h
class HCameraSessionManager : public Singleton<HCameraSessionManager> {
    std::list<sptr<HCaptureSession>> sessions_;
    // 观察：缺少显式 mutex 保护
};
```

**分析**: 会话列表在多线程环境下访问（主线程 + 回调线程），可能存在竞争条件。

**修复建议**:
```cpp
class HCameraSessionManager {
    mutable std::mutex sessionsMutex_;
    std::list<sptr<HCaptureSession>> sessions_;
    
    void AddSession(sptr<HCaptureSession> session) {
        std::lock_guard<std::mutex> lock(sessionsMutex_);
        sessions_.push_back(session);
    }
};
```

---

### R3.2 TOCTOU (Time-Of-Check-Time-Of-Use)

**风险等级**: 🟡 **中危**

**位置**: `services/camera_service/src/camera_util.cpp:497-510`

**证据**:
```cpp
// services/camera_service/src/camera_util.cpp:497-510
std::string GetFileStream(const std::string &filepath) {
    char *canonicalPath = realpath(filepath.c_str(), nullptr);
    CHECK_RETURN_RET(canonicalPath == nullptr, "");
    // TOCTOU 窗口：realpath 检查通过后，文件可能被替换
    std::ifstream file(canonicalPath, std::ios::in | std::ios::binary);
    // ...
}
```

**风险**: 在 `realpath()` 和 `fopen()` 之间，文件可能被恶意替换。

**修复建议**: 使用 `open()` + `O_NOFOLLOW` 原子操作，或使用文件描述符传递。

---

## R4: 逻辑漏洞 🟡

### R4.1 错误处理不一致

**风险等级**: 🟡 **中危**

**位置**: 多处

**证据**:
```cpp
// 某些路径返回错误码
int32_t ret = SomeOperation();
if (ret != CAMERA_OK) {
    return ret;
}

// 其他路径仅打印日志
int32_t ret = OtherOperation();
if (ret != CAMERA_OK) {
    MEDIA_ERR_LOG("Operation failed");
    // ❌ 继续执行，未返回错误
}
```

**影响**: 可能导致部分失败的操作被误判为成功，进入不一致状态。

---

### R4.2 资源泄露风险

**风险等级**: 🟡 **中危**

**位置**: 错误处理路径

**证据**:
```cpp
// 某些错误处理路径未正确释放资源
int32_t HCameraDevice::Open() {
    sptr<SomeResource> resource = new SomeResource();
    // ... 某些错误返回 ...
    if (error) {
        return CAMERA_ERROR;  // ❌ resource 可能泄露
    }
    // ...
}
```

**修复建议**: 使用 RAII（智能指针）管理资源生命周期。

---

### R4.3 整数溢出风险

**风险等级**: 🟢 **低危**

**位置**: 大小计算

**证据**:
```cpp
// frameworks/cj/camera/src/video_output_impl.cpp:164
(FrameRateRange *)malloc(sizeof(FrameRateRange) * supportedFrameRatesRange.size());
// 如果 size() 极大，乘法可能溢出
```

**修复建议**:
```cpp
size_t allocSize = sizeof(FrameRateRange) * supportedFrameRatesRange.size();
if (allocSize / sizeof(FrameRateRange) != supportedFrameRatesRange.size()) {
    // 溢出检测
    return nullptr;
}
```

---

## R5: 权限与鉴权 🟢

### R5.1 权限检查总体良好

**正面观察**:

```cpp
// services/camera_service/src/camera_util.cpp:390-408
int32_t CheckPermission(uint32_t tokenId, const std::string& permissionName) {
    // Token 类型检查
    ATokenTypeEnum tokenType = AccessTokenKit::GetTokenType(tokenId);
    if (tokenType == TOKEN_NATIVE) {
        return CAMERA_OK;  // Native 进程免检
    }
    
    // HAP 应用权限检查
    int32_t ret = AccessTokenKit::VerifyAccessToken(tokenId, permissionName);
    if (ret != PERMISSION_GRANTED) {
        return CAMERA_NO_PERMISSION;
    }
    return CAMERA_OK;
}
```

**结论**: 权限检查机制正确实现，遵循最小权限原则。

---

### R5.2 系统应用白名单检查

**正面观察**:

```cpp
// frameworks/js/camera_napi/src/camera_napi_security_utils.cpp
bool CheckSystemApp() {
    uint64_t accessTokenID = OHOS::IPCSkeleton::GetCallingFullTokenID();
    return TokenIdKit::IsSystemAppByFullTokenID(accessTokenID);
}
```

**结论**: 系统 API 正确检查系统应用身份。

---

## 修复优先级建议

| 优先级 | 风险项 | 工作量 | 影响 |
|--------|--------|--------|------|
| **P0 (立即)** | R1.1 内存分配 null 检查 | 小 | 稳定性 |
| **P1 (高)** | R2.1 IPC 反序列化验证 | 中 | 安全 |
| **P1 (高)** | R1.2 缓冲区大小限制 | 小 | 安全 |
| **P2 (中)** | R3.1 并发安全 | 中 | 稳定性 |
| **P2 (中)** | R4.1 错误处理一致性 | 中 | 可靠性 |
| **P3 (低)** | R4.3 整数溢出 | 小 | 防御性 |

---

## 安全测试建议

### 推荐测试方法

1. **模糊测试 (Fuzzing)**
   ```bash
   # 已存在 fuzzer 列表
   test/fuzztest/cameramanager_fuzzer/
   test/fuzztest/capturesession_fuzzer/
   test/fuzzest/hstreamcapture_fuzzer/
   ```

2. **内存安全检测**
   ```bash
   # 使用 ASan 编译
   ./build.sh --gn-args "use_asan=true"
   ```

3. **IPC 接口测试**
   - 构造畸形 IPC 数据包
   - 测试边界条件

4. **权限绕过测试**
   - 验证非系统应用无法调用系统 API
   - 验证无 CAMERA 权限无法打开相机

---

## 总结

| 风险类别 | 数量 | 最高等级 | 状态 |
|----------|------|----------|------|
| 内存安全 | 4处 | 🔴 高危 | 需立即修复 |
| 输入验证 | 3处 | 🔴 高危 | 需修复 |
| 并发安全 | 2处 | 🟡 中危 | 建议修复 |
| 逻辑漏洞 | 3处 | 🟡 中危 | 建议修复 |
| 权限鉴权 | - | 🟢 良好 | 保持 |

**总体评估**: 代码整体安全设计合理，但存在多处内存安全问题需要立即关注。IPC 接口和输入处理需要加强验证。

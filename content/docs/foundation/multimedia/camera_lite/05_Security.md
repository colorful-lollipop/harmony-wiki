# 05 - 安全风险评审

> Camera_Lite 组件安全风险分析与建议

---

## 评审范围

本次安全评审覆盖以下代码：
- **interfaces/kits/** - 对外API头文件
- **frameworks/** - 框架实现（含binder和passthrough）
- **services/** - 服务实现（服务端代码）
- **BUILD.gn** - 构建配置

**排除范围**: test/ 目录下的测试代码

---

## 攻击面清单

### 1. 权限攻击面

| 入口点 | 权限检查 | 风险等级 |
|--------|----------|----------|
| CameraKit::GetInstance() | ✅ 检查 `ohos.permission.CAMERA` | 低 |

**分析**: 
- 唯一的权限检查点在 `frameworks/camera_kit.cpp:32`
- 使用 `permission_lite` 组件的 `CheckSelfPermission()`
- 若权限不足直接返回 `nullptr`，阻止后续操作

**证据**:
```cpp
CameraKit *CameraKit::GetInstance()
{
    if (CheckSelfPermission("ohos.permission.CAMERA") != GRANTED) {
        MEDIA_WARNING_LOG("Process can not access camera.");
        return nullptr;
    }
    static CameraKit kit;
    return &kit;
}
```

### 2. IPC攻击面

| 攻击类型 | 风险等级 | 说明 |
|----------|----------|------|
| IPC数据伪造 | 中 | Binder模式存在跨进程通信 |
| 回调对象劫持 | 中 | 服务端保存客户端回调对象 |
| IPC拒绝服务 | 低 | 未处理大量并发请求 |

**相关代码**:
- `services/server/src/camera_server.cpp` - IPC请求处理
- `frameworks/binder/src/camera_service_client.cpp` - IPC客户端

### 3. 输入验证攻击面

| 输入点 | 验证情况 | 风险等级 |
|--------|----------|----------|
| cameraId | ⚠️ 部分检查 | 中 |
| Surface数量 | ✅ 检查 (max 2) | 低 |
| FrameConfig参数 | ⚠️ 部分检查 | 中 |

### 4. 内存操作攻击面

| 操作类型 | 使用情况 | 风险等级 |
|----------|----------|----------|
| memcpy | ✅ 使用 memcpy_s | 低 |
| memset | ✅ 使用 memset_s | 低 |
| new/delete | ⚠️ 部分检查不足 | 中 |
| 数组访问 | ⚠️ 部分边界检查 | 中 |

---

## 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│                    不信任区域 (应用进程)                        │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  CameraKit (应用调用入口)                              │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          │ IPC / Passthrough                 │
│                          ▼                                   │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│                    信任区域 (服务进程)                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  CameraServer ←→ CameraService ←→ CameraDevice       │   │
│  │         ↓                                            │   │
│  │    HAL层 (HalCamera, Codec, Display)                │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

**关键信任边界**:
1. **IPC边界** - Binder/Passthrough调用
2. **HAL边界** - 与硬件驱动的接口

---

## 风险点详细分析

### R1: IPC反序列化缺乏严格验证 [中危]

**位置**: `services/server/src/camera_server.cpp`

**问题描述**:
IPC通信中从客户端接收的数据缺乏严格的边界检查，可能导致：
- 缓冲区溢出
- 类型混淆
- 拒绝服务

**代码证据** (camera_server.cpp:88-94):
```cpp
void CameraServer::GetCameraAbility(IpcIo *req, IpcIo *reply)
{
    size_t sz;
    string cameraId((const char*)(ReadString(req, &sz)));  // ⚠️ 未限制sz大小
    // ...
}
```

**影响**:
- 恶意应用可能通过构造超大字符串导致服务端内存耗尽

**修复建议**:
```cpp
void CameraServer::GetCameraAbility(IpcIo *req, IpcIo *reply)
{
    size_t sz;
    const char* str = ReadString(req, &sz);
    if (sz == 0 || sz > MAX_CAMERA_ID_LEN) {  // 添加长度限制
        MEDIA_ERR_LOG("Invalid cameraId length: %zu", sz);
        return;
    }
    string cameraId(str, sz);
    // ...
}
```

---

### R2: 动态内存分配未充分检查 [中危]

**位置**: 多处使用 `new` 未检查返回值

**问题描述**:
部分代码使用 `new` 而非 `new (nothrow)`，且未检查返回值，可能导致空指针解引用。

**代码证据** (camera_server.cpp:203):
```cpp
FrameConfig *DeserializeFrameConfig(IpcIo &io)
{
    int32_t type;
    ReadInt32(&io, &type);
    auto fc = new FrameConfig(type);  // ⚠️ 可能返回nullptr
    // 后续代码直接使用 fc，未检查
}
```

**影响**:
- 内存不足时程序崩溃

**修复建议**:
```cpp
auto fc = new (nothrow) FrameConfig(type);
if (fc == nullptr) {
    MEDIA_ERR_LOG("Failed to allocate FrameConfig");
    return nullptr;
}
```

---

### R3: Surface反序列化可能存在Use-After-Free [中危]

**位置**: `services/server/src/camera_server.cpp:207-214`

**问题描述**:
反序列化Surface时，若 `GenericSurfaceByIpcIo` 失败返回nullptr，后续代码可能误用。

**代码证据**:
```cpp
for (uint32_t i = 0; i < surfaceNum; i++) {
    Surface *surface = SurfaceImpl::GenericSurfaceByIpcIo(io);
    if (surface == nullptr) {
        MEDIA_ERR_LOG("Camera server receive null surface.");
        delete fc;  // ✅ 正确释放
        return nullptr;
    }
    fc->AddSurface(*surface);  // ⚠️ 这里没问题，但其他地方可能遗漏
}
```

**影响**:
- 空指针解引用导致崩溃

**当前状态**: ✅ 该位置已正确处理，但需检查其他位置

---

### R4: 整数溢出风险 [低危]

**位置**: `frameworks/binder/src/camera_service_client.cpp:82-88`

**问题描述**:
从IPC读取列表大小时，未检查是否超过合理范围。

**代码证据**:
```cpp
uint32_t listSize;
ReadUint32(reply, &listSize);  // ⚠️ listSize可能极大
list<CameraPicSize> supportSizeList;
for (uint32_t i = 0; i < listSize; i++) {
    CameraPicSize *cameraPicSize = static_cast<CameraPicSize*>(
        ReadRawData(reply, sizeof(CameraPicSize)));
    // ...
}
```

**影响**:
- listSize过大导致循环耗尽内存或CPU

**修复建议**:
```cpp
const uint32_t MAX_LIST_SIZE = 100;
if (listSize > MAX_LIST_SIZE) {
    MEDIA_ERR_LOG("Invalid list size: %u", listSize);
    return;
}
```

---

### R5: 类型转换缺乏验证 [中危]

**位置**: `frameworks/binder/src/camera_service_client.cpp:74-87`

**问题描述**:
使用 `static_cast` 转换IPC数据，缺乏类型验证。

**代码证据**:
```cpp
CallBackPara* para = (CallBackPara*)owner;  // ⚠️ C风格转换
CameraServiceClient *client = static_cast<CameraServiceClient*>(para->data);  // ⚠️ 信任data指针
```

**影响**:
- 恶意构造的IPC数据可能导致类型混淆

**修复建议**:
- 使用 `reinterpret_cast` 并添加校验
- 对 `para->data` 添加魔法数或校验和验证

---

### R6: 竞态条件风险 [低危]

**位置**: `services/impl/src/camera_device.cpp` (CallbackAssistant)

**问题描述**:
`CallbackAssistant` 的 `StreamCopyProcess` 线程与主线程共享 `state_` 变量，缺乏同步。

**代码证据** (camera_device.cpp:768):
```cpp
while (assistant->state_ == LOOP_LOOPING) {  // ⚠️ 无锁访问
    // ...
}
```

**影响**:
- 状态判断可能不一致

**修复建议**:
```cpp
// 使用原子变量
std::atomic<LoopState> state_;
```

---

### R7: 私有标签越界访问 [中危]

**位置**: `frameworks/binder/src/camera_device_client.cpp:144-147`

**问题描述**:
`GetVendorParameter` 读取私有标签数据，但未验证传入的 `len` 参数。

**代码证据**:
```cpp
uint8_t data[PRIVATE_TAG_LEN];
fc.GetVendorParameter(data, sizeof(data));  // 假设PRIVATE_TAG_LEN=32
WriteUint32(&io, (uint32_t)sizeof(data));
WriteBuffer(&io, (void *)data, sizeof(data));
```

**注意**: 当前实现使用了固定大小的栈上数组，相对安全。但如果改为动态分配，需格外注意。

---

### R8: std::stoi异常未捕获 [低危]

**位置**: `services/impl/src/camera_device.cpp:538-541`

**问题描述**:
使用 `std::stoi` 转换字符串，可能抛出异常。

**代码证据**:
```cpp
static void GetSurfaceRect(Surface *surface, IRect *attr)
{
    attr->x = std::stoi(surface->GetUserData(string("region_position_x")));  // ⚠️ 可能抛异常
    // ...
}
```

**影响**:
- 无效输入导致未捕获异常，进程崩溃

**修复建议**:
```cpp
try {
    attr->x = std::stoi(surface->GetUserData(string("region_position_x")));
} catch (const std::exception& e) {
    MEDIA_ERR_LOG("Invalid position data: %s", e.what());
    attr->x = 0;
}
```

---

## 内存安全分析

### 安全函数使用

| 函数 | 使用情况 | 评价 |
|------|----------|------|
| memcpy | ✅ 使用 memcpy_s | 安全 |
| memset | ✅ 使用 memset_s | 安全 |
| strcpy | ✅ 未使用 | 安全 |
| strcat | ✅ 未使用 | 安全 |
| sprintf | ✅ 未使用 | 安全 |

**证据**: 依赖 `bounds_checking_function` 库

### 动态内存管理

| 模式 | 使用情况 | 评价 |
|------|----------|------|
| new (nothrow) | 部分使用 | 较好 |
| new | 部分使用，未检查 | 有风险 |
| delete | 基本成对出现 | 较好 |
| 智能指针 | ❌ 未使用 | 建议引入 |

---

## 修复建议优先级

| 优先级 | 风险点 | 建议操作 |
|--------|--------|----------|
| 🔴 高 | R1 - IPC反序列化 | 添加长度限制、类型验证 |
| 🔴 高 | R2 - 内存分配检查 | 统一使用 `new (nothrow)` |
| 🟡 中 | R5 - 类型转换 | 添加校验机制 |
| 🟡 中 | R7 - 私有标签 | 验证长度参数 |
| 🟢 低 | R4 - 整数溢出 | 添加范围检查 |
| 🟢 低 | R6 - 竞态条件 | 使用原子变量 |
| 🟢 低 | R8 - stoi异常 | 添加try-catch |

---

## 安全最佳实践建议

### 1. 输入验证

所有外部输入（IPC、HAL回调、应用参数）都应验证：
- 长度限制
- 范围检查
- 类型匹配

### 2. 内存安全

- 统一使用 `new (nothrow)` 并检查返回值
- 考虑引入智能指针（`std::unique_ptr`, `std::shared_ptr`）
- 使用RAII模式管理资源

### 3. 并发安全

- 共享状态使用原子变量或互斥锁保护
- 避免数据竞争

### 4. IPC安全

- 添加魔法数或校验和验证IPC数据完整性
- 限制IPC数据大小
- 对敏感操作添加额外权限检查

### 5. 日志脱敏

- 避免在日志中打印敏感信息（如原始buffer内容）
- 限制cameraId等标识符长度

---

## 检查局限性

本次评审存在以下局限性：

1. **HAL层未评审** - HalCamera, CodecInterface, DisplayLayer实现未包含在代码库中
2. **IPC协议未完整分析** - 依赖OpenHarmony IPC框架的安全性
3. **fuzz测试缺失** - 未进行自动化fuzz测试
4. **运行时行为** - 基于静态代码分析，未覆盖运行时异常场景

---

## 结论

Camera_Lite 组件整体安全风险可控，主要安全措施：
- ✅ 入口处进行权限检查
- ✅ 使用安全函数（memcpy_s, memset_s）
- ✅ 部分内存分配使用 `new (nothrow)`

但仍需关注：
- ⚠️ IPC反序列化缺乏严格验证
- ⚠️ 部分动态内存分配未充分检查
- ⚠️ 类型转换缺乏验证

**建议**: 按优先级修复风险点，并增加fuzz测试覆盖。

---

## 参考

- [内部接口](03_Inner_API.md) - 代码位置详情
- [OpenHarmony安全开发指南](https://gitee.com/openharmony/docs)

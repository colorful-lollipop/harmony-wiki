# 安全风险评审

## 目的与适用范围

本文档基于代码证据分析分布式屏幕的安全风险，包括攻击面、信任边界和可利用点。

---

## 威胁模型

### 系统边界图

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              分布式屏幕威胁模型                                      │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   外部攻击者                                                                        │
│        │                                                                            │
│        │ 攻击向量                                                                   │
│        ▼                                                                            │
│   ┌─────────────────────────────────────────────────────────────────────────────┐  │
│   │                              信任边界                                        │  │
│   │  ┌─────────────────────────────────────────────────────────────────────────┐ │  │
│   │  │                         IPC接口层 (N/A - 纯Native)                       │ │  │
│   │  │  - 接口描述符验证                                                        │ │  │
│   │  └─────────────────────────────────────────────────────────────────────────┘ │  │
│   │                                    │                                          │  │
│   │                                    ▼                                          │  │
│   │  ┌─────────────────────────────────────────────────────────────────────────┐ │  │
│   │  │                         System Ability服务层                              │ │  │
│   │  │  - 权限检查: ohos.permission.ENABLE_DISTRIBUTED_HARDWARE                 │ │  │
│   │  │  - 参数校验: devId, dhId, reqId                                          │ │  │
│   │  │  - IPC调用者Token验证                                                     │ │  │
│   │  └─────────────────────────────────────────────────────────────────────────┘ │  │
│   │                                    │                                          │  │
│   │                                    ▼                                          │  │
│   │  ┌─────────────────────────────────────────────────────────────────────────┐ │  │
│   │  │                         网络传输层 (SoftBus)                              │ │  │
│   │  │  - 设备认证: 同账号验证 (need_same_account)                               │ │  │
│   │  │  - 访问控制: CheckSrcPermission/CheckSinkPermission                      │ │  │
│   │  │  - 会话加密: 软总线提供                                                   │ │  │
│   │  └─────────────────────────────────────────────────────────────────────────┘ │  │
│   │                                    │                                          │  │
│   │                                    ▼                                          │  │
│   │  ┌─────────────────────────────────────────────────────────────────────────┐ │  │
│   │  │                         文件系统层                                         │ │  │
│   │  │  - Dump路径: /data/data/dscreen/ (root only)                             │ │  │
│   │  │  - 配置访问: SA配置文件                                                   │ │  │
│   │  └─────────────────────────────────────────────────────────────────────────┘ │  │
│   │                                                                               │  │
│   └─────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 攻击面清单

### 1. IPC接口攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| IPC接口描述符伪造 | 低 | 使用标准Binder机制，需系统权限才能伪造 |
| IPC参数注入 | 中 | 外部输入参数（devId, dhId等）需严格校验 |
| IPC权限绕过 | 中 | 依赖`VerifyAccessToken`，需检查实现完整性 |

### 2. 网络传输攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 中间人攻击 | 低 | 软总线提供会话加密 |
| 设备伪装 | 中 | 依赖同账号验证和设备认证 |
| 数据篡改 | 低 | 软总线层处理完整性校验 |
| 拒绝服务 | 中 | 大数据包可能导致资源耗尽 |

### 3. 文件系统攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 路径遍历 | 中 | Dump文件路径使用固定前缀，需防止遍历 |
| 敏感信息泄露 | 低 | Dump数据可能包含屏幕内容 |
| 存储耗尽 | 中 | Dump文件大小限制295MB |

### 4. 内存攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 缓冲区溢出 | 低 | 启用边界检查和栈保护 |
| 整数溢出 | 低 | 启用整数溢出检查 |
| Use-after-free | 低 | 智能指针管理，CFI保护 |

---

## 可被利用点分析

### 可利用点 #1: IPC参数长度检查不足

**风险等级**: 中

**证据位置**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp`

```cpp
// 当前实现读取字符串但未严格限制长度
std::string devId = data.ReadString();
std::string dhId = data.ReadString();
std::string reqId = data.ReadString();
```

**触发路径**:
1. 攻击者获取`ohos.permission.ENABLE_DISTRIBUTED_HARDWARE`权限
2. 构造超长devId/dhId/reqId调用`RegisterDistributedHardware`
3. 可能导致内存分配过大或后续处理溢出

**影响**: 可能导致服务崩溃或内存耗尽

**修复建议**:
```cpp
// 增加长度校验
constexpr size_t MAX_ID_LEN = 256;
std::string devId = data.ReadString();
if (devId.length() > MAX_ID_LEN) {
    return ERR_DH_SCREEN_INPUT_PARAM_INVALID;
}
```

---

### 可利用点 #2: JSON参数解析缺乏严格校验

**风险等级**: 中

**证据位置**: `services/screenservice/sourceservice/dscreenmgr/2.0/src/dscreen_manager.cpp`

```cpp
// 解析屏幕信息JSON
int32_t DScreenManager::EnableDistributedScreen(...) {
    // param参数被解析为JSON，但缺乏schema验证
    // 可能导致解析异常或后续逻辑错误
}
```

**触发路径**:
1. 通过IPC传入恶意构造的JSON参数
2. JSON解析器可能解析异常数据
3. 后续逻辑使用未经验证的字段值

**影响**: 可能导致逻辑错误或崩溃

**修复建议**:
```cpp
// 使用JSON Schema验证或严格字段检查
if (!IsValidScreenInfoJson(screenInfo)) {
    return ERR_DH_SCREEN_SA_ENABLE_JSON_ERROR;
}
```

---

### 可利用点 #3: 软总线会话ID越界访问

**风险等级**: 中

**证据位置**: `services/softbusadapter/src/softbus_adapter.cpp`

```cpp
// 会话ID管理可能存在并发问题
int32_t SoftbusAdapter::SendData(int32_t sessionId, ...) {
    // 未验证sessionId是否在有效范围内
    auto it = sessionMap_.find(sessionId);
    if (it == sessionMap_.end()) {
        return ERR_DH_SCREEN_ADAPTER_SESSION_ID_NOT_FIND;
    }
    // ...
}
```

**触发路径**:
1. 多线程环境下会话ID被并发修改
2. 使用无效的sessionId调用SendData
3. 可能访问已释放的资源

**影响**: 可能导致Use-after-free或崩溃

**修复建议**:
```cpp
// 增加sessionId范围检查
if (sessionId < 0 || sessionId >= MAX_SESSION_NUM) {
    return ERR_DH_SCREEN_ADAPTER_BAD_VALUE;
}
// 使用锁保护会话映射访问
std::lock_guard<std::mutex> lock(sessionMutex_);
```

---

### 可利用点 #4: Dump文件路径遍历风险

**风险等级**: 中

**证据位置**: `common/include/dscreen_constants.h:94`

```cpp
const std::string DUMP_FILE_PATH = "/data/data/dscreen";
```

**触发路径**:
1. root版本中Dump功能启用
2. 如果dump文件名由外部输入拼接
3. 可能导致路径遍历写入任意文件

**影响**: 可能写入系统关键文件

**修复建议**:
```cpp
// 确保dump文件名不包含路径分隔符
if (fileName.find('/') != std::string::npos || 
    fileName.find("..") != std::string::npos) {
    return ERR_DH_SCREEN_BAD_VALUE;
}
```

**当前状态**: 需要检查dump文件创建逻辑（代码中未直接找到）

---

### 可利用点 #5: 数据传输无大小限制

**风险等级**: 中

**证据位置**: `common/include/dscreen_constants.h:129`

```cpp
constexpr uint32_t DSCREEN_MAX_RECV_DATA_LEN = 104857600;  // 100MB
```

虽然设置了最大接收长度，但在实际传输中：

```cpp
// ScreenSourceTrans可能发送大数据
int32_t ScreenSourceTrans::FeedChannelData(const std::shared_ptr<DataBuffer> &data) {
    // data的大小可能接近100MB
    // 频繁的大数据传输可能导致内存压力
}
```

**触发路径**:
1. 高分辨率屏幕持续传输大数据
2. 接收端缓冲区堆积
3. 可能导致OOM

**影响**: 可能导致内存耗尽

**修复建议**:
```cpp
// 增加流控机制
if (pendingDataSize_ > MAX_PENDING_DATA_THRESHOLD) {
    return ERR_DH_SCREEN_TRANS_TIMEOUT;  // 反压
}
```

---

### 可利用点 #6: ConfigDistributedHardware 接口权限检查缺失 ⭐高危⭐

**风险等级**: **高**

**证据位置**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp:160-176`

```cpp
int32_t DScreenSourceStub::ConfigDistributedHardwareInner(MessageParcel &data, MessageParcel &reply,
    MessageOption &option)
{
    (void)option;
    // ❌ 缺少 HasEnableDHPermission() 权限检查！
    std::string devId = data.ReadString();
    std::string dhId = data.ReadString();
    std::string key = data.ReadString();
    std::string value = data.ReadString();
    // ...
}
```

**对比其他接口** (均有权限检查):
- `InitSourceInner` (行74): `if (!HasEnableDHPermission())` ✅
- `ReleaseSourceInner` (行104): `if (!HasEnableDHPermission())` ✅
- `RegisterDistributedHardwareInner` (行117): `if (!HasEnableDHPermission())` ✅
- `UnregisterDistributedHardwareInner` (行143): `if (!HasEnableDHPermission())` ✅

**触发路径**:
1. 攻击者通过任意应用调用 IPC 接口
2. 调用 `ConfigDistributedHardware` 无需 `ohos.permission.ENABLE_DISTRIBUTED_HARDWARE` 权限
3. 可直接修改分布式屏幕配置参数

**影响**: 可能导致配置篡改、服务异常或信息泄露

**修复建议**:
```cpp
int32_t DScreenSourceStub::ConfigDistributedHardwareInner(MessageParcel &data, MessageParcel &reply,
    MessageOption &option)
{
    (void)option;
    // ✅ 添加权限检查
    if (!HasEnableDHPermission()) {
        DHLOGE("The caller has no ENABLE_DISTRIBUTED_HARDWARE permission.");
        return ERR_DH_SCREEN_SA_CHECK_ENABLE_PERMISSION_FAIL;
    }
    // ... 原有逻辑
}
```

---

### 可利用点 #7: Sink端接口权限检查缺失

**风险等级**: 中

**证据位置**: `services/screenservice/sinkservice/dscreenservice/src/dscreen_sink_stub.cpp`

**无需权限检查的接口**:
- `SubscribeDistributedHardwareInner` (行97-110): 无权限检查 ⚠️
- `UnsubscribeDistributedHardwareInner` (行112-124): 无权限检查 ⚠️
- `DScreenNotifyInner` (行126-141): 无权限检查 ⚠️

**受保护的接口** (有权限检查):
- `InitSinkInner` (行69): `if (!HasEnableDHPermission())` ✅
- `ReleaseSinkInner` (行88): `if (!HasEnableDHPermission())` ✅

**分析**:
Sink端订阅/取消订阅接口缺乏权限校验，可能被恶意调用。

---

### 可利用点 #8: 内存分配未使用 nothrow

**风险等级**: **高**

**证据位置**: `services/screentransport/screensinkprocessor/decoder/src/image_sink_decoder.cpp:88`

```cpp
// 高危：未使用 std::nothrow，分配失败会抛出异常
lastFrame_ = new uint8_t[lastFrameSize_];
```

**其他风险位置**:
- `services/common/imageJpeg/src/jpeg_image_processor.cpp:123` - `new unsigned char[partialSize]`
- `services/common/imageJpeg/src/jpeg_image_processor.cpp:160` - `new uint8_t[item.dirtySize]`
- `services/common/imageJpeg/src/jpeg_image_processor.cpp:174` - `new uint8_t[dirtyImageDataSize]`
- `services/screentransport/screendatachannel/src/screen_data_channel_impl.cpp:224` - `new char[rectInfo.length() + 1]`

**影响**: 内存分配失败时抛出异常，可能导致服务崩溃

**修复建议**:
```cpp
// 使用 nothrow 并检查返回值
lastFrame_ = new (std::nothrow) uint8_t[lastFrameSize_];
if (lastFrame_ == nullptr) {
    DHLOGE("Failed to allocate lastFrame buffer");
    return ERR_DH_SCREEN_SA_ALLOC_MEMORY_FAIL;
}
```

---

## 安全机制评估

### 已启用的安全机制

| 机制 | 状态 | 说明 |
|------|------|------|
| CFI (Control Flow Integrity) | ✅ | 所有target启用 |
| 边界检查 (Boundary Sanitize) | ✅ | 所有target启用 |
| 整数溢出检查 | ✅ | 所有target启用 |
| UBSan | ✅ | 所有target启用 |
| PAC-RET | ✅ | ARM指针认证 |
| 栈保护 | ✅ | `-fstack-protector-strong` |
| IPC Token验证 | ✅ | Stub层验证 |
| 权限检查 | ✅ | AccessTokenKit验证 |
| 同账号验证 | ✅ | SoftBusPermissionCheck |
| 会话加密 | ✅ | 软总线提供 |

### 安全机制代码证据

**证据**: `common/BUILD.gn:19-27`

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true
}
branch_protector_ret = "pac_ret"
cflags = [ "-fstack-protector-strong" ]
```

---

## 修复建议汇总

| 优先级 | 问题 | 证据位置 | 建议修复 |
|--------|------|----------|----------|
| **高** | ConfigDistributedHardware 权限缺失 | `dscreen_source_stub.cpp:160` | 添加 `HasEnableDHPermission()` 检查 |
| **高** | 内存分配未使用 nothrow | `image_sink_decoder.cpp:88` | 改为 `new (std::nothrow)` 并检查返回值 |
| **高** | JPEG处理器内存分配风险 | `jpeg_image_processor.cpp:123,160,174` | 统一使用 nothrow 并检查 |
| 中 | Sink端订阅接口权限缺失 | `dscreen_sink_stub.cpp:97,112,126` | 评估是否需要权限保护 |
| 中 | IPC参数长度检查 | `dscreen_source_stub.cpp:78-125` | 统一使用 `CheckRegParams` 验证 |
| 中 | JSON参数无Schema验证 | `dscreen_manager.cpp` | 增加JSON字段校验 |
| 中 | 会话ID并发访问 | `softbus_adapter.cpp` | 增加锁保护和范围检查 |
| 中 | Dump文件路径遍历 | `dscreen_constants.h:94` | 校验文件名合法性 |
| 中 | 数据传输无流控 | `screen_source_trans.cpp` | 增加反压机制 |
| 低 | 权限检查完整性 | 全部Stub文件 | 定期审计所有接口 |

---

## 检查范围与局限性

### 已检查范围

- ✅ IPC接口权限检查
- ✅ 软总线权限验证
- ✅ 安全编译选项
- ✅ 关键参数校验
- ✅ 错误码定义

### 未深入检查范围

- ⏳ 完整的数据流fuzz测试
- ⏳ 第三方库依赖安全
- ⏳ 软总线底层安全
- ⏳ 编解码器安全

### 局限性说明

1. **静态分析限制**: 部分动态行为（如竞争条件）难以通过静态分析发现
2. **测试覆盖**: 未执行fuzz测试和渗透测试
3. **依赖链**: 未深入分析所有外部依赖的安全状况

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目基本信息
- [架构设计](01_Architecture.md) - 架构说明
- [对外接口](03_Interfaces.md) - 接口定义
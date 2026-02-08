# 05_Security - 安全风险评审

> 攻击面、信任边界、可利用点与修复建议

---

## 1. 安全架构概述

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  非信任域: 第三方应用                                         │
│  - 无USE_USER_IDM权限                                        │
│  - 无法访问API                                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ 权限检查
┌─────────────────────────────────────────────────────────────┐
│  半信任域: 系统应用                                           │
│  - 有USE_USER_IDM权限                                        │
│  - 需通过IPC调用                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC
┌─────────────────────────────────────────────────────────────┐
│  信任域: companion_device_auth服务                           │
│  - useriam进程                                               │
│  - 权限验证                                                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ FFI
┌─────────────────────────────────────────────────────────────┐
│  高信任域: SecurityAgent (Rust)                              │
│  - 密钥操作                                                  │
│  - 安全存储                                                  │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 权限模型

**双重检查机制**:
1. **权限检查**: `ohos.permission.USE_USER_IDM`
2. **系统应用检查**: `TokenIdKit::IsSystemAppByFullTokenID()`

**检查位置**:
- N-API层: `frameworks/js/napi/src/companion_device_auth_entry.cpp:35-60`
- 服务层: `services/service_entry/src/companion_device_auth_service.cpp:515-528`

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 攻击面 | 入口 | 风险等级 |
|--------|------|----------|
| JS API | N-API接口 | 中 |
| IPC接口 | ICompanionDeviceAuth | 中 |
| 跨设备通信 | SoftBus通道 | 高 |
| 回调接口 | IIpc*Callback | 中 |
| 设备选择 | DeviceSelectCallback | 中 |
| 文件系统 | 模板存储 | 高 |
| 内存操作 | Rust/C++边界 | 高 |

### 2.2 输入来源

**外部输入**:
- JS调用参数 (templateId, businessIds等)
- IPC调用参数 (localUserId, callback对象)
- 跨设备消息 (SoftBus接收的数据)
- 回调注册 (IRemoteObject引用)

**内部状态**:
- 模板数据库存储
- 设备绑定关系
- Token凭证
- 会话状态

---

## 3. 可利用点分析

### 3.1 可利用点 #1: 权限检查绕过

**证据**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:35-45`

```cpp
bool CheckUseUserIdmPermission() {
    uint64_t fullTokenId = IPCSkeleton::GetCallingFullTokenID();
    AccessTokenID tokenId = fullTokenId & TOKEN_ID_LOW_MASK;
    if (AccessTokenKit::VerifyAccessToken(tokenId, USE_USER_IDM_PERMISSION) 
        != RET_SUCCESS) {
        return false;
    }
    return true;
}
```

**触发条件**:
- 获取系统应用权限
- 获取USE_USER_IDM权限

**影响**: 可调用所有API，管理伴随设备

**修复建议**:
- 定期进行权限审计
- 限制权限授予范围
- 增加操作日志记录

### 3.2 可利用点 #2: 回调对象生命周期

**证据**: `services/utils/src/callback_death_recipient.cpp:32-48`

```cpp
sptr<CallbackDeathRecipient> CallbackDeathRecipient::Register(
    const sptr<IRemoteObject> &remoteObj,
    DeathCallback &&callback)
{
    ENSURE_OR_RETURN_VAL(remoteObj != nullptr, nullptr);
    // ...
    if (!remoteObj->AddDeathRecipient(recipient)) {
        IAM_LOGE("AddDeathRecipient failed");
        return nullptr;
    }
    return recipient;
}
```

**触发条件**:
- 注册回调后进程崩溃
- 多次注册相同回调

**影响**: 资源泄漏或重复回调

**修复建议**:
- 确保回调唯一性检查
- 完善DeathRecipient清理逻辑

### 3.3 可利用点 #3: 跨设备消息处理

**证据**: SoftBus消息处理缺乏输入验证

**文件**: `services/cross_device_channels/soft_bus/src/soft_bus_channel.cpp`

**风险**: 
- 消息长度未严格校验
- 消息类型信任度不足

**影响**: 可能导致服务崩溃或信息泄露

**修复建议**:
- 严格校验消息长度和格式
- 添加消息签名验证
- 限制单设备消息频率

### 3.4 可利用点 #4: 模板ID处理

**证据**: `frameworks/js/napi/src/companion_device_auth_napi_helper.cpp:138-171`

```cpp
napi_status CompanionDeviceAuthNapiHelper::GetUint8ArrayValue(
    napi_env env, napi_value value, std::vector<uint8_t> &array)
{
    // 获取TypedArray信息
    napi_get_typedarray_info(env, value, &type, &length, &data, ...);
    if (type != napi_uint8_array) { return napi_invalid_arg; }
    
    array.resize(length);
    memcpy_s(array.data(), length, data, length);
}
```

**风险**:
- length为0时是否处理
- 超长数组可能导致内存问题

**修复建议**:
- 添加长度范围检查 (8字节模板ID)
- 限制数组最大长度

### 3.5 可利用点 #5: 纯软件实现安全存储

**证据**: `README_ZH.md`

> OpenHarmony开源架构内提供了伴随设备认证的纯软件实现，供开发者demo伴随设备认证功能，**纯软件实现部分并未包含伴随设备认证相关信息的安全存储能力**。

**风险**:
- 绑定凭证以明文或弱加密存储
- 密钥在内存中可被dump

**影响**: 攻击者可窃取凭证，伪造伴随设备

**修复建议**:
- 生产环境必须使用TEE/SE实现
- 基于硬件的安全存储
- 密钥派生使用硬件根密钥

---

## 4. 信任边界安全

### 4.1 JS/Native边界

**保护措施**:
- 权限检查 (`USE_USER_IDM`)
- 系统应用验证
- 参数类型检查 (N-API)

**风险**: JS层参数可直接传递至Native

### 4.2 Client/Service边界

**保护措施**:
- IPC权限检查
- Token提取验证
- DeathRecipient管理

**风险**: IPC调用可能被中间人攻击

### 4.3 C++/Rust边界

**保护措施**:
- FFI接口定义
- 数据序列化检查

**风险**: 内存安全边界问题

---

## 5. 输入验证

### 5.1 验证机制

**ENSURE_OR_RETURN宏** (约965处使用):

```cpp
#define ENSURE_OR_RETURN_VAL(cond, retVal) \
    do { \
        if (!(cond)) { \
            IAM_LOGE("(" #cond ") check fail, return"); \
            return (retVal); \
        } \
    } while (0)
```

**常见验证点**:
- 空指针检查: `ENSURE_OR_RETURN_VAL(ptr != nullptr, false)`
- 空容器检查: `ENSURE_OR_RETURN_VAL(!container.empty(), false)`
- 有效值检查: `ENSURE_OR_RETURN_VAL(value != 0, nullptr)`

### 5.2 验证覆盖度

| 输入类型 | 验证位置 | 覆盖度 |
|----------|----------|--------|
| JS参数 | N-API层 | 高 |
| IPC参数 | Service层 | 中 |
| 跨设备消息 | SoftBus层 | 低 |
| 存储数据 | 加载时 | 中 |

---

## 6. 安全加固

### 6.1 编译时加固

从 `companion_device_auth.gni`:

```gn
companion_device_auth_sanitize = {
    integer_overflow = true    # 整数溢出检测
    ubsan = true               # 未定义行为检测
    boundary_sanitize = true   # 边界检测
    cfi = true                 # 控制流完整性
    cfi_cross_dso = true       # 跨DSO CFI
}
```

### 6.2 运行时保护

**XCollie看门狗** (20s超时):

```cpp
XCollieHelper xcollie("AccessTokenKitAdapterImpl-CheckPermission", API_CALL_TIMEOUT);
```

**使用位置**:
- AccessToken检查
- UserAuth操作
- DeviceManager操作

---

## 7. 安全建议

### 7.1 高优先级

1. **生产环境使用TEE/SE**: 替换纯软件安全存储
2. **跨设备消息加密**: SoftBus消息添加端到端加密
3. **输入验证增强**: 统一跨设备消息验证框架

### 7.2 中优先级

1. **操作日志审计**: 记录关键操作(添加/删除设备、认证)
2. **限流机制**: 防止暴力攻击API
3. **证书固定**: 跨设备通信绑定设备证书

### 7.3 低优先级

1. **代码混淆**: N-API层添加混淆
2. **反调试**: 生产版本添加反调试

---

## 8. Rust FFI 边界安全风险 (补充分析)

### 8.1 FFI 架构概述

**位置**: `services/external_adapters/security_command_adapter/rust/`

Rust 代码通过 FFI 与 C++ 服务层交互，主要接口：

| 函数 | 位置 | 功能 |
|------|------|------|
| `init_rust_env()` | `entry/companion_device_auth_ffi.rs:1157` | 初始化 Rust 环境 |
| `uninit_rust_env()` | `entry/companion_device_auth_ffi.rs:1161` | 清理 Rust 环境 |
| `invoke_rust_command()` | `entry/companion_device_auth_ffi.rs:1171` | 命令分发入口 |

### 8.2 Unsafe 代码分析

#### 8.2.1 原始指针操作 (高风险)

**位置**: `services/external_adapters/security_command_adapter/rust/entry/companion_device_auth_ffi.rs:1193-1206`

```rust
// 行1193-1206: FFI 参数处理
ensure_or_return_val!(param.input_data_len != 0, ErrorCode::BadParam);
ensure_or_return_val!(!param.input_data.is_null(), ErrorCode::BadParam);

// unsafe 块 - 将 C 指针转为 Rust 切片
let input = unsafe { 
    slice::from_raw_parts(param.input_data, param.input_data_len as usize) 
};
let output = unsafe { 
    slice::from_raw_parts_mut(param.output_data, param.output_data_len as usize) 
};
```

**风险分析**:
- **空指针**: 已检查 `is_null()`
- **长度验证**: 已检查 `len != 0`
- **缓冲区溢出**: **风险** - 依赖 C 调用者提供正确的长度，如果 C 侧计算错误，会导致越界读写

**攻击场景**:
```
攻击者控制 C++ 层代码 → 传递错误的 input_data_len 
→ Rust 侧读取越界内存 → 信息泄露或崩溃
```

#### 8.2.2 内存转换操作 (中高风险)

**位置**: `services/external_adapters/security_command_adapter/rust/entry/companion_device_auth_entry.rs:86-94`

```rust
// 将字节流反序列化为结构体
*input_val = input.as_ptr().cast::<T>().read_unaligned();
// 将结构体序列化为字节流
output.as_mut_ptr().cast::<R>().write_unaligned(*output_val);
```

**风险分析**:
- **对齐问题**: 使用 `read_unaligned`/`write_unaligned` 避免对齐要求
- **大小匹配**: 依赖 `#[repr(C)]` 结构体布局与 C 侧完全匹配
- **类型混淆**: 如果 C 侧传递错误的 command_id，可能导致类型混淆

#### 8.2.3 全局可变状态 (中风险)

**位置**: `services/external_adapters/security_command_adapter/rust/traits/singleton_registry.rs:25-50`

```rust
static mut INSTANCE: Option<Box<dyn SingletonRegistry>> = None;

pub fn get_singleton_registry() -> &'static dyn SingletonRegistry {
    unsafe { INSTANCE.as_ref().unwrap() }
}
```

**风险分析**:
- **线程安全**: 依赖单线程初始化假设
- **空指针**: 如果 `init_rust_env()` 未调用，会 panic

#### 8.2.4 文件权限操作 (低风险)

**位置**: `services/external_adapters/security_command_adapter/rust/impls/default_storage_io.rs:83-91`

```rust
unsafe {
    libc::chmod(c_path.as_ptr(), S_IRUSR | S_IWUSR)
}
```

**风险分析**:
- 设置文件权限为 0o600 (owner read/write)
- 路径通过 `CString::new()` 验证
- 标准 POSIX 调用，风险较低

### 8.3 Token 序列化安全

**位置**: `services/external_adapters/security_command_adapter/rust/utils/auth_token.rs:94-105`

```rust
// 序列化
pub fn serialize(&self) -> &[u8] {
    unsafe { 
        core::slice::from_raw_parts(self as *const Self as *const u8, 
                                     core::mem::size_of::<Self>()) 
    }
}

// 反序列化
pub fn deserialize(bytes: &[u8]) -> Result<Self, ErrorCode> {
    if bytes.len() != core::mem::size_of::<Self>() {
        return Err(ErrorCode::GeneralError);
    }
    let raw = unsafe { 
        core::ptr::read_unaligned(bytes.as_ptr() as *const UserAuthTokenRaw) 
    };
    raw.into_token()
}
```

**安全评估**:
- ✅ 使用 `#[repr(C, packed)]` 确保布局稳定
- ✅ 反序列化前检查长度
- ✅ 使用 `read_unaligned` 避免对齐要求
- ⚠️ 依赖结构体无填充字节

### 8.4 加密实现安全

**位置**: `services/external_adapters/security_command_adapter/rust/impls/openssl_crypto_engine.rs`

| 算法 | 实现 | 安全检查 |
|------|------|----------|
| AES-256-GCM | OpenSSL | 密钥长度检查 (32字节) |
| Ed25519 | OpenSSL | 私钥零化 (Drop trait) |
| HKDF-SHA256 | OpenSSL | 输出长度固定 32字节 |
| X25519 | OpenSSL | 密钥长度检查 (32字节) |

**密钥零化实现**:
```rust
impl Drop for KeyPair {
    fn drop(&mut self) {
        self.pri_key.fill(0);  // 零化私钥
        self.pri_key.clear();
    }
}
```

### 8.5 FFI 风险缓解建议

#### 高优先级
1. **边界检查强化**: 在 FFI 层添加更严格的缓冲区边界验证
2. **类型混淆防护**: 添加 command_id 与数据结构类型的映射校验
3. **线程安全**: 将 `static mut` 替换为 `std::sync::OnceLock`

#### 中优先级
4. **模糊测试**: 对 FFI 接口进行 fuzzing，测试异常输入
5. **内存审计**: 定期检查 Rust/C++ 边界内存使用情况
6. **日志增强**: FFI 入口/出口添加详细日志

---

## 9. 检查局限性

本次安全分析的范围和局限:

1. **已覆盖**: Rust FFI 边界基础安全风险
2. **部分覆盖**: Rust 安全核心实现 (需要更深入的 Rust 安全审计)
3. **未覆盖**: SoftBus底层安全机制
4. **未覆盖**: TEE/SE生产实现细节
5. **未覆盖**: UserAuth框架安全假设

---

*文档更新时间: 2026-02-07*  
*更新内容: 补充 Rust FFI 边界安全风险分析*

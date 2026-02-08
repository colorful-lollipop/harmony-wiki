# 安全风险评审

## 目的

本文档基于代码证据对 `telephony_core_service` 进行安全风险评审，包括攻击面分析、信任边界识别和可被利用点梳理。

---

## 威胁模型

### 攻击面清单

| 攻击面 | 入口 | 风险等级 |
|--------|------|----------|
| N-API (JS 接口) | JS API 调用 | 高 |
| IPC 接口 | SA:4010 接口 | 高 |
| RIL HDI 接口 | libril_proxy | 中 |
| 文件系统 | 配置文件、数据库 | 中 |
| 系统事件 | CommonEvent | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  Untrusted Zone (Third-party Apps)                          │
│  ─────────────────────────────────                          │
│  JS API calls with permission checks                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ Permission Check
┌─────────────────────────────────────────────────────────────┐
│  Semi-Trusted Zone (System Apps)                            │
│  ─────────────────────────────────                          │
│  JS API calls with system permissions                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC
┌─────────────────────────────────────────────────────────────┐
│  Trusted Zone (CoreService SA:4010)                         │
│  ─────────────────────────────────                          │
│  Internal API calls, RIL communication                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 可被利用点分析

### 1. SlotId 参数校验绕过风险

**证据** (`frameworks/js/sim/src/napi_sim.cpp:52-66`):

```cpp
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}

static inline bool IsValidSlotIdEx(int32_t slotId)
{
    // One more slot for VSim.
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT + 1));
}
```

**问题**: 
- 不同 API 使用不同的校验函数 (`IsValidSlotId` vs `IsValidSlotIdEx`)
- 某些 API 可能遗漏 slotId 校验

**触发路径**:
```
JS API → NAPI 层 → 校验通过 → CoreService → SimManager
                                    ↓
                              可能访问越界数组
```

**影响**: 
- 数组越界访问
- 信息泄露
- 潜在的拒绝服务

**修复建议**:
1. 统一 slotId 校验函数
2. 在 CoreService 层增加二次校验
3. 使用 `std::vector` 代替原始数组

---

### 2. 权限检查绕过风险 (IRawParcelCallback)

**证据** (`services/core/src/core_service.cpp:196-200`):

```cpp
int32_t CoreService::GetPsRadioTech(int32_t slotId, int32_t &psRadioTech)
{
    if (!TelephonyPermission::CheckPermission(Permission::GET_NETWORK_INFO)) {
        TELEPHONY_LOGE("permission denied!");
        return TELEPHONY_ERR_PERMISSION_ERR;
    }
    // ...
}
```

**证据** (`frameworks/native/src/core_service_client.cpp`):

某些接口通过 `IRawParcelCallback` 返回数据，回调内部可能绕过权限检查。

**问题**:
- 回调模式下的权限检查可能不完整
- 某些敏感接口可能缺少权限校验

**触发路径**:
```
JS → NAPI → CoreServiceClient → IPC → CoreService
                                         ↓
                                  某些路径缺少 CheckPermission()
```

**影响**:
- 未授权访问敏感信息 (IMEI/IMSI)
- 违反最小权限原则

**修复建议**:
1. 审计所有 IPC 接口的权限检查
2. 在 CoreServiceStub 统一层增加权限校验
3. 建立权限检查清单

---

### 3. PIN/PUK 明文处理风险

**证据** (多处涉及 PIN/PUK 的接口):

```cpp
int32_t CoreService::UnlockPin(int32_t slotId, const std::u16string &pin, 
                               const sptr<IRawParcelCallback> &callback);
int32_t CoreService::UnlockPuk(int32_t slotId, const std::u16string &newPin, 
                               const std::u16string &puk, ...);
int32_t CoreService::AlterPin(int32_t slotId, const std::u16string &newPin, 
                              const std::u16string &oldPin, ...);
```

**问题**:
- PIN/PUK 以 `std::u16string` 明文传递
- 可能在内存中残留
- IPC 传输过程可能被截获

**触发路径**:
```
JS 输入 PIN → NAPI 转换 → IPC Parcel → CoreService → RIL
     ↑                                           ↓
  内存中明文                               日志可能泄露
```

**影响**:
- 敏感信息泄露
- SIM 卡被解锁的风险

**修复建议**:
1. 使用安全内存 (`SecureString`) 存储敏感数据
2. 及时清零内存
3. 禁止在日志中输出 PIN/PUK
4. 使用加密 IPC 通道

---

### 4. IPC 回调对象生命周期风险

**证据** (`interfaces/innerkits/include/i_raw_parcel_callback.h`):

```cpp
class IRawParcelCallback : public IRemoteBroker {
public:
    virtual int32_t OnCallback(int32_t errorCode, const MessageParcel &data) = 0;
};
```

**问题**:
- 回调对象通过 `sptr` 管理，但可能存在时序问题
- 客户端销毁后服务端仍可能调用回调

**触发路径**:
```
Client ──sptr──→ CoreService (注册回调)
   │                    ↓
   │  客户端提前销毁   异步操作完成，调用回调
   │                    ↓
   └────────────── 悬空指针访问
```

**影响**:
-  Use-after-free
-  程序崩溃
-  潜在的代码执行

**修复建议**:
1. 使用 `DeathRecipient` 监听客户端死亡
2. 回调前检查对象有效性
3. 使用弱引用或 ID 代替直接指针

---

### 5. RIL 数据解析整数溢出风险

**证据** (`services/tel_ril/src/tel_ril_base.cpp` 等):

```cpp
// 典型的 Parcel 读取代码
int32_t slotId = data.ReadInt32();
int32_t serialId = data.ReadInt32();
// 直接使用，无范围校验
```

**问题**:
- 从 RIL 接收的数据直接解析使用
- 缺少对数值范围的校验
- 可能导致整数溢出

**触发路径**:
```
Modem → RIL Adapter → CoreService → 解析数据
                           ↓
                    恶意构造的数据包
                           ↓
                    整数溢出/数组越界
```

**影响**:
- 缓冲区溢出
- 拒绝服务
- 远程代码执行

**修复建议**:
1. 所有外部输入数据严格校验范围
2. 使用安全的解析函数
3. 增加 fuzz 测试

---

### 6. 配置文件解析风险

**证据** (`services/sim/src/operator_file_parser.cpp`):

```cpp
// 解析运营商配置文件
bool OperatorFileParser::ParseFromFile(const std::string &path) {
    // 读取并解析 XML/JSON 配置
}
```

**问题**:
- 运营商配置从文件系统加载
- 解析器可能存在漏洞
- 路径可能被篡改

**触发路径**:
```
文件系统 → OperatorFileParser → 解析配置
               ↓
        恶意构造的配置文件
               ↓
        XXE/路径遍历/解析漏洞
```

**影响**:
- 路径遍历攻击
- XML 外部实体注入 (XXE)
- 配置篡改

**修复建议**:
1. 使用沙箱路径
2. 禁用 XML 外部实体
3. 校验配置文件签名
4. 使用 JSON 替代 XML

---

### 7. Dump 接口信息泄露风险

**证据** (`services/core/src/core_service_dump_helper.cpp`):

```cpp
int32_t CoreServiceDumpHelper::Dump(int32_t fd, const std::vector<std::u16string> &args) {
    // 输出各种内部状态信息
}
```

**问题**:
- Dump 接口输出敏感信息
- 可能包含 IMSI、电话号码等

**触发路径**:
```
shell → hidumper -s 4010 → CoreService::Dump()
                                ↓
                         输出敏感信息到文件
```

**影响**:
- 敏感信息泄露
- 隐私侵犯

**修复建议**:
1. Dump 输出脱敏处理
2. 限制 Dump 接口访问权限
3. 分级 Dump 级别

---

## 安全检查清单

### 已检查的代码范围

| 范围 | 文件/目录 | 检查结果 |
|------|----------|----------|
| N-API 入口 | `frameworks/js/*` | ✅ 已检查 |
| IPC 接口 | `services/core/src/core_service*.cpp` | ✅ 已检查 |
| 权限校验 | `utils/common/src/telephony_permission.cpp` | ✅ 已检查 |
| RIL 通信 | `services/tel_ril/src/*.cpp` | ✅ 已检查 |
| 配置文件解析 | `services/sim/src/operator_file_parser.cpp` | ✅ 已检查 |
| Dump 功能 | `services/core/src/core_service_dump_helper.cpp` | ✅ 已检查 |

### 未检查的范围

- 测试代码 (`test/`)
- eSIM ASN.1 编解码 (`utils/codec/`)
- vCard 解析 (`utils/vcard/`)

---

## 安全建议总结

### 高优先级

1. **统一参数校验**: 所有 slotId 和数组索引统一校验
2. **权限审计**: 全面审计 IPC 接口权限检查
3. **敏感数据处理**: PIN/PUK 使用安全内存

### 中优先级

4. **回调生命周期**: 修复回调对象生命周期管理
5. **输入校验**: 强化 RIL 数据解析的边界检查
6. **配置安全**: 配置文件解析加固

### 低优先级

7. **Dump 脱敏**: 敏感信息输出脱敏
8. **日志审计**: 检查日志中是否包含敏感信息

---

## 相关链接

- [N-API 接口](./03_NAPI_API.md)
- [内部 API](./04_Inner_API.md)
- [架构设计](./02_Architecture.md)

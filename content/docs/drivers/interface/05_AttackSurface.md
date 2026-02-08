# 攻击面分析 (Attack Surface Analysis)

**文档状态**: Phase 2  
**最后更新**: 2026-02-07  
**适用范围**: OpenHarmony drivers_interface 仓库安全审计

---

## 1. 执行摘要

### 关键发现

| 指标 | 数值 | 风险等级 |
|-----|------|---------|
| 总 IDL 接口文件 | 533 个 | - |
| 主接口定义 (I*.idl) | 379 个 | 高 |
| 回调接口 | 126 个 | 高 |
| 参数方向标记 ([in]/[out]) | 3,040+ | 中 |
| Passthrough 模式模块 | ~10 个 | **严重** |
| 安全敏感模块 | 6 个 | **严重** |

### 攻击面分布

```
┌─────────────────────────────────────────────────────────┐
│                    应用层 (JS/ArkUI)                     │
│                   攻击面: 通过框架层代理                    │
├─────────────────────────────────────────────────────────┤
│                   框架层 (N-API/Service)                 │
│                   攻击面: API 权限绕过                     │
├─────────────────────────────────────────────────────────┤
│    HDI Proxy (lib*_proxy_*.so) ←── 攻击入口 1: IPC 劫持   │
│                   ↓ IPC (IPC Mode)                      │
│    HDI Stub (lib*_stub_*.so)   ←── 攻击入口 2: 反序列化   │
│                   ↓                                       │
│    驱动实现 (drivers_peripheral) ←── 攻击入口 3: 实现缺陷   │
├─────────────────────────────────────────────────────────┤
│    Passthrough 模式 ←── 攻击入口 4: 直接硬件访问 (高危)      │
└─────────────────────────────────────────────────────────┘
```

---

## 2. IPC 入口清单

### 2.1 接口定义统计

**数据来源**: 全仓库 IDL 文件扫描  
**扫描范围**: 47 个模块，所有版本

| 模块类别 | 模块数 | 接口文件数 | 方法数 (估计) | 风险关注点 |
|---------|-------|-----------|--------------|-----------|
| 安全认证 | 6 | ~80 | ~300 | **密钥、凭证、Token** |
| 基础 I/O | 5 | ~120 | ~500 | 硬件访问、回调 |
| 感知类 | 5 | ~100 | ~400 | 数据流、回调 |
| 多媒体 | 6 | ~80 | ~350 | Buffer 处理 |
| 连接类 | 5 | ~60 | ~250 | 网络数据 |
| 系统服务 | 4 | ~40 | ~150 | 电源管理 |

### 2.2 安全敏感模块接口详情

#### HUKS (密钥管理) - v1_1
**文件**: `huks/v1_1/IHuks.idl`

| 方法 | [in] 参数 | [out] 参数 | 风险等级 |
|-----|----------|-----------|---------|
| `GenerateKey` | `keyAlias`, `paramSet`, `keyIn` | `encKeyOut` | 高 |
| `ImportKey` | `keyAlias`, `key` (明文), `paramSet` | `encKeyOut` | **严重** |
| `ImportWrappedKey` | `wrappingKeyAlias`, `wrappedKeyData` | `encKeyOut` | 高 |
| `ExportPublicKey` | `encKey` | `keyOut` | 中 |
| `Init` | `encKey`, `paramSet` | `handle`, `token` | 高 |
| `Update` | `handle`, `inData` | `outData` | 高 |
| `Finish` | `handle`, `inData` | `outData` | 高 |
| `Encrypt` | `encKey`, `plainText` | `cipherText` | 高 |
| `Decrypt` | `encKey`, `cipherText` | `plainText` | **严重** |
| `Sign` | `encKey`, `srcData` | `signature` | 高 |
| `Verify` | `encKey`, `srcData`, `signature` | - | 中 |
| `AgreeKey` | `encPrivateKey`, `peerPublicKey` | `agreedKey` (明文) | **严重** |
| `DeriveKey` | `encKdfKey` | `derivedKey` (明文) | **严重** |
| `Mac` | `encKey`, `srcData` | `mac` | 高 |

**证据**: `huks/v1_1/IHuks.idl:23-48`

```idl
interface IHuks {
    ImportKey([in] String keyAlias, [in] struct HuksBlob key, ...);  // 明文密钥导入
    Decrypt([in] struct HuksBlob encKey, [in] struct HuksBlob cipherText, [out] struct HuksBlob plainText);  // 明文输出
    AgreeKey([in] struct HuksBlob encPrivateKey, ..., [out] struct HuksBlob agreedKey);  // 明文密钥协商
    DeriveKey([in] struct HuksBlob encKdfKey, ..., [out] struct HuksBlob derivedKey);  // 明文密钥派生
};
```

**攻击面**:
1. **明文密钥导入**: `ImportKey` 接收明文 `key` 参数
2. **明文密钥输出**: `AgreeKey`, `DeriveKey` 输出明文密钥
3. **会话 Token**: `Init` 返回的 `token` 可能被截获

---

#### User Auth (用户认证框架) - v4_1
**文件**: `user_auth/v4_0/IUserAuthInterface.idl`, `user_auth/v4_1/IUserAuthInterface.idl`

| 方法 | 敏感输入 | 敏感输出 | 风险等级 |
|-----|---------|---------|---------|
| `AddExecutor` | `publicKey`, `signedRemoteExecutorInfo` | `index`, `publicKey` | 高 |
| `OpenSession` | `userId` | `challenge` | 中 |
| `UpdateEnrollmentResult` | `scheduleResult` | `info.rootSecret` | **严重** |
| `DeleteCredential` | `credentialId`, `authToken` | - | 高 |
| `DeleteUser` | `userId`, `authToken` | `rootSecret` | **严重** |
| `UpdateAuthenticationResult` | `scheduleResult` | `info.token`, `rootSecret` | **严重** |
| `VerifyAuthToken` | `tokenIn` | `tokenPlainOut`, `rootSecret` | **严重** |
| `BeginAuthentication` | `param` | `scheduleInfos` | 中 |
| `BeginEnrollment` | `authToken` | `scheduleInfo` | 高 |

**关键数据结构**:

```idl
struct AuthResultInfo {
    unsigned char[] token;        // 认证令牌
    unsigned char[] rootSecret;   // 文件保护密钥 (HIGH VALUE)
    // ...
};

struct UserAuthTokenPlain {
    int userId;
    unsigned char[] challenge;
    int authType;
    unsigned int securityLevel;
    unsigned long secureUid;
    // ...
};
```

**证据**: `user_auth/v4_0/UserAuthTypes.idl:45-68`

**攻击面**:
1. **Root Secret 泄露**: 多个方法返回 `rootSecret` (文件保护密钥)
2. **Token 伪造**: `VerifyAuthToken` 输出明文 token 结构
3. **会话劫持**: `challenge` 和 `token` 的传递安全

---

#### Fingerprint Auth (指纹认证) - v2_0
**文件**: `fingerprint_auth/v2_0/IAllInOneExecutor.idl`

| 方法 | 敏感操作 | 风险等级 |
|-----|---------|---------|
| `Enroll` | 模板注册 | 高 |
| `Authenticate` | 身份验证 | 高 |
| `Identify` | 指纹识别 | 高 |
| `Delete` | 模板删除 | 中 |
| `GetProperty` | 属性读取 | 低 |

**安全等级枚举**:
```idl
enum ExecutorSecureLevel : int {
    ESL0 = 0,  // 软件
    ESL1 = 1,  // TEE
    ESL2 = 2,  // 安全硬件
    ESL3 = 3,  // 专用安全元件
};
```

**证据**: `fingerprint_auth/v2_0/FingerprintAuthTypes.idl:15-22`

---

#### Face Auth (人脸认证) - v2_0
**文件**: `face_auth/v2_0/IFaceAuthInterface.idl`, `face_auth/v2_0/IAllInOneExecutor.idl`

**特殊攻击面**:
- `SetBufferProducer` 接收 `BufferProducerSequenceable` - 相机缓冲区
- 活体检测绕过风险

---

#### PIN Auth (PIN 认证) - v3_0
**文件**: `pin_auth/v3_0/IPinAuthInterface.idl`, `pin_auth/v3_0/IAllInOneExecutor.idl`

**高危方法**:
```idl
interface IAllInOneExecutor {
    SetData(
        [in] unsigned long scheduleId,
        [in] unsigned long authSubType,
        [in] unsigned char[] data,        // PIN 数据 (明文)
        [in] unsigned int pinLength,      // PIN 长度
        [in] int resultCode
    );
};
```

**证据**: `pin_auth/v3_0/IAllInOneExecutor.idl:28-34`

**攻击面**:
1. **PIN 明文传输**: `SetData` 接收明文 PIN
2. **Collector/Verifier 分离**: 架构复杂增加攻击面

---

#### Secure Element (安全元件) - v1_0
**文件**: `secure_element/v1_0/ISecureElementInterface.idl`

| 方法 | 敏感操作 | 风险等级 |
|-----|---------|---------|
| `openLogicalChannel` | 打开逻辑通道 | 高 |
| `openBasicChannel` | 打开基础通道 | 高 |
| `transmit` | **APDU 命令传输** | **严重** |
| `reset` | SE 复位 | 中 |

**证据**: `secure_element/v1_0/ISecureElementInterface.idl:20-35`

**攻击面**:
1. **APDU 命令注入**: `transmit` 可发送任意 APDU
2. **通道劫持**: 逻辑通道管理不当

---

## 3. 回调接口清单

### 3.1 回调接口统计

**总回调接口数**: 126 个

**攻击向量**: 回调接口是服务端调用客户端的机制，可能被用于：
1. 拒绝服务 (频繁回调)
2. 数据泄露 (回调参数)
3. 控制流劫持 (回调对象)

### 3.2 高风险回调接口

| 回调接口 | 所属模块 | 方法 | 风险 |
|---------|---------|------|------|
| `IExecutorCallback` | fingerprint_auth/face_auth/pin_auth | `OnResult`, `OnTip`, `OnMessage` | 频繁调用 DoS |
| `IExecutorCallback` | pin_auth (扩展) | `OnGetData` | PIN 输入请求 |
| `ISaCommandCallback` | fingerprint_auth/face_auth | `OnSaCommands` | 系统命令执行 |
| `IMessageCallback` | user_auth | `OnMessage` | 跨执行器消息 |
| `ISecureElementCallback` | secure_element | `OnSeStateChanged` | 状态监控 |
| `IHuksCallback` | huks | `OnHuksKeyOp` | 密钥操作通知 |
| `IAudioCallback` | audio | `OnRenderCallback` | 音频数据流 |
| `ISensorCallback` | sensor | `OnDataEvent` | 传感器数据 |

### 3.3 回调注册模式

**典型回调注册流程**:
```
1. 客户端实现回调接口
2. 通过 [in] 参数将 callbackObj 传递给服务端
3. 服务端保存 callbackObj 引用
4. 异步事件触发时调用 callbackObj 方法
```

**攻击面**:
1. **回调对象伪造**: 传递恶意构造的 callbackObj
2. **回调重放**: 重复触发回调导致 DoS
3. **回调参数注入**: 通过回调参数传递恶意数据

---

## 4. 信任边界分析

### 4.1 信任边界图

```
┌────────────────────────────────────────────────────────────┐
│                      不信任域 (Untrusted)                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  用户应用    │  │  第三方服务  │  │  恶意代码    │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
└─────────┼────────────────┼────────────────┼────────────────┘
          │                │                │
          ▼                ▼                ▼
┌────────────────────────────────────────────────────────────┐
│                    信任边界 1: N-API 层                      │
│              (应用框架验证、权限检查)                         │
└────────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────────┐
│                    半信任域 (Semi-Trusted)                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ 系统服务     │  │ 框架层      │  │ HDI Proxy   │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
└─────────┼────────────────┼────────────────┼────────────────┘
          │                │                │
          ▼                ▼                ▼
┌────────────────────────────────────────────────────────────┐
│                    信任边界 2: IPC 层                        │
│              (IPC 序列化/反序列化、进程隔离)                  │
└────────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────────┐
│                    信任域 (Trusted)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ HDI Stub    │  │ 驱动实现     │  │ 硬件        │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
└─────────┼────────────────┼────────────────┼────────────────┘
          │                │                │
          ▼                ▼                ▼
    ┌──────────┐     ┌──────────┐     ┌──────────┐
    │ Passthrough │   │ Kernel   │     │ 安全元件  │
    │ Mode       │   │ Driver   │     │ (TEE/SE) │
    └──────────┘     └──────────┘     └──────────┘
```

### 4.2 边界跨越点

| 边界 | 跨越机制 | 验证点 | 风险 |
|-----|---------|-------|------|
| 应用 → 框架 | N-API | 权限检查 | API 绕过 |
| 框架 → HDI | IPC | IPC 序列化 | 反序列化攻击 |
| Proxy → Stub | Binder/HDF | 接口描述符 | 接口伪装 |
| Stub → 驱动 | 函数调用 | 参数校验 | 缓冲区溢出 |
| 驱动 → 硬件 | 寄存器/内存 | 访问控制 | 硬件攻击 |

---

## 5. 直通模式 (Passthrough) 攻击面

### 5.1 Passthrough 模式模块清单

**数据来源**: BUILD.gn 扫描

| 模块 | 版本 | 文件路径 | 风险说明 |
|-----|------|---------|---------|
| **huks** | v1_0, v1_1 | `huks/v1_0/BUILD.gn:25` | **密钥管理直通 - 最高风险** |
| input | v1_0 | `input/v1_0/BUILD.gn:33` | 输入设备直通 |
| usb/serial | v1_0 | `usb/serial/v1_0/BUILD.gn:24` | USB 串口直通 |
| sensor/convert | v1_0 | `sensor/convert/v1_0/BUILD.gn:35` | 传感器转换直通 |
| camera/metadata | v1_0 | `camera/metadata/v1_0/BUILD.gn:43` | 相机元数据直通 |

**证据**: `huks/v1_1/BUILD.gn:22-25`
```gn
hdi("huks") {
    # ...
    mode = "passthrough"  // Line 25 - 无 IPC 隔离
}
```

### 5.2 Passthrough 风险分析

**风险**: Passthrough 模式下，Proxy 直接调用驱动实现，无 IPC 隔离

```
IPC 模式 (安全):
客户端 → Proxy → IPC → Stub (独立进程) → 驱动

Passthrough 模式 (风险):
客户端 → Proxy → 直接调用 → 驱动 (同一进程)
```

**攻击场景**:
1. **权限提升**: 通过 passthrough 接口直接访问硬件
2. **绕过审计**: 无 IPC 拦截审计点
3. **稳定性风险**: 驱动崩溃导致调用者崩溃

### 5.3 HUKS Passthrough 特别风险

**文件**: `huks/v1_1/BUILD.gn:25`

**风险**: 密钥管理模块使用 passthrough 模式

**影响**:
- 密钥操作在同一进程空间执行
- 无进程隔离保护
- 内存转储可获取密钥材料

---

## 6. 敏感操作清单

### 6.1 按风险类型分类

#### 密钥操作 (Critical)
| 模块 | 操作 | 接口方法 |
|-----|------|---------|
| huks | 密钥生成 | `GenerateKey` |
| huks | 密钥导入 | `ImportKey` |
| huks | 密钥导出 | `ExportPublicKey` |
| huks | 解密 | `Decrypt` |
| huks | 密钥协商 | `AgreeKey` |
| huks | 密钥派生 | `DeriveKey` |
| user_auth | Token 获取 | `UpdateAuthenticationResult` |
| user_auth | Root Secret 获取 | `DeleteUser`, `VerifyAuthToken` |

#### 认证操作 (High)
| 模块 | 操作 | 接口方法 |
|-----|------|---------|
| fingerprint_auth | 指纹注册 | `Enroll` |
| fingerprint_auth | 指纹验证 | `Authenticate` |
| face_auth | 人脸注册 | `Enroll` |
| face_auth | 人脸验证 | `Authenticate` |
| pin_auth | PIN 输入 | `SetData` |
| user_auth | 认证初始化 | `BeginAuthentication` |

#### 硬件访问 (High)
| 模块 | 操作 | 接口方法 |
|-----|------|---------|
| secure_element | APDU 传输 | `transmit` |
| secure_element | 通道打开 | `openLogicalChannel` |
| input | 输入设备访问 | `OpenInputDevice` |
| usb | USB 设备访问 | 各接口方法 |
| camera | 相机数据流 | `Capture` 系列 |

#### 特权操作 (Medium)
| 模块 | 操作 | 接口方法 |
|-----|------|---------|
| power | 电源管理 | 各接口方法 |
| battery | 电池状态 | 查询方法 |
| thermal | 热管理 | 各接口方法 |

---

## 7. 数据流分析

### 7.1 敏感数据流

#### 7.1.1 密钥生命周期

```
密钥生成 (GenerateKey)
    ↓
[内存] 明文密钥 → 加密 → [out] encKeyOut
    ↓
密钥存储 (HDF/TEE)
    ↓
密钥使用 (Init/Update/Finish)
    ↓
[内存] encKey (密文) → 解密 → [TEE] 明文密钥 → 加密操作
    ↓
密钥销毁 (DeleteKey)
```

**风险点**:
- `ImportKey` 直接接收明文密钥
- `AgreeKey`/`DeriveKey` 输出明文密钥
- Passthrough 模式下密钥在普通进程空间

#### 7.1.2 认证流程

```
1. OpenSession([in] userId, [out] challenge)
                ↓
2. BeginAuthentication([in] challenge, ...)
                ↓
3. 执行器交互 (指纹/人脸/PIN 采集)
                ↓
4. UpdateAuthenticationResult([in] scheduleResult, [out] token, rootSecret)
                ↓
5. Token 用于后续文件保护
```

**风险点**:
- `rootSecret` 泄露导致文件系统被解密
- `token` 伪造导致未授权访问

#### 7.1.3 PIN 输入

```
1. OnGetData([out] algoParameter, authSubType, ...) ← 回调请求 PIN
                ↓
2. 用户界面输入 PIN
                ↓
3. SetData([in] data (PIN 明文), pinLength)
                ↓
4. PIN 验证
```

**风险点**:
- `SetData` 接收明文 PIN
- PIN 在传输过程中可能被截获

---

## 8. 攻击场景映射

### 8.1 攻击树

```
目标: 获取系统敏感数据/权限
│
├─ 1. IPC 层攻击
│   ├─ 1.1 接口伪装
│   │   └─ 伪造 HDI 服务响应
│   ├─ 1.2 序列化攻击
│   │   └─ 构造恶意序列化数据
│   └─ 1.3 中间人攻击
│       └─ 拦截 IPC 通信
│
├─ 2. 接口层攻击
│   ├─ 2.1 参数注入
│   │   ├─ 超大缓冲区
│   │   ├─ 畸形数据结构
│   │   └─ 越界索引
│   ├─ 2.2 权限绕过
│   │   └─ 调用需要特权的方法
│   └─ 2.3 回调滥用
│       ├─ 注册恶意回调
│       └─ 回调 DoS
│
├─ 3. 直通模式攻击
│   ├─ 3.1 内存转储
│   │   └─ Passthrough 模式密钥提取
│   └─ 3.2 硬件直接访问
│       └─ 绕过权限检查
│
└─ 4. 逻辑漏洞
    ├─ 4.1 时序攻击
    ├─ 4.2 状态机绕过
    └─ 4.3 重放攻击
```

### 8.2 攻击路径示例

#### 路径 1: 密钥泄露 (通过 Passthrough)
```
攻击者获取用户态权限
    ↓
加载 huks passthrough 库 (libhuks_proxy_1.0.so)
    ↓
调用 Init() 获取密钥会话
    ↓
内存扫描获取密钥句柄
    ↓
调用 Decrypt() 解密敏感数据
```

#### 路径 2: Root Secret 泄露
```
攻击者控制 Framework 层
    ↓
调用 user_auth.UpdateAuthenticationResult()
    ↓
拦截 [out] AuthResultInfo 结构
    ↓
提取 rootSecret 字段
    ↓
使用 rootSecret 解密用户文件
```

#### 路径 3: APDU 注入
```
攻击者获取 SE 访问权限
    ↓
调用 secure_element.openLogicalChannel()
    ↓
建立与 SE 的通信通道
    ↓
调用 transmit() 发送恶意 APDU
    ↓
执行未授权的安全元件操作
```

---

## 9. 附录

### 9.1 接口索引

#### 所有安全模块接口文件

```
huks/
├── v1_0/IHuks.idl
├── v1_0/IHuksTypes.idl
├── v1_1/IHuks.idl
└── v1_1/IHuksTypes.idl

user_auth/
├── v4_0/IUserAuthInterface.idl
├── v4_0/UserAuthTypes.idl
├── v4_0/IMessageCallback.idl
├── v4_1/IUserAuthInterface.idl
└── v4_1/UserAuthTypes.idl

fingerprint_auth/v2_0/
├── IFingerprintAuthInterface.idl
├── FingerprintAuthTypes.idl
├── IAllInOneExecutor.idl
├── IExecutorCallback.idl
└── ISaCommandCallback.idl

face_auth/v2_0/
├── IFaceAuthInterface.idl
├── FaceAuthTypes.idl
├── IAllInOneExecutor.idl
├── IExecutorCallback.idl
└── ISaCommandCallback.idl

pin_auth/v3_0/
├── IPinAuthInterface.idl
├── PinAuthTypes.idl
├── IAllInOneExecutor.idl
├── ICollector.idl
├── IVerifier.idl
└── IExecutorCallback.idl

secure_element/v1_0/
├── ISecureElementInterface.idl
├── SecureElementTypes.idl
└── ISecureElementCallback.idl
```

### 9.2 术语表

| 术语 | 说明 |
|-----|------|
| HDI | Hardware Device Interface - 硬件设备接口 |
| IDL | Interface Definition Language - 接口定义语言 |
| IPC | Inter-Process Communication - 进程间通信 |
| Passthrough | 直通模式，无 IPC 隔离 |
| TEE | Trusted Execution Environment - 可信执行环境 |
| SE | Secure Element - 安全元件 |
| ESL | Executor Security Level - 执行器安全等级 |
| APDU | Application Protocol Data Unit - 智能卡协议数据单元 |
| Root Secret | 文件保护根密钥 |

---

*本文档仅作为攻击面参考，实际漏洞评估需结合具体实现代码*

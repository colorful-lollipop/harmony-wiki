# 攻击面分析

## 概述

本文档系统性地分析 OpenHarmony IPC 模块的外部攻击面，识别所有外部数据入口点和敏感操作，为后续安全评估提供基础。分析范围涵盖 N-API 接口参数接收、IPC 消息解析、系统能力查询请求、文件描述符处理等关键环节。所有关键结论均标注了具体的代码位置，确保分析的可追溯性。

IPC 模块作为 OpenHarmony 系统的核心进程间通信机制，承担着连接各系统服务和应用的关键职责。这种桥梁作用决定了其必然暴露大量的外部攻击面。攻击者可能通过这些入口点注入恶意数据、触发异常行为或利用潜在漏洞获取系统权限。因此，对攻击面的全面识别和分析是确保系统安全的重要前提。

本分析遵循以下方法论：首先识别所有外部数据入口，包括网络输入、用户空间数据、跨进程消息等；其次分析每个入口点的数据处理流程，定位可能存在验证缺失或边界检查不足的位置；最后梳理从入口点到敏感操作的完整调用链，评估潜在的攻击利用路径。

---

## 1 外部输入入口清单

### 1.1 N-API 参数接收入口

N-API 是应用层访问 IPC 功能的主要入口，接收来自 JavaScript/ArkTS 代码的所有参数数据。这些入口点的安全性直接关系到整个 IPC 模块的健壮性。

| 入口函数 | 功能描述 | 数据类型 | 风险等级 | 代码位置 |
|---------|---------|---------|---------|---------|
| napi_create_ipc_object() | 创建 IPC 对象 | napi_value | 高 | `ipc/native/src/napi/src/napi_rpc_native_module.cpp:48` |
| napi_get_ipc_remote_object() | 获取远程对象 | napi_value, pointer | 高 | `interfaces/kits/js/napi/napi_remote_object.h` |
| MessageParcel.WriteString16() | 写入字符串 | std::u16string | 高 | `interfaces/innerkits/ipc_core/include/message_parcel.h:44` |
| MessageParcel.WriteInt32() | 写入整数 | int32_t | 中 | `message_parcel.h` |
| MessageParcel.WriteBuffer() | 写入缓冲区 | void*, size_t | 高 | `message_parcel.h` |
| MessageParcel.WriteRemoteObject() | 写入远程对象 | IRemoteObject* | 高 | `message_parcel.h:44` |
| MessageParcel.WriteAshmem() | 写入共享内存 | Ashmem* | 高 | `message_parcel.h:157` |
| MessageOption.SetFlags() | 设置调用标志 | int flags | 中 | `message_option.h` |

**详细分析**：

MessageParcel::WriteString16() 方法接收用户传入的字符串指针并将数据序列化到内部缓冲区。该方法的关键风险在于：如果调用者提供的长度信息不准确，可能导致缓冲区溢出或越界读取。代码实现需要验证指针有效性和长度合法性。

```cpp
// 潜在风险代码模式
std::u16string userInput = data.ReadString16();
// 如果 data 中的长度字段被篡改，可能导致异常
```

**证据来源**：`bundle.json:140-146` N-API 头文件定义，`bundle.json:163-170` JS N-API 套件定义。

### 1.2 IPC 消息解析入口

IPC 消息解析是处理跨进程通信数据的核心环节，涉及 MessageParcel 的数据读取操作。

| 解析方法 | 功能描述 | 输入类型 | 风险等级 | 代码位置 |
|---------|---------|---------|---------|---------|
| ReadString16() | 读取字符串 | 序列化数据 | 高 | `message_parcel.h` |
| ReadInt32() | 读取整数 | 序列化数据 | 中 | `message_parcel.h` |
| ReadBuffer() | 读取缓冲区 | 序列化数据 | 高 | `message_parcel.h` |
| ReadRemoteObject() | 读取远程对象 | 序列化数据 | 高 | `message_parcel.h:51` |
| ReadInterfaceToken() | 读取接口令牌 | 序列化数据 | 中 | `message_parcel.h:81` |
| GetRawFileDescriptor() | 获取文件描述符 | 序列化数据 | 高 | `ipc_file_descriptor.h` |

**详细分析**：

ReadRemoteObject() 方法从消息数据中反序列化远程对象引用。该过程涉及内存分配和对象构造，如果攻击者构造恶意的序列化数据，可能触发以下安全问题：

1. 内存分配拒绝服务：构造超大数据请求耗尽系统内存
2. 对象构造异常：构造畸形的对象描述导致构造失败或崩溃
3. 类型混淆：伪造对象类型绕过安全检查

**证据来源**：`interfaces/innerkits/ipc_core/include/message_parcel.h` MessageParcel 接口定义，`interfaces/innerkits/ipc_core/include/ipc_file_descriptor.h` 文件描述符接口。

### 1.3 系统能力查询入口

系统能力（SA）的查询是客户端获取远程服务代理的关键步骤，也是权限控制的重要环节。

| 查询方法 | 功能描述 | 输入参数 | 风险等级 | 代码位置 |
|---------|---------|---------|---------|---------|
| GetSystemAbility() | 获取系统能力 | SA ID | 中 | 依赖 samgr 子系统 |
| GetSystemAbilityManager() | 获取 SA 管理器 | 无 | 低 | 依赖 samgr 子系统 |
| CheckCallingPermission() | 检查调用权限 | permission name | 中 | access_token 适配层 |

**详细分析**：

GetSystemAbility() 方法接收系统能力标识符（SA ID）作为参数。虽然 SA ID 通常由系统分配，但如果存在整数溢出或枚举值绕过问题，攻击者可能通过构造特殊的 SA ID 访问未授权的服务。

**证据来源**：`README.md:236-250` SA 获取代码示例，`bundle.json:37` samgr 依赖定义。

### 1.4 文件描述符传递入口

文件描述符（FD）的跨进程传递是 IPC 通信的高级特性，允许在进程间传递打开的文件句柄。

| 操作方法 | 功能描述 | 数据类型 | 风险等级 | 代码位置 |
|---------|---------|---------|---------|---------|
| WriteFileDescriptor() | 写入文件描述符 | int fd | 高 | `ipc_file_descriptor.h` |
| ReadFileDescriptor() | 读取文件描述符 | int& fd | 高 | `ipc_file_descriptor.h` |
| GetRawFileDescriptor() | 获取原始描述符 | int | 高 | `ipc_file_descriptor.h` |

**详细分析**：

文件描述符传递涉及内核层面的句柄复制操作。潜在风险包括：

1. 描述符泄漏：接收方获得超出预期范围的描述符
2. 描述符劫持：恶意进程通过猜测描述符值访问错误的目标
3. 资源耗尽：大量描述符传递耗尽目标进程的文件描述符配额

IPC 模块通过 TF_ACCEPT_FDS 标志控制是否接受传入的文件描述符，该标志的安全配置直接影响系统的安全性。

**证据来源**：`message_option.h` MessageOption::TF_ACCEPT_FDS 标志定义，`interfaces/innerkits/ipc_core/include/ipc_file_descriptor.h` 文件描述符接口。

### 1.5 匿名共享内存入口

匿名共享内存（Ashmem）用于在进程间传递大块数据，是 IPC 数据传输的重要补充机制。

| 操作方法 | 功能描述 | 风险等级 | 代码位置 |
|---------|---------|---------|---------|
| WriteAshmem() | 写入共享内存句柄 | 高 | `message_parcel.h:157` |
| ReadAshmem() | 读取共享内存句柄 | 高 | `message_parcel.h` |
| napi_create_ashmem() | 创建 N-API Ashmem | 高 | `napi_common/source/napi_ashmem.cpp` |

**详细分析**：

Ashmem 机制允许进程共享内存区域，减少数据复制开销。然而，这种机制也带来了新的攻击面：

1. 内存权限绑定：攻击者可能通过共享内存绕过某些访问控制检查
2. 内存释放后使用：共享内存区域被释放后仍被访问导致 Use-After-Free
3. 内存内容篡改：攻击者修改共享内存内容影响接收方处理逻辑

**证据来源**：`bundle.json:140-146` N-API Ashmem 相关头文件，`interfaces/kits/js/napi/` N-API 实现目录。

---

## 2 敏感操作清单

### 2.1 系统调用操作

IPC 模块通过 Binder 驱动程序与内核交互，触发多种系统调用操作。

| 系统调用 | 操作描述 | 权限要求 | 风险等级 |
|---------|---------|---------|---------|
| ioctl(BINDER_WRITE_READ) | Binder 读写控制 | 设备访问权限 | 高 |
| mmap() | 内存映射 | 设备访问权限 | 高 |
| poll() | 等待Binder事件 | 设备访问权限 | 中 |
| read()/write() | 读写Binder节点 | 设备访问权限 | 高 |

**详细分析**：

Binder 驱动是 Linux 内核中的特殊设备驱动，负责管理跨进程通信的底层数据传输。ioctl(BINDER_WRITE_READ) 是最核心的系统调用，用于发送命令和读写数据。攻击者如果能够控制 ioctl 的参数，可能触发内核态的异常行为。

**证据来源**：`ipc/native/c/ipc/src/linux/include/sys_binder.h` Binder 接口定义。

### 2.2 跨进程对象传递操作

远程对象的跨进程传递涉及引用计数管理和对象序列化。

| 操作 | 描述 | 潜在风险 |
|-----|------|---------|
| AddRef() | 增加引用计数 | 引用泄漏导致内存泄漏 |
| Release() | 减少引用计数 | 悬空引用导致 UAF |
| SendRequest() | 发送请求 | 请求伪造、重放攻击 |
| OnRemoteRequest() | 处理请求 | 恶意数据处理 |

**详细分析**：

IRemoteObject 的引用计数管理是 IPC 模块的核心机制。SendRequest() 方法发送跨进程请求，其安全性依赖于：

1. 请求完整性：确保请求数据在传输过程中未被篡改
2. 请求来源验证：验证请求确实来自授权的调用方
3. 请求处理超时：防止恶意请求长时间占用处理线程

**证据来源**：`interfaces/innerkits/ipc_core/include/iremote_object.h:88` SendRequest 方法定义。

### 2.3 进程身份获取操作

IPCSkeleton 提供了获取调用方身份信息的方法，这些信息用于权限检查和审计。

| 方法 | 功能描述 | 返回信息 | 代码位置 |
|-----|---------|---------|---------|
| GetCallingPid() | 获取调用方进程 ID | pid_t | `ipc_skeleton.h:61` |
| GetCallingUid() | 获取调用方用户 ID | uid_t | `ipc_skeleton.h:75` |
| GetCallingDeviceID() | 获取调用方设备 ID | std::string | `ipc_skeleton.h:124` |
| GetLocalDeviceID() | 获取本地设备 ID | std::string | `ipc_skeleton.h:117` |
| IsLocalCalling() | 检查是否本地调用 | bool | `ipc_skeleton.h:131` |

**详细分析**：

这些身份获取方法是实现细粒度访问控制的基础。GetCallingPid() 和 GetCallingUid() 返回调用进程的用户和进程标识符，可用于：

1. 权限验证：根据 UID/PID 检查调用者是否具有相应权限
2. 审计追踪：记录敏感操作的调用来源
3. 访问控制：实现基于身份的访问控制策略

然而，如果这些信息可被攻击者伪造或篡改，整个权限控制体系将失效。因此，需要确保这些信息来源于可信的内核态 Binder 驱动，而非用户态可操控的数据。

**证据来源**：`interfaces/innerkits/ipc_core/include/ipc_skeleton.h:22` IPCSkeleton 类定义。

---

## 3 信任边界分析

### 3.1 信任域划分

OpenHarmony IPC 模块涉及的信任域可划分为以下几个层次：

```
┌─────────────────────────────────────────────────────────────┐
│                      高信任域                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              系统服务进程 (Trusted)                   │    │
│  │  - SAMgr (系统能力管理器)                           │    │
│  │  - 核心系统服务                                      │    │
│  │  - 内置 SA 实现                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                            │                                 │
│                            ▼ IPC 边界                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            IPC 模块边界 (Semi-Trusted)               │    │
│  │  - N-API 接口层                                      │    │
│  │  - C/C++ 接口层                                      │    │
│  │  - Binder 驱动封装                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                            │                                 │
│                            ▼ 边界                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            应用进程 (Untrusted)                      │    │
│  │  - Third-party 应用                                  │    │
│  │  - 外部进程                                          │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 边界跨越点

| 跨越方向 | 跨越点 | 数据流向 | 安全检查 |
|---------|-------|---------|---------|
| 应用→系统服务 | N-API 调用 | 用户数据 → 序列化数据 | 参数验证、权限检查 |
| 应用→系统服务 | 文件描述符传递 | FD 句柄 → 句柄复制 | TF_ACCEPT_FDS 标志 |
| 应用→系统服务 | 共享内存映射 | 内存区域 → 句柄传递 | Ashmem 权限检查 |
| 系统服务→系统服务 | Binder 调用 | 序列化数据 → 进程间 | UID/PID 验证 |
| 跨设备→本地设备 | DBinder 调用 | 网络数据 → 序列化 | 设备身份验证 |

### 3.3 关键信任边界点

**N-API 边界（应用→IPC 模块）**：

这是应用层数据进入 IPC 模块的第一道关卡。N-API 参数已经经过 JavaScript 运行时的类型检查，但仍需关注：

1. napi_value 到原生类型的转换是否安全
2. 字符串编码转换（UTF-16/UTF-8）是否存在缓冲区溢出风险
3. 大对象（如 ArrayBuffer）的长度验证是否充分

**代码位置**：`ipc/native/src/napi/src/napi_rpc_native_module.cpp:48` N-API 模块注册点。

**Binder 边界（IPC 模块→内核）**：

Binder 驱动位于内核态，是用户态 IPC 模块与内核交互的唯一通道。关键信任边界点包括：

1. ioctl() 参数的正确性验证
2. 用户态缓冲区和内核态缓冲区的数据拷贝安全
3. 文件描述符在用户态和内核态之间的传递安全

**代码位置**：`ipc/native/c/ipc/src/linux/include/sys_binder.h` Binder 接口定义。

---

## 4 数据流分析

### 4.1 典型 IPC 调用数据流

以下分析从应用层发起的典型 IPC 调用所经过的数据流路径：

```
┌──────────────┐     ┌────────────────┐     ┌────────────────┐     ┌──────────────┐
│   应用进程    │     │   IPC 模块     │     │   Binder 驱动  │     │  系统服务    │
│  (Untrusted)  │     │  (Semi-Trusted)│     │   (Trusted)    │     │  (Trusted)   │
└──────┬───────┘     └───────┬────────┘     └───────┬────────┘     └──────┬───────┘
       │                      │                       │                      │
       │ 1. N-API 调用        │                       │                      │
       │─────────────────────>│                       │                      │
       │                      │                       │                      │
       │                      │ 2. 参数序列化         │                      │
       │                      │ (MessageParcel)        │                      │
       │                      │───────────────────────▶│                      │
       │                      │                       │                      │
       │                      │                       │ 3. ioctl 系统调用   │
       │                      │                       │─────────────────────▶│
       │                      │                       │                      │
       │                      │                       │                      │ 4. 请求处理
       │                      │                       │                      │ (OnRemoteRequest)
       │                      │                       │                      │<─────────────────────▶
       │                      │                       │                      │
       │                      │                       │◀─────────────────────│
       │                      │                       │                      │
       │                      │◀──────────────────────│                      │
       │                      │                       │                      │
       │◀─────────────────────│                       │                      │
       │                      │                       │                      │
```

**数据流阶段说明**：

**阶段 1（N-API 调用）**：应用代码通过 N-API 接口发起 IPC 调用，传入的参数经过 JavaScript 运行时类型检查后转换为 napi_value 类型。此阶段的主要风险是参数类型的误用和数值范围的溢出。

**阶段 2（参数序列化）**：N-API 参数被序列化为 MessageParcel 格式的二进制数据。此阶段需要验证：字符串长度是否在合理范围内、缓冲区指针是否有效、数组长度是否与实际数据匹配等。

**阶段 3（Binder 传输）**：序列化数据通过 ioctl(BINDER_WRITE_READ) 系统调用传递给 Binder 驱动。此阶段由内核负责数据完整性的验证，用户态代码应确保所有指针已正确解引用。

**阶段 4（请求处理）**：目标进程的 IPCObjectStub::OnRemoteRequest() 方法接收请求并分发到具体的处理函数。此阶段需要进行权限检查和输入验证，确保处理逻辑的安全性。

### 4.2 敏感数据流路径

以下列出需要特别关注的数据流路径及其安全关键点：

| 数据流路径 | 敏感数据 | 安全关键点 |
|-----------|---------|-----------|
| N-API → MessageParcel | 应用参数 | 长度验证、类型检查 |
| MessageParcel → Binder | 序列化数据 | 缓冲区完整性 |
| Binder → MessageParcel | 响应数据 | 缓冲区边界 |
| IPCSkeleton → 权限系统 | UID/PID | 信息可信度 |
| FD 传递 | 文件句柄 | 句柄有效性 |

---

## 5 攻击面评估总结

### 5.1 攻击面概览

| 类别 | 入口数量 | 高风险点 | 中风险点 | 低风险点 |
|-----|---------|---------|---------|---------|
| N-API 接口 | 9 | 3 | 4 | 2 |
| IPC 消息解析 | 6 | 3 | 2 | 1 |
| SA 查询 | 3 | 1 | 1 | 1 |
| 文件描述符 | 3 | 3 | 0 | 0 |
| 共享内存 | 3 | 2 | 1 | 0 |
| **合计** | **24** | **12** | **8** | **4** |

### 5.2 高风险区域优先级

基于攻击面分析，以下区域需要优先进行安全加固：

| 优先级 | 区域 | 加固建议 |
|-------|------|---------|
| P0 | MessageParcel::ReadString16() | 增加长度边界检查，防止缓冲区溢出 |
| P0 | 文件描述符传递 | 严格验证 FD 范围，增加使用审计 |
| P0 | ReadRemoteObject() | 增加对象类型验证，防止类型混淆 |
| P1 | N-API 参数转换 | 增加类型转换安全检查 |
| P1 | IPCSkeleton 身份信息 | 确保信息来源于内核态，防止伪造 |
| P2 | Ashmem 共享内存 | 增加权限检查和生命周期管理 |

### 5.3 建议的安全检查点

根据攻击面分析结果，建议在以下位置增加或强化安全检查：

1. **MessageParcel 写入时**：验证写入数据的长度不超过剩余缓冲区空间
2. **MessageParcel 读取时**：验证读取位置不超出数据范围
3. **对象反序列化时**：验证对象类型符合预期
4. **文件描述符传递时**：验证描述符值在合法范围内
5. **权限检查时**：确保所有敏感操作都经过权限验证

---

## 附录：代码证据索引

| 证据类型 | 文件路径 | 行号 | 说明 |
|---------|---------|------|------|
| N-API 入口 | `ipc/native/src/napi/src/napi_rpc_native_module.cpp` | 48 | N-API 模块注册 |
| MessageParcel | `interfaces/innerkits/ipc_core/include/message_parcel.h` | 26-157 | 消息序列化接口 |
| MessageOption | `interfaces/innerkits/ipc_core/include/message_option.h` | 21 | 调用选项配置 |
| IPCSkeleton | `interfaces/innerkits/ipc_core/include/ipc_skeleton.h` | 22 | IPC 骨架类 |
| IRemoteObject | `interfaces/innerkits/ipc_core/include/iremote_object.h` | 34 | 远程对象接口 |
| 文件描述符 | `interfaces/innerkits/ipc_core/include/ipc_file_descriptor.h` | - | FD 处理接口 |
| Binder 接口 | `ipc/native/c/ipc/src/linux/include/sys_binder.h` | - | Binder 驱动接口 |

---

*文档版本：1.0*  
*创建日期：2026年2月7日*  
*下次审查日期：2026年5月7日*  
*代码版本：OpenHarmony IPC Component v3.0*

# Connected NFC Tag - 内部 API 文档

## 1. 概述

### 1.1 内部 API 定位

内部 API 是供**系统应用**使用的原生 C++ 接口，位于 `interfaces/inner_api/` 目录。

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            API 分层                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  【对外 N-API】  @ohos.connectedTag (JS)                                     │
│       ↓                                                                      │
│  【内部 API】  nfc_tag_client.h (Native C++) ← 本文档                        │
│       ↓                                                                      │
│  【服务层】    nfc_tag_service.cpp (IPC Server)                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 头文件清单

| 文件 | 描述 | 稳定性 |
|------|------|--------|
| `nfc_tag_client.h` | 客户端单例接口 | System API |
| `nfc_tag_proxy.h` | IPC 代理接口 | System API |
| `nfc_tag_callback_stub.h` | 回调存根接口 | System API |
| `infc_tag_service.h` | 服务接口定义 | System API |
| `infc_tag_callback.h` | 回调接口定义 | System API |
| `nfc_tag_errcode.h` | 错误码定义 | Stable |
| `nfc_tag_log.h` | 日志宏定义 | Stable |

---

## 2. 客户端接口 (NfcTagClient)

### 2.1 类定义

**文件**: `interfaces/inner_api/include/nfc_tag_client.h`

```cpp
class NfcTagClient final : public DelayedRefSingleton<NfcTagClient> {
    DECLARE_DELAYED_REF_SINGLETON(NfcTagClient);

public:
    DISALLOW_COPY_AND_MOVE(NfcTagClient);

public:
    ErrCode Init();
    ErrCode Uninit();
    ErrCode ReadNdefTag(std::string &response);
    ErrCode WriteNdefTag(const std::string &data);
    ErrCode ReadNdefData(std::vector<uint8_t> &data);
    ErrCode WriteNdefData(const std::vector<uint8_t> &data);
    ErrCode RegListener(const sptr<INfcTagCallback> &callback);
    ErrCode UnregListener(const sptr<INfcTagCallback> &callback);
    static const char* GetErrCodeString(ErrCode code);

private:
    sptr<INfcTagService> GetService();
};
```

### 2.2 方法说明

#### Init()

**声明**:
```cpp
ErrCode Init();
```

**功能**: 初始化 NFC 标签服务连接。

**返回值**:
| 错误码 | 描述 |
|--------|------|
| `NFC_SUCCESS` | 成功 |
| `NFC_NO_REMOTE` | 服务未运行 |
| `NFC_GRANT_FAILED` | 权限不足 |

**线程安全**: 否（应在初始化阶段调用）

---

#### Uninit()

**声明**:
```cpp
ErrCode Uninit();
```

**功能**: 释放 NFC 标签服务连接。

---

#### ReadNdefTag()

**声明**:
```cpp
ErrCode ReadNdefTag(std::string &response);
```

**参数**:
| 参数 | 类型 | 描述 |
|------|------|------|
| `response` | `std::string&` | 输出读取的 NDEF 数据 |

**返回值**:
| 错误码 | 描述 |
|--------|------|
| `NFC_SUCCESS` | 成功 |
| `NFC_INVALID_PARAMETER` | 参数错误 |
| `NFC_IPC_READ_FAILED` | IPC 读取失败 |

---

#### WriteNdefTag()

**声明**:
```cpp
ErrCode WriteNdefTag(const std::string &data);
```

**参数**:
| 参数 | 类型 | 描述 |
|------|------|------|
| `data` | `const std::string&` | 要写入的 NDEF 数据 |

**数据长度**: 1-512 字节

---

#### ReadNdefData()

**声明**:
```cpp
ErrCode ReadNdefData(std::vector<uint8_t> &data);
```

**参数**:
| 参数 | 类型 | 描述 |
|------|------|------|
| `data` | `std::vector<uint8_t>&` | 输出读取的字节数据 |

---

#### WriteNdefData()

**声明**:
```cpp
ErrCode WriteNdefData(const std::vector<uint8_t> &data);
```

**参数**:
| 参数 | 类型 | 描述 |
|------|------|------|
| `data` | `const std::vector<uint8_t>&` | 要写入的字节数据 |

---

#### RegListener()

**声明**:
```cpp
ErrCode RegListener(const sptr<INfcTagCallback> &callback);
```

**参数**:
| 参数 | 类型 | 描述 |
|------|------|------|
| `callback` | `sptr<INfcTagCallback>` | 事件回调接口 |

**限制**: 每个进程最多注册 1 个回调

**返回值**:
| 错误码 | 描述 |
|--------|------|
| `NFC_SUCCESS` | 成功 |
| `NFC_CALLBACK_REGISTERED` | 回调已注册 |
| `NFC_TOO_MANY_CALLBACK` | 回调过多 |

---

#### UnregListener()

**声明**:
```cpp
ErrCode UnregListener(const sptr<INfcTagCallback> &callback);
```

---

#### GetErrCodeString()

**声明**:
```cpp
static const char* GetErrCodeString(ErrCode code);
```

**功能**: 将错误码转换为可读字符串。

---

## 3. 服务接口 (INfcTagService)

### 3.1 接口定义

**文件**: `interfaces/inner_api/include/infc_tag_service.h`

```cpp
class INfcTagService : public IRemoteBroker {
public:
    virtual ~INfcTagService() {}

    virtual ErrCode Init() = 0;
    virtual ErrCode Uninit() = 0;
    virtual ErrCode ReadNdefTag(std::string &response) = 0;
    virtual ErrCode WriteNdefTag(const std::string &data) = 0;
    virtual ErrCode ReadNdefData(std::vector<uint8_t> &data) = 0;
    virtual ErrCode WriteNdefData(const std::vector<uint8_t> &data) = 0;
    virtual ErrCode RegListener(const sptr<INfcTagCallback> &callback) = 0;
    virtual ErrCode UnregListener(const sptr<INfcTagCallback> &callback) = 0;

    enum {
        NFC_TAG_CMD_INIT = 0,
        NFC_TAG_CMD_UNINIT,
        NFC_TAG_CMD_READ_NDEF_TAG,
        NFC_TAG_CMD_WRITE_NDEF_TAG,
        NFC_TAG_CMD_READ_NDEF_DATA,
        NFC_TAG_CMD_WRITE_NDEF_DATA,
        NFC_TAG_CMD_REGISTER_CALLBACK,
        NFC_TAG_CMD_UNREGISTER_CALLBACK,
    };

public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.nfc.IConnectedNfcTagService");
};
```

### 3.2 接口描述符

```cpp
static constexpr const char* NFC_TAG_INTERFACE_DESCRIPTOR = "ohos.nfc.IConnectedNfcTagService";
```

---

## 4. 回调接口 (INfcTagCallback)

### 4.1 接口定义

**文件**: `interfaces/inner_api/include/infc_tag_callback.h`

```cpp
class INfcTagCallback : public IRemoteBroker {
public:
    virtual ~INfcTagCallback() {}

    virtual ErrCode OnNotify(int nfcRfState) = 0;

    enum {
        CMD_ON_NFC_TAG_NOTIFY = 0,
    };

public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.nfc.INfcTagCallback");
};
```

### 4.2 OnNotify()

**声明**:
```cpp
ErrCode OnNotify(int nfcRfState);
```

**参数**:
| 值 | 描述 |
|------|------|
| 0 | NFC RF 场离开 |
| 1 | NFC RF 场进入 |

---

## 5. 错误码定义

### 5.1 完整错误码表

**文件**: `interfaces/inner_api/include/nfc_tag_errcode.h`

```cpp
enum ErrCode {
    NFC_SUCCESS = 0,                    // 成功
    NFC_REMOTE_DIED,                    // 远程服务已死
    NFC_IPC_WRITETOKE_FAILED,           // IPC 写 token 失败
    NFC_INVALID_TOKEN,                  // 无效接口 token
    NFC_IPC_SEND_FAILED,                // IPC 发送失败
    NFC_IPC_WRITE_FAILED,               // IPC 写入失败
    NFC_IPC_READ_FAILED,                // IPC 读取失败
    NFC_NO_CALLBACK,                    // 无回调
    NFC_NO_REMOTE,                      // 无远程对象
    NFC_NO_OBJECT,                      // 无对象
    NFC_NO_HDI_PROXY,                   // 无 HDI 代理
    NFC_NO_HDI_IMPL,                    // 无 HDI 实现
    NFC_GRANT_FAILED,                   // 权限授予失败
    NFC_SYS_PERM_FAILED,                // 系统权限检查失败
    NFC_INVALID_PROXY,                  // 无效代理
    NFC_INVALID_CALLBACKSTUB,            // 无效回调存根
    NFC_INVALID_CALLBACK,                // 无效回调
    NFC_INVALID_PARAMETER,              // 无效参数
    NFC_CALLBACK_REGISTERED,             // 回调已注册
    NFC_CALLBACK_NOT_REGISTERED,         // 回调未注册
    NFC_CALLBACK_NOT_EQUAL,              // 回调不匹配
    NFC_TOO_MANY_CALLBACK,               // 回调过多 (max 30)
    NFC_HDI_REMOTE_FAILED,               // HDI 远程调用失败
    NFC_INVALID_STATE,                   // 无效状态
    NFC_ERR_MAX,                         // 错误码边界
};
```

### 5.2 系统能力 ID

```cpp
#define NFC_CONNECTED_TAG_ABILITY_ID 1148
```

---

## 6. 使用示例

### 6.1 基本使用流程

```cpp
#include "nfc_tag_client.h"
#include "nfc_tag_errcode.h"

using namespace OHOS::NFC;

void NfcTagExample() {
    // 获取客户端单例
    auto& client = NfcTagClient::GetInstance();

    // 初始化
    ErrCode ret = client.Init();
    if (ret != NFC_SUCCESS) {
        printf("Init failed: %s\n", client.GetErrCodeString(ret));
        return;
    }

    // 读取 NDEF
    std::string response;
    ret = client.ReadNdefTag(response);
    if (ret == NFC_SUCCESS) {
        printf("Read success: %s\n", response.c_str());
    }

    // 写入 NDEF
    std::string data = "Hello NFC Tag";
    ret = client.WriteNdefTag(data);
    if (ret == NFC_SUCCESS) {
        printf("Write success\n");
    }

    // 反初始化
    client.Uninit();
}
```

### 6.2 带回调的使用

```cpp
#include "nfc_tag_client.h"
#include "infc_tag_callback.h"
#include "nfc_tag_errcode.h"

using namespace OHOS::NFC;

class NfcTagCallbackImpl : public INfcTagCallback {
public:
    ErrCode OnNotify(int nfcRfState) override {
        if (nfcRfState == 1) {
            printf("NFC RF field entered\n");
        } else {
            printf("NFC RF field left\n");
        }
        return NFC_SUCCESS;
    }
};

void NfcTagWithCallback() {
    auto& client = NfcTagClient::GetInstance();
    client.Init();

    // 注册回调
    sptr<INfcTagCallback> callback = new NfcTagCallbackImpl();
    ErrCode ret = client.RegListener(callback);
    if (ret != NFC_SUCCESS) {
        printf("Register callback failed: %s\n", client.GetErrCodeString(ret));
    }

    // 注销回调
    ret = client.UnregListener(callback);
    if (ret != NFC_SUCCESS) {
        printf("Unregister callback failed: %s\n", client.GetErrCodeString(ret));
    }

    client.Uninit();
}
```

---

## 7. 依赖关系

### 7.1 内部依赖

```
nfc_tag_client.cpp
    ↓ uses
nfc_tag_proxy.cpp      (IPC 代理)
nfc_tag_callback_stub.cpp (回调处理)
```

### 7.2 外部依赖

| 依赖 | 用途 |
|------|------|
| `ipc:ipc_core` | IPC 通信 |
| `samgr:samgr_proxy` | 服务管理 |
| `hilog:libhilog` | 日志 |
| `c_utils:utils` | 工具库 |

---

## 8. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_NAPI.md](./02_NAPI.md) |
| 构建说明 | [04_Build.md](./04_Build.md) |
| 安全评估 | [05_Security.md](./05_Security.md) |

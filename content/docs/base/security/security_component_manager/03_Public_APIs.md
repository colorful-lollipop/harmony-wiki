# 对外 API 参考 - Security Component Manager

> 目的：提供 C++ Native SDK API 完整参考

---

## 适用范围

本文档适用于：
- 需要使用安全组件功能的 C++ 应用开发者
- 需要集成增强框架的厂商
- 需要验证权限的应用开发者

---

## 关键结论

1. **本项目无 N-API 绑定**，仅提供 C++ Native SDK
2. **ArkTS 应用通过 Ace Engine 组件调用**，不直接使用本 API
3. **主要 API 类**：`SecCompKit`（主 Kit）、`SecCompEnhanceKit`（增强 Kit）
4. **错误码范围**：服务错误（-50 ~ -65）、增强错误（-100 ~ -111）

---

## SecCompKit - 主 API 类

### 类定义

```cpp
namespace OHOS {
namespace Security {
namespace SecurityComponent {
class __attribute__((visibility("default"))) SecCompKit {
public:
    static int32_t RegisterSecurityComponent(SecCompType type, std::string& componentInfo, int32_t& scId);
    static int32_t UpdateSecurityComponent(int32_t scId, std::string& componentInfo);
    static int32_t UnregisterSecurityComponent(int32_t scId);
    static int32_t ReportSecurityComponentClickEvent(SecCompInfo& SecCompInfo, sptr<IRemoteObject> callerToken,
        OnFirstUseDialogCloseFunc&& callback, std::string& message);
    static bool VerifySavePermission(AccessToken::AccessTokenID tokenId);
    static int32_t PreRegisterSecCompProcess();
    static bool IsServiceExist();
    static bool LoadService();
    static bool IsSystemAppCalling();
    static bool HasCustomPermissionForSecComp();
};
}  // namespace SecurityComponent
}  // namespace Security
}  // namespace OHOS
```

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_kit.h:28-41`

### API 方法表

| 方法 | 功能 | 参数 | 返回值 | 错误码 | 前置条件 |
|------|------|------|--------|----------|
| `RegisterSecurityComponent` | 注册安全组件 | `type`: 组件类型<br/>`componentInfo`: JSON 字符串<br/>`scId`: 输出组件 ID | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -50: 参数无效<br/>-56: 组件信息无效<br/>-57: 组件重叠<br/>-62: 调用者无效 | 组件信息 JSON 格式正确 |
| `UpdateSecurityComponent` | 更新组件信息 | `scId`: 组件 ID<br/>`componentInfo`: 新 JSON 字符串 | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -50: 参数无效<br/>-56: 组件信息无效<br/>-58: 组件不存在<br/>-61: 信息不匹配 | 组件已注册 |
| `UnregisterSecurityComponent` | 注销组件 | `scId`: 组件 ID | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -50: 参数无效<br/>-58: 组件不存在 | 组件已注册 |
| `ReportSecurityComponentClickEvent` | 报告点击事件，申请临时权限 | `SecCompInfo`: 组件信息和点击事件<br/>`callerToken`: 调用者 token<br/>`callback`: 对话框回调<br/>`message`: 输出消息 | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -50: 参数无效<br/>-55: 服务不存在<br/>-60: 点击事件无效<br/>-63: 等待对话框关闭 | 组件已注册，点击事件有效 |
| `VerifySavePermission` | 验证保存权限 | `tokenId`: 访问令牌 ID | `bool`<br/>true: 有权限<br/>false: 无权限 | 无 | token ID 有效 |
| `PreRegisterSecCompProcess` | 预注册进程 | 无 | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -55: 服务不存在 | 服务已启动 |
| `IsServiceExist` | 检查服务是否存在 | 无 | `bool`<br/>true: 存在<br/>false: 不存在 | 无 | 无 |
| `LoadService` | 按需加载服务 | 无 | `bool`<br/>true: 成功<br/>false: 失败 | 无 | 无 |
| `IsSystemAppCalling` | 检查是否为系统应用调用 | 无 | `bool`<br/>true: 系统应用<br/>false: 非系统应用 | 无 | 无 |
| `HasCustomPermissionForSecComp` | 检查是否有安全组件自定义权限 | 无 | `bool`<br/>true: 有权限<br/>false: 无权限 | 无 | 无 |

---

## SecCompEnhanceKit - 增强 Kit 类

### 类定义

```cpp
namespace OHOS {
namespace Security {
namespace SecurityComponent {
struct __attribute__((visibility("default"))) SecCompEnhanceKit {
    static void InitClientEnhance();
    static int32_t SetEnhanceCfg(uint8_t* cfg, uint32_t cfgLen);
    static int32_t GetPointerEventEnhanceData(void* data, uint32_t dataLen,
        uint8_t* enhanceData, uint32_t& enHancedataLen);
};
}  // namespace SecurityComponent
}  // namespace Security
}  // namespace OHOS
```

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_enhance_kit.h:23-28`

### API 方法表

| 方法 | 功能 | 参数 | 返回值 | 错误码 | 用途 |
|------|------|------|--------|------|
| `InitClientEnhance` | 初始化客户端增强 | 无 | `void` | 无 | 应用启动时初始化增强框架 |
| `SetEnhanceCfg` | 设置增强配置 | `cfg`: 配置数据<br/>`cfgLen`: 数据长度 | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -100: 无增强库<br/>-101: 参数无效<br/>-102: 操作失败 | 厂商配置安全参数 |
| `GetPointerEventEnhanceData` | 获取点击事件增强数据 | `data`: 原始输入数据<br/>`dataLen`: 数据长度<br/>`enhanceData`: 输出增强数据<br/>`enHancedataLen`: 输出长度 | `int32_t`<br/>成功: 0<br/>失败: 错误码 | -100: 无增强库<br/>-101: 参数无效<br/>-102: 操作失败 | 获取 HMAC、Challenge 值等增强数据 |

---

## 数据结构

### SecCompType - 组件类型

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_info.h:61-67`

```cpp
enum SecCompType {
    UNKNOWN_SC_TYPE = 0,
    LOCATION_COMPONENT,      // 位置按钮
    PASTE_COMPONENT,        // 粘贴按钮
    SAVE_COMPONENT,          // 保存按钮
    MAX_SC_TYPE
};
```

### SecCompClickEvent - 点击事件

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_info.h:188-196`

```cpp
enum class ClickEventType : int32_t {
    UNKNOWN_EVENT_TYPE,
    POINT_EVENT_TYPE,           // 触摸/点击事件
    KEY_EVENT_TYPE,             // 键盘事件
    ACCESSIBILITY_EVENT_TYPE      // 无障碍事件
};

struct SecCompPointEvent {
    double touchX;
    double touchY;
    uint64_t timestamp;
};

struct SecCompKeyEvent {
    uint64_t timestamp;
    int32_t keyCode;
};

struct SecCompAccessibilityEvent {
    int64_t timestamp = 0;
    int64_t componentId = 0;
};

struct SecCompClickEvent {
    ClickEventType type;
    union {
        SecCompPointEvent point;
        SecCompKeyEvent key;
        SecCompAccessibilityEvent accessibility;
    };
    ExtraInfo extraInfo;  // 增强数据（HMAC、Challenge 等）
};
```

### SecCompInfo - 组件信息

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_info.h:198-202`

```cpp
struct SecCompInfo {
    int32_t scId;                        // 组件 ID
    std::string componentInfo;             // JSON 格式的组件信息
    SecCompClickEvent clickInfo;           // 点击事件
};
```

### ExtraInfo - 增强数据

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_info.h:160-163`

```cpp
struct ExtraInfo {
    uint32_t dataSize;  // 数据长度
    uint8_t* data;      // 数据指针（HMAC、Challenge 等）
};
```

---

## 错误码

### 服务错误（-50 ~ -65）

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_err.h:23-39`

| 错误码 | 名称 | 描述 | 触发条件 |
|--------|------|------|----------|
| 0 | `SC_OK` | 成功 | 操作成功 |
| -50 | `SC_SERVICE_ERROR_VALUE_INVALID` | 参数无效 | 输入参数为空或超出范围 |
| -51 | `SC_SERVICE_ERROR_PARCEL_OPERATE_FAIL` | 序列化失败 | MessageParcel 操作失败 |
| -52 | `SC_SERVICE_ERROR_MEMORY_OPERATE_FAIL` | 内存操作失败 | 内存分配或拷贝失败 |
| -54 | `SC_SERVICE_ERROR_IPC_REQUEST_FAIL` | IPC 请求失败 | 与服务端通信失败 |
| -55 | `SC_SERVICE_ERROR_SERVICE_NOT_EXIST` | 服务不存在 | Security Component Service 未启动 |
| -56 | `SC_SERVICE_ERROR_COMPONENT_INFO_INVALID` | 组件信息无效 | JSON 解析失败或校验失败 |
| -57 | `SC_SERVICE_ERROR_COMPONENT_RECT_OVERLAP` | 组件重叠 | 组件与其他组件或窗口重叠 |
| -58 | `SC_SERVICE_ERROR_COMPONENT_NOT_EXIST` | 组件不存在 | scId 无效或组件已注销 |
| -59 | `SC_SERVICE_ERROR_PERMISSION_OPER_FAIL` | 权限操作失败 | 权限授予/撤销失败 |
| -60 | `SC_SERVICE_ERROR_CLICK_EVENT_INVALID` | 点击事件无效 | 点击验证失败（超出范围、时间戳过期等） |
| -61 | `SC_SERVICE_ERROR_COMPONENT_INFO_NOT_EQUAL` | 组件信息不匹配 | 更新时信息与注册时不一致 |
| -62 | `SC_SERVICE_ERROR_CALLER_INVALID` | 调用者无效 | Token 或 UID 校验失败 |
| -63 | `SC_SERVICE_ERROR_WAIT_FOR_DIALOG_CLOSE` | 等待对话框关闭 | 首次使用对话框未关闭 |
| -64 | `SC_SERVICE_ERROR_GRANT_CANCEL_FOR_DIALOG_CLOSE` | 对话框关闭时取消授予 | 用户在对话框中点击取消 |
| -65 | `SC_SERVICE_ERROR_START_FIRST_USE_DIALOG_FAILED` | 启动首次使用对话框失败 | Permission Manager 应用启动失败 |

### 增强错误（-100 ~ -111）

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_err.h:42-54`

| 错误码 | 名称 | 描述 | 触发条件 |
|--------|------|------|----------|
| -100 | `SC_ENHANCE_ERROR_NOT_EXIST_ENHANCE` | 无增强库 | 增强库未找到或加载失败 |
| -101 | `SC_ENHANCE_ERROR_VALUE_INVALID` | 增强参数无效 | 输入参数为空或超出范围 |
| -102 | `SC_ENHANCE_ERROR_OPER_FAIL` | 增强操作失败 | 增强库操作失败 |
| -103 | `SC_ENHANCE_ERROR_CALLBACK_REDIRECT` | 回调重定向 | 回调地址异常 |
| -104 | `SC_ENHANCE_ERROR_CALLBACK_REGIST_FAIL` | 回调注册失败 | 无法注册增强回调 |
| -105 | `SC_ENHANCE_ERROR_CALLBACK_HAS_EXIST` | 回调已存在 | 回调已注册 |
| -106 | `SC_ENHANCE_ERROR_CALLBACK_NOT_EXIST` | 回调不存在 | 回调未注册 |
| -107 | `SC_ENHANCE_ERROR_CALLBACK_OPER_FAIL` | 回调操作失败 | 回调执行失败 |
| -108 | `SC_ENHANCE_ERROR_CALLBACK_CHECK_FAIL` | 回调检查失败 | 回调验证失败 |
| -109 | `SC_ENHANCE_ERROR_IN_MALICIOUS_LIST` | 在恶意应用列表 | 应用在黑名单中 |
| -110 | `SC_ENHANCE_ERROR_CHALLENGE_CHECK_FAIL` | Challenge 检查失败 | Challenge 值验证失败 |
| -111 | `SC_ENHANCE_ERROR_CLICK_EXTRA_CHECK_FAIL` | 点击额外检查失败 | 增强数据验证失败 |

---

## 使用示例

### 注册安全组件

```cpp
#include "sec_comp_kit.h"
#include "sec_comp_info.h"

using namespace OHOS::Security::SecurityComponent;

// 准备组件信息（JSON 格式）
std::string componentInfo = R"({
    "type": 2,
    "text": "粘贴",
    "icon": "path/to/icon.png",
    "rect": {
        "x": 100,
        "y": 200,
        "width": 120,
        "height": 40
    }
})";

// 注册组件
int32_t scId;
int32_t ret = SecCompKit::RegisterSecurityComponent(
    PASTE_COMPONENT, componentInfo, scId);

if (ret == SC_OK) {
    // 注册成功，scId 返回组件 ID
} else {
    // 注册失败，检查错误码
}
```

### 报告点击事件

```cpp
#include "sec_comp_kit.h"
#include "sec_comp_info.h"

using namespace OHOS::Security::SecurityComponent;

// 准备点击事件
SecCompClickEvent clickInfo;
clickInfo.type = ClickEventType::POINT_EVENT_TYPE;
clickInfo.point.touchX = 150.0;
clickInfo.point.touchY = 220.0;
clickInfo.point.timestamp = GetTimestamp();

// 准备组件信息
SecCompInfo secCompInfo;
secCompInfo.scId = scId;
secCompInfo.componentInfo = componentInfo;
secCompInfo.clickInfo = clickInfo;

// 定义对话框回调
OnFirstUseDialogCloseFunc callback = [](int32_t result) {
    if (result == 0) {
        // 用户确认授权
    } else {
        // 用户拒绝授权
    }
};

// 报告点击事件
std::string message;
sptr<IRemoteObject> callerToken = GetCallerToken();
int32_t ret = SecCompKit::ReportSecurityComponentClickEvent(
    secCompInfo, callerToken, std::move(callback), message);

if (ret == SC_OK) {
    // 授予临时权限成功
} else {
    // 授予失败
}
```

### 验证保存权限

```cpp
#include "sec_comp_kit.h"
#include "accesstoken_kit.h"

using namespace OHOS::Security::SecurityComponent;
using namespace OHOS::Security::AccessToken;

// 获取当前应用的 token ID
AccessTokenID tokenId = GetSelfTokenID();

// 验证保存权限
bool hasPermission = SecCompKit::VerifySavePermission(tokenId);

if (hasPermission) {
    // 有保存权限，可以执行保存操作
} else {
    // 无权限，需要用户点击保存按钮
}
```

### 使用增强 Kit

```cpp
#include "sec_comp_enhance_kit.h"

using namespace OHOS::Security::SecurityComponent;

// 初始化客户端增强
SecCompEnhanceKit::InitClientEnhance();

// 设置增强配置
uint8_t cfg[] = {/* 配置数据 */};
int32_t ret = SecCompEnhanceKit::SetEnhanceCfg(cfg, sizeof(cfg));

// 获取点击事件增强数据
void* inputEventData = /* 获取输入事件数据 */;
uint32_t inputLen = /* 输入数据长度 */;
uint8_t enhanceData[256];
uint32_t enhanceLen = sizeof(enhanceData);

ret = SecCompEnhanceKit::GetPointerEventEnhanceData(
    inputEventData, inputLen, enhanceData, enhanceLen);

if (ret == SC_OK) {
    // 增强数据获取成功
} else {
    // 获取失败
}
```

---

## 从 ArkTS 调用说明

**重要**：本项目无 N-API 绑定，ArkTS 应用无法直接调用上述 C++ API。

### 正确使用方式

ArkTS 应用应使用 Ace Engine 提供的组件：

```typescript
// LocationButton
import { LocationButton } from '@kit.ArkUI'

LocationButton({
  icon: 'path/to/location_icon.png',
  text: '获取位置'
})
.onClick(() => {
  // 用户点击时，自动授予临时位置权限
  // 应用可以直接调用位置 API
})

// SaveButton
import { SaveButton } from '@kit.ArkUI'

SaveButton({
  icon: 'path/to/save_icon.png',
  text: '保存'
})
.onClick(() => {
  // 首次使用时显示确认对话框
  // 用户确认后授予临时保存权限
})

// PasteButton
import { PasteButton } from '@kit.ArkUI'

PasteButton({
  icon: 'path/to/paste_icon.png',
  text: '粘贴'
})
.onClick(() => {
  // 用户点击时，自动授予临时粘贴权限
  // 应用可以直接读取剪贴板
})
```

**底层流程**：
1. Ace Engine 的 ArkTS 组件调用底层 N-API 绑定
2. N-API 绑定通过 IPC 调用 `SecCompKit` C++ API
3. `SecCompKit` 通过 `SecCompClient` 与 Security Component Service 通信
4. Security Component Service 处理请求并授予临时权限

**相关仓库**：
- Ace Engine N-API 实现：`foundation/arkui/ace_engine`

---

## 性能敏感接口

以下接口在关键路径上调用，必须快速完成：

| 接口 | 调用时机 | 性能要求 | 原因 |
|------|----------|----------|------|
| `SecCompEnhanceKit::InitClientEnhance()` | 应用启动时 | 必须 < 100ms | 阻塞应用启动 |
| `SecCompKit::RegisterSecurityComponent()` | 组件注册时 | 必须 < 50ms | 阻塞 UI 渲染 |

**证据路径**：`AGENTS.md:Common_Issues` - Performance-Sensitive Interfaces

---

## 相关跳转

- [架构说明](./02_Architecture.md) - 理解 API 调用流程
- [内部 API](./04_Internal_APIs.md) - 查看服务端接口

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)

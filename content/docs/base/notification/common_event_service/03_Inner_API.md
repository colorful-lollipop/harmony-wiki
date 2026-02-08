# 03_内部 API

## Native 接口概述

Native API 主要面向 C/C++ 开发者，提供更底层的公共事件操作能力。

## 头文件清单

### 核心接口头文件

**位置**: `frameworks/core/include/`

| 头文件 | 说明 |
|--------|------|
| `common_event.h` | 公共事件基础类 |
| `common_event_data.h` | 公共事件数据 |
| `common_event_death_recipient.h` | 死亡接收者 |
| `common_event_listener.h` | 公共事件监听器 |
| `common_event_constant.h` | 常量定义 |

### Native 套件头文件

**位置**: `interfaces/inner_api/`

| 头文件 | 说明 |
|--------|------|
| `common_event_manager.h` | 公共事件管理器 |
| `common_event_subscriber.h` | 公共事件订阅者 |
| `common_event_data.h` | 公共事件数据 |
| `common_event_publish_info.h` | 发布信息 |
| `common_event_subscribe_info.h` | 订阅信息 |
| `common_event_support.h` | 支持类型定义 |
| `matching_skills.h` | 匹配技能 |
| `async_common_event_result.h` | 异步结果 |

## CommonEventManager

**头文件**: `interfaces/inner_api/common_event_manager.h`

### 发布公共事件

```cpp
// 发布标准公共事件
static bool PublishCommonEvent(const CommonEventData &data);

// 带发布信息发布
static bool PublishCommonEvent(
    const CommonEventData &data,
    const CommonEventPublishInfo &publishInfo);

// 带用户ID发布
static bool PublishCommonEventAsUser(
    const CommonEventData &data,
    const CommonEventPublishInfo &publishInfo,
    const int32_t &userId);

// 新版发布接口（返回错误码）
static int32_t NewPublishCommonEvent(
    const CommonEventData &data,
    const CommonEventPublishInfo &publishInfo);

static int32_t NewPublishCommonEventAsUser(
    const CommonEventData &data,
    const CommonEventPublishInfo &publishInfo,
    const int32_t &userId);
```

### 订阅公共事件

```cpp
// 标准订阅
static bool SubscribeCommonEvent(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);

// 支持订阅更新
static int32_t Subscribe(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);

// 异步订阅
static int32_t NewSubscribeCommonEvent(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);
```

### 取消订阅

```cpp
// 标准取消订阅
static bool UnSubscribeCommonEvent(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);

// 异步取消订阅
static int32_t NewUnSubscribeCommonEvent(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);

// 同步取消订阅
static int32_t NewUnSubscribeCommonEventSync(
    const std::shared_ptr<CommonEventSubscriber> &subscriber);
```

### 粘性事件

```cpp
// 获取粘性事件
static bool GetStickyCommonEvent(
    const std::string &event,
    CommonEventData &data);

// 移除粘性事件
static int32_t RemoveStickyCommonEvent(const std::string &event);
```

### 应用冻结

```cpp
// 冻结应用
static bool Freeze(const uid_t &uid);

// 解冻应用
static bool Unfreeze(const uid_t &uid);

// 解冻所有应用
static bool UnfreezeAll();
```

### 静态订阅者

```cpp
// 设置静态订阅者状态
static int32_t SetStaticSubscriberState(bool enable);
static int32_t SetStaticSubscriberState(
    const std::vector<std::string> &events,
    bool enable);
```

## CommonEventSubscriber

**头文件**: `interfaces/inner_api/common_event_subscriber.h`

### 公共事件数据

```cpp
// 获取事件（Want）
inline const Want &GetWant() const;

// 设置结果代码
void SetCode(int code);

// 获取结果代码
int GetCode() const;

// 设置结果数据
void SetData(const std::string &data);

// 获取结果数据
std::string GetData() const;
```

### 有序事件控制

```cpp
// 是否有序事件
bool IsOrderedCommonEvent() const;

// 终止有序事件
void AbortCommonEvent();

// 清除终止状态
void ClearAbortCommonEvent();

// 获取终止状态
bool IsAbortCommonEvent() const;

// 完成事件处理
void FinishCommonEvent();
```

### 订阅信息

```cpp
// 获取订阅信息
std::shared_ptr<CommonEventSubscribeInfo> GetSubscribeInfo() const;
```

## CommonEventData

**头文件**: `interfaces/inner_api/common_event_data.h`

### 构造函数

```cpp
// 默认构造
CommonEventData();

// 带 Want 构造
explicit CommonEventData(const Want &want);

// 带 Want 和发布信息构造
CommonEventData(const Want &want, const CommonEventPublishInfo &publishInfo);
```

### 数据获取与设置

```cpp
// 获取 Want
const Want &GetWant() const;

// 设置 Want
void SetWant(const Want &want);

// 获取发布信息
const CommonEventPublishInfo &GetPublishInfo() const;

// 设置发布信息
void SetPublishInfo(const CommonEventPublishInfo &publishInfo);
```

## CommonEventPublishInfo

**头文件**: `interfaces/inner_api/common_event_publish_info.h`

### 属性设置

```cpp
// 设置有序事件
void SetOrdered(bool ordered);
bool IsOrdered() const;

// 设置粘性事件
void SetSticky(bool sticky);
bool IsSticky() const;

// 设置订阅者权限
void SetSubscriberPermissions(
    const std::vector<std::string> &permissions);
std::vector<std::string> GetSubscriberPermissions() const;

// 设置用户 ID
void SetUserId(int32_t userId);
int32_t GetUserId() const;
```

## CommonEventSubscribeInfo

**头文件**: `interfaces/inner_api/common_event_subscribe_info.h`

### 构造与配置

```cpp
// 默认构造
CommonEventSubscribeInfo();

// 带 MatchingSkills 构造
explicit CommonEventSubscribeInfo(const MatchingSkills &matchingSkills);
```

### 订阅技能

```cpp
// 获取订阅技能
MatchingSkills GetMatchingSkills() const;

// 添加订阅事件
void AddEvent(const std::string &event);

// 添加匹配技能
void AddMatchingSkill(const MatchingSkills &matchingSkill);
```

### 权限与优先级

```cpp
// 设置发布者权限
void SetPublisherPermission(const std::string &permission);
std::string GetPublisherPermission() const;

// 设置优先级
void SetPriority(int32_t priority);
int32_t GetPriority() const;

// 设置设备 ID
void SetPublisherDeviceId(const std::string &deviceId);
std::string GetPublisherDeviceId() const;
```

## MatchingSkills

**头文件**: `interfaces/inner_api/matching_skills.h`

### 技能匹配

```cpp
// 添加匹配事件
void AddEvent(const std::string &event);

// 获取匹配事件
std::vector<std::string> GetEvents() const;

// 添加匹配实体
void AddEntity(const std::string &entity);

// 获取匹配实体
std::vector<std::string> GetEntities() const;
```

## AsyncCommonEventResult

**头文件**: `interfaces/inner_api/async_common_event_result.h`

### 异步结果

```cpp
// 设置结果代码
void SetCode(int32_t code);

// 获取结果代码
int32_t GetCode() const;

// 设置结果数据
void SetData(const std::string &data);

// 获取结果数据
std::string GetData() const;

// 设置完成状态
void SetResult(int32_t result);

// 获取完成状态
int32_t GetResult() const;
```

## 模块依赖

### 依赖关系

```
CommonEventManager
    ├── CommonEventData
    ├── CommonEventPublishInfo
    ├── CommonEventSubscriber
    │   └── CommonEventSubscribeInfo
    │       └── MatchingSkills
    └── CommonEventConstant
```

### 外部依赖

| 依赖 | 用途 |
|------|------|
| ability_base/want | 意图（Want）封装 |
| ipc | 进程间通信 |
| samgr | 服务管理 |
| eventhandler | 事件处理 |

## 稳定性标注

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `CommonEventManager::PublishCommonEvent()` | 稳定 | 公共 API |
| `CommonEventManager::SubscribeCommonEvent()` | 稳定 | 公共 API |
| `CommonEventManager::New*()` | 稳定 | 新版 API |
| `CommonEventSubscriber` | 稳定 | 公共类 |
| 内部实现类 | 不稳定 | 可能变更 |

## 相关文档

- [概览](00_Overview.md)
- [架构](01_Architecture.md)
- [N-API 接口](02_N-API.md)
- [构建编译](04_Build.md)

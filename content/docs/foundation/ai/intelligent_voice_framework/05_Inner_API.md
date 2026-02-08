# Inner API 参考

> **目的**: 提供 Intelligent Voice Framework 内部 Native API 的详细参考  
> **适用范围**: 框架开发者、NDK 开发者、模块集成  
> **最后更新**: 2026-02-06

---

## 1. 概述

Inner API 是框架内部的 C++ 接口，主要用于：
- Native 模块间的通信
- N-API 实现层的底层调用
- 服务层与引擎层的交互

### 1.1 稳定性标注

| 标注 | 含义 | 使用范围 |
|------|------|---------|
| **稳定 (Stable)** | 经过充分测试，接口不变 | 框架内部使用 |
| **不稳定 (Unstable)** | 可能变更 | 框架内部使用 |

### 1.2 头文件位置

- 路径: `interfaces/inner_api/native/`
- 安装: NDK (`innerapi_tags: ["ndk"]`)

---

## 2. 核心接口

### 2.1 IntellVoiceManager

**头文件**: `interfaces/inner_api/native/intell_voice_manager.h`

**职责**: 智能语音管理器，提供引擎创建和生命周期管理

```cpp
namespace OHOS {
namespace IntellVoice {

class IntellVoiceManager {
public:
    // 获取单例
    static IntellVoiceManager *GetInstance();

    // 创建引擎
    int32_t CreateIntellVoiceEngine(IntellVoiceEngineType type, 
                                     sptr<IIntellVoiceEngine> &inst);
    
    // 释放引擎
    int32_t ReleaseIntellVoiceEngine(IntellVoiceEngineType type);
    
    // 初始化
    bool Init();
    
    // 创建耳机唤醒引擎
    std::shared_ptr<WakeupIntellVoiceEngine> CreateHeadsetWakeupEngine();

    // 注册服务死亡回调
    int32_t RegisterServiceDeathRecipient(
        sptr<IRemoteObject::DeathRecipient> callback);
    
    // 获取上传文件
    int32_t GetUploadFiles(int numMax, std::vector<UploadFilesInfo> &reportedFilesInfo);

    // 设置参数
    int32_t SetParameter(const std::string &key, const std::string &value);
    
    // 获取参数
    std::string GetParameter(const std::string &key);
    
    // 获取唤醒源文件
    int32_t GetWakeupSourceFiles(std::vector<WakeupSourceFile> &cloneFileInfo);
    
    // 清除用户数据
    int32_t ClearUserData();

private:
    IntellVoiceManager();
    ~IntellVoiceManager();
    int32_t GetFileDataFromAshmem(sptr<Ashmem> ashmem, 
                                   std::vector<uint8_t> &fileData);

    std::mutex mutex_;
    std::condition_variable proxyConVar_;
    sptr<OHOS::IntellVoiceEngine::IIntellVoiceService> g_sProxy_;
    sptr<UpdateCallbackInner> callback_;
};

}  // namespace IntellVoice
}  // namespace OHOS
```

**证据来源**: `interfaces/inner_api/native/intell_voice_manager.h`

---

### 2.2 EnrollIntellVoiceEngine

**头文件**: `interfaces/inner_api/native/enroll_intell_voice_engine.h`

**职责**: 注册引擎内部接口

```cpp
namespace OHOS {
namespace IntellVoice {

class EnrollIntellVoiceEngine {
public:
    // 获取支持的地区
    virtual std::vector<std::string> GetSupportRegions() = 0;
    
    // 初始化
    virtual int32_t Init(const EngineConfig &config) = 0;
    
    // 执行注册
    virtual int32_t EnrollForResult(bool isLast, 
                                     std::string &context) = 0;
    
    // 停止
    virtual int32_t Stop() = 0;
    
    // 提交
    virtual int32_t Commit() = 0;
    
    // 设置唤醒应用信息
    virtual int32_t SetWakeupHapInfo(const WakeupHapInfo &info) = 0;
    
    // 设置灵敏度
    virtual int32_t SetSensibility(SensibilityType sensibility) = 0;
    
    // 设置参数
    virtual int32_t SetParameter(const std::string &key, 
                                  const std::string &value) = 0;
    
    // 获取参数
    virtual std::string GetParameter(const std::string &key) = 0;
    
    // 评估
    virtual int32_t EvaluateForResult(const std::string &word,
                                       EvaluationResult &result) = 0;
    
    // 释放
    virtual void Release() = 0;

protected:
    virtual ~EnrollIntellVoiceEngine() = default;
};

}  // namespace IntellVoice
}  // namespace OHOS
```

**证据来源**: `frameworks/native/enroll_intell_voice_engine.cpp` (通过头文件分析)

---

### 2.3 WakeupIntellVoiceEngine

**头文件**: `interfaces/inner_api/native/wakeup_intell_voice_engine.h`

**职责**: 唤醒引擎内部接口

```cpp
namespace OHOS {
namespace IntellVoice {

class WakeupIntellVoiceEngine {
public:
    // 获取支持的地区
    virtual std::vector<std::string> GetSupportRegions() = 0;
    
    // 设置唤醒应用信息
    virtual int32_t SetWakeupHapInfo(const WakeupHapInfo &info) = 0;
    
    // 设置灵敏度
    virtual int32_t SetSensibility(SensibilityType sensibility) = 0;
    
    // 设置参数
    virtual int32_t SetParameter(const std::string &key, 
                                  const std::string &value) = 0;
    
    // 获取参数
    virtual std::string GetParameter(const std::string &key) = 0;
    
    // 释放
    virtual void Release() = 0;

protected:
    virtual ~WakeupIntellVoiceEngine() = default;
};

}  // namespace IntellVoice
}  // namespace OHOS
```

---

## 3. 数据结构

### 3.1 引擎配置

```cpp
struct EngineConfig {
    std::string language;  // 语言
    std::string region;    // 地区
};

struct WakeupHapInfo {
    std::string bundleName;    // Bundle 名称
    std::string abilityName;   // Ability 名称
};

enum SensibilityType {
    LOW_SENSIBILITY = 1,
    MIDDLE_SENSIBILITY = 2,
    HIGH_SENSIBILITY = 3
};
```

### 3.2 错误码

```cpp
enum IntelligentVoiceErrorCode {
    INTELLIGENT_VOICE_SUCCESS = 0,
    INTELLIGENT_VOICE_PERMISSION_DENIED = 201,
    INTELLIGENT_VOICE_NOT_SYSTEM_APPLICATION = 202,
    INTELLIGENT_VOICE_NO_MEMORY = 22700101,
    INTELLIGENT_VOICE_INVALID_PARAM = 22700102,
    // ...
};
```

**证据来源**: `interfaces/inner_api/native/intell_voice_info.h`

---

## 4. IIntellVoiceService (IPC 接口)

**头文件**: `services/intell_voice_service/inc/i_intell_voice_service.h`

**职责**: SA 服务的 IPC 接口定义

```cpp
class IIntellVoiceService : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"IntellVoiceFramework.Service");

    enum {
        HDI_INTELL_VOICE_SERVICE_CREATE_ENGINE = 0,
        HDI_INTELL_VOICE_SERVICE_RELEASE_ENGINE,
        HDI_INTELL_VOICE_SERVICE_GET_PARAMETER,
        HDI_INTELL_VOICE_SERVICE_SET_PARAMETER,
        HDI_INTELL_VOICE_SERVICE_CLEAR_USER_DATA,
        // ...
    };

    // 创建引擎
    virtual int32_t CreateIntellVoiceEngine(IntellVoiceEngineType type,
                                              sptr<IIntellVoiceEngine> &inst) = 0;
    
    // 释放引擎
    virtual int32_t ReleaseIntellVoiceEngine(IntellVoiceEngineType type) = 0;
    
    // 设置参数
    virtual int32_t SetParameter(const std::string &keyValueList) = 0;
    
    // 获取参数
    virtual std::string GetParameter(const std::string &key) = 0;
    
    // 清除用户数据
    virtual int32_t ClearUserData() = 0;
};
```

**证据来源**: `services/intell_voice_service/inc/i_intell_voice_service.h:32-62`

---

## 5. IIntellVoiceEngine (IPC 接口)

**头文件**: `services/intell_voice_service/inc/i_intell_voice_engine.h`

**职责**: 引擎的 IPC 接口定义

```cpp
class IIntellVoiceEngine : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"IntellVoiceFramework.Engine");

    enum {
        ENGINE_INIT = 0,
        ENGINE_START,
        ENGINE_STOP,
        ENGINE_RELEASE,
        // ...
    };

    // 引擎操作接口
    virtual int32_t Init(const std::string &params) = 0;
    virtual int32_t Start(const std::string &params) = 0;
    virtual int32_t Stop() = 0;
    virtual int32_t Release() = 0;
    // ...
};
```

---

## 6. 回调接口

### 6.1 IIntellVoiceEngineCallback

**头文件**: `services/intell_voice_service/inc/i_intell_voice_engine_callback.h`

**职责**: 引擎事件回调

```cpp
class IIntellVoiceEngineCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"IntellVoiceFramework.Callback");

    // 引擎事件回调
    virtual void OnCallback(const IntellVoiceEngineCallbackInfo &info) = 0;
};
```

### 6.2 IIntellVoiceUpdateCallback

**头文件**: `services/intell_voice_service/inc/i_intell_voice_update_callback.h`

**职责**: 引擎更新回调

```cpp
class IIntellVoiceUpdateCallback : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"IntellVoiceFramework.UpdateCallback");

    // 更新回调
    virtual void OnUpdateStateUpdate(const UpdateInfo &info) = 0;
};
```

---

## 7. 依赖方向

```
┌────────────────────────────────────────────────────────────┐
│                     frameworks/js (N-API)                  │
│                            │                               │
│                            ▼                               │
│                  frameworks/native (Native)                │ 依赖
│                            │                               │
│         ┌──────────────────┼──────────────────┐             │
│         ▼                  ▼                  ▼             │
│  ┌─────────────┐    ┌─────────────┐    ┌───────────┐     │
│  │ interfaces/ │    │  services/  │    │   utils/  │     │
│  │ inner_api/ │    │             │    │           │     │
│  └─────────────┘    └─────────────┘    └───────────┘     │
└────────────────────────────────────────────────────────────┘
```

**依赖规则**:
- 上层依赖下层
- 同层不循环依赖
- 通过接口抽象解耦

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [N-API 参考](./04_NAPI_Reference.md) | JS 接口 |
| [架构设计](./02_Architecture.md) | 模块关系 |
| [构建系统](./06_Build_System.md) | 构建配置 |

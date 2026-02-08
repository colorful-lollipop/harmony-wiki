# 内部 API 文档

## 模块概览

### common 模块 (静态库)

提供 IPC 通信、工具函数等基础能力。

| 头文件 | 功能 |
|--------|------|
| `error_code/ad_inner_error_code.h` | 错误码定义 |
| `ipc/include/ad_load_proxy.h` | IPC 代理 |
| `ipc/include/ad_request_body_stub.h` | 请求桩 |
| `ipc/include/ad_load_callback_stub.h` | 回调桩 |
| `utils/include/ad_common_util.h` | 通用工具 |
| `utils/include/ad_json_util.h` | JSON 工具 |
| `model/include/ad_constant.h` | 常量定义 |
| `log/include/ad_hilog_wreapper.h` | 日志封装 |

### frameworks/js/napi 模块

| 模块 | 产物 | 职责 |
|------|------|------|
| `ads` | `libadvertising.so` | 主 N-API 入口 |
| `adcomponent` | `libadcomponent.so` | 广告组件 |
| `autoadcomponent` | `libautoadcomponent.so` | 自动广告组件 |
| `adsservice_extension_ability` | `libadsserviceextensionability_napi.so` | SA 扩展能力 |
| `adsservice_extension_context` | `libadsserviceextensioncontext_napi.so` | SA 上下文 |
| `extension` | `libadsservice_extension.so` | 扩展基类 |

## 内部 API 清单

### AdLoadService

广告加载服务单例，管理广告请求和连接。

**文件**: `frameworks/js/napi/ads/include/ad_load_service.h`

```cpp
class AdLoadService {
public:
    static sptr<AdLoadService> GetInstance();
    
    ErrCode LoadAd(const std::string& request, const std::string& options,
                   const sptr<Cloud::IAdLoadCallback>& callback, int32_t loadAdType);
    
    int32_t RequestAdBody(const std::string& request, const std::string& options,
                          const sptr<Cloud::IAdRequestBody>& callback);
    
private:
    void GetAdServiceElement(AdServiceElementName& adServiceElementName);
    bool ConnectAdKit(const sptr<Cloud::AdRequestData>& data,
                      const sptr<Cloud::IAdLoadCallback>& callback, int32_t loadAdType);
};
```

### IPC 接口

#### IAdLoadSendRequest

广告加载请求接口。

**文件**: `common/ipc/include/iad_load_proxy.h`

```cpp
class IAdLoadSendRequest : public IRemoteBroker {
public:
    virtual ErrCode SendAdLoadRequest(const sptr<AdRequestData>& data,
                                        const sptr<IAdLoadCallback>& callback,
                                        int32_t loadAdType) = 0;
    enum { AD_LOAD = 1, MULTI_AD_LOAD = 2 };
};
```

#### IAdLoadCallback

广告加载回调接口。

**文件**: `common/ipc/include/iad_load_callback.h`

```cpp
class IAdLoadCallback : public IRemoteBroker {
public:
    virtual void OnAdLoadSuccess(const std::string& result) = 0;
    virtual void OnAdLoadFailure(int32_t resultCode, const std::string& resultMsg) = 0;
    enum { AD_LOAD = 1, MULTI_AD_LOAD = 2 };
};
```

#### IAdRequestBody

广告请求体接口。

**文件**: `common/ipc/include/iad_request_body.h`

```cpp
class IAdRequestBody : public IRemoteBroker {
public:
    virtual void OnAdLoadSuccess(const std::string& result) = 0;
    enum { REQUEST_BODY_CODE = 1 };
};
```

## 数据结构

### AdRequestData

**文件**: `common/ipc/include/iad_load_proxy.h`

```cpp
struct AdRequestData {
    std::string adRequest;    // 广告请求 JSON
    std::string adOption;     // 广告选项 JSON
    std::string collection;    // 收集数据
    
    bool Marshalling(MessageParcel& parcel) const;
    static sptr<AdRequestData> Unmarshalling(MessageParcel& parcel);
};
```

### AdServiceElementName

**文件**: `frameworks/js/napi/ads/include/ad_load_service.h`

```cpp
struct AdServiceElementName {
    std::string bundleName;      // 服务包名
    std::string extensionName;   // 扩展能力名
    std::string apiServiceName;  // API 服务名
    int32_t userId = -1;         // 用户 ID
};
```

### CloudServiceProvider

**文件**: `frameworks/js/napi/ads/include/advertising.h`

```cpp
struct CloudServiceProvider {
    std::string bundleName;        // 提供者包名
    std::string abilityName;       // 能力名
    std::string ueaAbilityName;    // UIExtension 能力名
};
```

## 常量定义

**文件**: `common/model/include/ad_constant.h`

| 常量 | 值 | 说明 |
|-----|---|------|
| `ADVERTISING_ID` | 6104 | SA 系统能力 ID |
| `SEND_LOAD_AD_REQUEST_CODE` | 1 | 单广告请求码 |
| `SEND_LOAD_MULTI_SOLTS_AD_REQUEST_CODE` | 2 | 多槽位请求码 |
| `CONNECT_TIME_OUT` | 3s | 连接超时 |
| `USER_ID` | -1 | 默认用户 ID |

### 参数键名

| 用途 | 常量 | 值 |
|-----|------|---|
| 请求参数 ID | `AD_REQUEST_PARAM_ID` | "adId" |
| 请求参数类型 | `AD_REQUEST_PARAM_TYPE` | "adType" |
| 请求参数宽 | `AD_REQUEST_PARAM_WIDTH` | "adWidth" |
| 请求参数高 | `AD_REQUEST_PARAM_HEIGHT` | "adHeight" |
| 内容分级 | `AD_CONTENT_CLASSIFICATION` | "adContentClassification" |
| 儿童保护 | `TAG_FOR_CHILD_PROTECTION` | "tagForChildProtection" |
| 非个性化 | `NON_PERSONALIZED_AD` | "nonPersonalizedAd" |
| 显示静音 | `AD_DISPLAY_OPTIONS_MUTE` | "mute" |

## 依赖方向

```
advertising (N-API)
    │
    ├──► advertising_common (静态库)
    │       │
    │       ├──► ipc_core (系统 IPC)
    │       ├──► hilog (系统日志)
    │       ├──► safwk (SA 框架)
    │       └──► samgr (服务管理)
    │
    ├──► ability_runtime
    │       ├──► ability_manager
    │       └──► ability_context
    │
    ├──► ace_engine
    │
    ├──► napi
    │
    └──► bundle_framework
```

## 稳定性标注

| 模块 | 稳定性 | 说明 |
|-----|--------|------|
| advertising N-API | 稳定 | 对外公开 API |
| AdLoadService | 稳定 | 内部单例 |
| advertising_common | 稳定 | 静态库 |
| Extension 基类 | 稳定 | 框架层 |

## 相关文档

- [架构说明](Architecture.md) - 模块关系图
- [N-API 参考](NAPI_Reference.md) - 对外 API
- [构建配置](Build_Configuration.md) - 模块编译配置

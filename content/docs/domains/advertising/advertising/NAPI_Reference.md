# N-API 接口参考

## 模块注册

### advertising 模块入口

**文件**: `frameworks/js/napi/ads/src/ad_init.cpp:51-89`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "advertising",
    .nm_priv = ((void *)0),
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&_module);
}
```

## 导出 API 清单

### 全局函数

| JS API | C++ 实现 | 同步/异步 | 说明 |
|--------|---------|----------|------|
| `showAd` | `ShowAd()` | 异步 | 展示广告 |
| `getAdRequestBody` | `GetAdRequestBody()` | Promise | 获取请求体 |

### AdLoader 类

| JS API | C++ 实现 | 参数 | 说明 |
|--------|---------|------|------|
| `new AdLoader(context)` | `JsConstructor()` | UIAbilityContext | 构造函数 |
| `loadAd(request, options, listener)` | `LoadAd()` | 见下表 | 加载单个广告 |
| `loadAdWithMultiSlots(requests, options, listener)` | `LoadAdWithMultiSlots()` | 见下表 | 加载多槽位广告 |

### AdLoader.loadAd 参数

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| `request` | AdRequestParams | 是 | 广告请求参数 |
| `options` | AdOptions | 是 | 广告配置选项 |
| `listener` | AdLoadListener | 是 | 加载回调 |

### AdLoadListener 回调

| 回调 | 参数 | 说明 |
|-----|------|------|
| `onAdLoadSuccess` | `ads: Advertisement[]` | 加载成功 |
| `onAdLoadFailure` | `errorCode: number, errorMsg: string` | 加载失败 |

## API 详细说明

### advertising.showAd

展示全屏广告（如奖励广告）。

```javascript
import advertising from '@ohos.advertising';
import common from '@ohos.app.ability.common';

const ad = ...; // 从 loadAd 获取
const options = { mute: false };
const context = getContext(this) as common.UIAbilityContext;

advertising.showAd(ad, options, context);
```

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| `advertisement` | Advertisement | 是 | 广告对象 |
| `options` | AdDisplayOptions | 是 | 显示选项 |
| `context` | UIAbilityContext | 是 | 上下文 |

**AdDisplayOptions**:

| 字段 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| `mute` | boolean | false | 是否静音 |
| `customData` | string | - | 自定义数据 |
| `useMobileDataReminder` | boolean | - | 移动数据提醒 |

### advertising.getAdRequestBody

获取广告请求体，用于自定义广告请求。

```javascript
import advertising from '@ohos.advertising';

const requests = [{ adType: 3, adId: "test" }];
const options = { adContentClassification: 'A' };

advertising.getAdRequestBody(requests, options)
  .then((body) => {
    console.info('Ad request body:', body);
  });
```

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| `requests` | AdRequestParams[] | 是 | 广告请求参数数组 |
| `options` | AdOptions | 是 | 广告配置选项 |

**返回值**: Promise\<AdRequestBody\> - 包含签名的请求体

### AdLoader.loadAd

加载单个广告。

```javascript
import advertising from '@ohos.advertising';
import common from '@ohos.app.ability.common';

const context = getContext(this) as common.UIAbilityContext;
const adLoader = new advertising.AdLoader(context);

const request = {
  adType: 3,    // 广告类型
  adId: "test", // 广告位 ID
  adWidth: 360,
  adHeight: 360
};

const options = {
  adContentClassification: 'A'
};

const listener = {
  onAdLoadSuccess: (ads) => {
    console.info('Load success:', ads);
  },
  onAdLoadFailure: (code, msg) => {
    console.error('Load failed:', code, msg);
  }
};

adLoader.loadAd(request, options, listener);
```

**AdRequestParams**:

| 字段 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| `adId` | string | 是 | 广告位 ID |
| `adType` | number | 是 | 广告类型 |
| `adCount` | number | 否 | 请求数量 |
| `adWidth` | number | 否 | 广告宽度 |
| `adHeight` | number | 否 | 广告高度 |
| `adExtra` | string | 否 | 扩展数据 |

**AdOptions**:

| 字段 | 类型 | 说明 |
|-----|------|------|
| `adContentClassification` | string | 内容分级 (A/B/C) |
| `tagForChildProtection` | number | 儿童保护标签 |
| `nonPersonalizedAd` | number | 非个性化广告 |
| `optionsExtra` | string | 扩展选项 |

### AdLoader.loadAdWithMultiSlots

加载多个广告位的广告。

```javascript
const requests = [
  { adId: "slot1", adType: 3 },
  { adId: "slot2", adType: 3 }
];
const options = { adContentClassification: 'A' };

adLoader.loadAdWithMultiSlots(requests, options, listener);
```

## 错误码

### 业务错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 200 | `CODE_SUCCESS` | 成功 |
| 401 | `CODE_INVALID_PARAMETERS` | 参数无效 |
| 100001 | `CODE_INIT_CONFIG_FAILURE` | 初始化配置失败 |
| 100003 | `CODE_LOAD_ADS_FAILURE` | 加载广告失败 |

### 内部错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 401 | `PARAM_ERR` | 参数错误 |
| 801 | `DEVICE_ERR` | 设备不支持 |
| 21800001 | `INNER_ERR` | 内部错误 |
| 21800003 | `REQUEST_FAIL` | 请求失败 |
| 21800004 | `DISPLAY_ERR` | 显示错误 |

### 公共错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 1 | `ERR_AD_COMMON_CHECK_DESCRIPTOR_ERROR` | 描述符校验失败 |
| 2 | `ERR_AD_COMMON_READ_AD_SIZE_ERROR` | 读取广告大小失败 |
| 4 | `ERR_AD_COMMON_SEND_REQUEST_ERROR` | 发送请求失败 |
| 5 | `ERR_AD_COMMON_NAPI_CALLBACK_NULL_ERROR` | 回调为空 |
| 6 | `ERR_AD_COMMON_AD_PROXY_NULL_ERROR` | 代理为空 |
| 12 | `ERR_AD_COMMON_AD_CONNECT_KIT_ERROR` | 连接广告 Kit 失败 |
| 14 | `ERR_AD_COMMON_AD_SA_REMOTE_OBJECT_ERROR` | SA 远程对象为空 |

## 调用链

### showAd 调用链

```
JS: advertising.showAd()
    ↓
C++: Advertising::ShowAd() [advertising.cpp:525]
    ↓
C++: ParseObjectFromJs() [advertising.cpp:317]
    ↓
C++: StartUIExtensionAbility() [advertising.cpp:440]
    ↓
C++: uiContent->CreateModalUIExtension() [Ace API]
    ↓
UIExtensionAbility: AdsUIExtensionAbility (由广告平台实现)
```

### loadAd 调用链

```
JS: adLoader.loadAd()
    ↓
C++: Advertising::LoadAd() [advertising.cpp:646]
    ↓
C++: ParseContextForLoadAd() [advertising.cpp:606]
    ↓
C++: napi_create_async_work() [libuv]
    ↓
C++: AdLoadService::LoadAd() [ad_load_service.cpp:113]
    ↓
C++: ConnectAdKit() → ConnectExtensionAbility() [ad_load_service.cpp:207]
    ↓
IPC: AdLoadSendRequestProxy::SendAdLoadRequest() [ad_load_proxy.cpp:36]
    ↓
IPC: SendRequest() [Binder 同步调用]
    ↓
SA: AdsServiceExtensionAbility::onRemoteMessageRequest()
    ↓
Platform: onLoadAd() (第三方广告平台实现)
```

## 相关文档

- [架构说明](Architecture.md) - 完整数据流图
- [安全评审](Security_Review.md) - API 安全考量

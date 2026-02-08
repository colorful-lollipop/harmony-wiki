# N-API 参考

> 本文档描述 Form Fwk 的 N-API 接口，包括 JS API 清单、参数、返回值及 C++ 实现位置

## N-API 模块总览

| 模块 | N-API 路径 | 功能 |
|------|-----------|------|
| **formProvider** | `frameworks/js/napi/formProvider/` | 卡片提供方接口 |
| **formHost** | `frameworks/js/napi/formHost/` | 卡片使用方接口 |
| **formAgent** | `frameworks/js/napi/form_agent/` | 卡片操作代理 |
| **formObserver** | `frameworks/js/napi/form_observer/` | 卡片状态观察 |
| **formInfo** | `frameworks/js/napi/form_info/` | 卡片信息查询 |
| **formBindingData** | `frameworks/js/napi/form_binding_data/` | 卡片数据绑定 |
| **formError** | `frameworks/js/napi/form_error/` | 错误码定义 |
| **formUtil** | `frameworks/js/napi/formUtil/` | N-API 工具函数 |
| **FormExtensionAbility** | `frameworks/js/napi/form_extension_ability/` | Stage 模型扩展 |
| **LiveFormExtensionAbility** | `frameworks/js/napi/live_form_extension_ability/` | 实时卡片扩展 |
| **FormEditExtensionAbility** | `frameworks/js/napi/form_edit_extension_ability/` | 卡片编辑扩展 |

## formProvider 模块

**N-API 入口**: `frameworks/js/napi/formProvider/native_module.cpp`
**JS 模块名**: `app.form.formProvider`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| getFormsInfo | `JsFormProvider::GetFormsInfo` | filter?: FormInfoFilter | FormInfo[] | 同步 |
| getPublishedFormInfos | `JsFormProvider::GetPublishedFormInfos` | - | RunningFormInfo[] | 同步 |
| getPublishedFormInfoById | `JsFormProvider::GetPublishedFormInfoById` | formId: number | RunningFormInfo | 同步 |
| openFormManager | `JsFormProvider::OpenFormManager` | want: Want | FormManager | 同步 |
| openFormManagerCrossBundle | `JsFormProvider::OpenFormManagerCrossBundle` | want: Want | FormManager | 同步 |
| setFormNextRefreshTime | `JsFormProvider::SetFormNextRefreshTime` | formId, time | void | 异步 Promise |
| updateForm | `JsFormProvider::UpdateForm` | formId, data | void | 异步 Promise |
| requestPublishForm | `JsFormProvider::RequestPublishForm` | want, data | number | 异步 Promise |
| isRequestPublishFormSupported | `JsFormProvider::IsRequestPublishFormSupported` | - | boolean | 同步 |
| openFormEditAbility | `JsFormProvider::OpenFormEditAbility` | abilityName, formId, isMainPage | void | 异步 Promise |
| closeFormEditAbility | `JsFormProvider::CloseFormEditAbility` | isMainPage | void | 异步 Promise |
| requestOverflow | `JsFormProvider::RequestOverflow` | formId, overflowInfo | void | 异步 Promise |
| cancelOverflow | `JsFormProvider::CancelOverflow` | formId | void | 异步 Promise |
| activateSceneAnimation | `JsFormProvider::ActivateSceneAnimation` | formId | void | 异步 Promise |
| deactivateSceneAnimation | `JsFormProvider::DeactivateSceneAnimation` | formId | void | 异步 Promise |
| getFormRect | `JsFormProvider::GetFormRect` | formId | Rect | 异步 Promise |
| getPublishedRunningFormInfos | `JsFormProvider::GetPublishedRunningFormInfos` | - | RunningFormInfo[] | 同步 |
| getPublishedRunningFormInfoById | `JsFormProvider::GetPublishedRunningFormInfoById` | formId | RunningFormInfo | 同步 |
| reloadForms | `JsFormProvider::ReloadForms` | params | void | 异步 Promise |
| reloadAllForms | `JsFormProvider::ReloadAllForms` | - | void | 异步 Promise |
| onPublishFormCrossBundleControl | `JsFormProvider::RegisterPublishFormCrossBundleControl` | callback | void | 同步 |
| offPublishFormCrossBundleControl | `JsFormProvider::UnregisterPublishFormCrossBundleControl` | callback | void | 同步 |
| updateTemplateFormDetailInfo | `JsFormProvider::UpdateTemplateFormDetailInfo` | infos | void | 异步 Promise |

## formHost 模块

**N-API 入口**: `frameworks/js/napi/formHost/native_module.cpp`
**JS 模块名**: `app.form.formHost`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| acquireForm | `NapiFormHost::AcquireForm` | want, callbacks | number | 异步 Promise |
| shareForm | `NapiFormHost::ShareForm` | formId, deviceId | void | 异步 Promise |
| deleteForm | `NapiFormHost::DeleteForm` | formId | void | 异步 Promise |
| releaseForm | `NapiFormHost::ReleaseForm` | formId, isCache | void | 异步 Promise |
| requestForm | `NapiFormHost::RequestForm` | formId | void | 异步 Promise |
| castTempForm | `NapiFormHost::CastTempForm` | formId | void | 异步 Promise |
| getAllFormsInfo | `NapiFormHost::GetAllFormsInfo` | - | FormInfo[] | 同步 |
| getFormsInfo | `NapiFormHost::GetFormsInfo` | filter | FormInfo[] | 同步 |
| enableFormsUpdate | `NapiFormHost::EnableFormsUpdate` | formIds | void | 异步 Promise |
| disableFormsUpdate | `NapiFormHost::DisableFormsUpdate` | formIds | void | 异步 Promise |
| notifyFormsVisible | `NapiFormHost::NotifyFormsVisible` | formIds, visible | void | 异步 Promise |
| notifyFormsPrivacyProtected | `NapiFormHost::NotifyFormsPrivacyProtected` | formIds, privacy | void | 异步 Promise |
| deleteInvalidForms | `NapiFormHost::DeleteInvalidForms` | formIds | number | 异步 Promise |
| acquireFormState | `NapiFormHost::AcquireFormState` | want | FormState | 异步 Promise |
| on | `NapiFormHost::Register` | type, callback | void | 同步 |
| off | `NapiFormHost::Unregister` | type, callback | void | 同步 |

## formAgent 模块

**N-API 入口**: `frameworks/js/napi/form_agent/native_module.cpp`
**JS 模块名**: `app.form.formAgent`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| requestPublishForm | `JsFormAgent::RequestPublishForm` | want | number | 异步 Promise |
| deleteForm | `JsFormAgent::DeleteForm` | formId | void | 异步 Promise |
| convertForm | `JsFormAgent::ConvertForm` | formId, tempId | void | 异步 Promise |

## formObserver 模块

**N-API 入口**: `frameworks/js/napi/form_observer/native_module.cpp`
**JS 模块名**: `app.form.formObserver`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| on | `JsFormObserver::Register` | type, callback | void | 同步 |
| off | `JsFormObserver::Unregister` | type, callback | void | 同步 |

## formInfo 模块

**N-API 入口**: `frameworks/js/napi/form_info/form_info_module.cpp`
**JS 模块名**: `app.form.formInfo` / `application.formInfo`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| isFormSupposedToBeModel | `JsFormInfo::IsFormSupposedToBeModel` | bundleName, moduleName, formName | boolean | 同步 |
| isSystemForm | `JsFormInfo::IsSystemForm` | bundleName, moduleName, formName | boolean | 同步 |

## formBindingData 模块

**N-API 入口**: `frameworks/js/napi/form_binding_data/js_form_binding_data_module.cpp`
**JS 模块名**: `app.form.formBindingData`

### API 清单

| JS API | C++ 实现 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| constructor | `JsFormBindingData::Constructor` | data | FormBindingData | 同步 |

## formError 模块

**N-API 入口**: `frameworks/js/napi/form_error/form_error_module.cpp`
**JS 模块名**: `application.formError`

### 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 16500000 | 系统错误 |
| 16500001 | 卡片不存在 |
| 16500002 | 卡片状态异常 |
| 16500003 | 卡片操作失败 |
| 16500050 | 权限不足 |

## formUtil 工具函数

**头文件**: `frameworks/js/napi/formUtil/napi_form_util.h`

### 工具函数

| 函数 | 用途 |
|------|------|
| `Throw()` | 抛出异常 |
| `ThrowByInternalErrorCode()` | 根据内部错误码抛出 |
| `ThrowByExternalErrorCode()` | 根据外部错误码抛出 |
| `ThrowParamTypeError()` | 参数类型错误 |
| `ThrowParamNumError()` | 参数数量错误 |
| `GetStringFromNapi()` | 从 N-API 获取字符串 |
| `ParseParam()` | 解析参数 |

## N-API 调用链

```
JS 调用
    │
    ▼
N-API Module (native_module.cpp)
    │
    ├── BindNativeFunction() / napi_define_properties()
    │
    ▼
C++ 实现 (js_*.cpp)
    │
    ├── 参数解析 (formUtil)
    │
    ├── IPC 调用 (FormMgrProxy)
    │
    ▼
FormMgrService (IPC Server)
```

## 错误码说明

| 错误码范围 | 类型 |
|-----------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 16500000-16500100 | Form 框架错误 |
| 16500100+ | 业务错误 |

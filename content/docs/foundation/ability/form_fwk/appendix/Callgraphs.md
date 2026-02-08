# 关键调用链

> 本文档描述 Form Fwk 的关键调用链，展示从入口到核心逻辑的完整路径

## 1. 添加卡片调用链

```
JS 入口                    N-API 层                    IPC 层                    服务层
─────────────────────────────────────────────────────────────────────────────────────────
formHost.acquireForm()
         │
         ▼
native_module.cpp: Init()
         │
         ├── BindNativeFunction("acquireForm")
         │
         ▼
js_form_host.cpp: AcquireForm()
         │
         ├── 参数解析 (formUtil)
         │
         ├── FormMgrProxy::AddForm()
         │
         ▼ IPC ───────────────────────────────────────────────────────────────▶
form_mgr_proxy.cpp: AddForm()
         
form_mgr_stub.cpp: OnRemoteRequest(FORM_MGR_ADD_FORM)
         │
         ├── 权限检查 (CheckFormPermission)
         │
         ├── FormMgrService::OnAddForm()
         │         │
         │         ├── FormAmsHelper::GetAbilityToken()
         │         │
         │         ├── FormDataMgr::AddFormRecord()
         │         │
         │         ├── FormProviderMgr::InitFormProvider()
         │         │
         │         └── FormRenderMgr::StartRender()
         │
         └── 返回 FormJsInfo
```

## 2. 更新卡片调用链

```
JS 入口                    N-API 层                    IPC 层                    服务层
─────────────────────────────────────────────────────────────────────────────────────────
formProvider.updateForm()
         │
         ▼
native_module.cpp: Init()
         │
         ├── BindNativeFunction("updateForm")
         │
         ▼
js_form_provider.cpp: UpdateForm()
         │
         ├── 参数解析
         │
         ├── FormMgrProxy::UpdateForm()
         │
         ▼ IPC ───────────────────────────────────────────────────────────────▶
form_mgr_proxy.cpp: UpdateForm()
         
form_mgr_stub.cpp: OnRemoteRequest(FORM_MGR_UPDATE_FORM)
         │
         ├── FormMgrService::OnUpdateForm()
         │         │
         │         ├── FormDataMgr::UpdateFormData()
         │         │
         │         └── FormRenderMgr::NotifyUpdate()
         │
         └── 返回结果
```

## 3. 删除卡片调用链

```
JS 入口                    N-API 层                    IPC 层                    服务层
─────────────────────────────────────────────────────────────────────────────────────────
formHost.deleteForm()
         │
         ▼
js_form_host.cpp: DeleteForm()
         │
         ├── FormMgrProxy::DeleteForm()
         │
         ▼ IPC ───────────────────────────────────────────────────────────────▶
form_mgr_proxy.cpp: DeleteForm()
         
form_mgr_stub.cpp: OnRemoteRequest(FORM_MGR_DELETE_FORM)
         │
         ├── 权限检查
         │
         ├── FormMgrService::OnDeleteForm()
         │         │
         │         ├── FormDataMgr::DeleteFormRecord()
         │         │
         │         ├── FormRenderMgr::StopRender()
         │         │
         │         └── FormProviderMgr::NotifyFormDeleted()
         │
         └── 返回结果
```

## 4. 卡片刷新调用链

```
刷新触发                    检查层                    刷新层                    服务层
─────────────────────────────────────────────────────────────────────────────────────────
FormRefreshMgr::StartRefresh()
         │
         ▼
RefreshCheckMgr::Check()
         │
         ├── ActiveUserChecker::Check()
         │
         ├── CallingUserChecker::Check()
         │
         ├── CallingBundleChecker::Check()
         │
         ├── UntrustAppChecker::Check()
         │
         ├── SystemAppChecker::Check()
         │
         ├── SelfFormChecker::Check()
         │
         └── MultiActiveUsersChecker::Check()
         │
         ▼ (全部通过)
RefreshExecMgr::Execute()
         │
         ├── FormTimerRefreshImpl::Execute()
         │         │
         │         └── FormRefreshConnection::Execute()
         │
         ├── FormDataRefreshImpl::Execute()
         │
         └── FormNetConnRefreshImpl::Execute()
```

## 5. 关键文件索引

| 调用步骤 | 文件 | 行号 | 函数 |
|----------|------|------|------|
| JS 入口 | `formHost/native_module.cpp` | L85 | `Init()` |
| N-API 绑定 | `formHost/napi_form_host.cpp` | - | `AcquireForm()` |
| IPC Proxy | `form_mgr_proxy.cpp` | L200+ | `AddForm()` |
| IPC Stub | `form_mgr_stub.cpp` | L50+ | `OnRemoteRequest()` |
| 权限检查 | `form_mgr_service.cpp` | L987 | `CheckFormPermission()` |
| 卡片管理 | `form_data_mgr.cpp` | L500+ | `AddFormRecord()` |
| 渲染管理 | `form_render_mgr.cpp` | L100+ | `StartRender()` |
| 刷新检查 | `refresh_check_mgr.cpp` | L50+ | `Check()` |

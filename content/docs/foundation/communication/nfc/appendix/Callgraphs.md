# 附录：关键调用链

## 1. NFC 开启调用链

```
JS: enableNfc()
  ↓
N-API: EnableNfc() [nfc_napi_controller_adapter.cpp:47]
  ↓
Inner API: NfcController::TurnOn() [nfc_controller.cpp]
  ↓
IPC: INfcController::TurnOn() (ipc code 102)
  ↓
Stub: NfcControllerImpl::OnRemoteRequest() [nfc_controller_impl.cpp]
  ↓
Stub: NfcControllerImpl::TurnOn()
  ↓
Service: NfcService::ExecuteTask(TASK_TURN_ON) [nfc_service.cpp]
  ↓
Handler: NfcSwitchEventHandler::ProcessEvent() [nfc_service.cpp:114]
  ↓
Service: NfcService::DoTurnOn()
  ↓
NCI: NciNfccProxy::Initialize() [nci_nfcc_proxy.cpp]
  ↓
Native: nfcc_nci_adapter::Initialize() [nfcc_nci_adapter.cpp]
  ↓
Hardware: NFC Controller
```

## 2. 标签发现调用链

```
Hardware: Tag Detected
  ↓
Native: tag_nci_adapter::HandleTagNotification() [tag_nci_adapter_ntf.cpp]
  ↓
NCI: NciTagProxy::OnTagDiscovered() [nci_tag_proxy.cpp]
  ↓
Listener: NfcService::OnTagDiscovered() [nfc_service.cpp]
  ↓
Handler: NfcEventHandler::ProcessEvent(MSG_TAG_FOUND)
  ↓
Dispatcher: TagDispatcher::DispatchTag() [tag_dispatcher.cpp]
  ↓
  ├─ Reader Mode → ReaderModeCallback::OnTagDiscovered()
  ├─ Foreground → ForegroundCallback::OnTagDiscovered()
  ├─ NDEF Dispatch → App launch via Want
  └─ Default → Notification
```

## 3. NDEF 读取调用链

```
JS: ndefTag.readNdef()
  ↓
N-API: NapiNdefTag::ReadNdef() [nfc_napi_tag_ndef.cpp:379]
  ↓
Async: NativeReadNdef() (worker thread)
  ↓
Inner API: NdefTag::ReadNdef() [ndef_tag.cpp]
  ↓
IPC: ITagSession::NdefRead() (ipc code 208)
  ↓
Stub: TagSession::OnRemoteRequest() [tag_session.cpp]
  ↓
Stub: TagSession::NdefRead()
  ↓
NCI: NciTagProxy::ReadNdef()
  ↓
Native: tag_nci_adapter::ReadNdef() [tag_nci_adapter_rw.cpp]
  ↓
Hardware: NFC Controller ←→ Tag
  ↓
Callback: ReadNdefCallback() (main thread)
  ↓
N-API: Promise Resolve → JS
```

## 4. HCE APDU 处理调用链

```
Hardware: APDU Command from Reader
  ↓
Native: nci_ce_impl_default::OnApduReceived()
  ↓
NCI: NciCeProxy::OnCardEmulationData() [nci_ce_proxy.cpp]
  ↓
Listener: NfcService::OnCardEmulationData() [nfc_service.cpp]
  ↓
Service: CeService::OnCardEmulationData() [ce_service.cpp]
  ↓
Manager: HostCardEmulationManager::OnCardEmulationData()
  ↓
  ├─ Check: IsCorrespondentService() (security check)
  ↓
Callback: IHceCmdCallback::OnCeApduData()
  ↓
IPC → App: HceService.on('hceCmd', callback)
  ↓
App: Process APDU → HceSession.sendResponse()
  ↓
Reverse chain back to Hardware
```

## 5. 前台分发注册调用链

```
JS: tag.registerForegroundDispatch(want, callback)
  ↓
N-API: RegisterForegroundDispatch() [nfc_napi_foreground_dispatch.cpp]
  ↓
Inner API: TagForeground::RegForegroundDispatch() [tag_foreground.cpp]
  ↓
IPC: ITagSession::RegForegroundDispatch() (ipc code 109)
  ↓
Stub: TagSession::RegForegroundDispatch() [tag_session.cpp]
  ↓
Manager: NfcPollingManager::RegForegroundDispatch() [nfc_polling_manager.cpp]
  ↓
Registry: Store ForegroundRegistryData
  ↓
NCI: NciNfccProxy::EnableDiscovery() (update tech mask)
```


# 关键调用链

## 1. PerfRequest 调用链

### 完整调用链

```
应用层
    ↓
ReportData() [ressched_client]
    ↓
资源调度框架
    ↓
SocPerfPlugin::OnRemoteRequest() [socperf_plugin]
    ↓
ISocPerf::PerfRequest() [IPC]
    ↓
SocPerfServer::PerfRequest()
    ↓
HasPerfPermission() [权限校验]
    ↓
SocPerf::PerfRequest()
    ↓
GetActionsInfo(cmdId) [配置查询]
    ↓
DoFreqActions() [动作执行]
    ↓
SocPerfThreadWrap::Execute()
    ↓
Linux cpufreq 接口
```

### 关键代码位置

| 阶段 | 文件:行号 |
|------|----------|
| IPC 调用 | `socperf_client.cpp:113-121` |
| 服务端入口 | `socperf_server.cpp:72-79` |
| 权限校验 | `socperf_server.cpp:183-212` |
| 核心处理 | `socperf.cpp:PerfRequest()` |
| 配置查询 | `socperf.cpp:GetActionsInfo()` |
| 动作执行 | `socperf.cpp:DoFreqActions()` |
| 线程封装 | `socperf_thread_wrap.cpp:Execute()` |

---

## 2. 权限校验调用链

```
IPC 调用到达
    ↓
IPCSkeleton::GetCallingTokenID()
    ↓
GetTokenTypeFlag()
    ├─ TOKEN_HAP → IsSystemAppByFullTokenID()
    │                    ↓
    │              VerifyAccessToken()
    │                    ↓
    │              permissionCache_.get()
    │                    ↓
    │              缓存命中 → 直接返回
    │              缓存未命中 → VerifyAccessToken()
    │                    ↓
    │              permissionCache_.put()
    │
    └─ 其他 TokenType → 直接通过
```

### 关键代码位置

| 函数 | 文件:行号 |
|------|----------|
| `GetCallingTokenID()` | `socperf_server.cpp:184` |
| `GetTokenTypeFlag()` | `socperf_server.cpp:186` |
| `IsSystemAppByFullTokenID()` | `socperf_server.cpp:189-191` |
| `VerifyAccessToken()` | `socperf_server.cpp:198` |
| `permissionCache_.get()` | `socperf_server.cpp:196` |
| `permissionCache_.put()` | `socperf_server.cpp:200` |

---

## 3. 配置加载调用链

```
SocPerf::Init()
    ↓
SocPerfConfig::Init()
    ↓
GetRealConfigPath("socperf_resource_config.xml")
    ↓
LoadAllConfigXmlFile()
    ↓
LoadConfigXmlFile()
    ├─ ParseResourceXmlFile()
    └─ ParseBoostXmlFile()
    ↓
解析完成后注册回调
```

### 关键代码位置

| 函数 | 文件:行号 |
|------|----------|
| `Init()` | `socperf.cpp:Init()` |
| `SocPerfConfig::Init()` | `socperf_config.cpp:Init()` |
| `GetRealConfigPath()` | `socperf_config.cpp:GetRealConfigPath()` |
| `LoadAllConfigXmlFile()` | `socperf_config.cpp:LoadAllConfigXmlFile()` |
| `ParseResourceXmlFile()` | `socperf_config.cpp:ParseResourceXmlFile()` |
| `ParseBoostXmlFile()` | `socperf_config.cpp:ParseBoostXmlFile()` |

---

## 4. 服务端生命周期调用链

```
系统启动
    ↓
SystemAbility::MakeAndRegisterAbility()
    ↓
SocPerfServer 构造函数
    ↓
OnStart()
    ├─ Publish(this) [注册到 SAMgr]
    ├─ socPerf.Init() [配置加载]
    └─ OnStartDone()
    ↓
OnStop()
    ├─ SocPerf 清理
    └─ 服务注销
```

### 关键代码位置

| 函数 | 文件:行号 |
|------|----------|
| `MakeAndRegisterAbility()` | `socperf_server.cpp:30-31` |
| `OnStart()` | `socperf_server.cpp:42-56` |
| `Publish()` | `socperf_server.cpp:48` |
| `OnStop()` | `socperf_server.cpp:58-62` |

---

## 5. 客户端连接调用链

```
SocPerfClient::GetInstance()
    ↓
PerfRequest(cmdId, msg)
    ↓
CheckClientValid()
    ├─ GetSystemAbilityManager()
    ├─ CheckSystemAbility(SOC_PERF_SERVICE_SA_ID)
    └─ iface_cast<ISocPerf>()
    ↓
AddDeathRecipient()
    ↓
client->PerfRequest(cmdId, newMsg)
```

### 关键代码位置

| 函数 | 文件:行号 |
|------|----------|
| `GetInstance()` | `socperf_client.cpp:41-44` |
| `CheckClientValid()` | `socperf_client.cpp:47-78` |
| `GetSystemAbilityManager()` | `socperf_client.cpp:53` |
| `CheckSystemAbility()` | `socperf_client.cpp:59` |
| `iface_cast()` | `socperf_client.cpp:65` |
| `AddDeathRecipient()` | `socperf_client.cpp:75` |

---

## 6. 设备模式匹配调用链

```
RequestDeviceMode(mode, status)
    ↓
MatchDeviceMode(mode, status, scenes)
    ↓
GetMatchCmdId(cmdId, isTagOnOff)
    ↓
GetActionsInfo(cmdId)
    ↓
DoFreqActions()
```

### 关键代码位置

| 函数 | 文件:行号 |
|------|----------|
| `RequestDeviceMode()` | `socperf.cpp:RequestDeviceMode()` |
| `MatchDeviceMode()` | `socperf.cpp:MatchDeviceMode()` |
| `GetMatchCmdId()` | `socperf.cpp:GetMatchCmdId()` |
| `GetActionsInfo()` | `socperf.cpp:GetActionsInfo()` |
| `DoFreqActions()` | `socperf.cpp:DoFreqActions()` |

# AVSession 架构

## 逻辑架构图

```mermaid
graph TB
    subgraph 应用层
        JS["JS 应用"]
        Native["Native 应用"]
        CJ["Cangjie 应用"]
    end

    subgraph 框架层 frameworks/
        NAPI["JS N-API<br/>frameworks/js/napi/"]
        NativeFW["Native 框架<br/>frameworks/native/"]
        CJFW["Cangjie 框架<br/>frameworks/cj/"]
        Taihe["Taihe 框架<br/>frameworks/taihe/"]
    end

    subgraph 接口层 interfaces/
        InnerAPI["Native Inner API<br/>interfaces/inner_api/"]
    end

    subgraph 服务层 services/session/
        IPC["IPC 通信<br/>ipc/proxy & stub/"]
        Service["AVSessionService<br/>server/"]
        Distributed["分布式服务<br/>server/remote/"]
    end

    subgraph 系统服务
        SAMgr["SA Manager"]
        Audio["Audio 服务"]
        Bundle["Bundle 服务"]
        Device["Device Manager"]
    end

    JS --> NAPI
    Native --> NativeFW
    CJ --> CJFW
    Taihe --> NativeFW

    NAPI --> InnerAPI
    NativeFW --> InnerAPI
    CJFW --> InnerAPI

    InnerAPI --> IPC
    IPC --> Service
    Service --> Distributed
    Service --> SAMgr
    Service --> Audio
    Service --> Bundle
    Service --> Device
```

---

## 组件职责

### 1. 应用层

| 组件 | 职责 |
|------|------|
| **JS 应用** | 通过 N-API 接口使用 AVSession |
| **Native 应用** | 通过 Native API 使用 AVSession |
| **Cangjie 应用** | 通过 FFI 使用 AVSession |

### 2. 框架层

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **JS N-API** | JS 到 Native 的桥接层 | `napi_module.cpp`, `napi_avsession*.cpp` |
| **Native 框架** | Native 客户端实现 | `avsession_manager_impl.cpp` |
| **Cangjie 框架** | Cangjie 语言绑定 | `cj_avsession*.cpp` |
| **Taihe 框架** | 跨语言框架 | `taihe_avsession*.cpp` |

### 3. 接口层

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **Inner API** | 定义 Native 接口契约 | `av_session.h`, `avsession_controller.h`, `avsession_manager.h` |

### 4. 服务层

| 组件 | 职责 | 关键文件 |
|------|------|----------|
| **IPC 通信** | 处理进程间通信 | `avsession_service_proxy/stub` |
| **AVSessionService** | 全局会话管理 SA 服务 | `avsession_service.h/cpp` |
| **分布式服务** | 跨设备会话同步 | `remote_session_*.cpp`, `migrate_*.cpp` |

---

## 数据流向

### 典型调用链: JS 创建会话

```mermaid
sequenceDiagram
    participant JS as JS 应用
    participant NAPI as N-API 层
    participant IPC as IPC Proxy
    participant SA as AVSessionService
    participant Item as AVSessionItem

    JS->>NAPI: createAVSession(tag, type, elementName)
    NAPI->>IPC: CreateSession()
    IPC->>SA: IPC 调用
    SA->>Item: 创建 AVSessionItem
    Item->>SA: 返回 sessionId
    SA->>IPC: 返回 AVSessionProxy
    IPC->>NAPI: 返回结果
    NAPI->>JS: 返回 AVSession 实例
```

### 控制命令调用链

```mermaid
sequenceDiagram
    participant Ctrl as 播控中心
    participant Proxy as Controller Proxy
    participant SA as AVSessionService
    participant Item as AVSessionItem
    participant App as 媒体应用

    Ctrl->>Proxy: sendControlCommand(command)
    Proxy->>SA: IPC 调用
    SA->>Item: 路由命令
    Item->>App: 回调 OnPlay/OnPause 等
    App->>Item: 更新播放状态
    Item->>SA: 通知状态变化
    SA->>Ctrl: 广播状态变化
```

---

## 线程模型

### 关键线程

| 线程 | 职责 | 代码位置 |
|------|------|----------|
| **Main UI Thread** | 应用主线程，处理 JS 调用 | JS 运行环境 |
| **IPC Thread** | 处理 IPC 通信 | `ipc/proxy/*.cpp` |
| **Service Main Thread** | SA 服务主线程 | `server/avsession_service.cpp` |
| **Event Handler Thread** | 事件处理线程 | `utils/src/avsession_event_handler.cpp` |

### 线程间通信

```cpp
// 事件投递示例 (avsession_event_handler.cpp)
AVSessionEventHandler &handler = AVSessionEventHandler::GetInstance();
handler.AVSessionPostTask([callback]() {
    callback->OnPlay();
}, std::string(__FUNCTION__));
```

---

## IPC 接口定义

### SA 服务配置

```json
// sa_profile/av_session.json
{
    "id": 3010,
    "name": "avsession_service",
    "lib-path": "libavsession_service.z.so",
    "run-on-create": true,
    "distributed": true
}
```

### IPC 接口列表

| 接口 | 描述符 | 主要方法 |
|------|--------|----------|
| **IAVSessionService** | `ohos.avsession.IAVSessionService` | CreateSession, GetAllSessionDescriptors |
| **IAVSession** | `ohos.avsession.IAVSession` | SetAVMetaData, SetAVPlaybackState, Activate |
| **IAVSessionController** | `ohos.avsession.IAVSessionController` | SendControlCommand, GetAVPlaybackState |
| **IAVCastController** | `ohos.avsession.IAVCastController** | Start, Prepare, SendControlCommand |

---

## 分布式架构

### 组件分布

```mermaid
graph LR
    subgraph 本地设备
        App1["媒体应用"]
        LocalSA["AVSessionService"]
        Controller["播控中心"]
    end

    subgraph 远端设备
        RemoteSA["AVSessionService"]
        RemoteApp["远端媒体应用"]
    end

    subgraph 分布式通信
        Softbus["Softbus"]
        CastEngine["Cast Engine"]
    end

    App1 --> LocalSA
    Controller --> LocalSA
    LocalSA --> Softbus
    Softbus --> RemoteSA
    RemoteSA --> RemoteApp
    LocalSA --> CastEngine
```

### 分布式能力

| 能力 | 实现组件 | 说明 |
|------|----------|------|
| **会话迁移** | `migrate_avsession_*` | 跨设备会话迁移 |
| **远端控制** | `remote_session_*` | 远端会话控制 |
| **音频投送** | `cast_audio_*` | 本地音频投送到远端 |
| **设备发现** | `cast_discovery_*` | 发现可投送设备 |

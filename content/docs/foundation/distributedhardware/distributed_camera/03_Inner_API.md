# 内部 API 与接口

## SDK 接口 (Native C++)

> **注意**：本仓库无 N-API 接口，仅提供 Native C++ SDK。

SDK 接口通过 IPC 机制暴露，采用 IRemoteBroker 模式。

---

## Source SDK 接口

### 接口定义

**文件**：`interfaces/inner_kits/native_cpp/camera_source/include/idistributed_camera_source.h`

```cpp
namespace OHOS {
namespace DistributedHardware {
class IDistributedCameraSource : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedcamerasource");

    virtual int32_t InitSource(const std::string& params,
        const sptr<IDCameraSourceCallback>& callback) = 0;
    virtual int32_t ReleaseSource() = 0;
    virtual int32_t RegisterDistributedHardware(const std::string& devId,
        const std::string& dhId, const std::string& reqId,
        const EnableParam& param) = 0;
    virtual int32_t UnregisterDistributedHardware(const std::string& devId,
        const std::string& dhId, const std::string& reqId) = 0;
    virtual int32_t DCameraNotify(const std::string& devId,
        const std::string& dhId, std::string& events) = 0;
    virtual int32_t UpdateDistributedHardwareWorkMode(const std::string& devId,
        const std::string& dhId, const WorkModeParam& param) = 0;
};
} // namespace DistributedHardware
} // namespace OHOS
```

### SDK API 清单

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|------|------|--------|-----------|------|------|
| `InitSource` | params, callback | int32 | 异步回调 | 无 | 初始化源端 |
| `ReleaseSource` | - | int32 | 同步 | 无 | 释放源端 |
| `RegisterDistributedHardware` | devId, dhId, reqId, param | int32 | 异步回调 | ENABLE_DISTRIBUTED_HARDWARE | 注册分布式硬件 |
| `UnregisterDistributedHardware` | devId, dhId, reqId | int32 | 异步回调 | ENABLE_DISTRIBUTED_HARDWARE | 注销分布式硬件 |
| `DCameraNotify` | devId, dhId, events | int32 | 同步 | ENABLE_DISTRIBUTED_HARDWARE | 通知事件 |
| `UpdateDistributedHardwareWorkMode` | devId, dhId, param | int32 | 同步 | ENABLE_DISTRIBUTED_HARDWARE | 更新工作模式 |

### 回调接口

**文件**：`interfaces/inner_kits/native_cpp/camera_source/include/callback/idcamera_source_callback.h`

| 回调方法 | 参数 | 说明 |
|----------|------|------|
| `OnNotifyRegResult` | devId, dhId, reqId, status, data | 注册结果通知 |
| `OnNotifyUnregResult` | devId, dhId, reqId, status, data | 注销结果通知 |
| `OnHardwareStateChanged` | devId, dhId, status | 硬件状态变更 |
| `OnDataSyncTrigger` | devId | 数据同步触发 |

---

## Sink SDK 接口

### 接口定义

**文件**：`interfaces/inner_kits/native_cpp/camera_sink/include/idistributed_camera_sink.h`

```cpp
namespace OHOS {
namespace DistributedHardware {
class IDistributedCameraSink : public OHOS::IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.distributedhardware.distributedcamerasink");

    virtual int32_t InitSink(const std::string& params,
        const sptr<IDCameraSinkCallback>& sinkCallback) = 0;
    virtual int32_t ReleaseSink() = 0;
    virtual int32_t SubscribeLocalHardware(const std::string& dhId,
        const std::string& parameters) = 0;
    virtual int32_t UnsubscribeLocalHardware(const std::string& dhId) = 0;
    virtual int32_t StopCapture(const std::string& dhId) = 0;
    virtual int32_t ChannelNeg(const std::string& dhId,
        std::string& channelInfo) = 0;
    virtual int32_t GetCameraInfo(const std::string& dhId,
        std::string& cameraInfo) = 0;
    virtual int32_t OpenChannel(const std::string& dhId,
        std::string& openInfo) = 0;
    virtual int32_t CloseChannel(const std::string& dhId) = 0;
    virtual int32_t PauseDistributedHardware(const std::string& networkId) = 0;
    virtual int32_t ResumeDistributedHardware(const std::string& networkId) = 0;
    virtual int32_t StopDistributedHardware(const std::string& networkId) = 0;
    virtual int32_t SetAccessListener(const sptr<IAccessListener>& listener,
        int32_t timeOut, const std::string& pkgName) = 0;
    virtual int32_t RemoveAccessListener(const std::string& pkgName) = 0;
    virtual int32_t SetAuthorizationResult(const std::string& requestId,
        bool granted) = 0;
};
} // namespace DistributedHardware
} // namespace OHOS
```

### SDK API 清单

| 方法 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|------|------|--------|-----------|------|------|
| `InitSink` | params, callback | int32 | 异步回调 | 无 | 初始化被控端 |
| `ReleaseSink` | - | int32 | 同步 | 无 | 释放被控端 |
| `SubscribeLocalHardware` | dhId, parameters | int32 | 异步回调 | ACCESS_DISTRIBUTED_HARDWARE | 订阅本地硬件 |
| `UnsubscribeLocalHardware` | dhId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 取消订阅 |
| `StopCapture` | dhId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 停止采集 |
| `ChannelNeg` | dhId, channelInfo | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 通道协商 |
| `GetCameraInfo` | dhId, cameraInfo | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 获取相机信息 |
| `OpenChannel` | dhId, openInfo | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 打开通道 |
| `CloseChannel` | dhId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 关闭通道 |
| `PauseDistributedHardware` | networkId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 暂停硬件 |
| `ResumeDistributedHardware` | networkId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 恢复硬件 |
| `StopDistributedHardware` | networkId | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 停止硬件 |
| `SetAccessListener` | listener, timeOut, pkgName | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 设置访问监听 |
| `RemoveAccessListener` | pkgName | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 移除访问监听 |
| `SetAuthorizationResult` | requestId, granted | int32 | 同步 | ACCESS_DISTRIBUTED_HARDWARE | 设置授权结果 |

### 回调接口

**文件**：`interfaces/inner_kits/native_cpp/camera_sink/include/callback/idcamera_sink_callback.h`

| 回调方法 | 参数 | 说明 |
|----------|------|------|
| `OnNotifyResourceInfo` | type, subtype, networkId, isSensitive, isSameAccount | 资源信息通知 |
| `OnHardwareStateChanged` | devId, dhId, status | 硬件状态变更 |

---

## 错误码定义

**文件**：`common/include/constants/distributed_camera_errno.h`

| 错误码 | 值 | 含义 |
|--------|-----|------|
| `DCAMERA_OK` | 0 | 成功 |
| `ERR_DH_CAMERA_BASE` | 0x05C20000 | 错误基址 |
| `DCAMERA_MEMORY_OPT_ERROR` | BASE + 1 | 内存操作错误 |
| `DCAMERA_BAD_VALUE` | BASE + 2 | 无效参数值 |
| `DCAMERA_BAD_TYPE` | BASE + 3 | 无效类型 |
| `DCAMERA_ALREADY_EXISTS` | BASE + 4 | 已存在 |
| `DCAMERA_INIT_ERR` | BASE + 5 | 初始化错误 |
| `DCAMERA_NOT_FOUND` | BASE + 6 | 未找到 |
| `DCAMERA_WRONG_STATE` | BASE + 7 | 错误状态 |
| `DCAMERA_BAD_OPERATE` | BASE + 8 | 操作错误 |
| `DCAMERA_OPEN_CONFLICT` | BASE + 9 | 打开冲突 |
| `DCAMERA_DISABLE_PROCESS` | BASE + 10 | 禁用处理 |
| `DCAMERA_INDEX_OVERFLOW` | BASE + 11 | 索引溢出 |
| `DCAMERA_REGIST_HAL_FAILED` | BASE + 12 | 注册 HAL 失败 |
| `DCAMERA_UNREGIST_HAL_FAILED` | BASE + 13 | 注销 HAL 失败 |
| `DCAMERA_ALLOC_ERROR` | BASE + 14 | 分配错误 |
| `DCAMERA_DEVICE_BUSY` | BASE + 15 | 设备忙 |
| `DCAMERA_ERR_APPLY_RESULT` | BASE + 16 | 应用结果错误 |
| `DCAMERA_ERR_DLOPEN` | BASE + 17 | 动态库打开错误 |
| `DCAMERA_ERR_PUBLISH_STATE` | BASE + 18 | 发布状态错误 |
| `DCAMERA_ERR_ALLCONNECT` | BASE + 19 | AllConnect 错误 |
| `DCAMERA_TRANS_BUSY` | BASE + 20 | 传输忙 |

---

## 内部模块接口

### Channel 模块接口

| 接口 | 文件 | 说明 |
|------|------|------|
| `IDCameraChannel` | `dcamera_channel.h` | 通道抽象接口 |
| `DCameraChannelSourceImpl` | `dcamera_channel_source_impl.cpp` | Source 通道实现 |
| `DCameraChannelSinkImpl` | `dcamera_channel_sink_impl.cpp` | Sink 通道实现 |
| `DCameraSoftbusAdapter` | `dcamera_softbus_adapter.cpp` | 软总线适配器 |
| `DCameraSoftbusSession` | `dcamera_softbus_session.cpp` | 会话管理 |

### DataProcess 模块接口

| 接口 | 文件 | 说明 |
|------|------|------|
| `IDataProcess` | `abstract_data_process.h` | 处理流水线抽象 |
| `DCameraPipelineSource` | `dcamera_pipeline_source.cpp` | Source 流水线 |
| `DCameraPipelineSink` | `dcamera_pipeline_sink.cpp` | Sink 流水线 |

---

## 依赖方向

```
┌─────────────────────────────────────────────────────────────────┐
│                     依赖方向 (依赖 → 被依赖)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   common/utils ──► channel ──► sourceservice                    │
│        │                │                                       │
│        │                └──► sinkservice                        │
│        │                                                         │
│        └──────────────► data_process                             │
│                          │                                       │
│                          └──► *service                           │
│                                                                  │
│   cameraoperator ──► *service                                   │
│   (client/handler)                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**关键依赖**：
- `common/utils` → 所有模块
- `channel` → `sourceservice`, `sinkservice`
- `data_process` → `sourceservice`, `sinkservice`
- `cameraoperator/*` → `sourceservice`, `sinkservice`

---

## 稳定性标注

| 接口/模块 | 稳定性 | 说明 |
|-----------|--------|------|
| **IDistributedCameraSource** | **高** | SDK 接口，版本兼容 |
| **IDistributedCameraSink** | **高** | SDK 接口，版本兼容 |
| **IDCameraSourceCallback** | **高** | 回调接口，稳定 |
| **IDCameraSinkCallback** | **高** | 回调接口，稳定 |
| **DCameraSourceController** | **中** | 内部控制层，可能演进 |
| **DCameraSinkController** | **中** | 内部控制层，可能演进 |
| **IDCameraChannel** | **中** | 通道抽象，可能优化 |
| **IDataProcess** | **中** | 处理抽象，可能演进 |

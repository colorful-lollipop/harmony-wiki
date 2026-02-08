# 配置标志和宏

## 目的

本文档记录 Request 项目的关键配置标志、宏定义和 Feature Flags。

## 适用范围

- GN 构建配置
- 运行时配置
- 条件编译宏
- Feature Flags

## 关键结论

1. **Feature Flags**: 控制可选功能（如电话服务）
2. **配置文件**: SA profile、服务配置
3. **编译宏**: 版本控制、调试标志
4. **运行时参数**: 系统属性、命令行参数

---

## GN 构建标志

### request_aafwk.gni

**文件**: `request_aafwk.gni`

| 标志 | 类型 | 默认值 | 描述 |
|-----|------|--------|------|
| `request_telephony_core_service` | boolean | false | 是否包含电话核心服务支持 |
| `request_telephony_cellular_data` | boolean | false | 是否包含蜂窝数据支持 |

**使用方式**:
```gni
# 在 BUILD.gn 中使用
import("//base/request/request/request_aafwk.gni")

if (request_telephony_core_service) {
  deps += [ "//foundation/telephony/core_service:tel_core_service_client" ]
}
```

---

## 服务配置

### SA Profile

**文件**: `etc/sa_profile/3706.json`

```json
{
  "process": "download_server",
  "systemability": [{
    "name": 3706,
    "libpath": "libdownload_server.dylib.so",
    "run-on-create": false,
    "distributed": false,
    "dump_level": 1,
    "recycle-strategy": "low-memory",
    "start-on-demand": {
      "allow-update": true,
      "commonevent": [{
        "name": "usual.event.USER_REMOVED"
      }]
    },
    "stop-on-demand": {
      "param": [{
        "name": "resourceschedule.memmgr.low.memory.prepare",
        "value": "true"
      }]
    }
  }]
}
```

**配置项说明**:

| 配置项 | 值 | 描述 |
|-------|-----|------|
| name | 3706 | System Ability ID |
| libpath | libdownload_server.dylib.so | 库文件名 |
| run-on-create | false | 不在启动时自动运行 |
| distributed | false | 不跨设备 |
| dump_level | 1 | Dump 级别 |
| recycle-strategy | low-memory | 低内存回收策略 |
| start-on-demand.allow-update | true | 允许按需启动和更新 |
| start-on-demand.commonevent | usual.event.USER_REMOVED | 启动事件：用户移除 |
| stop-on-demand.param | resourceschedule.memmgr.low.memory.prepare | 停止参数：内存压力准备 |

---

### init 配置

**文件**: `etc/init/downloadservice.cfg`

```cfg
{
  "services" : [{
      "name" : "download_server",
      "path" : ["/system/bin/download_server"],
      "uid" : "system",
      "writepid" : "/data/service/el0/download_server.pid",
      "secon" : true,
      "sandbox" : 0,
      "cpu" : 0,
      "iplimit" : 1024,
      "socket" : 32,
      "oom_score_adj" : -200,
      "permission" : [
          "ohos.permission.RUNNING_STATE_OBSERVER",
          "ohos.permission.GET_NETWORK_INFO",
          "ohos.permission.CONNECTIVITY_INTERNAL",
          "ohos.permission.SEND_TASK_COMPLETE_EVENT",
          "ohos.permission.ACCESS_CERT_MANAGER",
          "ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS",
          "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
          "ohos.permission.GET_DISTRIBUTED_ACCOUNTS",
          "ohos.permission.GET_RUNNING_INFO",
          "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
          "ohos.permission.INTERNET",
          "ohos.permission.READ_IMAGEVIDEO",
          "ohos.permission.WRITE_IMAGEVIDEO"
      ],
      "apl" : "system_core",
      "start-mode" : "normal"
  }]
}
```

**权限说明**:

| 权限 | 用途 |
|-------|------|
| `RUNNING_STATE_OBSERVER` | 观察运行状态 |
| `GET_NETWORK_INFO` | 获取网络信息 |
| `CONNECTIVITY_INTERNAL` | 内部连接性访问 |
| `SEND_TASK_COMPLETE_EVENT` | 发送任务完成事件 |
| `ACCESS_CERT_MANAGER` | 访问证书管理器 |
| `INTERNET` | 网络访问 |
| `READ_IMAGEVIDEO` | 读视频 |
| `WRITE_IMAGEVIDEO` | 写视频 |

---

## 编译宏和常量

### 版本控制

```cpp
// common/include/constant.h
enum class Version {
    API8 = 8,
    API9 = 9,
    API10 = 10
};
```

### 错误码

```cpp
// 常见错误码
#define E_OK                0
#define E_PERMISSION         201
#define E_NOT_SYSTEM_APP   202
#define E_PARAMETER_CHECK   202
#define E_UNSUPPORTED        203
#define E_FILE_IO          204
#define E_FILE_PATH        205
#define E_SERVICE_ERROR    206
#define E_OTHER             207
```

**证据**: `common/include/constant.h`

---

### 网络常量

```cpp
// 网络类型
#define NETWORK_MOBILE       1
#define NETWORK_WIFI          2
```

```cpp
// 任务状态
#define PAUSED_QUEUED_FOR_WIFI        0
#define PAUSED_WAITING_FOR_NETWORK      1
#define PAUSED_WAITING_TO_RETRY         2
#define PAUSED_BY_USER                 3
#define PAUSED_UNKNOWN                 4
```

**证据**: `common/include/constant.h`

---

### 下载错误码

```cpp
// 下载相关错误
#define ERROR_CANNOT_RESUME            0
#define ERROR_DEVICE_NOT_FOUND           1
#define ERROR_FILE_ALREADY_EXISTS       2
#define ERROR_FILE_ERROR               3
#define ERROR_HTTP_DATA_ERROR          4
#define ERROR_INSUFFICIENT_SPACE       5
#define ERROR_TOO_MANY_REDIRECTS        6
#define ERROR_UNHANDLED_HTTP_CODE     7
#define ERROR_UNKNOWN                 8
#define ERROR_OFFLINE                 9
#define ERROR_UNSUPPORTED_NETWORK_TYPE 10
```

**证据**: `common/include/constant.h`

---

## IPC 命令码

### RequestServiceInterface

```cpp
// frameworks/native/request/include/download_server_ipc_interface_code.h
enum RequestInterfaceCode {
    CMD_CONSTRUCT      = 0,
    CMD_PAUSE         = 1,
    CMD_QUERY         = 2,
    CMD_QUERYMIMETYPE = 3,
    CMD_REMOVE        = 4,
    CMD_RESUME        = 5,
    CMD_START         = 6,
    CMD_STOP          = 7,
    CMD_SHOW          = 8,
    CMD_TOUCH         = 9,
    CMD_SEARCH        = 10,
    CMD_GETTASK       = 11,
    CMD_OPENCHANNEL   = 13,
    CMD_SUBSCRIBE     = 14,
    CMD_UNSUBSCRIBE   = 15,
    CMD_SUB_RUNCOUNT  = 16,
    CMD_UNSUB_RUNCOUNT = 17,
    CMD_CREATE_GROUP  = 18,
    CMD_ATTACH_GROUP  = 19,
    CMD_DELETE_GROUP  = 20,
    CMD_SET_MAX_SPEED = 21,
    CMD_SET_MODE     = 100,
    CMD_DISABLE_TASK_NOTIFICATIONS = 101
};
```

**证据**: `services/src/service/interface.rs`

---

## 系统能力

### 系统能力定义

| 能力 ID | 描述 | 状态 |
|---------|------|------|
| `SystemCapability.MiscServices.Download` | 下载能力 | 可用 |
| `SystemCapability.MiscServices.Upload` | 上传能力 | 可用 |
| `SystemCapability.Request.FileTransferAgent` | 文件传输代理 | 可用 |

**证据**: `bundle.json:15-19`

---

## 系统事件

### HiSysEvent 定义

**文件**: `hisysevent.yaml`

```yaml
domain: REQUEST_SERVICE
name: DOWNLOAD_SERVICE
events:
  - name: CREATE_TASK
    type: STATISTIC
    level: MINOR
  - name: DELETE_TASK
    type: STATISTIC
    level: MINOR
  - name: FAULT_EVENT
    type: FAULT
    level: CRITICAL
  - name: STANDARD_FAULT
    type: FAULT
    level: MAJOR
```

**事件类型**:

| 事件类型 | 级别 | 触发时机 |
|---------|------|----------|
| CREATE_TASK | MINOR | 任务创建 |
| DELETE_TASK | MINOR | 任务删除 |
| FAULT_EVENT | CRITICAL | 故障发生 |
| STANDARD_FAULT | MAJOR | 标准故障 |

**证据**: `hisysevent.yaml`, `common/sys_event/src/cxx/common_event.cpp`

---

## 调试标志

### 日志宏

```cpp
// common/include/log.h
#define REQUEST_HILOGD(fmt, ...) HiLog::Label(LOG_CORE, DEBUG, fmt, ##__VA_ARGS__)
#define REQUEST_HILOGI(fmt, ...) HiLog::Label(LOG_CORE, INFO, fmt, ##__VA_ARGS__)
#define REQUEST_HILOGW(fmt, ...) HiLog::Label(LOG_CORE, WARN, fmt, ##__VA_ARGS__)
#define REQUEST_HILOGE(fmt, ...) HiLog::Label(LOG_CORE, ERROR, fmt, ##__VA_ARGS__)
#define REQUEST_HILOGF(fmt, ...) HiLog::Label(LOG_CORE, FATAL, fmt, ##__VA_ARGS__)
```

---

## 运行时参数

### 环境变量

| 变量 | 用途 | 示例 |
|-------|------|------|
| `HARNESS_SA_ROOT` | SA 根目录 | `/system/profile/` |
| `HARNESS_STORAGE` | 存储路径 | `/data/` |
| `HARNESS_STORAGE2` | 辅助存储 | `/data/storage/` |

### 系统属性

| 属性 | 用途 |
|-------|------|
| `persist.request.download.taskcount` | 下载任务计数 |
| `ro.config.request` | 请求配置 |

---

## 相关跳转

- [GN Targets](05_GN_Targets.md) - 构建配置
- [对外 N-API](03_NAPI_JS_API.md) - API 常量
- [安全风险评审](07_Security_Review.md) - 安全配置

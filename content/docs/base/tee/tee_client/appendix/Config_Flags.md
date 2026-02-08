# 附录：关键编译宏与 Feature Flags

## 1. 概述

本文档记录 TEE Client 组件中所有重要的编译宏、Feature Flags 和配置项。

## 2. 全局 Feature Flags

### 2.1 tee_client_features_tui

| 属性 | 值 |
|------|-----|
| **宏名称** | `tee_client_features_tui` |
| **定义位置** | `tee_client.gni` |
| **默认值** | `false` |
| **类型** | boolean |

**作用**：
启用 TUI (Trusted User Interface) 功能。

**影响范围**：

| 模块 | 变化 |
|------|------|
| cadaemon | 编译 `libcadaemon_tui.so` |
| cadaemon | 创建 TUI 专用线程 |
| cadaemon | 启用 TUI 事件监听 |

**启用方式**：
```gn
# 在产品 gn 文件中
declare_args() {
    tee_client_features_tui = true
}
```

**相关代码**：
```
services/cadaemon/build/standard/BUILD.gn:58-60
services/cadaemon/src/ca_daemon/cadaemon_service.cpp:89-94
```

---

## 3. 框架层编译宏

### 3.1 libteec.so 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| 无特定宏 | - | - | 使用标准 NDK 接口 |

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "hilog:libhilog",
  "ipc:ipc_single",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
]
```

---

### 3.2 libteec_vendor.so 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `LIB_TEEC_VENDOR` | 定义 | `BUILD.gn:66` | 标识厂商实现 |
| `CONFIG_LOG_REPORT` | 定义 | `BUILD.gn:67` | 启用 Hisysevent 日志上报 |

**LIB_TEEC_VENDOR**：
```c
// 使用示例
#ifdef LIB_TEEC_VENDOR
// 厂商特定代码路径
#else
// 标准代码路径
#endif
```

**CONFIG_LOG_REPORT**：
```c
#ifdef CONFIG_LOG_REPORT
// 启用 Hisysevent 日志上报
HILOG_ERROR(HiSysEvent:: domain_tee_client, "TEEC_ERROR");
#endif
```

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "hilog:libhilog",
  "hisysevent:libhisysevent",
]
```

---

## 4. 服务层编译宏

### 4.1 teecd 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `CONFIG_FSWORK_THREAD_ELEVATE_PRIO` | 定义 | `BUILD.gn:49` | 提升 FS Agent 线程优先级 |
| `DYNAMIC_DRV_DIR` | `"/vendor/bin/tee_dynamic_drv/"` | `BUILD.gn:50` | 动态驱动目录 |
| `DYNAMIC_SRV_DIR` | `"/vendor/bin/tee_dynamic_srv/"` | `BUILD.gn:51` | 动态服务目录 |
| `CONFIG_LATE_INIT` | 定义 | `BUILD.gn:52` | 启用延迟初始化 |
| `ENABLE_FDSAN_CHECK` | 定义 | `BUILD.gn:53` | 启用文件描述符泄漏检测 |
| `DYNAMIC_SRV_FEIMA_DIR` | `"/vendor/etc/passthrough/teeos/dynamic_srv"` | `BUILD.gn:54` | FEIMA 动态服务目录 |
| `DYNAMIC_DRV_FEIMA_DIR` | `"/vendor/etc/passthrough/teeos/dynamic_drv"` | `BUILD.gn:55` | FEIMA 动态驱动目录 |

**CONFIG_FSWORK_THREAD_ELEVATE_PRIO**：
```c
#ifdef CONFIG_FSWORK_THREAD_ELEVATE_PRIO
// 提升线程优先级
pthread_attr_t attr;
struct sched_param param;
pthread_attr_init(&attr);
pthread_attr_getschedparam(&attr, &param);
param.sched_priority = PRIORITY_MAX;
pthread_attr_setschedparam(&attr, &param);
#endif
```

**DYNAMIC_DRV_DIR / DYNAMIC_SRV_DIR**：
```c
// 动态加载路径
#define DYNAMIC_DRV_DIR "/vendor/bin/tee_dynamic_drv/"
#define DYNAMIC_SRV_DIR "/vendor/bin/tee_dynamic_srv/"
```

---

### 4.2 cadaemon 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `ENABLE_FDSAN_CHECK` | 条件定义 | `BUILD.gn:28` | 启用 FDSAN 检测 |
| `CONFIG_LOG_REPORT` | 条件定义 | `BUILD.gn:29` | 启用日志上报 |

**条件编译**：
```gn
if (component_type == "system") {
  defines += [
    "ENABLE_FDSAN_CHECK",
    "CONFIG_LOG_REPORT",
  ]
}
```

---

### 4.3 libcadaemon_tui.so 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `FONT_HASH_VAL` | MD5 哈希值 | `BUILD.gn:96-102` | TUI 字体文件哈希 |
| `ENABLE_FDSAN_CHECK` | 定义 | `BUILD.gn:104` | 启用 FDSAN 检测 |
| `SCENE_BOARD_ENABLE` | 定义 | `BUILD.gn:140` | 启用场景板支持 |

**FONT_HASH_VAL**：
```gn
hash_string = "8978e05044e7089ad6a9de38c505c8148305607983487435a916d2610700a7ca"

if (hash_string != "") {
  defines += [ "FONT_HASH_VAL=\"$hash_string\"" ]
} else {
  defines += [ "FONT_HASH_VAL=\"hash_string can not be set\"" ]
}
```

---

### 4.4 tlogcat 编译宏

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `TEE_LOG_PATH_BASE` | `"/data/log"` | `BUILD.gn` | TEE 日志基础路径 |
| `CONFIG_TLOGCAT_TAG` | 定义 | `BUILD.gn` | 启用日志标签 |
| `CONFIG_TEE_PRIVATE_LOGFILE` | 定义 | `BUILD.gn` | 启用私有日志文件 |
| `ENABLE_FDSAN_CHECK` | 定义 | `BUILD.gn` | 启用 FDSAN 检测 |

**TEE_LOG_PATH_BASE**：
```c
#define TEE_LOG_PATH_BASE "/data/log"

// 使用示例
char logPath[PATH_MAX];
snprintf(logPath, sizeof(logPath), "%s/tee/", TEE_LOG_PATH_BASE);
```

---

## 5. 头文件中的配置宏

### 5.1 限制常量

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `MAX_CXTCNT_ONECA` | 16 | `tee_client_inner.h:68` | 每个 CA 最多上下文数 |
| `MAX_TA_PATH_LEN` | 256 | `tee_client_inner.h:69` | TA 路径最大长度 |
| `PARAM_SIZE_LIMIT` | 0x400000 + 0x100 | `tee_client_inner.h:70` | 参数大小限制 |
| `NUM_OF_SHAREMEM_BITMAP` | 8 | `tee_client_inner.h:71` | 共享内存位图数量 |
| `BUFF_LEN_MAX` | 4096 | `tee_client_inner.h:77` | 缓冲区最大长度 |
| `MAX_SHAREDMEM_LEN` | 0x10000000 | `tee_client_inner.h:41` | 共享内存最大长度 |

**代码证据**：
```c
// tee_client_inner.h
#define MAX_CXTCNT_ONECA 16                 /* one ca only can get 16 contexts */
#define MAX_TA_PATH_LEN 256
#define PARAM_SIZE_LIMIT (0x400000 + 0x100) /* 0x100 is for share mem flag etc */
#define NUM_OF_SHAREMEM_BITMAP 8
#define BUFF_LEN_MAX 4096
#define MAX_SHAREDMEM_LEN 0x10000000
```

---

### 5.2 参数类型宏

| 宏名称 | 定义位置 | 说明 |
|--------|----------|------|
| `TEEC_PARAM_TYPES` | `tee_client_api.h:52` | 构建参数类型值 |
| `TEEC_PARAM_TYPE_GET` | `tee_client_api.h:60` | 获取指定索引的参数类型 |
| `IS_TEMP_MEM` | `tee_client_inner.h:27` | 判断临时内存类型 |
| `IS_PARTIAL_MEM` | `tee_client_inner.h:31` | 判断部分内存类型 |
| `IS_VALUE_MEM` | `tee_client_inner.h:35` | 判断值类型 |
| `IS_SHARED_MEM` | `tee_client_inner.h:38` | 判断共享内存类型 |

**使用示例**：
```c
// 构建参数类型
operation.paramTypes = TEEC_PARAM_TYPES(
    TEEC_VALUE_INPUT,
    TEEC_MEMREF_TEMP_OUTPUT,
    TEEC_NONE,
    TEEC_NONE
);

// 获取参数类型
uint32_t type = TEEC_PARAM_TYPE_GET(operation.paramTypes, 0);

// 判断参数类型
if (IS_TEMP_MEM(type)) {
    // 处理临时内存
}
```

---

## 6. 设备路径配置

### 6.1 TEE 设备节点

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `TC_NS_CLIENT_DEV_NAME` | `"/dev/tc_ns_client"` | `tc_ns_client.h` | TEE 客户端设备 |
| `TC_PRIVATE_DEV_NAME` | `"/dev/tc_private"` | `tc_ns_client.h` | 私有设备 |

**代码证据**：
```c
// tc_ns_client.h
#define TC_NS_CLIENT_DEV_NAME "/dev/tc_ns_client"
#define TC_PRIVATE_DEV_NAME "/dev/tc_private"
```

---

### 6.2 Socket 路径

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `TC_NS_SOCKET_NAME` | `"#tc_ns_socket"` | `tee_ca_daemon.c` | 抽象 Socket 名称 |

**代码证据**：
```c
// tee_ca_daemon.c
#define TC_NS_SOCKET_NAME "#tc_ns_socket"
```

---

### 6.3 TA 加载路径

| 宏名称 | 值 | 定义位置 | 说明 |
|--------|------|----------|------|
| `TEE_FEIMA_DEFAULT_PATH` | `"/vendor/etc/passthrough/teeos/ta"` | `tee_client_app_load.c` | FEIMA TA 默认路径 |
| `TEE_DEFAULT_PATH` | `"/vendor/bin"` | `tee_client_app_load.c` | TA 默认路径 |

**优先级**：
1. 用户指定的 `taPath`
2. `TEE_FEIMA_DEFAULT_PATH`
3. `TEE_DEFAULT_PATH`

---

## 7. Agent ID 定义

### 7.1 Agent 标识符

| Agent | ID | 值 | 定义位置 |
|-------|-----|------|----------|
| FS Agent | `AGENT_FS_ID` | `0x46536673` | `tee_agent.h` |
| MISC Agent | `AGENT_MISC_ID` | `0x4D495343` | `tee_agent.h` |
| Secfile Load Agent | `SECFILE_LOAD_AGENT_ID` | `0x4C4F4144` | `tee_agent.h` |

**ID 编码**：
```c
// FS Agent: "FSfs"
#define AGENT_FS_ID     0x46536673  // 'F'='0x46', 'S'='0x53', ...

// MISC Agent: "MISC"
#define AGENT_MISC_ID   0x4D495343  // 'M'='0x4D', ...

// Secfile Load Agent: "LOAD"
#define SECFILE_LOAD_AGENT_ID 0x4C4F4144  // 'L'='0x4C', ...
```

---

## 8. ioctl 命令定义

### 8.1 TC_NS_CLIENT ioctl 命令

| 命令 | 值 | 用途 |
|------|------|------|
| `TC_NS_CLIENT_IOCTL_SES_OPEN_REQ` | `_IOW('t', 1)` | 打开会话 |
| `TC_NS_CLIENT_IOCTL_SES_CLOSE_REQ` | `_IOWR('t', 2)` | 关闭会话 |
| `TC_NS_CLIENT_IOCTL_SEND_CMD_REQ` | `_IOWR('t', 3)` | 发送命令 |
| `TC_NS_CLIENT_IOCTL_LOAD_APP_REQ` | `_IOWR('t', 9)` | 加载应用 |
| `TC_NS_CLIENT_IOCTL_CANCEL_CMD_REQ` | `_IOWR('t', 13)` | 取消命令 |
| `TC_NS_CLIENT_IOCTL_REGISTER_AGENT` | `_IOWR('t', 7)` | 注册 Agent |
| `TC_NS_CLIENT_IOCTL_UNREGISTER_AGENT` | `_IOWR('t', 8)` | 注销 Agent |
| `TC_NS_CLIENT_IOCTL_WAIT_EVENT` | - | 等待事件 |
| `TC_NS_CLIENT_IOCTL_SEND_EVENT_RESPONSE` | - | 发送事件响应 |
| `TC_NS_CLIENT_IOCTL_LOGIN` | - | CA 登录认证 |
| `TC_NS_CLIENT_IOCTL_SYC_SYS_TIME` | - | 同步系统时间 |
| `TC_NS_CLIENT_IOCTL_LATEINIT` | - | 延迟初始化 |
| `TC_NS_CLIENT_IOCTL_GET_TEE_VERSION` | - | 获取 TEE 版本 |
| `TC_NS_CLIENT_IOCTL_TUI_EVENT` | - | TUI 事件 |

**代码证据**：
```c
// tee_ioctl_cmd.h
#define TC_NS_CLIENT_IOCTL_SES_OPEN_REQ    _IOW('t', 1)
#define TC_NS_CLIENT_IOCTL_SES_CLOSE_REQ   _IOWR('t', 2)
#define TC_NS_CLIENT_IOCTL_SEND_CMD_REQ   _IOWR('t', 3)
// ...
```

---

## 9. 日志配置

### 9.1 日志标签

| 模块 | 标签 | 定义位置 |
|------|------|----------|
| teecd | `"teecd"` | `tee_ca_daemon.c:39` |
| tee_client | - | `tee_log.h` |

**日志级别**：
```c
// tee_log.h
#define LOG_LEVEL_ERROR  0
#define LOG_LEVEL_WARN   1
#define LOG_LEVEL_INFO    2
#define LOG_LEVEL_DEBUG   3

// 日志宏定义
#define tloge(fmt, ...)   // Error
#define tlogw(fmt, ...)  // Warn
#define tlogi(fmt, ...)  // Info
#define tlogd(fmt, ...)  // Debug
```

---

## 10. 认证配置

### 10.1 CA 类型

| 类型 | 值 | 定义位置 |
|------|------|----------|
| `SYSTEM_CA` | 1 | `tee_auth_common.h` |
| `VENDOR_CA` | 2 | `tee_auth_common.h` |
| `APP_CA` | 3 | `tee_auth_common.h` |
| `SA_CA` | 4 | `tee_auth_common.h` |

**代码证据**：
```c
// tee_auth_common.h
typedef enum {
    SYSTEM_CA = 1,   // 系统 CA
    VENDOR_CA,       // 厂商 CA
    APP_CA,          // 应用 CA
    SA_CA,           // 系统 Ability CA
    MAX_CA,
} CaType;
```

---

## 11. 存储路径配置

### 11.1 安全存储路径

| 路径 | 用途 | 定义 |
|------|------|------|
| `/sec_storage/` | 持久化存储根目录 | 运行时 |
| `/data/sec_storage_data/` | 临时存储根目录 | 运行时 |
| `/data/sec_storage_data_users/{userid}/` | 用户隔离存储 | 运行时 |
| `/data/service/el2/{userid}/tee/sec_storage_data/` | CE 存储 | 运行时 |

### 11.2 文件权限

| 权限 | 值 | 用途 |
|------|------|------|
| `SFS_DIR_PERM` | 0700 | 安全存储目录权限 |
| `SFS_FILE_PERM` | 0600 | 安全存储文件权限 |

---

## 12. 错误码配置

### 12.1 错误码基址

| 区域 | 基址 | 用途 |
|------|------|------|
| 标准错误 | `0xFFFF0000` | 通用错误码范围 |
| TA 错误 | `0xFFFF3000` | TA 错误码范围 |
| 内部错误 | `0xFFFF9000` | 内部使用 |

### 12.2 常用错误码

| 错误码 | 值 | 说明 |
|--------|------|------|
| `TEEC_SUCCESS` | 0x0 | 成功 |
| `TEEC_ERROR_GENERIC` | 0xFFFF0000 | 通用错误 |
| `TEEC_ERROR_ACCESS_DENIED` | 0xFFFF0001 | 访问被拒绝 |
| `TEEC_ERROR_BAD_PARAMETERS` | 0xFFFF0006 | 参数错误 |
| `TEEC_ERROR_TARGET_DEAD` | 0xFFFF3024 | TA 崩溃 |
| `TEE_ERROR_CA_AUTH_FAIL` | 0xFFFFCFE5 | CA 认证失败 |
| `TEE_ERROR_RETRY_OPEN_SESSION` | 0xFFFF920E | 会话重试 |

---

## 13. 系统能力 ID

### 13.1 SA ID 配置

| 服务 | ID | 定义位置 |
|------|------|----------|
| CaDaemonService | 8001 | `services/cadaemon/build/standard/sa_profile/8001.json` |

**配置文件**：
```json
{
    "name": "CaDaemonService",
    "libpath": "/system/lib/libcadaemon.z.so",
    "run-on-create": true,
    "start-mode": "boot"
}
```

---

## 14. 条件编译示例

### 14.1 组件类型条件

```gn
if (component_type == "system") {
  defines += [
    "ENABLE_FDSAN_CHECK",
    "CONFIG_LOG_REPORT",
  ]
  
  sources += [
    "../../../services/authentication/tee_auth_system.cpp",
  ]
}
```

### 14.2 TUI 功能条件

```c
#ifdef CONFIG_TEE_CLIENT_TUI
// TUI 相关代码
void CreateTuiThread() {
    std::thread tuiThread(TeeTuiThreadWork);
    tuiThread.detach();
}
#endif
```

---

## 15. 链接配置

### 15.1 链接选项

| 选项 | 值 | 用途 |
|------|------|------|
| `-Wl,-z,max-page-size=4096` | 页大小 | 优化内存布局 |
| `-Wl,-z,separate-code` | 代码分离 | 安全增强 |

**代码证据**：
```gn
ldflags = [
  "-Wl,-z,max-page-size=4096",
  "-Wl,-z,separate-code",
]
```

---

## 16. CFI 配置

### 16.1 Control Flow Integrity

| 属性 | 值 | 定义位置 |
|------|------|----------|
| cfi | true | `BUILD.gn` |
| cfi_cross_dso | true | `BUILD.gn` |

**代码证据**：
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

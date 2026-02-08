# Inner API 接口文档

> **重要说明**: appspawn 是**纯Native C/C++组件**，不提供JavaScript/N-API接口。所有对外接口均为C语言Inner API。

## 目录

- [概述](#概述)
- [主API (appspawn.h)](#主api-appspawnh)
- [客户端API (appspawn_client.h)](#客户端api-appspawn_clienth)
- [HNP API (hnp_api.h)](#hnp-api-hnp_apih)
- [消息格式](#消息格式)
- [错误码](#错误码)

## 概述

appspawn 提供三类 Inner API：

1. **客户端API**: 用于连接appspawn服务并发送孵化请求
2. **HNP API**: 用于原生包(Native Package)的安装/卸载
3. **DEC API**: 用于沙箱路径管理

### API 头文件位置

| API类型 | 头文件 | 位置 |
|---------|--------|------|
| 主API | `appspawn.h` | `interfaces/innerkits/include/` |
| 客户端API | `appspawn_client.h` | `interfaces/innerkits/client/` |
| HNP API | `hnp_api.h` | `interfaces/innerkits/hnp/include/` |
| DEC API | `dec_api.h` | `interfaces/innerkits/dec_util/include/` |

## 主API (appspawn.h)

> 文件: `interfaces/innerkits/include/appspawn.h`

### 常量定义

```c
#define APPSPAWN_SERVER_NAME "appspawn"
#define NWEBSPAWN_SERVER_NAME "nwebspawn"
#define CJAPPSPAWN_SERVER_NAME "cjappspawn"
#define NATIVESPAWN_SERVER_NAME "nativespawn"
#define HYBRIDSPAWN_SERVER_NAME "hybridspawn"

#define APP_MAX_GIDS 64
#define APP_USER_NAME 64
#define APP_MAX_FD_COUNT 16
```

### 数据结构

#### AppDacInfo - DAC权限信息
```c
typedef struct {
    uint32_t uid;       // Unix uid
    uint32_t gid;       // Unix gid
    uint32_t gidCount;  // gid表数量
    uint32_t gidTable[APP_MAX_GIDS];
    char userName[APP_USER_NAME];
} AppDacInfo;
```

#### AppSpawnResult - 孵化结果
```c
typedef struct {
    int result;         // 错误码
    pid_t pid;         // 子进程PID
} AppSpawnResult;
```

#### AppSpawnReqMsgHandle - 请求消息句柄
```c
typedef void *AppSpawnReqMsgHandle;
```

#### AppSpawnClientHandle - 客户端句柄
```c
typedef void *AppSpawnClientHandle;
```

### 消息类型枚举

```c
typedef enum {
    MSG_APP_SPAWN = 0,                    // 普通应用孵化
    MSG_GET_RENDER_TERMINATION_STATUS,    // 获取渲染终止状态
    MSG_SPAWN_NATIVE_PROCESS,             // 孵化Native进程
    MSG_DUMP,                             //Dump
    MSG_BEGET_CMD,                        // Beget命令
    MSG_BEGET_SPAWNTIME,                  // Beget启动时间
    MSG_UPDATE_MOUNT_POINTS,              // 更新挂载点
    MSG_RESTART_SPAWNER,                  // 重启Spawner
    MSG_DEVICE_DEBUG,                      // 设备调试
    MSG_UNINSTALL_DEBUG_HAP,               // 卸载调试HAP
    MSG_LOCK_STATUS,                       // 锁状态
    MSG_OBSERVE_PROCESS_SIGNAL_STATUS,    // 观察进程信号状态
    MSG_UNLOAD_WEBLIB_IN_APPSPAWN,        // 卸载Web库
    MSG_LOAD_WEBLIB_IN_APPSPAWN,          // 加载Web库
    MAX_TYPE_INVALID
} AppSpawnMsgType;
```

### 应用标志枚举

```c
typedef enum {
    APP_FLAGS_COLD_BOOT = 0,              // 冷启动
    APP_FLAGS_BACKUP_EXTENSION,           // 备份扩展
    APP_FLAGS_DLP_MANAGER,                // DLP管理器
    APP_FLAGS_DEBUGGABLE,                 // 可调试
    APP_FLAGS_ASANENABLED,                // ASAN启用
    APP_FLAGS_ACCESS_BUNDLE_DIR,          // 访问Bundle目录
    APP_FLAGS_NATIVEDEBUG,                // Native调试
    APP_FLAGS_NO_SANDBOX,                 // 无沙箱
    APP_FLAGS_OVERLAY,                    // Overlay
    APP_FLAGS_BUNDLE_RESOURCES,           // Bundle资源
    APP_FLAGS_GWP_ENABLED_FORCE,          // GWP强制启用
    APP_FLAGS_GWP_ENABLED_NORMAL,         // GWP普通启用
    APP_FLAGS_TSAN_ENABLED,               // TSAN启用
    APP_FLAGS_IGNORE_SANDBOX,             // 忽略沙箱结果
    APP_FLAGS_ISOLATED_SANDBOX,           // 隔离沙箱
    APP_FLAGS_EXTENSION_SANDBOX,          // 扩展沙箱
    APP_FLAGS_CLONE_ENABLE,               // 克隆启用
    APP_FLAGS_DEVELOPER_MODE,             // 开发者模式
    APP_FLAGS_BEGETCTL_BOOT,              // Begetctl启动
    APP_FLAGS_ATOMIC_SERVICE,             // 原子服务
    APP_FLAGS_CHILDPROCESS,               // 子进程
    APP_FLAGS_HWASAN_ENABLED,             // HWASAN启用
    APP_FLAGS_UBSAN_ENABLED,              // UBSAN启用
    APP_FLAGS_ISOLATED_SANDBOX_TYPE,      // 隔离沙箱类型
    APP_FLAGS_ISOLATED_SELINUX_LABEL,     // SELinux标签
    APP_FLAGS_ISOLATED_SECCOMP_TYPE,     // Seccomp类型
    APP_FLAGS_ISOLATED_NETWORK,           // 隔离网络
    APP_FLAGS_ISOLATED_DATAGROUP,         // 隔离数据组
    APP_FLAGS_TEMP_JIT,                   // 临时JIT
    APP_FLAGS_PRE_INSTALLED_HAP,         // 预安装HAP
    APP_FLAGS_GET_ALL_PROCESSES,         // 获取所有进程
    APP_FLAGS_CUSTOM_SANDBOX,             // 自定义沙箱
    APP_FLAGS_SET_CAPS_FOWNER,            // 设置Capability
    APP_FLAGS_ALLOW_IOURING,              // 允许IO_URING
    APP_FLAGS_UNLOCKED_STATUS,            // 解锁状态
    APP_FLAGS_FILE_CROSS_APP,            // 跨应用文件
    APP_FLAGS_FILE_ACCESS_COMMON_DIR,    // 访问公共目录
    APP_FLAGS_DLP_MANAGER_FULL_CONTROL,  // DLP完全控制
    APP_FLAGS_DLP_MANAGER_READ_ONLY,     // DLP只读
    APP_FLAGS_CLOUD_FILE_SYNC_ENABLED,  // 云文件同步
    MAX_FLAGS_INDEX = 63
} AppFlagsIndex;
```

### 扩展字段宏

```c
#define MSG_EXT_NAME_RENDER_CMD "render-cmd"        // 渲染命令
#define MSG_EXT_NAME_HSP_LIST "HspList"             // HSP列表
#define MSG_EXT_NAME_OVERLAY "Overlay"              // Overlay
#define MSG_EXT_NAME_DATA_GROUP "DataGroup"         // 数据组
#define MSG_EXT_NAME_APP_ENV "AppEnv"              // 应用环境
#define MSG_EXT_NAME_APP_EXTENSION "AppExtension"   // 应用扩展
#define MSG_EXT_NAME_BEGET_PID "AppPid"           // PID
#define MSG_EXT_NAME_BEGET_PTY_NAME "ptyName"      //PTY名称
#define MSG_EXT_NAME_ACCOUNT_ID "AccountId"       // 账户ID
#define MSG_EXT_NAME_PROVISION_TYPE "ProvisionType" // 供应类型
#define MSG_EXT_NAME_PROCESS_TYPE "ProcessType"    // 进程类型
#define MSG_EXT_NAME_MAX_CHILD_PROCCESS_MAX "MaxChildProcess" // 最大子进程
#define MSG_EXT_NAME_APP_FD "AppFd"               // 应用FD
#define MSG_EXT_NAME_JIT_PERMISSIONS "Permissions" // JIT权限
#define MSG_EXT_NAME_USERID "uid"                 // 用户ID
#define MSG_EXT_NAME_EXTENSION_TYPE "ExtensionType" // 扩展类型
#define MSG_EXT_NAME_API_TARGET_VERSION "APITargetVersion" // API目标版本
#define MSG_EXT_NAME_PARENT_UID "ParentUid"       // 父进程UID
#define MSG_EXT_NAME_APP_SIGN_TYPE "AppSignType"  // 应用签名类型
```

### API 函数

#### AppSpawnClientInit - 初始化客户端
```c
int AppSpawnClientInit(const char *serviceName, AppSpawnClientHandle *handle);
```
- **参数**: 
  - `serviceName`: 服务名 (如 "appspawn", "nwebspawn")
  - `handle`: 输出客户端句柄
- **返回值**: 0成功，失败返回错误码

#### AppSpawnClientDestroy - 销毁客户端
```c
int AppSpawnClientDestroy(AppSpawnClientHandle handle);
```

#### AppSpawnClientSendMsg - 发送消息
```c
int AppSpawnClientSendMsg(AppSpawnClientHandle handle, 
                         AppSpawnReqMsgHandle reqHandle, 
                         AppSpawnResult *result);
```
- **功能**: 发送孵化请求并等待响应

#### AppSpawnReqMsgCreate - 创建请求
```c
int AppSpawnReqMsgCreate(AppSpawnMsgType msgType, 
                        const char *processName, 
                        AppSpawnReqMsgHandle *reqHandle);
```

#### AppSpawnReqMsgFree - 释放请求
```c
void AppSpawnReqMsgFree(AppSpawnReqMsgHandle reqHandle);
```

#### AppSpawnReqMsgSetBundleInfo - 设置Bundle信息
```c
int AppSpawnReqMsgSetBundleInfo(AppSpawnReqMsgHandle reqHandle, 
                               uint32_t bundleIndex, 
                               const char *bundleName);
```

#### AppSpawnReqMsgSetAppFlag - 设置应用标志
```c
int AppSpawnReqMsgSetAppFlag(AppSpawnReqMsgHandle reqHandle, 
                            AppFlagsIndex flagIndex);
```

#### AppSpawnReqMsgSetAppDacInfo - 设置DAC权限
```c
int AppSpawnReqMsgSetAppDacInfo(AppSpawnReqMsgHandle reqHandle, 
                               const AppDacInfo *dacInfo);
```

#### AppSpawnReqMsgSetAppDomainInfo - 设置域信息
```c
int AppSpawnReqMsgSetAppDomainInfo(AppSpawnReqMsgHandle reqHandle, 
                                   uint32_t hapFlags, 
                                   const char *apl);
```

#### AppSpawnReqMsgSetAppInternetPermissionInfo - 设置互联网权限
```c
int AppSpawnReqMsgSetAppInternetPermissionInfo(AppSpawnReqMsgHandle reqHandle,
                                               uint8_t allow, 
                                               uint8_t setAllow);
```

#### AppSpawnReqMsgSetAppAccessToken - 设置访问令牌
```c
int AppSpawnReqMsgSetAppAccessToken(AppSpawnReqMsgHandle reqHandle, 
                                    uint64_t accessTokenIdEx);
```

#### AppSpawnReqMsgSetAppOwnerId - 设置所有者ID
```c
int AppSpawnReqMsgSetAppOwnerId(AppSpawnReqMsgHandle reqHandle, 
                                const char *ownerId);
```

#### AppSpawnReqMsgAddPermission - 添加权限
```c
int AppSpawnReqMsgAddPermission(AppSpawnReqMsgHandle reqHandle, 
                                const char *permission);
```

#### AppSpawnReqMsgAddExtInfo - 添加扩展信息
```c
int AppSpawnReqMsgAddExtInfo(AppSpawnReqMsgHandle reqHandle, 
                            const char *name, 
                            const uint8_t *value, 
                            uint32_t valueLen);
```

#### AppSpawnReqMsgAddStringInfo - 添加字符串信息
```c
int AppSpawnReqMsgAddStringInfo(AppSpawnReqMsgHandle reqHandle, 
                                const char *name, 
                                const char *value);
```

#### AppSpawnReqMsgAddFd - 添加文件描述符
```c
int AppSpawnReqMsgAddFd(AppSpawnReqMsgHandle reqHandle, 
                        const char* fdName, 
                        int fd);
```

#### GetPermissionIndex - 获取权限索引
```c
int32_t GetPermissionIndex(AppSpawnClientHandle handle, 
                          const char *permission);
```

#### GetMaxPermissionIndex - 获取最大权限索引
```c
int32_t GetMaxPermissionIndex(AppSpawnClientHandle handle);
```

#### GetPermissionByIndex - 根据索引获取权限名
```c
const char *GetPermissionByIndex(AppSpawnClientHandle handle, 
                                 int32_t index);
```

#### SpawnListenFdSet - 设置监听FD (appspawn子进程)
```c
int SpawnListenFdSet(int fd);
```

#### SpawnListenCloseSet - 关闭监听 (appspawn子进程)
```c
int SpawnListenCloseSet(void);
```

## 客户端API (appspawn_client.h)

> 文件: `interfaces/innerkits/client/appspawn_client.h`

### 超时配置

```c
#define TIMEOUT_DEF 2          // 默认超时2秒
#define ASAN_TIMEOUT 10        // ASAN超时10秒
#define CJAPPSPAWN_CLIENT_TIMEOUT 10  // CJ客户端超时

#define RETRY_TIME (200 * 1000)      // 重试间隔200ms
#define MAX_RETRY_SEND_COUNT 2       // 最大重试次数
```

### 客户端类型

```c
typedef enum {
    CLIENT_FOR_APPSPAWN,
    CLIENT_FOR_NWEBSPAWN,
    CLIENT_FOR_CJAPPSPAWN,
    CLIENT_FOR_NATIVESPAWN,
    CLIENT_FOR_HYBRIDSPAWN,
    CLIENT_MAX
} AppSpawnClientType;
```

## HNP API (hnp_api.h)

> 文件: `interfaces/innerkits/hnp/include/hnp_api.h`

### 错误码基址

```c
#define HNP_API_ERRNO_BASE 0x2000
```

### 错误码定义

| 错误码 | 值 | 说明 |
|--------|-----|------|
| HNP_API_ERRNO_PARAM_INVALID | 0x2001 | 参数非法 |
| HNP_API_ERRNO_FORK_FAILED | 0x2002 | fork失败 |
| HNP_API_WAIT_PID_FAILED | 0x2003 | 等待PID失败 |
| HNP_API_NOT_IN_DEVELOPER_MODE | 0x2004 | 非开发者模式 |
| HNP_API_ERRNO_PIPE_CREATED_FAILED | 0x2005 | 创建管道失败 |
| HNP_API_ERRNO_PIPE_READ_FAILED | 0x2006 | 读取管道失败 |
| HNP_API_ERRNO_RETURN_VALUE_GET_FAILED | 0x2007 | 获取返回值失败 |
| HNP_API_ERRNO_MEMMOVE_FAILED | 0x2008 | 内存移动失败 |
| HNP_API_ERRNO_HNP_INSTALL_DISABLED | 0x2009 | 产品不支持HNP |
| HNP_API_ERRNO_TOO_MANY_PARAM | 0x2010 | 参数过多 |

### 数据结构

#### HapInfo - HAP信息
```c
typedef struct {
    char packageName[PACK_NAME_LENTH];      // 包名
    char hapPath[HAP_PATH_LENTH];            // HAP路径
    char abi[ABI_LENTH];                    // ABI
    char appIdentifier[APP_IDENTIFIER_LEN];  // 应用标识
    int count;                               // 数量
    char **independentSignHnpPaths;          // 独立签名HNP路径
} HapInfo;
```

#### HnpResult - HNP结果
```c
typedef struct {
    int result;
} HnpResult;
```

### API 函数

#### NativeInstallHnp - 安装HNP
```c
int NativeInstallHnp(const char *userId, 
                    const char *hnpRootPath, 
                    const HapInfo *hapInfo, 
                    int installOptions);
```

#### NativeUnInstallHnp - 卸载HNP
```c
int NativeUnInstallHnp(const char *userId, 
                      const char *packageName);
```

## 消息格式

### TLV结构

```c
#pragma pack(4)
typedef struct {
    uint16_t tlvLen;   // TLV长度
    uint16_t tlvType;  // TLV类型
    // 数据...
} AppSpawnTlv;
```

### 消息头

```c
typedef struct {
    uint32_t magic;        // 魔数 0xEF201234
    uint32_t msgType;      // 消息类型
    uint32_t msgLen;       // 消息总长度
    uint32_t msgId;        // 消息ID
    uint32_t tlvCount;     // TLV数量
    char processName[APP_LEN_PROC_NAME];  // 进程名
} AppSpawnMsg;
```

### TLV类型枚举

| 类型 | 值 | 说明 |
|------|-----|------|
| TLV_BUNDLE_INFO | 0 | Bundle信息 |
| TLV_MSG_FLAGS | 1 | 消息标志 |
| TLV_DAC_INFO | 2 | DAC权限 |
| TLV_DOMAIN_INFO | 3 | 域信息 |
| TLV_OWNER_INFO | 4 | 所有者信息 |
| TLV_ACCESS_TOKEN_INFO | 5 | 访问令牌 |
| TLV_PERMISSION | 6 | 权限 |
| TLV_INTERNET_INFO | 7 | 互联网权限 |
| TLV_RENDER_TERMINATION_INFO | 8 | 渲染终止信息 |
| TLV_MAX | 9 | 最大值 |

## 错误码

### 主错误码

| 错误码 | 说明 |
|--------|------|
| APPSPAWN_ARG_INVALID | 参数非法 |
| APPSPAWN_MSG_INVALID | 消息非法 |
| APPSPAWN_PERMISSION_NOT_SUPPORT | 权限不支持 |
| APPSPAWN_SYSTEM_ERROR | 系统错误 |
| APPSPAWN_CHILD_CRASH | 子进程崩溃 |
| APPSPAWN_SPAWN_TIMEOUT | 孵化超时 |
| APPSPAWN_DEBUG_MODE_NOT_SUPPORT | 调试模式不支持 |
| APPSPAWN_SANDBOX_INVALID | 沙箱无效 |

### 使用示例

```c
// 初始化客户端
AppSpawnClientHandle handle;
int ret = AppSpawnClientInit(APPSPAWN_SERVER_NAME, &handle);
if (ret != 0) {
    // 处理错误
}

// 创建请求
AppSpawnReqMsgHandle reqHandle;
ret = AppSpawnReqMsgCreate(MSG_APP_SPAWN, "com.example.app", &reqHandle);
if (ret != 0) {
    AppSpawnClientDestroy(handle);
    return ret;
}

// 设置DAC信息
AppDacInfo dacInfo = {
    .uid = 10000,
    .gid = 10000,
    .gidCount = 2,
    .gidTable = {1006, 1008}  // GID_FILE_ACCESS, GID_USER_DATA_RW
};
AppSpawnReqMsgSetAppDacInfo(reqHandle, &dacInfo);

// 发送消息
AppSpawnResult result;
ret = AppSpawnClientSendMsg(handle, reqHandle, &result);
if (ret == 0) {
    printf("Spawned PID: %d\n", result.pid);
}

// 清理
AppSpawnReqMsgFree(reqHandle);
AppSpawnClientDestroy(handle);
```

## 相关文件

| 文件 | 说明 |
|------|------|
| `interfaces/innerkits/include/appspawn.h` | 主API定义 |
| `interfaces/innerkits/client/appspawn_client.h` | 客户端头文件 |
| `interfaces/innerkits/client/appspawn_client.c` | 客户端实现 |
| `interfaces/innerkits/client/appspawn_msg.c` | 消息构建 |
| `interfaces/innerkits/hnp/include/hnp_api.h` | HNP API |

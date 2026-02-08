# Inner API 内部接口

本章节描述 init 模块内部组件之间的接口，用于理解模块边界和依赖方向。

## 接口分类

| 分类 | 路径 | 说明 |
|------|------|------|
| **innerkits** | `interfaces/innerkits/` | 内部组件接口，供系统服务使用 |
| **services** | `services/*/src/` | 服务内部实现 |

---

## 核心模块职责

### 1. init 主服务

**路径**: `services/init/`

| 文件 | 职责 |
|------|------|
| `init_service_manager.c` | 服务生命周期管理 |
| `init_common_cmds.c` | 通用命令实现 |
| `init_common_service.c` | 服务公共功能 |
| `init_capability.c` | Capabilities 配置 |
| `init_cgroup.c` | Cgroup 配置 |
| `init_config.c` | init.cfg 解析 |

**关键结构体**:

```c
// services/init/include/init_service.h
struct Perms {
    int uID;
    int gIDArray[MAX_GIDS];
    int caps[MAX_CAPS_CNT_FOR_ONE_SERVICE];
};
```

### 2. 参数服务 (param)

**路径**: `services/param/`

| 子模块 | 职责 |
|--------|------|
| `base/` | 参数基础实现 |
| `manager/` | 参数管理 |
| `watcher/` | 参数监视器 |
| `trigger/` | 触发器 |

**关键接口**:

| 接口 | 文件 | 说明 |
|------|------|------|
| `GetParameter()` | `param_base/` | 获取参数 |
| `SetParameter()` | `param_base/` | 设置参数 |
| `WatchParameter()` | `watcher/` | 监听参数变化 |

### 3. 事件循环 (loopevent)

**路径**: `services/loopevent/`

| 子模块 | 职责 |
|--------|------|
| `loop/` | epoll 事件循环 |
| `socket/` | Socket 事件处理 |
| `signal/` | 信号处理 |
| `timer/` | 定时器 |
| `task/` | 任务队列 |

### 4. 功能模块 (modules)

**路径**: `services/modules/`

| 模块 | 职责 |
|------|------|
| `bootevent/` | 启动事件管理 |
| `crashhandler/` | 崩溃处理 |
| `seccomp/` | Seccomp 策略 |
| `selinux/` | SELinux 适配 |
| `reboot/` | 重启功能 |
| `init_hook/` | 钩子管理 |
| `init_eng/` | 工程模式 |
| `sysevent/` | 系统事件 |

---

## innerkits 接口清单

### 1. 服务控制

**路径**: `interfaces/innerkits/include/service_control.h`

| 函数 | 说明 |
|------|------|
| `StartServiceById()` | 按 ID 启动服务 |
| `StopServiceById()` | 按 ID 停止服务 |

### 2. 循环事件

**路径**: `interfaces/innerkits/include/loop_event.h`

| 函数 | 说明 |
|------|------|
| `LeLoopCreate()` | 创建事件循环 |
| `LeLoopRun()` | 运行事件循环 |
| `LeLoopStop()` | 停止事件循环 |

### 3. 钩子管理

**路径**: `interfaces/innerkits/include/hookmgr.h`

| 函数 | 说明 |
|------|------|
| `RegisterHook()` | 注册钩子 |
| `UnregisterHook()` | 注销钩子 |
| `TriggerHook()` | 触发钩子 |

### 4. 文件系统管理

**路径**: `interfaces/innerkits/include/fs_manager/fs_manager.h`

| 函数 | 说明 |
|------|------|
| `MountAll()` | 挂载所有文件系统 |
| `UmountAll()` | 卸载所有文件系统 |

### 5. 系统参数

**路径**: `interfaces/innerkits/include/syspara/`

| 函数 | 说明 |
|------|------|
| `GetParameter()` | 获取参数 |
| `SetParameter()` | 设置参数 |
| `GetSerial()` | 获取序列号 |

---

## 依赖方向

```
                          ┌─────────────────┐
                          │   init (主进程)   │
                          └────────┬────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          ↓                        ↓                        ↓
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   loopevent     │    │    modules      │    │     param       │
│   (事件循环)     │    │   (功能模块)    │    │    (参数服务)    │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                                ↓
                    ┌─────────────────────┐
                    │   services/utils    │
                    │     (工具函数库)     │
                    └─────────────────────┘
```

**依赖规则**:

1. `init` 可调用所有模块
2. `modules` 可调用 `loopevent`、`param`
3. `loopevent` 和 `param` 无外部依赖
4. 禁止循环依赖

---

## 稳定性标注

| 层级 | 路径模式 | 说明 |
|------|----------|------|
| **稳定** | `interfaces/innerkits/include/` | 供系统服务使用，有版本维护 |
| **实验性** | `services/*/include/` | 内部使用，可能变更 |
| **不稳定** | `services/*/src/` | 实现细节，随时可能变更 |

---

## 错误传播机制

```
┌──────────────────────────────────────────────────────────┐
│                      错误处理流程                         │
├──────────────────────────────────────────────────────────┤
│  1. 底层捕获错误                                         │
│  2. 转换为错误码 (参数服务: -1/-2/-3)                    │
│  3. 向上返回错误码                                       │
│  4. 顶层记录日志 (hilog)                                 │
│  5. (可选) 触发系统事件                                  │
└──────────────────────────────────────────────────────────┘
```

### 常见错误码

| 模块 | 错误码 | 含义 |
|------|--------|------|
| param | -1 | 参数错误 |
| param | -2 | 参数不存在 |
| param | -3 | 系统错误 |
| device_info | DEV_INFO_ENULLPTR | 空指针 |
| device_info | DEV_INFO_EGETODID | 获取 ODID 失败 |
| device_info | DEV_INFO_ESTRCOPY | 字符串拷贝失败 |

---

## 关键数据接口

### 1. init 服务配置

```c
// services/init/include/init_service.h
typedef struct {
    char name[SERVICE_NAME_LEN];        // 服务名
    char path[PATH_LEN];               // 路径
    int uid;                           // UID
    int gid;                           // GID
    int gids[MAX_GIDS];                // 附加组
    int caps[MAX_CAPS_CNT_FOR_ONE_SERVICE];  // Capabilities
    char secon[SECON_LEN];             // SELinux 上下文
    int importance;                    // 重要性
    int once;                          // 是否一次运行
} ServiceEntry;
```

### 2. 服务 Socket 配置

```c
// services/init/include/init_service_socket.h
typedef struct {
    char name[SOCKET_NAME_LEN];
    char path[PATH_LEN];
    int uid;
    int gid;
    int perm;                          // 权限
    int backlog;                        // backlog
} ServiceSocket;
```

### 3. 服务文件配置

```c
// services/init/include/init_service_file.h
typedef struct {
    char name[FILE_NAME_LEN];
    char path[PATH_LEN];
    int uid;
    int gid;
    int perm;                          // 权限
} ServiceFile;
```

### 4. 设备节点配置

```c
// ueventd/include/ueventd_read_cfg.h
typedef struct {
    char deviceName[DEVICE_NAME_LEN];   // 设备名
    char path[PATH_LEN];               // 设备路径
    int uid;                           // UID
    int gid;                           // GID
    int perm;                          // 权限
    char secon[SECON_LEN];             // SELinux 上下文
    long long major;                   // 主设备号
    long long minor;                   // 次设备号
    char *action;                      // 动作
    char *subsystem;                   // 子系统
} DeviceNodeCfg;
```

---

## innerkits 详细接口

### 1. 系统能力接口

**路径**: `interfaces/innerkits/include/systemcapability.h`

| 函数 | 说明 | 返回值 |
|------|------|--------|
| `HasSystemCapability(const char *cap)` | 检查系统能力 | bool |

### 2. Seccomp 策略接口

**路径**: `interfaces/innerkits/seccomp/include/seccomp_policy.h`

| 函数 | 说明 |
|------|------|
| `SeccompFilterType` | 过滤器类型枚举 |
| `GetSeccompFilter()` | 获取过滤器 |
| `SetSeccompPolicy()` | 设置策略 |

### 3. 重启接口

**路径**: `interfaces/innerkits/include/reboot/`

| 函数 | 说明 |
|------|------|
| `DoReboot()` | 执行重启 |
| `DoRebootClear()` | 清除后重启 |

### 4. 服务观察接口

**路径**: `interfaces/innerkits/include/service_watcher.h`

| 函数 | 说明 |
|------|------|
| `WatchServiceStatus()` | 观察服务状态 |
| `UnwatchServiceStatus()` | 取消观察 |

### 5. FD 持有者接口

**路径**: `interfaces/innerkits/fd_holder/`

| 函数 | 说明 |
|------|------|
| `AddFdToHolder()` | 添加 FD |
| `RemoveFdFromHolder()` | 移除 FD |

### 6. Token 管理接口

**路径**: `interfaces/innerkits/token/`

| 函数 | 说明 |
|------|------|
| `GetAccessTokenId()` | 获取访问令牌 ID |
| `SetProcessToken()` | 设置进程令牌 |

### 7. 模块引擎接口

**路径**: `interfaces/innerkits/init_module_engine/include/init_module_engine.h`

| 函数 | 说明 |
|------|------|
| `InitModuleEngine()` | 初始化模块引擎 |
| `LoadModule()` | 加载模块 |
| `UnloadModule()` | 卸载模块 |

---

## 相关跳转

- [架构](02_Architecture.md)
- [N-API](03_NAPI.md)
- [构建](05_Build.md)

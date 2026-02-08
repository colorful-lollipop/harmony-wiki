# 项目概览

## 项目定位

`@ohos/init` 是 OpenHarmony 系统的**初始化模块**，负责从内核加载第一个用户空间进程到第一个应用启动之间的所有系统初始化工作。

**核心职责**:

| 职责 | 说明 |
|------|------|
| 进程启动 | 按配置顺序启动系统服务 |
| 权限配置 | 配置服务进程的 UID/GID/Capabilities |
| 进程监控 | 监控服务状态，异常时重启 |
| 系统重置 | 关键进程退出时触发系统复位 |

## 目标设备

| 系统类型 | 支持状态 | 内存要求 |
|----------|----------|----------|
| Mini 系统 | ✅ 支持 | ≥ 1 MB |
| Small 系统 | ✅ 支持 | ≥ 1 MB |
| Standard 系统 | ✅ 支持 | 无限制 |

## 启动阶段

Init 模块将系统启动分为三个阶段：

```
┌─────────────────────────────────────────────────────┐
│                    Kernel Space                       │
├─────────────────────────────────────────────────────┤
│  1. pre-init 阶段                                   │
│     - 文件系统挂载                                   │
│     - 创建系统目录                                   │
│     - 设置权限                                      │
├─────────────────────────────────────────────────────┤
│  2. init 阶段                                       │
│     - 启动系统服务                                  │
│     - 配置服务权限                                  │
│     - 建立进程间通信                                │
├─────────────────────────────────────────────────────┤
│  3. post-init 阶段                                  │
│     - 驱动初始化后处理                              │
│     - 启动应用孵化器                                │
├─────────────────────────────────────────────────────┤
│                    User Space                        │
│     - 第一个应用启动                                │
└─────────────────────────────────────────────────────┘
```

### 配置文件格式

系统启动行为由 `init.cfg` 配置，JSON 格式（≤ 100KB）：

```json
{
    "jobs": [
        {
            "name": "pre-init",
            "cmds": [
                "mkdir /testdir",
                "chmod 0700 /testdir",
                "mount vfat /dev/mmcblk0p0 /storage noexec nosuid"
            ]
        },
        {
            "name": "init",
            "cmds": [
                "start foundation",
                "start appspawn"
            ]
        }
    ],
    "services": [
        {
            "name": "foundation",
            "path": "/system/bin/foundation",
            "uid": 0,
            "gid": 0,
            "once": 0,
            "importance": 1,
            "caps": [0, 1, 2, 5]
        }
    ]
}
```

## 目录结构

```
base/startup/init/
├── interfaces/              # 接口层
│   ├── innerkits/          # 内部组件接口
│   └── kits/               # 外部 SDK 接口 (N-API)
├── services/               # 核心服务实现
│   ├── init/              # init 主进程
│   ├── modules/           # 功能模块
│   ├── param/             # 参数服务
│   ├── loopevent/         # 事件循环
│   ├── sandbox/           # 沙箱服务
│   └── begetctl/          # 控制工具
├── device_info/           # 设备信息服务
├── ueventd/               # uevent 守护进程
├── watchdog/              # 看门狗服务
├── remount/               # 重新挂载服务
└── BUILD.gn              # 构建入口
```

## 关键能力

### 系统能力 (Syscap)

| 能力 | 说明 |
|------|------|
| `SystemCapability.Startup.SystemInfo` | 系统信息查询 |
| `SystemCapability.Startup.SystemInfo.Lite` | 轻量系统信息 |

### 主要功能特性

- **进程管理**: 服务启动、重启、监控
- **权限控制**: DAC + Capabilities + SELinux + Seccomp
- **参数服务**: 系统参数读写、监听
- **设备管理**: ueventd + 设备节点权限
- **启动同步**: initsync 同步点

## 相关跳转

- [架构说明](02_Architecture.md)
- [N-API 接口](03_NAPI.md)
- [构建配置](05_Build.md)

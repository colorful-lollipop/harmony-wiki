# 编译产物和运行时

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档说明 CastEngine 框架的编译产物、安装路径、运行时加载关系和部署注意事项。

## 产物清单

### 主要产物

| GN Target | 产物类型 | 文件名 | 安装目录 | 说明 |
|-----------|----------|---------|----------|------|
| cast | ohos_shared_library | libcast.z.so | /system/lib64/module/ | N-API 模块 |
| cast_engine_client | ohos_shared_library | libcast_engine_client.z.so | /system/lib64/ | 内部 API 库 |
| cast_engine_service | ohos_shared_library | libcast_engine_service.z.so | /system/lib64/ | System Ability 服务 |
| cast_engine_sa_profile | ohos_sa_profile | 5526.json | /system/profile/ | SA 配置 |
| cast_engine_service.cfg | ohos_prebuilt_etc | cast_engine_service.cfg | /etc/init/ | 服务启动配置 |

### 静态库（内部使用）

| GN Target | 文件名 | 用途 |
|-----------|---------|------|
| cast_engine_common_sources | libcast_engine_common_sources.a | 公共代码库 |
| cast_client_inner | - | 客户端静态库 |
| cast_discovery | - | 设备发现库 |
| cast_session | - | 会话管理库 |
| cast_session_channel | - | 通道管理库 |
| cast_session_rtsp | - | RTSP 协议库 |
| cast_session_stream | - | 流播放器库 |
| cast_session_mirror | - | 镜像播放器库 |
| cast_session_utils | - | 会话工具库 |

---

## 安装路径详解

### /system/lib64/

**内容**: 动态共享库

| 文件 | 说明 | 依赖 |
|-----|------|------|
| libcast.z.so | N-API 模块 | libcast_engine_client.z.so |
| libcast_engine_client.z.so | 内部 API 库 | libcast_engine_common_sources.a |
| libcast_engine_service.z.so | System Ability 服务 | 多个静态库 |

### /system/lib64/module/

**说明**: N-API 模块专用目录

| 文件 | 说明 | 模块名 |
|-----|------|--------|
| libcast.z.so | CastEngine N-API 模块 | cast |

### /system/profile/

**说明**: System Ability 配置目录

| 文件 | 说明 | 内容 |
|-----|------|------|
| 5526.json | SA 5526 配置 | SA ID 5526 的定义 |
| (其他 SA) | 其他 SA 配置 | - |

### /etc/init/

**说明**: init 系统启动配置目录

| 文件 | 说明 | 内容 |
|-----|------|------|
| cast_engine_service.cfg | CastEngine 服务配置 | 服务启动参数、权限、依赖 |

---

## 运行时加载关系

### 启动流程

```
系统启动
    │
    ├── 1. /etc/init/cast_engine_service.cfg 被加载
    │
    ├── 2. init 进程创建 cast_engine_service 进程
    │
    ├── 3. cast_engine_service 进程加载 libcast_engine_service.z.so
    │
    ├── 4. 服务注册为 SA 5526（延迟启动）
    │
    └── 5. 等待第一次调用（按需启动）
         │
         ▼
    ┌──────────────────────────┐
    │ System Ability 就绪  │
    │ (SA 5526)             │
    └──────────────────────────┘
```

### 应用加载流程

```
应用启动
    │
    ├── 1. Node.js/ArkTS 运行
    │
    ├── 2. import cast from '@ohos.cast'
    │
    ├── 3. 系统加载 libcast.z.so
    │
    ├── 4. libcast.z.so 调用 napi_module_register
    │
    ├── 5. 注册 "cast" 模块
    │
    └── 6. 调用 Init() 函数初始化导出的类
         │
         ▼
    ┌──────────────────────────┐
    │ N-API 模块就绪       │
    └──────────────────────────┘
```

### 依赖加载链

```
应用加载 libcast.z.so
    │
    ├─> 依赖 libcast_engine_client.z.so
    │
    ├─> libcast_engine_client.z.so 依赖 libcast_engine_common_sources.a
    │
    └─> 所有动态加载外部系统库
            ├─> libhilog.so
            ├─> libipc_core.so
            ├─> libsamgr.so
            └─> libace_napi.z.so
```

### IPC 连接建立

```
应用 (libcast.z.so)
    │
    ├── 1. 调用 CastSessionManager
    │
    ├── 2. 通过 libcast_engine_client.z.so 发起 IPC 调用
    │
    ├── 3. libsamgr.so 路由到 SA 5526
    │
    ├── 4. 启动/连接 cast_engine_service 进程
    │
    └── 5. libcast_engine_service.z.so 处理请求
         │
         ▼
    ┌──────────────────────────┐
    │ IPC 连接建立        │
    └──────────────────────────┘
```

---

## 运行时资源使用

### 内存使用

**估算**（基于 ROM/RAM 配置）:

| 组件 | ROM 估算 | RAM 估算 |
|-------|----------|----------|
| CastEngine 总计 | 5MB | 50MB |

**证据**: `bundle.json:22-23`

**内存使用场景**:
- **空闲**: ~5MB 常驻内存
- **单会话**: +2-5MB（会话状态、设备信息）
- **镜像播放**: +10-20MB（编码缓冲区）
- **流播放**: +5-10MB（播放缓冲区）

### 线程使用

| 模块 | 线程类型 | 数量（估算） |
|-----|----------|-------------|
| N-API 主线程 | JavaScript 线程 | 1 |
| N-API 工作线程 | 线程池 | 4-8 |
| IPC 线程 | Binder 线程 | 1-2 |
| 设备发现线程 | 独立线程 | 1-2 |
| 播放器线程 | 播放/解码线程 | 2-4 |

### 系统能力依赖

| 能力 | 用途 | 依赖方式 |
|-----|------|---------|
| IPC | 跨进程通信 | 动态加载 libipc_core.so |
| Samgr | SA 管理 | 动态加载 libsamgr.so |
| HiLog | 日志 | 动态加载 libhilog.so |
| Surface | 图形显示 | 动态加载 libsurface.so |
| Audio 框架 | 音频采集/播放 | 动态加载 libaudio_client.so |
| Player 框架 | 媒体播放 | 动态加载 libmedia_client.so |

---

## 部署注意事项

### 1. System Ability 配置

**延迟启动**: SA 配置为 `run-on-create: false`

**意义**:
- 首次调用时才启动服务
- 节省系统启动时间
- 按需加载（延迟加载）

**配置**: `sa_profile/5526.json:7`

### 2. 权限声明

应用在使用 CastEngine 时必须在 `module.json5` 中声明：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_CAST_ENGINE_MIRROR",
        "reason": "$string:mirror_reason"
      },
      {
        "name": "ohos.permission.ACCESS_CAST_ENGINE_STREAM",
        "reason": "$string:stream_reason"
      }
    ]
  }
}
```

### 3. 适配器仓库部署

CastEngine 依赖的协议适配器需要单独部署：

| 适配器 | 仓库名称 | 部署方式 |
|-------|----------|---------|
| Cast+Stream | castengine_cast_plus_stream | 作为系统包或独立服务 |
| WiFi Display | castengine_wifi_display | 作为系统包或独立服务 |
| DLNA | castengine_dlna | 作为系统包或独立服务 |

### 4. 配置文件位置

| 文件 | 路径 | 作用 |
|-----|------|------|
| cast_engine_service.cfg | /etc/init/cast_engine_service.cfg | 服务启动配置 |
| 5526.json | /system/profile/5526.json | SA 注册信息 |
| hisysevent.yaml | /system/share/hi_sys_event/hisysevent.yaml | 事件上报配置 |

---

## 调试和分析

### 日志标签

**日志接口**: `common/include/private/cast_engine_log.h`

**日志标签示例**:
- `Cast-Napi-SessionManager` - N-API 会话管理器
- `Cast-Napi-Session` - N-API 会话
- `Cast-Napi-StreamPlayer` - N-API 流播放器
- `Cast-Napi-MirrorPlayer` - N-API 镜像播放器
- `Cast-Permission` - 权限检查
- `Cast-Discovery` - 设备发现

### HiSysEvent 事件

**配置**: `hisysevent.yaml`

**事件类别**: 投屏相关事件

**用途**:
- 监控服务状态
- 记录异常情况
- 性能数据收集

---

## 运行时故障排查

### 常见问题

#### 1. SA 未启动

**症状**: 应用调用 CastEngine 时超时或失败

**可能原因**:
- SA 进程未启动
- SA 注册失败
- 依赖的系统服务未就绪

**排查步骤**:
1. 检查 `/etc/init/cast_engine_service.cfg` 是否正确
2. 使用 `hdc shell ps -ef | grep cast_engine_service` 检查进程
3. 使用 `hdc shell hidumper -s CastEngine` 查看 SA 状态
4. 检查 `system/lib64/` 目录下是否存在库文件

#### 2. 权限被拒绝

**症状**: 返回 `ERR_NO_PERMISSION` (-1003)

**可能原因**:
- 应用未声明所需权限
- 权限未授予
- UID 不匹配

**排查步骤**:
1. 检查 `module.json5` 中的权限声明
2. 使用 `hdc shell pm list permissions` 查看已授予权限
3. 检查权限是否正确授予

#### 3. 设备发现失败

**症状**: `startDiscovery()` 成功但 `deviceFound` 事件不触发

**可能原因**:
- 网络未开启
- WiFi/蓝牙未启用
- 适配器服务未运行

**排查步骤**:
1. 检查网络连接
2. 使用 `hdc shell ifconfig` 检查网络接口
3. 检查适配器服务状态

---

## 相关文档

- [GN Targets](05_GN_Targets.md) - 构建目标和产物定义
- [目录结构](01_Directory_Structure.md) - 代码组织和模块划分
- [安全评审](07_Security_Review.md) - 编译和运行时安全特性

---

**返回**: [SUMMARY.md](SUMMARY.md)

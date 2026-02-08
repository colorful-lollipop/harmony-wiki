# 07_Build_Artifacts - 编译产物

> 本文档详细说明 Cast+ Stream 模块的编译产物、安装路径和运行时加载关系。

---

## 1. 产物清单

### 1.1 静态库产物

| 产物名 | 文件名 | 大小(预估) | 说明 |
|--------|--------|-----------|------|
| 主会话库 | `libcast_session.a` | ~2MB | 聚合所有子模块的静态库 |
| 通道库 | `libcast_session_channel.a` | ~500KB | SoftBus/TCP 通道实现 |
| 工具库 | `libcast_session_utils.a` | ~300KB | 加密、状态机、工具函数 |
| RTSP 库 | `libcast_session_rtsp.a` | ~800KB | RTSP 协议实现 |
| 流媒体库 | `libcast_session_stream.a` | ~1MB | 流媒体播放控制 |
| 镜像库 | `libcast_session_mirror.a` | ~600KB | 镜像播放实现 |

### 1.2 产物依赖关系

```
libcast_session.a (聚合库)
├── libcast_session_channel.a
├── libcast_session_utils.a
├── libcast_session_rtsp.a
│   ├── libcast_session_channel.a
│   └── libcast_session_utils.a
├── libcast_session_stream.a
│   ├── libcast_session_channel.a
│   └── libcast_session_utils.a
└── libcast_session_mirror.a
    ├── libcast_session_channel.a
    ├── libcast_session_rtsp.a
    ├── libcast_session_stream.a
    └── libcast_session_utils.a
```

---

## 2. 输出路径

### 2.1 构建输出目录

```
out/{product_name}/
├── obj/
│   └── foundation/CastEngine/castengine_cast_plus_stream/
│       ├── libcast_session.a
│       └── src/
│           ├── channel/libcast_session_channel.a
│           ├── mirror/libcast_session_mirror.a
│           ├── rtsp/libcast_session_rtsp.a
│           ├── stream/libcast_session_stream.a
│           └── utils/libcast_session_utils.a
│
└── {product_name}/
    └── system/lib/
        └── (运行时共享库，由父级框架提供)
```

### 2.2 安装路径

| 产物类型 | 安装路径 | 说明 |
|----------|----------|------|
| 静态库 | 不安装 | 作为依赖被链接到父级框架 |
| 头文件 | `//foundation/CastEngine/castengine_cast_framework/include` | 通过 public_configs 暴露 |

---

## 3. 运行时加载关系

### 3.1 模块加载图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           运行时加载关系                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   应用进程 (Application)                                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐    │
│   │                     libcast_engine_client.z.so                     │    │
│   │                   (父级框架提供的客户端库)                          │    │
│   │                                                                     │    │
│   │  包含:                                                              │    │
│   │  - ICastSessionImpl Proxy                                           │    │
│   │  - IMirrorPlayerImpl Proxy                                          │    │
│   │  - IStreamPlayerIpc Proxy                                           │    │
│   │                                                                     │    │
│   │  静态链接: libcast_session.a (本模块静态库)                         │    │
│   └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│                                    ▼ IPC                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐    │
│   │                     CastEngine Service 进程                         │    │
│   │                                                                     │    │
│   │  ┌─────────────────────────────────────────────────────────────┐   │    │
│   │  │              libcast_engine_service.z.so                   │   │    │
│   │  │            (父级框架提供的服务端库)                         │   │    │
│   │  │                                                             │   │    │
│   │  │  包含:                                                      │   │    │
│   │  │  - CastSessionImplStub (本模块实现)                         │   │    │
│   │  │  - MirrorPlayerImplStub (本模块实现)                        │   │    │
│   │  │  - StreamPlayerImplStub (本模块实现)                        │   │    │
│   │  │                                                             │   │    │
│   │  │  静态链接: libcast_session.a (本模块静态库)                 │   │    │
│   │  └─────────────────────────────────────────────────────────────┘   │    │
│   │                                                                     │    │
│   │  依赖的系统库:                                                       │    │
│   │  - libipc_core.z.so (IPC 核心)                                      │    │
│   │  - libmedia_client.z.so (媒体框架)                                  │    │
│   │  - libaudio_client.z.so (音频框架)                                  │    │
│   │  - libdsoftbus.z.so (SoftBus)                                       │    │
│   │  - libhilog.z.so (日志)                                             │    │
│   │                                                                     │    │
│   └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 加载顺序

```
1. 系统启动
   └── 加载 CastEngine Service 进程
       └── 加载 libcast_engine_service.z.so
           └── 静态链接 libcast_session.a
               └── 初始化 ChannelManager、RtspController 等

2. 应用启动投屏
   └── 加载 libcast_engine_client.z.so
       └── 静态链接 libcast_session.a (接口定义部分)
       └── 通过 IPC 连接到 CastEngine Service
           └── 创建 CastSessionImpl 实例
               └── 创建 Channel、RTSP Controller 等
```

---

## 4. 产物使用方式

### 4.1 父级框架链接

```gn
# 父级框架 (castengine_cast_framework) 的 BUILD.gn

ohos_shared_library("cast_engine_service") {
  deps = [
    # 链接本模块的静态库
    "${cast_engine_service}/src/session:cast_session",
    # ... 其他依赖
  ]
  
  # 导出头文件配置
  public_configs = [
    "${cast_engine_service}/src/session:cast_session_config",
  ]
}

ohos_shared_library("cast_engine_client") {
  deps = [
    # 客户端只需要接口定义
    "${cast_engine_service}/src/session:cast_session",
    # ... 其他依赖
  ]
}
```

### 4.2 最终产物

根据 README.md，编译最终生成：

| 产物 | 类型 | 说明 |
|------|------|------|
| `libcast.z.so` | 共享库 | CastEngine 主库 |
| `libcast_engine_client.z.so` | 共享库 | 客户端库 |
| `libcast_engine_service.z.so` | 共享库 | 服务端库 |

**注意**: 本模块的静态库被链接到上述共享库中，不单独发布。

---

## 5. 符号导出

### 5.1 对外符号

通过 IPC 接口暴露，无直接 C++ 符号导出。

### 5.2 内部符号

所有符号为内部使用，不对外暴露。

---

## 6. 调试信息

### 6.1 日志标签

| 模块 | 日志标签 | 说明 |
|------|----------|------|
| 会话 | `Cast-SessionImpl` | 会话实现日志 |
| 监听器 | `Cast-Session-Listener` | 监听器日志 |
| 权限 | `Cast-Permission` | 权限检查日志 |
| 加密 | `Cast-EncryptDecrypt` | 加密解密日志 |

### 6.2 日志域

```
0xD004601 - CastEngine 主域
0xD002B00 - 会话域
0xD00ff00 - 通道域
0xD002b2b - RTSP 域
0xD003900 - 流媒体域
0xD0015c0 - 镜像域
```

### 6.3 调试命令

```bash
# 开启调试日志
hdc shell hilog -b D -D 0xD004601
hdc shell hilog -b D -D 0xD002B00
hdc shell hilog -b D -D 0xD00ff00

# 查看日志
hdc shell hilog -g
```

---

## 7. 相关文档

- [06_GN_Targets.md](./06_GN_Targets.md) - GN 构建目标
- [09_Troubleshooting.md](./09_Troubleshooting.md) - 问题定位

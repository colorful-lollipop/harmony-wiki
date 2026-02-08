# 编译产物与安装路径

## 产物清单

### 1. libsharingwfd_napi.z.so

| 属性 | 值 |
|------|-----|
| 类型 | N-API 共享库 |
| 源文件 | `interfaces/kits/js/wfd/BUILD.gn` |
| 安装路径 | `/system/lib64/module/` |
| 用途 | 提供 JS API 接口 |

### 2. libsharing_service.z.so

| 属性 | 值 |
|------|-----|
| 类型 | 服务共享库 |
| 源文件 | `services/BUILD.gn` |
| 安装路径 | `/system/lib64/` |
| 用途 | WFD 服务核心实现 |

### 3. SA 配置文件

| 文件 | 源目录 | 安装路径 |
|------|--------|----------|
| `5527.json` | `sa_profile/` | `/system/profile/` |
| `5528.json` | `sa_profile/` | `/system/profile/` |

### 4. 进程配置

| 文件 | 源目录 | 安装路径 |
|------|--------|----------|
| `sharing_service.cfg` | `services/etc/` | `/system/etc/` |

### 5. 其他模块产物

| 产物 | 类型 | 说明 |
|------|------|------|
| `libsharing_codec.z.so` | 共享库 | 编解码模块 |
| `libsharing_network.z.so` | 共享库 | 网络模块 |
| `libsharing_rtp.z.so` | 共享库 | RTP 协议 |
| `libsharing_rtsp.z.so` | 共享库 | RTSP 协议 |
| `libsharing_common.z.so` | 共享库 | 公共模块 |
| `libsharing_utils.z.so` | 共享库 | 工具模块 |

## 运行时加载关系

```
┌──────────────────────────────────────────────────────────────┐
│                        应用进程                               │
│                                                               │
│  @ohos.multimedia.sharingwfd (N-API)                        │
│         │                                                    │
│         ▼                                                    │
│  libsharingwfd_napi.z.so ◄── dlopen()                      │
│         │                                                    │
│         ├──► WfdSinkNapi / WfdSourceNapi                   │
│         │            │                                       │
│         │            ▼                                       │
│         │     libsharing_service.z.so ◄── IPC/RPC           │
│         │                                                    │
│         └──► libhilog.z.so ◄── 日志                         │
│                                                               │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC/RPC
┌──────────────────────────────────────────────────────────────┐
│                     sharing_service 进程                       │
│                                                               │
│  sa_main ◄─── loading                                       │
│      │                                                       │
│      ▼                                                       │
│  libsharing_service.z.so ◄── 加载                           │
│      │                                                       │
│      ├──► libsharing_codec.z.so ◄── 编解码                 │
│      │            │                                           │
│      │            ├──► libavcodec.z.so (FFmpeg)             │
│      │            └──► 硬件解码器                             │
│      │                                                       │
│      ├──► libsharing_network.z.so ◄── 网络通信             │
│      │                                                       │
│      ├──► libsharing_rtp.z.so ◄── RTP 协议                 │
│      │                                                       │
│      ├──► libsharing_rtsp.z.so ◄── RTSP 协议               │
│      │                                                       │
│      ├──► libhilog.z.so ◄── 日志                            │
│      │                                                       │
│      ├──► libaccesstoken_sdk.z.so ◄── 权限                 │
│      │                                                       │
│      ├──► libsurface.z.so ◄── 图形                          │
│      │                                                       │
│      └──► libaudio_*.z.so ◄── 音频                         │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## 数据目录

### 运行时数据

| 目录 | 用途 | 权限 |
|------|------|------|
| `/data/service/el1/public/database/sharingcodec/` | 业务数据 | 0777 |
| `/data/service/el1/public/database/sharingcodec/cache/` | 缓存 | 0777 |
| `/data/service/el1/public/database/sharingcodec/meta/` | 元数据 | 0777 |
| `/data/service/el1/public/database/sharingcodec/kvdb/` | KV 数据库 | 0777 |
| `/data/service/el1/public/database/sharingcodec/key/` | 密钥 | 0777 |
| `/data/service/el1/public/sharing_service/` | 服务数据 | 0700 |

**配置文件**: `services/etc/sharing_service.cfg:5-14`

## 服务启动流程

```
1. 系统启动
      │
      ▼
2. samgr 进程启动
      │
      ▼
3. 读取 SA 配置文件 (/system/profile/5527.json, 5528.json)
      │
      ▼
4. 加载 libsharing_service.z.so
      │
      ▼
5. 调用 sa_main 入口
      │
      ▼
6. 初始化 sharing_service
      │
      ├──► 初始化 Interaction 模块
      ├──► 初始化 ContextMgr
      ├──► 初始化 EventScheduler
      └──► 启动 TCP/UDP 监听
```

## 相关文档

- [GN 构建配置](05_GN_Build.md)
- [SA 配置](07_SA_Configuration.md)

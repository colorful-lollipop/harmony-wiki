# 目录结构与模块职责

## 顶层目录结构

```
/foundation/CastEngine/castengine_wifi_display  # 投播部件业务代码
├── figures                               # 图片资源
├── interfaces                            # 外部接口层
│   ├── kits                              # 应用接口（N-API）
│   └── innerkits                         # 系统内部件接口（Native C++）
├── frameworks                            # 部件无独立进程的实现
│   └── innerkitsimpl                     # native c++实现
├── sa_profile                            # SA 配置文件
├── services                              # 服务 C/S 实现
│   ├── interaction                       # 进程交互
│   ├── configure                         # 配置管理
│   ├── context                           # 业务容器
│   ├── agent                             # 业务代理
│   ├── mediachannel                      # 媒体通道
│   ├── mediaplayer                       # 播放渲染
│   ├── etc                               # 部件进程配置
│   ├── event                             # 事件中心
│   ├── impl                              # 业务实现
│   │   └── wfd                           # WFD 业务实现
│   │       ├── wfd_sink                 # Sink 端实现
│   │       ├── wfd_source               # Source 端实现
│   │       └── screen_capture           # 屏幕捕获
│   ├── inputback                         # 反控模块
│   ├── scheduler                         # 调度中心
│   ├── windowmgr                         # 窗口管理
│   ├── protocol                          # 协议库
│   │   ├── rtsp                          # RTSP 协议
│   │   └── rtp                           # RTP 协议
│   ├── codec                             # 编解码库
│   ├── network                           # 网络库
│   ├── extend                            # 引入库
│   ├── common                            # 公共类
│   └── utils                             # 工具类
├── tests                                 # 测试代码
├── bundle.json                           # 部件描述文件
├── BUILD.gn                              # 编译入口
└── config.gni                            # 构建配置
```

## 模块职责说明

| 模块名称 | 职责 | 关键文件 |
|----------|------|----------|
| **Interaction** | 框架层交互模块，负责与外部进程进行交互，基于 IPC 与 RPC 机制用于实现设备内和设备间的跨进程通信，支持与多个进程并发交互 | `services/interaction/` |
| **Scene** | 交互模块的业务实现部分，和 Interaction 实例共同完成对外交互和对内框架调用 | `services/interaction/scene/` |
| **ContextMgr** | 框架层业务容器模块，负责将不同的业务 Agent 关联在一起，用于实现收流，转发，发流等业务；每个业务容器实例可包含多个 Agent | `services/context/` |
| **Agent** | 业务在框架层的代理对象，负责信令层的交互。Agent 分为 Sink 端 Agent 和 Src 端 Agent。其中，Sink Agent 负责收流（获取媒体数据）业务，Src Agent 负责发流（输出媒体数据）业务 | `services/agent/` |
| **Session** | 业务控制层的具体实现，和 Agent 对象共同完成业务的信令交互 | `services/impl/wfd/wfd_sink/wfd_sink_session.cpp` |
| **Configuration** | 配置管理模块，设置框架和业务的配置数据，服务启动时加载 | `services/configuration/` |
| **EventScheduler** | 事件分发调度管理器，集中分发处理模块上报事件，采用异步线程池方式处理，不处理磁盘 IO 和网络 IO 等耗时操作 | `services/event/` |
| **MediachannelMgr** | 框架层媒体通道模块，管理媒体通道，每个媒体通道实例可实现媒体数据的接入、预览和发送；具备编解码能力、混流能力、流媒体数据包透传能力 | `services/mediachannel/` |
| **Consumer** | 获取媒体数据对象，可根据业务属性通过任何方式获取媒体数据，通常用于收流 | `services/impl/wfd/wfd_sink/wfd_rtp_consumer.cpp` |
| **Producer** | 输出媒体数据对象，可根据业务属性通过任何方式输出媒体数据，通常用于推流 | `services/impl/wfd/wfd_source/wfd_rtp_producer.cpp` |
| **ServiceMgr** | 框架层服务管理模块，服务监听的管理模块，每个 service 实例用于对指定的端口进行 tcp 或者 udp 监听，可与外部进程或设备进行数据交互 | `services/utils/` |
| **InputBack** | 反控模块，跨设备反控及坐标变化等处理 | `services/inputback/` |
| **WindowMgr** | 框架层窗口管理模块，窗口实例用于自触发预览窗口时使用 | `services/windowmgr/` |
| **Protocol** | 实现 rtsp、rtp、wfd、dlna、uibc 等协议封装，用于对外协议交互与对接 | `services/protocol/` |
| **Codec** | 媒体数据的封装与解封装，编码与解码，硬解加速等 | `services/codec/` |
| **Network** | 网络协议封装，包括 tcp/udp 的服务端、客户端等 | `services/network/` |

## 接口层说明

### 应用接口层（kits）

提供 N-API 接口供应用开发者使用，JS 模块名为 `multimedia.SharingWfd`。

- 路径: `interfaces/kits/js/wfd/`
- 实现: `frameworks/kitsimpl/js/wfd/`

### 系统内部件接口层（innerkits）

提供 Native C++ 接口供系统级组件使用。

- 路径: `interfaces/innerkits/native/wfd/`
- 关键头文件:
  - `wfd.h` - WFD 公共接口
  - `wfd_sink.h` - Sink 接口
  - `wfd_source.h` - Source 接口

## 相关文档

- [项目概览](00_Overview.md)
- [逻辑架构](02_Architecture.md)
- [N-API 接口](03_N-API_Interface.md)

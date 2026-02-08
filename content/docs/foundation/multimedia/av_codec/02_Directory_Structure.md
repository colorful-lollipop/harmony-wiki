# 02_目录结构

## 顶层结构

```
/foundation/multimedia/av_codec/
├── .git/                    # Git 版本控制
├── .gitignore              # Git 忽略配置
├── .gitattributes          # Git 属性配置
├── BUILD.gn                # 根构建入口
├── bundle.json             # 部件描述文件 (11,933 bytes)
├── config.gni              # GN 配置 (6,100 bytes)
├── LICENSE                 # Apache 2.0 许可证
├── OAT.xml                 # 开源合规配置
├── hisysevent.yaml         # 华为 HiSysEvent 配置
├── README.md               # 项目说明
├── README_zh.md           # 中文项目说明
├── figures/                # 文档图片资源
│
├── frameworks/             # 框架层代码（无独立进程）
│   └── native/             # Native C++ 实现
│       ├── capi/           # C-API 实现
│       │   ├── avcodec/    # 编解码器 C-API
│       │   ├── avdemuxer/  # 解封装 C-API
│       │   ├── avmuxer/    # 封装 C-API
│       │   ├── avsource/   # 源 C-API
│       │   └── common/     # 公共 C-API 组件
│       ├── avcodeclist/    # 编解码能力列表
│       ├── avdemuxer/      # 解封装框架
│       ├── avmuxer/        # 封装框架
│       ├── avsource/       # 媒体源框架
│       └── common/         # 框架公共组件
│
├── interfaces/             # 对外接口层
│   ├── inner_api/          # 系统内部件接口（Inner API）
│   │   └── native/         # Native Inner API 头文件
│   └── kits/               # 应用接口
│       └── c/              # C-API 头文件 (对外)
│           ├── native_avcodec_base.h      # 基础类型/回调
│           ├── native_avcodec_videodecoder.h
│           ├── native_avcodec_videoencoder.h
│           ├── native_avcodec_audiodecoder.h
│           ├── native_avcodec_audioencoder.h
│           ├── native_avcodec_audiocodec.h
│           ├── native_avdemuxer.h
│           ├── native_avmuxer.h
│           ├── native_avsource.h
│           ├── native_avcapability.h
│           ├── native_cencinfo.h
│           └── BUILD.gn
│
├── services/               # 服务层代码（独立进程）
│   ├── include/            # 服务头文件
│   ├── etc/                # SA 配置文件
│   ├── dfx/                # DFX/调试代码
│   ├── utils/              # 服务通用工具
│   ├── engine/             # 引擎层
│   │   ├── base/           # 功能基类
│   │   ├── codec/          # 编解码实现
│   │   ├── codeclist/      # 能力列表
│   │   ├── common/         # 公共实现
│   │   ├── demuxer/        # 解封装实现
│   │   ├── factory/        # 工厂实现
│   │   ├── muxer/          # 封装实现
│   │   ├── plugin/         # 插件实现
│   │   └── source/         # 媒体源实现
│   ├── media_engine/       # 媒体引擎
│   │   ├── filters/        # 滤镜/过滤器
│   │   ├── modules/        # 模块
│   │   └── plugins/        # 插件
│   └── services/           # SA IPC 层
│       ├── codec/          # 编解码 IPC
│       ├── codeclist/      # 能力列表 IPC
│       ├── demuxer/        # 解封装 IPC
│       ├── muxer/          # 封装 IPC
│       ├── sa_avcodec/     # 主 SA IPC
│       └── source/         # 媒体源 IPC
│
└── test/                   # 测试代码（本文档不覆盖）
    ├── nativedemo/
    ├── unittest/
    ├── fuzztest/
    └── moduletest/
```

## 模块职责说明

### frameworks/native

**职责**: 提供跨进程边界的框架层，桥接 C-API 和服务层

| 子目录 | 职责 | 证据 |
|--------|------|------|
| `capi/` | C-API 实现，暴露给外部应用 | `interfaces/kits/c/BUILD.gn:17-36` |
| `avcodeclist/` | 编解码能力列表框架 | - |
| `avdemuxer/` | 解封装框架 | - |
| `avmuxer/` | 封装框架 | - |
| `avsource/` | 媒体源框架 | - |
| `common/` | 框架公共组件 | - |

### interfaces/kits/c

**职责**: 对外 C-API 头文件，应用层通过 N-API 间接调用

| 头文件 | 导出能力 | 证据 |
|--------|---------|------|
| `native_avcodec_base.h` | 基础类型/回调/错误码 | `native_avcodec_base.h:17-37` |
| `native_avcodec_videodecoder.h` | 视频解码器 | `native_avcodec_videodecoder.h` |
| `native_avcodec_videoencoder.h` | 视频编码器 | `native_avcodec_videoencoder.h` |
| `native_avdemuxer.h` | 解封装器 | `native_avdemuxer.h` |
| `native_avmuxer.h` | 封装器 | `native_avmuxer.h` |
| `native_avsource.h` | 媒体源 | `native_avsource.h` |
| `native_avcapability.h` | 能力查询 | `native_avcapability.h` |

### services/

**职责**: 服务端实现，提供 IPC 接口和编解码核心逻辑

| 目录 | 职责 | 关键类 |
|------|------|--------|
| `services/sa_avcodec/` | SA IPC 层，处理 SA 级别的 IPC 请求 | `AVCodecServer` |
| `services/codec/` | 编解码 IPC 实现 | `CodecServiceStub` |
| `services/codeclist/` | 能力列表 IPC | `CodecListServiceStub` |
| `services/engine/` | 编解码核心逻辑 | `Hcodec`, `Fcodec` |
| `services/media_engine/` | 媒体引擎框架 | `MediaCodec` |
| `services/dfx/` | 调试与监控 | - |
| `services/utils/` | 通用工具 | - |

## 目录规模统计

| 目录 | 文件数 | 说明 |
|------|--------|------|
| `frameworks/native/capi/` | ~10 | C-API 实现 |
| `interfaces/kits/c/` | ~15 | C-API 头文件 |
| `interfaces/inner_api/native/` | ~30 | Inner API 头文件 |
| `services/*/` | ~500+ | 服务实现代码 |
| **总计（非测试）** | **~600** | - |

---

**相关文档**: [架构设计](03_Architecture.md) | [C-API 文档](04_C_API.md)

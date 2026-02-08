# Audio Lite Wiki - 文档导航

本文档为 [foundation/multimedia/audio_lite](../README.md) 项目的工程 Wiki，涵盖架构、API、构建、安全等维度。

## 文档索引

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [README](README.md) | 本 Wiki 的覆盖范围、更新方式 | 2 min |
| [Overview](01_Overview.md) | 项目定位、能力概述、运行环境 | 3 min |
| [Architecture](02_Architecture.md) | 组件图、数据流、线程模型、调用链 | 10 min |
| [API Reference](03_API_Reference.md) | C++ 对外接口、N-API 清单（本项目无 N-API） | 8 min |
| [Build System](04_Build_System.md) | GN targets、编译产物、依赖关系 | 5 min |
| [Security Review](05_Security_Review.md) | 攻击面分析、风险点、修复建议 | 7 min |
| [Troubleshooting](06_Troubleshooting.md) | 常见构建/运行/调试问题 | 5 min |

## 新人阅读路线

```
1. 先读 [Overview](01_Overview.md) 了解项目定位
2. 再读 [Architecture](02_Architecture.md) 理解整体架构
3. 根据需要查阅 [API Reference](03_API_Reference.md)
4. 开发/编译时参考 [Build System](04_Build_System.md)
5. 安全相关评审参考 [Security Review](05_Security_Review.md)
6. 遇到问题查阅 [Troubleshooting](06_Troubleshooting.md)
```

## 快速跳转

### 核心类与模块

| 模块 | 关键类 | 文件路径 |
|------|--------|----------|
| 对外 API | `AudioCapturer` | [interfaces/kits/audio_capturer.h](../../interfaces/kits/audio_capturer.h) |
| 框架层 | `AudioCapturerClient` | [frameworks/binder/audio_capturer_client.h](../../frameworks/binder/audio_capturer_client.h) |
| 服务端 | `AudioCapturerServer` | [services/server/include/audio_capturer_server.h](../../services/server/include/audio_capturer_server.h) |
| 服务注册 | `AudioCapturerService` | [services/server/src/audio_capturer_samgr.cpp](../../services/server/src/audio_capturer_samgr.cpp) |
| 实现层 | `AudioCapturerImpl` | [services/impl/audio_capturer_impl.h](../../services/impl/audio_capturer_impl.h) |

### 构建产物

| 产物 | 类型 | 位置 |
|------|------|------|
| `libaudio_capturer_lite.so` | 动态库 | frameworks |
| `libaudio_capturer_impl.so` | 动态库 | services |
| `libaudio_capturer_server.a` | 静态库 | services |

### 关键配置

| 配置项 | 文件 |
|--------|------|
| bundle.json | [bundle.json](../../bundle.json) |
| frameworks BUILD.gn | [frameworks/BUILD.gn](../../frameworks/BUILD.gn) |
| services BUILD.gn | [services/BUILD.gn](../../services/BUILD.gn) |

## 版本信息

| 属性 | 值 |
|------|-----|
| Wiki 版本 | 1.0 |
| 生成时间 | 2026-02-06 |
| 项目版本 | 3.1 |
| 适配系统 | mini, small |

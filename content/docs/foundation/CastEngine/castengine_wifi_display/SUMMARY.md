# castengine_wifi_display Wiki 导航

## 新人阅读路线

建议新人按以下顺序阅读本文档：

1. **[00_Overview.md](00_Overview.md)** - 项目概览，了解核心能力和定位
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构和模块职责
3. **[02_Architecture.md](02_Architecture.md)** - 逻辑架构和组件图
4. **[03_N-API_Interface.md](03_N-API_Interface.md)** - JS API 接口（面向应用开发者）
5. **[04_Inner_API.md](04_Inner_API.md)** - Native C++ 接口（面向系统开发者）
6. **[05_GN_Build.md](05_GN_Build.md)** - GN 构建配置和 Targets
7. **[06_Build_Artifacts.md](06_Build_Artifacts.md)** - 编译产物和安装路径
8. **[07_SA_Configuration.md](07_SA_Configuration.md)** - SA 配置和服务
9. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险评审

## 附录

- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
- **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 关键配置项

## 快速索引

### 按功能分类

| 功能 | 文档 | 关键文件 |
|------|------|----------|
| WFD Sink（被投端） | N-API 接口 | `wfd_napi_sink.cpp`, `wfd_sink.h` |
| WFD Source（主投端） | N-API 接口 | `wfd_napi_source.cpp`, `wfd_source.h` |
| 进程交互 | Inner API | `services/interaction/` |
| 媒体通道 | Inner API | `services/mediachannel/` |
| 编解码 | Inner API | `services/codec/` |
| 协议栈 | Inner API | `services/protocol/` |

### 按文件类型分类

| 类型 | 位置 |
|------|------|
| N-API 头文件 | `interfaces/kits/js/wfd/include/` |
| Inner API 头文件 | `interfaces/innerkits/native/wfd/include/` |
| 服务实现 | `services/impl/` |
| 构建配置 | `BUILD.gn`, `config.gni` |
| SA 配置 | `sa_profile/`, `services/etc/` |

# ArkXtest Wiki 导航

## 新人阅读路线

```
1. 先读 [01_Overview.md] 了解项目定位与核心能力
2. 再读 [02_Architecture.md] 理解整体架构与数据流
3. 根据需要查阅 [03_NAPI.md] 或 [04_InnerAPI.md]
4. 如需构建，查阅 [05_GN_Build.md] 和 [06_Build_Artifacts.md]
5. 遇到问题查阅 [08_Troubleshooting.md]
6. 安全相关查阅 [07_Security.md]
```

## 文档索引

### 入门指南

| 文档 | 描述 |
|------|------|
| [README.md](README.md) | 文档概述、使用说明、术语表 |
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |

### 架构设计

| 文档 | 描述 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、关键时序（Mermaid） |

### API 参考

| 文档 | 描述 |
|------|------|
| [03_NAPI.md](03_NAPI.md) | N-API/ANI 接口清单、参数校验、错误码 |
| [04_InnerAPI.md](04_InnerAPI.md) | 内部模块接口、依赖方向、生命周期 |

### 构建部署

| 文档 | 描述 |
|------|------|
| [05_GN_Build.md](05_GN_Build.md) | BUILD.gn targets 梳理、依赖关系 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物、安装路径、运行时加载关系 |

### 安全与运维

| 文档 | 描述 |
|------|------|
| [07_Security.md](07_Security.md) | 攻击面分析、信任边界、风险点与修复建议 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见构建/运行/调试问题与定位路径 |

## 组件速查

### UiTest 快速链接

| 主题 | 链接 |
|------|------|
| N-API 注册点 | [03_NAPI.md#uitest-n-api](03_NAPI.md#uitest-n-api) |
| 核心模块 | [04_InnerAPI.md#uitest-核心模块](04_InnerAPI.md#uitest-核心模块) |
| 构建配置 | [05_GN_Build.md#uitest](05_GN_Build.md#uitest) |
| 服务端 | [06_Build_Artifacts.md#uitest-产物](06_Build_Artifacts.md#uitest-产物) |

### PerfTest 快速链接

| 主题 | 链接 |
|------|------|
| N-API 注册点 | [03_NAPI.md#perftest-n-api](03_NAPI.md#perftest-n-api) |
| 性能指标 | [04_InnerAPI.md#perftest-核心模块](04_InnerAPI.md#perftest-核心模块) |
| 构建配置 | [05_GN_Build.md#perftest](05_GN_Build.md#perftest) |
| 服务端 | [06_Build_Artifacts.md#perftest-产物](06_Build_Artifacts.md#perftest-产物) |

### TestServer 快速链接

| 主题 | 链接 |
|------|------|
| SA 配置 | [02_Architecture.md#testserver-sa-架构](02_Architecture.md#testserver-sa-架构) |
| IPC 接口 | [03_NAPI.md#testserver-ipc-接口](03_NAPI.md#testserver-ipc-接口) |
| 构建配置 | [05_GN_Build.md#testserver](05_GN_Build.md#testserver) |
| 权限配置 | [06_Build_Artifacts.md#testserver-产物](06_Build_Artifacts.md#testserver-产物) |

## 相关文档链接

- [OpenHarmony 测试框架官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/application-test/uitest-guidelines.md)
- [UiTest API 声明](interface/sdk-js/api/@ohos.UiTest.d.ts)
- [PerfTest API 声明](interface/sdk-js/api/@ohos.test.PerfTest.d.ts)

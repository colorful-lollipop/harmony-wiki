# SUMMARY - 多模态输入子系统 Wiki 导航

## 快速开始

- [README](README.md) - 项目概述与快速入门
- [README_zh.md](../README_zh.md) - 中文项目说明

## 核心文档

1. **[01_Overview.md](01_Overview.md)** - 项目定位与核心能力
   - 项目背景与定位
   - 核心功能模块
   - 运行环境与依赖
   - 关键概念说明

2. **[02_NAPI_Reference.md](02_NAPI_Reference.md)** - N-API 接口参考
   - 14个N-API模块完整清单
   - API参数与返回值
   - 权限要求
   - 错误码说明

3. **[03_Architecture.md](03_Architecture.md)** - 架构设计与数据流
   - 系统架构图
   - IPC通信机制
   - 事件处理流程
   - 线程模型

4. **[04_GN_Build.md](04_GN_Build.md)** - GN 编译配置
   - 根构建入口 (BUILD.gn)
   - 关键 targets 清单
   - 编译产物说明
   - 条件编译配置

5. **[05_Security_Review.md](05_Security_Review.md)** - 安全风险评审
   - 攻击面分析
   - 信任边界
   - 风险点识别
   - 修复建议

6. **[06_Inner_API.md](06_Inner_API.md)** - 内部模块接口
   - Inner API 清单
   - 模块依赖关系
   - 稳定性标注

## 附录

- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
  - 事件注入调用链
  - 设备查询调用链
  - IPC 通信调用链

- **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 配置开关
  - feature flags 清单
  - 编译开关说明
  - 产品适配配置

## 新人阅读推荐顺序

```
┌────────────────────────────────────────────────────────────┐
│                    新人阅读路线图                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ① README.md (5分钟)                                       │
│     → 了解项目整体                                          │
│                                                            │
│  ② 01_Overview.md (10分钟)                                │
│     → 理解核心定位                                          │
│                                                            │
│  ③ 02_NAPI_Reference.md (15分钟)                          │
│     → 掌握对外接口                                          │
│                                                            │
│  ④ 03_Architecture.md (15分钟)                             │
│     → 理解内部架构                                          │
│                                                            │
│  ⑤ 04_GN_Build.md (10分钟)                                │
│     → 掌握编译构建                                          │
│                                                            │
│  ⑥ 05_Security_Review.md (10分钟)                         │
│     → 了解安全要点                                          │
│                                                            │
│  预计总时间: 约 65 分钟                                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## 模块速查

### N-API 模块快速入口

| 功能 | 文档章节 | 关键文件 |
|------|---------|---------|
| 事件注入 | 02_NAPI_Reference.md#inputEventClient | `frameworks/napi/input_event_client/` |
| 设备管理 | 02_NAPI_Reference.md#inputDevice | `frameworks/napi/input_device/` |
| 事件监控 | 02_NAPI_Reference.md#inputMonitor | `frameworks/napi/input_monitor/` |
| 指针配置 | 02_NAPI_Reference.md#pointer | `frameworks/napi/pointer/` |
| 红外控制 | 02_NAPI_Reference.md#infraredEmitter | `frameworks/napi/infrared_emitter/` |

### 服务组件快速入口

| 功能 | 文档章节 | 关键文件 |
|------|---------|---------|
| 主服务 | 03_Architecture.md#mmiservice | `service/module_loader/` |
| 事件处理 | 03_Architecture.md#event-handler | `service/event_handler/` |
| 设备管理 | 03_Architecture.md#device-manager | `service/device_manager/` |
| 意图服务 | 03_Architecture.md#intention | `intention/` |
| 权限校验 | 05_Security_Review.md | `service/permission_helper/` |

## 常见问题

- **Q: 如何添加新的 N-API 模块？**
  → 参考 02_NAPI_Reference.md#new-module-creation

- **Q: 如何调试事件注入问题？**
  → 参考 03_Architecture.md#debugging

- **Q: 如何配置新的设备支持？**
  → 参考 04_GN_Build.md#device-config

- **Q: 安全权限如何申请？**
  → 参考 05_Security_Review.md#permission-request

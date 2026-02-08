# Hiview Wiki - 全站导航

## 新人阅读路线

建议按以下顺序阅读：

```
1. 00_Overview.md (项目概览)
   ↓
2. 01_Architecture.md (架构说明)
   ↓
3. 02_NAPI_Reference.md (N-API 接口 - 按需)
   ↓
4. 04_Build_System.md (构建系统)
   ↓
5. 05_Artifacts.md (编译产物)
   ↓
6. 06_Security_Review.md (安全评审 - 按需)
```

## 文档目录

### 核心文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [README.md](README.md) | 文档说明 | 覆盖范围、更新方式 |
| [00_Overview.md](00_Overview.md) | 项目概览 | 定位、边界、能力、依赖 |
| [01_Architecture.md](01_Architecture.md) | 架构说明 | 组件图、数据流、线程模型 |
| [02_NAPI_Reference.md](02_NAPI_Reference.md) | N-API 接口 | JS API 清单、参数、错误码 |
| [03_Inner_API.md](03_Inner_API.md) | 内部 API | 模块接口、依赖关系 |
| [04_Build_System.md](04_Build_System.md) | 构建系统 | GN Targets、Feature Flags |
| [05_Artifacts.md](05_Artifacts.md) | 编译产物 | .so/.a/.hap、安装路径 |
| [06_Security_Review.md](06_Security_Review.md) | 安全评审 | 攻击面、风险、修复建议 |
| [07_Troubleshooting.md](07_Troubleshooting.md) | 常见问题 | 构建/运行/调试问题 |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |

## 代码跳转

### 核心入口

| 符号 | 文件路径 | 说明 |
|------|----------|------|
| `main()` | `main.cpp:29` | 应用入口 |
| `HiviewPlatform` | `core/include/hiview_platform.h` | 平台核心类 |
| `HiviewService` | `service/hiview_service.cpp` | 主服务 |
| `Plugin` | `base/include/plugin.h` | 插件基类 |
| `Event` | `base/include/event.h` | 事件基类 |

### N-API 模块

| 模块名 | JS API | C++ 文件 |
|--------|--------|----------|
| `faultLogger` | `querySelfFaultLog`, `addFaultLog` | `plugins/faultlogger/interfaces/js/napi/napi_faultlogger.cpp` |
| `logLibrary` | `list`, `copy`, `move`, `remove` | `interfaces/js/napi/src/napi_hiview_js.cpp` |

### System Ability

| SA ID | 服务名 | 注册文件 |
|-------|--------|----------|
| `DFX_SYS_HIVIEW_ABILITY_ID` | HiviewServiceAbility | `adapter/service/server/src/hiview_service_ability.cpp` |
| `DFX_SYS_EVENT_SERVICE_ABILITY_ID` | SysEventServiceOhos | `adapter/plugins/eventservice/service/idl/src/sys_event_service_ohos.cpp` |
| `DFX_FAULT_LOGGER_ABILITY_ID` | FaultloggerServiceOhos | `plugins/faultlogger/service/idl/faultlogger_service_ohos.cpp` |

## 相关资源

- [OpenHarmony DFX 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [hiviewdfx_hilog](../hiviewdfx_hilog/README_zh.md)
- [hiviewdfx_hisysevent](../hiviewdfx_hisysevent/README_zh.md)
- [hiviewdfx_faultloggerd](../hiviewdfx_faultloggerd/README_zh.md)

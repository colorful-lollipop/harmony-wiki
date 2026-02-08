# 项目概览

## 组件定位

`frame_aware_sched` 是 OpenHarmony **Resource Schedule Subsystem** 的核心组件，负责感知应用帧绘制状态并调整内核调度策略，以确保系统 CPU 调度的供给。

## 核心能力

1. **帧信息感知**：接收 JS-UI 子系统与 Graphic 子系统的帧绘制消息，分析帧率信息
2. **调度策略调整**：根据帧状态更新进程调度组，调整内核调度参数（RTG）
3. **场景识别**：识别滑动、视频、游戏等场景，提供基于场景的细粒度调度
4. **关键线程保障**：识别关键线程（绘制帧线程、渲染线程），提升其资源供给

## 运行环境

| 项目 | 要求 |
|------|------|
| **系统类型** | standard（标准系统） |
| **依赖子系统** | resource_schedule_service, ace_ace_engine, graphic_graphic_2d, aafwk_standard |
| **ROM 限制** | 2048KB |
| **RAM 限制** | 10240KB |

## 关键概念

| 概念 | 说明 |
|------|------|
| **RTG (Related-Thread-Group)** | 相关线程组，用于将相关线程绑定调度 |
| **Frame Aware Collector** | 帧信息收集器，负责帧事件处理、滑动场景策略 |
| **Frame Aware Policy** | 帧感知策略，负责应用状态管理、RTG 管理 |
| **FFRT** | Fast Framework Runtime，任务队列框架 |

## 启用方式

系统开发者可通过配置产品定义 JSON 文件启用/禁用本组件：

```json
"resourceschedule:frame_aware_sched":{}
```

文件位置：`/productdefine/common/products/{product}/base/productdefine/common/products/`

## 相关仓库

- [resource_schedule_service](https://gitee.com/openharmony/resourceschedule_resource_schedule_service)
- [ace_ace_engine](https://gitee.com/openharmony/ace_ace_engine)
- [graphic_graphic_2d](https://gitee.com/openharmony/graphic_graphic_2d)
- [aafwk_standard](https://gitee.com/openharmony/aafwk_standard)

# 对外 API

> **说明**: 本章内容适用于需要暴露 N-API 的组件。**safwk_lite 是一个纯 C 语言框架，不暴露任何 JS/N-API**。

## N-API 清单

本组件 **无 N-API 暴露**，原因如下：

| 原因 | 说明 |
|------|------|
| 定位 | safwk_lite 是系统底层框架，提供进程容器能力 |
| 语言 | 纯 C 实现，无 C++ 绑定层 |
| 调用方式 | 通过 Samgr IPC 进行进程间通信 |

## 替代方案

如需与 safwk_lite 交互，请通过以下方式：

### 1. Samgr IPC

通过 Samgr Lite 框架进行进程间通信：

```c
// 示例：获取系统能力
// 请参考 samgr_lite 相关文档
```

### 2. 配置文件

通过配置文件指定系统能力信息：

```json
{
  "process": {
    "name": "foundation",
    "services": [
      {
        "name": "abilityms"
      },
      {
        "name": "bundlems"
      }
    ]
  }
}
```

## 相关文档

> **注意**: 以下链接指向 OpenHarmony 其他子系统文档，请确保对应子系统已克隆到正确位置。

- [Samgr Lite 接口文档](https://gitee.com/openharmony/systemabilitymgr_samgr_lite)
- [IPC 通信机制](https://gitee.com/openharmony/communication_ipc_lite)
- [系统能力开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/系统能力.md)

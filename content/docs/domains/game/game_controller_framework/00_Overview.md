# 00_Overview - 项目概览

## 目的

本文档提供 GameController Framework 的快速概览，帮助新人快速理解项目全貌。

## 适用范围

- 所有项目相关人员（开发者、架构师、测试、运维）
- OpenHarmony API Level 21 及以上

## 快速理解

### 项目是什么

GameController Framework 是 OpenHarmony 游戏子系统中的游戏控制器框架，提供：

1. **游戏开发者 API** - 监听游戏外设连接和输入事件
2. **终端厂商 InnerAPI** - 配置外设识别信息和输入转触控映射

### 核心价值

- **游戏开发者**: 无需适配，即可支持键盘/鼠标等游戏外设（通过输入转触控）
- **终端厂商**: 灵活配置外设识别和按键映射规则
- **系统**: 按需拉起 SA，非常驻，节省资源

### 技术栈

| 层次 | 技术 |
|------|------|
| 编程语言 | C++ 11+ |
| 对外接口 | CAPI (C Native API) |
| IPC 机制 | OpenHarmony IPC + System Ability |
| 事件系统 | 多模态输入 (MultiModalInput) + Window Framework |
| 配置存储 | JSON 文件 |

### 系统能力

- **SysCap**: `SystemCapability.Game.GameController`
- **SA ID**: 8450
- **子系统**: game
- **组件**: game_controller_framework

## 核心模块关系

```
┌─────────────────────────────────────────────────────────────┐
│                  应用进程                              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │      libohgame_controller.z.so (CAPI)          │  │
│  └───────────────────────┬─────────────────────────┘  │
│                        │                               │
│  ┌───────────────────────▼─────────────────────────┐  │
│  │  libgamecontroller_event.z.so (事件监听)      │  │
│  │  - InputMonitor (对接 Window)                │  │
│  │  - DeviceMonitor (对接 MultiModalInput)       │  │
│  └───────────────────────┬─────────────────────────┘  │
│                        │                               │
│  ┌───────────────────────▼─────────────────────────┐  │
│  │  libgamecontroller_client.z.so (InnerAPI)       │  │
│  │  - GameControllerClient (IPC 客户端)          │  │
│  └───────────────────────┬─────────────────────────┘  │
└────────────────────────│────────────────────────────────┘
                         │ IPC
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              GameControllerSA 进程                      │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  libgamecontroller_server.z.so                 │ │
│  │  - DeviceManager (设备识别)                    │ │
│  │  - KeyMappingManager (配置管理)               │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## 启动场景

GameControllerSA 是按需启动的非常驻进程，以下场景会拉起：

1. **外设连接时**: 通过 samgr 拉起 SA 进行设备类型识别
2. **配置变更时**: 终端厂商服务通过 InnerAPI 配置时拉起 SA

证据: `frameworks/native/sa_client/include/gamecontroller_server_client.h:34-35`

## 编译产物

| 产物 | 说明 | 目标 |
|------|------|--------|
| libohgame_controller.z.so | CAPI 接口库 | 游戏开发者使用 |
| libgamecontroller_event.z.so | 事件监听库 | Window Framework 加载 |
| libgamecontroller_client.z.so | InnerAPI 库 | 终端厂商使用 |
| libgamecontroller_server.z.so | System Ability 服务 | 独立进程运行 |

证据: `README_zh.md:126-129`

## 快速开始

### 游戏开发者

```c
#include <OHGameController.h>

// 1. 注册设备监听
OH_GameDevice_RegisterDeviceMonitor(deviceCallback);

// 2. 注册按键监听
OH_GamePad_ButtonA_RegisterButtonInputMonitor(buttonCallback);

// 3. 处理事件
void OnDeviceEvent(GameDevice_DeviceEvent event) {
    // 处理设备连接/断开
}

void OnButtonEvent(GamePad_ButtonEvent event) {
    // 处理按键输入
}
```

### 终端厂商（使用 InnerAPI）

```cpp
#include "gamecontroller_server_client.h"

// 获取 InnerAPI 客户端
auto client = GameControllerServerClient::GetInstance();

// 识别设备
client->IdentifyDevice(deviceInfos, identifyResult);

// 同步配置
client->SyncSupportKeyMappingGames(true, gameInfos);
```

## 关键结论

1. **当前仅提供 CAPI**，ArkTS 接口未来规划
2. **输入转触控是核心特性**，无需游戏厂商适配
3. **SA 按需启动**，非常驻进程
4. **配置文件驱动**: device_config.json, game_support_key_mapping.json 等

## 相关文档

- [01_Project_Position.md](./01_Project_Position.md) - 详细项目定位
- [03_Architecture.md](./03_Architecture.md) - 完整架构说明
- [04_External_CAPI.md](./04_External_CAPI.md) - CAPI 接口详解

---

**版本**: 1.0 | **更新时间**: 2026-02-06

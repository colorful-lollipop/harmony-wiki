# 01_Project_Position - 项目定位与边界

## 目的

本文档详细说明 GameController Framework 的项目定位、边界、核心能力、运行环境和关键概念。

## 适用范围

- 架构师、产品经理、技术决策者
- 需要深入理解框架边界和能力的开发人员

## 项目定位

### 在 OpenHarmony 中的位置

GameController Framework 属于 **Game 子系统**，提供游戏外设管理能力。

**证据**:
- `bundle.json:18-20`: `"subsystem": "game"`, `"syscap": ["SystemCapability.Game.GameController"]`
- `sa_profile/8450.json:2-3`: SA 进程名为 "gamecontroller_server"

### 目标用户

| 用户类型 | 需求 | 提供能力 |
|---------|------|---------|
| **游戏开发者** | 监听外设连接和输入事件 | CAPI 接口（设备信息、按键监听）|
| **终端设备厂商** | 配置外设识别和输入转触控 | InnerAPI（设备识别、配置同步）|
| **系统** | 按需启动服务，节省资源 | System Ability 按需加载机制 |

### 核心价值主张

1. **对游戏开发者**:
   - 简化外设接入，无需适配即可支持键盘、鼠标
   - 统一的输入事件监听接口

2. **对终端厂商**:
   - 灵活的设备识别配置
   - 可自定义按键映射规则
   - InnerAPI 支持动态配置更新

3. **对系统**:
   - SA 按需启动，非常驻，节省内存
   - 配置文件驱动，易于维护

## 边界定义

### 功能边界

| 功能范围 | 包含 | 不包含 |
|---------|------|--------|
| **设备管理** | 外设上线/下线监听、设备信息查询 | 设备驱动实现（依赖 MultiModalInput）|
| **输入监听** | 手柄按键/轴事件监听、键盘/鼠标事件 | 输入事件采集（依赖 Window Framework）|
| **输入转触控** | 按键映射规则配置、输入事件转换 | 触控事件分发（依赖 Input 模块）|
| **配置管理** | 配置文件读写、配置同步 | 配置编辑器 UI |

### 技术边界

| 技术维度 | 包含 | 不包含 |
|----------|------|--------|
| **对外接口** | CAPI (C Native API) | N-API (ArkTS) - 未来规划 |
| **IPC 机制** | OpenHarmony IPC (Binder) | 自定义 RPC 协议 |
| **数据存储** | JSON 配置文件 | 数据库存储 |
| **跨设备** | 单设备管理 | 分布式协同 |

### 依赖边界

**上层依赖**:
- `bundle.json:28-43` 明确声明的组件依赖
  - hilog, ipc, safwk, samgr
  - c_utils, common_event_service, ffrt
  - json, input, window_manager
  - eventhandler, init, access_token, bundle_framework

**下层服务**:
- MultiModalInput: 多模态输入服务（设备管理）
- Window Manager: 窗口框架（输入事件）
- System Ability Framework: SA 管理
- Bundle Manager: 包信息查询

## 核心能力

### 1. GameDevice - 设备管理

**能力**: 查询在线设备、注册设备监听

**证据**: `interfaces/kits/c/game_device.h:58-86`

```c
// 获取所有在线设备
GameController_ErrorCode OH_GameDevice_GetAllDeviceInfos(
    GameDevice_AllDeviceInfos** allDeviceInfos);

// 注册设备监听
GameController_ErrorCode OH_GameDevice_RegisterDeviceMonitor(
    GameDevice_DeviceMonitorCallback deviceMonitorCallback);

// 注销设备监听
GameController_ErrorCode OH_GameDevice_UnregisterDeviceMonitor(void);
```

### 2. GamePad - 手柄输入

**能力**: 监听手柄按键和轴事件（肩键、扳机键、菜单键、十字键、摇杆等）

**证据**: `interfaces/kits/c/game_pad.h:52-438`

支持的按键类型：
- 肩键: LeftShoulder, RightShoulder
- 扳机键: LeftTrigger, RightTrigger (支持按钮和轴事件)
- 功能键: ButtonMenu, ButtonHome
- 动作键: ButtonA, ButtonB, ButtonX, ButtonY, ButtonC
- 十字键: Dpad_UpButton, Dpad_DownButton, Dpad_LeftButton, Dpad_RightButton
- 摇杆: LeftThumbstick, RightThumbstick (支持按钮和轴事件)

### 3. Input-to-Touch - 输入转触控

**能力**: 将键盘/鼠标/手柄输入转换为屏幕触控事件

**配置文件**:
- `etc/config/game_support_key_mapping.json`: 支持转触控的游戏列表
- `etc/config/default_key_mapping.json`: 默认按键映射配置
- `etc/config/custom_key_mapping.json`: 自定义按键映射配置

**触发方式**: README_zh.md:54 - 同时按下 Q、W、P 键打开配置界面

**证据**:
- `README_zh.md:52-55`: 基于输入事件和设备类别判断是否需要发送编辑输入转触控配置的通知

### 4. InnerAPI - 内部接口

**能力**: 供终端厂商调用，用于设备识别和配置管理

**证据**: `frameworks/native/sa_client/include/gamecontroller_server_client.h:25-99`

主要接口：
```cpp
// 设备识别
int32_t IdentifyDevice(const std::vector<DeviceInfo> &deviceInfos,
                   std::vector<DeviceInfo> &identifyResult);

// 同步已识别设备信息
int32_t SyncIdentifiedDeviceInfos(const std::vector<IdentifiedDeviceInfo> &deviceInfos);

// 同步支持转触控的游戏
int32_t SyncSupportKeyMappingGames(bool isSyncAll, const std::vector<GameInfo> &gameInfos);

// 获取按键映射配置
int32_t GetGameKeyMappingConfig(const GetGameKeyMappingInfoParam &param,
                              GameKeyMappingInfo &gameKeyMappingInfo);

// 设置自定义按键映射（仅系统服务）
int32_t SetCustomGameKeyMappingConfig(const GameKeyMappingInfo &gameKeyMappingInfo);

// 设置默认按键映射（仅系统服务）
int32_t SetDefaultGameKeyMappingConfig(const GameKeyMappingInfo &gameKeyMappingInfo);

// 广播设备信息
int32_t BroadcastDeviceInfo(const GameInfo &gameInfo, const DeviceInfo &deviceInfo);

// 广播打开配置页面
int32_t BroadcastOpenTemplateConfig(const GameInfo &gameInfo, const DeviceInfo &deviceInfo);

// 启用游戏按键映射
int32_t EnableGameKeyMapping(const GameInfo &gameInfo, const bool isEnable);
```

### 5. 设备识别

**能力**: 识别外设类别（键盘、手柄、鼠标等）

**证据**: `README_zh.md:64-65` - 对游戏外设进行设备类别识别，判断设备是键盘、还是游戏手柄等

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| OpenHarmony 版本 | API Level 21+ |
| 子系统 | game, input, window_manager |
| 系统能力 | SystemCapability.Game.GameController |
| 编译环境 | C++ 11+ |

### 进程模型

**证据**: `README_zh.md:72-77` - GameControllerSA 是独立进程，不是常驻进程

| 进程 | 说明 | 启动方式 |
|------|------|---------|
| **应用进程** | 加载 libohgame_controller.z.so 和 libgamecontroller_event.z.so | 应用启动时 dlopen 加载 |
| **GameControllerSA** | 运行 libgamecontroller_server.z.so | 按需启动（外设连接或配置变更）|

### SA 启动场景

**证据**: `README_zh.md:75-77`

1. **场景一**: 游戏外设连接上线时，通过 samgr 拉起 SA 进行设备类型识别
2. **场景二**: 终端厂商游戏服务通过 InnerAPI 配置时，通过 samgr 拉起 SA

### 加载机制

**证据**: `README_zh.md:81-84` - 应用启动时，Window Framework 通过 dlopen 加载 libgamecontroller_event.z.so

## 关键概念

### Input-to-Touch（输入转触控）

**定义**: 将游戏外设（键盘、鼠标、手柄）的输入事件转换为屏幕触控事件

**价值**: 游戏开发者无需适配即可支持键盘/鼠标操作

**流程**:
```
键盘/鼠标输入 → KeyMapping 处理 → 配置文件查询 → 触控事件生成 → Input 模块分发
```

### System Ability（SA）

**SA ID**: 8450

**证据**:
- `sa_profile/8450.json:5`: `"name": 8450`
- `frameworks/native/common/include/gamecontroller_constants.h`: 定义 GAME_CONTROLLER_SA_ID = 8450

**特性**:
- 按需启动
- 分布式关闭
- 低内存回收策略
- Dump level 1

### Device Monitor（设备监听）

**定义**: 监听游戏外设的连接和断开事件

**实现**:
- 对接 MultiModalInput 服务
- 事件回调通知给应用

**证据**: `README_zh.md:44-49`

### Input Monitor（输入监听）

**定义**: 监听游戏外设的输入事件

**实现**:
- 对接 Window Framework
- 注册需要拦截的输入事件
- 事件回调或转触控处理

**证据**: `README_zh.md:38-43`

## 关键结论

1. **项目定位**: OpenHarmony 游戏子系统的外设管理框架
2. **目标用户**: 游戏开发者 + 终端厂商
3. **核心价值**: 输入转触控特性，无需游戏厂商适配
4. **技术栈**: C++ 11+, CAPI, OpenHarmony IPC, System Ability
5. **运行模型**: 应用进程 + 按需 SA 进程
6. **SA ID**: 8450
7. **当前接口**: 仅 CAPI，ArkTS 未来规划

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构详解
- [03_Architecture.md](./03_Architecture.md) - 完整架构图

---

**版本**: 1.0 | **更新时间**: 2026-02-06

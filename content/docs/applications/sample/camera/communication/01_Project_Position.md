# 项目定位与边界

> 明确项目的职责边界、核心能力和运行环境

---

## 目的

本文档说明 `@ohos/camera_sample_communication` 项目的定位、职责边界、核心能力和运行环境，帮助读者理解项目的设计目标和限制。

## 适用范围

本文档适用于：
- 需要理解项目边界的开发者
- 评估项目适用性的架构师
- 基于 sample 进行二次开发的工程师

## 关键结论

- **项目性质**：OpenHarmony WiFi 通信子系统示例代码，非生产级实现
- **核心能力**：展示 wpa_supplicant 在 OpenHarmony 上的集成和使用方法
- **边界**：不包含 N-API、IPC、权限管理等高级功能
- **目标系统**：OpenHarmony mini / small 系统

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [架构说明](03_Architecture.md) - 组件交互和数据流

---

## 项目定位

### 核心定位

`@ohos/camera_sample_communication` 是一个 **示例代码项目**，专注于展示如何在 OpenHarmony 平台上集成和使用 wpa_supplicant 库实现 WiFi 功能。

### 设计目标

1. **教学导向**：为开发者提供清晰、简洁的 WiFi 功能集成示例
2. **易于理解**：代码结构简单，便于学习和修改
3. **多模式覆盖**：同时展示接入点（AP）和客户端（STA）两种模式

### 项目类型

| 特性 | 说明 |
|------|------|
| 项目性质 | 示例代码（Sample） |
| 代码复杂度 | 简单 |
| 目标读者 | OpenHarmony 开发者 |
| 生产就绪 | ❌ 否 |

---

## 核心能力

### 支持的 WiFi 模式

| 模式 | 组件 | 功能 | 入口文件 |
|------|------|------|---------|
| 接入点（AP） | hostapd | 创建 WiFi 热点 | `hostapd/src/hostapd_sample.c` |
| 客户端（STA） | wpa_supplicant | 连接 WiFi 网络 | `wpa_supplicant/src/wpa_sample.c` |
| 控制接口 | wpa_cli | 控制 wpa_supplicant | `wpa_cli/src/wpa_cli_sample.c` |

### 核心功能

#### hostapd（接入点模式）

- 启动 WiFi 接入点
- 支持 802.11g 模式
- 动态加载 libwpa.so 的 `ap_main` 函数

**证据**: `hostapd/src/hostapd_sample.c:26-54`

#### wpa_supplicant（客户端模式）

- 连接 WiFi 网络
- 支持配置文件加载
- 动态加载 libwpa.so 的 `wpa_main` 函数

**证据**: `wpa_supplicant/src/wpa_sample.c:26-54`

#### wpa_cli（控制客户端）

- 通过 wpa_ctrl 接口控制 wpa_supplicant
- 扫描 WiFi 网络
- 配置和连接网络
- 监听 WiFi 事件

**证据**: `wpa_cli/src/wpa_cli_sample.c:230-255`

---

## 职责边界

### 包含的功能 ✅

| 功能 | 说明 |
|------|------|
| WiFi AP 模式启动 | ✅ hostapd 组件 |
| WiFi STA 模式启动 | ✅ wpa_supplicant 组件 |
| WiFi 网络扫描 | ✅ wpa_cli 组件 |
| WiFi 网络配置 | ✅ wpa_cli 组件 |
| WiFi 事件监听 | ✅ wpa_cli 组件 |
| wpa_ctrl 接口使用 | ✅ wpa_cli 组件 |

### 不包含的功能 ❌

| 功能 | 说明 |
|------|------|
| N-API / JavaScript API | ❌ 本项目不提供 JS 绑定 |
| IPC / ServiceAbility | ❌ 不使用跨进程通信机制 |
| 权限管理 | ❌ 无权限检查逻辑 |
| 输入验证 | ❌ 无参数校验机制 |
| 错误恢复 | ❌ 无错误恢复逻辑 |
| 生产级错误处理 | ❌ 示例代码，错误处理简单 |
| 安全验证 | ❌ 无安全验证逻辑 |

### 接口边界

#### 外部接口

- **配置文件**: JSON 格式配置文件（hostapd.conf, wpa_supplicant.conf）
- **命令行参数**: 通过 main 函数传递给 wpa_main/ap_main
- **wpa_ctrl 接口**: UDP socket 控制接口

#### 内部接口

- **动态加载接口**: dlopen/dlsym 加载 libwpa.so
- **wpa_ctrl API**: 使用 wpa_supplicant 提供的控制接口

---

## 运行环境

### 支持的系统

| 系统 | 支持 | 说明 |
|------|------|------|
| OpenHarmony mini | ✅ | 轻量级系统 |
| OpenHarmony small | ✅ | 小型系统 |
| OpenHarmony standard | ⚠️ | 未测试，可能兼容 |
| Linux | ❌ | 不直接支持（依赖 OpenHarmony SDK） |

### 硬件要求

| 组件 | 硬件要求 |
|------|---------|
| WiFi 芯片 | 支持 OpenHarmony HDF WiFi 驱动 |
| RAM | 最小配置（具体值参考 OpenHarmony mini/small 系统要求） |
| 存储 | 用于存储配置文件和可执行文件 |

### 软件依赖

| 依赖 | 版本 | 说明 |
|------|------|------|
| wpa_supplicant | 2.9 | WiFi 核心库 |
| libsec_shared | - | 安全函数库（securec） |
| OpenHarmony SDK | - | 系统开发套件 |

---

## 关键概念

### 动态加载（dlopen）

项目使用动态加载机制加载 libwpa.so：

```c
void *handleLibWpa = dlopen("/usr/lib/libwpa.so", RTLD_NOW | RTLD_LOCAL);
int (*func)(int, char **) = dlsym(handleLibWpa, "ap_main");  // 或 "wpa_main"
```

**证据**:
- `hostapd/src/hostapd_sample.c:30, 36`
- `wpa_supplicant/src/wpa_sample.c:30, 36`

### wpa_ctrl 控制接口

wpa_supplicant 提供控制接口用于发送命令和接收事件：

| 操作 | 函数 | 说明 |
|------|------|------|
| 打开接口 | `wpa_ctrl_open()` | 创建控制连接 |
| 附加监控 | `wpa_ctrl_attach()` | 启用事件监听 |
| 发送命令 | `wpa_ctrl_request()` | 发送控制命令 |
| 接收事件 | `wpa_ctrl_recv()` | 接收事件消息 |
| 检查消息 | `wpa_ctrl_pending()` | 检查待处理消息 |

**证据**: `wpa_cli/src/wpa_cli_sample.c:232, 238, 130, 96, 93`

### WiFi 事件

wpa_ctrl 接口可以监听以下 WiFi 事件：

| 事件 | 宏定义 | 说明 |
|------|--------|------|
| 连接成功 | `WPA_EVENT_CONNECTED` | WiFi 连接建立 |
| 扫描完成 | `WPA_EVENT_SCAN_RESULTS` | 网络扫描结束 |
| 认证失败 | `WPA_EVENT_TEMP_DISABLED` | 密码错误等认证失败 |
| 断开连接 | `WPA_EVENT_DISCONNECTED` | WiFi 连接断开 |

**证据**: `wpa_cli/src/wpa_cli_sample.c:72, 76, 81, 85`

---

## 使用场景

### 适用场景 ✅

| 场景 | 说明 |
|------|------|
| 学习 WiFi 集成 | 学习如何在 OpenHarmony 上集成 wpa_supplicant |
| 参考实现 | 参考 wpa_ctrl 接口的使用方法 |
| 功能验证 | 验证 OpenHarmony WiFi 功能 |
| 原型开发 | 基于 sample 进行原型开发 |

### 不适用场景 ❌

| 场景 | 说明 |
|------|------|
| 生产环境 | 本项目是示例代码，不适用于生产环境 |
| 高安全要求 | 无权限管理和安全验证 |
| 复杂业务逻辑 | 需要自己扩展 |
| JS 应用开发 | 无 N-API 绑定 |

---

## 限制与假设

### 已知限制

1. **无权限检查**：任何进程都可以调用 wpa_ctrl 接口
2. **无输入验证**：命令行参数直接传递给 wpa_main/ap_main
3. **无错误恢复**：错误发生后程序直接退出
4. **配置简单**：配置文件非常基础，不包含高级功能

### 假设前提

1. **libwpa.so 已存在**：假设系统中已安装 wpa_supplicant 库
2. **WiFi 驱动正常**：假设底层 WiFi 驱动工作正常
3. **配置文件路径固定**：配置文件安装在 `/etc/` 目录

---

## 版本信息

| 项目 | 值 |
|------|-----|
| 项目版本 | 3.1 |
| OpenHarmony 目标 | mini, small |
| wpa_supplicant 版本 | 2.9 |

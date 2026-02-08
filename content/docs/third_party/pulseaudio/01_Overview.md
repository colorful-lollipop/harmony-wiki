# 01 - PulseAudio 概览与 OHOS 定位

## 1.1 原始库简介

### 基本信息
| 属性 | 值 |
|-----|-----|
| 库名称 | PulseAudio |
| 上游版本 | 17.0 |
| 许可证 | LGPL-2.1 |
| 上游地址 | https://gitlab.freedesktop.org/pulseaudio/pulseaudio/-/tree/v17.0 |

### 功能描述
PulseAudio 是一个**声音服务器系统**，主要功能包括：
- **音频播放管理**：管理音频流的混合、路由和输出
- **录音管理**：处理音频输入流的捕获和处理
- **音频策略控制**：音量管理、设备切换、音频效果处理
- **网络音频支持**：支持音频流的网络传输
- **低延迟处理**：提供专业的音频延迟控制

### 典型应用场景
- Linux 桌面系统的音频服务器
- 嵌入式设备的音频管理
- VoIP 和会议系统的音频处理
- 专业音频工作站

---

## 1.2 在 OpenHarmony 中的作用

### 定位
PulseAudio 在 OpenHarmony 中作为**多媒体音频子系统的核心音频服务器**，提供基础音频服务能力。

### 角色关系
```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Applications)                     │
│         (媒体播放器、录音机、VoIP 应用等)                     │
└───────────────────────┬─────────────────────────────────────┘
                        │ 音频 API
┌───────────────────────▼─────────────────────────────────────┐
│               Audio Framework (音频框架)                     │
│    (foundation/multimedia/audio_framework)                   │
│         - 音频策略管理                                       │
│         - 音频路由控制                                       │
│         - 音效处理                                          │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              Audio Service (音频服务)                        │
│    (foundation/multimedia/audio_framework/services/          │
│                  audio_service)                              │
│         - 依赖于 pulseaudio:sonic (音频变速)                 │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              PulseAudio (音频服务器)                         │
│         - 音频流管理                                        │
│         - 设备抽象层                                        │
│         - 客户端-服务器通信                                  │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                   硬件抽象层 (HAL)                           │
│              (音频驱动、蓝牙音频等)                           │
└─────────────────────────────────────────────────────────────┘
```

### OHOS 版本信息
| 属性 | 值 |
|-----|-----|
| OHOS Bundle 版本 | 3.1 |
| 组件名称 | @ohos/pulseaudio |
| 所属子系统 | thirdparty |
| 适配系统类型 | standard (标准系统) |

---

## 1.3 OHOS 集成的独特性

### 与传统 Linux 集成的差异

| 方面 | 传统 Linux | OpenHarmony |
|-----|-----------|-------------|
| 启动方式 | systemd/自启动 | init 进程管理 |
| Socket 创建 | 自主创建 | init 预创建，通过 GetControlSocket 获取 |
| 日志系统 | syslog/journald | HiLog (hilog:libhilog) |
| 模块加载 | 动态加载 (.so) | 静态链接/预加载 |
| 权限模型 | Unix 用户/组 | 沙箱 + 权限令牌 |

### 无 Patch 文件的适配策略

**独特之处**：PulseAudio 是 OpenHarmony 中**没有传统 `.patch` 文件**的第三方库。

**OHOS 采用的方式**：
1. **文件替换**：提供带 `ohos_` 前缀的替代实现
   - `main.c` → `ohos_pa_main.c`
   - `daemon-conf.c` → `ohos_daemon-conf.c`
   - `socket-server.c` → `ohos_socket-server.c`

2. **条件编译**：使用 `HAVE_NO_OHOS` 宏控制功能裁剪
   - 禁用原生动态模块加载
   - 禁用某些系统调用
   - 调整权限设置

3. **新增组件**：
   - `audio_log.h` - HiLog 集成
   - Sonic 变速库集成

### 关键适配点总结

| 适配点 | 说明 | 影响 |
|-------|------|------|
| Socket 机制 | 使用 init 预创建 socket | 必须与 init 配合启动 |
| 日志系统 | 替换为 HiLog | 日志查看方式变化 |
| 主入口 | 自定义 ohos_pa_main | 初始化流程调整 |
| 协议扩展 | 新增 UNDERFLOW_OHOS 命令 | 客户端需适配 |

---

## 1.4 版本与上游同步

### 当前版本
- **上游版本**：17.0 (2024年发布)
- **OHOS Bundle 版本**：3.1

### 上游主要变更 (17.0)
根据 NEWS 文件，17.0 版本的主要改进：
- ALSA UCM 配置更新
- 蓝牙设备电池电量指示
- Bluetooth FastStream 编解码器支持
- webrtc-audio-processing 依赖更新
- 模块角色组触发器增强

### OHOS 维护注意事项
1. **升级风险**：由于采用文件替换式适配，上游升级时需重新对比适配文件
2. **协议兼容**：新增的 `PA_COMMAND_UNDERFLOW_OHOS` 需保持兼容
3. **HiLog 集成**：升级时需确保日志宏定义保持有效
4. **Init 集成**：socket 机制依赖 init，升级时不得破坏

---

## 1.5 参考资料

### 上游文档
- PulseAudio 官方文档：https://www.freedesktop.org/wiki/Software/PulseAudio/
- 上游 CHANGELOG：`/NEWS`

### OHOS 相关文档
- 组件配置文件：`/bundle.json`
- 开源说明：`/README.OpenSource`
- 构建配置：`/ohosbuild/` 目录下的 BUILD.gn 文件

### 关键源码文件
- OHOS 主入口：`/src/daemon/ohos_pa_main.c`
- OHOS Socket 服务器：`/src/pulsecore/ohos_socket-server.c`
- OHOS 日志头：`/include/log/audio_log.h`
- 协议定义：`/src/pulsecore/native-common.h`

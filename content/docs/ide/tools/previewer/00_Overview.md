# 项目概览

## 项目定位

**OpenHarmony Previewer** 是 OpenHarmony IDE 子系统的核心组件，为 DevEco Studio 提供 ArkUI 页面实时预览能力。

| 维度 | 描述 |
|------|------|
| **产品定位** | IDE 工具组件，非应用运行时 |
| **核心功能** | 接收 IDE 命令，调用渲染引擎，输出预览图像 |
| **用户群体** | OpenHarmony 应用开发者 |
| **部署方式** |随 OpenHarmony SDK 分发 |

## 核心能力

### 1. 实时页面预览
- 支持 ArkTS/JS 应用页面预览
- 双向交互：IDE 命令 ↔ 预览图像
- 支持页面刷新、组件树检查

### 2. 多设备模拟
| 版本 | 渲染引擎 | 目标设备 | 渲染方式 |
|------|----------|----------|----------|
| **Rich** | ArkUI Ace Engine | 标准设备 | OpenGL/Glfw |
| **Lite** | ACELite | 轻量设备 | 软件渲染 |

### 3. 多平台支持
- ✅ Windows (mingw_x86_64)
- ✅ macOS (arm64/x64)
- ✅ Linux (x64/arm64)

### 4. 输入模拟
- 鼠标/触摸事件注入
- 键盘输入
- 滚轮/表冠滚动
- 物理传感器模拟（Lite）

## 运行环境

### 系统依赖
```
arkui_ace_engine      - ArkUI 引擎
window_manager         - 窗口管理
ability_runtime       - 能力运行时
graphic_2d            - 2D 图形
zlib                  - 压缩库
```

### 第三方依赖
```
bounds_checking_function - 边界检查
libjpeg-turbo           - JPEG 编码
libwebsockets            - WebSocket 通信
cJSON                   - JSON 解析
```

### 运行时要求
- **ROM**: 25,600 KB
- **RAM**: 102,400 KB

## 关键概念

### Rich vs Lite
- **Rich**: 完整 ArkUI 引擎，支持 Stage 模型、HSP 模块、组件检查器
- Lite: 轻量引擎，支持手表等轻量设备，API 子集

### 虚拟屏幕 (VirtualScreen)
- 渲染目标缓冲区
- JPEG 压缩与质量控制
- 帧率统计与动态/静态模式切换

### 命名管道 (Named Pipe)
- 命令传输通道
- Windows: `\\.\pipe\` 命名空间
- Unix: Domain Socket

### WebSocket 图像流
- 渲染图像传输
- SID 认证机制
- 二进制 JPEG 数据

---

## 相关文档

- 目录结构: [01_Directory_Structure.md](./01_Directory_Structure.md)
- 架构设计: [02_Architecture.md](./02_Architecture.md)
- 通信协议: [03_Communication_Protocol.md](./03_Communication_Protocol.md)

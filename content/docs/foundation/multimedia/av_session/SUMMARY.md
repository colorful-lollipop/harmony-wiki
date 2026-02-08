# AVSession Wiki 导航

## 新人阅读路线

建议阅读顺序：`概览` → `架构` → `N-API` → `Inner API` → `构建` → `安全`

---

## 核心文档

### [README](README.md)
- 项目概述
- Wiki 覆盖范围
- 快速开始示例
- 相关链接

### [概览](00_Overview.md)
- 项目定位与边界
- 核心能力
- 运行环境
- 关键概念
- 目录结构与模块职责

### [架构](01_Architecture.md)
- 逻辑架构图
- 组件职责说明
- 数据流向
- 线程模型
- 关键时序图 (Mermaid)

### [N-API](02_NAPI.md)
- JS API 完整清单
- AVSessionManager API
- AVSession API
- AVSessionController API
- AVCastController API
- AVCastPickerHelper API
- 枚举常量

### [Inner API](03_InnerAPI.md)
- Native 接口头文件清单
- 核心类定义
- AVSession / Controller / Manager
- 数据结构 (AVPlaybackState, AVMetaData)
- IPC 接口定义
- 回调机制

### [构建](04_Build.md)
- GN 构建入口
- 主要 Targets 清单
- 依赖关系
- 编译产物
- 安装路径

### [安全](05_Security.md)
- 攻击面清单
- 信任边界与数据流
- 安全风险点分析
- 修复建议

---

## 附录

### 关键调用链
- [调用链入口](appendix/Callgraphs.md)

---

## 快速索引

### 按功能查找

| 功能 | 相关模块 | API |
|------|----------|-----|
| 创建会话 | AVSessionManager | `createAVSession()` |
| 获取状态 | AVSessionController | `getAVPlaybackState()` |
| 设置元数据 | AVSession | `setAVMetadata()` |
| 发送控制 | AVSessionController | `sendControlCommand()` |
| 投屏控制 | AVCastController | `start()`, `prepare()` |
| 设备发现 | AVSessionManager | `startCastDeviceDiscovery()` |
| 事件监听 | AVSession/Controller | `on()`, `off()` |

### 按代码位置查找

| 功能 | 代码路径 |
|------|----------|
| N-API 实现 | `frameworks/js/napi/session/` |
| Native 框架 | `frameworks/native/session/` |
| 服务端实现 | `services/session/server/` |
| IPC 通信 | `services/session/ipc/` |
| 接口定义 | `interfaces/inner_api/native/session/include/` |
| 工具类 | `utils/` |

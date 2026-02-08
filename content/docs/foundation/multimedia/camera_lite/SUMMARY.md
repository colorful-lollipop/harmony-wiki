# SUMMARY - Camera_Lite Wiki 导航

> 全站导航与新手指南

---

## 快速导航

### 核心文档

| 序号 | 文档 | 说明 | 阅读建议 |
|:----:|------|------|----------|
| 00 | [项目概览](00_Overview.md) | 组件定位、能力、运行环境 | ⭐ 必读 |
| 01 | [架构说明](01_Architecture.md) | 组件图、数据流、线程模型 | ⭐ 必读 |
| 02 | [API参考](02_API_Reference.md) | C++ API完整清单 | ⭐ 必读 |
| 03 | [内部接口](03_Inner_API.md) | 模块接口、依赖关系 | 进阶 |
| 04 | [构建系统](04_Build_System.md) | GN targets、编译产物 | 开发者 |
| 05 | [安全风险评审](05_Security.md) | 攻击面、风险点、修复建议 | 安全审计 |
| 06 | [常见问题](06_Troubleshooting.md) | 构建/运行/调试问题 | 问题排查 |

### 附录

| 文档 | 说明 |
|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链详细分析 |

---

## 按角色阅读

### 👤 应用开发者 (使用相机能力)

```
开始 → 00_Overview.md → 02_API_Reference.md → 06_Troubleshooting.md
```

**关注重点**:
- [CameraKit](02_API_Reference.md#camerakit) - 入口类
- [Camera](02_API_Reference.md#camera) - 相机操作
- [FrameConfig](02_API_Reference.md#frameconfig) - 帧配置
- [回调机制](02_API_Reference.md#回调接口) - 异步处理

### 🔧 系统开发者 (集成/移植相机)

```
开始 → 00_Overview.md → 01_Architecture.md → 04_Build_System.md → 03_Inner_API.md
```

**关注重点**:
- [运行模式](01_Architecture.md#运行模式) - Binder vs Passthrough
- [GN Targets](04_Build_System.md#gn-targets) - 编译配置
- [HAL接口](03_Inner_API.md#hal层接口) - 硬件抽象层

### 🔒 安全审计员

```
开始 → 01_Architecture.md → 05_Security.md → 03_Inner_API.md
```

**关注重点**:
- [攻击面分析](05_Security.md#攻击面清单)
- [信任边界](05_Security.md#信任边界)
- [风险点详情](05_Security.md#风险点详细分析)

### 📚 架构师

```
开始 → 00_Overview.md → 01_Architecture.md → 03_Inner_API.md → appendix/Callgraphs.md
```

**关注重点**:
- [系统架构](01_Architecture.md#系统架构图)
- [数据流](01_Architecture.md#数据流图)
- [模块职责](03_Inner_API.md#模块职责)

---

## 关键概念速查

| 概念 | 说明 | 文档 |
|------|------|------|
| CameraKit | 相机套件单例，应用入口 | [API参考](02_API_Reference.md#camerakit) |
| Camera | 相机实例，操作接口 | [API参考](02_API_Reference.md#camera) |
| FrameConfig | 帧配置，定义输出格式 | [API参考](02_API_Reference.md#frameconfig) |
| Surface | 共享内存，数据输出 | [API参考](02_API_Reference.md#surface) |
| Binder模式 | IPC通信模式 | [架构](01_Architecture.md#binder模式) |
| Passthrough模式 | 直通模式，无IPC | [架构](01_Architecture.md#passthrough模式) |
| HAL层 | 硬件抽象层接口 | [内部接口](03_Inner_API.md#hal层) |

---

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 预览 | Preview | 实时取景显示 |
| 录像 | Record | 视频录制 |
| 拍照 | Capture | 单帧图像捕获 |
| 回调 | Callback | 原始帧数据回调 |
| 能力 | Ability | 相机支持的参数范围 |
| 配置 | Config | 相机/帧参数配置 |
| 流 | Stream | 数据流，如预览流、录像流 |

---

## 相关资源

### 官方文档
- [OpenHarmony文档中心](https://gitee.com/openharmony/docs)
- [媒体子系统说明](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/媒体子系统.md)

### 相关仓库
- [camera_sample_lite](https://gitee.com/openharmony/applications_sample_camera) - 示例应用
- [media_utils_lite](https://gitee.com/openharmony/multimedia_utils_lite) - 媒体工具
- [surface_lite](https://gitee.com/openharmony/graphic_surface_lite) - 图形Surface

---

*本文档由工程Agent自动生成，如有疑问请参考源代码或联系维护者。*

# OpenHarmony Screensaver 屏保应用

## 项目简介

本项目是 OpenHarmony 图形子系统的屏保功能参考实现，提供完整的屏保应用示例代码。

**核心能力**：
- 循环播放预设的屏保图片
- 点击屏幕任意位置退出屏保
- 自动适配屏幕尺寸

## 快速开始

### 环境要求

- OpenHarmony Mini/Small 系统
- C++11 或更高版本
- 支持的设备类型：phone, tv, tablet, car, smartWatch, sportsWatch, smartVision

### 构建命令

```bash
# 完整构建
hb build -f

# 或使用 GN 直接构建
python build.py -p ipcamera_hi3516dv300 -b release
```

### 运行方式

1. 构建生成 `screensaver.hap` 应用包
2. 通过 `hdc` 工具安装到设备
3. 屏保作为系统服务自动触发

## 项目定位

```
用户态应用
    │
    ├── 依赖 Ability Lite 框架（生命周期管理）
    ├── 依赖 UI Lite 框架（界面渲染）
    └── 依赖 Surface Lite（图形表面管理）
```

## 关键特性

| 特性 | 实现方式 |
|------|----------|
| 图片动画 | UIImageAnimatorView |
| 事件处理 | EventListener (点击/长按) |
| 资源释放 | 析构函数中完整清理 |
| 屏幕适配 | 动态获取屏幕尺寸 |

## 文档导航

- [详细概览](01_Overview.md) - 项目定位与运行环境
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [架构设计](03_Architecture.md) - 组件与数据流
- [内部 API](05_Inner_API.md) - 模块接口说明
- [构建系统](06_Build.md) - 编译流程与产物
- [安全评审](07_Security.md) - 风险分析与修复建议

---

*文档版本: 1.0 | 更新日期: 2026-02-05*

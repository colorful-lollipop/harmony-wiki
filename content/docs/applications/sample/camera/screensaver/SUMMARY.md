# 文档导航

本文档为 OpenHarmony Screensaver 项目的完整工程 Wiki。

## 新人阅读顺序（推荐）

```
1. 概览 → 了解项目定位与核心能力
      ↓
2. 目录结构 → 熟悉代码组织方式
      ↓
3. 架构设计 → 理解组件关系与数据流
      ↓
4. 内部 API → 掌握模块接口与依赖
      ↓
5. 构建系统 → 了解编译流程与产物
      ↓
6. 安全评审 → 识别潜在风险
      ↓
7. 故障排查 → 快速定位问题
```

## 完整目录

### 核心文档

| 文档 | 说明 |
|------|------|
| [README](README.md) | 文档说明、更新方式、导航入口 |
| [首页/概览](index.md) | 项目快速入门 |
| [01_概览](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [02_目录结构](02_Directory_Structure.md) | 目录组织、模块职责（不含测试） |
| [03_架构设计](03_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [04_N-API](04_N-API.md) | **本项目不涉及 N-API**（原生 C++ 应用） |
| [05_内部 API](05_Inner_API.md) | 模块接口、依赖方向、稳定性标注 |
| [06_构建系统](06_Build.md) | GN Targets、编译产物、安装路径 |
| [07_安全评审](07_Security.md) | 攻击面、信任边界、风险点与修复建议 |
| [08_故障排查](08_Troubleshooting.md) | 常见构建/运行/调试问题与定位 |

### 附录

| 文档 | 说明 |
|------|------|
| [附录_调用链](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [附录_配置项](appendix/Config_Flags.md) | 关键宏与 Feature Flags |

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Graphic 子系统文档](https://gitee.com/openharmony/docs/blob/master/en/readme/graphics.md)
- [Window Manager Lite](https://gitee.com/openharmony/window_window_manager_lite/blob/master/README.md)
- [UI Lite](https://gitee.com/openharmony/arkui_ui_lite/blob/master/README.md)

# SUMMARY - 全站导航

## 新人阅读路线

建议按以下顺序阅读，以快速理解项目全貌：

1. **[README](./README.md)** - 了解 Wiki 范围和更新方式
2. **[项目概览](./index.md)** - 理解项目定位、边界和核心能力
3. **[目录结构](./Structure.md)** - 熟悉代码组织方式
4. **[架构说明](./Architecture.md)** - 理解系统架构、数据流和模块关系
5. **[GN 构建系统](./Build_System.md)** - 了解如何构建项目
6. **[安全风险分析](./Security.md)** - 理解安全注意事项（关键！）

---

## 完整导航

### 入门
- [README - 关于本文档](./README.md)
- [Index - 项目概览](./index.md)
  - 项目定位与边界
  - 核心能力
  - 运行环境
  - 关键概念

### 代码结构
- [目录结构与模块职责](./Structure.md)
  - 顶层目录
  - 各模块详细结构
  - 排除测试代码后的有效代码量

### 架构设计
- [架构说明](./Architecture.md)
  - 组件图 (Mermaid)
  - 数据流
  - 线程模型
  - 关键时序

### 接口文档
- [对外接口 (External API)](./External_API.md)
  - 重要发现：无 N-API 接口
  - Ability 生命周期
  - 权限声明
- [内部接口 (Internal API)](./Internal_API.md)
  - 模块间接口
  - 依赖方向
  - 稳定性标注

### 构建系统
- [GN 构建系统](./Build_System.md)
  - BUILD.gn 分析
  - Targets 清单
  - 依赖关系
- [编译产物](./Build_Outputs.md)
  - 产物类型 (.so/.hap/可执行文件)
  - 安装路径
  - 运行时加载关系

### 安全
- [安全风险分析](./Security.md)
  - 攻击面清单
  - 信任边界
  - 可被利用点（含修复建议）

### 调试与问题定位
- [常见问题 (FAQ)](./FAQ.md)
  - 构建问题
  - 运行问题
  - 调试方法

### 附录
- [关键调用链](./appendix/Callgraphs.md)
- [配置开关](./appendix/Config_Flags.md)

### 模块详细文档
- [cameraApp - 相机应用](./modules/cameraApp.md)
- [gallery - 图库应用](./modules/gallery.md)
- [launcher - 桌面应用](./modules/launcher.md)
- [setting - 设置应用](./modules/setting.md)
- [media - 媒体示例](./modules/media.md)

---

## 索引

### 按主题
| 主题 | 相关页面 |
|------|----------|
| 架构设计 | [Architecture](./Architecture.md), [Structure](./Structure.md) |
| 接口文档 | [External_API](./External_API.md), [Internal_API](./Internal_API.md) |
| 构建系统 | [Build_System](./Build_System.md), [Build_Outputs](./Build_Outputs.md) |
| 安全审计 | [Security](./Security.md) |
| 问题排查 | [FAQ](./FAQ.md) |

### 按模块
| 模块 | 文档 | BUILD.gn |
|------|------|----------|
| cameraApp | [modules/cameraApp](./modules/cameraApp.md) | `cameraApp/BUILD.gn` |
| gallery | [modules/gallery](./modules/gallery.md) | `gallery/BUILD.gn` |
| launcher | [modules/launcher](./modules/launcher.md) | `launcher/BUILD.gn` |
| setting | [modules/setting](./modules/setting.md) | `setting/BUILD.gn` |
| media | [modules/media](./modules/media.md) | `media/BUILD.gn` |

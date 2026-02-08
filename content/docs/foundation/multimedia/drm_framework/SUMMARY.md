# SUMMARY

> OpenHarmony DRM 框架 Wiki - 双路线导航
> 
> 本文档提供两种阅读路径：**新人学习路线** 和 **安全研究路线**

---

## 新人学习路线

如果你是刚接触 DRM 框架的开发者，建议按以下顺序阅读：

1. **[项目概览](01_Overview.md)** - 5分钟了解 DRM 框架是什么、能做什么
2. **[架构与数据流](02_Architecture.md)** - 理解分层架构和组件交互
3. **[目录结构与代码地图](03_CodeMap.md)** - 快速定位核心代码位置
4. **[对外接口文档](04_Interface.md)** - JS/C API 详细参考
5. **[构建与产物](07_Build.md)** - 编译配置和输出产物说明

**快速开始**: [01_Overview.md#快速开始](01_Overview.md#快速开始)

---

## 安全研究路线

如果你是安全研究员，建议按以下顺序阅读：

1. **[项目概览](01_Overview.md)** - 了解项目定位和能力边界
2. **[攻击面分析](05_AttackSurface.md)** - 识别所有外部输入入口和敏感操作
3. **[安全风险评估](06_SecurityReview.md)** - 深度分析潜在漏洞点和利用路径
4. **[架构与数据流](02_Architecture.md)** - 理解信任边界和数据流向

**关键入口**: [05_AttackSurface.md#外部输入清单](05_AttackSurface.md#外部输入清单)

---

## 完整文档索引

### 核心文档

| 文档 | 目标读者 | 主要内容 |
|------|----------|----------|
| [01_Overview.md](01_Overview.md) | 所有读者 | 项目定位、核心功能、快速开始 |
| [02_Architecture.md](02_Architecture.md) | 开发者 | 分层架构、组件图、数据流、时序 |
| [03_CodeMap.md](03_CodeMap.md) | 开发者 | 目录结构、文件职责、代码导航 |
| [04_Interface.md](04_Interface.md) | 开发者 | JS API、C API、IPC 接口清单 |
| [05_AttackSurface.md](05_AttackSurface.md) | 安全研究员 | 攻击面识别、信任边界 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全研究员 | 风险评估、漏洞分析、修复建议 |
| [07_Build.md](07_Build.md) | 开发者 | GN Targets、编译产物、Feature 开关 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链分析 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 配置开关说明 |

### 工作文件

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

## 术语表

| 术语 | 说明 |
|------|------|
| DRM | Digital Rights Management，数字版权管理 |
| MediaKeySystem | 媒体密钥系统，管理 DRM 方案的生命周期 |
| MediaKeySession | 媒体密钥会话，管理许可证和密钥 |
| Provision | 证书配置，设备与 DRM 服务商建立信任关系 |
| License | 许可证，包含解密密钥的授权文件 |
| HDI | Hardware Device Interface，硬件设备接口 |
| SA | System Ability，系统服务 |
| N-API | Native API，JS 与 C++ 的桥接接口 |
| C-API | C 语言接口，NDK 导出函数 |
| SVP | Secure Video Path，安全视频通路 |
| ContentProtectionLevel | 内容保护级别（软件加密/硬件加密） |

---

## 外部链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [DRM 框架源码](https://gitee.com/openharmony/multimedia_drm_framework)
- [API 参考 - multimedia.drm](https://docs.openharmony.cn/pages/v5.0/zh-cn/application-dev/reference/apis-media-kit/)

---

*最后更新: 2025-02-07*

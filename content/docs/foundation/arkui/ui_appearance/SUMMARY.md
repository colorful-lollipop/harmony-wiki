# 文档导航

> OpenHarmony UI Appearance Wiki 全站导航

## 新人阅读路线

```
建议阅读顺序:
1. README.md (本文档说明)
2. 01_Overview.md (项目概述)
3. 02_Architecture.md (架构设计)
4. 03_NAPI.md (接口文档 - 按需查阅)
5. 04_InnerAPI.md (内部实现 - 按需查阅)
6. 05_Build.md (构建配置 - 按需查阅)
7. 06_Security.md (安全评审 - 按需查阅)
```

## 文档目录

### 快速入门

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README](README.md) | 文档说明、覆盖范围、更新方式 | ⭐⭐⭐ |
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力 | ⭐⭐⭐ |
| [02_Architecture](02_Architecture.md) | 架构图、数据流、线程模型 | ⭐⭐⭐ |

### API 参考

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [03_NAPI](03_NAPI.md) | N-API 接口、参数、错误码 | ⭐⭐⭐ |
| [04_InnerAPI](04_InnerAPI.md) | 内部模块、依赖、生命周期 | ⭐⭐ |

### 构建与部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力 | ⭐⭐⭐ |
| [02_Architecture](02_Architecture.md) | 架构图、数据流、线程模型 | ⭐⭐⭐ |
 
### API 参考
 
| 文档 | 描述 | 优先级 |
|------|------|--------|
| [03_NAPI](03_NAPI.md) | N-API 接口、参数、错误码 | ⭐⭐⭐ |
| [04_InnerAPI](04_InnerAPI.md) | 内部模块、依赖、生命周期 | ⭐⭐ |
 
### 代码导航
 
| 文档 | 描述 | 优先级 |
|------|------|--------|
| [03_CodeMap](03_CodeMap.md) | 目录结构、核心文件定位、功能导航 | ⭐⭐⭐ |
| [05_AttackSurface](05_AttackSurface.md) | 攻击面分析、外部输入、敏感操作、信任边界 | ⭐⭐⭐ |
 
### 构建与部署
 
| 文档 | 描述 | 优先级 |
|------|------|--------|
| [05_Build](05_Build.md) | GN Targets、编译产物、安装路径 | ⭐⭐ |
| [06_Security](06_Security.md) | 安全风险、攻击面、修复建议 | ⭐⭐ |

## 快速索引

### 按功能查找

| 功能 | 文档 | 章节 |
|------|------|------|
| 深色模式设置 | 03_NAPI | setDarkMode |
| 深色模式获取 | 03_NAPI | getDarkMode |
| 字体缩放设置 | 03_NAPI | setFontScale |
| 权限要求 | 03_NAPI | 权限说明 |
| SA 注册 | 04_InnerAPI | UiAppearanceAbility |
| 编译命令 | 05_Build | 编译说明 |
| 安全风险 | 06_Security | 风险列表 |

### 按文件查找

| 文件 | 描述 | 位置 |
|------|------|------|
| js_ui_appearance.cpp | N-API 实现 | `interfaces/kits/napi/src/` |
| ui_appearance_ability.cpp | SA 主逻辑 | `services/src/` |
| dark_mode_manager.cpp | 深色模式管理 | `services/src/` |
| BUILD.gn | 构建配置 | `services/` |

### 按关键词查找

| 关键词 | 相关文档 | 说明 |
|--------|----------|------|
| NAPI_MODULE | 03_NAPI | N-API 注册入口 |
| SA ID 7002 | 02_Architecture, 04_InnerAPI | 系统能力标识 |
| UPDATE_CONFIGURATION | 03_NAPI, 06_Security | 权限名 |
| DarkMode | 03_NAPI | 深色模式枚举 |
| AsyncCallback | 03_NAPI | 异步回调模式 |

## 术语表

| 术语 | 全称 | 描述 |
|------|------|------|
| SA | System Ability | 系统能力 |
| AMS | Ability Manager Service | 能力管理服务 |
| WMS | Window Manager Service | 窗口管理服务 |
| N-API | Node.js API | OpenHarmony JS API |
| IPC | Inter-Process Communication | 进程间通信 |

## 相关资源

### 内部链接

- [架构图](figures/uiAppearance-architecture_zh.png)
- [代码证据索引](_work/NOTES.md)

### 外部链接

- [OpenHarmony Docs](https://gitee.com/openharmony/docs)
- [UIAppearance API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-uiappearance.md)
- [ArkUI 框架](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/ArkUI%E6%A1%86%E6%9E%B6%E5%AD%90%E7%B3%BB%E7%BB%9F.md)

---

*本文档最后更新: 2025-02-06*

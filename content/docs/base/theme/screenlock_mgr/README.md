# ScreenLock Mgr - OpenHarmony 屏幕锁定管理器

## 项目概述

本项目是 OpenHarmony 系统的屏幕锁定管理器子系统 (`theme_screenlock_mgr`)，负责管理系统级屏幕锁定功能。

### 核心能力

- **面向应用**: 为第三方应用提供屏幕解锁/锁定状态查询、密码设置查询等能力
- **面向系统**: 向运营管理提供屏幕开关机回调、屏保进出回调、用户切换回调等事件通知

### 系统能力

```json
// bundle.json
"syscap": [
  "SystemCapability.MiscServices.ScreenLock"
]
```

### 仓库位置

```
/base/theme/screenlock_mgr
```

---

## 文档结构

### 快速导航

| 文档 | 说明 |
|------|------|
| [README](README.md) | 本文档 |
| [SUMMARY](SUMMARY.md) | 全站导航与阅读顺序 |
| [00_Overview](00_Overview.md) | 项目概览与核心概念 |
| [01_NAPI_Reference](01_NAPI_Reference.md) | N-API 接口参考 |
| [02_Architecture](02_Architecture.md) | 系统架构与组件关系 |
| [03_GN_Build](03_GN_Build.md) | 构建系统与编译产物 |
| [04_Security_Review](04_Security_Review.md) | 安全风险评审 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

---

## 快速开始

### 编译

```bash
./build.sh --product-name <产品名> --build-target screenlock_native
```

### 推送调试

```bash
# 从 out 目录推送 so 文件
# 推送 libscreenlock_server.z.so, libscreenlock_client.z.so, libscreenlock_utils.z.so 到 system/lib/
# 推送 libscreenlockability.z.so 到 system/lib/module/app/
```

---

## 版本信息

- **当前版本**: 3.1 (来自 bundle.json)
- **License**: Apache-2.0
- **适用系统**: standard (标准系统)

---

## 贡献指南

1. Fork 仓库
2. 创建特性分支
3. 提交代码
4. 创建 Pull Request

---

## 更新日志

| 日期 | 更新内容 |
|------|----------|
| 2026-02-06 | 初始化 Wiki 文档 |

---

## 文档覆盖范围

### 已覆盖

| 模块 | 状态 | 说明 |
|------|------|------|
| N-API 接口 | ✅ 完成 | 15 个 API 详细说明 |
| 系统架构 | ✅ 完成 | 组件图、数据流、线程模型 |
| 构建系统 | ✅ 完成 | GN targets、编译产物 |
| 安全评审 | ✅ 完成 | 7 个风险点分析 |
| 调用链 | ✅ 完成 | 关键流程图解 |

### 未覆盖

| 模块 | 原因 |
|------|------|
| ETS API 详情 | 需要更深入分析 ets/ani 目录 |
| 运行时配置 | screenlock.cfg 详细说明 |
| 测试代码 | 按规范不引用测试 |

---

## 文档更新方式

1. 代码变更后评估是否需要更新相关文档
2. API 变更 → 更新 01_NAPI_Reference.md
3. 架构变更 → 更新 02_Architecture.md
4. 构建变更 → 更新 03_GN_Build.md
5. 安全问题 → 更新 04_Security_Review.md
6. 调用链变更 → 更新 appendix/Callgraphs.md

# Audio Framework Wiki - README

## 概述

本文档是 OpenHarmony 多媒体音频框架 (audio_framework) 的工程 Wiki，旨在帮助开发者快速理解项目架构、N-API 接口、构建系统和安全机制。

## 覆盖范围

### ✅ 已覆盖
- [x] 项目定位与核心能力
- [x] 目录结构与模块职责
- [x] N-API 接口（JS API 清单）
- [x] IPC/System Ability 架构
- [x] GN 构建系统与 targets
- [x] 编译产物清单
- [x] 权限与安全机制

### ⏳ 补充中
- [ ] 架构图（Mermaid）
- [ ] 关键时序图
- [ ] 常见问题与调试指南

## 未覆盖范围

- ❌ 测试相关内容（test/ 目录）
- ❌ 特定设备驱动实现
- ❌ 第三方库内部实现

## 更新方式

当代码发生以下变更时，需同步更新 Wiki：

1. **新增/删除 N-API 方法** → 更新 `wiki/03_NAPI.md`
2. **新增/删除 GN targets** → 更新 `wiki/04_Build.md`
3. **修改 SA ID 或 IPC 接口** → 更新 `wiki/02_Architecture.md`
4. **新增权限或安全检查** → 更新 `wiki/05_Security.md`

## 生成信息

- **仓库**: `multimedia/audio_framework`
- **版本**: 4.0
- **生成时间**: 2025-02-06
- **生成方式**: 自动扫描代码仓库

## 相关链接

- [OpenHarmony Audio 官方文档](https://gitee.com/openharmony/docs)
- [Audio Framework 源码](https://gitee.com/openharmony/multimedia_audio_framework)
- [JS API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-audio-kit/js-apis-audio.md)

---

*本文档由 Wiki 生成工具自动生成*

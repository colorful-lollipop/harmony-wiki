# 文档导航

## 快速入门

- [首页](README.md)
- [项目概览](01_Overview.md)

## 架构设计

- [目录结构与模块职责](02_Directory_Structure.md)
- [系统架构](03_Architecture.md)
- [数据流与线程模型](04_DataFlow_Threading.md)

## API 接口

- [N-API 接口总览](05_NAPI_Overview.md)
- [播放器 API](06_AVPlayer.md)
- [录制器 API](07_AVRecorder.md)
- [音频相关 API](08_Audio_APIs.md)
- [系统声音管理](09_SystemSoundManager.md)
- [其他 API](10_Other_APIs.md)

## 构建与编译

- [GN 构建系统](11_GN_Build.md)
- [编译产物说明](12_Build_Artifacts.md)

## 内部实现

- [Inner API 参考](13_Inner_API.md)
- [引擎实现](14_Engine_Implementation.md)

## 安全

- [安全风险评审](15_Security_Review.md)

## 附录

- [故障排查指南](appendix/Troubleshooting.md)
- [常见问题](appendix/FAQ.md)

## 阅读路径建议

### 新人快速上手
1. [项目概览](01_Overview.md) → 2. [目录结构](02_Directory_Structure.md) → 3. [N-API 概览](05_NAPI_Overview.md) → 4. 对应模块 API

### 架构研究者
1. [项目概览](01_Overview.md) → 2. [系统架构](03_Architecture.md) → 4. [数据流](04_DataFlow_Threading.md) → 5. [引擎实现](14_Engine_Implementation.md)

### 安全审计
1. [安全风险评审](15_Security_Review.md) → 2. 架构相关章节

### 构建开发
1. [GN 构建系统](11_GN_Build.md) → 2. [编译产物说明](12_Build_Artifacts.md)

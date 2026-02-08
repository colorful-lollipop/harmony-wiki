# 全站导航

## 核心文档

### [README](README.md)
文档概述、覆盖范围、新人阅读路线

### [00. 项目概览](00_Overview.md)
- 项目定位与边界
- 核心能力
- 运行环境
- 关键概念

### [01. 架构设计](01_Architecture.md)
- 整体架构图
- 模块职责
- 组件关系
- 线程模型
- 关键时序

### [02. 目录结构](02_DirectoryStructure.md)
- 顶层目录
- 源代码组织
- 资源文件
- 配置文件

### [03. 系统 API](03_SystemAPIs.md)
- 电话/短信 API
- 数据存储 API
- 通知 API
- 系统服务 API

### [04. 数据流](04_DataFlow.md)
- 短信发送流程
- 短信接收流程
- 数据存储流程
- 通知流程

### [05. 安全评审](05_SecurityReview.md)
- 攻击面分析
- 信任边界
- 风险点清单
- 修复建议

### [06. 构建产物](06_BuildArtifacts.md)
- HAP 包结构
- 签名配置
- 安装路径
- 运行时加载

### [07. 问题排查](07_Troubleshooting.md)
- 常见问题
- 调试方法
- 日志路径
- 定位技巧

---

## 附录

### [附录 A: 调用链](appendix/Callgraphs.md)
- 短信发送调用链
- 短信接收调用链
- 通知调用链

### [附录 B: 常量定义](appendix/Constants.md)
- 状态码
- URI 定义
- 配置键值

---

## 阅读顺序建议

### 🔰 新手上路
```
README → 00_Overview → 02_DirectoryStructure → 01_Architecture(架构图部分)
```

### 🔧 开发调试
```
00_Overview → 01_Architecture → 03_SystemAPIs → 04_DataFlow → 07_Troubleshooting
```

### 🔒 安全审计
```
00_Overview → 01_Architecture → 05_SecurityReview → 附录A(调用链)
```

### 📦 构建部署
```
00_Overview → 02_DirectoryStructure → 06_BuildArtifacts
```

---

## 快速链接

### 关键文件位置
- 主入口: `entry/src/main/ets/MainAbility/MainAbility.ts`
- 短信接收: `entry/src/main/ets/StaticSubscriber/MmsStaticSubscriber.ts`
- 发送服务: `entry/src/main/ets/service/SendMsgService.ets`
- 会话服务: `entry/src/main/ets/service/ConversationService.ets`
- 常量定义: `entry/src/main/ets/data/commonData.ets`
- 权限配置: `entry/src/main/module.json5`

### 外部依赖
- [telephony_sms_mms](https://gitee.com/openharmony/telephony_sms_mms) - 底层短信服务
- [applications_contacts](https://gitee.com/openharmony/applications_contacts) - 联系人应用

---

*最后更新: 2026-02-05*

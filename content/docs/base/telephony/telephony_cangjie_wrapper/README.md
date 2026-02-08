# telephony_cangjie_wrapper Wiki

## 概述

 telephony_cangjie_wrapper 是 OpenHarmony 中面向仓颉（Cangjie）语言开发者的电话呼叫管理能力封装，提供拨打电话、获取通话属性、格式化电话号码等功能。

**核心能力**：
- 拨打电话（跳转到系统拨号界面）
- 获取通话状态与属性
- 格式化电话号码（标准格式 / E.164）

**支持设备**：standard 设备（需要扬声器/麦克风、SIM 卡）

**当前状态**：Beta 特性

## 文档覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 概览与架构 | ✅ | 项目定位、目录结构 |
| API 参考 | ✅ | 仓颉 API 清单与签名 |
| 构建系统 | ✅ | GN targets 与编译产物 |
| 内部架构 | ✅ | 模块职责与 FFI 边界 |
| 安全评审 | ✅ | 威胁建模与风险点 |

## 快速开始

### 安装依赖
```bash
# 需要以下子系统依赖
# - call_manager: 原生通话管理实现
# - ability_cangjie_wrapper: 应用上下文
# - hiviewdfx_cangjie_wrapper: 日志能力
# - cangjie_ark_interop: 跨语言互操作框架
```

### 基本使用
```cj
import ohos.telephony.call.{Call, CallState}

// 拨打电话
Call.makeCall("13800138000")

// 获取通话状态
let state = Call.getCallState()

// 格式化号码
let formatted = Call.formatPhoneNumber("13800138000")
```

## 相关资源

- [API 参考文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/TelephonyKit/cj-apis-telephony-call.md)
- [仓颉电话使用指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/telephony/cj-telephony-call.md)
- [OpenHarmony 贡献指南](https://gitcode.com/openharmony/docs/blob/master/en/contribute/code-contribution.md)

## 文档更新

**最后更新**：2026-02-06

**更新方式**：手动维护，随代码变更同步更新

**源码位置**：`/Volumes/lexar/code/d/work/oh/base/telephony/telephony_cangjie_wrapper/wiki/`

# 概览与架构

## 项目定位

 telephony_cangjie_wrapper 是 OpenHarmony 电话子系统的仓颉语言封装层，为仓颉开发者提供电话呼叫管理能力。

**核心职责**：
- 封装原生 call_manager 的呼叫功能
- 提供仓颉友好的 FFI 接口
- 处理错误码映射与异常抛出

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                     应用层 (Cangjie)                         │
│  kit.TelephonyKit → ohos.telephony.call.Call                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   封装层 (Cangjie FFI)                       │
│  ohos/telephony/call/:                                        │
│    - call.cj          (API 实现)                              │
│    - telephony_call_ffi.cj (FFI 声明)                        │
│    - number_format_options.cj (数据类型)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (external_deps: call_manager:cj_telephony_call_ffi)
┌─────────────────────────────────────────────────────────────┐
│                   原生层 (C++)                               │
│  telephony_call_manager (外部依赖)                           │
│    - 通话状态管理                                             │
│    - 拨号/接听/挂断逻辑                                       │
│    - 紧急号码判断                                             │
└─────────────────────────────────────────────────────────────┘
```

## 依赖关系

### 直接依赖

| 依赖模块 | 用途 | 类型 |
|----------|------|------|
| [call_manager](https://gitcode.com/openharmony/telephony_call_manager) | 原生通话管理实现 | external_deps (C++) |
| [ability_cangjie_wrapper](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper) | UIAbilityContext, Want | cj_external_deps |
| [hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) | HiLog 日志 | cj_external_deps |
| [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) | APILevel, BusinessException | cj_external_deps |

### 支持的 System Capability

- `SystemCapability.Telephony.CallManager` - 通话管理基础能力
- `SystemCapability.Applications.Contacts` - 联系人应用能力

## 目录结构

```
telephony_cangjie_wrapper/
├── figures/                      # 架构图资源
├── kit/
│   └── TelephonyKit/
│       ├── index.cj              # Kit 层重新导出
│       └── BUILD.gn              # kit.TelephonyKit 构建配置
├── ohos/
│   └── telephony/
│       ├── telephony.cj           # 空包 (stub for Windows/Mac)
│       └── call/
│           ├── call.cj           # Call 类主体
│           ├── telephony_call_ffi.cj  # FFI 声明
│           ├── number_format_options.cj  # 数据类型/错误码
│           └── BUILD.gn          # ohos.telephony.call 构建配置
├── mock/                         # Windows/Mac 构建占位
│   ├── ohos.telephony.cj
│   └── ohos.telephony.call.cj
├── BUILD.gn                      # 根构建配置
└── bundle.json                   # 组件描述
```

## 功能矩阵

| 功能 | 状态 | API | 说明 |
|------|------|-----|------|
| 拨打电话 | ✅ | `Call.makeCall()` | 跳转到拨号界面 |
| 获取通话状态 | ✅ | `Call.getCallState()` | 返回 CallState 枚举 |
| 判断是否有通话 | ✅ | `Call.hasCall()` | Bool 返回 |
| 判断语音能力 | ✅ | `Call.hasVoiceCapability()` | Bool 返回 |
| 紧急号码判断 | ✅ | `Call.isEmergencyPhoneNumber()` | 带 slotId 选项 |
| 号码格式化 | ✅ | `Call.formatPhoneNumber()` | 标准格式 |
| E.164 格式化 | ✅ | `Call.formatPhoneNumberToE164()` | 国际格式 |

**注意**：以下功能暂不支持（相比 ArkTS API）：
- 蜂窝数据
- eSIM 卡管理
- 订阅管理
- 网络搜索
- SIM 卡管理
- 短信服务 (SMS)

## 运行环境要求

### 硬件要求
- 扬声器或听筒
- 麦克风
- 已插入 SIM 卡

### 系统要求
- OpenHarmony API Level 22+
- standard 设备类型
- cangjie_ark_interop 子系统

## 约束与限制

1. **Beta 状态**：API 可能在后续版本中变更
2. **平台限制**：仅支持 standard 设备（Linux），Windows/Mac 使用 mock 实现
3. **异步操作**：部分 API 在 workerthread 执行，需注意线程切换

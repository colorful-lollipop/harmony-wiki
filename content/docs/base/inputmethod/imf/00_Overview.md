# 项目概述

## 项目定位

**OpenHarmony 输入法框架 (inputmethod_imf)** 是 OpenHarmony 系统中负责连接应用与输入法的核心子系统。其主要作用是：

1. **拉通应用和输入法**：保证应用可以通过输入法进行文本输入
2. **统一输入交互**：提供统一的接口给应用和输入法进行交互
3. **管理输入会话**：处理输入法的显示、隐藏、切换等生命周期

## 核心能力

| 能力 | 描述 |
|------|------|
| 应用客户端交互 | 应用与输入法框架的绑定、显示/隐藏请求 |
| 输入法客户端管理 | 输入法状态监听、输入能力交付 |
| 输入法服务 | 核心输入逻辑处理 |
| JS/ArkTS 接口 | 对外暴露的 JS/ArkTS API |
| NDK C 接口 | 原生开发接口 |
| 多窗口支持 | 支持多显示设备 |

## 运行环境

| 属性 | 值 |
|------|-----|
| **操作系统** | OpenHarmony 标准系统 (standard) |
| **系统能力** | `SystemCapability.MiscServices.InputMethodFramework` |
| **组件版本** | 3.1 |
| **目标设备** | 智能手机、平板、穿戴设备等 |
| **SA ID** | 3008 (`INPUT_METHOD_SYSTEM_ABILITY_ID`) |
| **启动模式** | 按需启动 (On-demand) |

### 系统能力声明

```json
{
  "component": {
    "name": "imf",
    "subsystem": "inputmethod",
    "syscap": ["SystemCapability.MiscServices.InputMethodFramework"],
    "adapted_system_type": ["standard"]
  }
}
```

### 对外暴露面

| 暴露类型 | 存在性 | 位置 | 说明 |
|----------|--------|------|------|
| **N-API 接口** | ✅ | `frameworks/js/napi/*` | 7 个 JS/ArkTS 模块 |
| **IPC 接口 (SA)** | ✅ | `services/src/input_method_system_ability.cpp` | SystemAbility 跨进程调用 |
| **Inner API** | ✅ | `interfaces/inner_api/*` | C++ 内部接口 |
| **NDK C API** | ✅ | `frameworks/ndk/*` | 原生 C 接口 |
| **配置文件** | ✅ | `etc/init/`, `etc/para/` | 服务配置、参数配置 |
| **命令行工具** | ✅ | `tools/ime/ime` | IME 调试工具 |

### 依赖的系统服务

- `samgr` - 服务管理器
- `ability_runtime` - Ability 运行时
- `window_manager` - 窗口管理器
- `input` - 多模输入服务
- `bundle_framework` - 包管理服务
- `access_token` - 访问令牌服务

## 关键概念

### 四大核心模块

| 模块 | 路径 | 职责 |
|------|------|------|
| 应用客户端 | `frameworks/native/inputmethod_controller` | 实现应用与输入法框架服务交付 |
| 输入法客户端 | `frameworks/native/inputmethod_ability` | 输入法框架服务与输入法交付的中间桥梁 |
| 输入法服务 | `services` | 输入法核心处理逻辑 |
| JS 接口 | `frameworks/js/napi` | 对外暴露的 JS/ArkTS 接口 |

### 客户端类型

| 类型 | 描述 |
|------|------|
| `INNER_KIT` | 内部套件调用 |
| `CLIENT` | 普通应用客户端 |
| `EXTENTION` | 扩展客户端 |

### 输入类型

| 类型 | 描述 |
|------|------|
| `PATTERN_TEXT` | 文本输入 |
| `PATTERN_NUMBER` | 数字输入 |
| `PATTERN_EMAIL` | 邮箱输入 |
| `PATTERN_PASSWORD` | 密码输入 |
| 等等... |

## 仓库信息

- **仓库路径**: `/base/inputmethod/imf`
- **组件名称**: `imf`
- **子系统**: `inputmethod`
- **许可证**: Apache 2.0

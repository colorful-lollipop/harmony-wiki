# N-API 接口

## 概述

**本项目不涉及 N-API（Native API）接口暴露。**

安全隐私中心是一个纯前端 ArkTS/ETS 应用，所有功能通过系统 JavaScript/TypeScript API 实现，不涉及 Native 层 C/C++ 代码的封装与暴露。

## 证据说明

### 代码层面证据

| 检查项 | 结果 | 说明 |
|-------|------|------|
| .cpp/.c/.h 源文件 | 0 | 项目中不存在任何 C/C++ 源文件 |
| napi_ 前缀函数 | 0 | 搜索结果为空 |
| NAPI_MODULE 宏 | 0 | 搜索结果为空 |
| napi_module_register | 0 | 搜索结果为空 |
| napi_define_properties | 0 | 搜索结果为空 |

**搜索命令**：
```bash
find . -type f \( -name "*.cpp" -o -name "*.c" -o -name "*.h" \) 2>/dev/null
grep -r "napi_\|NAPI_MODULE" --include="*.ts" --include="*.ets" . 2>/dev/null
```

### 项目配置证据

| 配置文件 | 内容 |
|---------|------|
| `README.md:79` | 明确声明："不涉及" |
| `entry/src/main/ets/` | 目录下全为 .ets 文件，无 native 代码 |
| `build-profile.json5` | 配置为 ArkTS 模块，无 native 编译配置 |

## 为什么不涉及 N-API

本项目的设计定位是：

1. **系统设置入口**：作为系统设置应用的一部分，主要提供 UI 交互
2. **系统 API 调用**：通过 OpenHarmony 提供的 JS/TS 系统 API 访问系统能力
3. **轻量级模块**：不需要 Native 层的高性能计算或底层访问

## 替代方案

虽然本项目不暴露 N-API，但通过以下系统 API 实现功能：

| 功能 | API 模块 | 示例调用 |
|-----|---------|---------|
| 位置开关控制 | `@ohos.geoLocationManager` | `geolocation.isLocationEnabled()` |
| 应用包信息 | `@ohos.bundle.bundleManager` | `bundleManager.getAllBundleInfoByFunctionAccess()` |
| 数据存储 | `@ohos.data.relationalStore` | `relationalStore.RdbPredicates` |
| 账户管理 | `@ohos.account.osAccount` | `osAccount.getAccountManager()` |
| 页面路由 | `@ohos.router` | `router.pushUrl()` |
| UI 扩展 | `@ohos.app.ability.uiSessionManager` | `UIExtensionComponent` |

## 相关文档

| 文档 | 链接 |
|-----|------|
| OpenHarmony JS/TS API 参考 | 开发者文档 |
| 应用接入指导 | [auto-menu-guidelines.md](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/SecurityPrivacyCenter/auto-menu-guidelines.md) |

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [03_Architecture.md](./03_Architecture.md) → 架构设计
- [05_Inner_API.md](./05_Inner_API.md) → 内部 API

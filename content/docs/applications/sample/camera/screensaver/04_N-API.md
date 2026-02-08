# N-API 对外接口

## 1. 重要说明

### 1.1 本项目不涉及 N-API

**核心结论**：本项目是 **原生 C++ 应用**，**不提供** N-API 接口供 JS/TS 调用。

**原因分析**：
1. 项目基于 `ability_lite` 框架，而非 ArkTS/JS 框架
2. 使用 `REGISTER_AA` / `REGISTER_AS` 宏注册 Ability（原生注册）
3. 没有使用 `NAPI_MODULE` 或 `napi_` 系列函数
4. 没有导出 JS 可调用的符号

**证据**：
- `screensaver_ability.cpp:19` - `REGISTER_AA(ScreensaverAbility)`
- `screensaver_ability_slice.cpp:20` - `REGISTER_AS(ScreensaverAbilitySlice)`

---

## 2. N-API 定位说明

### 2.1 什么是 N-API

N-API 是 OpenHarmony 提供的原生模块接口，允许 C/C++ 模块被 JS/TS 调用。典型特征包括：
- 使用 `NAPI_MODULE` 宏注册模块
- 使用 `napi_define_properties` 导出函数
- 使用 `napi_create_*` 创建 JS 类型

### 2.2 本项目不适用 N-API 的原因

| 原因 | 说明 |
|------|------|
| 架构定位 | 屏保是系统级功能，由系统调度触发 |
| 调用方式 | 通过 Ability 生命周期回调启动，非 JS 调用 |
| 入口类型 | 使用 page ability 类型，非 service ability |

---

## 3. 相关参考

如需了解 OpenHarmony N-API 用法，请参考：
- [N-API 官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/README.md)
- [Ability Framework 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/ability/ability-lite-guidelines.md)

---

## 4. 文档导航

- **返回**：[概览](01_Overview.md) → 项目定位
- **下一步**：[内部 API](05_Inner_API.md) → C++ 模块接口
- **相关**：[架构设计](03_Architecture.md) → 组件关系

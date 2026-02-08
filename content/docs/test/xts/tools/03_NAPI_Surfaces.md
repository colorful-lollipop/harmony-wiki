# 03_NAPI_Surfaces.md

## 目的

本文档梳理 XTS Tools 仓库中可能存在的 N-API（JavaScript Native API）暴露情况，包括 API 清单、参数、返回值、错误码等。

## 适用范围

- 仅涉及 tools/ 仓库内的 N-API 导出
- 若不存在 N-API，将明确说明证据范围

---

## 总体结论

- **本仓库不包含任何 N-API 实现**（已通过全局搜索确认）
- 证据：
  - napi_、NAPI_MODULE、napi_module_register、napi_define_properties 这些符号仅在 `wiki/_work/NOTES.md` 和 `wiki/_work/PLAN.md` 中被提及（作为待办事项），并未在实际源代码中使用
  - 代码库中的 C/C++ 文件（共12个）都是测试工具相关的，使用传统测试框架（Unity、GTest），而非 N-API
  - 未发现 `#include <node_api.h>`、`#include <napi.h>` 或类似头文件引用
- tools 作为测试工具包，不直接面向开发者提供 N-API；N-API 实现位于 OpenHarmony 系统框架层的其他仓库

---

## N-API 模块清单

TODO: 经 Phase 1 扫描后补充。若确认无 N-API，将说明“本仓库不对外提供 N-API”。

---

## API 清单表

| JS API 名称/命名空间 | 参数/返回值 | 同步/异步 | 对应 C/C++ 入口与绑定位置 | 参数校验与错误码 | 权限/前置条件 |
|----------------------|-------------|------------|----------------------------|------------------|----------------|
| 不适用 | 不适用 | 不适用 | 不适用 | 不适用 | 不适用 |

说明：本仓库不对外提供 N-API。

---

## 关键 API 调用链

不适用（本仓库不对外提供 N-API）。

---

## 参数解析与校验策略

不适用（本仓库不对外提供 N-API）。

---

## 错误码与异常封装

不适用（本仓库不对外提供 N-API）。

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [02_Architecture.md](02_Architecture.md)
- [04_Inner_APIs.md](04_Inner_APIs.md)

# 对外 N-API 接口

## 目的

本文档说明 DSLM 模块对外提供的 JavaScript Native API（N-API）接口。

## 适用范围

- ✅ N-API 接口清单
- ✅ 接口详细说明
- ✅ 参数校验与错误码

## 重要说明

**DSLM 模块不提供 N-API（JavaScript Native API）接口。**

## 证据

- **grep 搜索结果**：未找到 `napi_`、`NAPI_MODULE`、`napi_module_register`、`napi_define_properties` 关键字
- **文件搜索结果**：无 .ts/.js 绑定文件
- **对外接口文件**：`interfaces/inner_api/include/device_security_info.h` - 仅提供 C 原生接口

## 对外接口类型

### Native C API（C 接口）

DSLM 提供 **4 个 C 原生接口**，仅对系统组件开放，不对应用层开放。

详见 [01_Project_Boundaries.md](./01_Project_Boundaries.md) 和 [00_Overview.md](./00_Overview.md)。

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [01_Project_Boundaries.md](./01_Project_Boundaries.md) - C API 详细说明
- [05_Inner_APIs.md](./05_Inner_APIs.md) - 内部接口详解

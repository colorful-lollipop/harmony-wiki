# N-API 模块文档

## 目的

本文档说明 OpenHarmony Hisilicon 板卡仓库的 N-API (Node-API) 模块情况。

## 适用范围

本文档适用于：
- 开发者了解板卡层是否有 N-API 模块
- 系统工程师理解 API 分层

---

## N-API 模块清单

**结论: 本仓库不包含任何 N-API 模块**

**证据**:
- **搜索范围**: 所有板卡目录，文件类型 `*.c`, `*.cpp`, `*.h`, `*.ts`, `*.js`
- **搜索模式**: `napi_`, `NAPI_MODULE`, `napi_module_register`, `napi_define_properties`
- **搜索结果**: 无匹配项

**原因分析**:
- 本仓库是**板卡级（Board Level）** 仓库
- N-API 模块位于**应用框架层**或**子系统层**
- 板卡层主要负责硬件抽象和驱动配置

---

## N-API 在 OpenHarmony 中的位置

```
┌─────────────────────────────────────────────────────┐
│         N-API 模块（应用框架层）                   │
│   foundation/arkui/napi/                          │
│   各子系统 N-API 实现                               │
│   例如:                                          │
│   - foundation/multimedia/camera_framework/napi       │
│   - foundation/multimedia/audio_framework/napi        │
│   - base/communication/wifi/services/napi           │
└─────────────────────────────────────────────────────┘
              ↑
              │ 依赖
              │
┌─────────────────────────────────────────────────────┐
│         板卡层（本仓库）                             │
│   device_board_hisilicon/                         │
│   - 硬件配置                                       │
│   - U-Boot                                        │
│   - 驱动 HAL                                       │
│   - Secure Boot                                    │
└─────────────────────────────────────────────────────┘
```

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和边界
- [架构说明](02_Architecture.md) - 系统架构和组件关系
- [内部 API](03_Inner_API.md) - 板卡层内部接口

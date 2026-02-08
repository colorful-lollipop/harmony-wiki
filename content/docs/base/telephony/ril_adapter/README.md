# RIL Adapter Wiki

> RIL Adapter 项目技术文档

## 文档范围

本文档集全面描述 `ril_adapter` 模块的架构、接口、构建和安全特性，帮助开发人员快速理解和维护该模块。

### 包含内容

- 项目概览与边界
- 目录结构与模块职责
- 架构设计（含组件图和数据流）
- 内部 API 文档
- GN 构建配置与产物
- 安全风险分析

### 不包含内容

- **测试代码**：所有 `test/` 目录相关内容
- **JavaScript API**：本项目不提供 N-API 层（纯 C/C++ 服务）
- **上层服务**：`telephony_core_service`、`drivers_peripheral` 等上层模块

## 更新方式

本文档基于源代码自动生成和分析。当源代码变更后，建议执行以下步骤更新文档：

1. 重新运行代码扫描工具
2. 更新 `wiki/_work/NOTES.md` 记录新发现
3. 根据变更更新对应章节
4. 更新本文档的"最后更新时间"

## 快速导航

请从 [SUMMARY.md](SUMMARY.md) 开始阅读，了解完整文档结构。

---

## 技术栈

| 层级 | 技术 |
|------|------|
| **接口层** | C/C++ 头文件 (interfaces/innerkits) |
| **服务层** | C++ (services/hril) |
| **HDF 桥接层** | C (services/hril_hdf) |
| **Vendor 层** | C (services/vendor) |
| **构建系统** | GN |
| **通信框架** | HDF (Hardware Driver Foundation) |
| **IPC** | HDI (Hardware Device Interface) V1.5 |

---

## 关键概念

- **HRIL** (Hardware RIL Interface Adapter)：RIL 适配层的核心服务
- **HDF** (Hardware Driver Foundation)：OpenHarmony 硬件驱动框架
- **HDI** (Hardware Device Interface)：硬件设备接口，用于跨进程通信
- **Vendor 库**：由 modem 厂商提供的库，实现 `HRilOps` 接口
- **AT 命令**：与 modem 通信的标准命令集

---

## 依赖组件

- `bounds_checking_function`: 边界检查函数库
- `c_utils`: C 语言工具库
- `drivers_interface_ril`: RIL HDI 接口 (v1.5)
- `drivers_interface_power`: 电源 HDI 接口 (可选)
- `hdf_core`: HDF 核心库
- `hilog`: 日志系统
- `init`: 初始化工具
- `ipc`: IPC 通信库
- `samgr`: 系统服务管理器

---

## 文档结构

```
wiki/
├── README.md              # 项目导航入口
├── SUMMARY.md             # 全站导航 + 推荐阅读路径
├── 01_Overview.md         # 项目概览
├── 02_Architecture.md     # 架构与数据流
├── 03_CodeMap.md         # 代码地图
├── 04_Interface.md       # 接口文档
├── 05_AttackSurface.md   # 攻击面分析
├── 06_SecurityReview.md  # 安全风险评估
├── 07_Build.md          # 构建与产物
└── _work/                # 工作区
    ├── ASSESS.md        # 项目评估报告
    ├── NOTES.md          # 代码证据库
    └── PLAN.md          # 任务进度追踪
```

---

最后更新时间: 2026-02-07

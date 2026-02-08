# OpenHarmony Power Manager 模块 Wiki

## 概述

本文档是 OpenHarmony `power_manager` 模块的工程 Wiki，旨在帮助开发者快速理解项目架构、API 用法、构建配置和安全注意事项。

**当前版本**: v1.0  
**最后更新**: 2026-02-06  
**数据来源**: 基于代码仓库 `//base/powermgr/power_manager` 静态分析

---

## 覆盖范围

### 已涵盖内容
- 模块定位与核心能力
- 目录结构与模块职责
- N-API 接口清单（JS/TS）
- Inner API 接口（C++）
- SA/IPC 架构
- GN 构建目标与编译产物
- Feature Flags 配置
- 工具模块说明

### 未涵盖内容
- 详细时序图与调用链（待补充）
- 线程模型细节（待补充）
- 运行时行为说明（待补充）
- 性能调优指南（待补充）

---

## 快速导航

```
├── 01_Overview.md           # 项目概览
├── 02_Architecture.md       # 系统架构
├── 03_NAPI_Reference.md     # N-API 接口参考
├── 04_Inner_API.md         # 内部 C++ API
├── 05_SA_IPC.md            # SA/IPC 通信架构
├── 06_Build.md             # 构建配置与产物
├── 07_Security.md          # 安全风险评审
├── 08_FAQ.md               # 常见问题
├── appendix/
│   ├── Callgraphs.md       # 关键调用链
│   └── Config_Flags.md     # 特性开关清单
└── SUMMARY.md              # 全文导航
```

---

## 项目定位

**模块名称**: `@ohos/power_manager`  
**子系统**: `powermgr`  
**系统能力**: `SystemCapability.PowerManager.PowerManager.Core`

### 核心功能
1. **设备电源控制**: 关机、重启、休眠、唤醒
2. **运行锁管理**: 防止系统休眠的后台任务锁
3. **电源状态管理**: 状态机、屏幕控制、电源模式
4. **事件回调**: 状态变化、关机、挂起的异步通知

### 版本信息
- **bundle.json 版本**: 3.1
- **目标系统**: OpenHarmony Standard (标准系统)
- **代码规模**: 核心服务 113KB + 状态机 121KB

---

## 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **应用层 API** | N-API | JavaScript/TypeScript 接口 |
| **ArkTS API** | ETS/Taihe | ArkTS 运行时绑定 |
| **Cangjie API** | FFI | Cangjie 语言绑定 |
| **Native API** | C++ | 内部 C++ 客户端库 |
| **IPC 通信** | ZIDL | OpenHarmony IDL IPC 框架 |
| **系统服务** | SA | SystemAbility 架构 |

---

## 代码结构概览

```
power_manager/
├── frameworks/              # 框架层 (客户端 API)
│   ├── native/            # Native C++ 客户端
│   ├── napi/              # JavaScript/TypeScript 绑定
│   ├── ets/taihe/         # ArkTS 运行时绑定
│   └── cj/                # Cangjie FFI 绑定
├── interfaces/            # 接口定义层
│   └── inner_api/         # 内部 API 头文件
├── services/              # 服务层 (核心实现)
│   ├── native/            # 服务端实现
│   └── zidl/              # IPC 接口定义
├── utils/                 # 工具层
├── power_dialog/          # 电源对话框 UI
├── sa_profile/            # SA 配置文件
└── etc/                   # 系统配置
```

---

## 更新说明

### 如何更新本文档

1. **代码变更后**: 运行 `python3 gen_wiki.py` (如有) 或手动更新相关章节
2. **新增 API**: 在 `03_NAPI_Reference.md` 添加 API 清单
3. **新增模块**: 更新 `01_Overview.md` 的目录结构
4. **构建变更**: 更新 `06_Build.md` 的目标清单

### 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v1.0 | 2026-02-06 | 初始版本，完成基础文档 |

---

## 贡献指南

1. 所有文档修改必须基于代码证据（文件路径+符号）
2. 禁止引用测试代码作为业务证据
3. API 文档必须包含完整参数说明
4. 安全评审必须包含可追溯的代码路径

---

## 许可证

本项目遵循 Apache License 2.0。

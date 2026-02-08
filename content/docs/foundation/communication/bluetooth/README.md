# OpenHarmony Bluetooth 模块 Wiki

> **生成时间**: 2025-02-06  
> **目标仓库**: /Volumes/lexar/code/d/work/oh/foundation/communication/bluetooth  
> **文档版本**: 1.0.0

## 文档覆盖范围

本 Wiki 旨在为 OpenHarmony Bluetooth 模块提供完整的技术文档，覆盖以下方面：

| 分类 | 覆盖内容 | 状态 |
|------|----------|------|
| 项目概览 | 定位、边界、核心能力、运行环境 | ✅ |
| 目录结构 | 模块职责划分（不含测试） | ✅ |
| 架构设计 | 组件图、数据流、线程模型、关键时序 | ⏳ |
| 对外 API | N-API（JS/TS 层面）、导出符号、权限/参数/错误码 | ⏳ |
| 内部 API | 模块接口、依赖方向、稳定性标注 | ⏳ |
| GN 构建 | targets 列表、依赖关系、编译产物 | ⏳ |
| 安全评审 | 攻击面、信任边界、风险点、修复建议 | ⏳ |
| 附录 | 关键调用链、配置开关 | ⏳ |

## 文档结构

```
wiki/
├── README.md                    # 本文档（覆盖范围、更新方式）
├── SUMMARY.md                   # 全站导航 + 新人阅读顺序
├── 00_Overview.md              # 项目概览
├── 01_Directory_Structure.md   # 目录结构
├── 02_Architecture.md          # 架构设计
├── 03_API_Reference.md         # API 参考
│   ├── 03_NAPI_Reference.md   # N-API 详细参考
│   └── 03_CAPI_Reference.md   # C API 参考
├── 04_Build_System.md          # 构建系统
├── 05_Security_Review.md       # 安全评审
├── 06_Troubleshooting.md       # 问题排查
└── appendix/
    ├── Callgraphs.md          # 关键调用链
    └── Config_Flags.md        # 配置开关
```

## 阅读建议（新人路线）

### 路线 A：快速上手（约 15 分钟）
1. `README.md` → 了解文档结构
2. `00_Overview.md` → 理解项目定位
3. `01_Directory_Structure.md` → 熟悉代码布局
4. `03_API_Reference.md` → 查看 API 使用方式

### 路线 B：深入开发（约 45 分钟）
完成路线 A 后，继续：
1. `02_Architecture.md` → 理解架构设计
2. `04_Build_System.md` → 掌握编译流程
3. `appendix/Callgraphs.md` → 追踪关键调用链
4. `05_Security_Review.md` → 了解安全注意事项

### 路线 C：架构贡献者（约 2 小时）
完成路线 A+B 后，深入：
1. 精读 `02_Architecture.md` 全部章节
2. 研究 `04_Build_System.md` 的 targets 依赖
3. 审查 `05_Security_Review.md` 的风险模型
4. 贡献代码前先阅读 `06_Troubleshooting.md`

## 证据与引用规范

所有关键结论必须基于代码证据，格式如下：

| 类型 | 格式 | 示例 |
|------|------|------|
| 文件路径 | `path:line` | `native_module.cpp:92` |
| 符号定义 | 符号名 (类型) | `bluetoothModule` (`napi_module`) |
| 调用链 | 入口 → 中间 → 目标 | `EnableBt() → HostProxy → SA 1130` |
| 配置项 | 文件:值 | `bluetooth.gni: bluetooth_kia_enable = false` |

**示例**：
> 蓝牙 N-API 模块通过 `napi_module_register()` 注册（证据：`native_module.cpp:109`）。

## 更新方式

### 何时需要更新文档

| 场景 | 操作 |
|------|------|
| 新增 N-API 模块 | 更新 `03_NAPI_Reference.md` |
| 新增 GN target | 更新 `04_Build_System.md` |
| 架构重构 | 更新 `02_Architecture.md` 和 `appendix/Callgraphs.md` |
| 发现安全风险 | 更新 `05_Security_Review.md` |
| 修复构建问题 | 更新 `06_Troubleshooting.md` |

### 更新流程

1. 在对应章节添加/修改内容
2. 确保所有结论有代码证据（路径+符号）
3. 更新 `SUMMARY.md` 的链接（如新增页面）
4. 运行 `lsp_diagnostics` 验证 Markdown 语法
5. 提交 PR 时附带文档变更

## 术语表

| 术语 | 含义 | 代码位置 |
|------|------|----------|
| SAID | System Ability ID | `bluetooth_service_ipc_interface_code.h:22` |
| N-API | Node.js API（ArkTS 绑定层） | `frameworks/js/napi/` |
| Proxy | IPC 客户端代理 | `bluetooth_host_proxy.cpp` |
| Stub | IPC 服务端存根 | `bluetooth_host_stub.cpp` |
| FFI | Foreign Function Interface | `frameworks/cj/` |

## 局限性说明

1. **测试代码不引用**：本 Wiki 仅基于生产代码分析，不引用测试用例作为证据
2. **服务实现分离**：SA 1130 的服务端实现在独立仓库，本 Wiki 仅覆盖客户端框架
3. **版本锁定**：文档基于当前 commit `HEAD` 生成，后续代码变更需同步更新

---

**维护者**: OpenHarmony Bluetooth 团队  
**问题反馈**: 在 Gitee 仓库提交 Issue

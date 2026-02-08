# Permission Lite Wiki

## 文档说明

本文档为 OpenHarmony `permission_lite` 子系统的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、编译构建及安全风险。

### 覆盖范围

本文档涵盖以下内容：

| 分类 | 覆盖内容 |
|------|----------|
| 项目概览 | 定位、边界、核心能力、运行环境 |
| 目录结构 | 模块职责划分（不含测试） |
| 架构说明 | 组件关系、数据流、线程模型 |
| 对外 API | C 接口、JS API、IPC 认证接口 |
| 内部架构 | 模块依赖、Inner API、稳定性标注 |
| GN 构建 | Targets 列表、依赖关系、编译产物 |
| 安全评审 | 攻击面、信任边界、风险点 |

### 适用范围

- **系统类型**：Mini System (≥128 KiB RAM)、Small System (≥1 MiB RAM)
- **目标设备**：ARM Cortex-M/RISC-V (Mini)、ARM Cortex-A (Small)
- **调用方**：仅限系统应用和系统服务

### 关键约束

| 约束 | 说明 |
|------|------|
| 调用权限 | 所有 API 仅供系统使用，第三方应用通过系统服务间接调用 |
| 编译产物 | ROM ~150KB，RAM ~500KB |
| 许可证 | Apache License 2.0 |

### 更新方式

本文档基于代码注释、接口定义和构建配置自动生成。如需更新：

1. 修改源码注释或接口定义
2. 更新 `bundle.json` 或 `BUILD.gn` 配置
3. 重新运行文档生成脚本

### 相关链接

| 资源 | 链接 |
|------|------|
| 源码仓库 | [security_permission_lite](https://gitee.com/openharmony/security_permission_lite) |
| OpenHarmony | [官方文档](https://www.openharmony.cn/) |
| SAMGR | 系统能力管理框架 |

---

## 阅读指南

### 新人阅读路线

建议按以下顺序阅读：

1. **概览** → `01_Overview.md` - 快速了解项目定位
2. **架构** → `02_Architecture.md` - 理解核心组件和数据流
3. **API** → `03_APIs.md` - 掌握对外接口使用方法
4. **构建** → `04_Build.md` - 了解编译配置和产物
5. **安全** → `05_Security.md` - 关注安全风险

### 高级主题

| 主题 | 文档 |
|------|------|
| 内部 API 详解 | `appendix/06_Inner_APIs.md` |
| 调用链分析 | `appendix/07_Callgraphs.md` |
| 配置参数 | `appendix/08_Config_Flags.md` |

---

## 目录结构

```
wiki/
├── README.md                    # 文档说明（本文档）
├── SUMMARY.md                   # 全站导航
├── 01_Overview.md              # 项目概览
├── 02_Architecture.md          # 架构说明
├── 03_APIs.md                  # 对外 API
├── 04_Build.md                 # 构建与产物
├── 05_Security.md              # 安全风险评审
└── appendix/
    ├── 06_Inner_APIs.md        # 内部 API
    ├── 07_Callgraphs.md        # 调用链图谱
    └── 08_Config_Flags.md     # 配置参数
```

---

## 文档版本

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2024-02-06 | 初始版本 |

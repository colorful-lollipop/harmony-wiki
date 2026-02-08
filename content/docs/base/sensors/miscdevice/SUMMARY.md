# 文档导航

## 快速开始

- [首页](README.md) - 项目概览与导航
- [00_Overview](00_Overview.md) - 项目定位与核心概念

## 核心文档

| 序号 | 文档 | 内容说明 |
|------|------|----------|
| 01 | [01_Architecture](01_Architecture.md) | 分层架构图、数据流、线程模型、时序图 |
| 02 | [02_NAPI](02_NAPI.md) | JS/C/ArkTS API 清单、参数说明、调用链 |
| 03 | [03_Build](03_Build.md) | GN Targets、编译产物、Feature Flags |
| 04 | [04_Security](04_Security.md) | 攻击面、信任边界、风险点、修复建议 |

## 附录

| 文档 | 内容说明 |
|------|----------|
| [Callgraphs](appendix/Callgraphs.md) | 关键 API 调用链图示 |
| [Config_Flags](appendix/Config_Flags.md) | 编译开关与 Feature Flags 详解 |

## 文档结构

```
wiki/
├── README.md              # 首页导航
├── SUMMARY.md            # 本文件，全站导航
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md     # 架构设计
├── 02_NAPI.md            # API 接口文档
├── 03_Build.md           # 构建配置
├── 04_Security.md        # 安全评审
└── appendix/
    ├── Callgraphs.md     # 调用链图示
    └── Config_Flags.md   # 配置开关
```

## 阅读建议

### 新人入门
1. `README.md` → 了解项目定位
2. `00_Overview.md` → 理解核心概念
3. `01_Architecture.md` → 把握整体架构

### API 使用
1. `02_NAPI.md` → 查找 API 清单
2. 查看示例代码

### 开发调试
1. `01_Architecture.md` → 理解模块边界
2. `03_Build.md` → 了解构建配置
3. `appendix/Callgraphs.md` → 追踪调用链

### 安全审计
1. `04_Security.md` → 查看攻击面
2. 重点关注权限检查点

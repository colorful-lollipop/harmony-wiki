# 文档导航

## 快速开始

- [首页](README.md)
- [项目概述](01_Overview.md)

## 核心文档

- [架构设计](02_Architecture.md)
- [代码地图](03_CodeMap.md) — 关键代码文件定位
- [N-API 接口参考](04_NAPI_Reference.md)
- [Inner API 参考](05_Inner_API.md)
- [GN 构建指南](06_Build_GN.md)

## 安全分析

- [攻击面分析](07_AttackSurface.md) — 外部入口与信任边界
- [安全风险评审](08_Security_Review.md) — 详细风险评估与修复建议

## 运维与附录

- [故障排查](09_Troubleshooting.md)
- [关键调用链](appendix/Callgraphs.md)
- [编译配置开关](appendix/Config_Flags.md)

---

## 阅读路线图

### 新人入门

```mermaid
flowchart LR
    A[README] --> B[01_概述]
    B --> C[03_N-API]
    C --> D[05_构建]
```

### 开发者深入

```mermaid
flowchart LR
    A[02_架构] --> B[04_Inner API]
    B --> C[06_安全]
    C --> D[07_故障排查]
```

---

## 关键链接

| 主题 | 入口 |
|-----|------|
| JS API 列表 | [03_NAPI_Reference.md](03_NAPI_Reference.md#api-清单) |
| GN Targets | [05_Build_GN.md](05_Build_GN.md#targets-清单) |
| 安全风险 | [06_Security_Review.md](06_Security_Review.md) |
| 编译产物 | [05_Build_GN.md](05_Build_GN.md#编译产物) |

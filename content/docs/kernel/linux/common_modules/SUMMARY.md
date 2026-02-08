# Wiki 导航

> 快速导航 | [首页](README.md) | [项目概览](00_Overview.md) | [模块清单](01_Modules.md)

---

## 一、入门指南

| 章节 | 内容 | 难度 |
|------|------|------|
| [README.md](README.md) | Wiki 说明、更新方式、覆盖范围 | ⭐ |
| [00_Overview.md](00_Overview.md) | 项目定位、许可证、目录结构 | ⭐ |
| [01_Modules.md](01_Modules.md) | 所有模块功能概览 | ⭐ |

---

## 二、架构与设计

| 章节 | 内容 | 难度 |
|------|------|------|
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型 | ⭐⭐ |
| [03_Build_System.md](03_Build_System.md) | GN 构建配置、targets 清单 | ⭐⭐ |

---

## 三、模块文档（按优先级）

### P0 - 安全核心模块

| 模块 | 功能 | 难度 |
|------|------|------|
| [modules/tzdriver.md](modules/tzdriver.md) | TrustZone 驱动、REE-TEE 通信 | ⭐⭐⭐ |
| [modules/memory_security.md](modules/memory_security.md) | 内存安全、KASLR/SMEP/SMAP 防护 | ⭐⭐⭐ |
| [modules/pac.md](modules/pac.md) | 指针认证码、控制流保护 | ⭐⭐⭐ |
| [modules/container_escape_detection.md](modules/container_escape_detection.md) | 容器逃逸检测、进程监控 | ⭐⭐⭐ |

### P1 - 安全功能模块

| 模块 | 功能 | 难度 |
|------|------|------|
| [modules/code_sign.md](modules/code_sign.md) | 代码签名、证书链管理 | ⭐⭐ |
| [modules/qos_auth.md](modules/qos_auth.md) | QoS 认证、资源调度 | ⭐⭐ |
| [modules/newip.md](modules/newip.md) | 新 IP 协议栈、网络安全 | ⭐⭐⭐ |
| [modules/xpm.md](modules/xpm.md) | 可执行权限管理、代码完整性 | ⭐⭐ |

### P2 - 基础与示例

| 模块 | 功能 | 难度 |
|------|------|------|
| [modules/ucollection.md](modules/ucollection.md) | 性能采集、CPU 维测数据 | ⭐ |
| [modules/module_sample.md](modules/module_sample.md) | 模块示例、BUILD.gn 模板 | ⭐ |

---

## 四、安全与故障

| 章节 | 内容 | 难度 |
|------|------|------|
| [04_Security_Review.md](04_Security_Review.md) | 攻击面分析、风险点、修复建议 | ⭐⭐⭐ |
| [05_Troubleshooting.md](05_Troubleshooting.md) | 构建问题、调试方法、定位路径 | ⭐⭐ |

---

## 五、贡献者

| 资源 | 说明 |
|------|------|
| [README.md#贡献流程](../README.md#贡献流程) | 模块合入流程 |
| [README.md#ko模块指导](../README.md#ko模块指导) | 内核模块构建指南 |

---

## 阅读建议

### 新人入门
```
README.md → 00_Overview.md → 01_Modules.md → module_sample.md
```

### 安全研究人员
```
04_Security_Review.md → modules/pac.md → modules/memory_security.md
```

### 内核开发
```
03_Build_System.md → modules/tzdriver.md → modules/newip.md
```

### 架构师
```
02_Architecture.md → 04_Security_Review.md → 所有模块文档
```

---

*最后更新: 2026-02-06*

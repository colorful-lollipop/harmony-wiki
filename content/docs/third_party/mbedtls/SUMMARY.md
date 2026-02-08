# mbedtls OpenHarmony 集成文档

## 文档阅读指南

本文档描述 mbedtls 库在 OpenHarmony 系统中的集成与适配情况。

### 推荐阅读路径

#### 新手快速入门
```
README.md → 01_Overview.md → 04_Usage_in_OH.md
```

#### 开发者参考
```
README.md → 03_Build_Integration.md → 02_OH_Adaptation.md
```

#### 安全相关
```
README.md → 06_Security.md → 04_Usage_in_OH.md
```

#### 维护者升级指南
```
README.md → 02_OH_Adaptation.md → 06_Security.md
```

## 文档结构

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](./README.md) | 库概览、OH 适配概述、文档导航 | 必读 |
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 定位 | 推荐 |
| [02_OH_Adaptation.md](./02_OH_Adaptation.md) | OH 适配层详细分析 | 核心 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配 | 核心 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 推荐 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异 | 参考 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 安全相关 |

## 快速索引

### 关键 OH 适配点

- **Port 适配层**: `port/` 目录
- **配置文件**: `port/config/config_liteos_*.h`
- **构建入口**: `BUILD.gn`, `mbedtls.gni`
- **TLS 客户端**: `port/src/tls_client.c`

### 主要依赖模块

- 分布式软总线 (dsoftbus)
- 密钥管理服务 (HUKS)
- 应用验证 (appverify)
- 系统日志 (hiviewdfx)

## 版本信息

- **mbedtls 版本**: v3.6.5
- **OH 组件版本**: 5.0
- **文档版本**: 1.0
- **最后更新**: 2024

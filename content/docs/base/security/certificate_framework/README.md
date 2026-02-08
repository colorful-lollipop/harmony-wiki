# 证书算法库框架 (Certificate Framework) Wiki

## 概述

本文档为 OpenHarmony 证书算法库框架的完整技术文档，旨在帮助开发者快速理解项目架构、使用方法及内部实现细节。

## 文档覆盖范围

### 已覆盖内容

- **项目定位与核心能力**：证书、CRL、证书链的解析与校验能力
- **架构设计**：三层架构（API接口层、框架实现层、算法库适配层）
- **N-API 接口**：完整的 JavaScript API 参考
- **内部 C API**：核心框架 API 和类型定义
- **GN 构建系统**：编译配置与产物说明
- **安全风险分析**：基于代码证据的安全评审

### 未覆盖内容

- 具体算法库实现细节（OpenSSL/Mbed TLS 内部）
- 完整测试用例分析
- 运行时性能优化指南

## 文档更新方式

本文档基于代码自动生成，文档与代码同步更新。当代码发生以下变化时需要重新生成：

1. 新增/删除/修改 N-API 接口
2. 修改构建配置或 targets
3. 新增安全相关代码

**生成命令**：
```bash
# 在项目根目录执行
./generate_wiki.sh  # 假设存在
```

## 文档结构

```
wiki/
├── README.md              # 本文件
├── SUMMARY.md             # 全站导航
├── index.md               # 快速入门
├── 01_Architecture.md     # 架构说明
├── 02_NAPI_Reference.md  # JS API 参考
├── 03_Inner_API.md       # 内部 C API
├── 04_Build_System.md     # GN 构建系统
├── 05_Artifacts.md       # 编译产物
├── 06_Security_Review.md # 安全评审
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md   # 配置开关
```

## 相关资源

- **OpenHarmony 官方文档**：[DeviceCertificateKit](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/DeviceCertificateKit/)
- **接口文档**：[JS APIs - Certificate](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-device-certificate-kit/js-apis-cert.md)
- **源码仓库**：[security_certificate_framework](https://gitcode.com/openharmony/security_certificate_framework)

## 反馈与贡献

如发现文档错误或需要补充内容，请提交 Issue 或 PR 到源码仓库。

---

*最后更新：2024-02-06*
*基于代码版本：certificate_framework v4.0*

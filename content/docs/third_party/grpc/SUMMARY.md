# gRPC Wiki 阅读指南

## 阅读路线建议

根据您的角色和目的，选择不同的阅读路线：

### 路线 1: 快速了解（5分钟）

适合：项目经理、新接触 OH 的开发者

1. [README.md](./README.md) - 概览
2. [01_Overview.md](./01_Overview.md) - 阅读 "该库在 OH 中的作用和定位" 部分
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖关系图

### 路线 2: 集成开发（15分钟）

适合：需要使用 gRPC 的开发者

1. [01_Overview.md](./01_Overview.md) - 完整阅读
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解如何在自己的模块中使用
3. [05_API_Differences.md](./05_API_Differences.md) - 检查是否有 API 差异
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 参考使用示例

### 路线 3: 维护升级（30分钟）

适合：负责 gRPC 维护的开发者

1. [02_Patches.md](./02_Patches.md) - 详细了解所有 Patch
2. [03_Build_Integration.md](./03_Build_Integration.md) - 完整理解构建配置
3. [06_Security.md](./06_Security.md) - 了解安全状况
4. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 查看完整评估报告

### 路线 4: 安全审计（20分钟）

适合：安全工程师

1. [06_Security.md](./06_Security.md) - 完整阅读
2. [02_Patches.md](./02_Patches.md) - 了解 Patch 引入的安全风险
3. [03_Build_Integration.md](./03_Build_Integration.md) - 查看编译选项中的安全相关配置

## 文档依赖关系

```
README.md (入口)
    ├── 01_Overview.md (基础)
    │       └── 依赖: README.OpenSource, bundle.json
    ├── 02_Patches.md (核心)
    │       └── 依赖: 所有 *.patch 文件
    ├── 03_Build_Integration.md (核心)
    │       └── 依赖: BUILD.gn
    ├── 04_Usage_in_OH.md (核心)
    │       └── 依赖: bundle.json, OH 代码库分析
    ├── 05_API_Differences.md (可选)
    └── 06_Security.md (可选)
```

## 术语表

| 术语 | 说明 |
|------|------|
| gRPC | Google 开源的 RPC 框架 |
| protobuf | Protocol Buffers，Google 的数据序列化协议 |
| GN | Generate Ninja，OH 的元构建系统 |
| Bundle | OH 的组件包管理单元 |
| Inner Kit | OH 组件对外暴露的接口 |
| Patch | 对上游代码的修改 |
| CVE | 常见漏洞和暴露（安全术语） |

## 相关资源

- [gRPC 官方文档](https://grpc.io/docs/)
- [Protocol Buffers 文档](https://protobuf.dev/)
- [OH 构建系统文档](https://gitee.com/openharmony/docs)

## 反馈与贡献

如发现文档错误或有改进建议，请：

1. 在代码仓库创建 Issue
2. 或联系 gRPC 组件维护者

---

**文档版本**: 1.0 (对应 gRPC v1.73.0)

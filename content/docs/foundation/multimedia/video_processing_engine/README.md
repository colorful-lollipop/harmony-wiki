# VPE 视频处理引擎 Wiki 文档

本文档为 OpenHarmony VPE（Video Processing Engine）视频处理引擎的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建系统和安全风险。

---

## 文档覆盖范围

本 Wiki 覆盖以下内容：

| 分类 | 覆盖内容 |
|------|---------|
| **项目定位** | 核心能力、模块边界、依赖关系 |
| **架构设计** | 组件图、数据流、线程模型、调用时序 |
| **对外 API** | N-API（JS/TS）、C API 接口清单与使用示例 |
| **内部 API** | Inner API 模块接口、依赖方向、稳定性标注 |
| **构建系统** | GN Targets 列表、依赖关系、编译产物 |
| **运行时** | 产物安装路径、动态库加载关系 |
| **安全评审** | 攻击面、信任边界、风险点与修复建议 |
| **问题定位** | 常见构建/运行/调试问题与解决方案 |

---

## 文档未覆盖范围

| 分类 | 说明 |
|------|------|
| **测试代码** | test/ 目录下所有测试用例、测试框架 |
| **单元测试** | *_test.*、unit_test、fuzz_test 相关内容 |
| **算法实现细节** | 第三方算法库内部实现（如 EVE、AI 超分） |
| **运行时配置** | 非源码内的系统配置、HAL 层实现 |

---

## 文档更新方式

### 1. 手动更新

当代码发生以下变更时，需同步更新 Wiki：

| 变更类型 | 更新内容 | 负责人 |
|---------|---------|--------|
| 新增/删除 N-API | 更新 `NAPI_Reference.md` API 清单 | 接口作者 |
| 新增模块/服务 | 更新 `Architecture.md` 组件图 | 架构师 |
| 修改构建配置 | 更新 `Build_System.md` Targets | 构建维护者 |
| 新增安全风险 | 更新 `Security_Review.md` | 安全审计者 |
| 修复已知问题 | 更新 `Troubleshooting.md` | 开发者 |

### 2. 自动生成（TODO）

当前 Wiki 为手工编写，未来可考虑：
- 使用 Clang AST 工具自动生成 API 文档
- 使用 Build 脚本自动提取 Targets 清单
- 使用静态分析工具自动更新安全风险清单

---

## 文档版本

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，完成基础架构和 API 文档 |

---

## 阅读指南

### 新人阅读顺序（推荐）

```mermaid
graph LR
    A[README] --> B[概览]
    B --> C[架构设计]
    C --> D[对外API]
    D --> E[构建系统]
    E --> F[运行时加载]
    F --> G[安全评审]
    G --> H[问题定位]
```

1. **README.md** - 了解项目整体定位和目录结构
2. **index.md** - 快速概览核心能力
3. **Architecture.md** - 深入理解模块划分和数据流
4. **NAPI_Reference.md** - 查阅具体 API 使用方法
5. **Build_System.md** - 了解如何编译项目
6. **Artifacts.md** - 了解产物安装和加载
7. **Security_Review.md** - 了解安全注意事项
8. **Troubleshooting.md** - 常见问题快速定位

### 快速查找

| 需求 | 跳转链接 |
|------|---------|
| 查找 JS/TS API | [NAPI_Reference.md](./NAPI_Reference.md) |
| 查找 C API | [NAPI_Reference.md](./NAPI_Reference.md) |
| 查找 Inner API | [Inner_API.md](./Inner_API.md) |
| 了解构建配置 | [Build_System.md](./Build_System.md) |
| 了解安全风险 | [Security_Review.md](./Security_Review.md) |
| 解决问题 | [Troubleshooting.md](./Troubleshooting.md) |

---

## 符号说明

| 符号 | 含义 |
|------|------|
| ✅ | 已验证（可在代码中找到证据） |
| ⚠️ | 待确认（需要进一步验证） |
| ❌ | 不存在（已确认代码中无此功能） |
| `路径:行号` | 代码证据位置 |

---

## 贡献指南

### 贡献方式

1. **Issue 反馈**：发现文档错误或遗漏，请提 Issue
2. **Pull Request**：直接修改文档并提交 PR
3. **讨论**：在 Commits 中参与技术讨论

### 贡献要求

- 所有关键结论必须有代码证据支持
- 术语使用需与代码保持一致
- 避免引用测试代码作为业务证据
- 文档语言统一使用中文（简体）

---

## 联系方式

- **仓库地址**：`//foundation/multimedia/video_processing_engine`
- **维护团队**：OpenHarmony 多媒体组
- **问题反馈**：请在 OpenHarmony Gitee 仓库提 Issue

---

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [VPE 源码仓库](https://gitee.com/openharmony/multimedia_video_processing_engine)
- [API 参考](https://docs.openharmony.cn/api/)

# User File Service Wiki - 导航

## 文档概览

本文档提供 Wiki 全站导航，支持双受众阅读路线：
- **新人学习路线**：快速理解项目、上手开发
- **安全研究路线**：识别攻击面、评估安全风险

---

## 双路线导航

### 路线一：新人学习路线 👨‍💻

面向 OpenHarmony 开发者和新成员，快速理解项目定位、架构和接口使用。

```
┌─────────────────────────────────────────────────────────────────┐
│  第一阶段：理解项目 (15分钟)                                      │
│  ├── 00_Overview.md       → 项目定位、能力边界、运行环境          │
│  └── 01_Architecture.md   → 组件图、数据流、线程模型              │
│                                                                 │
│  第二阶段：使用接口 (20分钟)                                      │
│  ├── 02_NAPI_Reference.md → JS API 完整参考                      │
│  └── 03_Inner_API.md      → 内部 API 使用指南                    │
│                                                                 │
│  第三阶段：构建部署 (10分钟)                                      │
│  ├── 04_GN_Build.md       → GN 构建系统                          │
│  └── 05_Build_Artifacts.md → 编译产物、安装路径                  │
│                                                                 │
│  第四阶段：问题排查 (按需)                                        │
│  └── 07_Troubleshooting.md → 常见问题、调试技巧                  │
└─────────────────────────────────────────────────────────────────┘
```

### 路线二：安全研究路线 🔒

面向安全研究员和审计人员，识别攻击面、分析信任边界、评估安全风险。

```
┌─────────────────────────────────────────────────────────────────┐
│  第一阶段：全局认知 (15分钟)                                      │
│  ├── 00_Overview.md        → 项目类型、对外暴露面                 │
│  └── 01_Architecture.md    → 信任边界、数据流图                  │
│                                                                 │
│  第二阶段：攻击面分析 (30分钟) ⭐ 关键                            │
│  └── 05_AttackSurface.md   → 外部输入清单、敏感操作、入口点        │
│                                                                 │
│  第三阶段：风险评估 (40分钟)                                      │
│  └── 06_Security_Review.md → 风险点、利用路径、修复建议          │
│                                                                 │
│  第四阶段：代码审计 (按需)                                        │
│  ├── 02_NAPI_Reference.md  → API 入口点                          │
│  └── 03_Inner_API.md       → IPC 接口、权限检查点                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 目录索引

### 入门必读

| 文档 | 描述 | 受众 |
|------|------|------|
| [README](README.md) | Wiki 使用说明、更新指南 | 双受众 |
| [00_Overview](00_Overview.md) | 项目定位、边界、核心能力、目录结构 | 双受众 |

### 架构与实现

| 文档 | 描述 | 受众 |
|------|------|------|
| [01_Architecture](01_Architecture.md) | 组件图、数据流、线程模型、时序图、信任边界 | 双受众 |
| [03_Inner_API](03_Inner_API.md) | 模块接口、依赖方向、稳定性标注 | 开发者 |

### 接口参考

| 文档 | 描述 | 受众 |
|------|------|------|
| [02_NAPI_Reference](02_NAPI_Reference.md) | N-API (JS API) 完整参考、调用链、错误码 | 开发者 |

### 构建与部署

| 文档 | 描述 | 受众 |
|------|------|------|
| [04_GN_Build](04_GN_Build.md) | GN Targets、构建配置、安全编译选项 | 开发者 |
| [05_Build_Artifacts](05_Build_Artifacts.md) | 编译产物清单、安装路径、运行时加载 | 开发者 |

### 安全分析 ⭐

| 文档 | 描述 | 受众 |
|------|------|------|
| [05_AttackSurface](05_AttackSurface.md) | 攻击面清单、外部输入、敏感操作、信任边界图 | 安全研究员 |
| [06_Security_Review](06_Security_Review.md) | 风险点评析、利用路径、修复建议、缓解措施 | 安全研究员 |

### 运维支持

| 文档 | 描述 | 受众 |
|------|------|------|
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题、定位路径、调试技巧 | 开发者 |

---

## 快速查找

### 按主题查找

| 主题 | 相关文档 |
|------|---------|
| **项目是什么** | 00_Overview.md |
| **架构如何设计** | 01_Architecture.md |
| **API 怎么用** | 02_NAPI_Reference.md |
| **如何构建** | 04_GN_Build.md |
| **产物在哪里** | 05_Build_Artifacts.md |
| **攻击面在哪** | 05_AttackSurface.md |
| **有什么风险** | 06_Security_Review.md |
| **遇到问题** | 07_Troubleshooting.md |

### 按关键词查找

| 关键词 | 文档 |
|--------|------|
| `file.fileAccess` | 02_NAPI_Reference.md |
| `file.picker` | 02_NAPI_Reference.md |
| `SA 5010` | 01_Architecture.md, 05_AttackSurface.md |
| `CheckCallingPermission` | 05_AttackSurface.md, 06_Security_Review.md |
| `IsFilePathValid` | 05_AttackSurface.md, 06_Security_Review.md |
| `路径遍历` | 05_AttackSurface.md, 06_Security_Review.md |
| `IPC` | 01_Architecture.md, 03_Inner_API.md |
| `BUILD.gn` | 04_GN_Build.md |

---

## 核心速查表

### N-API 模块清单

| JS 模块 | 命名空间 | 注册文件 | 行号 |
|---------|----------|----------|------|
| FileAccess | `@ohos.file.fileAccess` | `native_fileaccess_module.cpp` | 81 |
| FileExtensionInfo | `@ohos.file.fileExtensionInfo` | `module_export_napi.cpp` | 49 |
| Picker | `@ohos.file.picker` | `native_module_ohos_picker.cpp` | 81 |
| Recent | `@ohos.file.recent` | `recent/module.cpp` | 53 |
| Trash | `@ohos.file.trash` | `trash/module.cpp` | - |
| CloudDiskManager | `@ohos.file.clouddiskmanager` | `clouddiskmanager/module.cpp` | 50 |

### 系统服务

| 服务名 | SA ID | 头文件 | 关键方法 |
|--------|-------|--------|----------|
| FileAccessService | 5010 | `file_access_service.h` | `CheckCallingPermission():227` |

### 关键安全函数

| 函数 | 路径 | 行号 | 用途 |
|------|------|------|------|
| `IsFilePathValid` | `file_uri_check.h` | 29 | 路径遍历检查 |
| `CheckCallingPermission` | `file_access_ext_stub_impl.cpp` | 38 | IPC 权限校验 |
| `CheckPermission` | `ufs_access_token_helper.cpp` | 63 | Token 权限验证 |

---

## 版本信息

| 项目 | 版本 |
|------|------|
| OpenHarmony | 3.1+ (Standard) |
| user_file_service | 3.1 |

---

## 更新日志

| 日期 | 更新内容 |
|------|----------|
| 2026-02-07 | 优化导航结构，新增双路线指引、05_AttackSurface.md |
| 2026-02-06 | 初始化 Wiki 文档 |

---

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [FileAccessFramework 架构图](figures/file_access_framework.png)
- [bundle.json](bundle.json) - 组件配置
- [BUILD.gn](BUILD.gn) - 构建入口

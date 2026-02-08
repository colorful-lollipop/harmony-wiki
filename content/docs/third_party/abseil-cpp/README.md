# Abseil-CPP（OpenHarmony 集成文档）

> Abseil 是 Google 开源的 C++ 基础库集合，旨在增强 C++ 标准库。本文档重点记录 Abseil 在 OpenHarmony 中的集成与适配。

---

## 文档导航

### 快速了解

| 文档 | 描述 | 读者 |
|------|------|------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | 所有人 |
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的作用和定位 | 新手 |
| [02_Patches.md](./02_Patches.md) | **Patch/适配详细分析**（核心文档） | 开发者、维护者 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 | 构建工程师 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 架构师、开发者 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异（如有） | API 设计者 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 安全工程师 |
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果（Phase 0） | 分析师 |

---

## 核心信息

| 属性 | 值 |
|------|-----|
| **上游版本** | 20250127.0 |
| **OH 版本** | 3.1 |
| **许可证** | Apache License 2.0 |
| **上游地址** | https://github.com/abseil/abseil-cpp |
| **OH 子系统** | thirdparty |
| **OH 部件** | abseil-cpp |

---

## 适配特点

### 1. 无侵入式集成

- **无 .patch 文件**：所有适配通过 `__OHOS__` 条件编译宏实现
- **保守式禁用**：对不确定的平台特性选择禁用而非崩溃

### 2. 功能限制

| 功能模块 | 状态 | 说明 |
|---------|------|------|
| 核心库（容器、字符串、同步、时间、哈希） | ✅ 完全可用 | 所有核心功能正常 |
| 调试/符号化 | ⚠️ 限制 | 堆栈跟踪、信号处理被禁用 |
| CPU 特性检测 | ⚠️ 限制 | ARM64 HWCAP 检测被禁用 |
| C++ 运行时 | ⚠️ 限制 | 符号还原被禁用 |

### 3. 构建系统集成

- **26 个库目标**：25 个共享库 + 1 个静态库
- **安全加固**：4 个关键目标启用 PAC-RET 分支保护
- **系统镜像分发**：所有共享库设置 `install_enable = true`

---

## 主要使用者

| 子系统 | 主要模块 | 用途 |
|--------|---------|------|
| **developtools** | profiler, hiperf | 性能分析工具 |
| **thirdparty** | gRPC, RE2, protobuf | RPC、正则、序列化 |
| **arkcompiler** | es2panda, merge_abc | TS 编译器 |

**依赖链**：`应用 → protobuf/grpc → abseil-cpp`

---

## 技术债务

| 项目 | 状态 |
|------|------|
| **NDEBUG 硬编码** | 需要调查 PR #1206 并移除 |
| **堆栈跟踪未实现** | 使用 `unimplemented` stub，可考虑 OHOS 原生实现 |
| **符号还原禁用** | 需验证 OHOS libc++ 是否支持 |

---

## 文档结构

```
wiki/
├── README.md                      # 本文件
├── SUMMARY.md                     # 阅读路线建议
├── 01_Overview.md                 # 原始库简介
├── 02_Patches.md                  # Patch/适配详细分析（核心）
├── 03_Build_Integration.md         # OH 构建适配
├── 04_Usage_in_OH.md             # 依赖关系与使用
├── 05_API_Differences.md          # API 差异（如有）
├── 06_Security.md                # 安全风险分析
└── _work/
    ├── ASSESSMENT.md              # 项目评估结果
    ├── NOTES.md                  # 分析过程记录
    └── PLAN.md                  # 任务进度
```

---

## 贡献指南

### 修改 abseil-cpp 集成

1. **添加新的 __OHOS__ 宏使用**：更新 `02_Patches.md`
2. **修改构建配置**：更新 `03_Build_Integration.md`
3. **修复上游兼容性**：在 `02_Patches.md` 中记录回归风险
4. **启用新特性**：在 `05_API_Differences.md` 中说明 API 变更

### 升级上游版本

1. **检查 `__OHOS__` 宏**：确保所有适配在新版本中仍然有效
2. **重新分析依赖关系**：使用 `grep "third_party/abseil-cpp"` 验证构建
3. **运行测试**：测试所有依赖模块（profiler、grpc、protobuf 等）
4. **更新文档**：更新版本号和变更日志

---

## 相关资源

- [上游仓库](https://github.com/abseil/abseil-cpp)
- [上游文档](https://abseil.io/)
- [OpenHarmony 构建系统](https://gitee.com/openharmony/build)
- [OpenHarmony 文档](https://docs.openharmony.cn/)

---

**最后更新**: 2026-02-07
**维护者**: OpenHarmony 第三方库维护团队

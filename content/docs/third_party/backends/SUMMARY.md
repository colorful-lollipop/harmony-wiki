# 阅读路线指南

本文档提供 SANE-backends Wiki 的阅读路线建议，根据不同角色和需求提供不同的阅读路径。

---

## 快速导航

| 文档 | 适合场景 | 阅读时间 |
|------|----------|----------|
| [README.md](./README.md) | 快速了解库概览 | 5 分钟 |
| [01_Overview.md](./01_Overview.md) | 了解 OH 定位和架构 | 10 分钟 |
| [02_Patches.md](./02_Patches.md) | 深入理解 OH 适配细节 | 20 分钟 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建和编译相关 | 15 分钟 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 使用和集成相关 | 15 分钟 |

---

## 按角色推荐

### 如果你是一名...

#### 应用开发者
**目标**: 在我的应用中使用扫描功能

**推荐阅读顺序**:
1. [README.md](./README.md) - 了解基础信息
2. [01_Overview.md](./01_Overview.md) - 了解 API 兼容性
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 学习如何使用 SANE API

**重点关注**:
- API 兼容性说明
- 完整扫描流程示例代码
- 调试和故障排除

---

#### 系统开发者
**目标**: 集成或修改 SANE-backends

**推荐阅读顺序**:
1. [README.md](./README.md) - 概览
2. [01_Overview.md](./01_Overview.md) - OH 架构定位
3. [02_Patches.md](./02_Patches.md) - 理解所有 Patch
4. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统

**重点关注**:
- 4 个 OH 特有 Patch 的详细分析
- BUILD.gn 结构和配置
- 依赖关系和集成方式

---

#### 维护者/升级负责人
**目标**: 维护 SANE-backends，处理上游升级

**推荐阅读顺序**:
1. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估报告
2. [02_Patches.md](./02_Patches.md) - 深入理解每个 Patch
3. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统

**重点关注**:
- Patch 清单和分类
- 升级检查清单
- 推向上游评估
- 版本兼容性矩阵

---

#### 新接触此库的工程师
**目标**: 全面了解 SANE-backends 在 OH 中的情况

**推荐阅读顺序**:
1. [README.md](./README.md) - 库概览
2. [01_Overview.md](./01_Overview.md) - 原始库和 OH 定位
3. [02_Patches.md](./02_Patches.md) - Patch 分析（核心）
4. [03_Build_Integration.md](./03_Build_Integration.md) - 构建适配
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用方式

**预计总时间**: 约 60 分钟

---

## 按任务推荐

### 任务: 升级上游版本

**阅读章节**:
- [02_Patches.md - Patch 维护建议](./02_Patches.md#patch-维护建议)
- [02_Patches.md - 升级检查清单](./02_Patches.md#升级检查清单)
- [02_Patches.md - 版本兼容性矩阵](./02_Patches.md#版本兼容性矩阵)

**行动清单**:
1. [ ] 检查新版本是否包含已有 Patch 的功能
2. [ ] 验证 4 个 OH 特有 Patch 是否还能应用
3. [ ] 测试构建是否成功
4. [ ] 运行功能测试
5. [ ] 性能测试对比

---

### 任务: 添加新后端支持

**阅读章节**:
- [03_Build_Integration.md - BUILD.gn 结构](./03_Build_Integration.md#buildgn-结构)
- [03_Build_Integration.md - 构建目标详解](./03_Build_Integration.md#构建目标详解)
- [03_Build_Integration.md - 常见问题](./03_Build_Integration.md#常见问题)

**行动清单**:
1. [ ] 准备后端源代码
2. [ ] 修改 `BUILD.gn` 添加源文件
3. [ ] 添加必要的依赖
4. [ ] 测试构建
5. [ ] 验证功能

---

### 任务: 调试扫描问题

**阅读章节**:
- [04_Usage_in_OH.md - 调试和故障排除](./04_Usage_in_OH.md#调试和故障排除)
- [04_Usage_in_OH.md - 常见问题](./04_Usage_in_OH.md#常见问题)
- [02_Patches.md - hilog_debug.patch](./02_Patches.md#patch-2-hilog_debugpatch)

**行动清单**:
1. [ ] 启用 SANE 调试日志
2. [ ] 检查 HiLog 输出
3. [ ] 验证设备权限
4. [ ] 测试基础功能

---

### 任务: 优化扫描性能

**阅读章节**:
- [04_Usage_in_OH.md - 性能优化建议](./04_Usage_in_OH.md#性能优化建议)
- [02_Patches.md - add_thread_poll.patch](./02_Patches.md#patch-1-add_thread_pollpatch)

**行动清单**:
1. [ ] 检查线程池是否启用
2. [ ] 优化 dll.conf 后端列表
3. [ ] 调整缓冲区大小
4. [ ] 使用合适的图像格式

---

## 关键文档速查

### 核心概念速查表

| 概念 | 说明 | 详细文档 |
|------|------|----------|
| **SANE** | Scanner Access Now Easy - 扫描仪访问接口标准 | [01_Overview.md](./01_Overview.md) |
| **后端 (Backend)** | 特定扫描仪厂商/协议的驱动模块 | [02_Patches.md](./02_Patches.md) |
| **线程池** | OH 特有的并发设备发现优化 | [02_Patches.md](./02_Patches.md#patch-1-add_thread_pollpatch) |
| **HiLog** | OpenHarmony 统一日志系统 | [02_Patches.md](./02_Patches.md#patch-2-hilog_debugpatch) |
| **沙箱路径** | OH 应用隔离目录结构 | [03_Build_Integration.md](./03_Build_Integration.md#路径定义oh-沙箱) |

### Patch 速查表

| Patch | 功能 | OH 特有 | 升级注意 |
|-------|------|---------|----------|
| `add_thread_poll.patch` | 线程池并发设备发现 | 是 | 需重新适配 |
| `hilog_debug.patch` | HiLog 日志集成 | 是 | 需重新适配 |
| `modifying_driver_search_path.patch` | 驱动路径适配 | 是 | 需重新适配 |
| `modify_load_function.patch` | 错误码优化 | 是 | 需重新适配 |
| `Rules-quot.patch` | 国际化构建修复 | 否 | 可能已合并 |
| `ltmain.sh.patch` | 库名处理 | 否 | OH 使用 GN |
| `ax_create_stdint_h.*.patch` | Autotools 修复 | 否 | OH 使用 GN |

---

## 文档更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02 | 1.0 | 初始版本，基于 SANE-backends 1.4.0 |

---

## 反馈和贡献

如果发现文档中的错误或有改进建议，请通过以下方式反馈：

1. 提交 Issue 到 [third_party_backends](https://gitee.com/openharmony/third_party_backends/issues)
2. 在相关仓库提交 PR 更新 Wiki

---

## 附录: 相关资源

### 上游资源
- [SANE 官方网站](http://sane-project.org/)
- [SANE 标准文档](https://sane-project.gitlab.io/standard/)
- [上游 GitLab 仓库](https://gitlab.com/sane-project/backends)
- [支持的设备列表](http://www.sane-project.org/sane-supported-devices.html)

### OpenHarmony 资源
- [打印框架](https://gitee.com/openharmony/print_print_fwk)
- [OpenHarmony 文档](https://gitee.com/openharmony/docs)
- [构建系统指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-gn-coding-style.md)

### 开发工具
- [GN 构建系统](https://gn.googlesource.com/gn/+/main/docs/)
- [Ninja 构建](https://ninja-build.org/)

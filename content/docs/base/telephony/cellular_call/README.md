# Cellular Call 模块 Wiki

## 概述

本文档是 OpenHarmony telephony 子系统中 **Cellular Call（蜂窝通话）** 模块的完整技术 Wiki。

Cellular Call 模块提供设备的基础蜂窝通话服务，支持传统电路交换（CS）通话和现代 IP 多媒体子系统（IMS）通话，包括 VoLTE、VoWIFI、VoNR 等技术。

## 覆盖范围

### 本 Wiki 包含

| 章节 | 内容 | 状态 |
|------|------|------|
| [README](README.md) | 本文档说明 | ✅ 完成 |
| [SUMMARY](SUMMARY.md) | 全站导航 | ✅ 完成 |
| [01_Overview](01_Overview.md) | 项目概览与定位 | ✅ 完成 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责 | ✅ 完成 |
| [03_Architecture](03_Architecture.md) | 架构说明（三层模型） | ✅ 完成 |
| [04_Interfaces](04_Interfaces.md) | Inner API 接口规范 | 🔄 进行中 |
| [05_Inner_API](05_Inner_API.md) | 内部模块接口 | ⏳ 待完成 |
| [06_GN_Build](06_GN_Build.md) | GN 构建与编译产物 | ⏳ 待完成 |
| [07_Security_Review](07_Security_Review.md) | 安全风险评审 | ⏳ 待完成 |
| [08_Troubleshooting](08_Troubleshooting.md) | 常见问题与调试 | ⏳ 待完成 |
| [附录/Callgraphs](appendix/Callgraphs.md) | 关键调用链 | ⏳ 待完成 |

### 本 Wiki 不包含

- **测试代码相关内容**：测试代码不作为业务证据来源
- **外部依赖详细文档**：请参考各依赖模块的独立 Wiki
- **N-API 文档**：本模块不提供 JS/N-API 接口

## 项目定位

### 核心能力

```
┌─────────────────────────────────────────────────────────────┐
│                    Cellular Call 模块                        │
├─────────────────────────────────────────────────────────────┤
│  CS 通话           │  IMS 通话           │  卫星通话          │
│  • 2G/3G 通话     │  • VoLTE            │  (条件编译)        │
│  • 紧急呼叫       │  • VoWIFI           │                   │
│  • 补充业务       │  • VoNR             │                   │
│                  │  • 视频通话          │                   │
│                  │  • 会议通话          │                   │
├─────────────────────────────────────────────────────────────┤
│                   域选择与切换                               │
│              CS ↔ IMS 智能选择与无缝切换                     │
└─────────────────────────────────────────────────────────────┘
```

### 系统定位

- **子系统**: telephony
- **组件名**: `@ohos/cellular_call`
- **SA ID**: 4006
- **依赖方**: Call Manager (必须通过 Call Manager 间接使用)
- **语言**: C++

## 快速入门

### 新人阅读顺序（推荐）

1. **[01_Overview](01_Overview.md)** - 了解项目定位和核心能力
2. **[02_Directory_Structure](02_Directory_Structure.md)** - 熟悉代码组织结构
3. **[03_Architecture](03_Architecture.md)** - 掌握三层架构设计
4. **[04_Interfaces](04_Interfaces.md)** - 查看 Inner API 清单
5. **[06_GN_Build](06_GN_Build.md)** - 了解构建配置

### 代码证据索引

所有关键结论均可在以下位置找到证据：

| 主题 | 证据位置 |
|------|----------|
| 服务入口 | `services/manager/src/cellular_call_service.cpp` |
| IPC 权限检查 | `services/manager/src/cellular_call_stub.cpp:45-50` |
| SA 配置 | `sa_profile/4006.json` |
| 构建配置 | `BUILD.gn`, `cellularcall.gni` |
| 接口定义 | `interfaces/innerkits/ims/*.h` |

## 版本与更新

| 属性 | 值 |
|------|-----|
| 模块版本 | 4.0 |
| 代码语言 | C++ |
| ROM 占用 | ~1MB |
| RAM 占用 | ~650KB |
| 最后更新 | 2026-02-06 |

## 如何更新本文档

### 文档更新原则

1. **随代码变更同步更新**：API 变更时必须同步更新接口文档
2. **保持证据一致**：代码路径和符号变更时需同步更新文档
3. **新增功能需补充**：新功能需添加对应章节和调用链说明

### 更新步骤

```bash
# 1. 克隆仓库
git clone https://gitee.com/openharmony/telephony_cellular_call.git

# 2. 编辑对应 .md 文件
#    - 修改接口文档: wiki/04_Interfaces.md
#    - 修改架构文档: wiki/03_Architecture.md
#    - 等等...

# 3. 验证文档链接
#    - 检查 SUMMARY.md 链接完整性
#    - 检查内部跳转链接有效性

# 4. 提交变更
git add wiki/
git commit -m "docs: 更新 Wiki 文档 - [变更说明]"
git push
```

### 质量检查清单

更新后请确认：

- [ ] `SUMMARY.md` 中所有链接可正常访问
- [ ] 接口文档包含完整的参数和返回值说明
- [ ] 新增 API 已在清单中列出
- [ ] 安全相关变更已更新 [07_Security_Review](07_Security_Review.md)
- [ ] 无测试代码路径引用
- [ ] 中文术语使用一致

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [telephony_cellular_call](https://gitee.com/openharmony/telephony_cellular_call) | 本模块仓库 |
| [telephony_core_service](https://gitee.com/openharmony/telephony_core_service) | 核心服务依赖 |
| [telephony_call_manager](https://gitee.com/openharmony/telephony_call_manager) | 通话管理依赖 |

## 反馈与贡献

如发现文档错误或遗漏，请通过以下方式反馈：

1. **Issue**: 在仓库 Issue 中描述问题
2. **Pull Request**: 直接提交修改
3. **邮件**: 联系维护团队

---

*本文档由 Agent 自动生成，最后更新：2026-02-06*

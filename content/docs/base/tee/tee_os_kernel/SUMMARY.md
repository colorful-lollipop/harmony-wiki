# Wiki 导航

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **目的**: 新人快速学习 OpenHarmony TEE OS Kernel 的导航中心

---

## 新人阅读路线（推荐顺序）

### 第一阶段：概念理解（1-2 天）

1. [项目概览](00_Overview.md) ⏱️ **必读第一篇**
   - 了解 TEE OS 是什么
   - 掌握核心能力与特性
   - 理解关键概念（Capability、Badge、PMO、VMSpace）
   - 知道运行环境（ARM64 + RK3568/RK3399）
   - **预计阅读时间**: 2-3 小时

2. [目录结构](01_Directory_Structure.md) 📁 **必读第二篇**
   - 熟悉代码组织
   - 了解各模块职责
   - 快速定位代码位置
   - **预计阅读时间**: 1-2 小时

3. [架构设计](02_Architecture.md) 🏗️ **核心架构文档**
   - 理解微内核设计
   - 了解组件关系（模块依赖图）
   - 掌握线程模型和调度算法
   - 理解 IPC 架构
   - 理解内存管理（PMO、VMSpace）
   - **预计阅读时间**: 3-5 小时

### 第二阶段：接口与实现（2-4 天）

4. [系统调用接口](03_Syscall_Interfaces.md) 🔧 **开发者必读**
   - 掌握 256 个系统调用
   - 了解参数校验机制
   - 学习错误码定义
   - 了解 TEE 专用系统调用
   - **预计阅读时间**: 2-3 小时

5. [安全评审](07_Security_Review.md) 🔒 **安全相关必读**
   - 理解安全机制（Capability、Badge）
   - 了解攻击面
   - 学习信任边界
   - 查看可被利用点和修复建议
   - **预计阅读时间**: 2-3 小时

### 第三阶段：开发实践（按需阅读）

6. [内部 API](04_Internal_APIs.md) 🔬 **模块开发者阅读**
   - 了解模块间接口
   - 理解依赖关系
   - 掌握稳定/不稳定接口
   - **预计阅读时间**: 1-2 小时（按需）

7. [GN Targets](05_GN_Targets.md) 🔧 **构建相关阅读**
   - 了解构建系统
   - 掌握编译产物
   - 学习配置选项
   - **预计阅读时间**: 1 小时（按需）

8. [编译产物](06_Build_Artifacts.md) 📦 **部署相关阅读**
   - 了解最终产物
   - 掌握产物安装路径
   - 了解运行时加载关系
   - **预计阅读时间**: 1 小时（按需）

### 第四阶段：问题解决（随时查阅）

9. [常见问题](08_FAQ.md) ❓ **问题排查手册**
   - 构建问题解决方案
   - 运行时问题解决方案
   - 调试问题解决方案
   - 性能问题解决方案
   - **预计阅读时间**: 随时查阅

---

## 模块化学习路径

### 针对不同角色的学习路线

#### 🔬 内核开发者路径
```
00_Overview.md
    ↓
01_Directory_Structure.md
    ↓
02_Architecture.md (重点：sched, mm, ipc)
    ↓
03_Syscall_Interfaces.md
    ↓
04_Internal_APIs.md (重点：object, capability)
    ↓
07_Security_Review.md (重点：安全机制)
    ↓
05_GN_Targets.md
    ↓
06_Build_Artifacts.md
    ↓
08_FAQ.md (调试相关)
```

#### 🔬 应用/TA 开发者路径
```
00_Overview.md
    ↓
01_Directory_Structure.md
    ↓
03_Syscall_Interfaces.md (重点：系统调用)
    ↓
02_Architecture.md (重点：用户空间服务)
    ↓
05_GN_Targets.md
    ↓
06_Build_Artifacts.md
    ↓
08_FAQ.md
```

---

## 快速参考

### 常见任务快速查找

| 任务 | 查找文档 | 关键章节 |
|------|-----------|----------|
| 添加新系统调用 | [系统调用接口](03_Syscall_Interfaces.md) | "系统调用分类" |
| 添加新模块 | [目录结构](01_Directory_Structure.md) | 相关模块说明 |
| 理解 Capability | [项目概览](00_Overview.md#关键概念) | "Capability" |
| 了解 IPC | [架构设计](02_Architecture.md#ipc-架构) | "IPC 架构" |
| 调试问题 | [常见问题](08_FAQ.md#调试问题) | "GDB 调试" |
| 构建问题 | [常见问题](08_FAQ.md#构建问题) | "编译错误" |

### 关键概念速查表

| 概念 | 定义位置 | 文档 |
|------|-----------|------|
| **Capability** | `kernel/include/object/object.h` | [项目概览](00_Overview.md), [安全评审](07_Security_Review.md) |
| **Badge** | `kernel/include/object/cap_group.h` | [项目概览](00_Overview.md), [安全评审](07_Security_Review.md) |
| **PMO** | `kernel/include/object/memory.h` | [项目概览](00_Overview.md), [架构设计](02_Architecture.md#内存模型) |
| **VMSpace** | `kernel/include/mm/vmspace.h` | [架构设计](02_Architecture.md#内存模型) |
| **TEE UUID** | `kernel/include/common/tee_uuid.h` | [项目概览](00_Overview.md), [安全评审](07_Security_Review.md) |
| **系统调用** | `kernel/syscall/syscall_num.h` | [系统调用接口](03_Syscall_Interfaces.md) |

---

## 重要提示

### ⚠️ N-API 说明

**本仓库无 N-API 接口！**

tee_os_kernel 是 TEE 微内核，对外 API 通过系统调用（Syscall）和 IPC 提供。

- ❌ **不存在**：`napi_*`、`NAPI_MODULE`、`napi_define_properties`
- ✅ **实际接口**：256 个系统调用 + TEE 专用 IPC

N-API 功能由 [tee_os_framework](https://gitcode.com/openharmony/tee_tee_os_framework) 仓库提供。

### 🔒 安全第一

TEE（可信执行环境）是安全关键组件：

- ✅ 所有代码变更都应经过安全审查
- ✅ Capability 滥用需严格验证
- ✅ Badge 授权检查不能绕过
- ✅ 用户态地址必须经过验证
- ✅ 限制特权系统调用的调用者

---

## 文档更新历史

| 版本 | 日期 | 更新内容 |
|------|--------|----------|
| 1.0 | 2026-02-06 | 初始版本创建完整 Wiki 文档 |

---

**建议反馈**

如果在使用本文档过程中发现：
- ❌ 错误信息
- 🔗 损坏链接
- 📝 内容不清或过时
- 💡 建议改进

请在 [tee_os_framework](https://gitcode.com/openharmony/tee_tee_os_framework) 或本仓库提交 Issue。

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06

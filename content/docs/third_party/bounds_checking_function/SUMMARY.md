# bounds_checking_function 文档阅读指南

## 如何选择阅读路线

### 根据你的角色

#### 👨‍💻 应用开发者

**目标**: 在代码中使用安全函数

**阅读路线**:
1. [01_Overview.md](./01_Overview.md) - 了解库的基本功能（10 分钟）
2. [05_API_Differences.md](./05_API_Differences.md) - 学习 API 使用（20 分钟）
   - 重点关注"安全函数与标准函数的差异"
   - 查看"OpenHarmony 编码规范"
3. 开始编码！

**关键知识点**:
- 使用 `strcpy_s` 替代 `strcpy`
- 始终检查返回值
- 使用 `sizeof(buffer)` 作为大小参数

---

#### 🏗️ 系统开发者/模块维护者

**目标**: 在模块中集成 bounds_checking_function

**阅读路线**:
1. [01_Overview.md](./01_Overview.md) - 了解基础设施地位（10 分钟）
2. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置详解（20 分钟）
   - 查看"在 BUILD.gn 中引用"示例
   - 了解 libsec_shared vs libsec_static 的选择
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 参考其他模块的使用方式（15 分钟）

**关键知识点**:
- 使用 `external_deps` 引用 `bounds_checking_function:libsec_shared`
- 头文件路径已包含在依赖中，无需额外配置

---

#### 🔒 安全工程师

**目标**: 评估安全风险和合规性

**阅读路线**:
1. [01_Overview.md](./01_Overview.md) - 了解安全定位（10 分钟）
2. [06_Security.md](./06_Security.md) - 完整安全风险分析（30 分钟）
   - 安全机制分析
   - CVE 分析
   - 攻击场景分析
3. [02_Patches.md](./02_Patches.md) - 了解无 Patch 策略（10 分钟）

**关键知识点**:
- 无已知 CVE，攻击面小
- 通过边界检查消除 CWE-120/121
- 支持 PAC 等运行时保护

---

#### 📊 架构师

**目标**: 理解库在系统中的位置和依赖关系

**阅读路线**:
1. [01_Overview.md](./01_Overview.md) - 架构位置（15 分钟）
   - 重点看"在 OH 中的关键作用"
   - 查看架构图
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系分析（30 分钟）
   - 查看"依赖关系图"
   - 了解主要依赖者
3. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统（15 分钟）

**关键知识点**:
- 1941+ 个模块依赖，基础设施地位
- c_utils 是核心中转依赖
- 全系统类型、全启动阶段覆盖

---

#### 🔧 维护者/升级负责人

**目标**: 维护库、升级版本

**阅读路线**:
1. [02_Patches.md](./02_Patches.md) - 无 Patch 策略详解（20 分钟）
   - 理解为何无 Patch
   - 查看 Git 提交历史
2. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置（20 分钟）
   - 重点看"与上游的差异"
   - PAC 等安全编译选项
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估数据（10 分钟）

**关键知识点**:
- 无 Patch 文件，升级时直接替换源码
- 保留 BUILD.gn 中的 OH 特有配置
- 关注上游 libboundscheck 的安全更新

---

## 文档详细说明

### [01_Overview.md](./01_Overview.md) - 库概览

**内容**:
- 原始库简介（libboundscheck）
- 核心功能（40 个安全函数）
- 在 OpenHarmony 中的作用和定位
- 典型使用场景

**阅读时间**: 15-20 分钟

**核心图表**:
- OpenHarmony 安全体系层次图
- bounds_checking_function 架构位置图

---

### [02_Patches.md](./02_Patches.md) - Patch 分析

**内容**:
- Patch 清单（本库无 Patch）
- 无 Patch 的原因分析
- Git 提交历史分析
- 虚拟 Patch 概念

**阅读时间**: 15-20 分钟

**关键结论**:
- 无 Patch 文件
- OH 适配通过 BUILD.gn 实现
- 升级维护成本低

---

### [03_Build_Integration.md](./03_Build_Integration.md) - 构建适配

**内容**:
- BUILD.gn 结构详解
- 多系统类型支持（mini/small/standard）
- 关键编译选项（PAC 保护等）
- libsec_src.gni 源文件配置
- SDK 分层配置

**阅读时间**: 20-30 分钟

**核心配置**:
```gn
cflags = [
  "-D_INC_STRING_S",
  "-D_INC_WCHAR_S",
  # ...
]
branch_protector_ret = "pac_ret"
```

---

### [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用

**内容**:
- 依赖统计（1941+ 文件）
- 主要依赖者分类分析
- 依赖关系图（Mermaid）
- 典型使用场景代码示例
- 移除影响评估

**阅读时间**: 30-40 分钟

**核心图表**:
- 整体依赖拓扑图
- c_utils 中转依赖图

---

### [05_API_Differences.md](./05_API_Differences.md) - API 差异

**内容**:
- API 清单（C11 Annex K 标准）
- 安全函数与标准函数对比
- 错误码定义
- OpenHarmony 编码规范
- 常见问题 FAQ

**阅读时间**: 20-30 分钟

**核心表格**:
- 安全函数 vs 标准函数替换表
- 错误码定义表

---

### [06_Security.md](./06_Security.md) - 安全风险分析

**内容**:
- 安全机制分析（边界检查、PAC）
- CVE 分析（无已知 CVE）
- 潜在风险分析
- 攻击场景分析
- 安全升级策略

**阅读时间**: 25-35 分钟

**核心评估**:
- 自身漏洞风险：极低
- 防护有效性：极高
- 无已知 CVE

---

### [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估

**内容**:
- 基础信息读取
- Patch 分析结果
- OH 使用情况统计
- 特殊适配识别
- 关键发现总结

**阅读时间**: 10-15 分钟

**用途**:
- 维护者快速了解项目状态
- 升级评估参考
- 文档编写依据

---

## 时间有限的快速阅读

### ⏱️ 5 分钟速览

阅读 [README.md](./README.md) 的：
- 库概览
- 关键结论
- 快速参考

### ⏱️ 15 分钟核心内容

1. [01_Overview.md](./01_Overview.md) - 阅读"在 OH 中的关键作用"
2. [05_API_Differences.md](./05_API_Differences.md) - 查看"安全函数替换速查表"
3. [README.md](./README.md) - 快速参考部分

### ⏱️ 60 分钟完整理解

按照你的角色选择上面的阅读路线，完整阅读 2-3 个核心文档。

---

## 关联文档

### 外部参考

- [libboundscheck 上游仓库](https://gitee.com/openeuler/libboundscheck)
- [C11 Annex K 标准](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf) (Annex K)
- [OpenHarmony 构建系统文档](https://gitee.com/openharmony/build)

### 相关 Wiki

其他关键第三方库的 Wiki：
- curl - HTTP 客户端库
- openssl - 加密库
- cJSON - JSON 解析库
- sqlite - 数据库引擎

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2025-02 | 初始版本，完整文档集 |

---

## 反馈

如对阅读路线有建议，或发现文档问题，请联系维护团队。
